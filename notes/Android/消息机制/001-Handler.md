# Android 消息机制 —— Handler

## 前言

消息机制这种思想可以说是操作系统的基本机制之一，通过消息来驱动事件，满足交互的需求。

常见的操作系统如 Windows、Android 都有自己的消息机制的实现。

## 从 Handler 谈起

通常我们做消息分发时，都是通过 Handler 帮我们实现，

```java
// 直接发送一个消息
mHandler.sendEmptyMessage(YOUR_MSG);

...

// 发送一个待执行的Runnable
mHandler.post(new Runnable() {
    public void run() {
        // 待执行的逻辑
    }
});
```

上述的这些方法都是一层封装，通过构造 Message 对象，最终都会走到如下代码：

```java
...
public final boolean sendMessageDelayed(Message msg, long delayMillis) {
    if (delayMillis < 0) {
        delayMillis = 0;
    }
    // 将延迟的相对时间变成一个绝对的时间点
    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}
...

public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    // 这里出现了我们经常提起的消息队列类MessageQueue
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    // 将构造好的Message对象入队
    return enqueueMessage(queue, msg, uptimeMillis);
}
...

private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    // Message的target成员变量是一个Handler(消息分发是通过判断它，进而知道要交给哪个handler去处理消息的)
    // 这里将当前handler赋值给它
    msg.target = this;
    // mAsynchronous在Handler构造函数中赋值，交给调用方去选择是否异步
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    // 调用MessageQueue中的enqueueMessage入队，这个是核心的实现
    return queue.enqueueMessage(msg, uptimeMillis);
}
```

以上分析我们知道，Handler 最终消息的入队是交由 MessageQueue 来实现的，我们来看一下具体源码

```java
// 其实MessageQueue虽然是一个队列，但是是通过链表来实现的
boolean enqueueMessage(Message msg, long when) {
    ...

    synchronized (this) {
        ...

        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        // 如果是新的队列头，直接插入队列
        if (p == null || when == 0 || when < p.when) {
            msg.next = p;
            mMessages = msg;
            // 这个布尔值标志着是否需要唤醒事件队列，稍后将会讲到
            needWake = mBlocked;
        } else {
            // 按照未来执行的时间点when，插入链表中(这也是为什么用链表实现的原因)
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            for (;;) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            msg.next = p; // invariant: p == prev.next
            prev.next = msg;
        }

        // 插入消息的时候，一般不会唤醒消息队列
        // 如果消息是异步的，并且队列头不是一个异步消息的时候，会唤醒消息队列
        if (needWake) {
            // 唤醒消息队列是native层的逻辑
            nativeWake(mPtr);
        }
    }
    return true;
}
```

以上讲解了消息队列的入队的逻辑，我们来看看消息队列是怎么分发消息的。在 Android 中，是靠着 Looper 来实现从消息队列中取消息进而分发给对应的 Handler 的。Looper 有两个重要的方法`prepare()`和`loop()`

```java
// 通常我们直接调用prepare()方法时，quitAllowed默认是为true的
// quitAllowed最终会传递给MessageQueue的构造函数，通过这个布尔值来控制是否允许退出消息队列

// 查看源码发现，如果prepareMainLooper()，即初始化主线程上的Looper，默认quitAllowed是为false的。
// 这也符合常理，主线程不允许退出消息队列。如果强行执行MessageQueue.quit()将会抛出异常。
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    // 初始化Looper并设置给ThreadLocal
    sThreadLocal.set(new Looper(quitAllowed));
}
```

在 Java 中，ThreadLocal 为变量在每个线程都创建一个副本，每个线程都可以访问自己的内部副本变量。ThreadLocal 不是用来解决共享对象的多线程访问问题的，而是使用 ThreadLocal 使得各线程能够保持各自独立的一个对象。

```java
public static void loop() {
    ...
    for (;;) {
        // 可能会阻塞
        Message msg = queue.next();
        ...
        // 调用Handler.dispatchMessage方法分发Message消息
        msg.target.dispatchMessage(msg);
    }
    ...
}
```

`Handler.dispatchMessage`, 由源码可知，Handler 对 Runnable、Handler 自定义的 Callback 和继承 Handler 分别做了回调函数的适配，对开发者而言，使用 Handler 变得更为灵活和方便

```java
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        // 这里就是你传进来的runnable
        handleCallback(msg);
    } else {
        // mCallback可以在Handler的构造函数中定义
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        // 自带的handleMessage可以在继承Handler时实现
        handleMessage(msg);
    }
}
```

`MessageQueue.next`中实现细节

```java
Message next() {
    ...
    for (;;) {
        ...
        // 稍后会讲到，Linux系统的IO多路复用机制
        nativePollOnce(ptr, nextPollTimeoutMillis);
        ...
        // 如果消息队列中有异步消息，则优先执行异步消息，然后再顺序执行剩余消息
        if (msg != null && msg.target == null) {
            // Stalled by a barrier.  Find the next asynchronous message in the queue.
            do {
                prevMsg = msg;
                msg = msg.next;
            } while (msg != null && !msg.isAsynchronous());
        }
        
        if (msg != null) {
            if (now < msg.when) {
                // 下一条消息未到执行时间，赋值一个超时的时间
                nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
            } else {
                mBlocked = false;
                if (prevMsg != null) {
                    prevMsg.next = msg.next;
                } else {
                    mMessages = msg.next;
                }
                msg.next = null;
                ...
                msg.markInUse();
                // 将获取到Message移出队列，并返回
                return msg;
            }
        } else {
            // 没有Message了，赋值-1表示无限等待，直到下一条消息到来
            nextPollTimeoutMillis = -1;
        }
    }
}
```

在 Message 为空的时候，会去处理之前添加过的 IdleHandler，因此我们可以利用 IdleHandler 实现一些功能

```java
for (int i = 0; i < pendingIdleHandlerCount; i++) {
    ...
    boolean keep = false;
    try {
        keep = idler.queueIdle();
    } catch (Throwable t) {
        ...
    }

    if (!keep) {
        synchronized (this) {
            mIdleHandlers.remove(idler);
        }
    }
}

public static interface IdleHandler {
    //返回true将你的idle handler保留，false则从list中移除
    boolean queueIdle();
}
```

总结一下，Android 的消息机制就是，通过不断地使用 Handler 将消息发送给一个消息队列入队，并且有一个无限循环的 Looper，不断地从消息队列里取出消息然后分发给对应的目标 Handler 执行。