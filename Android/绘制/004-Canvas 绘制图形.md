# Android Canvas 图形绘制

**Canvas** 是画布对象，通常在 **View** 的`onDraw`方法中被提供，我们可以在`onDraw`中使用它完成 **View** 的绘制。

## 颜色绘制

在整个绘制区域统一涂上指定的颜色。

```java
drawColor(@ColorInt int color)
drawRGB(int r, int g, int b)
drawARGB(int a, int r, int g, int b)
```

这类颜色填充方法一般用于在绘制之前设置底色，或者在绘制之后为界面设置半透明蒙版。

## 绘制图形

### 点

绘制点依靠点的坐标确认。

> 连续绘制多个点，`offset`代表从第几个位置开始，`count`表示使用几个参数（8个参数就是4个点）。

```java
drawPoint(float x, float y, Paint paint)
drawPoints(float[] pts, int offset, int count, Paint paint)
drawPoints(float[] pts, Paint paint)
```

### 线

```java
drawLine(float startX, float startY, float stopX, float stopY, Paint paint)
drawLines(float[] pts, int offset, int count, Paint paint)
drawLines(float[] pts, Paint paint)
```

### 矩形

绘制矩形依靠4个顶点确认位置。

```java
drawRect(float left, float top, float right, float bottom, Paint paint)
drawRect(RectF rect, Paint paint)
drawRect(Rect rect, Paint paint)
```

### 圆角矩形

比绘制矩形多出两个圆角半径参数。

```java
drawRoundRect(float left, float top, float right, float bottom, float rx, float ry, Paint paint)
drawRoundRect(RectF rect, float rx, float ry, Paint paint)
```

### 圆形

绘制圆形是靠中心点位置和半径确认的。

```java
drawCircle(float centerX, float centerY, float radius, Paint paint)
```

### 椭圆

绘制椭圆实际上是在一个矩形内绘制，所以确认矩形就确认了椭圆。

```java
drawOval(float left, float top, float right, float bottom, Paint paint)
drawOval(RectF rect, Paint paint)
```

### 弧形或扇形

弧形或扇形是使用椭圆来描述的，在绘制椭圆的基础上添加起始角度和终止角度，并判断是否连接中心就可以截出一段弧形或扇形。

```java
drawArc(float left, float top, float right, float bottom, float startAngle, float sweepAngle, boolean useCenter, Paint paint)
drawArc(RectF oval, float startAngle, float sweepAngle, boolean useCenter, Paint paint)
```

> `useCenter`表示是否连接到圆心，如果不连接到圆心，就是弧形，如果连接到圆心，就是扇形。

### 自定义图形

根据`path`的路径描述绘制自定义图形。

```java
drawPath(Path path, Paint paint)
```