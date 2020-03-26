# 多线程之 Thread

## 一 进程和多线程简介

### 1.1 相关概念

> **何为线程？**

线程与进程相似，但线程是一个比进程更小的执行单位。一个进程在其执行的过程中可以产生多个线程。与进程不同的是同类的多个线程共享同一块内存空间和一组系统资源，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多，也正因为如此，线程也被称为轻量级进程。

> **何为进程？**

进程是程序的一次执行过程，是系统运行程序的基本单位，因此进程是动态的。系统运行一个程序即是一个进程从创建，运行到消亡的过程。简单来说，一个进程就是一个执行中的程序，它在计算机中一个指令接着一个指令地执行着，同时，每个进程还占有某些系统资源如 CPU 时间，内存空间，文件，文件，输入输出设备的使用权等等。换句话说，当程序在执行时，将会被操作系统载入内存中。

> **线程和进程有何不同？**

线程是进程划分成的更小的运行单位。线程和进程最大的不同在于基本上各进程是独立的，而各线程则不一定，因为同一进程中的线程极有可能会相互影响。从另一角度来说，进程属于操作系统的范畴，主要是同一段时间内，可以同时执行一个以上的程序，而线程则是在同一程序内几乎同时执行一个以上的程序段。

## 二 使用多线程

### 2.1 继承 Thread 类

```java
public class MyThread extends Thread {
    @Override
    public void run() {
        super.run();
        System.out.println("MyThread");
    }
}

public class Run {
    public static void main(String[] args) {
        MyThread mythread = new MyThread();
        mythread.start();
        System.out.println("运行结束");
    }
}
```

### 2.2 实现 Runnable 接口

推荐实现 Runnable 接口方式开发多线程，因为 Java 单继承但是可以实现多个接口。

```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("MyRunnable");
    }
}

public class Run {
    public static void main(String[] args) {
        Runnable runnable = new MyRunnable();
        Thread thread = new Thread(runnable);
        thread.start();
        System.out.println("运行结束！");
    }
}
```

## 三 实例变量和线程安全

定义线程类中的实例变量针对其他线程可以有共享和不共享之分，共享变量时需要考虑线程安全。

## 四 Thread 类的一些常用方法

| 方法名                       | 方法作用                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| currentThread()              | 返回对当前正在执行的线程对象的引用                           |
| getId()                      | 返回此线程的标识符                                           |
| getName()                    | 返回此线程的名称                                             |
| getPriority()                | 返回此线程的优先级                                           |
| isAlive()                    | 测试这个线程是否还处于活动状态                               |
| sleep(long millis)           | 使当前正在执行的线程以指定的毫秒数 “休眠”                    |
| interrupt()                  | 中断这个线程                                                 |
| interrupted()                | 测试当前线程是否已经是中断状态，将状态标志清除为 false       |
| isInterrupted()              | 测试线程 Thread 对相关是否已经是中断状态，但不清楚状态标志   |
| setName(String name)         | 将此线程的名称更改为参数 name                                |
| isDaemon()                   | 测试这个线程是否是守护线程                                   |
| setDaemon(boolean on)        | 将此线程标记为 daemon 线程或用户线程                         |
| join()                       | t.join() 方法阻塞调用此方法的线程(calling thread)，直到线程 t 完成，此线程再继续 |
| yield()                      | 放弃当前的 CPU 资源，将它让给其他的任务去占用 CPU 时间       |
| setPriority(int newPriority) | 更改此线程的优先级                                           |

## 五 停止一个线程

使用`interrupt()`, `isInterrupted()`和 return 关键词：

```java
public class MyThread extends Thread {
    @Override
    public void run() {
        while (true) {
            if (this.isInterrupted()) {
                System.out.println("ֹͣ停止了!");
                return;
            }
            System.out.println("timer=" + System.currentTimeMillis());
        }
    }
}

public class Run {
	public static void main(String[] args) throws InterruptedException {
      MyThread t = new MyThread();
      t.start();
      Thread.sleep(2000);
      t.interrupt();
	}
}
```

## 六 线程的优先级

每个线程都具有各自的优先级，线程的优先级可以在程序中表明该线程的重要性，如果有很多线程处于就绪状态，系统会根据优先级来决定首先使哪个线程进入运行状态。但这个并不意味着低 优先级的线程得不到运行，而只是它运行的几率比较小，如垃圾回收机制线程的优先级就比较低。所以很多垃圾得不到及时的回收处理。

线程优先级具有继承特性比如 A 线程启动 B 线程，则 B 线程的优先级和 A 是一样的。

线程优先级具有随机性也就是说线程优先级高的不一定每一次都先执行完。

Thread 类中包含的成员变量代表了线程的某些优先级。如 **Thread.MIN_PRIORITY（常数 1）**，**Thread.NORM_PRIORITY（常数 5）**，**Thread.MAX_PRIORITY（常数 10）**。其中每个线程的优先级都在 **Thread.MIN_PRIORITY（常数 1）** 到 **Thread.MAX_PRIORITY（常数 10）**之间，在默认情况下优先级都是 **Thread.NORM_PRIORITY（常数 5）**。

## 七 JAVA 多线程分类

### 7.1 多线程分类

* 用户线程：运行在前台，执行具体的任务，如程序的主线程、连接网络的子线程等都是用户线程

* 守护线程：运行在后台，为其他前台线程服务. 也可以说守护线程是 JVM 中非守护线程的 **“佣人”**。

  特点：一旦所有用户线程都结束运行，守护线程会随 JVM 一起结束工作

  应用：数据库连接池中的检测线程，JVM 虚拟机启动后的检测线程

  最常见的守护线程：垃圾回收线程

### 7.2 如何设置守护线程？

可以通过调用 Thead 类的`setDaemon(true)`方法设置当前的线程为守护线程。