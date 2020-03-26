---
title: java线程
date: 2020-03-18 10:26:36
tags: java 
---


# 线程和进程
一个程序就是一个进程，而一个程序中的多个任务则被称为线程。
进程是表示资源分配的基本单位，线程是进程中执行运算的最小单位，亦是调度运行的基本单位。

# 线程基础

## 如何创建一个线程
在创建一个线程时，最好为这个线程设置线程名称，因为这样在使用jstack分析程序或者进行问题排查时，就会给开发人员提供一些提示
1，直接继承Tread类，重写run()。
2，实现Runnable结构，实现run()方法。
3，实现Callable接口。通过FutureTask包装器来创建thread线程。
![](/img/java/线程/new_callable_thread.png)
4，ThreadPool。

## 线程的生命周期  
线程一共有 6 种状态(NEW、RUNNABLE、BLOCKED、 WAITING、TIME_WAITING、TERMINATED)
- NEW:初始状态，线程被构建，但是还没有调用 start 方法。
- RUNNABLE:运行状态，JAVA 线程把操作系统中的就绪 和运行两种状态统一称为“运行中”。
- BLOCKED:阻塞状态，表示线程进入等待状态,也就是线程 因为某种原因放弃了 CPU 使用权，阻塞也分为几种情况。<br>
 ➢等待阻塞:运行的线程执行 wait 方法，jvm 会把当前 线程放入到等待队列。 <br>
 ➢同步阻塞:运行的线程在获取对象的同步锁时，若该同 步锁被其他线程锁占用了，那么 jvm 会把当前的线程 放入到锁池中。<br>
 ➢其他阻塞:运行的线程执行 Thread.sleep 或者 t.join 方 法，或者发出了 I/O 请求时，JVM 会把当前线程设置 为阻塞状态，当 sleep 结束、join 线程终止、io 处理完 毕则线程恢复。
- WAITING 都是等待状态，与TIME_WAITING的区别只是不会超时。
- TIME_WAITING:超时等待状态，超时以后自动返回。
- TERMINATED:终止状态，表示当前线程执行完毕。

## 线程的启动
调用start()方法去启动一个线程，当run方法中的代码执行完毕以后，线程的生命周期也将终止，调用start方法的语义是当前线程告诉JVM，启动调用start方法的线程。<br>
线程启动的原理，在Thread方法中，底层实际上个是调用了start0()的一个方法，，这个方法是在这个方法native修饰的一个底层方法

## 线程的终止
线程的终止，并不是简单的调用 stop 命令去。虽然api仍然可以调用，但是和其他的线程控制方法如 suspend、 resume 一样都是过期了的不建议使用，就拿stop来说，stop方法在结束一个线程时并不会保证线程的资源正常释放，因此会导致程序可能出现一些不确定的状态。要优雅的去中断一个线程，在线程中提供了一个 interrupt 方法。</br>

### interrupt 方法 
> 当其他线程通过调用当前线程的 interrupt 方法，表示向当前线程打个招呼，告诉他可以中断线程的执行了，至于什 么时候中断，取决于当前线程自己。 线程通过检查自身是否被中断来进行相应，可以通过 isInterrupted()来判断是否被中断。 
![](/img/java/线程/Interrupt_stop_thread.jpg)

### Thread.interrupted()
线程中还提供了静态方法 Thread.interrupted()对设置中断标识的线程复位。
![](/img/java/线程/interrupted_thread.jpg)

### 其他的线程复位
除了通过 Thread.interrupted 方法对线程中断标识进行复 位以外，还有一种被动复位的场景，就是对抛出 InterruptedException 异 常 的 方 法 ，在 InterruptedException 抛出之前，JVM 会先把线程的中断 标识位清除，然后才会抛出 InterruptedException，这个时 候如果调用 isInterrupted方法，将会返回false。

### 为什么要复位
Thread.interrupted()是属于当前线程的，是当前线程对外 界中断信号的一个响应，表示自己已经得到了中断信号， 但不会立刻中断自己，具体什么时候中断由自己决定，让 外界知道在自身中断前，他的中断状态仍然是 false，这就 是复位的原因。
需要注意的是，InterruptedException 异常的抛出并不意味 着线程必须终止，而是提醒当前线程有中断的操作发生， 至于接下来怎么处理取决于线程本身，比如
1. 直接捕获异常不做任何处理
2. 将异常往外抛出
3. 停止当前线程，并打印异常信息



