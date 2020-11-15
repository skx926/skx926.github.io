---
title: OpenGL学习笔记（三）：向量、矩阵和变换
date: 2019-03-07 11:38:21
tags: [OpenGL, 向量, 矩阵, 变换]
categories: [Graphic]
math: true
---

[OpenGL学习笔记（一）：在Mac上编译GLFW并配置到Xcode项目](http://ky1e.me/2018/09/05/28.mac-glfw/)

## 向量

向量最基本的定义就是一个方向。或者更正式的说，向量有一个方向(Direction)和大小(Magnitude，也叫做强度或长度)。如果一个向量有2个维度，它表示一个平面的方向，当它有3个维度的时候它可以表达一个3D世界的方向。

下面你会看到3个向量，每个向量在2D图像中都用一个箭头$$(x, y)$$表示。我们在2D图片中展示这些向量，因为这样子会更直观一点。你可以把这些2D向量当做z坐标为0的3D向量。由于向量表示的是方向，起始于何处并不会改变它的值。下图我们可以看到向量$$\bar{v}$$和$$\bar{w}$$是相等的，尽管他们的起始点不同：

![]({{site.url}}/assets/img{{page.id}}/vectors.png)

用公式表示就是这样：$$\bar{v} = \begin{pmatrix} \color{red}x \\ \color{green}y \\ \color{blue}z \end{pmatrix} $$

由于向量是一个方向，所以有些时候会很难形象地将它们用位置(Position)表示出来。为了让其更为直观，我们通常设定这个方向的原点为$$(0, 0, 0)$$，然后指向一个方向，对应一个点，使其变为位置向量(Position Vector)（你也可以把起点设置为其他的点，然后说：这个向量从这个点起始指向另一个点）。比如说位置向量$$(3, 5)$$在图像中的起点会是$$(0, 0)$$，并会指向$$(3, 5)$$。我们可以使用向量在2D或3D空间中表示方向与位置.

### 向量与标量的运算

标量(Scalar)只是一个数字（或者说是仅有一个分量的向量）。当把一个向量加/减/乘/除一个标量，我们可以简单的把向量的每个分量分别进行该运算。对于加法来说会像这样:

$$\begin{pmatrix} \color{red}1 \\ \color{green}2 \\ \color{blue}3 \end{pmatrix} + x = \begin{pmatrix} \color{red}1 + x \\ \color{green}2 + x \\ \color{blue}3 + x \end{pmatrix}$$

### 向量加减

向量的加法可以被定义为是分量的(Component-wise)相加，即将一个向量中的每一个分量加上另一个向量的对应分量:

$$\bar{v} = \begin{pmatrix} \color{red}1 \\ \color{green}2 \\ \color{blue}3 \end{pmatrix}, \bar{k} = \begin{pmatrix} \color{red}4 \\ \color{green}5 \\ \color{blue}6 \end{pmatrix} \rightarrow \bar{v} + \bar{k} = \begin{pmatrix} \color{red}1 + \color{red}4 \\ \color{green}2 + \color{green}5 \\ \color{blue}3 + \color{blue}6 \end{pmatrix} = \begin{pmatrix} \color{red}5 \\ \color{green}7 \\ \color{blue}9 \end{pmatrix}$$

![]({{site.url}}/assets/img{{page.id}}/vectors_addition.png)

### 向量长度

我们使用勾股定理(Pythagoras Theorem)来获取向量的长度(Length)/大小(Magnitude)。如果你把向量的x与y分量画出来，该向量会和x与y分量为边形成一个三角形:

![]({{site.url}}/assets/img{{page.id}}/vectors_triangle.png)

因为两条边（x和y）是已知的，如果希望知道斜边$$\color{red}{\bar{v}}$$的长度，我们可以直接通过勾股定理来计算：

$$||\color{red}{\bar{v}}|| = \sqrt{\color{green}x^2 + \color{blue}y^2}$$

有一个特殊类型的向量叫做单位向量(Unit Vector)。单位向量有一个特别的性质——它的长度是1。我们可以用任意向量的每个分量除以向量的长度得到它的单位向量$$\hat{n}$$

$$\hat{n} = \frac{\bar{v}}{||\bar{v}||}$$

我们把这种方法叫做一个向量的标准化(Normalizing)。单位向量头上有一个^样子的记号。通常单位向量会变得很有用，特别是在我们只关心方向不关心长度的时候。

### 向量相乘

两个向量相乘是一种很奇怪的情况。普通的乘法在向量上是没有定义的，因为它在视觉上是没有意义的。但是在相乘的时候我们有两种特定情况可以选择：一个是点乘(Dot Product)，记作$$\bar{v} \cdot \bar{k}$$，另一个是叉乘(Cross Product)，记作$$\bar{v} \times \bar{k}$$。

#### 点乘

两个向量的点乘等于它们的数乘结果乘以两个向量之间夹角的余弦值。可能听起来有点费解，我们来看一下公式：

$$\bar{v} \cdot \bar{k} = ||\bar{v}|| \cdot ||\bar{k}|| \cdot \cos \theta$$

它们之间的夹角记作$$\theta$$。为什么这很有用？想象如果$$\bar{v}$$和$$\bar{k}$$都是单位向量，它们的长度会等于1。这样公式会有效简化成：

$$\bar{v} \cdot \bar{k} =1⋅1⋅cosθ=cosθ$$

现在点积只定义了两个向量的夹角。你也许记得90度的余弦值是0，0度的余弦值是1。使用点乘可以很容易测试两个向量是否正交(Orthogonal)或平行（正交意味着两个向量互为直角）。

也可以通过点乘的结果计算两个非单位向量的夹角，点乘的结果除以两个向量的长度之积，得到的结果就是夹角的余弦值，即$$cos\theta$$。

$$\cos \theta = \frac{\bar{v} \cdot \bar{k}}{||\bar{v}|| \cdot ||\bar{k}||}$$

点乘是通过将对应分量逐个相乘，然后再把所得积相加来计算的。

$$\begin{pmatrix} \color{red}{0.6} \\ -\color{green}{0.8} \\ \color{blue}0 \end{pmatrix} \cdot \begin{pmatrix} \color{red}0 \\ \color{green}1 \\ \color{blue}0 \end{pmatrix} = (\color{red}{0.6} * \color{red}0) + (-\color{green}{0.8} * \color{green}1) + (\color{blue}0 * \color{blue}0) = -0.8$$

#### 叉乘

叉乘只在3D空间中有定义，它需要两个不平行向量作为输入，生成一个正交于两个输入向量的第三个向量。如果输入的两个向量也是正交的，那么叉乘之后将会产生3个互相正交的向量。

![]({{site.url}}/assets/img{{page.id}}/vectors_crossproduct.png)

两个正交向量A和B叉积：

$$\begin{pmatrix} \color{red}{A_{x}} \\ \color{green}{A_{y}} \\ \color{blue}{A_{z}} \end{pmatrix} \times \begin{pmatrix} \color{red}{B_{x}} \\ \color{green}{B_{y}} \\ \color{blue}{B_{z}}  \end{pmatrix} = \begin{pmatrix} \color{green}{A_{y}} \cdot \color{blue}{B_{z}} - \color{blue}{A_{z}} \cdot \color{green}{B_{y}} \\ \color{blue}{A_{z}} \cdot \color{red}{B_{x}} - \color{red}{A_{x}} \cdot \color{blue}{B_{z}} \\ \color{red}{A_{x}} \cdot \color{green}{B_{y}} - \color{green}{A_{y}} \cdot \color{red}{B_{x}} \end{pmatrix}$$



## 矩阵

简单来说矩阵就是一个矩形的数字、符号或表达式数组。矩阵中每一项叫做矩阵的元素(Element)。下面是一个2×3矩阵的例子：

$$\begin{bmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \end{bmatrix}$$

矩阵可以通过$$(i, j)$$进行索引，i是行，j是列，这就是上面的矩阵叫做2×3矩阵的原因。

### 矩阵的加减

矩阵与标量之间的加减定义如下：

$$\begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix} + \color{green}3 = \begin{bmatrix} 1 + \color{green}3 & 2 + \color{green}3 \\ 3 + \color{green}3 & 4 + \color{green}3 \end{bmatrix} = \begin{bmatrix} 4 & 5 \\ 6 & 7 \end{bmatrix}$$

矩阵与矩阵之间的加减就是两个矩阵对应元素的加减运算，所以总体的规则和与标量运算是差不多的，只不过在相同索引下的元素才能进行运算。

$$\begin{bmatrix} \color{red}4 & \color{red}2 \\ \color{green}1 & \color{green}6 \end{bmatrix} - \begin{bmatrix} \color{red}2 & \color{red}4 \\ \color{green}0 & \color{green}1 \end{bmatrix} = \begin{bmatrix} \color{red}4 - \color{red}2 & \color{red}2  - \color{red}4 \\ \color{green}1 - \color{green}0 & \color{green}6 - \color{green}1 \end{bmatrix} = \begin{bmatrix} \color{red}2 & -\color{red}2 \\ \color{green}1 & \color{green}5 \end{bmatrix}$$

### 矩阵的数乘

和矩阵与标量的加减一样，矩阵与标量之间的乘法也是矩阵的每一个元素分别乘以该标量。

$$\color{green}2 \cdot \begin{bmatrix} 1 & 2 \\ 3 & 4 \end{bmatrix} = \begin{bmatrix} \color{green}2 \cdot 1 & \color{green}2 \cdot 2 \\ \color{green}2 \cdot 3 & \color{green}2 \cdot 4 \end{bmatrix} = \begin{bmatrix} 2 & 4 \\ 6 & 8 \end{bmatrix}$$

### 矩阵相乘

矩阵相乘有一些限制：

1. 只有当左侧矩阵的列数与右侧矩阵的行数相等，两个矩阵才能相乘。

2. 矩阵相乘不遵守交换律(Commutative)，也就是说$$A \cdot B \neq B \cdot A$$。

$$\begin{bmatrix} \color{red}1 & \color{red}2 \\ \color{green}3 & \color{green}4 \end{bmatrix} \cdot \begin{bmatrix} \color{blue}5 & \color{purple}6 \\ \color{blue}7 & \color{purple}8 \end{bmatrix} = \begin{bmatrix} \color{red}1 \cdot \color{blue}5 + \color{red}2 \cdot \color{blue}7 & \color{red}1 \cdot \color{purple}6 + \color{red}2 \cdot \color{purple}8 \\ \color{green}3 \cdot \color{blue}5 + \color{green}4 \cdot \color{blue}7 & \color{green}3 \cdot \color{purple}6 + \color{green}4 \cdot \color{purple}8 \end{bmatrix} = \begin{bmatrix} 19 & 22 \\ 43 & 50 \end{bmatrix}$$

矩阵的乘法是一系列乘法和加法组合的结果，它使用到了左侧矩阵的行和右侧矩阵的列。

![]({{site.url}}/assets/img{{page.id}}/matrix_multiplication.png)

结果矩阵的维度是(n, m)，n等于左侧矩阵的行数，m等于右侧矩阵的列数。

$$\begin{bmatrix} \color{red}4 & \color{red}2 & \color{red}0 \\ \color{green}0 & \color{green}8 & \color{green}1 \\ \color{blue}0 & \color{blue}1 & \color{blue}0 \end{bmatrix} \cdot \begin{bmatrix} \color{red}4 & \color{green}2 & \color{blue}1 \\ \color{red}2 & \color{green}0 & \color{blue}4 \\ \color{red}9 & \color{green}4 & \color{blue}2 \end{bmatrix} = \begin{bmatrix} \color{red}4 \cdot \color{red}4 + \color{red}2 \cdot \color{red}2 + \color{red}0 \cdot \color{red}9 & \color{red}4 \cdot \color{green}2 + \color{red}2 \cdot \color{green}0 + \color{red}0 \cdot \color{green}4 & \color{red}4 \cdot \color{blue}1 + \color{red}2 \cdot \color{blue}4 + \color{red}0 \cdot \color{blue}2 \\ \color{green}0 \cdot \color{red}4 + \color{green}8 \cdot \color{red}2 + \color{green}1 \cdot \color{red}9 & \color{green}0 \cdot \color{green}2 + \color{green}8 \cdot \color{green}0 + \color{green}1 \cdot \color{green}4 & \color{green}0 \cdot \color{blue}1 + \color{green}8 \cdot \color{blue}4 + \color{green}1 \cdot \color{blue}2 \\ \color{blue}0 \cdot \color{red}4 + \color{blue}1 \cdot \color{red}2 + \color{blue}0 \cdot \color{red}9 & \color{blue}0 \cdot \color{green}2 + \color{blue}1 \cdot \color{green}0 + \color{blue}0 \cdot \color{green}4 & \color{blue}0 \cdot \color{blue}1 + \color{blue}1 \cdot \color{blue}4 + \color{blue}0 \cdot \color{blue}2 \end{bmatrix}   \\ = \begin{bmatrix} 20 & 8 & 12 \\ 25 & 4 & 34 \\ 2 & 0 & 4 \end{bmatrix}$$

## 向量与矩阵相乘

前面我们把向量表示成这种形式$$\begin{pmatrix} \color{red}x \\ \color{green}y \\ \color{blue}z \end{pmatrix}$$，这种情况下它其实就是一个**N×1**矩阵，N表示向量分量的个数。我们也可以把它表示成一个**1×N**的矩阵，也就是这样$$\begin{pmatrix} \color{red}x & \color{green}y & \color{blue}z  \end{pmatrix}$$

如果我们有一个**N×N**矩阵，我们可以用我们的**1×N**向量乘以这个矩阵，因为这个矩阵的行数等于向量的列数，所以最终得到的结果也是一个**1×N**的矩阵，这就相当于是给向量做了一个变换。

### 单位矩阵

单位矩阵是一个除了对角线以外都是0的**N×N**矩阵。在下式中可以看到，这种变换矩使一个向量完全不变：

$$\begin{pmatrix} a & b & c & d \end{pmatrix} \cdot \begin{bmatrix} \color{red}1 & \color{green}0 & \color{blue}0 & \color{purple}0 \\ \color{red}0 & \color{green}1 & \color{blue}0 & \color{purple}0 \\ \color{red}0 & \color{green}0 & \color{blue}1 & \color{purple}0 \\ \color{red}0 & \color{green}0 & \color{blue}0 & \color{purple}1 \end{bmatrix}  = \begin{bmatrix} \color{red}1 \cdot a & \color{green}1 \cdot b & \color{blue}1 \cdot c & \color{purple}1 \cdot d \end{bmatrix} = \begin{pmatrix} a & b & c & d \end{pmatrix}$$

### 缩放

对一个向量进行缩放(Scaling)就是对向量的长度进行缩放，而保持它的方向不变。由于我们进行的是2维或3维操作，我们可以分别定义一个有2或3个缩放变量的向量，每个变量缩放一个轴(x、y或z)。

我们先来尝试缩放向量$$\color{red}{\bar{v}} = (3,2)$$。我们可以把向量沿着x轴缩放0.5，使它的宽度缩小为原来的二分之一；我们将沿着y轴把向量的高度缩放为原来的两倍。我们看看把向量缩放(0.5, 2)倍所获得的$$\color{blue}{\bar{s}}$$是什么样的：

![]({{site.url}}/assets/img{{page.id}}/vectors_scale.png)

OpenGL通常是在3D空间进行操作的，对于2D的情况我们可以把z轴缩放1倍，这样z轴的值就不变了。我们刚刚的缩放操作是不均匀(Non-uniform)缩放，因为每个轴的缩放因子(Scaling Factor)都不一样。如果每个轴的缩放因子都一样那么就叫均匀缩放(Uniform Scale)。

我们下面会构造一个变换矩阵来为我们提供缩放功能。我们从单位矩阵了解到，每个对角线元素会分别与向量的对应元素相乘。如果我们把1变为3会怎样？这样子的话，我们就把向量的每个元素乘以3了，这事实上就把向量缩放3倍。如果我们把缩放变量表示为$$(\color{red}{S_1}, \color{green}{S_2}, \color{blue}{S_3})$$可以为任意向量$$(x,y,z)$$定义一个缩放矩阵：

$$\begin{pmatrix} x & y & z  \end{pmatrix} \cdot \begin{bmatrix} \color{red}{S_1} & \color{green}0 & \color{blue}0  \\ \color{red}0 & \color{green}{S_2} & \color{blue}0  \\ \color{red}0 & \color{green}0 & \color{blue}{S_3}   \end{bmatrix}  = \begin{pmatrix} \color{red}{S_1} \cdot x & \color{green}{S_2} \cdot y & \color{blue}{S_3} \cdot z  \end{pmatrix}$$

### 位移

位移(Translation)是在原始向量的基础上加上另一个向量从而获得一个在不同位置的新向量的过程，从而在位移向量基础上移动了原始向量。

和缩放矩阵一样，在4×4矩阵上有几个特别的位置用来执行特定的操作，对于位移来说它们是第四列最上面的3个值。如果我们把位移向量表示为$$(\color{red}{T_x},\color{green}{T_y},\color{blue}{T_z})$$，我们就能把位移矩阵定义为：

$$\begin{pmatrix} x & y & z & w \end{pmatrix} \cdot \begin{bmatrix}  \color{red}1 & \color{green}0 & \color{blue}0 & \color{purple}{0} \\ \color{red}0 & \color{green}1 & \color{blue}0 & \color{purple}{0} \\ \color{red}0 & \color{green}0 & \color{blue}1 & \color{purple}{0} \\ \color{red}{T_x} & \color{green}{T_y} & \color{blue}{T_z} & \color{purple}1 \end{bmatrix}  = \begin{pmatrix} x + \color{red}{T_x} & y + \color{green}{T_y} & z + \color{blue}{T_z} & w \end{pmatrix}$$

这样是能工作的，因为所有的位移值都要乘以向量的**w**行，所以位移值会加到向量的原始值上（想想矩阵乘法法则）。而如果你用3x3矩阵我们的位移值就没地方放也没地方乘了，所以是不行的。

> **齐次坐标(Homogeneous Coordinates)**
>
> 向量的w分量也叫齐次坐标。想要从齐次向量得到3D向量，我们可以把x、y和z坐标分别除以w坐标。我们通常不会注意这个问题，因为w分量通常是1.0。使用齐次坐标有几点好处：它允许我们在3D向量上进行位移（如果没有w分量我们是不能位移向量的）。
>

### 旋转

首先我们来定义一个向量的旋转到底是什么。下图中展示的2D向量$$\bar{v}$$是由$$\bar{k}$$向右旋转72度所得的：

![]({{site.url}}/assets/img{{page.id}}/vectors_angle.png)

在3D空间中旋转需要定义一个角和一个旋转轴(Rotation Axis)。物体会沿着给定的旋转轴旋转特定角度。如果你想要更形象化的感受，可以试试向下看着一个特定的旋转轴，同时将你的头部旋转一定角度。当2D向量在3D空间中旋转时，我们把旋转轴设为z轴（尝试想象这种情况）。

沿x轴旋转：

$$\begin{pmatrix} x & y & z & 1 \end{pmatrix} \cdot  \begin{bmatrix} \color{red}1 & \color{green}0 & \color{blue}0 & \color{purple}0 \\ \color{red}0 & \color{green}{\cos \theta} & \color{blue}{\sin \theta} & \color{purple}0 \\ \color{red}0 & -\color{green}{\sin \theta} & \color{blue}{\cos \theta} & \color{purple}0 \\ \color{red}0 & \color{green}0 & \color{blue}0 & \color{purple}1 \end{bmatrix} = \begin{pmatrix} x & \color{green}{\cos \theta} \cdot y - \color{green}{\sin \theta} \cdot z & \color{blue}{\sin \theta} \cdot y + \color{blue}{\cos \theta} \cdot z & 1 \end{pmatrix}$$

沿y轴旋转：

$$\begin{pmatrix} x & y &z & 1 \end{pmatrix} \cdot \begin{bmatrix} \color{red}{\cos \theta} & \color{green}0 & -\color{blue}{\sin \theta} & \color{purple}0 \\ \color{red}0 & \color{green}1 & \color{blue}0 & \color{purple}0 \\  \color{red}{\sin \theta} & \color{green}0 & \color{blue}{\cos \theta} & \color{purple}0 \\ \color{red}0 & \color{green}0 & \color{blue}0 & \color{purple}1 \end{bmatrix} = \begin{pmatrix} \color{red}{\cos \theta} \cdot x + \color{red}{\sin \theta} \cdot z & y &- \color{blue}{\sin \theta} \cdot x + \color{blue}{\cos \theta} \cdot z & 1 \end{pmatrix}$$

沿z轴旋转：

$$\begin{pmatrix} x & y & z & 1 \end{pmatrix} \cdot \begin{bmatrix} \color{red}{\cos \theta} &  \color{green}{\sin \theta} & \color{blue}0 & \color{purple}0 \\ - \color{red}{\sin \theta} & \color{green}{\cos \theta} & \color{blue}0 & \color{purple}0 \\ \color{red}0 & \color{green}0 & \color{blue}1 & \color{purple}0 \\ \color{red}0 & \color{green}0 & \color{blue}0 & \color{purple}1 \end{bmatrix} = \begin{pmatrix} \color{red}{\cos \theta} \cdot x - \color{red}{\sin \theta} \cdot y  & \color{green}{\sin \theta} \cdot x + \color{green}{\cos \theta} \cdot y & z & 1 \end{pmatrix}$$

对于3D空间中的旋转，一个更好的模型是沿着任意的一个轴，比如单位向量$$(0.662, 0.2, 0.7222)$$旋转，而不是对一系列旋转矩阵进行复合。这样的一个（超级麻烦的）矩阵是存在的，见下面这个公式，其中$$(\color{red}{R_x}, \color{green}{R_y}, \color{blue}{R_z})$$代表任意旋转轴：

$$\begin{bmatrix} \cos \theta + \color{red}{R_x}^2(1 - \cos \theta) & \color{green}{R_y}\color{red}{R_x} (1 - \cos \theta) + \color{blue}{R_z} \sin \theta  & \color{blue}{R_z}\color{red}{R_x}(1 - \cos \theta) - \color{green}{R_y} \sin \theta & 0 \\ \color{red}{R_x}\color{green}{R_y}(1 - \cos \theta) - \color{blue}{R_z} \sin \theta & \cos \theta + \color{green}{R_y}^2(1 - \cos \theta) & \color{blue}{R_z}\color{green}{R_y}(1 - \cos \theta) + \color{red}{R_x} \sin \theta  & 0 \\ \color{red}{R_x}\color{blue}{R_z}(1 - \cos \theta) + \color{green}{R_y} \sin \theta & \color{green}{R_y}\color{blue}{R_z}(1 - \cos \theta) - \color{red}{R_x} \sin \theta& \cos \theta + \color{blue}{R_z}^2(1 - \cos \theta) & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix}$$

