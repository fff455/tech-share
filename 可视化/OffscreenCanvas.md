# OffscreenCanvas

Canvas 计算和渲染和用户操作响应都发生在同一个线程中，在动画中（有时候很耗时）的计算操作将会导致 App 卡顿，降低用户体验。Canvas 的绘制功能都与<canvas>标签绑定在一起，这意味着 Canvas API 和 DOM 是耦合的。而 OffscreenCanvas，正如它的名字一样，通过将 Canvas 移出屏幕来解耦了 DOM 和 Canvas API。
由于这种解耦，OffscreenCanvas 的渲染与 DOM 完全分离了开来，并且比普通 Canvas 速度提升了一些，而这只是因为两者（Canvas 和 DOM）之间没有同步。但更重要的是，将两者分离后，Canvas 将可以在 Web Worker 中使用，即使在 Web Worker 中没有 DOM。这给 Canvas 提供了更多的可能性。

## API 介绍

### 创建 OffscreenCanvas

<b>OffscreenCanvas()</b>

创建一个 OffscreenCanvas 有两种方式，一种是直接使用 OffscreenCanvas 的构造函数：

```js
const offscreen = new OffscreenCanvas(width, height);
HTMLCanvasElement.transferControlToOffscreen();
```

另外一种方式，是使用 canvas 的 transferControlToOffscreen 函数获取一个 OffscreenCanvas 对象，绘制该 OffscreenCanvas 对象，同时会绘制 canvas 对象。
canvas 对象调用了函数 transferControlToOffscreen 移交控制权之后，不能再获取绘制上下文，调用 canvas.getContext('2d')会报错； 同样的原理，如果 canvas 已经获取的绘制上下文，调用 transferControlToOffscreen 会报错。

```js
const canvas = document.getElementById("canvas");
const offscreen = canvas.transferControlToOffscreen();
```

### ImageBitmap

ImageBitmap 接口表示能够被绘制到 <canvas> 上的位图 p 图像，具有低延迟的特性。运用 createImageBitmap() 工厂方法模式，它可以从多种源中生成。 ImageBitmap 提供了一种异步且高资源利用率的方式来为 WebGL 的渲染准备基础结构。

<b>ImageBitmapRenderingContext</b>

ImageBitmapRenderingContext 是 canvas 的绘制上下文，能够提供使用 ImageBitmap 替换 canvas 的能力，可以在 window 或者 worker 中使用。

<b>ImageBitmapRenderingContext.transferToImageBitmap()</b>

通过 transferToImageBitmap 函数可以从 OffscreenCanvas 对象的绘制内容创建一个 ImageBitmap 对象。该对象可以用于到其他 canvas 的绘制。

<b>ImageBitmapRenderingContext.transferFromImageBitmap()</b>

通过 transferFromImageBitmap 函数可以在上下文对应的 canvas 中绘制 ImageBitmap 对象。该 ImageBitmap 对象的所有权同时转移到画布上。

## 同步显示 OffscreenCanvas

同步显示的思路就是渲染的代码使用 offscreen，渲染完成后，通过 ImageBitmap 的能力将绘制结果同步到展示的 canvas 中

```js
const canvasBitMapCtx = document
  .getElementById("canvas")
  .getContext("bitmaprenderer");
const offscreen = new OffscreenCanvas(width, height);
const ctx = offscreen.getContext("2d");
// do some drawing with ctx...
const bitmap = offscreen.transferToImageBitmap();
canvasBitMapCtx.transferFromImageBitmap(bitmap);
```

## 异步显示 OffscreenCanvas

异步显示的思路是将展示的 canvas 的控制权移交给 OffscreenCanvas，然后直接操作 OffscreenCanvas 的渲染上下文进行绘制，绘制结果会直接显示在展示的 canvas 中

```js
const canvas = document.getElementById("canvas");
const offscreen = canvas.transferControlToOffscreen();
// 使用 offscreen 中的上下文直接绘制
const ctx = offscreen.getContext("2d");
// do some drawing with ctx...
```

## 在 Worker 中使用

Worker 中使用 OffscreenCanvas 也是上文的两种思路，同步或者异步。在 Worker 中渲染的好处是不会占用主线程，能够在计算和绘制的过程中不阻塞页面的交互，也能够提升一定的性能。
同步
同步使用 OffscreenCanvas 是将展示的 canvas bitmaprenderer 上下文发送到 worker 中，worker 进行绘制的过程中不断将 ImageBitmap 传回主线程，主线程中通过 ImageBitmap 进行展示 canvas 的更新。
异步
将展示的 canvas 的控制权移交给 OffscreenCanvas，然后将 OffscreenCanvas 发送给 Worker，计算和渲染的操作都交给 Worker 当中。由于移交控制权的方式能直接控制到绘图上下文，Worker 中不再需要向主线程中回传，直接使用上下文绘制即可。
