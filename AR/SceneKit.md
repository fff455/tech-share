# SceneKit

SceneKit 允许开发人员在 3D 场景中构建和操作 3D 对象。在 AR 镜头下，如果想要添加 2D/3D 图形/物体就要用到 Scenekit

在 AR 场景中，坐标空间是一个 x,y,z 三维坐标下的空间\(y 坐标朝向上方\)。空间中的一个点可以用一个向量\(x,y,z\)来表示，在 SceneKit 中，是 SCNVector3。

## SCNNode

如果想要向 AR 场景中添加一个图形或者物体，我们需要用到 SCNNode，每个 2D/3D 物体都是一个 SCNNode。它本身不表示任何形状，而是通过构造函数`SCNNode(geometry: xxx)`中的`geometry`来指明使用什么形状来展示，`geometry`的类型是`SCNSphere`/`SCNGeometry`。在有了形状之后，使用属性`.position: SCNVector3`可以指定节点所在的位置。

### 画一个点

使用`SCNSphere(radius: Float)`可以创建一个点，结合 SCNNode 的属性`.position`可以指定点所在的位置。

### 画一条线

使用`SCNGeometry(sources: [SCNGeometrySource], elements: [SCNGeometryElement])`可以创建一条线。

`SCNGeometryElement(indices,primitiveType)`构造函数的第一个参数是一个坐标点的集合，第二个参数是 element 的类型，在这里，因为一条线需要两个顶点确定，所以 indices 就是`[0, 1]`，类型就是`.line`。

`SCNGeometrySource(vertices)`构造函数的第一个参数是用到描述物体的一系列点\(向量\)，因为一条线需要两个顶点确定，所以 vertices 就是两个顶点。

创建线类型的一个函数

```swift
let indices: [Int32] = [0, 1]
let source = SCNGeometrySource(vertices: [vector1, vector2])
let element = SCNGeometryElement(indices: indices,primitiveType: .line)
let line = SCNGeometry(sources: [source], elements: [element])
let node = SCNNode(geometry: line)
return node
```

### 创建 3D 物体

SceneKit 内置了一些 3D 物体，比较简单的像 SCNBox 可以创建一个立方体，SCNCylinder 可以创建一个圆柱体。创建难度都不大，按照 API 进行创建即可，以下是一个创建圆柱体的例子。

```swift
let cylinder = SCNCylinder(radius: 0.01, height: height)
let node = SCNNode(geometry: cylinder)
node.position = SCNVector3(x, y, z)
```

### 创建一个平面

创建平面与线类似，我们依旧使用上方的方法，`SCNGeometryElement(indices,primitiveType)`构造函数的第二个参数 element 的类型，有以下几种值可选：

```text
.point
.line
.triangles
.triangleStrip
.polygon
```

`.triangles`和`.polygon`都能够用于我们创建一个平面，除此之外，还有`SCNPlane(width, height)`可以帮助我们创建一个平面。

如果想要创建一个三角形，代码很简单：

```swift
let indices: [Int32] = [0, 1, 2]
let source = SCNGeometrySource(vertices: [vector1, vector2, vector3])
let element = SCNGeometryElement(indices: indices, primitiveType: .triangles)
let node = SCNNode(geometry: SCNGeometry(sources: [source], elements: [element]))
```

但是当我们想创建一个四边形的时候，变得有一些复杂。

首先考虑最简单的矩形，创建一个矩形我们可以使用`SCNPlane`，但是对于 plane 的定位和旋转会相对复杂些：

```swift
let plane = SCNPlane(width: distance, height: CGFloat(self.height))
let node = SCNNode(geometry: box)
node.renderingOrder = -10
let normalizedTo = vector2.normalized()
let normalizedFrom = vector1.normalized()
let angleBetweenTwoVectors = normalizedTo.cross(normalizedFrom)
var vector1 = vector1
var vector2 = vector2
// 调整定位
node.position = SCNVector3((vector2.x + vector1.x) * 0.5, vector1.y + height * 0.5,(vector2.z + vector1.z) * 0.5)
// 欧拉角旋转
node.eulerAngles = SCNVector3(0, -atan2(vector2.x - node.position.x, vector1.z - node.position.z) - Float.pi * 0.5, 0)
```

然后，我们考虑画一个平行四边形，可以使用`.triangles`和`.polygon`来进行绘制。使用`.triangles`即根据一条对角线画出来两个三角形，拼在一起就是一个平行四边形。

使用`.polygon`进行绘制时：

```swift
let indices: [Int32] = [
  4, // 4 point
  0, 1, 2, 3]
let source = SCNGeometrySource(vertices: [vector1, vector2, vector3, vector4])
let indexData = Data(bytes: indices, count: indices.count * MemoryLayout<Int>.size)
let element = SCNGeometryElement(data: indexData, primitiveType: .polygon, primitiveCount: 1, bytesPerIndex: MemoryLayout<Int32>.size)
let rectangle = SCNGeometry(sources: [source], elements: [element])
let node = SCNNode(geometry: rectangle)
```

使用 polygon 是可以绘制出来一个平行四边形的，但是它有一个致命的 bug，至今我也没找到答案：在使用 material 对平行四边形进行填充时，如果 material 是透明的，无论是设置了透明度还是用一张图片，会导致内存问题：SceneKit Renderer “EXC\_BAD\_ACCESS \(code=1, address=0xxxxx\)”，使用纯色的 material 就没问题。所以，如果想绘制一个平行四边形，使用`.triangles`应该是最好的方案了。