### 矩阵的组合

使用矩阵进行变换的真正力量在于，根据矩阵之间的乘法，我们可以把多个变换组合到一个矩阵中。让我们看看我们是否能生成一个变换矩阵，让它组合多个变换。假设我们有一个顶点(x, y, z)，我们希望将其缩放2倍，然后位移$$(1, 2, 3)$$个单位。我们需要一个位移和缩放矩阵来完成这些变换。结果的变换矩阵看起来像这样：

$$Trans = \begin{bmatrix} \color{red}2 & \color{green}0 & \color{blue}0 & \color{purple}0 \\ \color{red}0 & \color{green}2 & \color{blue}0 & \color{purple}0 \\ \color{red}0 & \color{green}0 & \color{blue}2 & \color{purple}0 \\ \color{red}0 & \color{green}0 & \color{blue}0 & \color{purple}1 \end{bmatrix} \cdot \begin{bmatrix} \color{red}1 & \color{green}0 & \color{blue}0 & \color{purple}0 \\ \color{red}0 & \color{green}1 & \color{blue}0 & \color{purple}0 \\ \color{red}0 & \color{green}0 & \color{blue}1 & \color{purple}0 \\ \color{red}1 & \color{green}2 & \color{blue}3 & \color{purple}1 \end{bmatrix} = \begin{bmatrix} \color{red}2 & \color{green}0 & \color{blue}0 & \color{purple}0 \\ \color{red}0 & \color{green}2 & \color{blue}0 & \color{purple}0 \\ \color{red}0 & \color{green}0 & \color{blue}2 & \color{purple}0 \\ \color{red}1 & \color{green}2 & \color{blue}3 & \color{purple}1 \end{bmatrix}$$

