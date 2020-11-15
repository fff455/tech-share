# 项目依赖分析

> 分析项目的依赖为树图，对比两次上线的 diff，得出受影响的页面，方便进行回归测试

## 背景

平台组件应用情况复杂，模块众多，在修改代码后，如何尽可能准确地回归受影响的模块依赖开发去检查。通过项目依赖的分析，能够：

1. 在上线打包时自动对模块进行分析，解析出受影响的页面，提供用于测试的回归列表
2. 分析出未使用的模块，缩减无用代码
3. 对项目的依赖关系进行分析，了解代码组织结构，为代码重构与拆分提供参考
4. 多页面 repo 而言，通过依赖分析拿到此次需要构建的资源，可以做到单页面发布

## Webpack 依赖解析

### Webpack 对于依赖对处理的过程

webpack 拿到入口文件 entry 后，会通过先获取资源的正确路径，再经过 loader 解析文件，最后通过遍历 ast 来拿到模块中引用的依赖 dependences ，再对 dependences 做递归处理，最终拿到依赖树。 这跟我们最初设想的思路基本一致，同时借助 loader 可以将不同的资源无法解析的问题也一并解决了。

### Why Plugin

webpack 官方已经给出了 webpack-bundle-analyzer 这类的工具了， 每次构建后 stats 中都能拿到文件依赖，不直接使用它的原因是：

1. 构建过程非常缓慢，特别是有几十个页面存在的情况下
2. 我们只是想拿到资源依赖，不想对整个前端 repo 进行一次构建，也不想生成任何 bundle

   所以能不能既可以使用 loader 找到文件依赖，又不需要生成和压缩 bundle 呢，这个时候我们就需要使用 Webpack Plugin

### 编写 Plugin

我们需要的 Plugin 整体结构比较简单，在 apply 函数中，我们将 webpack 的钩子与相应函数进行绑定。我们需要的主要有三个钩子：beforeResolve/afterResolve/finishModules。 在 beforeResolve 中，我们对不需要分析的模块，例如 node\_modules 的模块进行拦截终止。 在 afterResolve 中，将文件及其依赖传入，进行依赖树的构建。 在 finishModules 中，可以使用最终生成的依赖树做一些操作。

```javascript
class DependencesAnalysisPlugin {
  beforeResolve(resolveData, callback) {
    if (analysisBreak(request, issuer)) {
      callback(null, null);
    } else {
      callback(null, resolveData);
    }
  }
  afterResolve(result, callback) {
    const { resourceResolveData } = result;
    const {
      context: { issuer },
      path,
    } = resourceResolveData;
    tree.addDependency(issuer, path);
    callback(null, result);
  }
  handleFinishModules(modules, callback) {
    // handle tree, etc: upload...
    callback(null, modules);
  }
  apply(compiler) {
    compiler.hooks.normalModuleFactory.tap(pluginName, (nmf) => {
      nmf.hooks.beforeResolve.tapAsync(pluginName, this.beforeResolve);
      nmf.hooks.afterResolve.tapAsync(pluginName, this.afterResolve);
    });
    compiler.hooks.compilation.tap(pluginName, (compilation) => {
      compilation.hooks.finishModules.tapAsync(
        pluginName,
        this.handleFinishModules
      );
    });
  }
}
module.exports = DependencesAnalysisPlugin;
```

## 构建依赖树

构建依赖树有几点需要注意：

1. 同一个文件可能被不同的模块进行引用，在模块插入子节点时，应该为所有路径相同的模块插入子模块
2. 可能会出现重复依赖的情况，例如组件 A 引用了组件 B，但是组件 B 同时使用了组件 A 中定义的一些常量

   为了解决第一个问题，在构建树时，我们同时维护一个节点列表，使用节点的引用进行构建树，当插入子节点时，只需要从列表中找到引用，操作引用的子节点。

   当构建完依赖树，还需要进行一步对树的剪枝，在这一步我们解决循环依赖的问题。以树的来看，我们需要的最终结果是在标记节点修改时，要将父节点也同时标记为已修改。在从根结点到叶节点上，不同的节点只需要出现一次，所以做一遍遍历，同一条路径中只保留第一次出现的不重复节点，重复节点进行删除。

## 获取 Git diff

在产出最终的 json 之前，需要获取到更改的文件。通过 打包平台 中的环境变量，我们可以拿到这一次构建的 Commit ID 和分支。通过 打包平台 API，可以拿到分支下构建的历史版本信息。取到两次的 Commit ID，通过 gitlab API，可以获取到两次 commit 之间的 diff，也就能获取到更改的文件列表。

## 完整插件代码

