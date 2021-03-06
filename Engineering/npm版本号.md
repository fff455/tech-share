# npm 版本号

为了在软件版本号中包含更多意义，反映代码所做的修改，产生了语义化版本，软件的使用者能从版本号中推测软件做的修改。npm 包使用语义化版控制，我们可安装一定版本范围的 npm 包，npm 会选择和你指定的版本相匹配 的 (latest)最新版本安装。
npm 的版本号由三部分组成：
主版本号、次版本号、补丁版本号。变更不同的版本号，代表不同的意义：

主版本号（major）：软件做了不兼容的变更（breaking change 重大变更）；
次版本号（minor）：添加功能或者废弃功能，向下兼容；
补丁版本号（patch）：bug 修复，向下兼容。

## 标签/扩展

有时候为了表达更加确切的版本，还会在版本号后面添加标签或者扩展，来说明是预发布版本或者测试版本等。比如 3.2.3-beta-3。

常见的标签有 :

| 标签   | 意义           | 补充                                                                                                                               |
| ------ | -------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| demo   | 版本           | 可能用于验证问题的版本                                                                                                             |
| dev    | 开发版         | 开发阶段用的，bug 多，体积较大等特点，功能不完善                                                                                   |
| alpha  | α 版本         | 用于内部交流或者测试人员测试，bug 较多                                                                                             |
| beta   | 测试版(β 版本) | 较 α 版本，有较大的改进，但是还是有 bug                                                                                            |
| gamma  | （γ）伽马版本  | 较 α 和 β 版本有很大的改进，与稳定版相差无几，用户可使用                                                                           |
| trial  | 试用版本       | 本软件通常都有时间限制，过期之后用户如果希望继续使用，一般得交纳一定的费用进行注册或购买。有些试用版软件还在功能上做了一定的限制。 |
| stable | 稳定版         |                                                                                                                                    |
| csp    | 内容安全版本   | js 库常用                                                                                                                          |
| latest | 最新版本       | 不指定版本和标签，npm 默认安最新版                                                                                                 |

### 安装带标签的版本

```bash
npm i <pkg>@<tag>
npm i vue@beta # 安装 2.6.0-beta.3
```

## 版本号变更规则

1. 版本号只升不降，不得在数字前加 0，比如 2.01.2 不允许的；
2. 0.y.z，处于开发阶段的版本；
3. 第一个正式版版本往往命名为 1.0.0；
4. 先行版本必须在补丁版本之后添加，比如 2.3.7-0,0 表示先行版本，和补丁版本用-分隔；
5. 版本的比较依次比较主版本 → 次版本 → 补丁版本 → 先行版本，直到第一个能得出比较结果为止；
6. 不小心把一个不兼容的改版当成了次版本号发行了该怎么办？一旦发现自己破坏了语义化版本控制的规范，就要修正这个问题，并发行一个新的次版本号来更正这个问题并且恢复向下兼容。即使是这种情况，也不能去修改已发行的版本。

## 更新版本号

```bash
npm version [<newversion> | major | minor | patch | premajor | preminor | prepatch | prerelease | from-git]
```

newversion: 直接给一个版本号；
major: 主版本增加 1；
premajor: 预备主版本，主版本增加 1，增加先行版本号；
prelease: 预先发布版本，先行版本号增加 1；
git 和 npm version 结合

## 版本运算符

版本运算符指定了一定范围的版本。
主要有~、^、-、<、<=、>、>=、=版本运算符。

### ~ 版本号 ----- 指定主版本号或者次版本号相同

~ + 只含主版本 --- 主版本相同；
~ + 含有次版本 --- 主版本和次版本号相同。

| 版本范围 | 匹配版本                     |
| -------- | ---------------------------- |
| ~3       | 3.x 或者 3.0.0 <= v < 4.0.0  |
| ~3.1     | 3.1.x 或者 3.1.0 <= v <3.2.0 |
| ~3.1.2   | 3.1.2 < v < 3.2.0            |

指定的版本范围含有预发布版本，只会匹配和完整版本号相同的预发布版本。
~3.1.3-beta.2 匹配 3.1.3-beat.3 不匹配 3.1.4-beat-2

```bash
npm i lodash@~3 # 安装 3.10.1
npm i lodash@~3.9 # 安装 3.9.3
npm i lodash@~3.9.1 # 安装 3.9.3
npm i lodash@~3.8.0 # 安装 3.8.0
```

### ^ 版本号 --- 第一个非零 版本号相同

|版本范围| 匹配版本| 补充|
|^3.1.5 |3.1.5 <= v < 4.0.0||
|^0.3.6 |0.3.6 <= v < 0.4.0||
|^0.0.2 |0.0.2 <= v < 0.0.3||
|^3.x.x |3.0.0 <= v < 4.0.0 |版本号缺少的位置，会被 0 填充|
|^4.2.x |4.2.0 <= v < 4.3.0||

npm 安装包时，默认使用 ^ 匹配版本。

安装主版本号为 3 的最新版本：

```bash
npm i lodash@^3 # 安装 3.10.1
npm i lodash@^3.9 # 安装 3.10.1
npm i lodash@^3.8.0 # 安装 3.10.1
```

### ~ vs ^

|版本范围 |含义| 匹配的版本 |说明|
|~3.3.0 与 3.3.0 |相似 |3.3.0 <= v < 3.4.0 |主版本和次版本相同|
|^3.3.0 与 3.3.0 |兼容 |3.3.0 <= v < 4 |主版本相同|

同一个版本号，^ 能匹配的范围大些，更加激进。

例子

```bash
npm i lodash@^3.3.0 # 安装 3.10.1
npm i lodash@~3.3.0 # 安装 3.3.1
```

~ 和 ≈ 差不多，可将 ~ 理解成相似，这样就分辨和理解了，~指定的是相似版本。
^ 可理解成兼容版本。

### 指定精确范围

| 版本范围      | 匹配版本            | 补充                    |
| ------------- | ------------------- | ----------------------- |
| 2.0.0 - 3.2.7 | 2.0.0 <= v <= 3.2.7 | - 前后有空格            |
| 0.4 - 3       | 0.4.0 <= v <= 3.0.0 | 缺少的版本号，被 0 填充 |

```bash
npm i vue@"1 - 1.9" # 安装 1.0.28
```

| 版本范围 | 匹配版本              |
| -------- | --------------------- |
| <2.2.0   | 小于 2.2.0 的版本     |
| <=2.0.0  | 小于等于 2.0.0 的版本 |

> 4.2.0 大于 4.2.0 的版本  
> =4.2.0 大于等于 4.2.0 的版本  
> =4.3.0 等于 4.3.0 的版本

```bash
npm i lodash@\<3.5 # 安装 3.4.0
npm i lodash@\<=3.5 # 安装 3.5.0
npm i lodash@\>3.5 # 安装 4.17.11
npm i lodash@\>=3.5 # 安装 4.17.11
npm i vue@">1 <2.3" # 安装 2.2.6
```

## 分组 ||

以或者的关系连接两个版本范围，极少使用。

```bash
npm i vue@"^0.7 || ~2" # 安装 2.6.10
```