# 线程安全
> 一个对象是否是线程安全的，取决于它是否会被多个线程 访问，以及程序中是如何去使用这个对象的。所以，如果 多个线程访问同一个共享对象，在不需额外的同步以及调 用端代码不用做其他协调的情况下，这个共享对象的状态 依然是正确的(正确性意味着这个对象的结果与我们预期 规定的结果保持一致)，那说明这个对象是线程安全的。

## 理解锁的基础知识

### 基础知识之一：锁的类型
锁从宏观上分类，分为悲观锁与乐观锁。

乐观锁
乐观锁是一种乐观思想，即认为读多写少，遇到并发写的可能性低，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，采取在写时先读出当前版本号，然后加锁操作（比较跟上一次的版本号，如果一样则更新），如果失败则要重复读-比较-写的操作。

java中的乐观锁基本都是通过CAS操作实现的，CAS是一种更新的原子操作，比较当前值跟传入值是否一样，一样则更新，否则失败。

悲观锁
悲观锁是就是悲观思想，即认为写多，遇到并发写的可能性高，每次去拿数据的时候都认为别人会修改，所以每次在读写数据的时候都会上锁，这样别人想读写这个数据就会block直到拿到锁。java中的悲观锁就是Synchronized,AQS框架下的锁则是先尝试cas乐观锁去获取锁，获取不到，才会转换为悲观锁，如RetreenLock。

### 基础知识之二：java线程阻塞的代价
java的线程是映射到操作系统原生线程之上的，如果要阻塞或唤醒一个线程就需要操作系统介入，需要在户态与核心态之间切换，这种切换会消耗大量的系统资源，因为用户态与内核态都有各自专用的内存空间，专用的寄存器等，用户态切换至内核态需要传递给许多变量、参数给内核，内核也需要保护好用户态在切换时的一些寄存器值、变量等，以便内核态调用结束后切换回用户态继续工作。

如果线程状态切换是一个高频操作时，这将会消耗很多CPU处理时间；
如果对于那些需要同步的简单的代码块，获取锁挂起操作消耗的时间比用户代码执行的时间还要长，这种同步策略显然非常糟糕的。
synchronized会导致争用不到锁的线程进入阻塞状态，所以说它是java语言中一个重量级的同步操纵，被称为重量级锁，为了缓解上述性能问题，JVM从1.5开始，引入了轻量锁与偏向锁，默认启用了自旋锁，他们都属于乐观锁。

明确java线程切换的代价，是理解java中各种锁的优缺点的基础之一。

在介绍java锁之前，先说下什么是markword，markword是java对象数据结构中的一部分，要详细了解java对象的结构可以点击这里,这里只做markword的详细介绍，因为对象的markword和java各种类型的锁密切相关；

markword数据的长度在32位和64位的虚拟机（未开启压缩指针）中分别为32bit和64bit，它的最后2bit是锁状态标志位，用来标记当前对象的状态，对象的所处的状态，决定了markword存储的内容，如下表所示:
![](/img/java/线程/markword_detail.jpg)

32位虚拟机在不同状态下markword结构如下图所示：
![](/img/java/线程/markword_structure.jpg)

### 基础知识之三：markword


## synchronized 的基本认识
在多线程并发编程中 synchronized 一直是元老级角色，很 多人都会称呼它为重量级锁。但是，随着 Java SE 1.6 对 synchronized 进行了各种优化之后，有些情况下它就并不 那么重，Java SE 1.6 中为了减少获得锁和释放锁带来的性 能消耗而引入的偏向锁和轻量级锁。

## synchronized 的基本语法
synchronized 有三种方式来加锁，分别是
1. 修饰实例方法，作用于当前实例加锁，进入同步代码前要获得当前实例的锁
2. 静态方法，作用于当前类对象加锁，进入同步代码前要获得当前类对象的锁
3. 修饰代码块，指定加锁对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁。不同的修饰类型，代表锁的控制粒度
![](/img/java/线程/synchronized_use.png)

## 锁是如何存储的

![](/img/java/线程/synchronized_save.jpeg)


为什么任何对象都可以实现锁
1. 首先，Java 中的每个对象都派生自 Object 类，而每个 Java Object 在 JVM 内部都有一个 native 的 C++对象 oop/oopDesc 进行对应。
2. 线程在获取锁的时候，实际上就是获得一个监视器对象
(monitor) ,monitor 可以认为是一个同步对象，所有的 Java 对象是天生携带 monitor。
多个线程访问同步代码块时，相当于去争抢对象监视器 修改对象中的锁标识


