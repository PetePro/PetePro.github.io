---
layout: post
title:  "【Java】Java 05 Java并发编程"
crawlertitle: "Java并发编程"
summary: "Java Concurrency"
date:   2020-03-15 09:00:00 +0800
categories: posts
tags: 'CS'
author: xusc
bg: "CS.jpg"
---

`并发编程`是 Java 的重要特性之一，在 Java 平台上提供了许多基本的并发功能来辅助开发多线程应用程序。

多线程的价值
- 发挥多处理器的能力
- 建模的简单性
- 异步事件的简化处理
- 响应更灵敏的用户界面

### 0 目录
- [0 目录](#0-目录)
- [1 Java线程](#1-java线程)
  - [1.1 启动和终止线程](#11-启动和终止线程)
  - [1.2 线程状态](#12-线程状态)
  - [1.3 线程常用方法](#13-线程常用方法)
  - [1.4 两类线程](#14-两类线程)
- [2 关键字](#2-关键字)
  - [2.1 synchronized](#21-synchronized)
    - [2.1.1 CAS](#211-cas)
  - [2.2 volatile](#22-volatile)
- [3 锁和AQS](#3-锁和aqs)
  - [3.1 锁分类](#31-锁分类)
  - [3.2 JUC](#32-juc)
  - [3.3 AQS](#33-aqs)
- [4 线程池](#4-线程池)
  - [4.1 Executor框架](#41-executor框架)
  - [4.2 线程池](#42-线程池)
- [5 并发库](#5-并发库)
  - [5.1 并发工具类](#51-并发工具类)
  - [5.2 并发容器](#52-并发容器)
  - [5.3 阻塞队列](#53-阻塞队列)
  - [5.4 原子操作类](#54-原子操作类)
- [6 线程安全](#6-线程安全)
  - [6.1 Java内存模型](#61-java内存模型)
  - [6.2 协程](#62-协程)
- [7 并发编程建议](#7-并发编程建议)



### 1 Java线程

#### 1.1 启动和终止线程
线程的实现方式：线程类的构造方法、静态块是被 new 这个线程类所在的线程所调用的，而 run 方法里面的代码才是被线程自身所调用的。线程中抛出的异常需要在本地进行处理。
1. 继承 `Thread` 类—— Thread 类也实现了 Runable 接口；
2. 实现 `Runnable` 接口，实现 Runnable 接口会创建一个 Thread 对象，并将 Runnable 对象与 Thread 对象关联；
3. 实现 `Callable` 接口；
4. 线程池 —— Executor 框架。

Callable 和 Runnable 的异同

比较|Callable|Runnable
:-:|:-:|:-:
**规定（重写）方法**|call()|run()
**任务返回**|可拿到一个 Future 对象<br/>表示异步计算的结果|不可返回值
**异常**|可抛异常|不可抛异常

实现 Runnable 和 Callable 与 继承 Thread 的异同

比较|实现接口|继承Thread
:-:|:-:|:-:
**开销**|小|大
**编程**|复杂|简单
**使用当前线程**|Thread.currentThread()|this
**面向对象**|还可以继承其他类|不能再继承其他父类
**其它**|多个线程可共享同个 target 对象|

终止线程：每个线程都有自己的中断策略，因此除非你知道中断对该线程的含义，否则就不应该中断这个线程。
1. 自然执行完；
2. `stop()` `resume()` `suspend()` 过时的方法，不建议使用；
3. `interrupt()` 并不是立即停止目标线程正在进行的工作，而是传递了请求中断的消息；
4. 通过 `Future.cancel()` 来实现取消；
5. 通过 `ExecutorService.shutdown()` 来关闭。

#### 1.2 线程状态
- 新建（NEW）
- 可运行（RUNABLE）：正在 Java 虚拟机中运行。在操作系统层面，它可能处于运行状态，也可能等待资源调度。
- 阻塞（BLOCKED）：等待获取 monitor lock。阻塞是被动的
- 无限期等待（WAITING）：等待其它线程显式地唤醒。等待是主动的。进入方法：
  - 没有设置 Timeout 参数的 Object.wait() 方法
  - 没有设置 Timeout 参数的 Thread.join() 方法
- 限期等待（TIMED_WAITING）：一定时间之后会被系统自动唤醒。进入方法：
  - Thread.sleep()
  - 设置了 Timeout 参数的 Object.wait() 方法
  - 设置了 Timeout 参数的 Thread.join() 方法
- 死亡（TERMINATED）

#### 1.3 线程常用方法
- `run()` `start()` 把需要处理的代码放到 run() 方法中，start() 方法启动线程将自动调用 run() 方法。并且 run() 方法必需是 public 访问权限，返回值类型为 void。
- `currentThread()` 当前正在运行的线程
- `interrupted` 中断。中断不会真正中断一个正在运行的线程，而只是发出中断请求。
  - `void interrupt()` 中断操作，设置中断标志位为 true；
  - `boolean interrupted()` 判断当前线程的中断标志位是否为 true，并取消中断标志位；
  - `boolean isInterrupted()` 判断当前线程的中断标志位是否为 true，但不清除中断标志位；
  - 当抛出 InterruptedException 时候，会清除中断标志位。
- `join()` 使调用该方法的线程在此之前执行完毕，即等待该方法的线程执行完毕后再往下继续执行。该方法需要捕捉异常。
  - `isAlive()` 是核心方法，用来判断当前线程是否已经结束。
- `sleep()` 使当前线程（即调用该方法的线程）暂停执行一段时间，让其他线程有机会继续执行，但它并不释放对象锁。即如果有synchronized 同步块，其他线程仍然不能访问共享数据。该方法要捕捉异常。
- `yield()` 使当前线程让出 CPU 占有权，但让出的时间是不可设定的，同时不会释放锁标志。声明了当前线程已经完成了生命周期中最重要的部分，可以切换给相同优先级的其它线程来执行。
- `getStackTrace()` `getAllStackTraces()` 获取堆栈的情况/获取所有栈的情况

sleep() 与 wait() 对比

对比|sleep()|wait()
:-:|:-:|:-:
**提供者**|Thread|Object
**方法属性**|静态方法|实例方法
**使用**|任何地方|同步方法或块
**作用对象**|当前线程|当前对象
**唤醒条件**|超时或调用 interrupt()|notify() 或 notifyAll()
**释放锁资源**|否|是
**对象监视器**|不放弃|放弃


#### 1.4 两类线程
1. **User Thread（用户线程）**：就是我们平时线程池或者创建线程start后的线程，没有设置 setDaemon(true)
2. **Daemon Thread（守护线程）**：为其他线程服务的线程。所有非守护线程都执行完毕后，无论有没有守护线程，虚拟机都会自动退出。
   - `thread.setDaemon(true)` 必须在thread.start() 之前设置，否则会出现 IllegalThreadStateException 异常。不能把正在运行的常规线程设置为守护线程。
   - 在 Daemon 线程中产生的新线程也是 Daemon 的




### 2 关键字

#### 2.1 synchronized
synchronized 是 Java 提供的一种内置锁机制来支持原子性，是可重入的。

作用范围
- 同步一个代码块或方法 —— 作用于同一个对象
- 同步一个类或静态方法 —— 作用于整个类
- 原则：同步的范围越小越好。

##### 2.1.1 CAS
synchronized 优化：**原子操作CAS**

`CAS` 是 `compare and swap` 的缩写，即比较和交换。是 CPU 底层的一种技术，是一条 CPU 的原子指令。一种基于锁的操作，而且是乐观锁。指令级别保证这是一个原子操作.
- CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值（B）
- 如果内存地址里面的值和 A 的值是一样的，那么就将内存里面的值更新成 B。
- CAS是通过无限循环来获取数据的，若果在第一轮循环中，a 线程获取地址里面的值被 b 线程修改了，那么 a 线程需要自旋，到下次循环才有可能机会执行。

CAS 和 synchronized 的主要区别：如果使用 synchronized 关键字，当有多个线程竞争临界区资源的时候，当一个线程获得了监视器锁，其他线程则会进入阻塞状态。而 CAS 操作，当多个线程竞争的时候，当 CAS 操作失败时，并不是立即挂起，而是会进行一些重试操作。

CAS 的问题
- CAS 容易造成 ABA 问题：如果一个变量初次读取的时候是 A 值，它的值被改成了 B，后来又被改回为 A，那 CAS 操作就会误认为它从来没有被改变过。
- 不能保证代码块的原子性，只能保证一个共享变量的原子操作。
- CAS 造成 CPU 利用率增加：CAS 操作长期不成功，CPU 不断的循环。

JVM 对 synchronized 的优化
- **自旋锁**：让一个线程在请求一个共享数据的锁时执行忙循环（自旋）一段时间，如果在这段时间内能获得锁，就可以避免进入阻塞状态。
- **锁消除**：对于被检测出不可能存在竞争的共享数据的锁进行消除。
- **锁粗化**：如果虚拟机探测到由这样的一串零碎的操作都对同一个对象加锁，将会把加锁的范围扩展（粗化）到整个操作序列的外部。
- **轻量级锁**：是相对于传统的重量级锁而言，它使用 CAS 操作来避免重量级锁使用互斥量的开销。
- **偏向锁**：是偏向于让第一个获取锁对象的线程，这个线程在之后获取该锁就不再需要进行同步操作。

#### 2.2 volatile
相关概念
+ 重排序：CPU 避免内存访问延迟最常见的技术是将指令管道化，然后尽量重排这些管道的执行以最大化利用缓存。
+ 内存屏障：多线程环境中使程序结果尽快可见的技术。

volatile 是 JVM 提供的最轻量级的同步机制。被 volatile 修饰的变量能够保证每个线程能够获取该变量的最新值，从而避免数据出现脏读的现象。流程：
1. 被标记为 volatile 的共享变量被一个处理器修改。
2. JVM 会发送 Lock 前缀指令引起该处理器将工作缓存中的数据写回到主内存中。
3. 其他处理器通过嗅探机制会发现自己缓存的数据失效了，于是将该缓存行标记为无效状态。
4. 在其他处理器执行修改操作的时候，会强制重新从主内存中读取数据。

使用 volatile 的场景
- 对变量的写入操作不依赖变量的当前值，或只有单个线程更新变量的值。
- 该变量不会与其它状态变量一起纳入不变性条件中。
- 在访问变量时不需要加锁。




### 3 锁和AQS

#### 3.1 锁分类
- 乐观锁/悲观锁
  - 乐观锁总是认为不存在并发问题，每次去取数据的时候，总认为不会有其他线程对数据进行修改，因此不会上锁。但是在更新时会判断其他线程在这之前有没有对数据进行修改，一般会使用“数据版本机制（版本号、时间戳）”或“CAS 操作”来实现。
  - 悲观锁认为对于同一个数据的并发操作，一定会发生修改的。
- 独享锁/共享锁
  - 独享锁是指该锁一次只能被一个线程所持有。
  - 共享锁是指该锁可被多个线程所持有。
- 互斥锁/读写锁
  - 互斥锁/读写锁是独享锁/共享锁的具体实现
  - ReentrantLock
  - ReadWriteLock - ReentrantReadWriteLock
- 可重入锁：可重入锁又名递归锁，是指在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁。
- 公平锁/非公平锁
  - 公平锁是指多个线程按照申请锁的顺序来获取锁。
  - 非公平锁是指多个线程获取锁的顺序并不是按照申请锁的顺序。
- 分段锁：分段锁是一种锁的设计，目的是细化锁的粒度。
- 偏向锁/轻量级锁/重量级锁：这三种锁是指锁的状态，并且是针对Synchronized。
  - 偏向锁是指一段同步代码一直被一个线程所访问，那么该线程会自动获取锁。降低获取锁的代价。
  - 轻量级锁是指当锁是偏向锁的时候，被另一个线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞，提高性能。
  - 重量级锁是指当锁为轻量级锁的时候，另一个线程虽然是自旋，但自旋不会一直持续下去，当自旋一定次数的时候，还没有获取到锁，就会进入阻塞，该锁膨胀为重量级锁。重量级锁会让他申请的线程进入阻塞，性能降低。
- 自旋锁：自旋锁是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁

#### 3.2 JUC
`J.U.C` 即 `java.util.concurrent` 工具包，这是一个处理线程的工具包，大大提高了并发性能，AQS 是 JUC 的核心。Lock 是 JUC 中锁的接口。

`Lock` 接口比同步方法和同步块提供了更具扩展性的锁操作。
1. 可以使锁更公平
2. 可以使线程在等待锁的时候响应中断
3. 可以让线程尝试获取锁，并在无法获取锁的时候立即返回或者等待一段时间
4. 可以在不同的范围，以不同的顺序获取和释放锁

常用方法
```java
void lock(); // 获取锁
void lockInterruptibly() throws InterruptedException；// 获取锁的过程能够响应中断
boolean tryLock(); // 非阻塞式响应中断能立即返回，获取锁放回 true 反之返回 fasle
boolean tryLock(long time, TimeUnit unit) throws InterruptedException; // 超时获取锁，在超时内或者未中断的情况下能够获取锁
Condition newCondition(); // 获取与 lock 绑定的等待通知组件，当前线程必须获得了锁才能进行等待，进行等待时会先释放锁，当再次获取锁时才能从等待中返回
```

#### 3.3 AQS
`AQS` 是 `AbustactQueuedSynchronizer` 的简称，它是一个 Java 提高的底层同步工具类。AQS 的实现是底层底层维护了一个先进先出(FIFO)的双向队列，这个队列是基于链表实现的。如果线程竞争锁失败，那么就会进入到这个同步队列中进行等待。当获得锁的线程释放锁之后，会从队列中唤醒一个线程。
+ AQS 用一个 int 类型的变量表示同步状态，并提供了一系列的 CAS 操作来管理这个同步状态。
+ AQS 是一个用来构建锁和同步器的框架，使用 AQS 能简单且高效地构造出应用广泛的大量的同步器。
+ AQS 采用模板模式进行设计，AQS 的 protected 方法需要由继承 AQS 的子类进行重写，当调用 AQS 的子类方法时就会调用被重写的方法。
+ AQS 负责同步状态的管理、线程的排队、等待和唤醒这些底层操作；而 Lock 等同步组件主要专注于实现同步语义。

AQS 的实现：
1. 独占锁：每次只能有一个线程持有锁。ReentrantLock 就是通过独占锁实现互斥性的。
   - void acquire(int arg) 独占式的获取同步状态
   - void acquiredInterruptibly(int arg) 在上一方法上增加响应中断
   - void tryAcquireNanos(int arg, long nanos) 在上一方法上增加超时限制
   - boolean release(int arg) 独占式的释放同步状态
2. 共享锁：允许多个线程同时获取锁，并发地访问资源。如 ReentrantReadWriteLock。
   - void acquireShared(int arg) 共享式的获取同步状态
   - void acquiredSharedInterruptibly(int arg) 在上一方法上增加响应中断
   - void tryAcquireSharedNanos(int arg, long nanos) 在上一方法上增加超时限制
   - boolean releaseShared(int arg) 共享式的释放同步状态

`ReentrantLock` 是 java.util.concurrent（J.U.C）包中的锁。ReentrantLock 的重入性 —— 保证线程在获取锁的时候，如果已经获得了锁，可以进行多次加锁，并且重复加锁的线程不必等待，可以直接进行加锁；在释放锁的时候，加锁了多少次，也必须释放多少次。

比较|synchronized|ReentrantLock
:-:|:-:|:-:
**本质区别**|关键字|类
**实现**|JVM|JDK
**可否中断**|不可|可以
**是否公平锁**|非公平|非公平，也可以公平
**是否独占锁**|是|是
**加锁解锁**|自动|手动
**可否重入**|可以|可以

Java 中读写锁的主要实现为 `ReentrantReadWriteLock`，实现自 `ReadWriteLock` 接口。其提供了以下特性：
- 公平锁：支持公平与非公平（默认）的锁获取方式，吞吐量非公平优先于公平。
- 可重入：读线程获取读锁之后可以再次获取读锁，写线程获取写锁之后可以再次获取写锁。
- 可降级：写线程获取写锁之后，其还可以再次获取读锁，然后释放掉写锁，那么此时该线程是读锁状态，也就是降级操作。

如何选择：
- 当读写频率几乎相等，而且不需要特殊需求的时候，优先考虑 `synchronized`
- 当需要定制我们自己的 lock 的时候，或者需要更多的功能（类似定时锁、可中断锁等待)，我们使用 `ReetrantLock`
- 当很少的写操作，更多的读操作，并且读操作是一个相对耗时的操作，那么就是用 `ReetrantReadWriteLock`

比较|Object|Condition
:-:|:-:|:-:
**提供者**|JVM|JDK
**等待通信方法**|wait()<br/>notify()<br/>notifyAll()|await()<br/>signal()<br/>signalAll()
**配合对象**|对象监视器|Lock
**超时设置**|不支持|支持
***不*相应中断**|不支持|支持
**等待队列**|一个|多个

LockSupport 是用来阻塞线程的工具，park()方法用来阻塞线程，unpark()方法用来唤醒线程。



### 4 线程池
线程池优点
1. 减少资源消耗：重用存在的线程，减少对象创建或销毁的开销。
2. 可有效的控制最大并发线程数，提高系统资源的使用率，同时避免过多资源竞争，避免堵塞。
3. 提供定时执行、定期执行、单线程、并发数控制等功能。

#### 4.1 Executor框架
Executor 框架基于生产者-消费者模式。任务是一组逻辑工作单元，而线程则是使任务异步执行的机制。

主要组成
+ 任务：包括被执行的任务需要实现的接口：`Runnable` 接口、`Callable` 接口。
+ 任务的执行：包括任务执行机制的核心接口 `Executor`，以及继承自 `Executor` 的 `ExecutorService` `接口。两个实现了ExecutorService` 接口的关键类：`ThreadPoolExecutor` 和 `ScheduledThreadPoolExecutor`。
  + `Executor` 接口：是 Executor 框架的基础，它将任务的提交与任务的执行分离开来。
  + `ExecutorService` 接口：扩展了 Executor 接口，提供了管理终止的方法，以及可为跟踪一个或多个异步任务执行状况而生成 Future 的方法。
    + `ThreadPoolExecutor` 通常使用工厂类 Executors 来创建，可以创建三种类型：`CachedThreadPool`、`FixedThreadPool` 和 `SingleThreadExecutor`。
    + `ScheduledThreadPoolExecutor` 通常使用工厂类 Executors 来创建，可以创建两种类型：`ScheduledThreadPoolExecutor`、`SingleThreadScheduledExecutor`。
  + `AbstractExecutorService` 类：提供 ExecutorService 执行方法的默认实现。
  + `ScheduledExecutorService` 接口： 一个特殊的 ExecutorService，提供了可安排在给定的延迟后运行或定期执行的命令。
+ 任务的异步计算结果： 包括 Future 接口和实现 `Future` 接口的 `FutureTask` 类、`ForkJoinTask` 类。
  + `Future<V>` **接口**对具体的 Runnable 或者 Callable 任务的执行结果进行取消、查询是否完成、获取结果。Future 表示一个任务的生命周期，get 方法的行为取决于任务的状态。。
  + `FutureTask` **类**实现了 RunnableFuture 接口，该接口继承自 Runnable 和 Future 接口，这使得 FutureTask 既可以当做一个任务执行，也可以有返回值获取执行结果。
  + `ForkJoinPool` 是 Fork/Join 框架里面的管理者。它负责控制整个 Fork/Join 有多少个 ForkJoinWorkerThread，掌控 ForkJoinWorkerThread 的创建和激活。

```java
ExecutorService es = Executors.newSingleThreadExecutor();
MyCallable myCall = new MyCallable();

// Callable + Future 获取执行结果
Future<Integer> future = es.submit(myCall);

// Callable + FutureTask 获取执行结果
FutureTask<Integer> futureTask = new FutureTask<>(myCall);
es.submit(futureTask);

es.shutdown();
```

#### 4.2 线程池
线程池创建：通过调用 Executors 中的静态工厂方法之一来创建
1. `newCachedThreadPool()` 返回 ThreadPoolExecutor 实例
   - 创建一个可缓存线程池
   - 执行很多短期异步任务的程序
   - 使用了 SynchronousQueue（内部只能包含一个元素的队列）
2. `newFixedThreadPool()` 返回 ThreadPoolExecutor 实例
   - 创建一个定长线程池，可控制线程最大并发数
   - 适用于负载较重的服务器
   - 使用了无界队列
3. `newSingleThreadExecutor()`
   - 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务
   - 需要顺序保证执行任务，不会有多个线程活动
   - 使用了无界队列
4. `newScheduledThreadPool()`
   - 创建一个定长线程池，支持定时及周期性任务执行
   - 适合适度控制后台线程数量

常见参数：
- corePoolSize：核心线程数量
- maximumPoolSize：线程最大线程数
- keepAliveTime：线程没有任务时最多保持多久时间终止
- workQueue：阻塞队列，存储等待执行的任务

任务提交
- `void execute(Runnable command)`
- `Future submit(Callable task)`
- 提交任务时，线程池队列已满
  1. 如果使用的是无界队列 LinkedBlockingQueue，也就是无界队列的话，可以继续添加任务到阻塞队列中等待执行，因为 LinkedBlockingQueue 可以近乎认为是一个无穷大的队列，可以无限存放任务。
  2. 如果使用的是有界队列比如 ArrayBlockingQueue，任务首先会被添加到 ArrayBlockingQueue 中，ArrayBlockingQueue 满了，会根据 maximumPoolSize 的值增加线程数量，如果增加了线程数量还是处理不过来，ArrayBlockingQueue 继续满，那么则会使用拒绝策略 RejectedExecutionHandler 处理满了的任务，默认是中止策略。
- 饱和策略
  - 中止策略（默认）
  - 调用者策略

关闭线程池，ExecutorService 接口
- `shutdown()` 平缓的关闭：完成所有已经启动的任务，不再接受任何新任务
- `shutdownNow()` 粗暴的关闭：尝试取消所有已经启动的任务，不再接受任何新任务



### 5 并发库

#### 5.1 并发工具类
- `CountDownLatch` 能够使一个线程在等待另外一些线程完成各自工作之后，再继续执行。可以理解为加强版的 join()。
- `CyclicBarrier` 让一组线程达到某个屏障被阻塞，一直到组内最后一个线程达到屏障时，屏障开放，所有被阻塞的线程会继续运行。
- `Semaphore` 控制同时访问某个特定资源的线程数量，用在流量控制。
- `ThreadLocal` 是维持线程封闭性的一种规范方法，它提供了 get 和 set 等访问接口或方法，这些方法为每个使用该变量的线程都存有一份独立的副本。

#### [5.2 并发容器](https://petepro.github.io/posts/java-container/#线程安全的容器)

#### 5.3 阻塞队列
`BlockingQueue` 阻塞队列：当队列空时，读线程会等待队列变为非空；当队列满时，写线程会等待队列变为可用。适用于生产者-消费者场景。
- `java.util.concurrent.BlockingQueue` 接口的实现
  - `ArrayBlockingQueue`：一个由数组结构组成的有界阻塞队列。
    - 按照先进先出原则，要求设定初始大小。只有一个锁。支持直接插入元素。
  - `LinkedBlockingQueue`：一个由链表结构组成的有界阻塞队列。
    - 按照先进先出原则，可不设定初始大小。用了两个锁。插入元素需要转换。
  - `PriorityBlockingQueue`：一个支持优先级排序的无界阻塞队列。
    - 默认情况下，按照自然顺序，或者实现 compareTo() 方法。
- `SynchronousQueue`：一个不存储元素的阻塞队列。
  - 它维护一组线程，降低了将数据从生产者移动到消费者的延迟。
  - 仅当有足够多的消费者，并且总有一个消费者准备好获取交付的工作时使用。
- `DelayQueue`：一个使用优先级队列实现的无界阻塞队列。
  - 支持延时获取的元素的阻塞队列，元素必须要实现 Delayed 接口。
  - 适用场景：实现自己的缓存系统，订单到期，限时支付等等。
- `LinkedTransferQueue`：一个由链表结构组成的无界阻塞队列。
- `LinkedBlockingDeque`：一个由链表结构组成的双向阻塞队列。
  - 队列的头和尾都可以插入和移除元素，实现工作密取，方法名带 First 对头部操作，带 last 对尾部操作。

#### 5.4 原子操作类
从 Java1.5 开始，jdk 提供了 `java.util.concurrent.atomic` 包，使用 CAS 无锁技术保证变量的操作在多线程环境下正常，比 synchronized 控制的更细，量级更轻。atomic 包里面一共提供了 13 个类，分为 4 种类型。
- 原子更新基本类型：AtomicInteger、AtomicLong、AtomicBoolean
  - int addAndGet(int delta)：以原子的方式将输入的值与实例中的值相加，并把结果返回，可以解决 ABA 问题
- 原子更新数组：AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray
- 原子更新引用：AtomicReference、AtomicReferenceFieldUpdater、AtomicMarkableReference
- 原子更新属性：AtomicIntegerFieldUpdater、AtomicLongFieldUpdater、AtomicStampedReference




### 6 线程安全
**线程安全**：当多个线程访问某个类时，这个类始终都能表现出正确的行为。

线程不安全的原因
1. 主内存和工作内存中的数据不一致
2. 对于程序代码指令重排序导致的

线程安全有以下几种实现方式：
- 不可变（Immutable）的对象一定是线程安全的：final 关键字修饰的基本数据类型、String、枚举类型、Number 部分子类
- 互斥同步 Synchronized 和 ReentrantLock。
- 加锁和 CAS
- 安全发布
  - 发布：使对象能够在当前作用域之外的代码中使用
- 无同步方案
  - 栈封闭：多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于线程私有的。
  - 线程本地存储：可以使用 java.lang.ThreadLocal 类来实现线程本地存储功能。
  - 无状态类：没有任何成员变量的类

安全发布
- 在静态初始化函数中初始化一个对象的引用
- 将对象的引用保存到 volatile 类型的域或者 AtomicReference 对象中
- 将对象的引用保存到某个正确构造对象的 final 类型域中
- 将对象的引用保存到由一个锁保护的域中

单例模式的线程安全性
- 饿汉式单例模式：线程安全。
  - 在声明的时候就 new 这个类的实例
- 懒汉式单例模式：非线程安全。
  - 在单例类的内部由一个私有静态内部类来持有这个单例类的实例。
  - 可以通过加同步机制，或双重校验机制来解决。

#### 6.1 Java内存模型
***Java 内存模型（Java Memory Model，JMM）***试图屏蔽各种硬件和操作系统的内存访问差异，以实现让 Java 程序在各种平台下都能达到一致的内存访问效果。
- 主内存（Main Memory）与工作内存（Local Memory）
  - 所有的变量都存储在主内存中，每个线程还有自己的工作内存，工作内存存储在高速缓存或者寄存器中。
  - 线程只能直接操作工作内存中的变量，不同线程之间的变量值传递需要通过主内存来完成。
- Java 内存模型定义了 8 个原子操作来完成主内存和工作内存的交互操作
  - 作用于主内存的变量：read、write、lock、unlock
  - 作用于线程的工作内存的变量：load、use、assign、store
- 内存模型三大特性
  - **原子性**：指的是一个或者多个操作，要么全部执行并且在执行的过程中不被其他操作打断，要么就全部都不执行。
  - **可见性**：指多个线程操作一个共享变量时，其中一个线程对变量进行修改后，其他线程可以立即看到修改的结果。
  - **有序性**：即程序的执行顺序按照代码的先后顺序来执行。

JMM 原则
- as-if-serial 原则：不管怎么重排序，单线程程序的执行结果不能被改变。编译器、runtime 和处理器都必须遵守 as-if-serial 语义。
- happens-before 原则（先行先发原则）：
  - JMM 可以通过 happens-before 关系向程序员提供跨线程的内存可见性保证；
  - 两个操作之间存在 happens-before 关系，并不意味着 Java 平台的具体实现必须要按照 happens-before 关系指定的顺序来执行。如果重排序之后的执行结果，与 happens-before 关系来执行的结果一致，那么这种重排序也合法。
  - 具体规则：
    1. 程序次序原则：一个线程内保证语义的串行性。
    2. 管程锁定规则：解锁必然发生在随后的加锁前。
    3. volatile 规则：volatile 变量的写先于读发生，这保证了 volatile 变量的可见性。
    4. 线程启动规则：start() 方法先于它的每一个动作。
    5. 线程终止规则：线程的所有操作先于线程的终结（Thread.join()）。
    6. 线程中断规则：线程的中断（interrupt()）先于被中断线程的代码。
    7. 对象终结规则：对象的构造函数的执行、结束先于 finalize() 方法。
    8. 传递性：A 先于 B，B 先于 C，那么 A 必然先于 C。

#### 6.2 协程
协程：最初的用户线程被设计成协同式调度。主要优势是轻量，局限是应用层面要实现的东西特别多。分为有栈协程和无栈协程。





### 7 并发编程建议
- 给线程起个有意义的名字；
- 缩小同步范围，从而减少锁争用；
- 多用同步工具少用 wait() 和 notify()；
- 使用 BlockingQueue 实现生产者消费者问题；
- 多用并发集合少用同步集合；
- 使用本地变量和不可变类来保证线程安全；
- 使用线程池而不是直接创建线程。