$$Trans' = \begin{bmatrix} \color{red}1 & \color{green}0 & \color{blue}0 & \color{purple}0 \\ \color{red}0 & \color{green}1 & \color{blue}0 & \color{purple}0 \\ \color{red}0 & \color{green}0 & \color{blue}1 & \color{purple}0 \\ \color{red}1 & \color{green}2 & \color{blue}3 & \color{purple}1 \end{bmatrix} \cdot \begin{bmatrix} \color{red}2 & \color{green}0 & \color{blue}0 & \color{purple}0 \\ \color{red}0 & \color{green}2 & \color{blue}0 & \color{purple}0 \\ \color{red}0 & \color{green}0 & \color{blue}2 & \color{purple}0 \\ \color{red}0 & \color{green}0 & \color{blue}0 & \color{purple}1 \end{bmatrix} = \begin{bmatrix} \color{red}2 & \color{green}0 & \color{blue}0 & \color{purple}0 \\ \color{red}0 & \color{green}2 & \color{blue}0 & \color{purple}0 \\ \color{red}0 & \color{green}0 & \color{blue}2 & \color{purple}0 \\ \color{red}2 & \color{green}4 & \color{blue}6 & \color{purple}1 \end{bmatrix}$$

注意，当矩阵相乘时我们先写位移再写缩放变换的。矩阵乘法是不遵守交换律的，这意味着它们的顺序很重要。建议您在组合矩阵时，先进行缩放操作，然后是旋转，最后才是位移，否则它们会互相影响。比如，如果你先位移再缩放，位移的向量也会同样被缩放。

