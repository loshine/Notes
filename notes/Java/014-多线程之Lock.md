# Lock

## 1 简介

Lock 是 synchronized 关键字的进阶，掌握 Lock 有助于学习并发包中的源代码，在并发包中大量的类使用了 Lock 接口作为同步的处理方式。

**Lock 接口的实现类：** ReentrantLock, ReentrantReadWriteLock.ReadLock, ReentrantReadWriteLock.WriteLock。

## 2 特性

* **尝试非阻塞地获取锁**：当前线程尝试获取锁，如果这一时刻锁没有被其他线程获取到，则成功获取并持有锁
* **能被中断地获取锁**：获取到锁的线程能够响应中断，当获取到锁的线程被中断时，中断异常将会被抛出，同时锁会被释放
* **超时获取锁**：在指定的截止时间之前获取锁， 超过截止时间后仍旧无法获取则返回

## 3 基本方法

Lock 接口基本的方法：

| 方法名称                                  | 描述                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| void lock()                               | 获得锁。如果锁不可用，则当前线程将被禁用以进行线程调度，并处于休眠状态，直到获取锁。 |
| void lockInterruptibly()                  | 获取锁，如果可用并立即返回。如果锁不可用，那么当前线程将被禁用以进行线程调度，并且处于休眠状态，和 lock() 方法不同的是在锁的获取中可以中断当前线程（相应中断）。 |
| boolean tryLock()                         | 只有在调用时才可以获得锁。如果可用，则获取锁定，并立即返回值为 true；如果锁不可用，则此方法将立即返回值为 false 。 |
| boolean tryLock(long time, TimeUnit unit) | 超时获取锁，当前线程在一下三种情况下会返回： 1. 当前线程在超时时间内获得了锁；2. 当前线程在超时时间内被中断；3. 超时时间结束，返回 false. |
| void unlock()                             | 释放锁。                                                     |
| Condition newCondition()                  | 获取等待通知组件，该组件和当前的锁绑定，当前线程只有获得了锁，才能调用该组件的 wait() 方法，而调用后，当前线程将释放锁。 |

## 4 具体实现

### 4.1 ReentrantLock

ReentrantLock 和 synchronized 关键字一样可以用来实现线程之间的同步互斥，但是在功能是比 synchronized 关键字更强大而且更灵活。

构造方法：

| 方法名称                    | 描述                                                         |
| --------------------------- | ------------------------------------------------------------ |
| ReentrantLock()             | 创建一个 ReentrantLock 的实例。                              |
| ReentrantLock(boolean fair) | 创建一个特定锁类型（公平锁/非公平锁）的 ReentrantLock 的实例 |

Lock 锁分为：**公平锁**和**非公平锁**。公平锁表示线程获取锁的顺序是按照线程加锁的顺序来分配的，即先来先得的 **FIFO 先进先出顺序**。而非公平锁就是一种获取锁的抢占机制，是**随机获取**锁的，和公平锁不一样的就是先来的不一定先的到锁，这样可能造成某些线程一直拿不到锁，结果也就是不公平的了。ReentrantLock 的构造方法传入布尔值可以设定为公平锁还是非公平锁。

ReentrantLock 类常见方法：

| 方法名称                                                    | 描述                                                         |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| int getHoldCount()                                          | 查询当前线程保持此锁定的个数，也就是调用 lock() 方法的次数。 |
| protected Thread getOwner()                                 | 返回当前拥有此锁的线程，如果不拥有，则返回 null              |
| protected Collection getQueuedThreads()                     | 返回包含可能正在等待获取此锁的线程的集合                     |
| int getQueueLength()                                        | 返回等待获取此锁的线程数的估计。                             |
| protected Collection getWaitingThreads(Condition condition) | 返回包含可能在与此锁相关联的给定条件下等待的线程的集合。     |
| int getWaitQueueLength(Condition condition)                 | 返回与此锁相关联的给定条件等待的线程数的估计。               |
| boolean hasQueuedThread(Thread thread)                      | 查询给定线程是否等待获取此锁。                               |
| boolean hasQueuedThreads()                                  | 查询是否有线程正在等待获取此锁。                             |
| boolean hasWaiters(Condition condition)                     | 查询任何线程是否等待与此锁相关联的给定条件                   |
| boolean isFair()                                            | 如果此锁的公平设置为 true，则返回 true 。                    |
| boolean isHeldByCurrentThread()                             | 查询此锁是否由当前线程持有。                                 |
| boolean isLocked()                                          | 查询此锁是否由任何线程持有。                                 |

### 4.2 ReentrantReadWriteLock

ReentrantLock 是一个排他锁，同一时刻只允许一个线程访问。这样做虽然保证了实例变量的线程安全性，但效率非常低下。ReadWriteLock 接口的实现类 - **ReentrantReadWriteLock** 读写锁就是为了解决这个问题。

读写锁维护了两个锁，一个是读操作相关的锁也称为**共享锁**，一个是写操作相关的锁也称为**排他锁**。通过分离读锁和写锁，其并发性比一般排他锁有了很大提升。

多个读锁之间不互斥，读锁与写锁互斥，写锁与写锁互斥（只要出现写操作的过程就是互斥的）。在没有线程 Thread 进行写入操作时，进行读取操作的多个 Thread 都可以获取读锁，而进行写入操作的 Thread 只有在获取写锁后才能进行写入操作。即*多个 Thread 可以同时进行读取操作，但是同一时刻只允许一个 Thread 进行写入操作*。

读写锁的特性：

* **公平性选择**：支持非公平（默认）和公平的锁获取方式，吞吐量上来看还是非公平优于公平
* **重进入**：该锁支持重进入，以读写线程为例：读线程在获取了读锁之后，能够再次获取读锁。而写线程在获取了写锁之后能够再次获取写锁也能够同时获取读锁
* **锁降级**：遵循获取写锁、获取读锁再释放写锁的次序，写锁能够降级成为读锁

构造方法：

| 方法名称                             | 描述                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| ReentrantReadWriteLock()             | 创建一个 ReentrantReadWriteLock 的实例。                     |
| ReentrantReadWriteLock(boolean fair) | 创建一个特定锁类型（公平锁/非公平锁）的 ReentrantReadWriteLock 的实例 |

其主要提供获取读写锁的方法，其它方法和 ReentrantLock 类似。

| 方法名称         | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| Lock readLock()  | 查询当前线程保持此锁定的个数，也就是调用 lock() 方法的次数。 |
| Lock writeLock() | 返回当前拥有此锁的线程，如果不拥有，则返回 null              |

## 5 Condition

synchronized 关键字与`wait()`和`notify/notifyAll()`方法相结合可以实现等待/通知机制，ReentrantLock 类也可以实现，但是需要借助于 Condition 接口与`newCondition()`方法。

在使用`notify/notifyAll()`方法进行通知时，被通知的线程是由 JVM 选择的，而使用 ReentrantLock 类结合 Condition 实例可以实现 “选择性通知”。

Condition 接口的常见方法：

| 方法名称                                | 描述                                       |
| --------------------------------------- | ------------------------------------------ |
| void await()                            | 相当于 Object 类的 wait 方法               |
| boolean await(long time, TimeUnit unit) | 相当于 Object 类的 wait(long timeout) 方法 |
| signal()                                | 相当于 Object 类的 notify 方法             |
| signalAll()                             | 相当于 Object 类的 notifyAll 方法          |