## synchronized锁的升级
在分析 markword 时，提到了偏向锁、轻量级锁、重量级 锁。在分析这几种锁的区别时，我们先来思考一个问题 使用锁能够实现数据的安全性，但是会带来性能的下降。 不使用锁能够基于线程并行提升程序性能，但是却不能保 证线程安全性。这两者之间似乎是没有办法达到既能满足 性能也能满足安全性的要求。
hotspot 虚拟机的作者经过调查发现，大部分情况下，加锁 的代码不仅仅不存在多线程竞争，而且总是由同一个线程 多次获得。所以基于这样一个概率，是的 synchronized 在 JDK1.6 之后做了一些优化，为了减少获得锁和释放锁带来 的性能开销，引入了偏向锁、轻量级锁的概念。因此大家 会发现在 synchronized 中，锁存在四种状态 分别是:无锁、偏向锁、轻量级锁、重量级锁; 锁的状态 根据竞争激烈的程度从低到高不断升级。

### 偏向锁
偏向锁的基本原理
> 前面说过，大部分情况下，锁不仅仅不存在多线程竞争， 而是总是由同一个线程多次获得，为了让线程获取锁的代 价更低就引入了偏向锁的概念。怎么理解偏向锁呢? 当一个线程访问加了同步锁的代码块时，会在对象头中存 储当前线程的 ID，后续这个线程进入和退出这段加了同步 锁的代码块时，不需要再次加锁和释放锁。而是直接比较 对象头里面是否存储了指向当前线程的偏向锁。如果相等 表示偏向锁是偏向于当前线程的，就不需要再尝试获得锁了。

偏向锁的获取
1. 首先获取锁 对象的 Markword，判断是否处于可偏向状 态。(biased_lock=1、且 ThreadId 为空)
2. 如果是可偏向状态，则通过 CAS 操作，把当前线程的 ID 写入到 MarkWord
    
    a) 如果 cas 成功，那么 markword 就会变成这样。 表示已经获得了锁对象的偏向锁，接着执行同步代码块

    b) 如果 cas 失败，说明有其他线程已经获得了偏向锁， 这种情况说明当前锁存在竞争，需要撤销已获得偏向锁的线程，并且把它持有的锁升级为轻量级锁(这个 操作需要等到全局安全点，也就是没有线程在执行字节码)才能执行
3. 如果是已偏向状态，需要检查 markword 中存储的 ThreadID 是否等于当前线程 ThreadID
    a) 如果相等，不需要再次获得锁，可直接执行同步代码块.

    b) 如果不相等，说明当前锁偏向于其他线程，需要撤销 偏向锁并升级到轻量级锁.

偏向锁的撤销

偏向锁的撤销并不是把对象恢复到无锁可偏向状态(因为 偏向锁并不存在锁释放的概念)，而是在获取偏向锁的过程 中，发现 cas 失败也就是存在线程竞争时，直接把被偏向 的锁对象升级到被加了轻量级锁的状态。
对原持有偏向锁的线程进行撤销时，原获得偏向锁的线程 有两种情况:
1. 原获得偏向锁的线程如果已经退出了临界区，也就是同步代码块执行完了，那么这个时候会把对象头设置成无 锁状态并且争抢锁的线程可以基于 CAS 重新偏向当前线程。
2. 如果原获得偏向锁的线程的同步代码块还没执行完，处于临界区之内，这个时候会把原获得偏向锁的线程升级为轻量级锁后继续执行同步代码块。

偏向锁流程图分析：
![](/img/java/线程/pianxiangsuo_liucheng.jpg)



在我们的应用开发中，绝大部分情况下一定会存在2个以上的线程竞争，那么如果开启偏向锁，反而会提升获取锁的资源消耗。所以可以通过 jvm 参数 seBiasedLocking 来设置开启或关闭偏向锁


### 轻量锁
锁升级为轻量级锁之后，对象的 Markword 也会进行相应 的的变化。升级为轻量级锁的过程:
1. 线程在自己的栈桢中创建锁记录 LockRecord。
2. 将锁对象的对象头中的 MarkWord 复制到线程的刚刚创建的锁记录中。
3. 将锁记录中的 Owner 指针指向锁对象。
4. 将锁对象的对象头的 MarkWord 替换为指向锁记录的指针。
 

### 自旋锁
轻量级锁在加锁过程中，用到了自旋锁。所谓自旋，就是指当有另外一个线程来竞争锁时，这个线 程会在原地循环等待，而不是把该线程给阻塞，直到那个获得锁的线程释放锁之后，这个线程就可以马上获得锁的。 注意，锁在原地循环的时候，是会消耗 cpu 的，就相当于在执行一个啥也没有的 for 循环。所以，轻量级锁适用于那些同步代码块执行的很快的场景， 这样，线程原地等待很短的时间就能够获得锁了。 自旋锁的使用，其实也是有一定的概率背景，在大部分同 步代码块执行的时间都是很短的。所以通过看似无意义的循环反而能提升锁的性能。 但是自旋必须要有一定的条件控制，否则如果一个线程执行同步代码块的时间很长，那么这个线程不断的循环反而会消耗 CPU 资源。默认情况下自旋的次数是 10 次， 可以通过 preBlockSpin 来修改。


