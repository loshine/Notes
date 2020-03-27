# 绘制文字

## 1 普通绘制文字

```java
drawText(String text, float x, float y, Paint paint)
```

> `y`是文字的 baseline

![text](../../../attachments/Android/绘制/006-text.png)

## 2 绘制特殊文字

有些语言的文字会根据上下文的不同而变化写法，这个时候就要使用`drawTextRun()`方法来绘制了。

阿拉伯文里的「عربى（阿拉伯）」是一个四字词，它的中间两个字符「رب」在这个词里的样子，和单独写的时候的样子是不同的。也就是说，当这四个字写在一起的时候，中间两个字由于受到两边的字的影响，形状被改变了。

```java
drawTextRun(CharSequence text, int start, int end, int contextStart, int contextEnd, float x, float y, boolean isRtl, Paint paint)
```

![special-text](../../../attachments/Android/绘制/006-special-text.png)

### 2.1 参数

| 参数名       | 参数意义                                          |
| ------------ | ------------------------------------------------- |
| text         | 要绘制的文字                                      |
| start        | 从那个字开始绘制                                  |
| end          | 绘制到哪个字结束                                  |
| contextStart | 上下文的起始位置。contextStart 需要小于等于 start |
| contextEnd   | 上下文的结束位置。contextEnd 需要大于等于 end     |
| x            | 文字左边的坐标                                    |
| y            | 文字的基线坐标                                    |
| isRtl        | 是否是 RTL（Right-To-Left，从右向左）             |

## 3 按路径绘制文字

耍杂技的。

```java
canvas.drawPath(path, paint); // 把 Path 也绘制出来，理解起来更方便  
canvas.drawTextOnPath("Hello HeCoder", path, 0, 0, paint); 
```

![text-by-path](../../../attachments/Android/绘制/006-text-by-path.png)