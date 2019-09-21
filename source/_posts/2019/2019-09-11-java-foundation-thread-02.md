---
layout: post
title: ﻿【Java基础】多线程（二）
date: 2019-09-11 22:11:24.000000000 +09:00
categories:
- 技术
tags:
- Java
toc: true
---

**
摘要：本篇文章以电影院卖票为案例需求，主要阐述了多线程安全的问题是如何产生的，为了解决电影院售卖相同票的问题，引入`Java`中关键字`synchronized`同步代码块和同步方法的方式来解决数据安全问题，其中需要注意的是在同步方法中的静态同步方法的锁对象是当前函数所属类的字节码文件`类名.class`，非静态同步方法的锁对象是`this`，最后也简单介绍了`JDK5`以后提供的一个新的锁`Lock`机制来解决线程安全问题。线程同步的好处虽然帮我们更好的解决了多线程安全的问题，但当线程非常多时，为了同步上的锁也会大大降低程序的运行效率，我们应该根据实际业务场景进行权衡。
**

<!-- more -->
<The rest of contents | 余下全文>

# 前言
某电影院正在热映某大片，共有100张票，同时在售卖的有3个窗口，请设计一个程序模拟该电影院卖票，要求如下：
① 不能超出100张总数量
② 不能售卖相同的票

其实要解决售卖相同票的问题，实质上需要解决多线程安全的问题，那如何解决多线程安全的问题？

主要思路是把多条语句操作共享数据的代码给锁起来，让任意时刻只能有一个线程执行即可，`Java`提供了同步代码块的方式、同步方法的方式来解决数据安全问题。

虽然线程同步的好处更好的帮我们解决了多线程安全的问题，但是当线程非常多时，每个线程都会去判断同步上的锁，会很耗费资源，无形中会降低程序的运行效率，所以说线程同步是一把双刃剑，在实际中我们需要根据数据业务场景进行权衡，符合实际业务场景的设计才是好设计。


# 卖票应用

根据上面的案例需求，我们定义一个类`SellTicket`实现`Runnable`接口，并在里面定义一个成员变量：

```
private int tickets = 100;

```

在`SellTicket`类中重写`run()`方法实现卖票，具体代码步骤如下：


```
public class SellTicket implements Runnable {

    // 总票数
    private int tickets = 100;
    @Override
    public void run() {
        // 死循环让卖票动作一直执行（因为票没了，也有可能有人来问）
        while (true) {
            // 票数大于0，卖票并告知售票窗口
            if (tickets > 0) {
                System.out.println(Thread.currentThread().getName() + "正在出售第" + tickets + "张票");
                // 票数减1
                tickets--;
            }
        }
    }
}
```
接下来，我们在`SellTicketDemo`类中创建卖票`SellTicket`类的对象，创建三个`Thread`线程类对象并启动所创建的三个线程。

`SellTicketDemo`类：

```
public class SellTicketDemo {
    public static void main(String[] args) {
        // 创建SellTicket类的对象
        SellTicket st = new SellTicket();

        // 创建三个Thread类的对象，把SellTicket对象作为构造方法的参数，并给出对应的窗口名称
        Thread t1 = new Thread(st,"窗口1");
        Thread t2 = new Thread(st,"窗口2");
        Thread t3 = new Thread(st,"窗口3");

        // 启动线程
        t1.start();
        t2.start();
        t3.start();
    }
}
```

运行结果后我们会发现如下现象：

```
窗口3正在出售第100张票
窗口2正在出售第100张票
窗口1正在出售第100张票
窗口2正在出售第98张票
窗口3正在出售第99张票
窗口2正在出售第96张票
...
窗口1正在出售第1张票
窗口2正在出售第0张票
窗口3正在出售第-1张票
```

分析以上运行结果我们可以发现，卖票出现了两个问题：

* 相同的票出现了多次
* 出现了负数的票（运行中可能不会出现，但实际上是有这种可能性的）

首先相同的票出现了多次，原因是线程执行的随机性导致的。`t1`线程、`t2`线程、`t3`线程都有可能同时抢到`CPU`的执行权，这个时候共享的总票数资源是100，就会分别在控制台输出：窗口1/2/3正在出售第100张票；相应的票数也会减少，当下个线程再次抢到`CPU`的执行权的时候，票数刚好减为98。