在 JDK1.6 之后，引入了自适应自旋锁，自适应意味着自旋 的次数不是固定不变的，而是根据前一次在同一个锁上自旋的时间以及锁的拥有者的状态来决定。 如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也是很有可能再次成功，进而它将允许自旋等待持续相对更长的时间。如果对于某个锁，自旋很少成功获得过， 那在以后尝试获取这个锁时将可能省略掉自旋过程，直接阻塞线程，避免浪费处理器资源


轻量级锁的解锁
轻量级锁的锁释放逻辑其实就是获得锁的逆向逻辑，通过 CAS 操作把线程栈帧中的 LockRecord 替换回到锁对象的 MarkWord 中，如果成功表示没有竞争。如果失败，表示当前锁存在竞争，那么轻量级锁就会膨胀成为重量级锁

流程图如下图：
![](/img/java/线程/qingliangjisuo_liucheng.jpg)


### 重量级锁的基本原理
当轻量级锁膨胀到重量级锁之后，意味着线程只能被挂起 阻塞来等待被唤醒了。

#### 重量级锁的 monitor
加了同步代码块以后，在字节码中会看到一个 monitorenter 和 monitorexit。
每一个 JAVA 对象都会与一个监视器 monitor 关联，我们 可以把它理解成为一把锁，当一个线程想要执行一段被 synchronized 修饰的同步方法或者代码块时，该线程得先 获取到 synchronized 修饰的对象对应的 monitor。
monitorenter 表示去获得一个对象监视器。monitorexit 表 示释放 monitor 监视器的所有权，使得其他被阻塞的线程 可以尝试去获得这个监视器。
monitor 依赖操作系统的 MutexLock(互斥锁)来实现的, 线程被阻塞后便进入内核(Linux)调度状态，这个会导致系统在用户态与内核态之间来回切换，严重影响锁的性能。

#### 重量级锁的加锁的基本流程
任意线程对 Object(Object 由 synchronized 保护)的访问，首先要获得 Object 的监视器。如果获取失败，线程进 入同步队列，线程状态变为 BLOCKED。当访问 Object 的前驱(获得了锁的线程)释放了锁，则该释放操作唤醒阻 塞在同步队列中的线程，使其重新尝试对监视器的获取。

重量级锁的流程：

![](/img/java/线程/zhongliangsuo_liucheng.jpg)

### 整体梳理线程的竞争机制
再来回顾一下线程的竞争机制对于锁升级这块的一些基本 流程。方便更好的理解 加入有这样一个同步代码块，存在 Thread#1、Thread#2 等多个线程

``` java
synchronized (lock) { 
    // do something
}
```
- 情况一:只有 Thread#1 会进入临界区; 
- 情况二:Thread#1 和 Thread#2 交替进入临界区,竞争不激烈;
- 情况三:Thread#1/Thread#2/Thread3... 同时进入临界区，竞争激烈

偏向锁

此时当 Thread#1 进入临界区时，JVM 会将 lockObject 的 对象头 Mark Word 的锁标志位设为“01”，同时会用 CAS 操作把 Thread#1 的线程 ID 记录到 Mark Word 中，此时进 入偏向模式。所谓“偏向”，指的是这个锁会偏向于 Thread#1， 若接下来没有其他线程进入临界区，则 Thread#1 再出入 临界区无需再执行任何同步操作。也就是说，若只有 Thread#1 会进入临界区，实际上只有 Thread#1 初次进入 临界区时需要执行 CAS 操作，以后再出入临界区都不会有 同步操作带来的开销。

轻量级锁

偏向锁的场景太过于理想化，更多的时候是 Thread#2 也 会尝试进入临界区， 如果 Thread#2 也进入临界区但是 Thread#1 还没有执行完同步代码块时，会暂停 Thread#1 并且升级到轻量级锁。Thread#2 通过自旋再次尝试以轻量 级锁的方式来获取锁

重量级锁

如果 Thread#1 和 Thread#2 正常交替执行，那么轻量级锁 基本能够满足锁的需求。但是如果 Thread#1 和 Thread#2 同时进入临界区，那么轻量级锁就会膨胀为重量级锁，意 味着 Thread#1 线程获得了重量级锁的情况下，Thread#2 就会被阻塞
 