用最终的变换矩阵乘以我们的向量会得到以下结果：

$$\begin{pmatrix} x & y & z & 1 \end{pmatrix} \cdot \begin{bmatrix} \color{red}2 & \color{green}0 & \color{blue}0 & \color{purple}0 \\ \color{red}0 & \color{green}2 & \color{blue}0 & \color{purple}0 \\ \color{red}0 & \color{green}0 & \color{blue}2 & \color{purple}0 \\ \color{red}1 & \color{green}2 & \color{blue}3 & \color{purple}1 \end{bmatrix}  = \begin{pmatrix} \color{red}2x + \color{red}1 & \color{green}2y + \color{green}2  & \color{blue}2z + \color{blue}3 & 1 \end{pmatrix}$$

## CATransform3D

我们可以在`<QuartzCore/CATransform3D.h>`中看到CATransform3D的定义：

```c
struct CATransform3D
{
  CGFloat m11, m12, m13, m14;
  CGFloat m21, m22, m23, m24;
  CGFloat m31, m32, m33, m34;
  CGFloat m41, m42, m43, m44;
};
```

这个结构体对应的是这样一个4x4的变换矩阵：

$$\begin{bmatrix} m_{11} & m_{12} & m_{13}& m_{14} \\ m_{21}& m_{22} & m_{23} & m_{24} \\ m_{31} & m_{32} & m_{33} & m_{34} \\ m_{41} & m_{42} & m_{43} & m_{44} \end{bmatrix}$$

