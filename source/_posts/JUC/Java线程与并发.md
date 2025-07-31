---
title: Java 线程与并发
date: 2021-08-10 09:06:20
tags:
- JUC
- 锁
categories:
- JUC
---
<center>
  引言:
  JUC核心部分
</center>
<!-- more -->

#  Java 线程与并发

本章分为两个部分：

上半部分：Java多线程——基础

下半部分：Java并发编程——进阶

# Java多线程

## 创建线程的方式

在Java中，有**四种**创建线程的方法：

- 继承`Thread`类
  1. 继承`Thread`类
  2. 重写`run()`方法
  3. 用`start()`方法开启线程（是一个Native方法）
- 实现`Runnable`接口
  1. 实现`Runnable`接口
  2. 重写`run()`方法
  3. 使用`Thread`的构造方法，传入实现了`Runnable`接口的类对象创建对象
  4. 调用`Thread`对象的`start()`方法
- 实现`Callable`接口（一个**有返回值的线程**）
  1. 实现`Callable<T>`接口，注意有泛型
  2. 重写`call()`方法
  3. 通过`ExecutorService`对象的`submit( Callable<T> )`方法，将实现了Callable接口的`thread`上传，返回值是一个`Future`对象
  4. 通过`Future`对象的`get()`方法就可以获取到值
- 线程池（具体内容会在下一章**Java并发编程**进行介绍）
  1. 通过`Executor`来获取线程池
  2. 通过`ExecutorService`的`execute(Runnable接口)`执行任务，没有返回值
  3. 通过`ExecutorService`的`shutdown()`方法关闭线程池

第一种方式demo（过于简单可以跳过）:

```java
public class MyThread01 extends Thread{
    @Override
    public void run() {
        super.run();
        System.out.println("继承Thread实现线程");
    }

    public static void main(String[] args) {
        MyThread01 myThread = new MyThread01();
        myThread.start();
    }
}
```

第二种方式的demo（过于简单可以跳过）：

```java
public class MyThread02 implements Runnable{
    @Override
    public void run() {
        System.out.println("用接口新建线程");
    }

    public static void main(String[] args) {
        MyThread02 myThread02 = new MyThread02();
        Thread thread = new Thread(myThread02);
        thread.start();
    }
}
```

实现`Callable`接口demo：

```java
public class MyThread03 implements Callable<String> {

    @Override
    public String call() throws Exception {
        // 具有返回值的线程，重写call方法
        String[] strs = {"a","b","c","d","e"};
        return strs[new Random().nextInt(5)];
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyThread03 thread = new MyThread03();
        // 创建执行服务
        ExecutorService service = Executors.newFixedThreadPool(1);
        // 提交执行
        Future<String> res = service.submit(thread);
        // 使用get获取返回值
        String s = res.get();
        System.out.println(s);
        // 关闭服务
        service.shutdownNow();
    }
}
```

线程池demo：

```java
// 1 获取线程池
ExecutorService threadPool = Executors.newFixedThreadPool(10);
while(true) {
    // 2. 执行任务，这里使用了lambda表达式
    threadPool.execute(() -> {
        System.out.println(Thread.currentThread().getName() + " is running ..");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
}
```

## 不同创建方式的区别

- 继承`Thread`类：优点是简单方便；缺点是Java单继承，如果已经有一个父类，将不再能使用这种方法。
- 实现`Runnable`接口：较好的创建线程的方法
- 实现`Callable`接口：需要配合`ExecutorService`使用，如果需要返回值可以使用这种方法，返回值可以通过`Future`获得
- 使用线程池：较为复杂，但是功能多样。

## 线程中使用的设计模式：静态代理模式

静态代理模式中有 **真实对象、代理对象**

- 真实对象与代理对象要**实现同一个接口**
- 代理对象要**代理真实的角色**

优点：

静态代理模式可以帮助我们处理一些其他的事情，真实对象可以专注于做本职任务

---

举例：

在多线程中，实现`Runnable`接口的类就使用了静态代理模式：

例如这个demo：真实对象——`MyThread02`、代理对象`Thread`，他们实现了同一个接口`Runnable`，然后通过代理类`Thread`代理真实对象`myThread02`，执行`run`方法（通过`start`执行）

```java
public class MyThread02 implements Runnable{
    @Override
    public void run() {
        System.out.println("用接口新建线程");
    }

    public static void main(String[] args) {
        MyThread02 myThread02 = new MyThread02();
        Thread thread = new Thread(myThread02);
        thread.start();
    }
}
```

## 线程的五大状态

老生常谈的问题，说再多不如图：

（其实这里的五大状态，应该算OS层面的线程的五大状态，具体JVM里线程的状态，后面会说）

![线程五大状态](http://img.yesmylord.cn//image-20210727000042302.png)

除此外，还要说明几点：

1. 创建状态：此时Jvm会为其分配内存空间，初始化成员变量的值
2. 就绪状态：JVM为其创建方法栈和PC
3. 运行状态：获得了CPU
4. 阻塞状态分三种情况
   - 等待阻塞：线程调用了`wait()`方法，进入等待队列
   - 同步阻塞：要获取的同步锁被别的线程占用，JVM会将这个队列放入锁池（Lock Pool）中
   - 其他阻塞：由于`sleep()`、`join()`，或者是IO请求时产生中断
5. 导致线程死亡的情况（下一节详细介绍）
   - 正常结束：`run`或`call`方法运行结束
   - 异常结束：抛出未捕获的Error或是Exception
   - 调用stop：`stop()`不建议使用，因为很容易导致死锁；官方也声明这是一个即将过时的方法。

## 终止线程的方式

终止线程有很多方式，这里主要介绍四种：

### 正常退出

程序`run()`或`call()`方法运行结束，线程正常退出

### 使用flag退出线程

大多数情况下，线程是**伺服线程**，所以我们一般**使用一个变量**来控制线程的退出：

> 伺服线程：即需要长时间运行的线程，多为循环体

```java
public class ThreadSafe extends Thread {
    public volatile boolean exit = false;
    public void run() {
        while (!exit){
            //do something
        }
    }
}
```

注意到，此变量使用了`volatile`关键字，可以使同一时刻只能有一个线程修改`exit`的值（此关键字看下文详细阐述）

### 使用Interrupt

> 注意：中断并不会直接终止线程，而是给线程发送一个中断信号，线程可以根据中断信号来决定是否终止自己的执行。

因此终止线程需要我们自己动手，中断只是发出一个信号，在线程中，使用`isInterrupted()`判断

```java
public void run() {
    while (!Thread.currentThread().isInterrupted()) {
        // 执行线程逻辑
    }
}
```

根据线程是否处于阻塞状态，使用`interrupt`中断线程有两种情况：

1. 线程处于阻塞状态：

   - 一些操作（如 `sleep()`、`wait()`、`join()` 等）会导致线程阻塞

   - 当阻塞的线程调用 `interrupt()`方法时，会抛出 `InterruptException`异常。此时我们想跳出线程就必须通过代码捕获该异常，然后 break 跳出循环状态

   - 注意：只有当**捕获异常并`break`后，才能正常结束`run`方法**

   - ```java
     while (!Thread.currentThread().isInterrupted()){
         try {
             System.out.println("sleep");
             Thread.sleep(2000);
             System.out.println("wakeup");
         } catch (InterruptedException e) {
             break;
         }
     }
     ```

2. 线程不处于阻塞状态：

   - 使用`isInterrupted()`判断线程的中断标志来退出循环。当使用`interrupt()`方法时，中断标志就会置`true`
   
   ```java
   public class ThreadSafe extends Thread {
       public void run() { 
           while (!isInterrupted()){ 
               //非阻塞过程中通过判断中断标志来退出
               try{
                   Thread.sleep(5*1000);
                   //阻塞过程捕获中断异常来退出
               }catch(InterruptedException e){
                   e.printStackTrace();
                   break;//捕获到异常之后，执行 break 跳出循环
               }
           }
       } 
   }
   ```

### 使用stop

一个已经过时的方法，线程不安全 0

`thread.stop()`调用之后，创建子线程的线程就会抛出`ThreadDeatherror `的错误，并且会**释放子线程持有的隐式锁**。

一般任何进行加锁的代码块，都是为了保护数据的一致性，如果在调用 `thread.stop()`后导致了该线程所持有的所有锁的突然释放(不可控制)，那么被保护数据就有可能呈现不一致性，其他线程在使用这些被破坏的数据时，有可能导致一些很奇怪的应用程序错误。

## `sleep()`与`wait()`

- `sleep()`方法在`Thread`类中，是一个**本地静态方法**

```java
public static native void sleep(long millis) throws InterruptedException;
```

- `wait()`方法是在`Object`类中的，是一个**不可重写的本地方法**

```Java
public final native void wait(long timeout) throws InterruptedException;
```

| 对比项           | sleep                                           | wait               |
| ---------------- | ----------------------------------------------- | ------------------ |
| 是否让出CPU      | 是                                              | 是                 |
| 是否让出对象锁   | **不释放**                                      | **释放**           |
| 如何进入就绪状态 | 设定时间到或是调用`interrupt()`方法唤醒休眠线程 | 调用`notify`方法   |
| 使用范围         | 任何地方                                        | 必须在同步代码块中 |

> `wait`是醒着的等待，所以会释放锁
>
> `sleep`抱着锁睡着了，所以不会释放锁 

## `start`方法

在Java源码中，`start()`方法会调用本地方法`start0()`，由C来实现线程的创建

> 所以Java本质上来说，是创建不了线程的，需要调用C++来实现

源码如下：

```java
public synchronized void start() {
    if (threadStatus != 0)
        throw new IllegalThreadStateException();
    group.add(this);
    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
            /* do nothing. If start0 threw a Throwable then
it will be passed up the call stack */
        }
    }
}

private native void start0();
```

> start方法与run方法的区别：

```java
Thread t1 = new Thread(r1);
t1.start(); 
t1.run();
```

- 使用 `Thread.start()` 来启动新的线程，实现并行执行。
- 使用 `Thread.run()` 仅仅是在当前线程上同步地执行 `run()` 方法的代码，不会启动新的线程



# Java并发编程

## JUC

> 并发编程离不开JUC，什么是JUC？

指JDK下的包：`java.util.concurrent`，简写为JUC

这个包内包含所有的与并发相关的操作，因此取名为JUC

>并发：cpu快速切换程序执行，形成同时运行的假象（多个任务在同一时间段内交替执行）
>
>并行：相对于串行而言，指多个程序同时执行（多个任务在同一时刻同时执行）

## 守护线程

> 守护线程（也叫后台线程）：
>
> 为用户线程提供公共服务，没有用户线程时会自动离开

特点：

- 优先级比较低
- 普通线程可以通过`setDaemon(true)`来设置一个线程为守护线程
- 守护线程中**创建的新线程依然是守护线程**
- **守护线程是JVM级别的**；以 Tomcat 为例，如果你在 Web 应用中启动一个线程，这个线程的生命周期并不会和 Web 应用程序保持同步。也就是说，即使你停止了 Web 应用，这个线程依旧是活跃的
- 只要有一个用户线程，那么守护线程就不会退出；如果全是守护线程，那么守护线程也就会退出

Java默认有两个线程：

1. `main`线程： Java 程序的入口，它从 `main` 方法开始执行。在 `main` 方法中创建的任何线程都会成为主线程的子线程
2. `GC`线程：GC线程就是守护线程，当GC线程是JVM中仅剩的线程时，GC线程会自动离开

## 线程池

### 线程池的作用

1. **增快响应速度**
2. **控制并发量**（最主要的原因）
3. **对线程进行统一管理**
4. **减小线程切换时的上下文开销**

> 实现原理：每一个Thread都有一个start方法，当调用start启动线程时，JVM就会调用该类的run方法
>
> **线程池就是通过不断向start方法中传递Runnable对象**

### 线程池常见类的简介

![Executor框架](http://img.yesmylord.cn//Executor.png)

- `Executor`：顶级接口
- `ExecutorService`：次级接口，一般使用此类使用线程池，通过调用`execute`与`submit`方法执行任务
  - `execute`方法：**没有返回值**，执行Runnable方法
  - `submit`方法：返回Future接口对象，
- `Executors`：JDK官方实现的四类线程池，其本质就是ThreadPoolExecutor创建，只不过参数不同
- `ScheduledExecutorService`：ExecutorService的子接口，实现了任务定时执行，JDK实现的线程池中，newScheduledThreadPool会返回一个此接口的对象

- `ThreadPoolExecutor`：创建线程最详细的方法，有七个参数

### 线程池的组成&参数

1. 线程池管理器：用于创建并管理线程池 
2. 工作线程：线程池中的线程
3. 任务接口：每个任务必须实现的接口，用于工作线程调度其运行 
4. 任务队列：用于存放待处理的任务，提供一种缓冲机制

在`Executor`框架内，`ThreadPoolExecutor`负责创建线程池，构造方法如下：

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), defaultHandler);
}
```

- `corePoolSize`：线程池线程数量
- `maximumPoolSize`：最大线程数量
- `keepAliveTime`：最大连接时长（**当前线程数量处于上面两个数量之间，就会判断最大连接时长**）
- `unit`：时间单位
- `workQueue`：阻塞队列，被提交但是没有被执行的任务
- `threadFactory`：线程工厂，这里使用默认的线程工厂
- `handler`：拒绝策略

### 线程池的状态

`ThreadPoolExecutor`有五种状态：这五个状态由`ctl`来控制，`ctl`是一个`AtomicInteger`类型的变量，状态就由`ctl`来获取

```java
private static final int RUNNING    = -1 << COUNT_BITS;
// 线程池创建后处于Running状态
private static final int SHUTDOWN   =  0 << COUNT_BITS;
// 调用shutdown方法进入，不能接受新的任务，但是会将阻塞队列中的任务执行完毕
private static final int STOP       =  1 << COUNT_BITS;
// 调用shutDownNow进入STOP状态，线程池不能接受新的任务，阻塞队列中的任务也会被丢弃
private static final int TIDYING    =  2 << COUNT_BITS;
// 所有任务终止，ctl记录的任务数量为0，就会变为TIDYING（接着会执行Terminated()函数）
private static final int TERMINATED =  3 << COUNT_BITS;
// 执行完terminated方法后，就会由TIDYING转变为TERMINATED状态
```

### 拒绝策略

> 线程池中线程已经使用完，且任务队列也已经满了，此时就需要对新来的任务进行拒绝

JDK内置有四种拒绝策略，这四种拒绝策略是`ThreadPoolExecutor`类的内部类

1. `AbortPolicy` ：**直接抛出异常**，阻止系统正常运行。 
2. `CallerRunsPolicy`： 只要线程池未关闭，该策略**直接在调用者线程中，运行当前被丢弃的任务**。（显然这样做不会真的丢弃任务，但是，任务提交线程的性能极有可能会急剧下降）
3. `DiscardOldestPolicy` ： **丢弃最老的一个请求**，也就是即将被执行的一个任务，并尝试再次提交当前任务
4. `DiscardPolicy`： 该策略默默地丢弃无法处理的任务，不予任何处理。如果允许任务丢失，这是最好的一种方案。

### 不同线程池

----

Java中有四种线程池，他们的顶层接口是`Executor`，但是严格意义上来说`Executor`并不是一个线程池，而是一个执行线程池的工具，**真正的线程池接口是`ExecutorService`**

`ExecutorService`有四个静态方法：

- `newSingleThreadExecutor`
- `newFixedThreadPool`
- `newScheduledThreadPool`
- `newCachedThreadPool`

下面我们来说这些不同线程池的特点

#### newSingleThreadExecutor

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

特点：

- 核心线程只有一个
- 所有任务按照**先来先执行**的顺序执行

#### newScheduledThreadPool

```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}

