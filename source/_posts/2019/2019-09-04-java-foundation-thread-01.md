---
layout: post
title: ﻿【Java基础】多线程（一）
date: 2019-09-04 12:19:24.000000000 +09:00
categories:
- 技术
tags:
- Java
toc: true
---

**
摘要：本篇文章主要阐述了关于多线程的基础知识。首先，通过类比介绍了关于进程和线程的基本概念，进程是资源分配的最小单位，线程是资源调度的最小单位。之后介绍了`Java`中两种实现多线程的方式，一种是继承`Thread`类，一种是实现`Runnable`接口，实现`Runnable`接口更好的解决了`Java`单继承的局限性。再次，通过案例介绍了设置线程和获取线程的方法、线程控制的基本方法，其中主要包括`sleep`、`join`和`setDaemon`。紧接着，阐述了线程调度分为分时调度模型和抢占式调度模型两种方式，而`Java`使用的是抢占式调度模型，`Java`中的线程具有1-10的优先级，优先级越高，不一定就能抢到执行权，只是代表该线程抢占`CPU`时间片的几率更大。最后，主要分析了线程完整生命周期的六种状态，`NEW`新建、`RUNNABLE`运行、`BLOCKED`阻塞、`WAITING`无时限等待、`TIMED_WAITING`有时限等待、`TERMINATED`终止。
**

![](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/java/thread/thread-life.png?raw=true)

<!-- more -->
<The rest of contents | 余下全文>

###进程与线程
#### 什么是进程？
进程（`Process`）是具有一定独立功能的程序、它是系统进行资源分配和调度的一个独立单位，重点在系统调度和单独的单位，也就是说进程是可以独立运行的一段程序。

#### 什么是线程？
线程（`Thread`）进程的一个实体，是`CPU`调度和分派的基本单位，它是比进程更小的能独立运行的基本单位，线程自己基本上不拥有系统资源，在运行时，只是暂用一些计数器、寄存器和栈。

> 注：进程是资源分配的最小单位，线程是资源调度的最小单位。

#### 进程与线程的区别

* 进程是资源（`CPU`、内存等）分配的最小单位，线程是程序执行的最小单位（资源调度的最小单位）。
* 进程有自己的独立地址空间，每启动一个进程，系统就会为它分配地址空间，建立数据表来维护代码段、堆栈段和数据段，这种操作非常昂贵。线程是共享进程中的数据的，使用相同的地址空间，因此`CPU`切换一个线程的花费远比进程要小很多，同时创建一个线程的开销也比进程要小很多。
* 线程之间的通信更方便，同一进程下的线程共享全局变量、静态变量等数据，而进程之间的通信需要以进程间通信的方式 `IPC`（`Inter-Process Communication`）进行。不过如何处理好同步与互斥是编写多线程程序的难点
* 多进程程序更健壮，多线程程序只要有一个线程死掉，整个进程也死掉了，而一个进程死掉并不会对另外一个进程造成影响，因为进程有自己独立的地址空间。

#### 进程与线程的类比
类比：进程=火车，线程=车厢