坐标向量和变换矩阵的乘法为：

$$\begin{pmatrix} x & y & z & 1 \end{pmatrix} \cdot \begin{bmatrix}  \color{red}{m_{11}} &\color{green}{m_{12}} & \color{blue}{m_{13}} & \color{purple}{m_{14}} \\  \color{red}{m_{21}}& \color{green}{m_{22}} & \color{blue}{m_{23}} & \color{purple}{m_{24}} \\  \color{red}{m_{31}} & \color{green}{m_{32}} & \color{blue}{m_{33}} & \color{purple}{m_{34}} \\  \color{red}{m_{41}} &\color{green}{m_{42}} & \color{blue}{m_{43}} & \color{purple}{m_{44}} \end{bmatrix} =$$

$$\begin{pmatrix} \color{red}{m_{11}}x + \color{red}{m_{21}}y +\color{red}{m_{31}}z + \color{red}{m_{41}} & \color{green}{m_{12}}x + \color{green}{m_{22}}y + \color{green}{m_{32}}z + \color{green}{m_{42}} & \color{blue}{m_{13}}x +  \color{blue}{m_{23}}y + \color{blue}{m_{33}}z + \color{blue}{m_{43}} & \color{purple}{m_{14}}x + \color{purple}{m_{24}}y + \color{purple}{m_{34}}z + \color{purple}{m_{44}} \end{pmatrix}$$