其次出现负数的原因也是因为线程执行的随机性导致，当票数减为1张时，三个线程同时抢到`CPU`的执行权，`t1`在控制台输出：窗口1正在出售第1张票，执行`tickets--`操作，`tickets = 0`；`t2`在控制台输出：窗口2正在出售第0张票，3行`tickets--`操作，`tickets = -1`；`t3`在控制台输出：窗口3正在出售第-1张票，执行`tickets--`操作，`tickets = -2`；

# 同步代码块方式解决线程安全问题

同步代码块格式:

```
synchronized(任意对象obj) { 
多条语句操作共享数据的代码
}
```

任意对象`obj`称为同步监视器，也就是锁，它的原理是当线程开始执行同步代码块前，必须先获得对同步代码块的锁定，并且任何时刻只能有一个线程可以获得对同步监视器的锁定，当同步代码块执行完成后，该线程会释放对该同步监视器的锁定。

注：`synchronized`(任意对象)，就相当于给代码加锁，任意对象`obj`就可以看成是一把锁。

按照以上的思路，我们将`SellTicket`类的代码改写为：

```
public class SellTicket implements Runnable {
    private int tickets = 100;
    private Object obj = new Object();
    @Override

    public void run() {
        while (true) {
            // 假设t1线程、t2线程、t3线程都抢到了CPU的执行权
            synchronized (obj) {
                // t1线程进来后，就会把这段代码给锁起来
                if (tickets > 0) {
                    try {
                        Thread.sleep(100);
                        // t1线程休息100毫秒
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    // 窗口1正在出售第100张票
                    System.out.println(Thread.currentThread().getName() + "正在出售第" + tickets + "张票");
                    tickets--; // tickets = 99;
                }
            }
            // t1线程出来，这段代码的锁才会被释放
        }
    }
}

```

打印结果：

```
窗口1正在出售第100张票
窗口1正在出售第99张票
窗口1正在出售第98张票
窗口2正在出售第97张票
窗口3正在出售第96张票
...
窗口2正在出售第2张票
窗口2正在出售第1张票
```

假设`t1`线程、`t2`线程、`t3`线程都抢到了`CPU`的执行权，`t1`线程进来后，就会把这段代码给锁起来，当`t1`线程休息100毫秒，就会打印窗口1正在出售第100张票，并将票数减为99，直至执行完毕改同步代码块内的内容后，`t1`线程才出来，这段代码的锁才会被释放，`t2`线程才能进入代码块执行，如此循环执行...

# 同步方法解决线程安全问题

同步方法分为静态同步方法和非静态同步方法：

静态同步方法的格式：

```
修饰符 static synchronized 返回值类型 方法名(方法参数) { 方法体;
}
```

非静态同步方法的格式：

```
修饰符 synchronized 返回值类型 方法名(方法参数) { 方法体;
}
```

静态同步方法的锁对象是当前函数所属的类的字节码文件`类名.class`，非静态同步方法的锁对象`this`。

```
public class SellTicket implements Runnable {
    private static int tickets = 100;
    private Object obj = new Object();
    private int x = 0;
    @Override

    public void run() {
        while (true) {
            if (x % 2 == 0) {
                sellTicketSynchronized(); // 同步代码块
            } else {
                // 非静态同步方法，锁对象是this，所以需要配合同步代码块的synchronized (this) 使用
                // sellTicketNotStatic();
                // 静态同步方法，锁对象是类名.class，所以需要配合同步代码块的synchronized (SellTicket.class)使用
                sellTicket();
            }
            x++;
        }
    }

    /**

     * 同步代码块

     */

    private void sellTicketSynchronized() {
        // 同步代码块的锁对象是this，所以只能配合非静态同步方法使用
        // synchronized (this) {
        // 同步代码块的锁对象是类名.class，所以只能配合静态同步方法使用
        synchronized (SellTicket.class) {
            if (tickets > 0) {
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "正在出售第" + tickets + "张票");
                tickets--;
            }
        }
    }

     /**
     * 非静态同步方法
     */

    private synchronized void sellTicketNotStatic() {
        if (tickets > 0) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "正在出售第" + tickets + "张票");
            tickets--;
        }
    }


    /**
     * 静态同步方法
     */

    private static synchronized void sellTicket() {
        if (tickets > 0) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "正在出售第" + tickets + "张票");
            tickets--;
        }
    }
}
```