```javascript
const fs = require("fs");
const path = require("path");
const { exec } = require("child_process");
const axios = require("axios");
const { default: routerConfig } = require("../config/router.config");
const pluginName = "DependencesAnalysisPlugin";
const ignoreDependenciesArr = Object.keys(
  require("../package.json").dependencies
);

const analysisBreak = (request, issuer) => {
  return (
    (issuer && issuer.includes("node_modules")) ||
    (request && request.includes("node_modules"))
  );
};

const getRouterInfo = (config) => {
  if (!config.component && (!config.routes || !config.routes.length)) {
    return null;
  }
  const pagePath = path.resolve(__dirname, "../src/pages/");
  const proRoot = path.resolve(__dirname, "../");
  let children = undefined;
  if (config.routes && config.routes.length) {
    children = config.routes.map(getRouterInfo).filter(Boolean);
  }
  return {
    route: config.path,
    path: config.component
      ? path.relative(
          proRoot,
          require.resolve(path.resolve(pagePath, config.component))
        )
      : undefined,
    children,
  };
};

class DependencyTree {
  constructor() {
    this.uniqNodes = [];
  }

  /**
   * 遍历树找到对应的结点列表
   * @param {*} issuer
   */
  traverseTree(issuer) {
    const nodes = this.uniqNodes.filter((n) => n.name === issuer);
    return nodes;
  }

  /**
   * 为结点列表添加子节点
   * @param {*} nodes
   * @param {*} request
   */
  addChildNode(nodes, request) {
    let uniqNode = this.uniqNodes.find((n) => n.name === request);
    if (!uniqNode) {
      uniqNode = {
        name: request,
      };
      this.uniqNodes.push(uniqNode);
    }
    nodes.forEach((node) => {
      if (!node.children) {
        node.children = [uniqNode];
      } else {
        if (!node.children.find((n) => n.name === request)) {
          node.children.push(uniqNode);
        }
      }
    });
  }

  /**
   * 增加一个依赖信息
   * @param {*} issuer
   * @param {*} request
   */
  addDependency(issuer, request) {
    if (!this.root) {
      const firstChildNode = {
        name: request,
      };
      const rootNode = {
        name: issuer,
        children: [firstChildNode],
      };
      this.root = rootNode;
      this.uniqNodes.push(rootNode, firstChildNode);
      return;
    }
    const nodes = this.traverseTree(issuer);
    this.addChildNode(nodes, request);
  }

  /**
   * 遍历树，将同一条链路上的重复出现的结点删除，第一次出现的保留
   */
  clearTree(root, nodes) {
    if (root.changed) {
      nodes.forEach((n) => {
        n.changed = true;
      });
    }
    if (!root.children) {
      return;
    }
    const indexes = root.children
      .map((child) => nodes.find((n) => n.name === child.name))
      .reduce((iter, cur, index) => iter.concat(cur ? index : []), []);
    root.children = root.children.reduce((iter, cur, index) => {
      if (indexes.includes(index)) {
        return iter;
      }
      return iter.concat(cur);
    }, []);
    root.children.forEach((child) => {
      this.clearTree(child, nodes.concat(root));
    });
  }

  getFiles(filePath) {
    const sum = [];
    const files = fs.readdirSync(filePath);
    for (const file of files) {
      const stat = fs.statSync(path.resolve(filePath, file));
      if (stat.isFile()) {
        sum.push(path.resolve(filePath, file));
      } else if (stat.isDirectory()) {
        sum.push(...this.getFiles(path.resolve(filePath, file)));
      }
    }
    return sum;
  }

  unusedFiles() {
    const root = path.resolve(__dirname, "../src/");
    const proRoot = path.resolve(__dirname, "../");
    const proFiles = this.getFiles(root).map((filePath) =>
      path.relative(proRoot, filePath)
    );
    const unused = proFiles.filter(
      (filePath) => !this.uniqNodes.find((node) => node.name === filePath)
    );
    const filePath = `unused.json`;
    fs.writeFileSync(filePath, JSON.stringify(unused, null, 2));
  }

  gitDiffFiles(commitHash, branch) {
    return axios
      .get(
        `xxx` // 获取此次打包的信息
      )
      .then((res) => res && res.data)
      .then((res) => {
        if (!res || !res.results || !res.results.length) {
          return Promise.reject("当前分支无上线版本，无法进行依赖分析");
        }
        const index = res.results.findIndex(
          (r) => r.base_commit_hash === commitHash
        );
        return res.results[
          index === -1 || index >= res.results.length - 1 ? 0 : index + 1
        ].base_commit_hash;
      })
      .then((lastHash) => {
        console.log("Diff hash from: ", lastHash, " to: ", commitHash);
        return axios.get(
          `xxx` // get git diff files from gitlab
        );
      })
      .then((res) => res.data)
      .then((res) => res.diffs.map((diff) => diff.new_path));
  }

  generate(commitHash, branch) {
    const router = getRouterInfo(routerConfig[0]);
    return this.gitDiffFiles(commitHash, branch)
      .then((filesTemp) => {
        const files = filesTemp.filter((p) => p.match("^src/"));
        console.log("Diff files: ", files);
        return files;
      })
      .then((files) => {
        this.uniqNodes.forEach((node) => {
          if (files.includes(node.name)) {
            node.changed = true;
          }
        });
        this.clearTree(this.root, []);
        return this.root;
      })
      .then((data) =>
        axios({
          url: "xxx", // upload result
          method: "POST",
          data: { body: data, commitHash, router, branch },
          headers: { "Content-Type": "application/json" },
        })
      )
      .then((res) => res.data)
      .then((res) => {
        if (res.code === 0) {
          console.log("依赖分析构建成功！");
        } else {
          console.error("依赖分析构建失败：", res.message);
        }
      })
      .catch((err) => {
        console.error("依赖分析构建失败：", err);
      });
  }
}

const tree = new DependencyTree();

class DependencesAnalysisPlugin {
  constructor(options = {}) {
    this.options = options;
  }

  afterResolve(result, callback) {
    const { resourceResolveData } = result;
    const {
      context: { issuer },
      path,
      descriptionFileRoot,
    } = resourceResolveData;
    if (!analysisBreak(path, issuer)) {
      tree.addDependency(
        issuer && issuer.replace(new RegExp(`^${descriptionFileRoot}/`), ""),
        path.replace(new RegExp(`^${descriptionFileRoot}/`), "")
      );
    }
    callback(null, result);
  }

  apply(compiler) {
    compiler.hooks.normalModuleFactory.tap(pluginName, (nmf) => {
      nmf.hooks.afterResolve.tapAsync(pluginName, this.afterResolve);
    });
    compiler.hooks.done.tap(pluginName, (
      stats /* stats is passed as an argument when done hook is tapped.  */
    ) => {
      tree.generate(this.options.commitId, this.options.branch);
    });
  }
}

module.exports = DependencesAnalysisPlugin;
```