//ScheduledThreadPoolExecutor():
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE,
          DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
          new DelayedWorkQueue());
}
```

特点：

- 可以定时执行
- 返回`ScheduledExecutorService`接口，是`ExecutorService`的子接口
- 有两个重要的方法：
  - `schedule()`方法可以实现延迟执行，有三个参数：
    - `Runnable`接口
    - **延迟时间**
    - 时间单位

  - `scheduleAtFixedRate()`可以实现定时周期执行，有四个参数：
    - `Runnable`接口
    - **初始延迟时间**
    - **执行周期**
    - 时间单位


demo：

```java
public static void main(String[] args) {
    ScheduledExecutorService ses = Executors.newScheduledThreadPool(3);
    ses.schedule(()-> System.out.println("延迟3s后执行"),3, TimeUnit.SECONDS);
    ses.scheduleAtFixedRate(()-> System.out.println("最开始延迟5s后，每三秒执行一次"),5,3, TimeUnit.SECONDS);
    ses.shutdownNow();
}
```

#### newCachedThreadPool

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

特点：

- 没有创建核心线程（核心线程数为0），最大线程数为`Integer.MAX_VALUE`
- 将任务添加到**同步等待队列**`SynchronousQueue`（如果入列成功，那么会等待空闲的线程去运行，如果没有空闲线程，会创建线程运行）

- 适用于**短期异步程序**
- 若一个线程**60s**未被使用，会被移除

#### newFixedThreadPool

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
}
```

特点：

- 创建有n个线程的线程池
- **只会创建核心线程！**（**因为核心线程数与非核心线程数相等**）
- 如果任务队列没有任务，线程会阻塞在`take`方法，**不会被回收**
- 如果线程因失败或异常而终止，那么会创建一个新线程代替他持续后续的任务（可选）
- 池若不关闭，线程也不会移除

### 线程池工作原理