* 一个进程可以包含多个线程（一辆火车包含多节车厢）
* 线程依赖于进程，它是进程中一个完整的执行路径 （车厢依赖火车，单纯的车厢无法运行）
* 进程间的通信通过`IPC`(`Inter-Process Communication`）进行,比如管道(`pipe`)、信号量(`semophore`)、消息队列(`messagequeue`) 、 套接字(`socket`)等 （一辆火车上的乘客换到另外一辆火车，需要在站点进行换乘）
* 线程间的通信通过共享内存（`Shared Memory`）、消息队列等方式进行 （同一辆火车，A车厢换到B车厢很容易）
* 创建一个进程的开销比创建一个线程开销要消耗更多的计算机资源 （采用多列火车相比多个车厢更耗资源）
* 进程间不会相互影响，但是一个线程挂掉将导致整个进程挂掉（火车之间相互不影响，一个车厢断裂会影响火车运行）
* 一个线程使用共享内存时，其他线程必须等它结束，才能使用这一块内存 。多个线程同时对同一公共资源（比如全局变量）进行读写需要使用互斥锁（车厢中使用洗手间，需要上锁）
* 一个进程使用的内存地址可以限定使用量--信号量（火车上的餐厅最多同时容纳一定乘客数量，需要等有人出来才能进去）

### 多线程的两种实现方式

在`Java`中有两种方式实现多线程，一种是继承`Thread`类，一种是实现`Runnable`接口。

#### 通过 Thread 类继承实现多线程
方式1：继承`Thread`类。

定义一个`MyThread`类继承`Thread`类，并在`MyThread`类中重写`run()`方法，创建`MyThread`类的对象，启动线程。

`MyThread`类：

```
public class MyThread extends Thread {
    @Override
    public void run() {
        for(int i=0; i<100; i++) {
            System.out.println(i);
        }
    }
}
```
`MyThreadDemo`类：
```
public class MyThreadDemo {
    public static void main(String[] args) {
        // 创建MyThread类的对象
        MyThread my1 = new MyThread();
        MyThread my2 = new MyThread();

        // void run() 线程开启后，此方法将被调用执行
        // my1.run();
        // my2.run(); // 线程1和线程2按序执行

        // void start() 线程开始执行; Java虚拟机调用此线程的run方法
        my1.start(); // 0 1 2 ..78 0 1 2 .. 79 ..
        my2.start(); // 线程1和线程2交替执行
    }
}
```
> 调用`start()`方法方可启动线程，而`run()`方法只是`Thread`的一个普通方法调用，实质还是在主线程里执行。简单来讲，`start() `实现了多线程，`run()`没有实现多线程。

#### 通过 Runnable 接口实现多线程

方式2：通过`Runnable`接口实现。
 
定义一个类`MyRunnable`类并实现`Runnable`接口，在`MyRunnable`类中重写`run()`方法，在测试类中创建`MyRunnable`类的对象和`Thread`类的对象，把`MyRunnable`对象作为构造方法的参数，最后通过`start()`方法启动线程。

`MyRunnable`类：
```
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        for(int i=0; i<100; i++) {
            System.out.println(Thread.currentThread().getName()+":"+i);
        }
    }
}
```

`MyRunnableDemo`类：

```
public class MyRunnableDemo {
    public static void main(String[] args) {
        // 创建MyRunnable类的对象
        MyRunnable my = new MyRunnable();

        // 创建Thread类的对象，把MyRunnable对象作为构造方法的参数
        // Thread(Runnable target)
        // Thread t1 = new Thread(my);// Thread-0:0 Thread-0:1 Thread-1:0 ...
        // Thread t2 = new Thread(my);
        // Thread(Runnable target, String name)
        Thread t1 = new Thread(my,"subway");
        Thread t2 = new Thread(my,"airplane");

        // 启动线程
        t1.start(); // airplane:0 airplane:1 subway:0 airplane:2
        t2.start();
    }
}
```
#### 继承Thread类和实现Runnable接口的区别

* 实现`Runnable`接口的方式解决`JAVA`中单继承的局限性。如果你想写一个类C，但这个类C已经继承了一个类B，此时，你又想让C实现多线程，由于单继承的原因，此时无法使用继承`Thread`类的方式实现，只能通过实现`Runnable`接口的方式。

### 设置和获取线程名称

|方法名| 说明|
|-----|----|
|void setName(String name)|设置线程的名称，默认为Thread-0...|
|String getName()|返回线程的名称|
|Thread currentThread()|返回对当前正在执行的线程对象的引用|
`MyThread`类：
```
public class MyThread extends Thread {
    public MyThread() {}
    public MyThread(String name) {
        super(name);
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(getName()+":"+i);
        }
    }
}
```

`MyThreadDemo`类：
```
public class MyThreadDemo {
    public static void main(String[] args) {
         // MyThread my1 = new MyThread();
         // MyThread my2 = new MyThread();

        // void setName(String name)：将此线程的名称更改为等于参数 name
        // my1.setName("subway"); // airplane:0 subway:0 ...
        // my2.setName("airplane");

        // Thread(String name)
        MyThread my1 = new MyThread("subway");
        MyThread my2 = new MyThread("airplane");

        my1.start(); // airplane:0 subway:0 ...
        my2.start();

        // static Thread currentThread() 返回对当前正在执行的线程对象的引用
        System.out.println(Thread.currentThread().getName());
    }
}
```
### 线程的优先级
线程的调度分为分时调度模型和抢占式调度模型两种方式：

分时调度模型：所有线程轮流使用 `CPU` 的使用权，平均分配每个线程占用 `CPU` 的时间片。
抢占式调度模型：优先让优先级高的线程使用 `CPU`，如果线程的优先级相同，那么会随机选择一 个，优先级高的线程获取的 `CPU` 时间片相对多一些。

线程的调度具有随机性，优先级高代表抢到 `CPU` 的时间片的几率高，并不代表一定获得时间片。假如计算机只有一个 `CPU`，那么 `CPU` 在某一个时刻只能执行一条指令，线程只有得到`CPU`时间片，也就是使用权，才可以执行指令。所以说多线程程序的执行是有随机性，因为谁抢到`CPU`的使用权是不一 定的

> `Java`使用的是抢占式调度模型。

|方法名| 说明|
|-----|----|
|final int getPriority()|获取线程的优先级|
|final void setPriority(int newPriority)|设置线程的优先级，线程默认优先级是5；线程优先级的范围 是：1-10|

`ThreadPriority`类：
```
public class ThreadPriority extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(getName() + ":" + i);
        }
    }
}
```
`ThreadPriorityDemo`类：
```
public class ThreadPriorityDemo {
    public static void main(String[] args) {
        ThreadPriority tp1 = new ThreadPriority();
        ThreadPriority tp2 = new ThreadPriority();
        ThreadPriority tp3 = new ThreadPriority();

        tp1.setName("subway");
        tp2.setName("airplane");
        tp3.setName("car");

        // public final int getPriority()：返回此线程的优先级
        // System.out.println(tp1.getPriority()); // 5
        // System.out.println(tp2.getPriority()); // 5
        // System.out.println(tp3.getPriority()); // 5

        // public final void setPriority(int newPriority)：更改此线程的优先级
        // tp1.setPriority(10000); // IllegalArgumentException
        // System.out.println(Thread.MAX_PRIORITY); // 10
        // System.out.println(Thread.MIN_PRIORITY); // 1
        // System.out.println(Thread.NORM_PRIORITY); // 5

        // 设置正确的优先级
        tp1.setPriority(1);
        tp2.setPriority(10);
        tp3.setPriority(1);

        tp1.start(); // airplane:0 subway:0 car:0 subway:1
        tp2.start();
        tp3.start();
    }
}
```

### 线程控制
相关方法：

|方法名| 说明|
|-----|----|
|static void sleep(long millis)|使当前正在执行的线程停留(暂停执行)指定的毫秒数|
|void join()|等待这个线程死亡|
|void setDaemon(boolean on)|将此线程标记为守护线程，当运行的线程都是守护线程时，Java虚拟机将退出|

#### sleep 案例
`ThreadSleep`类：
```
public class ThreadSleep extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(getName() + ":" + i);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

`ThreadSleepDemo`类：
```
public class ThreadSleepDemo {
    public static void main(String[] args) {
        ThreadSleep ts1 = new ThreadSleep();
        ThreadSleep ts2 = new ThreadSleep();
        ThreadSleep ts3 = new ThreadSleep();

        ts1.setName("wx001");
        ts2.setName("wx002");
        ts3.setName("wx003");

        ts1.start(); 
        ts2.start();
        ts3.start();
    }
}
```
`ts1`、`ts2`、`ts3`每个在执行的线程都延迟一秒后被唤醒继续执行。

#### join  案例
`ThreadJoin`类：
```
public class ThreadJoin extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(getName() + ":" + i);
        }
    }
}
```
`ThreadJoinDemo`类：
```
public class ThreadJoinDemo {
    public static void main(String[] args) {
        ThreadJoin tj1 = new ThreadJoin();
        ThreadJoin tj2 = new ThreadJoin();
        ThreadJoin tj3 = new ThreadJoin();

        tj1.setName("wx001");
        tj2.setName("wx002");
        tj3.setName("wx003");

        tj1.start();
        try {
            tj1.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        tj2.start();
        tj3.start();
    }
}
```

等待`tj1`线程挂掉之后，才会执行`tj2`线程和`tj3`线程。

#### setDaemon 案例
`ThreadDaemon`类：
```
public class ThreadDaemon extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println(getName() + ":" + i);
        }
    }
}
```
`ThreadDaemonDemo`类：
```
public class ThreadDaemonDemo {
    public static void main(String[] args) {
        ThreadDaemon td1 = new ThreadDaemon();
        ThreadDaemon td2 = new ThreadDaemon();

        td1.setName("wx001");
        td2.setName("wx002");

        // 设置主线程为wangxiong
        Thread.currentThread().setName("wangxiong");

        // 设置守护线程
        td1.setDaemon(true);
        td2.setDaemon(true);

        td1.start();
        td2.start();

        for(int i=0; i<2; i++) {
            // td1和td2线程被标记为守护线程，当主线程运行结束后，td1和td2线程都是守护线程时，Java虚拟机将退出
            System.out.println(Thread.currentThread().getName()+":"+i);
        }
    }
}
```
`td1`和`td2`线程被标记为守护线程，当主线程运行结束后，因`td1`和`td2`线程都是守护线程，Java虚拟机将退出。

### 线程生命周期

下图是一个线程的生命周期状态流转图，很清楚的描绘了一个线程从创建到终止的过程。

![](https://github.com/LensXiong/hexo_source_code/blob/master/img/technology/2019/java/thread/thread-life.png?raw=true)

线程完整生命周期的状态枚举值定义在`java.lang.Thread.State`下。

```
public enum State {
        /**
         * Thread state for a thread which has not yet started.
         * 表示刚创建的线程对象，还没有开始启动。
         * 没有执行资格，也没有执行权。
         */
        NEW,