### CGAffineTransform

CGAffineTransform是用在2D中的变换矩阵，我们可以在`<CoreGraphic/CGAffineTransform.h>`中看到CGAffineTransform的定义：

``` c
struct CGAffineTransform {
  CGFloat a, b, c, d;
  CGFloat tx, ty;
};
```

这个结构体对应的是这样一个3x3的变换矩阵：

$$\begin{bmatrix} a & b & 0  \\ c & d & 0 \\ tx & ty & 1  \end{bmatrix}$$

坐标向量和变换矩阵的乘法为：

$$\begin{pmatrix} x & y & 1 \end{pmatrix} \cdot \begin{bmatrix}  a & b & 0 \\ c & d & 0 \\ tx & ty & 1 \end{bmatrix} = \begin{bmatrix}  x\cdot a + y \cdot c + tx &  x \cdot b + y \cdot d + ty & 1 \end{bmatrix}$$

缩放: 

$$\begin{pmatrix} x & y & 1  \end{pmatrix} \cdot \begin{bmatrix}  sx & 0 & 0 \\ 0 & sy & 0 \\ 0 & 0  &1 \end{bmatrix} = \begin{bmatrix}  sx \cdot x  &  sy \cdot y & 1 \end{bmatrix}$$

##### 

位移:

$$\begin{pmatrix} x & y & 1 \end{pmatrix} \cdot \begin{bmatrix}  1 & 0 & 0 \\ 0 & 1 & 0 \\ tx & ty & 1 \end{bmatrix} = \begin{bmatrix}  x + tx  &  y + ty & 1 \end{bmatrix}$$

