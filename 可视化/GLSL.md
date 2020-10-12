# GLGL ES

GLSL ES 是在 OpenGL 着色器语言（GLSL）的基础上，删除和简化了一部分功能后形成的。GLSL ES 的目标平台是电子产品和嵌入式设备，如智能手机和游戏主机等，因此使用更简化的 GLSL ES 能够允许硬件厂商对硬件进行简化，能够降低硬件的功耗，以及更重要的是减少了性能开销。

GLSL ES 的语法与 C 语言比较类似，可以用来完成一些通用的任务，如图像处理和数据运算。

GLSL 的入口是一个 main 函数，它不接受参数。

GLSL 支持数值类型和布尔类型，不支持字符串。

## 矢量和矩阵

- 矢量

vec2，vec3，vec4 具有 2，3，4 个浮点数元素的矢量
ivec2，ivec3，ivec4 具有 2，3，4 个整型数元素的矢量
bvec2，bvec3，bvec4 具有 2，3，4 个布尔值元素的矢量

- 矩阵

mat2，mat3，mat4 表示 2x2，3x3，4x4 的浮点数元素的矩阵

- 构造函数

矢量的构造函数可以传入矢量来进行构造，同样，矩阵的构造函数也可以传入数值和矢量，将按照顺序将值传入进行构造

- 访问元素

矢量可以使用`.`进行元素的访问，分量名如下：

| 类别       | 描述                 |
| ---------- | -------------------- |
| x, y, z, w | 用来获取顶点坐标分量 |
| r, g, b, a | 用来获取颜色分量     |
| s, t, p, q | 用来获取纹理分量     |

实际上，x, r, s 都会返回第一分量等等，它们属于不同的空间，不可混用。

```glsl
vec4 v4 = vec4(1.0, 2.0, 3.0, 4.0)
v4.x // 1.0
v4.r // 1.0
v4.s // 1.0
v4.xyz // vec3(1.0, 2.0, 3.0)
v4.xy = vec2(10.0, 20.0) // v4 => vec4(10.0, 20.0, 3.0, 4.0)
```

矩阵和矢量可使用`[]`来访问元素，把矢量看作一维数组，矩阵为二维数组，操作形式与 C 语言等类似。

## 取样器 sampler

通过此类型来访问纹理，只能是 uniform 变量。

```glsl
uniform sampler2D u_Sampler
```

唯一能赋值给取样器变量的就是纹理单元编号。

## 内置函数

| 类别         | 内置函数                                                                                                                                                                                         |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 角度函数     | radians（角度转弧度），degrees（弧度转角度）                                                                                                                                                     |
| 三角函数     | sin，cos，tan，asin，acos，atan                                                                                                                                                                  |
| 指数函数     | pow，exp（自然指数），log，exp2，log2，sqrt，inversesqrt（开平方的倒数）                                                                                                                         |
| 通用函数     | abs，min，max，mod，sign（取正负号），floor，ceil，clamp（限定范围），mix（线性内插），step（步进函数），smoothstep（艾米内插步进），fract（获取小数部分）                                       |
| 几何函数     | length，distance，dot（内积），cross（外积），normalize（归一化），reflect（矢量反射），faceforward（使向量朝前）                                                                                |
| 矩阵函数     | matrixCmpMult（逐元素乘法）                                                                                                                                                                      |
| 矢量函数     | lessThan（逐元素小于），lessThanEqual，greaterThan，greaterThanEqual，equal，notEqual，any，all，not                                                                                             |
| 纹理查询函数 | texture2D（在二维纹理中获取纹素），textureCube（在立方体中获取纹素），texture2DProj（texture2D 的投影版本），texture2DLod（texture2D 的金字塔版本），texture2DProjLod（texture2DLod 的投影版本） |

## 存储限定字

- const
- attribute
- uniform
- varying

## 精度限定字

- highp
- mediump
- lowp