## Synchronized 结合 Java Object 对象中的 wait,notify,notifyAll
被阻塞的线程什么时候被唤醒，取决于获得锁的线程什么时候执行完同步代码块并且释放锁。那怎么做到显示控制呢?我们就需要 借助一个信号机制: 在 Object 对象中，提供了 wait/notify/notifyall，可以用于控制线程的状态。

### wait/notify/notifyall 基本概念

> wait:表示持有对象锁的线程 A 准备释放对象锁权限，释放 cpu 资源并进入等待状态。 

> notify:表示持有对象锁的线程 A 准备释放对象锁权限，通 知 jvm 唤醒某个竞争该对象锁的线程 X。线程 A synchronized 代码执行结束并且释放了锁之后，线程 X 直 接获得对象锁权限，其他竞争线程继续等待(即使线程 X 同 步完毕，释放对象锁，其他竞争线程仍然等待，直至有新 的 notify ,notifyAll 被调用)。

> notifyAll:notifyall 和 notify 的区别在于，notifyAll 会唤醒 所有竞争同一个对象锁的所有线程，当已经获得锁的线程 A 释放锁之后，所有被唤醒的线程都有可能获得对象锁权 限

<b>需要注意的是:三个方法都必须在 synchronized 同步关键 字所限定的作用域中调用，否则会报错 java.lang.IllegalMonitorStateException ，意思是因为没有同步，所以线程对对象锁的状态是不确定的，不能调用这 些方法。

另外，通过同步机制来确保线程从 wait 方法返回时能够感知到感知到 notify 线程对变量做出的修改</b>

### wait/notify 的基本使用

``` java
public class ThreadA extends Thread{
    private Object lock;
    public ThreadA(Object lock) {
        this.lock = lock;
    }
    @Override
    public void run() {
        synchronized (lock){
            System.out.println("start ThreadA");
            try {
                lock.wait(); //实现线程的阻塞
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("end ThreadA");
        }
    }
}

public class ThreadB extends Thread{
    private Object lock=new Object();
    public ThreadB(Object lock) {
        this.lock = lock;
    }
    @Override
    public void run() {
        synchronized (lock){
            System.out.println("start ThreadB");
            lock.notify(); //唤醒被阻塞的线程
            System.out.println("end ThreadB");
        }
    }
}

public class WaitNotifyDemo {
    public static void main(String[] args) {
        Object lock=new Object();
        ThreadA threadA=new ThreadA(lock);
        threadA.start();
        ThreadB threadB=new ThreadB(lock);
        threadB.start();
    }
}


```

### wait/notify 的基本原理
![](/img/java/线程/wait_notify_yuanli.jpg)

 

# Volatile
volatile 可以使得在多处理器环境下保证了共享变量的可见性。

在单线程的环境下，如果向一个变量先写入一个值，然后在没有写干涉的情况下读取这个变量的值，那这个时候读取到的这个变量的值应该是之前写入的那个值。这本来是一个很正常的事情。但是在多线程环境下，读和写发生在不同的线程中的时候，可能会出现:读线程不能及时的读取到其他线程写入的最新的值。这就是所谓的可见性为了实现跨线程写入的内存可见性，必须使用到一些机制 来实现。而 volatile 就是这样一种机制。

``` java
public class VolatileDemo {
    public /*volatile*/ static boolean stop = false;
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(()->{
            int i = 0;
            while (!stop){
                i++;
            }
        });
        thread.start();
        System.out.println("begin start thread");
        Thread.sleep(1000);
        stop = true;
    }
}
```

# Thread.join 
``` java
public class JoinTest {
    public static void main(String [] args) throws InterruptedException {
        ThreadJoinTest t1 = new ThreadJoinTest("小明");
        ThreadJoinTest t2 = new ThreadJoinTest("小东");
        t1.start();
        /**join的意思是使得放弃当前线程的执行，并返回对应的线程，例如下面代码的意思就是：
         程序在main线程中调用t1线程的join方法，则main线程放弃cpu控制权，并返回t1线程继续执行直到线程t1执行完毕
         所以结果是t1线程执行完后，才到主线程执行，相当于在main线程中同步t1线程，t1执行完了，main线程才有执行的机会
         */
        t1.join();
        t2.start();
    }
}
class ThreadJoinTest extends Thread{
    public ThreadJoinTest(String name){
        super(name);
    }
    @Override
    public void run(){
        for(int i=0;i<1000;i++){
            System.out.println(this.getName() + ":" + i);
        }
    }
}
```

# JMM

