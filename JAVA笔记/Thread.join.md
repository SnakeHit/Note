# 概要

本文分三个部分对Thread.join()进行分析：

[1. join() 的示例和作用](https://www.cnblogs.com/huangzejun/p/7908898.html#p1)

[2. join() 源码分析](https://www.cnblogs.com/huangzejun/p/7908898.html#p2)

[3. 对网上其他分析 join() 的文章提出疑问](https://www.cnblogs.com/huangzejun/p/7908898.html#p3)

 

# 1. join() 的示例和作用

## 1.1 示例

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
 // 父线程
 public class Parent {
     public static void main(String[] args) {
         // 创建child对象，此时child表示的线程处于NEW状态
         Child child = new Child();
         // child表示的线程转换为RUNNABLE状态
         child.start();
         // 等待child线程运行完再继续运行
         child.join();
     }
 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
 // 子线程
 public class Child extends Thread {
     public void run() {
         // ...
     }
 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

上面代码展示了两个类：Parent（父线程类），Child（子线程类）。

Parent.main()方法是程序的入口，通过 Child child = new Child(); 新建child子线程（此时 child子线程处于NEW状态）；

然后调用child.start()（child子线程状态转换为RUNNABLE）；

再调用child.join()，此时，Parent父线程会等待child子线程运行完再继续运行。

 

下图是我总结的 Java 线程状态转换图：

![483183-20180829092510311-1546229320](C:\Users\k\Desktop\483183-20180829092510311-1546229320.png)

## **1.2 join() 的作用**

让父线程等待子线程结束之后才能继续运行。

我们来看看在 Java 7 Concurrency Cookbook 中相关的描述（很清楚地说明了**join()**的作用）：

> **Waiting for the finalization of a thread**
>
> **In some situations, we will have to wait for the finalization of a thread. For example, we may have a program that will begin initializing the resources it needs before proceeding with the rest of the execution. We can run the initialization tasks as threads and wait for its finalization before continuing with the rest of the program. ==For this purpose, we can use the join() method of the Thread class. When we call this method using a thread object, it suspends the execution of the calling thread until the object called finishes its execution==.**

​     当我们调用某个线程的这个方法时，这个方法会挂起调用线程，直到被调用线程结束执行，调用线程才会继续执行。

 

# **2. join() 源码分析**

以下是 JDK 8 中 join() 的源码：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
 1 public final void join() throws InterruptedException {
 2     join(0);
 3 }
 4 
 5 public final synchronized void join(long millis)
 6 throws InterruptedException {
 7     long base = System.currentTimeMillis();
 8     long now = 0;
 9 
10     if (millis < 0) {
11         throw new IllegalArgumentException("timeout value is negative");
12     }
13 
14     if (millis == 0) {
15         while (isAlive()) {
16             wait(0);
17         }
18     } else {
19         while (isAlive()) {
20             long delay = millis - now;
21             if (delay <= 0) {
22                 break;
23             }
24             wait(delay);
25             now = System.currentTimeMillis() - base;
26         }
27     }
28 }
29 
30 public final synchronized void join(long millis, int nanos)
31 throws InterruptedException {
32 
33     if (millis < 0) {
34         throw new IllegalArgumentException("timeout value is negative");
35     }
36 
37     if (nanos < 0 || nanos > 999999) {
38         throw new IllegalArgumentException(
39                             "nanosecond timeout value out of range");
40     }
41 
42     if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
43         millis++;
44     }
45 
46     join(millis);
47 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

join() 一共有三个重载版本，分别是无参、一个参数、两个参数：

```java
1 public final void join() throws InterruptedException;
2 
3 public final synchronized void join(long millis) throws InterruptedException;
4 
5 public final synchronized void join(long millis, int nanos) throws InterruptedException;
```

其中

(1) 三个方法都被final修饰，无法被子类重写。

(2) `join(long), join(long, long)` 是synchronized method，同步的对象是当前线程实例。

(2) 无参版本和两个参数版本最终都调用了一个参数的版本。

(3) `join()` 和 `join(0)` 是等价的，表示一直等下去；join(非0)表示等待一段时间。

从源码可以看到 join(0) 调用了Object.wait(0)，其中Object.wait(0) 会一直等待，直到被notify/中断才返回。

`while(isAlive())`是为了防止子线程伪唤醒(spurious wakeup)，只要子线程没有TERMINATED的，父线程就需要继续等下去。

(4) `join()` 和 `sleep()` 一样，可以被中断（被中断时，会抛出 InterrupptedException 异常）；不同的是，`join()` 内部调用了 `wait()`，会出让锁，而 `sleep()` 会一直保持锁。

 

以本文开头的代码为例，我们分析一下代码逻辑：

调用链：`Parent.main() -> child.join() -> child.join(0) -> child.wait(0)`（此时 Parent线程会获得 child 实例作为锁，其他线程可以进入 `child.join()` ，但不可以进入 `child.join(0)`， 因为`child.join(0)`是同步方法）。

如果 child 线程是 Active，则调用 `child.wait(0)`（为了防止子线程 spurious wakeup, 需要将 wait(0) 放入 `while(isAlive())` 循环中。

一旦 child 线程不为 Active （状态为 TERMINATED）, `child.notifyAll()` 会被调用-> `child.wait(0)`返回 -> `child.join(0)`返回 -> `child.join()`返回 -> `Parent.main()`继续执行, 子线程会调用this.notify()，child.wait(0)会返回到`child.join(0)` ，`child.join(0)`会返回到 `child.join(), child.join()` 会返回到 Parent 父线程，Parent 父线程就可以继续运行下去了。

 

# 3. 对网上其他分析 join() 的文章提出疑问

我觉得网上很多文章的描述有歧义，下面挑选一些描述进行分析，也欢迎大家留言一起讨论。

 

a. 子线程结束之后，"**会唤醒主线程"，**父线程重新获取cpu执行权，继续运行。

这里感谢[kerwinX](https://home.cnblogs.com/u/1709618/)的留言，子线程结束后，子线程的this.notifyAll()会被调用，join()返回，父线程只要获取到锁和CPU，就可以继续运行下去了。

 

 b. join() 将几个并行的线程"**合并为一个单线程"**执行。

我理解这个说法的意思，但是这样描述只会让读者更难理解。

在调用 join() 方法的程序中，原来的多个线程仍然多个线程，**并没有发生“合并为一个单线程”**。真正发生的是调用 join() 的线程进入 TIMED_WAITING 状态，等待 join() 所属线程运行结束后再继续运行。

 

一点感想：技术人员写作技术文章时，最好尽量避免使用过于口语化的词汇。

因为这种词汇歧义比较大，会让读者感到更加困惑或形成错误的理解。