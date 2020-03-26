# Android Canvas 变换

## 1 范围裁切

### 1.1 clipRect

按方形裁切

```java
canvas.save();  
canvas.clipRect(left, top, right, bottom);  
canvas.drawBitmap(bitmap, x, y, paint);  
canvas.restore(); 
```

> 要加上`Canvas.save()`和`Canvas.restore()`来及时恢复绘制范围

![clip](../../../attachments/Android/绘制/007-clip.png)

### 1.2 clipPath

按 Path 裁切，可以裁切更多形状

```java
canvas.save();  
canvas.clipPath(path1);  
canvas.drawBitmap(bitmap, point1.x, point1.y, paint);  
canvas.restore();

canvas.save();  
canvas.clipPath(path2);  
canvas.drawBitmap(bitmap, point2.x, point2.y, paint);  
canvas.restore(); 
```

![clip-path](../../../attachments/Android/绘制/007-clip-path.png)

## 2 几何变换

几何变换的使用大概分为三类：

1. 使用 **Canvas** 来做常见的二维变换；
2. 使用 **Matrix** 来做常见和不常见的二维变换；
3. 使用 **Camera** 来做三维变换。

### 2.1 Canvas 变换

#### 2.1.1 平移

```java
canvas.save();  
canvas.translate(200, 0);  
canvas.drawBitmap(bitmap, x, y, paint);  
canvas.restore(); 
```

![translate](../../../attachments/Android/绘制/007-translate.png)

#### 2.1.2 旋转

```java
canvas.save();  
canvas.rotate(45, centerX, centerY);  
canvas.drawBitmap(bitmap, x, y, paint);  
canvas.restore(); 
```

![rotate](../../../attachments/Android/绘制/007-rotate.png)

#### 2.1.3 缩放

```java
canvas.save();  
canvas.translate(200, 0);  
canvas.drawBitmap(bitmap, x, y, paint);  
canvas.restore(); 
```

![scale](../../../attachments/Android/绘制/007-scale.png)

#### 2.1.4 错切

```java
canvas.save();
canvas.skew(0, 0.5f);
canvas.drawBitmap(bitmap, x, y, paint);
canvas.restore();
```

![skew](../../../attachments/Android/绘制/007-skew.png)

### 2.2 使用 Matrix 变换

#### 2.2.1 Matrix 内置变换

使用 **Matrix** 变换的步骤：

1. 创建 **Matrix** 对象
2. 调用 **Matrix** 的`pre/postTranslate/Rotate/Scale/Skew()`方法来设置几何变换
3. 使用`Canvas.setMatrix(matrix)`或`Canvas.concat(matrix)`来把几何变换应用到 **Canvas**

```java
Matrix matrix = new Matrix();

...

matrix.reset();  
matrix.postTranslate();  
matrix.postRotate();

canvas.save();  
canvas.concat(matrix);  
canvas.drawBitmap(bitmap, x, y, paint);  
canvas.restore(); 
```

#### 2.2.2 Matrix 自定义变换

##### 2.2.2.1 点对点映射的方式设置变换

`Matrix.setPolyToPoly(float[] src, int srcIndex, float[] dst, int dstIndex, int pointCount)`

* src : 源点集
* dst : 目标点集
* srcIndex : 源点 index
* dstIndex : 目标点 index
* pointCount : 采集点个数

该函数的作用是通过多点的映射的方式来直接设置变换。

> 1. 单点映射可以实现平移
> 2. 多点映射可以实现任意变换
> 3. 映射点个数不能大于 4，大于 4 个点就无法计算变换了
> 4. 通常设置图片的四个顶点

![matrix](../../../attachments/Android/绘制/007-matrix.png)

### 2.3 使用 Camera 来做三维变换

`Camera` 的三维变换有三类：**旋转**、**平移**、**移动**相机。

#### 2.3.1 Camera.rotate*() 三维旋转

`Camera.rotate*()` 一共有四个方法： `rotateX(deg)` `rotateY(deg)` `rotateZ(deg)` `rotate(x, y, z)`。

```java
canvas.save();

camera.save(); // 保存 Camera 的状态
camera.rotateX(30); // 旋转 Camera 的三维空间
camera.applyToCanvas(canvas); // 把旋转投影到 Canvas
camera.restore(); // 恢复 Camera 的状态

canvas.drawBitmap(bitmap, point1.x, point1.y, paint);
canvas.restore();
```

![camera-rotate-x](../../../attachments/Android/绘制/007-camera-rotate-x.png)

如果你需要图形左右对称，需要配合上 `Canvas.translate()`，在三维旋转之前把绘制内容的中心点移动到原点，即旋转的轴心，然后在三维旋转后再把投影移动回来：

```java
canvas.save();

camera.save(); // 保存 Camera 的状态
camera.rotateX(30); // 旋转 Camera 的三维空间
canvas.translate(centerX, centerY); // 旋转之后把投影移动回来
camera.applyToCanvas(canvas); // 把旋转投影到 Canvas
canvas.translate(-centerX, -centerY); // 旋转之前把绘制内容移动到轴心（原点）
camera.restore(); // 恢复 Camera 的状态

canvas.drawBitmap(bitmap, point1.x, point1.y, paint);
canvas.restore();
```

> `Canvas` 的几何变换顺序是反的，所以要把移动到中心的代码写在下面，把从中心移动回来的代码写在上面。

![camera-rotate-x-fix](../../../attachments/Android/绘制/007-camera-rotate-x-fix.png)

#### 2.3.2 Camera.translate(float x, float y, float z) 移动

使用方式和 `Camera.rotate*()` 相同。

#### 2.3.3 Camera.setLocation(x, y, z) 设置虚拟相机的位置

*该方法的参数的单位是英寸*。

> 这种设计源自 Android 底层的图像引擎 [Skia](https://skia.org/) 。在 Skia 中，Camera 的位置单位是英寸，英寸和像素的换算单位在 Skia 中被写死为了 72 像素。
>
> 在 `Camera` 中，相机的默认位置是 (0, 0, -8)（英寸）。8 x 72 = 576，所以它的默认位置是 (0, 0, -576)（像素）。

而使用 `setLocation()` 方法来把相机往后移动，就可以修复固定尺寸导致的不同 dpi 手机表现不一的问题。