# Lock
Java.util.concurrent 是在并发编程中比较常用的工具类，里面包含很多用来在并发 场景中使用的组件。比如线程池、阻塞队列、计时器、同步器、并发集合等等。并 发包的作者是大名鼎鼎的 Doug Lea。
## Lock的基本使用以及原理分析


### Lock 简介 
在 Lock 接口出现之前，Java 中的应用程序对于多线程的并发安全处理只能基于 synchronized 关键字来解决。但是 synchronized 在有些场景中会存在一些短板，就是它并不适合于所有的并发场景。但是在 Java5 以后，Lock 的出现可以解决 synchronized 在某些场景中的短板，它比 synchronized 更加灵活。

### Lock 的实现

Lock 本质上是一个接口，它定义了释放锁和获得锁的抽象方法，定义成接口就意 味着它定义了锁的一个标准规范，也同时意味着锁的不同实现。实现 Lock 接口的
类有很多，以下为几个常见的锁实现:

- ReentrantLock:表示重入锁，它是唯一一个实现了 Lock 接口的类。重入锁指的是线程在获得锁之后，再次获取该锁不需要阻塞，而是直接关联一次计数器增加重入次数
- ReentrantReadWriteLock:重入读写锁，它实现了 ReadWriteLock 接口，在这个类中维护了两个锁，一个是 ReadLock，一个是 WriteLock，他们都分别实现了 Lock 接口。读写锁是一种适合读多写少的场景下解决线程安全问题的工具，基本原则 是: 读和读不互斥、读和写互斥、写和写互斥。也就是说涉及到影响数据变化的 操作都会存在互斥。
- StampedLock: stampedLock 是 JDK8 引入的新的锁机制，可以简单认为是读写锁的一个改进版本，读写锁虽然通过分离读和写的功能使得读和读之间可以完全并发，但是读和写是有冲突的，如果大量的读线程存在，可能会引起写线程的饥饿。 stampedLock 是一种乐观的读策略，使得乐观锁完全不会阻塞写线程。

Lock的类关系图
Lock 有很多的锁的实现，但是直观的实现是 ReentrantLock 重入锁
![](/img/java/线程/lock_shixianlei.jpg)

#### ReentrantLock 重入锁

重入锁，表示支持重新进入的锁，也就是说，如果当前线程 t1 通过调用 lock 方 法获取了锁之后，再次调用 lock，是不会再阻塞去获取锁的，直接增加重试次数 就行了。synchronized 和 ReentrantLock 都是可重入锁。

重入锁的设计目的
比如调用 demo 方法获得了当前的对象锁，然后在这个方法中再去调用 demo2，demo2 中的存在同一个实例锁，这个时候当前线程会因为无法获得 demo2 的对象锁而阻塞，就会产生死锁。重入锁的设计目的是避免线程的死锁。

``` java
public class ReentrantDemo{
    public synchronized void demo(){
        System.out.println("begin:demo");
        demo2();
    }
    public void demo2(){
        System.out.println("begin:demo1");
        synchronized (this){
        }
    }
    public static void main(String[] args) {
        ReentrantDemo rd=new ReentrantDemo();
        new Thread(rd::demo).start();
    }
}
```


ReentrantLock 的实现原理

我们知道锁的基本原理是，基于将多线程并行任务通过某一种机制实现线程的串行执行，从而达到线程安全性的目的。在 synchronized 中，我们分析了偏向锁、 轻量级锁、乐观锁。基于乐观锁以及自旋锁来优化了 synchronized 的加锁开销， 同时在重量级锁阶段，通过线程的阻塞以及唤醒来达到线程竞争和同步的目的。 那么在 ReentrantLock 中，也一定会存在这样的需要去解决的问题。就是在多线程竞争重入锁时，竞争失败的线程是如何实现阻塞以及被唤醒的呢?

AQS 是什么

在 Lock 中，用到了一个同步队列 AQS，全称 AbstractQueuedSynchronizer，它 是一个同步工具也是 Lock 用来实现线程同步的核心组件。如果你搞懂了 AQS，那 么 J.U.C 中绝大部分的工具都能轻松掌握。


AQS 的两种功能
从使用层面来说，AQS 的功能分为两种:独占和共享
> 独占锁，每次只能有一个线程持有锁，比如前面给大家演示的 ReentrantLock 就是以独占方式实现的互斥锁

> 共享锁，允许多个线程同时获取锁，并发访问共享资源，比如 ReentrantReadWriteLock


AQS 的内部实现 

AQS 队列内部维护的是一个 FIFO 的双向链表，这种结构的特点是每个数据结构