`SellTicketDemo`类：

```

public class SellTicketDemo {
    public static void main(String[] args) {
        // 创建SellTicket类的对象
        SellTicket st = new SellTicket();

        // 创建三个Thread类的对象，把SellTicket对象作为构造方法的参数，并给出对应的窗口名称
        Thread t1 = new Thread(st,"窗口1");
        Thread t2 = new Thread(st,"窗口2");
        Thread t3 = new Thread(st,"窗口3");

        // 启动线程
        t1.start();
        t2.start();
        t3.start();
    }
}
```

运行结果：

```
窗口1正在出售第100张票
窗口1正在出售第99张票
窗口1正在出售第98张票
...
窗口2正在出售第3张票
窗口2正在出售第2张票
窗口2正在出售第1张票
```

注：上面实例中，非静态同步方法，锁对象是`this`，所以需要配合同步代码块的`synchronized (this) `使用；静态同步方法，锁对象是当前函数所属的类的字节码文件`类名.class`，所以需要配合同步代码块的`synchronized (SellTicket.class)`使用。

# Lock锁

虽然我们可以理解同步代码块和同步方法的锁对象问题，但是我们并没有直接看到在哪里加上了锁，在哪里释放了锁，为了更清晰的表达如何加锁和释放锁。`JDK5`以后提供了一个新的锁`Lock`。`Lock`是接口不能直接实例化，`Lock` 接口主要有 `ReentrantLock`，`ReentrantReadWriteLock.ReadLock`，`ReentrantReadWriteLock.WriteLock `实现类，这里采用它的实现类`ReentrantLock`来实例化， 与`Synchronized`不同是 `Lock` 提供了获取锁和释放锁等相关接口，使得使用上更加灵活，同时也可以做更加复杂的操作。

接下来，我们采用 `Lock` 的方式实现`SellTicket`类：

```
public class SellTicket implements Runnable {
    private int tickets = 100;
    // 创建一个ReentrantLock的实例
    private Lock lock = new ReentrantLock();

    @Override
    public void run() {
        while (true) {
            try {
                // 获取锁
                lock.lock();
                if (tickets > 0) {
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + "正在出售第" + tickets + "张票");
                    tickets--;
                }
            } finally {
                // 释放锁
                lock.unlock();
            }
        }
    }
}
```

运行结果：

```
窗口1正在出售第100张票
窗口1正在出售第99张票
窗口2正在出售第98张票
窗口3正在出售第97张票
...
窗口1正在出售第2张票
窗口1正在出售第1张票
```


# 总结

> 为什么会出现多线程安全？

在多个线程并发环境下，多个线程共同访问同一共享内存资源时，其中一个线程对资源进行写操作的中途(写⼊已经开始，但还没结束)，其他线程对这个写了一半的资源进⾏了读操作，或者对这个写了一半的资源进⾏了写操作，导致此资源出现数据错误。

> 线程安全在三个方面体现?


* 原子性：提供互斥访问，同一时刻只能有一个线程对数据进行操作。
* 可见性：一个线程对主内存的修改可以及时地被其他线程看到。
* 有序性：一个线程观察其他线程中的指令执行顺序，由于指令重排序，该观察结果一般杂乱无序。

> 如何避免线程安全问题？

* `Java`中提供了`Synchronized` 关键字，通过保证方法或代码块操作的原子性来实现线程同步。其原理是在同⼀时间、由同⼀个 `Monitor`（监视锁） 监视的代码，最多只能有⼀个线程在访问。实质是保证共享资源在同一时间只能由一个线程进行操作(原子性，有序性)。
* 将线程操作的结果及时刷新，保证其他线程可以立即获取到修改后的最新数据（可见性）。