旋转:

$$\begin{pmatrix} x & y & 1 \end{pmatrix} \cdot \begin{bmatrix}  cos\theta & sin\theta & 0 \\ -sin\theta & cos\theta & 0 \\ 0 & 0 & 1 \end{bmatrix} = \begin{bmatrix}  cos\theta \cdot x - sin\theta \cdot y  &  sin\theta \cdot x + cos\theta\cdot y& 1 \end{bmatrix}$$



### 透视投影

实际中你如果直接使用旋转，会注意到旋转前后，结果看起来竟然和普通的缩放一模一样，这是为什么呢？原因其实很简单，假如绕y轴旋转，空间中的图层虽然旋转了，但是显示到XoY平面（也就是iPhone的屏幕上）的时候，会把3D的物体进行正投影，这样子看上去就像是左右压缩一样

而学过绘画的都知道人的视野并不是平行的，而是有一个透视图的概念，眼睛前有实际平行的两条线段发出（相当于z轴方向的向量），人眼看起来会相交于一点上（焦点，Focal point），这才产生了3D感。

![]({{site.url}}/assets/img{{page.id}}/projection.png)

如何用变换矩阵实现透视投影呢？只需要修改一个值$$m_{34}$$。为什么单单修改一个$$m_{34}$$的值，就能达到这种透视3D的效果呢？