都有两个指针，分别指向直接的后继节点和直接前驱节点。所以双向链表可以从任意一个节点开始很方便的访问前驱和后继。每个 Node 其实是由线程封装，当线 程争抢锁失败后会封装成 Node 加入到 ASQ 队列中去;当获取锁的线程释放锁以 后，会从队列中唤醒一个阻塞的节点(线程)。





ReentrantLock 的源码分析

以 ReentrantLock 作为切入点，来看看在这个场景中是如何使用 AQS 来实现线程 的同步的

ReentrantLock 的时序图

调用 ReentrantLock 中的 lock()方法，源码的调用过程我使用了时序图来展现。
![](/img/java/线程/reentrantlock_shixutu.jpg)

ReentrantLock.lock()
这个是 reentrantLock 获取锁的入口
``` java
public void lock() {
     sync.lock();
}
```
sync 实际上是一个抽象的静态内部类，它继承了 AQS 来实现重入锁的逻辑，我们前面说过 AQS 是一个同步队列，它能够实现线程的阻塞以及唤醒，但它并不具备业务功能，所以在不同的同步场景中，会继承 AQS 来实现对应场景的功能
Sync 有两个具体的实现类，分别是: 
> NofairSync:表示可以存在抢占锁的功能，也就是说不管当前队列上是否存在其他线程等待，新线程都有机会抢占锁
> FailSync: 表示所有线程严格按照 FIFO 来获取锁


NofairSync.lock
以非公平锁为例，来看看 lock 中的实现
1. 非公平锁和公平锁最大的区别在于，在非公平锁中我抢占锁的逻辑是，不管有
没有线程排队，我先上来 cas 去抢占一下 
2. CAS 成功，就表示成功获得了锁
3. CAS 失败，调用 acquire(1)走锁竞争逻辑


#### ReentrantReadWriteLock

我们以前理解的锁，基本都是排他锁，也就是这些锁在同一时刻只允许一个线程进 行访问，而读写所在同一时刻可以允许多个线程访问，但是在写线程访问时，所有 的读线程和其他写线程都会被阻塞。读写锁维护了一对锁，一个读锁、一个写锁; 一般情况下，读写锁的性能都会比排它锁好，因为大多数场景读是多于写的。在读 多于写的情况下，读写锁能够提供比排它锁更好的并发性和吞吐量。




### Lock阻塞以及唤醒
Condition

Condition 是一个多线程协调通信的工具类，可以让某些线 程一起等待某个条件(condition)，只有满足条件时，线程 才会被唤醒

``` java
public class ConditionWait implements Runnable{
    private Lock lock;
    private Condition condition;
    public ConditionWait(Lock lock, Condition condition) {
        this.lock = lock;
        this.condition = condition;
    }
    @Override
    public void run() {
        try {
            lock.lock(); //竞争锁
            try {
                System.out.println("begin - ConditionWait");
                condition.await();//阻塞(1. 释放锁, 2.阻塞当前线程, FIFO（单向、双向）)
                System.out.println("end - ConditionWait");

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }finally {
            lock.unlock();//释放锁
        }
    }
}

public class ConditionNotify implements Runnable{
    private Lock lock;
    private Condition condition;
    public ConditionNotify(Lock lock, Condition condition) {
        this.lock = lock;
        this.condition = condition;
    }
    @Override
    public void run() {
        try{
            lock.lock();//获得了锁.
            System.out.println("begin - conditionNotify");
            condition.signal();//唤醒阻塞状态的线程
            // condition.await();
            System.out.println("end - conditionNotify");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock(); //释放锁
        }
    }
}

public class App{
    public static void main( String[] args ){
        Lock lock=new ReentrantLock(); //重入锁
        Condition condition=lock.newCondition();
        new Thread(new ConditionWait(lock,condition)).start();// 阻塞await
        new Thread(new ConditionNotify(lock,condition)).start();
    }
}

```


# 线程池
在 Java 中，如果每个请求到达就创建一个新线程，创建和销毁线程花费的时间和消耗的系统 资源都相当大，甚至可能要比在处理实际的用户请求的时间和资源要多的多。
如果在一个 Jvm 里创建太多的线程，可能会使系统由于过度消耗内存或“切换过度”而导致系 统资源不足 为了解决这个问题,就有了线程池的概念.
## 什么是线程池
线程池的核心逻辑是提前创建好若干个线程放在一 个容器中。如果有任务需要处理，则将任务直接分配给线程池中的线程来执行就行，任务处 理完以后这个线程不会被销毁，而是等待后续分配任务。同时通过线程池来重复管理线程还 可以避免创建大量线程增加开销。

合理的使用线程池，可以带来一些好处
1. 降低创建线程和销毁线程的性能开销
2. 提高响应速度，当有新任务需要执行是不需要等待线程创建就可以立马执行 3. 合理的设置线程池大小可以避免因为线程数超过硬件资源瓶颈带来的问题

