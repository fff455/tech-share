# Canvas 图片处理

## Canvas 可处理的图片源

1. 通过 Image\(\)创建的对象
2. 通过 ImageBitmap\(\)创建的对象
3. dom 元素
4. dom 元素
5. dom 元素

## Canvas 图片 API

### drawImage

获取到图片源后，可以调用这个方法将图片源绘制到 canvas 中

* drawImage\(image, x, y\)

image 是 image 或者 canvas 对象，x 和 y 是其在目标 canvas 里的起始坐标。

* rotate

用于以原点为中心旋转 canvas 画板

* ImageData

ImageData 对象中存储着 canvas 对象真实的像素数据，它包含以下几个只读属性：

width: 图片宽度，单位是像素 height: 图片高度，单位是像素 data: Uint8ClampedArray 类型的一维数组，包含着 RGBA 格式的整型数据，范围在 0~255

## 变形

### 缩放/裁切

* 缩放 drawImage\(image, x, y, width, height\)

width 和 height，这两个参数用来控制 当向 canvas 画入时应该缩放的大小

* 切片 drawImage\(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight\)

sx, sy, sWidth, sHeight 是图片源的坐标/宽高信息

dx, dy, dWidth, dHeight 是绘制到 canvas 的坐标/宽高信息

### 旋转

```javascript
// 将参照点移动到画板的中心点；
ctx.translate(ctx.width / 2, ctx.height / 2);
// 旋转画板；
ctx.rotate = 90;
// 绘制图片；
ctx.drawImage(img);
```

## 图片效果

### 反色

反色即对每个像素点进行如下操作，公式：

```javascript
b = 255 - a;
```

JS 代码

```javascript
const imageData = cxt.getImageData(0, 0, canvas.width, canvas.height);
const imageData_length = imageData.data.length / 4; //一个像素点用4位:r,g,b,a
// 解析之后进行算法运算
for (let i = 0; i < imageData_length; i++) {
  imageData.data[i * 4] = 255 - imageData.data[i * 4];
  imageData.data[i * 4 + 1] = 255 - imageData.data[i * 4 + 1];
  imageData.data[i * 4 + 2] = 255 - imageData.data[i * 4 + 2];
}
cxt.putImageData(imageData, 0, 0);
```

### 去色

去色即将彩色图片变成黑白图片

公式：

```javascript
Gray = Red * 0.3 + Green * 0.59 + Blue * 0.11;
```

JS 代码

```javascript
const imageData = cxt.getImageData(0, 0, canvas.width, canvas.height);
const imageData_length = imageData.data.length / 4;
// 解析之后进行算法运算
for (let i = 0; i < imageData_length; i++) {
  const red = imageData.data[i * 4];
  const green = imageData.data[i * 4 + 1];
  const blue = imageData.data[i * 4 + 2];
  const gray = 0.3 * red + 0.59 * green + 0.11 * blue;
  imageData.data[i * 4] = gray;
  imageData.data[i * 4 + 1] = gray;
  imageData.data[i * 4 + 2] = gray;
}
cxt.putImageData(imageData, 0, 0);
```

### 单色

单色处理即将图片变为纯 red/green/blue 的纯色图，操作即将其余两色位 置为 0

### 二值处理

二值处理是非纯黑即纯白，在验证码 ocr 时，常用二值处理先处理，再将图片进行 ocr

算法是给定一个设置值，如果进行去色处理的结果高于这个值，就将颜色设为 255，否则设为 0

