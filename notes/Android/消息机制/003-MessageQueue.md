# MessageQueue

MessageQueue 是一个消息队列，用于管理 Handler 中的消息，按时间排序。

## next 方法

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

当消息队列中没有消息的时候会阻塞在 `nativePollOnce`方法，此时主线程释放 CPU 进入休眠状态，直到下个消息到达或者有事务发生，通过往 pipe 管道写端写入数据来唤醒主线程工作。

这里采用的 epoll 机制，是一种 IO 多路复用机制，可以同时监控多个描述符，当某个描述符就绪(读或写就绪)，则立刻通知相应程序进行读或写操作，本质同步 I/O，即读写是阻塞的。所以说，主线程大多数时候都是处于休眠状态，并不会消耗大量CPU资源。

1. handler 机制是使用 pipe 来实现的
2. 主线程没有消息处理时阻塞在管道的读端
3. binder 线程会往主线程消息队列里添加消息，然后往管道写端写一个字节，这样就能唤醒主线程从管道读端返回，也就是说 `queue.next()` 会调用返回

## enqueueMessage 方法

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

当有新消息入队之后，会调用 `nativeWake` 方法唤醒，`next` 方法那边本来被阻塞在 `nativePollOnce`，就会继续运行。

