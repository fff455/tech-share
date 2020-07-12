# NPM Dependency

node 项目中 package.json 中 dependencies 有几种不同的形式: dependencies/devDependencies/peerDependencies/optionalDependencies/bundleDependencies

## dependencies

dependencies 是开发中最常用的形式，dependencies 即项目的必需依赖。安装 dependencies 命令:

```bash
npm i A --save
```

或者

```bash
yarn add A
```

假设我们在开发的是项目 A，A 的 package.json 如下：

```json
{
  "name": "A",
  "dependencies": {
    "B": "^1.0.0"
  }
}
```

那么，假设项目 C 依赖的项目 A，那么进行安装时，会将 A 的 dependencies 也进行安装。

## devDependencies

项目 A 的 devDependencies 指在项目 A 开发中必需，但是其他项目依赖 A 时不必安装这个包，一般做测试、打包、ES6 转 ES5 此类的工作所依赖的库就使用 devDependencies。安装 devDependencies 命令:

```bash
npm i A --save-dev
```

或者

```bash
yarn add A --dev
```

举个例子，项目 A 在开发中需要建设 Example，选用 StoryBook 进行假设，那么 StoryBook 在项目 A 中就是一个 devDependencies，因为项目 A 在开发中使用了这一个依赖，必需安装它才可以进行 Example 的编写/运行/部署。但是其他项目依赖 A 时不需要安装 StoryBook，因为依赖 A 的是其核心功能文件，并不会引用到它的 Example。

## peerDependencies

项目 A 中的 peerDependencies 在项目单独运行中是不会被安装的，它的含义是指其他包在依赖项目 A 时，也应该同时安装的其他包，不然在使用 A 时会出现问题。

这个需要仔细考虑一下，为什么不直接把项目 A 的 peerDependencies 作为 A 的 dependencies 呢，这样就能保证安装 A 时也能把 A 需要的包给装上。

这是因为各个包里的依赖版本往往是不同的，比如假设项目中使用了 React，其他依赖中也把 React 作为了 dependencies，那么实际安装中就会安装多个版本的 React。实践中，安装不同版本的 React 会触发运行时错误。所以，可以使用 peerDependencies 来规避这个错误。例如 antd，它不将 React 作为项目的 dependencies，而是作为项目的 peerDependencies 和 devDependencies，这样就能保证 antd 在开发中会安装 react，而且如果使用 antd 的项目中没有安装 react，也是无法使用的。

## bundleDependencies

bundleDependencies 同 bundledDependencies，这个配置的作用如下：

假设项目 A 的 package.json 如下

```json
{
  "name": "A",
  "dependencies": {
    "B": "^1.0.0",
    "C": "^1.0.0",
    "D": "^1.0.0"
  }
}
```

项目的 node_modules 的文件结构为：

```
├── node_modules
 └── A
 └── B
 └── C
 └── D
```

如果使用 bundleDependencies，

```json
{
  "name": "A",
  "dependencies": {
    "B": "^1.0.0",
    "C": "^1.0.0",
    "D": "^1.0.0"
  },
  "bundleDependencies": ["B", "C"]
}
```

项目的 node_modules 的文件结构为：

```
├── node_modules
 └── A
  └── node_modules
    └── B
    └── C
 └── D
```

bundleDependencies 的作用就是在用户安装了项目之后，将项目所声明的依赖包汇总到项目自身的 node_modules 下，便于用户管理。

## optionalDependencies

如果你的 node 项目依赖了一个包 package-optional，假如这个 package-optional 没有安装，你仍然想让程序正常执行，这个时候 optionalDependencies 就非常适合你这个需求，optionalDependencies 跟 dependencies 声明方式完全一致，而且一个依赖如果同时在 dependencies 和 optionalDependencies 中声明，option 还会覆盖 dependencies 的声明。optionalDependencies 一个使用的场景为项目 A 本身引入了某些项目 ts 的 type 包，但是在将这个项目 A 为依赖的项目里没有引入这些 type 包，那么如果直接作为 dependencies 的话，使用项目 A 的项目会因为缺少类型定义而报错，但是实际情况是即使 ts 定义不完善，也能够使用项目 A，这时，type 等包就应该作为项目 A 的 optionalDependencies。
