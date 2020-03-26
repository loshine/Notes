# Android 消息机制 —— Looper

Looper 在 android 的消息机制充当消息循环的角色，它不停的从 MessageQueue 中拿出消息，并将消息交给 Hanlder 处理，下面是他的常用方法解析。

## Looper 的创建

> android 主线程在创建的时候会主动创建一个 Looper

ActivityThread 是 android 的主线程，在该类的`main`函数中有如下部分代码

```java
public static void main(String[] args) {
    ...
    // 创建Looper
    Looper.prepareMainLooper();
    
    ActivityThread thread = new ActivityThread();
    thread.attach(false);
    ...
    
    if (sMainThreadHandler == null) {
        sMainThreadHandler = thread.getHandler();
    }
    Looper.loop();
}
```

`Looper.prepareMainLooper()`方法的代码部分代码

```java
public static void prepareMainLooper() {
   // 调用Looper的prepare()方法创建Looper
   prepare(false);
 ...
}
```

从上面的信息我们可以知道，ActivityThread 的`main`函数调用了 `Looper.prepareMainLooper`函数，而在`prepareMainLooper`函数中有调用了 Looper 的`prepare`来创建 Looper。那么`prepare`函数是如何创建 Looper 呢？我们看下面第二种方式来分析。

## 在子线程中直接使用 Looper.prepare() 方法创建

```java
public static void prepare() {
    prepare(true);
}

private static void prepare(boolean quitAllowed) {
    // 一个线程中只能有一个Looper
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    // 创建Looper
    sThreadLocal.set(new Looper(quitAllowed));
}
```

在线程中我们可以调用`Looper.prepare`，这个函数中`sThreadLocal.set()`中创建一个 Looper，该方法不仅创建了 Looper，其实也创建了 MessageQueue, 以及将 Looper 将当前线程和 Looper 关联起来了。我们看一看 Looper 的构造就明白了

```java
private Looper(boolean quitAllowed) {
    // 创建消息队列
    mQueue = new MessageQueue(quitAllowed);
    // 获取当前线程的引用
    mThread = Thread.currentThread();
}
```

## Looper 的的 Loop 方法

该方法主要是用来从消息队列里面拿出消息。我们可以看下面的部分源码

```java
public static void loop() {
    ...
    final MessageQueue queue = me.mQueue;
    ...
    for (;;) {
        Message msg = queue.next(); // might block
        if (msg == null) {
            return;
        }
    ...    
        try {
            // 调用handler处理消息
            msg.target.dispatchMessage(msg);
            end = (slowDispatchThresholdMs == 0) ? 0 :SystemClock.uptimeMillis();
        } finally {
            if (traceTag != 0) {
                Trace.traceEnd(traceTag);
            }
        }
       ...
    }
}
```

上面的代码中`loop()`函数里面有一个无限循环，这个循环就是不停的从 MessageQueue 中取消息，并将取出消息交给 handler 的`dispatchMessage()`来处理，只有当`queue.next()`取得 Message 为`null`的时候才跳出循环。

## Looper 其他常用方法

```java
// 获取当前线程的Looper
Looper looper = Looper.myLooper();
// 获取主线程的Looper
Looper mainLooper = Looper.getMainLooper();
// 清除全部消息
looper.quit();
// 清除延迟消息
looper.quitSafely();
```