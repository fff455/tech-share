# 什么是BFC
> BFC(Block Fomatting Context)是一个经常能够见到的css概念，直译为"块级格式化上下文"。它是一个独立的渲染区域，只有Block-level box参与，它规定了内部的Block-level Box如何布局，并且与这个区域外部毫不相干。

## Box Model 与 Fomatting Context

* Box Model: 也就是我们常说的盒模型，其本质上为一个盒子，来封装周围的HTML元素。我们可以将一个盒子想象为CSS布局的基本单位，多个的盒子构成了整个界面的样式布局。元素的display属性决定了Box的类型，不同类型的Box意味着不同的渲染方式，比如耳熟能详的块级元素与行内元素。

  * block-level box : 如display为block、table、list-item的元素；
  
  * inline-level box : display为inline、inline-block、inline-table的元素

* Formatting context : 格式化上下文，其实就是对上文提到的，Box渲染定位方式的一个官方的命名概念。最常见的就有Block Fomatting Context(BFC)、Inline Fomatting Conext(IFC)，一般来说，这两种渲染定位方式中的Box就对应于上文中提到的两种Box类型。当然其他还有CSS3引入的Flex Fomatting Context(FFC)，对应于弹性盒子，display为flex、inline-flex的元素。GFC(Grid Layout Context)对应于display为grid的元素。

## Block Fomatting Conext

### 1. BFC的布局规则

在了解了Box与Formatting Context之后，我们就能知道，BFC就是一种为一块区域的总称，包括区域的布局机制、区域内的元素等，这块区域内的元素都由块级元素构成，且这块区域与外界的布局无关。BFC有如下的布局机制：

  * 内部的Box会在垂直方向，一个接一个地放置。

  * Box垂直方向的距离由margin决定。属于同一个BFC的两个相邻Box的margin会发生重叠。

  * 每个盒子（块盒与行盒）的margin box的左边，与包含块border box的左边相接触(对于从左往右的格式化，否则相反)。即使存在浮动也是如此。

  * BFC的区域不会与float box重叠。

  * BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。

  * 计算BFC的高度时，浮动元素也参与计算。

### 2. 如何创建一个BFC

我们需要区别两个概念，“创建BFC”和“构成BFC”:

  * 首先BFC本身是一个区域，区域也是一个元素，这个元素属性满足一定的条件（也就是下文列举的条件）后，他就是**创建一个BFC**，是具有BFC机制的区域。
  
  * 区域中的子元素具有**构成BFC**的条件，如display属性为block的元素。对这个区域而言，如果它的display为block，那么它就是构成一个更大BFC的子元素。

以下列举的是摘自MDN官方文档的一部分创建BFC的规则：

  * 根元素(<html\>)

  * 浮动元素（元素的 float 不是 none）

  * 绝对定位元素（元素的 position 为 absolute 或 fixed）

  * 行内块元素（元素的 display 为 inline-block）

  * 表格单元格（元素的 display为 table-cell，HTML表格单元格默认为该值）

  * 表格标题（元素的 display 为 table-caption，HTML表格标题默认为该值）

  * 匿名表格单元格元素（元素的 display为 table、table-row、 table-row-group、table-header-group、table-footer-group（分别是HTML table、row、tbody、thead、tfoot的默认属性）或 inline-table）

  * overflow 值不为 visible 的块元素
  
  * display 值为 flow-root 的元素

  * 弹性元素（display为 flex 或 inline-flex元素的直接子元素）

  * 网格元素（display为 grid 或 inline-grid 元素的直接子元素）

## BFC的作用

### 1. 处理CSS外边距合并

* CSS外边距合并 : 又可以称为margin塌陷或者margin越界。其实在BFC布局规则的第二点中也提到了。在BFC布局规则下，两个相邻的且垂直上下分布的元素之间，当他们的外边距相遇，如上元素的margin-bottm与下元素的margin-top相遇，那么这两者之间的外边距就会被合并。w3c上对CSS外边距合并问题做了一个很详细的介绍，[CSS外边距合并](https://www.w3school.com.cn/css/css_margin_collapsing.asp)。

那么问题来了，为什么BFC的布局规则是引起外边距合并的原因，如何又用BFC来解决。BFC有一个非常重要的规则，那就是内外部相隔离，内外样式不会互相影响。如果两个会发生margin塌陷的子元素本身也是一个BFC，由于BFC内部的样式不会被外部影响，那么也就解决了这一问题。

### 2. 阻止元素被浮动元素覆盖

在一个正常的文档流下，两个兄弟元素之间，block元素可能会被一个float元素所覆盖，挤占正常的文档流。此时，通过改变这两个元素所在的区域为BFC，即可避免这一问题的发生。

### 3. 可以包含浮动元素

这个包含可以理解为物理意义的包括与涵盖，对于一个区域来说，它的高度可以自己设置，可以通过子元素撑开。如果它内部的子元素为浮动元素。由于浮动元素脱离了正常的文档流，所以哪怕浮动元素有高度，这块区域也无法被撑开。如果这块区域为一个BFC，那么这块区域依旧能够被正常撑开。