![Executor框架](http://img.yesmylord.cn//Executor.png)

由图可以看出，创建线程池的是`Executors`类，回到第一节的demo

线程池的工作原理如下：

1. 线程池刚创建时，**内部没有一个线程**
2. 当调用`execute()`方法添加任务，会与`corePoolSize`进行对比
   - 如果正在运行的线程数量小于`corePoolSize`，马上创建线程运行这个任务
   - 如果正在运行的线程数量大于等于`corePoolSize`，那么这个任务放入**任务队列**
   - 如果任务队列满了，而且正在运行的线程数量小于`maxmumPoolSize`，那么还是要创建非核心线程立刻运行这个任务
   - 如果任务队列满了，而且正在运行的线程数量大于等于`maxmumPoolSize`，那么会抛出`RejectExecutionException`异常（默认的抛弃策略`AbortPolicy`）
3. 线程完成任务会从任务队列找下一个任务来执行
4. 当一个线程闲置，并且运行时间超过`keepAliveTime`时，线程池会判断，如果当前运行的线程数量大于`corePoolSize`，那么这个线程就被停掉。所以线程池的所有任务完成后，它最终会收缩到`corePoolSize`的大小

原理如图：

![线程池工作原理](http://img.yesmylord.cn//image-20210809171055450.png)

## 阻塞队列

`BolckingQueue`的API：

| 方法\处理方式 | 抛出异常  | 返回特殊值 |  一直阻塞  |      超时退出      |
| :-----------: | :-------: | :--------: | :--------: | :----------------: |
|   插入方法    |  add(e)   |  offer(e)  | **put(e)** | offer(e,time,unit) |
|   移除方法    | remove()  |   poll()   | **take()** |  poll(time,unit)   |
|   检查方法    | element() |   peek()   |     -      |         -          |

常用的实现了此接口的类有：

- `ArrayBlockingQueue`
  - 底层由数组组成，**有界**的阻塞队列
  - 可以指定初始化大小，一旦初始化不能修改
  - 构造方法中可以设置是否为公平锁
- `LinkedBlockingQueue`
  - **无界**的阻塞队列
  - 底层是链表
  - 队列按照**先进先出**的原则对元素进行排序
- `DelayQueue`
  - 该队列中的元素**只有当其指定的延迟时间到了，才能够从队列中获取到该元素**
  - 也没有大小限制
- `PriorityBlockingQueue`
  - 基于优先级的无界阻塞队列（优先级的判断通过构造函数传入的Compator对象来决定）
  - 内部控制线程同步的锁采用的是**非公平锁**
- `SynchronousQueue`
  - 比较特殊，没有容器存储，适用于一些线程间直接传递任务的场景
  - 是`newCachedThreadPool`使用的阻塞队列
  - 每个插入操作必须等待一个相应的删除操作：一个`put`必须等一个`take`，一个`take`必须等一个`put`

## Java中线程的方法与状态转换

在JDK源码中，`Thread.State`类代码如下，有**六个状态**：

```java
public enum State {
	// 新建
    NEW,
    // 运行
    RUNNABLE,
    // 阻塞
    BLOCKED,
    //等待
    WAITING,
    //超时等待
    TIMED_WAITING,
   	// 终止状态
    TERMINATED;
}
```

线程的基本方法有`wait()`、`notify()`、`notifyAll()`、`sleep()`、`join()`、`yield`

![Java线程方法与状态变化图](http://img.yesmylord.cn//image-20210803204224306.png)

- `wait()`：直接调用后会进入`waiting`状态；会释放锁；加时间参数的话，会进入`TIMED_WAITING`状态

  - **注意：**`wait()`方法不能写在`if`的执行语句中，如果有此需求，可以使用`while`进行判断（**虚假唤醒**）

- `notify()`：唤醒在一个锁上等待的**单个**线程；如果有很多线程，会随机选择一个唤醒

- `sleep()`：进入`TIMED_WAITING`状态，不会释放当前占有的锁；

- `yield()`：会让线程从执行进入就绪状态，让出当前CPU时间片

- `interrupt()`：本意是给这个线程一个通知信号，会影响这个线程内部的一个中断标示位；**不会改变线程的状态**

  1. 调用方法不会中断一个正在运行的线程；仅仅只是改变了一个中断标识位

  2. 若线程原本调用`sleep()`而处于`TIMED_WAITING`状态，调用此方法会抛出`InterruptException`，从而使线程提前结束`TIMED_WAITING`状态

  3. 抛出`InterruptException`后，会恢复中断标志位

  4. 中断状态是线程固有的一个标识位，可以通过此标识位安全的终止线程

     比如，你想终止 一个 `thread`时，可以调用`thread.interrupt()`方法，在线程的 `run` 方法内部可以根据`thread.isInterrupted()`的值来优雅的终止线程

- `join()`：**等待其他线程终止**，在当前线程中调用`join()`，会使当前线程阻塞，等到另一个线程结束，才会变为就绪状态。

  - 为什么要有`join()`方法？很多情况下主线程启动了子线程，需要用到子线程的返回结果，即主线需要等到子线程结束后再结束，就有了`join`方法

下面是一个join方法的使用示例：

```java
Thread thread1 = new Thread(() -> {
    System.out.println("Thread 1 started.");
    try {
        Thread.sleep(2000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("Thread 1 finished.");
});

Thread thread2 = new Thread(() -> {
    System.out.println("Thread 2 started.");
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("Thread 2 finished.");
});

thread1.start();
thread2.start();
try {
    thread1.join(); // Main等待 thread1 执行完毕
    thread2.join(); // Main等待 thread2 执行完毕
} catch (InterruptedException e) {
    e.printStackTrace();
}
System.out.println("Main is the last completed.");

// 输出结果
// Thread 1 started.
// Thread 2 started.
// Thread 1 finished.
// Thread 2 finished.
// Main is the last completed.
```

## wait的使用

wait并不是Thread的方法，而是Object的方法，但是wait能改变当前线程的状态。

wait一般搭配Synchronized使用。

### wait与notify打印ABC

下面是一个wait与notify的使用示例，例子循环打印ABC：

```java
public class WaitNotify {
    public static void main(String[] args) {
        Printer printer = new Printer();

        Thread threadA = new Thread(() -> {
            printer.printLetter("A", 0, 1);
        });

        Thread threadB = new Thread(() -> {
            printer.printLetter("B", 1, 2);
        });

        Thread threadC = new Thread(() -> {
            printer.printLetter("C", 2, 0);
        });

        threadA.start();
        threadB.start();
        threadC.start();
    }
}
class Printer {
    private int currentThreadIndex = 0;
    private final Object lock = new Object();

    public void printLetter(String letter, int threadIndex, int nextThreadIndex) {
        synchronized (lock) {
            for (;;) {
                while (currentThreadIndex != threadIndex) {
                    // 如果当前打印的ID不是需要的，那么进入等待，并且释放当前的锁
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.print(letter);
                currentThreadIndex = nextThreadIndex;
                lock.notifyAll();
            }
        }
    }
}
```

---

### 虚假唤醒

> 虚假唤醒：例如，生产者生产了1个商品，但是却唤醒了3个消费者来消费，最终只能有一个消费者消费成功，其他两个线程就被“忽悠”了

测试demo：

```java
class PV{
    private int x = 0;
    public synchronized void p() throws InterruptedException {
        if( x == 0){// 将这里改为while
            this.wait();
        }
        x --;
        System.out.println(Thread.currentThread().getName()+"："+x);
        this.notifyAll();
    }
    public synchronized void v() throws InterruptedException {
        if( x != 0){// 将这里改为while
            this.wait();
        }
        x ++;
        System.out.println(Thread.currentThread().getName()+"："+x);
        this.notifyAll();
    }
}
```

main方法

```java
public static void main(String[] args) {
    PV pv = new PV();
    new Thread(()-> {
        for (int i = 0; i < 10; i++) {
            try {
                pv.v();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    },"A").start();
    // 同上，继续创建线程 B、C、D分别运行 p()、v()、p()操作
}
```

运行结果如下：

```java
A：1
B：0
A：1
C：0
A：1
B：0
A：1
C：0
B：-1
B：-2
B：-3
C：-4
C：-5
...
```

发现出现了负数这种情况，显然不是我们想要的

为什么会出现这种问题？

注意这里

```java
if( x != 0){
    this.wait();
}
x ++;
this.notifyAll();
```

我们使用`if`进行判断，只会执行一次，如果该线程被唤醒，那么将不会去判断`x != 0`这个条件

所以要使用`while`，将方法中`if`判断改为while即可

总结：

**如果要判断条件并进行`wait()`方法，不能使用`if()`，会出现虚假唤醒的现象**

## 锁及相关概念（重点）

### 乐观锁与悲观锁

乐观锁与悲观锁是一种对于锁的思想：

- 乐观锁：认为**写入少**
- 悲观锁：认为**写入多**

由两种观点，就有不同的实现：

乐观锁认为写入少，所以**不会上锁**，但是更新时会进行一个判断（CAS操作），这样即使没有上锁，也不会出现线程安全问题。

悲观锁认为写入多，在**每次读/写数据时都会进行上锁**，其他线程想要进行读写数据，必须先拿到锁（`Synchronized`就是悲观锁的一种实现）。

### 什么是CAS

**CAS（Compare And Swap/Set）：比较并变换**，是一个**原子**操作，**相同则更新**

> `CAS(V,E,N)`
>
> V 表示要更新的变量（内存值，由于多线程的存在，可能与E不同）
>
> E 表示预期值（旧的）
>
> N 表示新值（想设置的新值）

CAS比较流程：

1. 如果`V==E`值时，会将 `V=N`（内存值 == 预期值，说明没有线程对当前变量进行写操作）
2. 如果`V!=E`，则当前线程什么都不做（内存值 != 预期值，说明已经有其他线程做了更新，那么现在就不能更改这个值）
3. 最后，CAS操作返回当前 V 的真实值

注意：

- CAS 操作是**抱着乐观的态度**进行的（乐观锁），它总是认为自己可以成功完成操作
- CAS可以用来实现**自旋锁**：即会有一个线程进行自旋，反复判断是否符合条件
- 当多个线程同时使用 CAS 操作一个变量时，**只有一个会胜出，并成功更新**，其余均会失败
- **失败的线程不会被挂起，仅是被告知失败，并且允许再次尝试**，当然也允许失败的线程放弃操作
- 基于这样的原理，**CAS 操作即使没有锁，也可以发现其他线程对当前线程的干扰**，并进行恰当的处理。

在Java中，原子类AtomicInteger有CompareAndSet的操作：

```java
AtomicInteger atomicValue1 = new AtomicInteger(9);
AtomicInteger atomicValue2 = new AtomicInteger(10);

int expectedValue = 10;
int newValue = 20;

boolean updatedState1 = atomicValue1.compareAndSet(expectedValue, newValue);
boolean updatedState2 = atomicValue2.compareAndSet(expectedValue, newValue);

// 因为期望值10与现有值9不同，因此不会设置，返回原值
System.out.println("Value updated: " + updatedState1); // false
System.out.println("Final Value: " + atomicValue1.get()); // 9
// 因为期望值10与现有值10相同，因此改变为20，并且返回原值
System.out.println("Value updated: " + updatedState2); // true
System.out.println("Final Value: " + atomicValue2.get()); // 20
```

> CAS存在的问题：

1. ABA问题
2. 循环性能开销大
3. 只能保证单个变量的原子操作

### AtomicInteger

AtomicInteger是一个保证原子操作的Integer类

它的关键结构如下：

```java
// 存储内存偏移量（相对于对象的起始位置，获得成员变量的偏移量）
private static final long valueOffset;

// 创建一个AtomicInteger，就先获得他的内存的偏移
static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}

// 真实的值由此存储，为了保证有序和可见，使用volatile修饰
private volatile int value;
```

我们知道，普通的int，他的i++操作并不是一个原子操作，但是AtomicInteger的getAndIncrement是一个原子操作

```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
```

getAndIncrement调用了unsafe类，unsafe类提供了一些底层的、危险的操作，通常用于实现Java标准库和虚拟机的内部功能

下面是其`getAndAddInt`：

```java
// 下面贴出unsafe的getAndAddInt方法传入三个参数:本实例，value的内存地址偏移(偏移量是相对于对象的起始地址的位置)，要加的数量
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    // 进入自旋
    do {
        // 使用 getIntVolatile 方法从指定对象的指定偏移量处获取整数值
        var5 = this.getIntVolatile(var1, var2);
        // 循环执行 CAS 操作，直到成功为止
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
    return var5;
}
```

因此AtomicInteger的实现原理就是：**volatile + cas**

### ABA问题

CAS过程中，有ABA问题

![ABA示意图](http://img.yesmylord.cn//image-20210814202529921.png)

> 如何解决ABA问题？

加一个版本号即可，每次更改这个值就对齐加1，然后cas比较这个版本号就知道是否出现了ABA问题

### 自旋锁

> 自旋锁：CPU对线程进行轮询，反复询问是否释放锁，直到释放为止
>
> 自旋周期：CPU轮询的时间

优点：减少了线程阻塞；对于锁的竞争不激烈，且占用锁时间非常短的代码块来说性能能大幅度上升

缺点：如果锁竞争激烈或是占用锁时间长，那么会持续的占用CPU是极大的性能损耗

在Java中，1.5时自旋周期时定死的，在1.6后加入了**适应性自旋锁**，由前一次在同一个锁上的自旋时间以及锁的拥有者的状态来决定

```java
// 自旋锁的开启
JDK1.6 中-XX:+UseSpinning 开启
-XX:PreBlockSpin=10 设置自旋次数
JDK1.7 后，去掉此参数，由 jvm 控制
```

使用CAS操作可以实现一个自旋锁。

### 可重入锁（递归锁）与不可重入锁

> 可重入锁（也叫递归锁）：
>
> 理解方式一：当一个线程获取对象锁之后，这个线程可以再次获取本对象上的锁，而其他的线程是不可以的
>
> 理解方式二：一个线程执行一个嵌套的方法时，当外部方法获取到锁，他内部调用的方法无需再去获取锁
>
> 理解方式三：**锁分配的单位是线程，而不是方法**。一个方法无论嵌套自身的方法多少次，锁依然在这个线程内，因此无需再获取锁

在JAVA环境下`ReentrantLock`和`synchronized`都是可重入锁

可重入锁的目的是为了解决死锁的问题

### 公平锁与非公平锁

公平锁（`Fair`）：加锁前检查是否有排队等待的线程，**优先排队等待的队列，先来先得**

非公平锁（`Nonfair`）：加锁不考虑排队等待问题，**直接尝试获取锁，获取不到自动到队尾等待**（可以插队）

注意：

- 非公平锁性能高于公平锁5-10倍，因为公平锁要维护等待队列
- 在Java中，`synchronized`是非公平锁，`ReentrantLock`**默认**的`lock()`方法采用的是非公平锁
- 非公平锁可能会导致“饥饿”的现象发生。

### 共享锁和独占锁

独占锁（也被称为写锁）：**每次只有一个线程能持有锁**；一种悲观策略，无论是读操作还是写操作，都会进行加锁。

共享锁（也被称为读锁）：**允许多个线程同时获取锁**，并发访问，共享资源。一种乐观锁

注意：

- JUC中的`ReadWriteLock`读写锁，允许一个资源可以被**多个读操作**访问，或者被**一个写操作**访问，但两者不能同时进行
- 在共享锁占有期间，不允许写操作，如果有写操作，需要释放共享锁，转为独占锁（这种转变也称为**锁升级**）

### AQS同步抽象队列

> AQS (AbstractQueuedSynchronizer)：抽象的队列式同步器，定义了一套多线程访问共享资源的同步器框架，很多锁都是通过AQS来实现的，例如`ReentrantLock、Semaphore、CountDownLatch`
>

这个抽象类主要维护了一个**状态state**还有一个**FIFO的线程等待队列**：

![AQS](http://img.yesmylord.cn//image-20210814200556108.png)

`state`状态：

```java
private volatile int state;
// state 代表共享资源；可以看到其使用volatile修饰
```

有三个方法可以操作这个状态的值：

```java
protected final int getState() {
    return state;
}

protected final void setState(int newState) {
    state = newState;
}

protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

这里贴一个使用AQS实现的独占锁demo：一般AQS都会使用静态内部类来实现

```java
public class AqsLock {
    // 建议使用内部类来实现
    static class Sync extends AbstractQueuedSynchronizer{
        private static final int UNLOCK = 0;
        private static final int LOCK = 1;
        @Override
        protected boolean tryAcquire(int acquires) {
            if (compareAndSetState(UNLOCK, LOCK)) {
                // 设置当前线程为独占线程
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        @Override
        protected boolean tryRelease(int releases) {
            if (getState() == UNLOCK) {
                throw new IllegalMonitorStateException("Lock is not held by the current thread.");
            }
            // 清除当前独占线程
            setExclusiveOwnerThread(null);
            setState(UNLOCK);
            return true;
        }
        @Override
        protected boolean isHeldExclusively() {
            return getState() == LOCK && getExclusiveOwnerThread() == Thread.currentThread();
        }

        public void lock() {
            acquire(1);
        }

        public void unlock() {
            release(1);
        }
    }
    
    public static void main(String[] args) {
        ...
    }
}
```

```java
public static void main(String[] args) {
    Sync sync = new Sync();

    Runnable task = () -> {
        System.out.println(Thread.currentThread().getName() + " is trying to acquire the lock.");
        sync.lock();
        try {
            System.out.println(Thread.currentThread().getName() + " has acquired the lock.");
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println(Thread.currentThread().getName() + " is releasing the lock.");
            sync.unlock();
        }
    };

    Thread thread1 = new Thread(task);
    Thread thread2 = new Thread(task);

    thread1.start();
    thread2.start();
}
```

执行结果为：

```
Thread-0 is trying to acquire the lock.
Thread-1 is trying to acquire the lock.
Thread-0 has acquired the lock.
Thread-0 is releasing the lock.
Thread-1 has acquired the lock.
Thread-1 is releasing the lock.
```

### 锁升级

总共有四种：**无状态锁、偏向锁、轻量级锁、重量级锁**

在内存中，锁的信息**存放在对象头中的markword中**（markword包含的内容有三大部分：Hashcode、锁信息、GC信息）

> 锁升级：
>
> ​		随着锁的竞争，锁可以从偏向锁升级到轻量级锁，再升级到重量级锁；
>
> ​		但是锁的升级只能是**单向**的，不存在降级

#### 偏向锁

大部分情况下锁并不存在多线程竞争，而总是由同一线程多次获得

由此提出了偏向锁

> 偏向锁：在某个线程获得锁后，消除这个线程重入（CAS）的开销，看起来非常偏向这个线程，所以叫偏向锁

轻量级锁的获取及释放依赖多次CAS，但是偏向锁只需要在置换线程ID时依赖一次CAS指令

特点：

- **偏向锁主要用来优化同一线程多次申请同一个锁的竞争**

- 一次CAS，只比较Thread ID
- 消除了重入开销

#### 轻量级锁

> “轻量级”是相对于使用操作系统互斥量来实现的传统锁而言的

作用：

- 多次CAS，自旋判断

#### 重量级锁

`synchronized`是通过对象内部的一个叫做**监视器锁**来实现的，但是监视器锁本质又是依赖于操作系统底层的`Mutex lock`来实现的

操作系统想要实现一个重量级锁，**必须从用户态切换到核心态**，所以这也是`synchronized`效率低的原因

> 重量级锁：**依赖于操作系统的`Mutex Lock`实现的锁**

**JDK1.6 以后**，为了减少获得锁和释放锁所带来的性能消耗，提高性能，**引入了“轻量级锁”和 “偏向锁”**

#### 锁升级过程

![图源自敖丙博客](http://img.yesmylord.cn//image-20210818091932328.png)

当一个线程A要去获取一个锁的时候，简单过程如下：

1. 如果处于无锁状态，那么将锁设置为偏向锁，并设置A的线程号记录在对象头
   - 如果A重复进入此锁，只需要判断线程A线程ID与记录是否相同，就给锁（一次CAS即可）
2. 如果线程B想要获取锁，进行一次CAS判断
   - CAS判断成功：线程B获取到锁
   - CAS判断失败：升级为轻量级锁
3. 轻量级锁进行多次CAS判断，如果仍然不能满足当前的竞争状况，那么升级为重量级锁
4. 重量级锁是OS实现的排他锁，需要从用户态进入到核心态，十分浪费性能。

### Synchronized

> Java中使用专门的关键字`Synchronized`，是**悲观锁**、**可重入锁**、**非公平锁**

直接修饰：

- **修饰方法**：锁住对象的实例(`this`)，即方法的调用者
- **修饰静态方法**：锁住`Class`实例（因为`Class`数据存放在永久代（元空间），此位置是全局共享的，所以相当于一个**全局锁**）

`synchronized(obj){}`同步块中

- `obj`称为**同步监视器**；可以是任何对象，但是推荐使用共享资源作为同步监视器（修饰方法时，同步监视器就是`this`或是`class`）

#### 底层实现

对象头的markword会关联到一个**monitor对象**（这个对象是用C++语言写的）

- 当我们进入一个方法的时候，执行**monitor enter**，就会获取当前对象的一个所有权，这个时候monitor进入数为1，当前的这个线程就是这个monitor的owner。
- 如果你已经是这个monitor的owner了，你再次进入，就会把进入数+1（每次重入加一）
- 同理，当他执行完**monitor exit**，对应的进入数就-1，直到为0，才可以被其他线程持有。

所有的互斥，其实在这里，就是看你能否获得monitor的所有权，一旦你成为owner就是获得者。

![syn修饰代码块](https://img.yesmylord.cn//image-20240803160822726.png)

> Synchronized修饰的方法在抛出异常时,会释放锁吗?

会

### Lock

`synchronized`是悲观锁，无论线程是读还是写都会独占整个资源，因此出现了`Lock`接口

JUC下有`locks`包，这个包内，最常见就有`ReentrantLock`与`ReentrantReadWriteLock`

![locks包的部分关系](http://img.yesmylord.cn//image-20210803181741298.png)

`ReentrantReadWriteLock`虽然没有实现`Lock`接口，但是他的两个静态内部类`ReadLock`与`WriteLock`均实现了`Lock`接口

`Lock`接口部分方法如下：

```java
void lock(); //若锁处于空闲状态，当前线程将获取到锁
boolean tryLock();//如果锁可用, 则获取锁, 并立即返回 true, 否则返回 false
/* tryLock()和lock()的区别在于：
	tryLock()只是"试图"获取锁, 如果锁不可用, 不会导致当前线程等待, 当前线程仍然继续往下执行代码. 
	lock()方法则是一定要获取到锁, 如果锁不可用, 就一直等待, 在未获得锁之前,当前线程并不继续向下执行.
*/
void unlock();
//执行此方法时, 当前线程将释放持有的锁. 锁只能由持有者释放, 如果线程并不持有锁, 却执行该方法, 可能导致异常的发生
Condition newCondition();
//条件对象，获取等待通知组件。该组件和当前的锁绑定，当前线程只有获取了锁，才能调用该组件的 await()方法，而调用后，当前线程将缩放锁。
void lockInterruptibly();//使用此方法获取锁时，如果线程正在等待获取锁，那么这个线程可以响应中断，即可以中断线程的等待状态
/*也就是说，
	当两个线程同时通过lock.lockInterruptibly()想获取某个锁时，
	假若此时线程A获取到了锁，而线程B只有在等待，
	那么对线程B调用threadB.interrupt()方法能够中断线程B的等待过程。
*/
```

注意：

- **当一个线程获取了锁之后（运行状态），是不会被`interrupt()`方法中断的**；除非调用的是`lockInterruptibly()`方法获取锁

- **中断只能作用于处于WAITING状态的线程**

因此使用锁的基本方式均为：

```java
Lock lock = ...; // 声明一个锁
lock.lock();//加锁
try {
    // 同步操作
} catch (Exception e){
    e.printStackTrace();
}finally {
    lock.unlock(); 
    // 必须在finally中释放锁；因为lock即使发生异常也不会自动释放锁
}
```

### ReentrantLock

`ReentrantLock`继承了`Lock`接口并实现了接口中定义的方法，是一种**可重入锁**

![locks包的部分关系](http://img.yesmylord.cn//image-20210803181741298.png)

方法介绍：

![locks方法](http://img.yesmylord.cn//Package locks.png)

首先是实现了`Lock`接口的方法（上面已经介绍），其他是`ReentrantLock`自己的方法：

```java
getHoldCount(); //查询当前线程保持此锁的次数，也就是执行此线程执行 lock 方法的次数。
getQueueLength(); //返回正等待获取此锁的线程估计数，比如启动 10 个线程，1 个线程获得锁，此时返回的是 9
getWaitQueueLength(Condition condition); //返回等待与此锁相关的给定条件的线程估计数。
/* 比如 10 个线程，用同一个 condition 对象，并且此时这 10 个线程都执行了condition 对象的 await() 方法，那么此时执行此方法返回 10 */
hasWaiters(Condition condition);// 查询是否有线程等待与此锁有关的给定条件(condition)，对于指定 contidion 对象，有多少线程执行了 condition.await 方法
hasQueuedThread(Thread thread); // 查询给定线程是否等待获取此锁
hasQueuedThreads(); //是否有线程等待此锁
isFair(); //该锁是否公平锁
isHeldByCurrentThread();// 当前线程是否保持锁锁定，线程的执行 lock 方法的前后分别是 false 和 true
isLock();//此锁是否有任意线程占用
```

这里可以对比一下`synchronized`与`ReentrantLock`：

| 对比项       | synchronized  | ReentrantLock       |
| ------------ | ------------- | ------------------- |
| 如何加锁解锁 | JVM自动控制   | 程序员手动进行      |
| 是否公平     | 非公平锁      | 默认为非公平锁      |
| 是否可重入   | 可重入        | 可重入              |
| 发生异常     | JVM自动释放锁 | finally中手动释放锁 |
| 可中断锁     | 不可中断锁    | 可中断锁            |

总结：`ReentrantLock`对比`synchronized`主要增加了三项功能：

1. **等待可中断**：当持有锁的线程长期不释放锁时，**正在等待的线程可以选择放弃等待**，改为处理其他事情，它对处理执行时间非常长的同步块很有帮助（而在等待由`synchronized`产生的互斥锁时，会一直**阻塞**，是不能被中断的）
2. **可实现公平锁**：可以使用`new ReentrantLock(true)`来使用公平锁
3. **锁可以绑定多个Condition**：
   - `ReentrantLock`对象可以同时绑定多个`Condition`对象（条件变量或条件队列）；
   - 而在`synchronized`中，锁对象的`wait()`和`notify()`或`notifyAll()`方法可以实现一个隐含条件，但如果要和多于一个的条件关联的时候，就不得不额外地添加一个锁
   - `ReentrantLock`则无需这么做，只需要多次调用`newCondition()`方法即可。而且我们还可以通过绑定`Condition`对象来判断当前线程通知的是哪些线程（即与`Condition`对象绑定在一起的其它线程）

demo：

```java
public class Test {
    private ArrayList<Integer> arrayList = new ArrayList<Integer>();
    private Lock lock = new ReentrantLock();
    //注意这个地方，锁对象要放在成员变量这个地方
    public static void main(String[] args)  {
        final Test test = new Test();
         
        new Thread(){
            public void run() {
                test.insert(Thread.currentThread());
            };
        }.start();
         
        new Thread(){
            public void run() {
                test.insert(Thread.currentThread());
            };
        }.start();
    }  
     
    public void insert(Thread thread) {
        // 锁的创建不能放在方法内，要不然每一个线程获得的都是不同的锁
        lock.lock();
        try {
            System.out.println(thread.getName()+"得到了锁");
            for(int i=0;i<5;i++) {
                arrayList.add(i);
            }
        } catch (Exception e) {
            // TODO: handle exception
        }finally {
            System.out.println(thread.getName()+"释放了锁");
            lock.unlock();
        }
    }
}
```

#### ReentrantLock的源码实现

ReentrantLock内部其实是用AQS来保证的同步：

```java
// ReentrantLock成员变量，Sync实现了AQS
private final Sync sync;
// ReentrantLock构造方法，默认使用非公平锁
public ReentrantLock() {
    sync = new NonfairSync();
}
```

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
    abstract void lock();
    // 默认就是非公平锁，因此此方法就是tryLock方法
    final boolean nonfairTryAcquire(int acquires) {
        // acquires代表资源数，这里可以看做是1
        final Thread current = Thread.currentThread();
        // 获取当前的state状态
        int c = getState();
        if (c == 0) {
            if (compareAndSetState(0, acquires)) {
                // 1、CAS判断如果没有被占用，那么我就占用
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            // 2、如果就是我占用了这个锁，就将重入次数+1
            int nextc = c + acquires;
            if (nextc < 0) // overflow 重入次数太多，加到了int的最大值，再加就是负数了，SOF了
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
    ....// 还有其他方法，不过不太重要
}
```

默认使用非公平锁：

```java
static final class NonfairSync extends Sync {

    final void lock() {
        if (compareAndSetState(0, 1))
            // 没人获取锁，则设置自己占有
            setExclusiveOwnerThread(Thread.currentThread());
        else
            // 如果已经被占用，那就去执行tryAcquire()
            acquire(1);
    }

    protected final boolean tryAcquire(int acquires) {
        // 在这里调用nonfairTryAcquire方法
        return nonfairTryAcquire(acquires);
    }
}
```









### ReadWriteLock

读写锁将读操作与写操作进行分离：

```java
public interface ReadWriteLock {
    Lock readLock();// 返回读锁

    Lock writeLock();// 返回写锁
}
```

ReentrantReadWriteLock是ReadWriteLock的一个实现类，将读写操作分离：

满足四个原则：

- 允许多个线程一起读
- 只允许一个线程写
- 读时不能写（悲观读）
- 写时不能读

锁的降级与升级：支持**锁降级**（写锁变为读锁），但是不支持**锁升级**（读锁变为写锁）！

ReentrantReadWriteLock的Demo如下：

```java
public class ReadWriteLockDemo {
    private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    public void readData() {
        readWriteLock.readLock().lock(); // 获取读锁
        try {
            // 执行读操作
        } finally {
            readWriteLock.readLock().unlock(); // 释放读锁
        }
    }

    public void writeData() {
        readWriteLock.writeLock().lock(); // 获取写锁
        try {
            // 执行写操作
        } finally {
            readWriteLock.writeLock().unlock(); // 释放写锁
        }
    }
}
```

### Semaphore

> Semaphore：信号量，是对具体物理资源的抽象
>
> 处理多个共享资源的问题
>
> 关于信号量的详细解释，可以看我的另一篇[blog](https://www.yesmylord.cn/2020/12/09/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E8%BF%9B%E7%A8%8B/)

注意：现有资源数目信号量S。P、V操作分别代表消费者（申请资源）、生产者（释放资源）

- `S == 1`：信号量就变为互斥信号量
- `S > 0`：说明S资源还有S个
- `S < 0`：说明等待队列还有-S个进程阻塞着

Java中demo：

```java
// 信号量值为 3
Semaphore s = new Semaphore(3);
try {
    s.acquire();
    // 省略业务逻辑
} catch (InterruptedException e) {
    e.printStackTrace();
} finally {
    s.release();
}
```

可见信号量与`ReentrantLock`使用该方法基本一致

### 锁优化

有了锁虽然解决了线程安全问题，但是带来了性能的下降，此时就要进行锁优化了。

一般我们会有如下的锁优化方法：

- **减少锁持有时间**：只在有线程安全问题的程序上加锁
- **减小锁粒度**：将大对象拆成小对象，降低锁竞争
- **锁分离**：根据功能分离锁，例如`ReadWriteLock`，将读与写进行分离
- **锁粗化**：通常情况下，为了保证多线程间的有效并发，会要求每个线程持有锁的时间尽量短。但是，凡事都有一个度，如果对同一个锁不停的进行请求、同步和释放，其本身也会消耗系统宝贵的资源，反而不利于性能的优化 
- **锁消除**：编辑器级别的事情，可以对没有共享需求的代码进行优化，直接消除锁，这些多半是程序员编码不规范引起的。

### Volatile关键字

> volatile本意是“易失的”，在计算机内代表，被这个关键字修饰的变量**不会被缓存**起来；
>
> 对于非volatile变量来说，访问它的值会先从内存copy到CPU cache中，如果刚copy完，内存中的值就发生了改变，那么CPU读到的是cache中的值，而不是最新值
>
> 对于volatile修饰的变量，每次都要去内存中读取

被这个关键字修饰的变量代表着两种特性：**可见性与有序性**

- **变量可见性**：变量对所有线程可见（这里的可见性指：一个线程修改了变量的值，那么**新的值对于其他线程是可以立即获取的**）
- **禁止重排序**：多核CPU会对指令进行重排序，以加快指令的执行速度，使用此关键字可以不让CPU这么做

**优点：**

比`synchronized`更轻量级的一个同步锁，不会使线程阻塞

volatile 适合这种场景：**一个变量**被多个线程共享，线程直接给这个变量赋值

**注意：**

- 被`volatile`修饰的变量可以保证**单次读/写操作**的原子性
- 不能保证`i++`这种操作的原子性，因为本质上其是两次操作 读+写
- 必须同时满足两个条件，才能保证线程安全：
  - 对变量的写操作不依赖于当前值（`i++`），或者说是单纯的变量赋值（类似`flag = true`，不是这种`a += 10`）
  - 该变量没有包含在具有其他变量的不变式中（**不同的 volatile 变量之间，不能互相依赖**）只有在状态真正独立于程序内其他内容时才能使用 `volatile`

### 可见性与有序性实现的底层原理

> 底层是如何确保volatile的可见性的？

通过**缓存一致性协议**：不同厂商有不同协议，这里以牙膏厂的MESI为例：

​		**当CPU写数据时**，**如果发现操作的变量是共享变量**，会**发出信号通知其他CPU将该变量的缓存行置为无效状态**

> 底层是如何确保volatile的有序性的？

通过**内存屏障**，这是一个CPU指令，不能对其进行重排序，volatile就是基于内存屏障实现的

## JUC通信工具类

| 类             | 作用                                       |
| -------------- | ------------------------------------------ |
| Semaphore      | 限制线程的数量                             |
| Exchanger      | 两个线程交换数据                           |
| CountDownLatch | 线程等待直到计数器减为0时开始工作          |
| CyclicBarrier  | 作用跟CountDownLatch类似，但是可以重复使用 |
| Phaser         | 增强的CyclicBarrier                        |

### Semaphore

**用于资源有限的场景中，可以限制线程的数量**

比如我想限制只有3个线程在工作：

```java
public class SemaphoreDemo {
    static class MyThread implements Runnable {

        private int value;
        private Semaphore semaphore;

        public MyThread(int value, Semaphore semaphore) {
            this.value = value;
            this.semaphore = semaphore;
        }

        @Override
        public void run() {
            try {
                semaphore.acquire(); // 获取permit
                System.out.println(
                        String.format(
                                "当前线程是%d, 还剩%d个资源，还有%d个线程在等待",
                                value,
                                semaphore.availablePermits(), semaphore.getQueueLength()
                        )
                );
                // 睡眠随机时间，打乱释放顺序
                Random random = new Random();
                Thread.sleep(random.nextInt(1000));
                System.out.println(String.format("线程%d释放了资源", value));
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                semaphore.release(); // 释放permit
            }
        }
    }

    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);
        // 最多只可以有三个线程在工作
        for (int i = 0; i < 10; i++) {
            new Thread(new MyThread(i, semaphore)).start();
        }
    }
}
```

### Exchanger

**用于两个线程交换数据，数据支持泛型**（所以我们可以传IO流之类的）

调用到`exchange()`方法，线程会进入阻塞状态，只有另一个`exchange()`

方法被调用，才会继续执行

核心方法：

- `exchange(E e)`：将数据交给另一个线程（会进入阻塞）

```java
final static Exchanger<String> ex1 = new Exchanger<>();
public static void main(String[] args) {
    for (int i = 0; i < 4; i++) {
        new Thread(() -> {
            try {
                String msg1 = ex1.exchange(Thread.currentThread().getName() + "向你问好");
                System.out.println(Thread.currentThread().getName()+"收到信息：" + msg1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```

### CountDownLatch

> 闭锁、或者叫门闩：在闭锁到达结束状态之前，这扇门一直是关闭的。可以用来等其他线程执行。

假设**某个任务执行之前，需要等待其他线程完成一些任务**，那么就可以用`CountDownLatch`类

主要的方法有：

- `new CountDownLatch(int count)`：构造方法，参数是一个`int`值，代表需要等待几个任务
- `await()`：进入等待状态
- `await(long time, TimeUnit unit)`：进入等待状态，如果count为0或者时间到也会释放
- `getCount()`：获取当前`count`值
- `countDown()`：让`count`值减1，如果`count`为0，就会自动解锁`await`

```java
public class CountDownLatchDemo {
    // 定义前置任务线程
    static class PreTaskThread implements Runnable {
        private String task;
        private CountDownLatch countDownLatch;

        public PreTaskThread(String task, CountDownLatch countDownLatch) {
            this.task = task;
            this.countDownLatch = countDownLatch;
        }

        @Override
        public void run() {
            try {
                Random random = new Random();
                Thread.sleep(random.nextInt(1000));
                System.out.println(task + " - 任务完成");
                countDownLatch.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        // 假设有三个模块需要加载
        CountDownLatch countDownLatch = new CountDownLatch(3);

        // 主任务
        new Thread(() -> {
            try {
                System.out.println("等待数据加载...");
                System.out.println(String.format("还有%d个前置任务", countDownLatch.getCount()));
                countDownLatch.await();
                System.out.println("数据加载完成，正式开始游戏！");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        // 前置任务
        new Thread(new PreTaskThread("加载地图数据", countDownLatch)).start();
        new Thread(new PreTaskThread("加载人物模型", countDownLatch)).start();
        new Thread(new PreTaskThread("加载背景音乐", countDownLatch)).start();
    }
}
```

### CyclicBarrier

> 栅栏，

`CyclicBarrirer`从名字上来理解是“循环的屏障”的意思。

前面提到了`CountDownLatch`一旦计数值`count`被降为0后，就不能再重新设置了，它**只能起一次“屏障”的作用**。

而`CyclicBarrier`拥有`CountDownLatch`的所有功能，还可以使用`reset()`方法重置屏障

```java
public class CyclicBarrierDemo {
    static class PreTaskThread implements Runnable {

        private String task;
        private CyclicBarrier cyclicBarrier;

        public PreTaskThread(String task, CyclicBarrier cyclicBarrier) {
            this.task = task;
            this.cyclicBarrier = cyclicBarrier;
        }

        @Override
        public void run() {
            // 假设总共三个关卡
            for (int i = 1; i < 4; i++) {
                try {
                    Random random = new Random();
                    Thread.sleep(random.nextInt(1000));
                    System.out.println(String.format("关卡%d的任务%s完成", i, task));
                    cyclicBarrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
                cyclicBarrier.reset(); 
                // 重置屏障
            }
        }
    }

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(3, () -> {
            System.out.println("本关卡所有前置任务完成，开始游戏...");
        });

        new Thread(new PreTaskThread("加载地图数据", cyclicBarrier)).start();
        new Thread(new PreTaskThread("加载人物模型", cyclicBarrier)).start();
        new Thread(new PreTaskThread("加载背景音乐", cyclicBarrier)).start();
    }
}
```

## CopyOnWriteArrayList

并发编程时，使用`ArrayList`会遇到`Concurrent Modification Exception`并发修改异常，表明ArrayList不能在并发开发中使用

```java
public static void main(String[] args) {
    final List<Integer> list = new ArrayList<>();
    //      1. 使用vector List<Integer> list = new Vector<>();
    //      2. 使用 List<Integer> list = Collections.synchronizedList(new ArrayList<>());
    //      3. CopyOnWriteArrayList<Integer> list = new CopyOnWriteArrayList<>();
    Runnable runnable = new Runnable() {
        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                list.add(i);
                System.out.println(list);
            }
        }
    };
    new Thread(runnable).start();
    new Thread(runnable).start();
    // ConcurrentModificationException 抛出
}
```

为了避免这个异常，我们有三种解决办法：

1. 使用`Vector`，这个类是线程安全的，但是效率极低
2. 使用集合类的`synchronizedList`方法
3. 使用`CopyOnWriteArrayList`，这是最佳的方法

CopyOnWrite：写入时复制（COW 计算机程序设计领域的一种优化策略）

## ThreadLocal

> ThreadLocal 线程本地变量：**在同一线程，不同组件之间传递数据**

当我们遇到这种情况：线程设置的变量只有自己读取（即保证线程隔离）

我们可以使用ThreadLocal也可以使用Synchronized，区别在于ThreadLocal并没有加锁，它的执行速度与效率会远远高于Synchronized。

Synchronized与ThreadLocal的区别：

- Synchronized
  - 使用时间换空间
  - 目的在于保证多个线程在操作共享资源时的顺序。
- ThreadLocal
  - 使用空间换时间
  - 目的在于保证多线程中数据隔离

**作用**：主要是实现了**数据隔离**，不同线程之间不会互相干扰

### ThreadLocal的使用

`ThreadLocal`的使用非常简单，例如这个demo

```java
ThreadLocal<String> threadLocal = new ThreadLocal<>();
threadLocal.set("存值");
String s = threadLocal.get();
threadLocal.remove();
```

主要使用的方法就四个：构造方法、`set`、`get`、`remove`

ThreadLocal本身并不存储值，而是将值存储在Thread类的ThreadLocalMap中，**ThreadLocalMap的key是ThreadLocal对象本身（注意：并不是Thread对象，而是ThreadLocal对象！！）**，value就是我们设置的值。

在看下面的例子：下面这个例子使用同一个ThreadLocal，互相之间不干扰不能获取到别人的值。

```java
ThreadLocal<Integer> threadLocal = new ThreadLocal<>();
threadLocal.set(15);
System.out.println(threadLocal); // java.lang.ThreadLocal@14ae5a5

Thread t1 = new Thread(() -> {
    System.out.println(threadLocal); // java.lang.ThreadLocal@14ae5a5 确实是同一个对象
    System.out.println(threadLocal.get()); // null 但是获取不到值
    threadLocal.set(20);
    System.out.println(threadLocal.get()); // 20
});
t1.start();
t1.join(); // main线程等待t1线程执行完成后执行
System.out.println(threadLocal.get()); // 15
```

### ThreadLocal的定义

> ThreadLocal与Thread的关系

- Thread类内部定义了两个ThreadLocalMap的引用

```java
// Thread类
ThreadLocal.ThreadLocalMap threadLocals;
ThreadLocal.ThreadLocalMap inheritableThreadLocals;
```

- ThreadLocalMap的定义是在ThreadLocal类内部定义的

```java
// ThreadLocal类内部定义了静态内部类ThreadLocalMap
static class ThreadLocalMap {
    static class Entry extends WeakReference<ThreadLocal<?>> {
        // Entry是弱引用的，且key就是ThreadLocal本身
        Object value;
        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
    ...
}
```

- 使用ThreadLocal的set方法，实际上是使用了对应线程的ThreadLocalMap的set方法

```java
private void set(Thread t, T value) {
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        map.set(this, value);
    } else {
        createMap(t, value);
    }
}
```

- ThreadLocal会在第一次被使用时（get、set）去创建对应线程的ThreadLocalMap

### ThreadLocal的原理

每一个Thread维护一个ThreadLocal，每一个Thread含有一个ThreadLocalMap

这个Map的key是ThreadLocal实例本身，value是我们想要设置的值

![ThreadLocal的结构](http://img.yesmylord.cn//image-20230828213049002.png)



要想搞清楚`ThreadLocal`，首先看`Thread`类中含有两个属性均为`ThreadLocalMap`类型（这里先知道Thread类有这个属性即可）

```java
ThreadLocal.ThreadLocalMap threadLocals = null;
// 初始值均为Null
ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
// inheritableThreadLocals是为了实现父子线程间共享threadLocal数据而提供的
```

再来看`ThreadLocal`的构造方法，很简单，与默认构造一样：

```java
public ThreadLocal() {
}
```

在看`get()`方法：（下面的方法在ThreadLocal类中）

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t); 
    // 此处getMap返回了当前线程的ThreadLocal的值，如果为null，说明没有初始化，那么就初始化一下
    if (map != null) {
        // 如果不为null，说明有ThreadLocalMap，就去取值，由getEntry实现了真正的取值
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    // 如果这个值为null，说明还没初始化，就初始化一下
    return setInitialValue();
    // 这个方法点到最后，就是通过 new 创建了 LocalThreadMap
}
// get 方法中初始化 ThreadLocal 对象源码如下：
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        // 创建LocalThreadMap
        createMap(t, value);
    return value;
}
//最终创建ThreadLocal对象的代码如下：
void createMap(Thread t, T firstValue) {
    // 这里new了一个ThreadLocal，这里证明了ThreadLocalMap的key是ThreadLocal对象本身this
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

真正的`get`其实是由这里的`getEntry()`方法完成的：值得一提的是在ThreadLocalMap中是使用**开放地址法**处理哈希碰撞的

> 处理哈希碰撞的方法：
>
> - **开放地址法**：如果遇到哈希冲突，就重新寻找真正的存放数据的下标位置（重新计算哈希也有不同的方法）
>   - **线性探测（ThreadLocalMap就是这种方式）**：从此下标开始，挨个往下找
>   - 二次探测：探测步数是原始相隔位置的平方
>   - 再哈希法：用不同哈希函数再求一遍哈希值
> - **链地址法**：如果遇到哈希冲突，就拉一条链表出来。HashMap中就是使用这种方法进行处理的
> - **建立公共溢出区**：专门存放所有哈希碰撞后的数据

```java
private Entry getEntry(ThreadLocal<?> key) {
    // Entry对象 是ThreadLocalMap的一个对象，他类似于HashMap中的Entry，都是实际存储数据的位置
    int i = key.threadLocalHashCode & (table.length - 1);
   	// 获取哈希表中该值的下标
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        // 因为使用开放地址法，所以这里需要重新找下标
        return getEntryAfterMiss(key, i, e);
}


private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;// table是一个Entry数组
    int len = tab.length;

    while (e != null) {
        ThreadLocal<?> k = e.get();
        if (k == key)
            return e;
        if (k == null)
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
// 可以看到，寻找的方式是线性探索
private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
}
```

（`set()`方法实现的原理与`get()`方法差不多，就不再赘述；）

Entry代码如下，注意继承了**弱引用**类：

> **弱引用** —— 发现即回收
>
> 特点：
>
> - 只被弱引用关联的对象**只能生存到下一次 GC 发生为止**（无论内存是否足够）
> - 由于垃圾回收线程的优先级很低，所以不一定很快被回收掉；这种情况可以存活较长时间

```java
static class ThreadLocalMap {
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** 注意这里Entry继承了弱引用类*/
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            // 注意key是ThreadLocal本身！！
            super(k);
            value = v;
        }
    }
    ...省略代码...
}
```

### ThreadLocal的内存泄漏问题

ThreadLocalMap定义在ThreadLocal内，但是实际上是Thread的成员。

Thread内含有ThreadLocalMap的强引用，一般来说ThreadLocalMap的生命周期是与Thread一致的。

但由于thread一般是复用的，因此Thread一般不会消亡，也就导致ThreadLocalMap不会消亡，即使对应Entry的key，也就是ThreadLocal被GC回收掉，其value也会存在，也就导致了内存泄漏。

ThreadLocal被设计为弱引用，是为了减少内存泄漏带来的影响，但是不能消除内存泄漏。

避免内存泄漏的方式：

1. 底层设计有优化，在每次`get`、`set`操作时，会自动清除key（也就是ThreadLocal）为null的Entry。但如果一直没有调用`get`、`set`方法，那么还是会有内存泄漏的问题。
2. 养成良好习惯，每次使用完后手动调用`remove`方法

### InheritableThreadLocal

为了使得父子线程之间可以传递数据，引入了InheritableThreadLocal

```java
ThreadLocal<String> tl1 = new InheritableThreadLocal<>();
tl1.set("main");
new Thread(()->{
    System.out.println("thread:" + tl1.get());
}).start();
System.out.println("main:" + tl1.get());
```

输出结果：

```java
main:main
thread:main
```

但注意，InheritableThreadLocal和线程池共同使用的时候，会出现问题，比如：

```java
InheritableThreadLocal<String> tl = new InheritableThreadLocal<>();
tl.set("main1");
ExecutorService es = Executors.newFixedThreadPool(1);
System.out.println("main:"+tl.get()); // main:main1
es.execute(()->{
    System.out.println("thread:"+tl.get());
    // thread:main1
});
Thread.sleep(1000);
tl.set("main2");
es.execute(()->{
    System.out.println("thread:"+tl.get());
    // thread:main1
});
```

输出结果为：

```java
main:main1
thread:main1
thread:main1 // 看这里
main:main2
```

注意到`tl.set("main2");`值没有发生作用，这是为什么呢？

1. main线程设置为`main1`
2. 一开始线程池之中没有线程，在第一次使用线程池执行任务时，会创建线程
   - 子线程会将父线程的`InheritableThreadLocal`copy到子线程的`InheritableThreadLocal`
   - 此后，子线程的InheritableThreadLocal与父线程其实就没有关系了

3. main线程更改值为`main2`
4. 在下一次执行任务时，线程池的线程还是之前的线程（线程复用），由于其copy的map没有发生变化，因此其存的值还是`main1`

因此，**InheritableThreadLocal不能再线程池场景下使用**

### TransmittableThreadLocal

如果我们现在有一个场景，**需要在多个线程之间（线程池）进行通信**，可以使用TTL

注意：线程池需要使用TtlExecutors.getTtlExecutorService包裹：

```java
ExecutorService es = TtlExecutors.getTtlExecutorService(Executors.newFixedThreadPool(1));
```

此时再去运行，即正常：

```java
TransmittableThreadLocal ttl = new TransmittableThreadLocal();
ttl.set("main1");
ExecutorService es = TtlExecutors.getTtlExecutorService(Executors.newFixedThreadPool(1));
System.out.println("main:"+ttl.get());
es.execute(()->{
    System.out.println("thread:"+ttl.get());
});
Thread.sleep(1000);
ttl.set("main2");
es.execute(()->{
    System.out.println("thread:"+ttl.get());
});
Thread.sleep(1000);
System.out.println("main:"+ttl.get());
```

输出结果为：

```java
main:main1
thread:main1
thread:main2 // 看这里
main:main2
```

原理：TTL如何做到的？

核心有两点：

1. 如何实现线程之间共享数据？
   - 使用一个静态的`static ThreadLocal<Map>`，静态成员属于类，以实现线程池之间共享
2. 有线程更改数据，那么在什么时候进行数据传递？
   - 任务的执行都是一个`Runnable`接口实现的方法，TTL额外实现了一个集成Runnable接口的类，在线程池调用run方法之前进行ThreadLocal的copy，在执行完成后进行复原
   - 这也是为什么要使用`TtlExecutors.getTtlExecutorService`包裹一下线程池的原因

对应源码：`TtlRunnable`源码

```java
public void run() {
    Object captured = this.capturedRef.get();
    if (captured != null && (!this.releaseTtlValueReferenceAfterRun || this.capturedRef.compareAndSet(captured, (Object)null))) {
        // 执行前对该线程原本的ThreadLocal进行备份
        Object backup = Transmitter.replay(captured);
        try {
            // 执行
            this.runnable.run();
        } finally {
             // 执行后复原
            Transmitter.restore(backup);
        }

    } else {
        throw new IllegalStateException("TTL value reference is released after run!");
    }
}
```

> TL、ITL、TTL之间的关系

从左到右依次继承：InheritableThreadLocal继承了ThreadLocal，TransmittableThreadLocal继承了InheritableThreadLocal

---

### 总结

- `Thread`维护了一个Map`ThreadLocalMap`，这个Map的key是`ThreadLocal`，value是我们想要设置的值
- `ThreadLocalMap`是每个`Thread`都拥有的一个属性
- `ThreadLocalMap`是`ThreadLocal `线程的内部类
- `ThreadLocalMap`中Entry的key是`ThreadLocal`类，而且是弱引用（有内存泄露的风险）
- `ThreadLocalMap`中避免哈希碰撞的方法是**开放地址法 + 线性探索**
- `ThreadLocalMap`与`synchronized`都可以进行数据隔离，区别是ThreadLocal使用空间换时间，`synchronized`则相反

# 详解

此部分关于AQS、CountDownLatch、ReentrantLock等的源码级理解

## AQS

AQS抽象同步队列是一个抽象类，Java的ReentrantLock、CountDownLatch都是AQS实现的。

AQS提出一个这样的模型：一个共享变量state，以及一个双向链表CLH队列，每一个请求资源的线程，都会被封装成一个CLH队列的结点

### 关键实现

`state` 状态：

```java
private volatile int state;
// state 代表共享资源；可以看到其使用volatile修饰
```

有三个方法可以操作这个状态的值：

```java
protected final int getState() {
    return state;
}

protected final void setState(int newState) {
    state = newState;
}

protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

### 数据结构

AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点(Node)来实现锁的分配。其中Sync queue，即同步队列，是双向链表，包括head结点和tail结点，head结点主要用作后续的调度。

而Condition queue不是必须的，其是一个单向链表，只有当使用Condition时，才会存在此单向链表。并且可能会有多个Condition queue。

![AQS](https://img.yesmylord.cn//java-thread-x-juc-aqs-1.png)



**AQS有两个内部类Node和ConditionObject**

```java
final boolean transferForSignal(Node node) {
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;

    Node p = enq(node);// 将节点加入到同步队列
    int ws = p.waitStatus;
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        LockSupport.unpark(node.thread);
    return true;
}
```



### 锁共享方式

- 互斥锁：只有一个线程可以执行，比如ReentrantLock就是这样的实现
- 共享锁：多个线程可以同时执行，比如CountDownLatch

自定义同步器在实现时**只需要实现共享资源 state 的获取与释放方式即可**，至于具体线程等待队列的维护(如获取资源失败入队/唤醒出队等)，AQS已经在上层已经帮我们实现好了（**模版方法模式**）

```java
isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
    
tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
```

### Node的状态

AQS的每一个Node，都是一个Thread的包装，AQS为什么是一个双向链表，是因为AQS需要通过前继节点来判断当前节点的行动：

Node有这几种状态

```java
static final int CANCELLED =  1; // 指示当前线程被取消执行
static final int SIGNAL    = -1; // 指示后续线程需要被唤醒
static final int CONDITION = -2; // 指示当前线程需要等待条件
static final int PROPAGATE = -3; // 指示当前线程需要传播（共享锁中）
// 此外，还有 0 是初始化的状态
```

如果Node是非负数表示均不需要被激活

**注意**：特别注意

- **SIGNAL：意思表示，后续节点被park了，在当前节点在释放或是取消时，一定要唤醒后续节点**
- **CANCEL**：由于超时或是中断，此节点被取消执行了！
- **CONDITION**：表示处于条件队列中，在条件没有满足的时候，不会进入同步等待队列
- **PROPAGATE**：共享模式的特有状态，用于传播唤醒状态

### AQS为什么得用一个双向链表？为什么不用单向链表？

因为AQS中多次使用到了前继节点：

**使用前继节点的状态，来判断当前节点的行为**，比如前继节点为SIGNAL，就表示前继节点在释放或是取消的时候，千万要唤醒后继节点

SIGNAL状态也就意味着，**只有前继节点才能唤醒后继节点**

### tryAcquire与acquire方法

加锁实际是由底层的`acquire`方法实现的：

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && // 调用一次我们的实现类
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

![AQS获取锁的流程](https://img.yesmylord.cn//java-thread-x-juc-aqs-2.png)

1. 首先调用`tryAcquire`获取，如果能获取成功，直接结束，如果没能获取成功就执行2
2. 执行`addWaiter`方法，将当前的Thread包装为一个Node节点，放在链表的尾部，执行3
3. 调用`acquireQueued`，下面列出代码：
   - 不断轮训
   - 首先：获取当前Node的前继节点，如果前继节点为head并且获取到了资源，那么表示自己可以执行了
   - 然后：判断前继节点的运行状态，根据不同状态执行不同操作`shouldParkAfterFailedAcquire`
     - 如果前继节点的状态为CANCELLED（说明线程取消执行），则进行下一次循环
     - 如果前继节点状态为SIGNAL，就表示当前节点需要park（也就是进入等待状态），返回True
   - 如果需要park线程，那么就调用`parkAndCheckInterrupt`，也就是`LockSupport.park()`

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor(); // 获取前继节点
            if (p == head && tryAcquire(arg)) { // 如果前继节点为head，并且再次tryAcquire成功，就可以执行了
                setHead(node); // 执行当前的Thread
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) && // // 在获取锁失败后，是否要被park呢？
                parkAndCheckInterrupt()) // 把当前Node park掉，调用LockSupport.park(this);
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

```java
// 在获取锁失败后，是否要被park呢？
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL) // 如果前继节点还是等待被激活，那么就返回true
        return true;
    if (ws > 0) { // 非负数表示都不需要被激活，通常会是1，cancel状态
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0); // 让node前进，找到一个有效的前驱节点
        pred.next = node;
    } else {
        // 进入这里一定是PROPAGATE或是0，此时表示我们可以被激活，但是无需park
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

- **返回 `true` 的情况**：如果前驱节点的状态为 `SIGNAL`，则表明当前线程应该阻塞（`park`），等待前驱节点唤醒它。

- **返回 `false` 的情况**：如果前驱节点的状态不是 `SIGNAL`，则当前线程暂时不会阻塞，还需要进一步处理。可能是因为前驱节点被取消，需要跳过，或者前驱节点还没有设置为 `SIGNAL` 状态，需要进行状态更新。

### tryRelease与release方法

释放锁也是调用tryRelease，如果释放成功就唤醒后继节点

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) { // 释放成功
        // 保存头节点
        Node h = head; 
        if (h != null && h.waitStatus != 0) // 头节点不为空并且头节点状态不为0
            unparkSuccessor(h); //释放头节点的后继结点
        return true;
    }
    return false;
}
```

### AQS实现简单的独占锁

AQS内部类部分：

- tryAcquire：用cas判断当前状态，如果获取到了就设为独占setExclusiveOwnerThread
- tryRelease：首先判断状态是不是unlock，清除独占，设置state为unlock

```java
static class Sync extends AbstractQueuedSynchronizer {
    private  static final int UNLOCK = 0;
    private static final int LOCK = 1;
    @Override
    protected boolean tryAcquire(int acquires) {
        if (compareAndSetState(UNLOCK, LOCK)) {
            setExclusiveOwnerThread(Thread.currentThread());
            return true;
        }
        return false;
    }
    @Override
    protected boolean tryRelease(int releases) {
        if (getState() == UNLOCK) {
            throw new IllegalMonitorStateException("Lock is not held by the current thread.");
        }
        // 清除当前独占线程
        setExclusiveOwnerThread(null);
        setState(UNLOCK);
        return true;
    }
    public Condition newCondition() {
        return new ConditionObject();// ConditionObject与Node一样，都是AQS内部类，负责条件判断
    }
}
```

外部实现Lock：

```java
public class AqsLock implements Lock {
    private Sync sync;
    public AqsLock() {sync = new Sync();}
    @Override
    public void lock() {sync.acquire(1);}
    @Override
    public void unlock() {sync.release(1);}

    @Override
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }
    @Override
    public boolean tryLock() {
        return sync.tryAcquire(1);
    }
    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(time));
    }
    @Override
    public Condition newCondition() {
        return sync.newCondition();
    }
    static class Sync extends AbstractQueuedSynchronizer {
        // 省略
    }
}
```

### tryAcquireShared与acquireShared方法

在共享锁中，`state`的数字就有了意义，他表示资源的个数：

```java
tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
```

同独占锁的实现，共享锁也是这样：

```java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0) // 调用一次tryAcquireShared，如果失败
        doAcquireShared(arg);
}
```

如果获取失败，调用下面的方法，与独占锁的`acquireQueued`方法很像，他们的主要区别在于`setHeadAndPropagate()`方法

```java
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED); 
    // 以共享的方式包装一个Node，在独占锁中，Node是在外部构造的
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();// 获取前继节点，判断是否为head
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) { // 如果当前有剩余资源，那么就可以执行
                    setHeadAndPropagate(node, r); // 与独占锁的区别，还会传播唤醒下一个节点
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

与独占锁的区别就在于下面这个方法，独占锁直接`setHead(node)`，而共享锁还会尝试唤醒后继节点

```java
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head; // Record old head for check below
    setHead(node);
	
    // 判断后继是否需要传播唤醒
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next; // s指向后继节点
        if (s == null || s.isShared()) // 如果后继节点为空，或者后继节点是共享模式，就进行释放操作
            doReleaseShared();
    }
}
```

需要唤醒后面节点的条件有：

- `propagate > 0`：如果 `propagate` 大于 0，表示需要继续传播唤醒后续线程。
- `h == null`：如果旧的头节点 `h` 为空，说明队列之前为空，需要唤醒新的节点。
- `h.waitStatus < 0`：如果旧的头节点的状态是负数，表示头节点处于等待状态（通常为 `SIGNAL`），表明可能有后续节点需要被唤醒
- `h = head`：再次检查当前的头节点是否已经发生变化。
- `h.waitStatus < 0`：再次检查新的头节点状态。

### tryReleaseShared与releaseShared方法

```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```

```java
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;
                unparkSuccessor(h); // 在这里释放了wait的Node
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;
        }
        if (h == head)
            break;
    }
}
```

### AQS要点总结

1、AQS有两个内部类Node和ConditionObject
2、AQS同步队列是一个双向链表，条件队列是一个单向链表
3、AQS的Node包裹了一个Thread，只有head才有执行的权利，而且只有head才能唤醒后继节点
4、Node的SIGNAL状态表示，后继节点处于park，当前节点释放时，一定要激活后继节点（unpark后继节点）
5、Condition满足后，会调用transferForSignal从条件队列转移到同步队列
6、acquire的流程为：tryAcquire（尝试获取一次）->addWaiter（包装为Node添加到队列）->acquireQueued（判断前继节点状态，决定当前节点行为）
7、当前继节点为SIGNAL，表示当前节点会在前继节点释放时被唤醒；前继节点为CANCEL，表示会直接跳过前继节点；前继节点propagate，表示处于传播状态，可以无条件唤醒后继节点（用于共享模式）；
8、AQS有两种同步方式：共享和独占。独占的实现类有ReentrantLock，共享的实现类有CountDownLatch。
9、共享模式实现方式与独占差不多，区别在于是否可以传播唤醒后继节点



## CountDownLatch

### 问题

> 1、CountDownLatch适用于什么场景？

CountDownLatch典型的用法是将一个程序分为n个互相独立的可解决任务，并创建值为n的CountDownLatch。

> 2、CountDownLatch的实现原理是什么？

当每一个任务完成时，都会在这个锁存器上调用countDown，等待问题被解决的任务调用这个锁存器的await，将他们自己拦住，直至锁存器计数结束

这个计数器是用AQS的state实现的，使用了AQS的共享模式，每次获取资源都调用tryAquireShared模式，每次释放资源都调用tryReleaseShared。

### Demo：七颗龙珠召唤神龙

当某一个线程需要等待N个线程执行完成时，就使用

```java
public class DragenBall implements Runnable{
    CountDownLatch cdl;

    @Override
    public void run() {
        System.out.println("已收集第"+no+"颗龙珠");
        cdl.countDown();
    }
    int no;

    public DragenBall(int no, CountDownLatch cdl) {
        this.no = no;
        this.cdl = cdl;
    }

    public static void main(String[] args) throws InterruptedException {
        CountDownLatch cdl = new CountDownLatch(7);

        for (int i = 0; i < 7; i++) {
            DragenBall dragenBall = new DragenBall(i, cdl);
            new Thread(dragenBall).start();
        }
        cdl.await();
        System.out.println("召唤神龙");
    }
}

```

### 实现原理

利用了**AQS的共享模式**，内部使用state作为count的数量，每次CountDown就释放一个资源

```java
tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
```

CountDownLatch有一个内部类继承了AQS：

```java
private static final class Sync extends AbstractQueuedSynchronizer {
    Sync(int count) {
        setState(count);
    }

    int getCount() {
        return getState();
    }

    protected int tryAcquireShared(int acquires) {
        return (getState() == 0) ? 1 : -1;
    }

    protected boolean tryReleaseShared(int releases) {
        // Decrement count; signal when transition to zero
        for (;;) {
            int c = getState();
            if (c == 0)
                return false;
            int nextc = c-1;
            if (compareAndSetState(c, nextc))
                return nextc == 0;
        }
    }
}
```

CountDownLatch的其他部分很简单：关键看`await()`与`countDown`实现

```java
public class CountDownLatch {
    private static final class Sync extends AbstractQueuedSynchronizer {
        // 省略
    }
    private final Sync sync;
    
    public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);
    }

    public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }

    public boolean await(long timeout, TimeUnit unit)
        throws InterruptedException {
        return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
    }

    public void countDown() {
        sync.releaseShared(1);
    }

    public long getCount() {
        return sync.getCount();
    }

    public String toString() {
        return super.toString() + "[Count = " + sync.getCount() + "]";
    }
}
```

## ReentrantLock

### 问题

> 1、ReentrantLock是什么锁？

可重入的、可公平可不公平、悲观锁

> 2、ReentrantLock是如何实现可重入的？

通过state状态维护可重入次数，每次tryAcquire时：

- 如果没有被占据，就获得锁
- 如果被占据，判断是不是当前线程占据，如果是的话，就将state+acquire次数，就实现了重入

> 3、ReentrantLock是如何实现公平与非公平的？

- 非公平锁：tryAcquire时，直接cas判断（直接抢占）
- 公平锁：tryAcquire时，会先判断head后是否有线程，如果有，就说明存在比自己等待时间长的线程。

> 4、对比Synchronized的有什么区别？

- `Synchronized`：加锁`syn(obj)`、进入等待`obj.wait()`、`obj.notify()`、`obj.notifyAll()`
  - **实现方式**：加锁解锁JVM自动实现，通过JVM内部的monitor对象，
  - **可重入**：调用字节码`monitor_enter`和`monitor_exit`实现可重入
  - **公平锁与非公平锁**：只支持非公平锁
  - **条件**：条件只能使用obj进行判断，不支持多个条件
  - **锁粒度**：1.5之前直接为重量级锁，之后引入了锁升级过程，但还是独占锁，读写不分离
- `ReentrantLock`：加锁`lock()`、解锁`unlock()`、进入等待`condition.await()`、唤醒`condition.signal()`
  - **实现方式**：加锁解锁使用AQS，同步队列+条件队列
  - **可重入**：`state`状态存储重入次数，tryAcquire时判断是否是当前线程重入
  - **公平锁与非公平锁**：支持公平锁，默认非公平锁
  - **条件**：通过`ConditionObject`对象及条件队列实现，满足条件后，调用`signal()`将Node传送给同步队列
  - **锁粒度**：独占锁，读写不分离

### 内部结构

内部集成关系，由一个静态内部类继承AQS（大部分都是这么实现的），然后又有两个静态内部类分别集成Sync，实现公平锁与非公平锁。

![ReentrantLock内部结构](https://pdai.tech/images/thread/java-thread-x-juc-reentrantlock-1.png)

### Sync类

Sync类实现了非公平的获取方式，tryAcquire在子类中实现。

下面是源码，可以看到可重入的实现方式：

- 如果没有线程占据锁，那么就获得
- 如果当前线程就是获取了锁的线程`current == getExclusiveOwnerThread()`，可以再次获得，将`state+acquires`次数

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
    abstract void lock();

    // Sync类实现了非公平的获取方式，tryAcquire在子类中实现
    final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) { // 如果当前没有线程占据
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            // 如果占据的线程就是当前线程（实现了可重入）
            int nextc = c + acquires;
            if (nextc < 0) // overflow，可重入大小不能超过Integer.MAX_VALUE
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }

    protected final boolean tryRelease(int releases) {
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread()) 
            // 如果当前占锁的线程不是调用release的线程，就报错了
            throw new IllegalMonitorStateException();
        boolean free = false;
        if (c == 0) { // 重入次数为0时，才可以释放锁
            free = true;
            setExclusiveOwnerThread(null);
        }
        setState(c);
        return free;
    }
	
    protected final boolean isHeldExclusively() {return getExclusiveOwnerThread() == Thread.currentThread();}
    final ConditionObject newCondition() {return new ConditionObject();}
    final Thread getOwner() {return getState() == 0 ? null : getExclusiveOwnerThread();}
    final int getHoldCount() {return isHeldExclusively() ? getState() : 0;}
    final boolean isLocked() {return getState() != 0;}
}
```

### NonFairSync

NonfairSync直接调用Sync的获取方式即可

```java
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = 7316153563782823691L;

    final void lock() {
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }

    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);// 调用父类方法
    }
}
```

### FairSync

公平锁如何实现？

在tryAcquire方法中，并不像非公平锁一样，直接调用cas，而是**先判断当前同步队列是否有节点**

（如果有节点，就说明该线程等待的时间比当前线程时间要长）

```java
static final class FairSync extends Sync {
    final void lock() {
        acquire(1);
    }

    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (!hasQueuedPredecessors() && // 这里判断
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```

这里看下hasQueuedPredecessors方法：

如果当前head后有节点（说明有线程在等待，且不是当前线程在等待），那么返回true

```java
public final boolean hasQueuedPredecessors() {
    Node t = tail; // Read fields in reverse initialization order
    Node h = head;
    Node s;
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

## ReentrantReadWriteLock

### 问题

> 1、为什么有ReentrantLock还要引入ReentrantReadWriteLock

对于大部分操作来说，读写的要求是不一致的，读锁可以一起读，写锁只能一个线程写，而ReentrantLock是悲观锁，不论读写都会锁住，不利于提高读操作的并发性。

> 2、如何实现的读写分离？

ReentrantReadWriteLock在内部实现了读锁与写锁，读锁使用共享模式的AQS，写锁使用独占模式的AQS。

- 写锁：状态存放在AQS的state的低16位，如果低16位不为0，表示写锁已被获取
- 读锁：读锁状态存放的位置有state的高16位以及ThreadLocalHoldCounter的HoldCounter，他存放了所有线程的读锁数量。

获取写锁时，是如何判断有没有读锁的？通过判断state和写锁（低16位），如果state==0但是写锁数不为0，那么就存在读锁。

> 3、本地线程计数器ThreadLocalHoldCounter是做什么的？

存放不同的线程的**读锁的计数状态**（没有存放写锁的状态）

> 4、缓存计数器cachedHoldCounter是做什么的？

避免每次都去读取ThreadLocal，存放当前线程的读锁计数，是一个优化机制

> 6、支持锁升级吗？为什么？

不支持，为了避免：

1. 防止死锁：如果有多个线程获取了读锁，然后想获取写锁，都会等待对方释放读锁，就会形成死锁问题
2. 为了避免数据不一致：多个线程获取了读锁，一个线程还获取了写锁，那么这个线程的写入操作对其他线程不可见

> 7、支持锁降级吗？为什么？

支持锁降级，可以保证数据一致性，获取写锁后获取读锁，再释放写锁，可以保证前后读取到的数据一致，不会有其他线程进行更改。

### 内部结构

![可重入读写锁的结构](https://pdai.tech/images/thread/java-thread-x-readwritelock-1.png)

内部有五个类：Sync、Fair、NonfairSync；Lock、ReadLock、WriteLock

### ReadLock与WriteLock

先从读写锁开始介绍，因为ReentrantReadWriteLock本质是一个锁

- ReadLock：调用Sync的acquireShared与releaseShared
- WriteLock：调用Sync的acquire与release

均调用了Sync，重点去看Sync的获取方法

```java
public static class WriteLock implements Lock, java.io.Serializable {
    private final Sync sync;
    public void lock() {
        sync.acquire(1);
    }
    public boolean tryLock( ) {
        return sync.tryWriteLock();
    }
    public void unlock() {
        sync.release(1);
    }
    public void unlock() {
            sync.releaseShared(1);
        }
}
```

```java
public static class ReadLock implements Lock, java.io.Serializable {
    private static final long serialVersionUID = -5992448646407690164L;
    private final Sync sync;
    public void lock() {
        sync.acquireShared(1);
    }
    public boolean tryLock() {
        return sync.tryReadLock();
    }
```

### Sync

Sync同样是一个继承了AQS的静态内部类，其内部还有两个类：

- HoldCount：一个计数器，专门用于计算读锁的个数
- ThreadLocalHoldCounter：继承了ThreadLocal，而且存储HoldCount

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
    // 省略成员和方法
    static final class HoldCounter {
        int count = 0;
        // 存储线程id而不是引用，为了避免留下垃圾
        final long tid = getThreadId(Thread.currentThread());
    }
    static final class ThreadLocalHoldCounter
        extends ThreadLocal<HoldCounter> {
        // 重写了initialValue，表示即使没有set，get时返回的也是HolderCount数量
        public HoldCounter initialValue() {
            return new HoldCounter();
        }
    }
}
```

Sync如何进行计数？**AQS的state中的高16读低16写**

```java
static final int SHARED_SHIFT   = 16;
static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1; // 最大次数为2^16-1
static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;
// 高16位表示读锁计数
static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
// 低16为表示写锁计数
static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }
```

Sync构造时，就会创建本地线程计数器**ThreadLocalHoldCounter**

```java
private transient ThreadLocalHoldCounter readHolds;
private transient HoldCounter cachedHoldCounter;
private transient Thread firstReader = null;
private transient int firstReaderHoldCount;

Sync() {
    readHolds = new ThreadLocalHoldCounter();
    setState(getState()); // ensures visibility of readHolds
}
```

这里中断介绍一下四个成员：

- readHolds：ThreadLocal对象，保存了每一个线程的HoldCounter，也就是保存了每个线程的读锁的个数
- cachedHoldCounter：缓存当前线程的读锁个数（因为查找ThreadLocal会有开销）
- firstReader：跟踪第一个获取读锁的线程，是一个优化手段
- firstReaderHoldCount：第一个读取锁线程的读锁个数，配合firstReader使用

### Sync的写实现

写锁调用Sync的acquire与release：

写锁的实现与普通的独占锁实现基本一致，有几个比较关键的点：

- 如果当前state不为0，但是写锁为0，表示有读锁，那么直接返回false
- 多了一个判断当前是否存在独占锁的逻辑，是否存在独占锁是通过判断state的低16位判断的。而且不允许数量超过2^16

```java
protected final boolean tryAcquire(int acquires) {
    Thread current = Thread.currentThread();
    int c = getState();
    int w = exclusiveCount(c); // state低16表示写锁数量
    if (c != 0) { // 存在读锁或是写锁
        // 写锁为0（写锁为0，c不为0，表示读锁不为0） 或 当前线程没有独占资源
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        // 写锁+要求的资源数>2^16-1，报错
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        // 获取到写锁
        setState(c + acquires);
        return true;
    }
    if (writerShouldBlock() ||  // writerShouldBlock对公平和非公平锁操作不同
        !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```

 writerShouldBlock：

- 对于公平锁操作，会判断同步队列头部是否有线程等待`hasQueuedPredecessors`
- 对于非公平锁，直接返回false，写锁应该一直允许抢占

写锁的释放

```java
protected final boolean tryRelease(int releases) {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    int nextc = getState() - releases;
    boolean free = exclusiveCount(nextc) == 0;
    // 如果释放完全，那么清空独占
    if (free)
        setExclusiveOwnerThread(null);
    setState(nextc);
    return free;
}
```

### Sync的读实现

读锁（共享锁）调用Sync的acquireShared与releaseShared：

```java
protected final int tryAcquireShared(int unused) {
    Thread current = Thread.currentThread();
    int c = getState();
    if (exclusiveCount(c) != 0 && // 如果存在写锁
        getExclusiveOwnerThread() != current) // 并且不是自己的写锁
        return -1; // 返回-1 表示失败
    int r = sharedCount(c); // 获取高16位，当前读锁的个数
    if (!readerShouldBlock() &&
        r < MAX_COUNT && // 读锁也得小于MAX_COUNT
        compareAndSetState(c, c + SHARED_UNIT)) { // 加读锁
        if (r == 0) { // 如果当前没有读锁
            firstReader = current; // firstReader指向第一个获取读锁的Thread
            firstReaderHoldCount = 1; // 读锁计数
        } else if (firstReader == current) { // 如果第一个获取锁的线程又来获取读锁
            firstReaderHoldCount++;
        } else {
            HoldCounter rh = cachedHoldCounter; // 获取
            if (rh == null || rh.tid != getThreadId(current))
                // 如果缓存为空或是不匹配，就要去ThreadLocal里面找
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                // 如果为0，说明当前线程之前没有获取过读锁。
                // 此时，需要重新设置，确保ThreadLocal中保存的计数器是最新的
                readHolds.set(rh);
            rh.count++; // 成功获取到读锁
        }
        return 1; // 获取成功
    }
    // 执行下面代码的情况有：CAS失败或是当前head后有线程，会进行额外的逻辑进行CAS操作，不再赘述
    return fullTryAcquireShared(current);
}
```

 readerShouldBlock：

- 对于公平锁操作，会判断同步队列头部是否有线程等待`hasQueuedPredecessors`（与写锁一样）
- 对于非公平锁，会判断同步队列第一个线程是否是独占锁，如果是返回true

firstReader：

- firstReader是一个Thread的指针，指向第一个获取读锁的线程

- firstReaderHoldCount是获取读锁的锁数量的计数

使用firstReader与firstReaderHoldCount是为了在一般情况下，避免遍历ThreadLocal的开销

cachedHoldCounter：

- `cachedHoldCounter` 保存当前线程的读锁个数

值得注意的是，如果线程已经获取了写锁，那么依然可以获取读锁，这意味着ReentrantReadWriteLock支持**锁降级**

### 锁升级与锁降级

锁升级与锁降级指的是：

- 锁升级：在已有读锁的情况下，获取写锁，然后释放读锁
- 锁降级：在已有写锁的情况下，获取读锁，然后释放写锁

> 锁降级的好处：

对于锁降级来说，如果我们先释放写锁，在获取读锁，那么这个过程可能数据就会变动，造成前后数据读取不一致，因此锁降级可以支持前后数据读取一致性

**ReentrantReadWriteLock支持锁降级，但不支持锁升级**，原因是：如果当前有很多线程持有读锁，其中一个线程进行了锁升级，那么他的写入改动，对其他已经获取读锁的线程是不可见的。

## Excutor的四种线程池实现

- SingleThreadExecutor：1个核心线程和最大线程，阻塞队列无限

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

- newFixedThreadPool：固定大小，核心与最大线程相同，阻塞队列也是无限

```java
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>(),
                                  threadFactory);
}
```

- CachedThreadPool：0核心数量，但是最大线程无限，适合于短期的大量短任务，60秒后会自动释放线程

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

如果任务数量过多且执行速度较慢，可能会创建大量的线程，从而导致 OOM

SynchronousQueue是一个继承了AQS的同步队列，没有容量，不存储元素。

- ScheduledThreadPool：最大线程数也是无限，阻塞队列是DelayedWorkQueue也无界

```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue());
}
```

线程池总结：

- SingleThreadExecutor、newFixedThreadPool、ScheduledThreadPool的阻塞队列都是无界的，如果请求堆积，容易引起OOM
- CachedThreadPool的阻塞队列虽然不存储元素，但是他的最大线程数无限，如果短期来了大量任务且执行时间长，也会出现OOM

线程池用的阻塞队列总结：

- LinkedBlockingQueue，无界队列，如果任务堆积有可能OOM
- SynchronousQueue：同步队列，是AQS的实现，不存储元素
- DelayedWorkQueue：延迟阻塞队列，内部是一个堆，按照执行时间排序，会自动扩容，也是无界的

## ThreadPoolExecutor

### 核心数据结构

核心数据结构是一个阻塞队列+Worker的hashset

```java
private final BlockingQueue<Runnable> workQueue; // 阻塞队列
private final HashSet<Worker> workers = new HashSet<Worker>();// 线程池是一个hashSet
// Worker就是一个线程
```

![线程池](https://img.yesmylord.cn//java-thread-x-executors-1.png)

worker就是一个**继承了AQS、实现了Runnable接口的包裹了Thread的内部类**：

```java
private final class Worker
    extends AbstractQueuedSynchronizer // AQS
    implements Runnable // Runnable
{
    final Thread thread; // 线程final复用
    Runnable firstTask; // 使用第一个任务来初始化Worker
    volatile long completedTasks;

    Worker(Runnable firstTask) {
        setState(-1); // 创建线程后的state状态是-1，不允许在初始化未完成前被中断
        this.firstTask = firstTask;
        // 使用线程工程
        this.thread = getThreadFactory().newThread(this);
    }
	
    public void run() {
        runWorker(this); // 调用ThreaPoolExecutor的runWorker执行
    }

    // 下面都是通过AQS来加锁的方法，实现方式与AQS简单独占锁相同，此处省略
    protected boolean isHeldExclusively() {}
    protected boolean tryAcquire(int unused) {}
    protected boolean tryRelease(int unused) {}
    public void lock()        { acquire(1); }
    public boolean tryLock()  { return tryAcquire(1); }
    public void unlock()      { release(1); }
    public boolean isLocked() { return isHeldExclusively(); }

    void interruptIfStarted() {
        Thread t;
        if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
            try {
                t.interrupt();
            } catch (SecurityException ignore) {
            }
        }
    }
}
```

### ctl状态

线程池使用一个AtomicInteger来控制状态，它包括两个概念**workercount**和**runState**

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
```

