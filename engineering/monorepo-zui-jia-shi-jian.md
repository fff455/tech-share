# Monorepo 最佳实践

## Lerna

Lerna 是一个管理多个 npm 模块的工具，是 Babel 自己用来维护自己的 Monorepo 并开源出的一个项目。优化维护多包的工作流，解决多个包互相依赖，且发布需要手动维护多个包的问题。Lerna 现在已经被很多著名的项目组织使用，如：Babel, React, Vue, Angular, Ember, Meteor, Jest 。一个基本的 Lerna 管理的仓库结构如下：安装推荐全局安装，因为会经常用到 lerna 命令

```bash
npm i -g lerna
```

### 初始化 lerna init

lerna init 其中 package.json & lerna.json 如下

```javascript
// package.json
{
  "name": "root",
  "private": true, // 私有的，不会被发布，是管理整个项目，与要发布到npm的解耦
  "devDependencies": {
    "lerna": "^3.15.0"
  }
}
```

```javascript
// lerna.json
{
  "packages": ["packages/*"],
  "version": "0.0.0"
}
```

### 增加 packages

```bash
lerna create A
```

### 给相应的 package 增加依赖模块

```bash
lerna add chalk // 为所有 package 增加 chalk 模块
lerna add semver --scope A // 为 A 增加 semver 模块
lerna add B --scope A // 增加内部模块之间的依赖
```

### 发布 lerna publish

lerna 会让你选择要发布的版本号，我发了@0.0.1-alpha.0 的版本。发布 npm 包需要登陆 npm 账号

### 安装依赖包 & 清理依赖包上述

1-4 步已经包含了 Lerna 整个生命周期的过程了，但当我们维护这个项目时，新拉下来仓库的代码后，需要为各个 package 安装依赖包。我们在第 4 步 lerna add 时也发现了，为某个 package 安装的包被放到了这个 package 目录下的 node\_modules 目录下。这样对于多个 package 都依赖的包，会被多个 package 安装多次，并且每个 package 下都维护 node\_modules ，也不清爽。于是我们使用 --hoist 来把每个 package 下的依赖包都提升到工程根目录，来降低安装以及管理的成本

```bash
lerna bootstrap --hoist
```

为了省去每次都输入 --hoist 参数的麻烦，可以在 lerna.json 配置：

```javascript
{
  "packages": ["packages/*"],
  "command": {
    "bootstrap": {
      "hoist": true
    }
  },
  "version": "0.0.1-alpha.0"
}
```

配置好后，对于之前依赖包已经被安装到各个 package 下的情况，我们只需要清理一下安装的依赖即可：

```bash
lerna clean
```

然后执行 lerna bootstrap 即可看到 package 的依赖都被安装到根目录下的 node\_modules 中了

lerna 不负责构建，测试等任务，它提出了一种集中管理 package 的目录模式，提供了一套自动化管理程序，让开发者不必再深耕到具体的组件里维护内容，在项目根目录就可以全局掌控，基于 npm scripts，使用者可以很好地完成组件构建，代码格式化等操作。接下来我们就来看看，如何基于 Lerna，并结合其它工具来搭建 Monorepo 项目的最佳实践。

## 优雅的提交

### commitizen && cz-lerna-changelogcommitizen

commitizen 是用来格式化 git commit message 的工具，它提供了一种问询式的方式去获取所需的提交信息。cz-lerna-changelog 是专门为 Lerna 项目量身定制的提交规范，在问询的过程，会有类似影响哪些 package 的选择。如下：我们使用 commitizen 和 cz-lerna-changelog 来规范提交，为后面自动生成日志作好准备。因为这是整个工程的开发依赖，所以在根目录安装：

```bash
npm i -D commitizen
npm i -D cz-lerna-changelog
```

安装完成后，在 package.json 中增加 config 字段，把 cz-lerna-changelog 配置给 commitizen。同时因为 commitizen 不是全局安全的，所以需要添加 scripts 脚本来执行 git-cz

```javascript
{
  "name": "root",
  "private": true,
  "scripts": {
    "c": "git-cz"
  },
  "config": {
    "commitizen": {
      "path": "./node_modules/cz-lerna-changelog"
    }
  },
  "devDependencies": {
    "commitizen": "^3.1.1",
    "cz-lerna-changelog": "^2.0.2",
    "lerna": "^3.15.0"
  }
}
```

之后在常规的开发中就可以使用 npm run c 来根据提示一步一步输入，来完成代码的提交。

### commitlint && husky

上面我们使用了 commitizen 来规范提交，但这个要靠开发自觉使用 npm run c 。万一忘记了，或者直接使用 git commit 提交怎么办？答案就是在提交时对提交信息进行校验，如果不符合要求就不让提交，并提示。校验的工作由 commitlint 来完成，校验的时机则由 husky 来指定。husky 继承了 Git 下所有的钩子，在触发钩子的时候，husky 可以阻止不合法的 commit,push 等等。// 安装 commitlint 以及要遵守的规范

```bash
npm i -D @commitlint/cli @commitlint/config-conventional
```

在工程根目录为 commitlint 增加配置文件 commitlint.config.js 为 commitlint 指定相应的规范

```javascript
module.exports = { extends: ["@commitlint/config-conventional"] };
```

安装 husky

```bash
npm i -D husky
```

在 package.json 中增加如下配置

```javascript
{
  "husky": {
    "hooks": {
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
  }
}
```

"commit-msg"是 git 提交时校验提交信息的钩子，当触发时便会使用 commitlit 来校验。安装配置完成后，想通过 git commit 或者其它第三方工具提交时，只要提交信息不符合规范就无法提交。从而约束开发者使用 npm run c 来提交。

### lint-staged

除了规范提交信息，代码本身肯定也少了靠规范来统一风格。lint-staged staged 是 Git 里的概念，表示暂存区，lint-staged 表示只检查并矫正暂存区中的文件。一来提高校验效率，二来可以为老的项目带去巨大的方便。 安装

```bash
npm i -D lint-staged
```

package.json

```javascript
{
  "name": "root",
  "private": true,
  "scripts": {
    "c": "git-cz"
  },
  "config": {
    "commitizen": {
      "path": "./node_modules/cz-lerna-changelog"
    }
  },
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged",
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
  },
  "devDependencies": {
    "@commitlint/cli": "^8.1.0",
    "@commitlint/config-conventional": "^8.1.0",
    "commitizen": "^3.1.1",
    "cz-lerna-changelog": "^2.0.2",
    "husky": "^3.0.0",
    "lerna": "^3.15.0",
    "lint-staged": "^9.2.0"
  }
}
```

安装完成后，在根目录创建 lint-staged.config.js，在其中设计 eslint/tsc/prettier/stylelint 等命令，表示对暂存区中的文件执行校验。那什么时候去校验呢，就又用到了上面安装的 husky ，husky 的配置中增加'pre-commit'的钩子用来执行 lint-staged 的校验操作，如上所示。此时提交 js 文件时，便会自动修正并校验错误。即保证了代码风格统一，又能提高代码质量。

