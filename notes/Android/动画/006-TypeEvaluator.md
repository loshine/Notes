

# TypeEvaluator

## ArgbEvaluator

`TypeEvaluator` 最经典的用法是使用 `ArgbEvaluator` 来做颜色渐变的动画。

```java
ObjectAnimator animator = ObjectAnimator.ofInt(view, "color", 0xffff0000, 0xff00ff00);
animator.setEvaluator(new ArgbEvaluator());
animator.start();
```

![argb-evaluator](../../../attachments/Android/动画/006-argb-evaluator.gif)

另外，在 Android 5.0 （API 21） 加入了新的方法 `ofArgb()`，所以如果你的 `minSdk` 大于或者等于 21（哈哈哈哈哈哈哈哈），你可以直接用下面这种方式：

```java
ObjectAnimator animator = ObjectAnimator.ofArgb(view, "color", 0xffff0000, 0xff00ff00);
animator.start();
```

## 自定义 Evaluator

```java
// 自定义 HsvEvaluator
private class HsvEvaluator implements TypeEvaluator<Integer> {
   float[] startHsv = new float[3];
   float[] endHsv = new float[3];
   float[] outHsv = new float[3];

   @Override
   public Integer evaluate(float fraction, Integer startValue, Integer endValue) {
       // 把 ARGB 转换成 HSV
       Color.colorToHSV(startValue, startHsv);
       Color.colorToHSV(endValue, endHsv);

       // 计算当前动画完成度（fraction）所对应的颜色值
       if (endHsv[0] - startHsv[0] > 180) {
           endHsv[0] -= 360;
       } else if (endHsv[0] - startHsv[0] < -180) {
           endHsv[0] += 360;
       }
       outHsv[0] = startHsv[0] + (endHsv[0] - startHsv[0]) * fraction;
       if (outHsv[0] > 360) {
           outHsv[0] -= 360;
       } else if (outHsv[0] < 0) {
           outHsv[0] += 360;
       }
       outHsv[1] = startHsv[1] + (endHsv[1] - startHsv[1]) * fraction;
       outHsv[2] = startHsv[2] + (endHsv[2] - startHsv[2]) * fraction;

       // 计算当前动画完成度（fraction）所对应的透明度
       int alpha = startValue >> 24 + (int) ((endValue >> 24 - startValue >> 24) * fraction);

       // 把 HSV 转换回 ARGB 返回
       return Color.HSVToColor(alpha, outHsv);
   }
}

ObjectAnimator animator = ObjectAnimator.ofInt(view, "color", 0xff00ff00);
// 使用自定义的 HslEvaluator
animator.setEvaluator(new HsvEvaluator());
animator.start();
```

## ofObject()

借助于 `TypeEvaluator`，属性动画就可以通过 `ofObject()` 来对不限定类型的属性做动画了。方式很简单：

1. 为目标属性写一个自定义的 `TypeEvaluator`
2. 使用 `ofObject()` 来创建 `Animator`，并把自定义的 `TypeEvaluator` 作为参数填入

```java
private class PointFEvaluator implements TypeEvaluator<PointF> {
   PointF newPoint = new PointF();

   @Override
   public PointF evaluate(float fraction, PointF startValue, PointF endValue) {
       float x = startValue.x + (fraction * (endValue.x - startValue.x));
       float y = startValue.y + (fraction * (endValue.y - startValue.y));

       newPoint.set(x, y);

       return newPoint;
   }
}

ObjectAnimator animator = ObjectAnimator.ofObject(view, "position",
        new PointFEvaluator(), new PointF(0, 0), new PointF(1, 1));
animator.start();
```

![of-object](../../../attachments/Android/动画/006-of-object.gif)