- workercount：线程池最大数量是2^29-1（高三位表示状态）
- runState：高三位表示状态

```java
private static final int COUNT_BITS = Integer.SIZE - 3; // 32 - 3 = 29
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// Packing and unpacking ctl
private static int runStateOf(int c)     { return c & ~CAPACITY; }
private static int workerCountOf(int c)  { return c & CAPACITY; }
private static int ctlOf(int rs, int wc) { return rs | wc; }
```

线程池的状态有：

```java
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;// 不会接受新任务，但还会处理队列任务
private static final int STOP       =  1 << COUNT_BITS;// 不接受新任务，忽略队列任务
private static final int TIDYING    =  2 << COUNT_BITS;// 所有任务都已终止
private static final int TERMINATED =  3 << COUNT_BITS;// terminated()方法已经执行完成
```

![线程池状态](https://img.yesmylord.cn//java-thread-x-executors-2.png)



### execute方法

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();

    int c = ctl.get();
    // 当前数量小于核心线程
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true)) // 参数为true，表示创建核心线程去执行任务
            return;
        c = ctl.get();
    }
    // 如果线程池仍在运行，并且任务队列可以添加任务
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        // 如果线程池没有运行，就会移除刚增加的任务，并且拒绝
        if (! isRunning(recheck) && remove(command))
            reject(command);
        // 如果线程池没有线程，就启动一个新的非核心线程来处理
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 如果任务不能加入队列（可能是队列满了）那么使用非核心线程执行
    else if (!addWorker(command, false))
        reject(command);
}
```

### addWorker方法

addWorker(command, true)：传入两个参数，分别是Runnable任务与bool，为true表示创建核心线程执行，为false表示创建非核心线程执行。

- 创建新线程需要全局锁ReentrantLock

详细代码如下：

```java
private boolean addWorker(Runnable firstTask, boolean core) {
    retry: // 这里还用了带标签的break方法
    // 外层无限循环，用于在某些条件下重试添加工作线程
    for (;;) {
        int c = ctl.get();  // 获取线程池的状态和当前工作线程数
        int rs = runStateOf(c);  // 获取线程池的运行状态（如RUNNING, SHUTDOWN等）

        // 只有在必要时才检查队列是否为空。
        // 如果线程池已SHUTDOWN且队列非空，且没有传入任务，则返回false
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        // 内层循环用于尝试增加工作线程计数，可能因竞争失败需要重试
        for (;;) {
            int wc = workerCountOf(c);  // 获取当前的工作线程数
            // 如果工作线程数超过限制（CAPACITY或核心线程数/最大线程数），则返回false
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            // 使用CAS操作尝试增加工作线程计数
            if (compareAndIncrementWorkerCount(c))
                break retry;  // 成功增加工作线程计数，退出外层循环
            c = ctl.get();  // 如果CAS失败，重新获取ctl
            // 如果在尝试期间线程池的运行状态发生了变化，重新尝试
            if (runStateOf(c) != rs)
                continue retry;  // 重新开始外层循环
            // 否则，CAS因工作线程计数变化而失败，重新尝试内层循环
        }
    }

    // 以下代码用于真正创建并启动工作线程
    boolean workerStarted = false;  // 标记工作线程是否成功启动
    boolean workerAdded = false;  // 标记工作线程是否成功添加
    Worker w = null;  // Worker对象表示一个工作线程及其任务
    try {
        w = new Worker(firstTask);  // 创建一个新的Worker对象，并绑定初始任务
        final Thread t = w.thread;  // 获取Worker对应的线程对象
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();  // 获取主锁，确保线程池的一致性操作
            try {
                // 在持有锁的情况下重新检查状态
                // 如果线程池仍然在运行或是SHUTDOWN且没有初始任务，继续
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    // 如果线程已经启动（不应该发生），抛出异常
                    if (t.isAlive())
                        throw new IllegalThreadStateException();
                    workers.add(w);  // 将新Worker添加到工作线程集合中
                    int s = workers.size();  // 获取当前工作线程集合的大小
                    if (s > largestPoolSize)  // 更新最大线程池大小
                        largestPoolSize = s;
                    workerAdded = true;  // 标记工作线程已成功添加
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                t.start();  // 启动工作线程！！！！
                workerStarted = true;
            }
        }
    } finally {
        if (!workerStarted)
            addWorkerFailed(w);  // 如果线程未启动成功，执行失败处理
    }
    return workerStarted;  // 返回是否成功启动工作线程
}
```

在addWorker里调用了`t.start()`方法，也就是调用worker的run方法！

### runWorker方法

worker是实现了Runnable接口的，在run方法中调用了runworker方法：

```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // 将state从-1变为0，就可以支持中断了
    boolean completedAbruptly = true;
    try {
        while (task != null || (task = getTask()) != null) {// 这里的getTask见下文
            w.lock(); // aqs上锁
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                beforeExecute(wt, task); // 可以进行执行任务前的操作
                Throwable thrown = null; // 执行额外操作可能会抛出异常
                try {
                    task.run(); // 真正执行
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock(); // aqs解锁
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```

-  `w.unlock()`：在创建完worker的时候，设置了state为-1（看构造函数），是为了防止在初始化未完成前被中断，这里unlock，解锁，将aqs的state变为0。此时就支持了被中断的能力

-  `getTask()`：会获取当前阻塞队列的任务，也有判断线程是否销毁的逻辑
-  ` task.run();`：真正执行run方法的位置

### getTask方法

下面看一下`getTask`，是如何获取队列任务，并且抛出异常的：

- `allowCoreThreadTimeOut`：是否允许核心线程过期？可以设置这个值，让核心线程也去销毁
- `workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS)`：在keepAliveTime时间内阻塞，除非获取到元素
- `workQueue.take()`：一直阻塞，直到可以获取出数据

> 线程是如何超时被销毁的？

在getTask方法中通过`workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS)`获取任务，如果任务为空，就标记超时为true，在下一次循环中就会销毁线程

```java
private Runnable getTask() {
    boolean timedOut = false; // 标记上一次poll()是否超时

    for (;;) { // 无限循环，直到获取到任务或决定终止线程
        int c = ctl.get();
        int rs = runStateOf(c);

        // 如果线程池处于SHUTDOWN或更高状态，并且队列为空或池状态至少为STOP
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount(); // 减少工作线程计数
            return null; // 返回null，表示没有任务，线程将退出
        }

        int wc = workerCountOf(c);

        // 判断工作线程是否需要根据超时策略进行回收
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

        // （如果工作线程数大于最大线程数 或（线程允许超时且上次操作超时））且 （当前有多于1个工作线程或队列为空）
        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            // 尝试减少工作线程计数
            if (compareAndDecrementWorkerCount(c))
                return null; // 返回null，表示线程将退出
            continue; // 如果未能成功减少工作线程计数，重新尝试
        }
        
        try {
            // 根据是否允许超时，决定从队列中获取任务的方式
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) : // 带超时的poll
                workQueue.take(); // 阻塞式获取任务
            if (r != null)
                return r; // 如果获取到任务，返回任务
            timedOut = true; // 如果poll超时且未获取到任务，标记为超时
        } catch (InterruptedException retry) {
            timedOut = false; // 如果线程被中断，重置超时标记，并重新尝试获取任务
        }
    }
}
```

### 线程池的关闭问题

线程池有shutdown()和shutdownNow()方法，原理是**遍历线程池中规定工作线程，然后逐个调用线程的`interrupt`方法**来中断。

- shutdown：不会立即停止，会停止接受外部任务，等待队列的任务执行完成后，才停止
- shutdownNow：停止接受任务，忽略队列等待的任务，尝试中断正在执行的任务（不一定会立马停止）

### 线程池总结

- ThreadPoolExecutor的结构：
  - 一个`AtomicInteger`的状态，高3bit表示线程池状态，低29位表示当前线程池数量
  - 一个`HashSet<Worker>`，线程池存储线程使用了一个set
  - `BlockingQueue<Runnable>`，任务的阻塞队列
  - Worker：其内部有一个线程，还是一个继承了AQS，实现了Runnable接口的类，既可以保证自己给自己加锁，又能执行任务，他的run方法的实现调用了`runWorker`方法，
- 调用链路：ThreaPoolExecutor中的executor方法->addWorker方法->getTask获取任务
  - `executor(Runnable command)`方法：根据不同逻辑（核心线程数量与当前线程数量的关系），调用addWorker方法
  - `addWorker`方法：addWorker负责Worker的线程的创建逻辑，创建时会以ReentrantLock加锁的逻辑，线程由线程工厂创建，创建完成后，将`worker`添加到`set`内，最后会执行`t.start()`方法，也就是执行了run方法，也就是执行了runWorker方法。
  - `runWorker`方法：runWorker方法真正调用了任务的run方法，并且可以添加一些前后的处理逻辑，执行时后不断调用getTask方法获取任务，执行任务时，使用worker的AQS lock与unlock方法进行加锁。
  - `getTask`方法：获取阻塞队列的任务，主要通过两个方法poll与take，poll中有保活时间，如果获取到的任务为null，说明当前线程空闲，下一次循环就会被销毁。而且还可以设置allowCoreThreadTimeOut为true，核心线程也会在空闲时被销毁。

简而言之：线程池的工作流程是，在任务传入后，调用executor方法，executor方法会判断线程数与核心线程数的关系，考虑是否创建Worker，创建Worker调用addWorker方法，创建后会将Worker放入线程池set内，然后执行线程的start方法，由于Worker实现了Runnable接口，重写了run方法，也就是调用了runWorker方法，runWorker方法会不断的调用getTask获取任务，getTask方法通过阻塞队列的poll与take方法获取任务，其中poll方法有保活时间，如果保活时间内都没有获取到任务，说明当前线程空闲，就会被销毁。

## 相关链接

1. [狂神说JUC编程](https://www.bilibili.com/video/BV1B7411L7tE?p=5)
2. [大佬猿人谷blog](https://yuanrengu.com/2020/7691e770.html)
3. [敖丙ThreadLocal](https://www.zhihu.com/question/341005993)
4. [我自己的博客](https://www.yesmylord.cn/2021/07/26/JVM/%E6%B7%B1%E5%85%A5Java%E8%99%9A%E6%8B%9F%E6%9C%BAGC%E7%AF%87/)
5. [敖丙锁升级](https://blog.csdn.net/qq_35190492/article/details/104691668?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162924889716780261918067%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=162924889716780261918067&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_v2~hot_rank-1-104691668.first_rank_v2_pc_rank_v29&utm_term=aqs&spm=1018.2226.3001.4187)
6. [RedSpider社区](http://concurrent.redspider.group/article)