        /**
         * Thread state for a runnable thread. A thread in the runnable
         * state is executing in the Java virtual machine but it may
         * be waiting for other resources from the operating system
         * such as processor.
         * 表示线程处于运行中状态，调用了start()方式，线程正式启动。
         * 有执行资格，也有执行权
         */
        RUNNABLE,

        /**
         * Thread state for a thread blocked waiting for a monitor lock.
         * A thread in the blocked state is waiting for a monitor lock
         * to enter a synchronized block/method or
         * reenter a synchronized block/method after calling
         * {@link Object#wait() Object.wait}.
         * 表示线程阻塞，等待获取锁，如碰到 synchronized、lock 等关键字等占用临界区的情况，一旦获取到锁就进行 RUNNABLE 状态继续运行。
         * 没有执行资格，也没有执行权。
         */
        BLOCKED,

        /**
         * Thread state for a waiting thread.
         * 表示线程进入了一个无时限的等待就绪状态，需要一个事件重新唤醒，从而进入到 RUNNABLE 状态。
         * 有执行资格，没有执行权。
         */
        WAITING,

        /**
         * Thread state for a waiting thread with a specified waiting time.
         * A thread is in the timed waiting state due to calling one of
         * 表示线程进入了一个有时限的等待，如 sleep(1000)，等待1秒后线程重新进行 RUNNABLE 状态继续运行。
         * 有执行资格，没有执行权。
         */
        TIMED_WAITING,

        /**
         * Thread state for a terminated thread.
         * The thread has completed execution.
         * 表示线程执行完毕后，进行终止状态。
         * 没有执行资格，也没有执行权。
         */
        TERMINATED;
    }
```

`NEW`：表示刚创建的线程对象，还没有开始启动。（没有执行资格，也没有执行权）

`RUNNABLE`：表示线程处于运行中状态，调用了`start()`方式，线程正式启动。（有执行资格，也有执行权）

`BLOCKED`：表示线程阻塞，等待获取锁，如碰到 `synchronized`、`lock` 等关键字等占用临界区的情况，一旦获取到锁就进行 `RUNNABLE`状态继续运行。（没有执行资格，也没有执行权）

`WAITING`：表示线程进入了一个无时限的等待就绪状态，需要一个事件重新唤醒，从而进入到 `RUNNABLE` 状态。（有执行资格，没有执行权）

`TIMED_WAITING`：表示线程进入了一个有时限的等待，如 `sleep`(1000)，等待1秒后线程重新进行 `RUNNABLE `状态继续运行。（有执行资格，没有执行权）

`TERMINATED`：表示线程执行完毕后，进行终止状态。（没有执行资格，也没有执行权）