Core Animation已经定义了焦点的x,y坐标，就是这个图层的anchorPoint（锚点），同时取z=0的XoY平面作为图像平面（也就是iPhone的屏幕平面），那么假如我希望焦点到图像平面的距离是d，可以假设焦点坐标为(0,0,d)，现在对m34的值进行赋值为w，初始向量坐标为$$(x,y,z)$$，开始推导：

矩阵乘法：

$$\begin{pmatrix} x & y & z & 1 \end{pmatrix} \cdot \begin{bmatrix} 1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 1 & w \\ 0 & 0 & 0 & 1 \end{bmatrix}  = \begin{pmatrix} x & y & z & zw+1 \end{pmatrix}$$

此时得到的向量不为齐次，需要进行齐次化，得到真正的坐标：

$$\begin{pmatrix} x' & y' & z' & 1 \end{pmatrix} = \begin{pmatrix} \frac{x}{zw+1} & \frac{y}{zw+1} & \frac{z}{zw+1} & 1 \end{pmatrix}$$

最后对XoY平面进行投影，则最终看到的二维向量应该为:

$$\begin{pmatrix} \frac{x}{zw+1} & \frac{y}{zw+1} \end{pmatrix}$$

现在考虑x轴的情况（y轴同理），我们知道真实三维空间的x坐标是x，现在得到透视投影下的x坐标是$$\frac{x}{zw+1}$$

为了得到d和w的关系，这里引用一幅图，绿色的点为原始点，红色的点为投影到XoY平面上的点，我们这里推导不需要管具体的值，只是为了更清晰地发现规律：

![]({{site.url}}/assets/img{{page.id}}/projection_axis.jpg)

根据相似三角形原理我们可以得到

$$|\frac{x}{zw+1} : x| = d : (|z| + d)$$

去绝对值号，且x!=0,z!=0，由图可得此处的z为负数，所以：

$$\frac{1}{zw+1}  = \frac{d} {d - z}$$

$$zw+1 = 1 - \frac{z}d$$

$$w = - \frac{1}d$$

因此$$m_{34}$$的值就为$$- \frac{1}d$$，这里的$$d$$就是焦点距离，也就是人眼到手机屏幕的距离，一般取值在500~1000之间。默认初始变换矩阵的$$m_{34}$$是0，也就是说认为焦点无限远，因此看起来没有任何3D感。假如我们取d越大，则看起来越没有投射和3D感；取d越小，则3D感和失真感越强烈。



参考文章：

[LearnOpenGL CN:变换](https://learnopengl-cn.github.io/01%20Getting%20started/07%20Transformations/)

[Core Animation 3D 仿射变换知识](https://zhuanlan.zhihu.com/p/23359747)