## java中提供的线程池
Executors 里面提供了几个线程池的工厂方法，
- newFixedThreadPool:该方法返回一个固定数量的线程池，线程数不变，当有一个任务提交 时，若线程池中空闲，则立即执行，若没有，则会被暂缓在一个任务队列中，等待有空闲的 线程去执行。
- newSingleThreadExecutor: 创建一个线程的线程池，若空闲则执行，若没有空闲线程则暂缓 在任务队列中。
- newCachedThreadPool:返回一个可根据实际情况调整线程个数的线程池，不限制最大线程 数量，若用空闲的线程则执行任务，若无任务则不创建线程。并且每一个空闲线程会在 60 秒 后自动回收
- newScheduledThreadPool: 创建一个可以指定线程的数量的线程池，但是这个线程池还带有 延迟和周期性执行任务的功能，类似定时器。

ThreadpoolExecutor
上面提到的四种线程池的构建，都是基于 ThreadpoolExecutor 来构建的

``` java
public ThreadPoolExecutor(int corePoolSize, //核心线程数量
    int maximumPoolSize, //最大线程数
    long keepAliveTime, //超时时间,超出核心线程数量以外的线程空余存活时间
    TimeUnit unit, //存活时间单位
    BlockingQueue<Runnable> workQueue, //保存执行任务的队列
    ThreadFactory threadFactory,//创建新线程使用的工厂
    RejectedExecutionHandler handler //当任务无法执行的时候的处理方式
)
```

线程池初始化时是没有创建线程的，线程池里的线程的初始化与其他线程一样，但是在完成 任务以后，该线程不会自行销毁，而是以挂起的状态返回到线程池。直到应用程序再次向线 程池发出请求时，线程池里挂起的线程就会再度激活执行任务。这样既节省了建立线程所造 成的性能损耗，也可以让多个任务反复重用同一线程，从而在应用程序生存期内节约大量开 销

### newFixedThreadPool
``` java
public static ExecutorService newFixedThreadPool(int nThreads) { 
    return new ThreadPoolExecutor(nThreads, nThreads,
                    0L, TimeUnit.MILLISECONDS,
                    new LinkedBlockingQueue<Runnable>());
}

```

FixedThreadPool 的核心线程数和最大线程数都是指定值，也就是说当线程池中的线程数超 过核心线程数后，任务都会被放到阻塞队列中。另外 keepAliveTime 为 0，也就是超出核心 线程数量以外的线程空余存活时间
而这里选用的阻塞队列是 LinkedBlockingQueue，使用的是默认容量 Integer.MAX_VALUE， 相当于没有上限

这个线程池执行任务的流程如下:
1. 线程数少于核心线程数，也就是设置的线程数时，新建线程执行任务
2. 线程数等于核心线程数后，将任务加入阻塞队列
3. 由于队列容量非常大，可以一直添加
4. 执行完任务的线程反复去队列中取任务执行

用途:FixedThreadPool 用于负载比较大的服务器，为了资源的合理利用，需要限制当前线 程数量

### newSingleThreadExecutor
创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定 顺序(FIFO, LIFO, 优先级)执行


### newCachedThreadPool

``` java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
        60L, TimeUnit.SECONDS,
        new SynchronousQueue<Runnable>());
}
```

CachedThreadPool 创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空 闲线程，若无可回收，则新建线程; 并且没有核心线程，非核心线程数无上限，但是每个空闲 的时间只有 60 秒，超过后就会被回收。
它的执行流程如下:
1. 没有核心线程，直接向 SynchronousQueue 中提交任务
2. 如果有空闲线程，就去取出任务执行;如果没有空闲线程，就新建一个
3. 执行完任务的线程有 60 秒生存时间，如果在这个时间内可以接到新任务，就可以继续活下去，否则就被回收


### newScheduledThreadPool
ScheduledThreadPoolExecutor 继承了 ThreadPoolExecutor，并另外提供一些调度方法以支 持定时和周期任务。

## 线程池的实现原理



# 线程问题如何排查









# 常见的面试题
## 如何保证多个线程串行执行
thread.join
## 线程池的实现原理

## sleep和wait的区别

## Synchronized 和Lock的区别

## 如何有效避免死锁
死锁产生的原因：
两个或多个线程各自持有对方线程等待释放的锁。
如何避免：
提供一个有序性的资源锁定和

## ConcurrentHashMap的实现原理

## 线程间如何通讯

## volatile 关键字在什么场景下使用。
在高并发场景下，保证共享变量的可见性。

