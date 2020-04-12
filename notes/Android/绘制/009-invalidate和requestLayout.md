## invalidate()

view 的 invalidate 不会导致 ViewRootImpl 的 invalidate 被调用，而是递归调用父 view 的 invalidateChildInParent，直到 ViewRootImpl 的 invalidateChildInParent，然后触发 performTraversals。

## requestLayout()

首先判断当前 View 树是否正在布局流程，接着为当前子 View 设置标记位，该标记位的作用就是标记了当前的 View 是需要进行重新布局的。最后调用父 View 的 requestLayout 方法，不断调用父 View，直至 ViewRootImpl。

和`invalidate`一样调用`scheduleTraversals`方法，但一定不会触发 draw 方法。

requestLayout 在 layout 过程中发现 l,t,r,b 和以前不一样，就会触发一次 invalidate，这种情况下回调用 onDraw 方法。

## 使用情况

只要刷新的时候就调用 invalidate，需要重新 measure 就调用 requestLayout

