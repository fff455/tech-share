# 纯CSS实现可拖动布局

## 需求简述

* 用于一个多图表的工作台界面，界面以多行多列式分割为多个面板，每个图表放于一块面板的

* 用户能够通过拖动面板边界来自由调节每个面板内图表的大小

* 适用 -webkit 类浏览器

* 完成后效果如下

    <img src="./image/dragdemo.gif" />

## 实例代码

* 代码实现的核心设计：

  1. 使用CSS3新加入的resize属性实现div的宽高可调整

  2. ``flex: 1`` 实现某一随动区域的宽高自适应

  3. 拖动区块与内容区块分开处理，内容层置于拖动层之上，通过``position: absolute``实现内容层随拖动层进行变化

  4. ``-webkit-scrollbar``更改原生拖动区域范围，拖动线的块覆盖原生拖动区域，提高可扩展性，并使用``pointer-events:none``实现事件穿透，保持拖动效果始终生效

* HTML部分

  ```html
  <div class="layout">
    <!-- 第一行分栏，高度可拖动 -->
    <div class="row-fix">
      <div class="row-resize-bar"></div><!-- 第一行高度拖动区域 -->
      <div class="row-resize-line"></div>
      <div class="row-content-bar"><!-- 第一行内容区域 -->
        <!-- 第一列分栏，宽度可拖动 -->
        <div class="col-fix">
          <div class="col-resize-bar"></div><!-- 第一列宽度拖动区域 -->
          <div class="col-resize-line"></div>
          <div class="col-content-bar"><!-- 第一列内容区域 -->
            <div class="panel" style="background:red;">1</div>
          </div>
        </div>
        <!-- 第二列分栏，宽度自适应 -->
        <div class="col-auto">
          <div class="panel" style="background:orange;">2</div>
        </div>
      </div>
    </div>
    <!-- 第二行分栏，高度自适应 -->
    <div class="row-auto">
      <!-- 第一列分栏，宽度可拖动 -->
      <div class="col-fix">
        <div class="col-resize-bar"></div>
        <div class="col-resize-line"></div>
        <div class="col-content-bar">
          <div class="panel" style="background:lightgreen;">3</div>
        </div>
      </div>
      <!-- 第二列分栏，宽度自适应 -->
      <div class="col-auto">
        <div class="panel" style="background:skyblue;">4</div>
      </div>
    </div>
  </div>
  ```

* CSS部分

  ```css
  .layout {
    height: 100vh;
    width: 100vw;
    display: flex;
    flex-direction: column; /* 多行式拖动，若为row则为多列拖动，可通过组件props控制 */
    overflow: hidden;
  }

  .row-fix {
    position: relative; /* 拖动区块基准 */
    width: 100%;
  }

  .row-auto {
    flex: 1; /* 自适应区块，使用flex实现自适应 */
    display: flex;
    overflow: hidden;
  }

  .row-resize-bar {
    height: 400px;
    min-height: 300px;
    max-height: 600px;
    width: inherit;
    background: #eee;
    resize: vertical; /* 纵向拖动 */
    overflow: scroll;
  }

  /* 更改原生可拖动区域范围 */
  .row-resize-bar::-webkit-scrollbar {
    height: 10px;
    width: inherit;
  }

  .row-content-bar {
    height: calc(100% - 10px);
    position: absolute; /* 内容区域，随拖动区块进行变化 */
    top: 0;
    right: 0;
    bottom: 10px;
    left: 0;
    overflow-y: hidden;
    background: #fff;
    display: flex;
    /* opacity: 0; 拖动线替换方案 */
  }

  /* 用于覆盖可拖动区域的拖动线，也可以直接使用上述opacity属性，考虑样式扩展性而使用拖动线 */
  .row-resize-line {
    position: absolute;
    right: 0;
    left: 0;
    bottom: 0;
    width: inherit;
    height: 10px;
    pointer-events: none; /* 鼠标事件穿透，使拖动线底部的拖动区域的resize功能生效 */
    background: #eee;
  }

  /* 以下为每列的拖动样式，与行的大同小异 */
  .col-fix {
    position: relative;
    height: 100%;
  }

  .col-resize-bar {
    width: 400px;
    min-width: 300px;
    max-width: 600px;
    height: inherit;
    background: #eee;
    resize: horizontal; /* 横向拖动 */
    overflow: scroll;
  }

  .col-resize-line {
    position: absolute;
    right: 0;
    top: 0;
    bottom: 0;
    height: inherit;
    width: 10px;
    pointer-events: none;
    background: #eee;
  }

  .col-content-bar {
    width: calc(100% - 10px);
    position: absolute;
    top: 0;
    right: 10px;
    bottom: 0;
    left: 0;
    overflow-x: hidden;
    background: #fff;
    display: flex;
    flex-direction: column;
  }

  .col-resize-bar::-webkit-scrollbar {
    width: 10px;
    height: inherit;
  }

  .col-auto {
    flex: 1;
    overflow: hidden;
  }
  ```

## 参考资料

* [纯CSS实现分栏宽度拉伸调整](https://www.zhangxinxu.com/study/201903/css-idea/behavior-stretch.php?aside=0)