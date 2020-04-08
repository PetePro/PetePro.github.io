---
layout: post
title:  "【Java】Java 总结 03 并发编程"
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

多线程的风险
- 安全性
- 活跃性
- 性能问题

并发编程三要素
- **原子性**：指的是一个或者多个操作，要么全部执行并且在执行的过程中不被其他操作打断，要么就全部都不执行。
- **可见性**：指多个线程操作一个共享变量时，其中一个线程对变量进行修改后，其他线程可以立即看到修改的结果。
- **有序性**：即程序的执行顺序按照代码的先后顺序来执行。



### 1 Java线程

#### 1.1 启动和终止线程
线程类的构造方法、静态块是被 new 这个线程类所在的线程所调用的，而 run 方法里面的代码才是被线程自身所调用的。

*线程调度器*是一个操作系统服务，它负责为 Runnable 状态的线程分配 CPU 时间。一旦我们创建一个线程并启动它，它的执行便依赖于线程调度器的实现。

线程中抛出的异常需要在本地进行处理。

线程的实现方式
1. 继承 `Thread` 类—— Thread 类也实现了 Runable 接口；
2. 实现 `Runnable` 接口——使用 Runnable 实例再创建一个 Thread 实例，然后调用 Thread 实例的 start() 方法来启动线程；
3. 实现 `Callable` 接口；
4. 线程池。

比较|Callable|Runnable
:-:|:-:|:-:
**规定（重写）方法**|call()|run()
**任务返回**|可拿到一个 Future 对象<br/>表示异步计算的结果|不可返回值
**异常**|可抛异常|不可抛异常

*时间分片*是指将可用的 CPU 时间分配给可用的 Runnable 线程的过程。分配 CPU 时间可以基于线程优先级或者线程等待的时间。线程调度并不受到 Java 虚拟机控制，所以由应用程序来控制它是更好的选择（也就是说不要让你的程序依赖于线程的优先级）。

比较|实现接口|继承Thread
:-:|:-:|:-:
**开销**|小|大
**编程**|复杂|简单
**使用当前线程**|Thread.currentThread()|this
**面向对象**|还可以继承其他类|不能再继承其他父类
**其它**|多个线程可共享同个 target 对象|

终止线程
1. 自然执行完
2. 抛出未处理异常 `InterruptedException`
3. `stop()` `resume()` `suspend()` 过时的方法，不建议使用
4. `interrupt() `
   - 如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出 InterruptedException，从而提前结束该线程。
   - java 线程是协作式，而非抢占式。线程是否中断，由其本身决定。
   - 不能中断 I/O 阻塞和 synchronized 锁阻塞。
   - 调用 interrupt() 方法会设置线程的中断标记。因此可以在循环体中使用 interrupted() 方法来判断线程是否处于中断状态，从而提前结束线程。

#### 1.2 线程常用方法
- `sleep()` 使当前线程（即调用该方法的线程）暂停执行一段时间，让其他线程有机会继续执行，但它并不释放对象锁。即如果有synchronized 同步块，其他线程仍然不能访问共享数据。该方法要捕捉异常。
- `join()` 使调用该方法的线程在此之前执行完毕，即等待该方法的线程执行完毕后再往下继续执行。该方法需要捕捉异常。
- `yield()` 使当前线程让出 CPU 占有权，但让出的时间是不可设定的，同时不会释放锁标志。声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。
- `run()` `start()` 把需要处理的代码放到 run() 方法中，start() 方法启动线程将自动调用 run() 方法。并且 run() 方法必需是 public 访问权限，返回值类型为 void。
- `sleep(long millis)` 使当前线程进入停滞状态，在指定的时间内肯定不会被执行，不会释放锁标志。
- `getStackTrace()` `getAllStackTraces()` 获取堆栈的情况/获取所有栈的情况
- `isAlive()` 线程处于“新建”状态时，该方法返回false。在线程的 run() 方法结束之前，即没有进入死亡状态之前，该方法返回true。

#### 1.3 线程状态
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

#### 1.4 两类线程
1. **User Thread（用户线程）**：就是我们平时线程池或者创建线程start后的线程，没有设置 setDaemon(true)
2. **Daemon Thread（守护线程）**：为其他线程服务的线程。所有非守护线程都执行完毕后，无论有没有守护线程，虚拟机都会自动退出。
   - `thread.setDaemon(true)` 必须在thread.start() 之前设置，否则会出现 IllegalThreadStateException 异常。不能把正在运行的常规线程设置为守护线程。
   - 在 Daemon 线程中产生的新线程也是 Daemon 的

#### 1.5 线程共享
- `synchronized` 控制线程同步的，在多线程的环境下，控制 synchronized 代码段不被多个线程同时执行。属于悲观锁。
  - 同步一个代码块或方法——作用于同一个对象
  - 同步一个类或静态方法——作用于整个类
  - 原则：同步的范围越小越好。
- `Semaphore` 是一个信号量，它的作用是限制某段代码块的并发数。如果 Semaphore 构造函数中传入的 int 型整数 n = 1，相当于变成了一个 synchronized。
- `volatile` 保证了可见性。当一个共享变量被 volatile 修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值。volatile 的一个重要作用就是和 CAS 结合，保证了原子性。
- `ThreadLocal` 是一个本地线程副本变量工具类。主要用于将私有线程和该线程存放的副本对象做一个映射，各个线程之间的变量互不干扰，在高并发场景下，可以实现无状态的调用，特别适用于各个线程依赖不通的变量值完成操作的场景。

#### 1.6 线程协作
- `join()` 在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。它可以使得线程之间的并行执行变为串行执行。
- `wait()` `notify()` `notifyAll()` 等待与通知
  - 调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。
  - JDK 强制要在同步块中被调用，在调用前都必须先获得对象的锁。
  - wait() 与 sleep()的区别
    - wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
    - wait() 会释放锁，sleep() 不会。
    - wait() 方法会放弃这个对象的监视器，sleep() 方法不会放弃这个对象的监视器
- 可以在 `Condition` 类上调用 `await()` 方法使线程等待，其它线程调用 `signal()` 或 `signalAll()` 方法唤醒等待的线程。await() 可以指定等待的条件，因此更加灵活。



### 2 原子操作CAS
`CAS` 是 `compare and swap` 的缩写，即比较和交换。是 CPU 底层的一种技术，是一条 CPU 的原子指令。一种基于锁的操作，而且是乐观锁。指令级别保证这是一个原子操作.
- CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值（B）
- 如果内存地址里面的值和 A 的值是一样的，那么就将内存里面的值更新成 B。
- CAS是通过无限循环来获取数据的，若果在第一轮循环中，a 线程获取地址里面的值被 b 线程修改了，那么 a 线程需要自旋，到下次循环才有可能机会执行。

CAS 的问题
- CAS 容易造成 ABA 问题：如果一个变量初次读取的时候是 A 值，它的值被改成了 B，后来又被改回为 A，那 CAS 操作就会误认为它从来没有被改变过。
- 不能保证代码块的原子性，只能保证一个共享变量的原子操作。
- CAS 造成 CPU 利用率增加：CAS 操作长期不成功，CPU 不断的循环。

#### 2.1 原子操作类
从 Java1.5 开始，jdk 提供了 `java.util.concurrent.atomic` 包，这个包中的原子操作类，提供了用法简单、性能高效、线程安全的更新变量的方式。atomic 包里面一共提供了13个类，分为4种类型。
- 原子更新基本类型：AtomicInteger、AtomicLong、AtomicBoolean
  - int addAndGet(int delta)：以原子的方式将输入的值与实例中的值相加，并把结果返回，可以解决 ABA 问题
- 原子更新数组：AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray
- 原子更新引用：AtomicReference、AtomicReferenceFieldUpdater、AtomicMarkableReference
- 原子更新属性：AtomicIntegerFieldUpdater、AtomicLongFieldUpdater、AtomicStampedReference



### 3 锁和AQS

#### 3.1 锁
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

#### 3.2 显式锁
`Lock` 接口比同步方法和同步块提供了更具扩展性的锁操作。
1. 可以使锁更公平
2. 可以使线程在等待锁的时候响应中断
3. 可以让线程尝试获取锁，并在无法获取锁的时候立即返回或者等待一段时间
4. 可以在不同的范围，以不同的顺序获取和释放锁

`ReentrantLock` 是 java.util.concurrent（J.U.C）包中的锁。ReentrantLock 通过 Condtion 等待/唤醒，相比 wait() 和 notify() / notifAll() 机制，具有更高的灵活性和可控性。

比较|synchronized|ReentrantLock
:-:|:-:|:-:
本质区别|关键字|类
实现|JVM|JDK
可否中断|不可|可以
是否公平锁|非公平|非公平，也可以公平
是否独占锁|是|是
加锁解锁|自动|手动
可否重入|可以|可以

Java 中读写锁的主要实现为 `ReentrantReadWriteLock`，实现自 `ReadWriteLock` 接口。其提供了以下特性：
- 公平锁：支持公平与非公平（默认）的锁获取方式，吞吐量非公平优先于公平。
- 可重入：读线程获取读锁之后可以再次获取读锁，写线程获取写锁之后可以再次获取写锁。
- 可降级：写线程获取写锁之后，其还可以再次获取读锁，然后释放掉写锁，那么此时该线程是读锁状态，也就是降级操作。

如何选择：
- 当读写频率几乎相等，而且不需要特殊需求的时候，优先考虑 `synchronized`
- 当需要定制我们自己的 lock 的时候，或者需要更多的功能（类似定时锁、可中断锁等待)，我们使用 `ReetrantLock`
- 当很少的写操作，更多的读操作，并且读操作是一个相对耗时的操作，那么就是用 `ReetrantReadWriteLock`

#### 3.3 JUC与AQS
`J.U.C` 即 `java.util.concurrent` 工具包，这是一个处理线程的工具包，大大提高了并发性能，AQS 是 J.U.C 的核心。

`AQS` 是 `AbustactQueuedSynchronizer` 的简称，它是一个 Java 提高的底层同步工具类，用一个 int 类型的变量表示同步状态，并提供了一系列的 CAS 操作来管理这个同步状态。AQS 使用的设计模式为模板模式。AQS 是一个用来构建锁和同步器的框架，使用 AQS 能简单且高效地构造出应用广泛的大量的同步器。AQS 同步方式：
1. 独占式 ReentrantLock
2. 共享式 Semaphore、CountDownLatch
3. 组合式 ReentrantReadWriteLock



### 4 线程池
线程池优点
1. 重用存在的线程，减少对象创建销毁的开销。
2. 可有效的控制最大并发线程数，提高系统资源的使用率，同时避免过多资源竞争，避免堵塞。
3. 提供定时执行、定期执行、单线程、并发数控制等功能。

#### 4.1 Executor框架
`Executor` 框架包括：线程池，Executor，Executors，ExecutorService，CompletionService，Future，Callable 等。

`Executor` 接口：Java 线程池的超级接口，其定义了一个接收 Runnable 对象的方法 executor()，无返回结果。

`ExecutorService` 接口：ExecutorService 接口继承了 Executor 接口，是 Executor 的子接口。submit() 方法可以接受 Runnable 和 Callable 接口的对象。其提供了生命周期管理的方法，返回 Future 对象，以及可跟踪一个或多个异步任务执行状况返回 Future 的方法。
- `Future<V>` 对具体的 Runnable 或者 Callable 任务的执行结果进行取消、查询是否完成、获取结果。
- `FutureTask` 实现了 RunnableFuture 接口，该接口继承自 Runnable 和 Future 接口，这使得 FutureTask 既可以当做一个任务执行，也可以有返回值获取执行结果。

`Executors` 类：java.util.concurrent 包下的一个类，提供了若干个静态方法，用于生成不同类型的线程池。

#### 4.2 线程池
线程池创建，java.util.concurrent.Executor 接口，由 Executors 类创建。
1. `newCachedThreadPool` 
   - 创建一个可缓存线程池
   - 执行很多短期异步任务的程序
   - 使用了 SynchronousQueue（内部只能包含一个元素的队列）
2. `newFixedThreadPool`
   - 创建一个定长线程池，可控制线程最大并发数
   - 适用于负载较重的服务器
   - 使用了无界队列
3. `newSingleThreadExecutor`
   - 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务
   - 需要顺序保证执行任务，不会有多个线程活动
   - 使用了无界队列
4. `newScheduledThreadPool`
   - 创建一个定长线程池，支持定时及周期性任务执行
   - 适合适度控制后台线程数量

任务提交
- void execute(Runnable command)
- Future submit(Callable task)
- 提交任务时，线程池队列已满
  1. 如果使用的是无界队列 LinkedBlockingQueue，也就是无界队列的话，可以继续添加任务到阻塞队列中等待执行，因为 LinkedBlockingQueue 可以近乎认为是一个无穷大的队列，可以无限存放任务
  2. 如果使用的是有界队列比如 ArrayBlockingQueue，任务首先会被添加到 ArrayBlockingQueue 中，ArrayBlockingQueue 满了，会根据 maximumPoolSize 的值增加线程数量，如果增加了线程数量还是处理不过来，ArrayBlockingQueue 继续满，那么则会使用拒绝策略 RejectedExecutionHandler 处理满了的任务，默认是 AbortPolicy

关闭线程池，ExecutorService 接口
- `shutdown()` 只会中断所有没有执行任务的线程
- `shutdownNow()` 还会尝试停止正在运行或者暂停任务的线程



### 5 并发库

#### 5.1 并发工具类
`Fork/Join` 框架是Java 7 提供的一个用于并行执行任务的框架，是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架。
- `ForkJoinPool` 是 Fork/Join 框架里面的管理者，最原始的任务都要交给它才能处理。它负责控制整个 Fork/Join 有多少个 WorkerThread，WorkerThread的创建和激活都是由它来掌控。
- `ForkJoinWorkerThread` Fork/Join 里面真正干活的“工人”，本质是一个线程。
- `ForkJoinPool.WorkQueue` 双端队列，负责存储接收的任务。
- `ForkJoinTask` Fork/Join 里面的任务类型。

其它工具类
- `CountDownLatch` 能够使一个线程在等待另外一些线程完成各自工作之后，再继续执行。可以理解为加强版的 join()。
- `CyclicBarrier` 让一组线程达到某个屏障被阻塞，一直到组内最后一个线程达到屏障时，屏障开放，所有被阻塞的线程会继续运行。
- `Semaphore` 控制同时访问某个特定资源的线程数量，用在流量控制。
- `Exchange` 两个线程间的数据交换，优先到达 Exchange 点的线程阻塞等待另一个线程执行到达这个点。都到达以后交换数据继续执行。

#### 5.2 并发容器
- `ConcurrentHashMap`
  - 链地址法解决冲突
  - 由 **Segment 数组结构**（继承自可重入锁 ReentrantLock，扮演锁的角色）和 **HashEntry 数组结构**（存储键值对数据）组成，每个 HashEntry 是一个链表结构的元素。
  - JDK1.8 已经抛弃了 Segment 分段锁机制，利用 CAS + Synchronized 来保证并发更新的安全。数据结构采用：数组 + 链表 + 红黑树。
    - 什么时候链表转红黑树？当 key 值相等的元素形成的链表中元素个数超过8的时候。
- `ConcurrentSkipListMap`：TreeMap 和 TreeSet 的并发版本
- `ConcurrentLinkedQueue`：无界非阻塞队列，底层是个链表，遵循先进先出原则。

阻塞队列
- `java.util.concurrent.BlockingQueue` 接口的实现
  - `ArrayBlockingQueue`：一个由数组结构组成的有界阻塞队列。
    - 按照先进先出原则，要求设定初始大小。只有一个锁。支持直接插入元素。
  - `LinkedBlockingQueue`：一个由链表结构组成的有界阻塞队列。
    - 按照先进先出原则，可不设定初始大小。用了两个锁。插入元素需要转换。
  - `PriorityBlockingQueue`：一个支持优先级排序的无界阻塞队列。
    - 默认情况下，按照自然顺序，或者实现 compareTo() 方法。
- `DelayQueue`：一个使用优先级队列实现的无界阻塞队列。
  - 支持延时获取的元素的阻塞队列，元素必须要实现 Delayed 接口。
  - 适用场景：实现自己的缓存系统，订单到期，限时支付等等。
- `SynchronousQueue`：一个不存储元素的阻塞队列。
  - 每一个 put 操作都要等待一个 take 操作
- `LinkedTransferQueue`：一个由链表结构组成的无界阻塞队列。
- `LinkedBlockingDeque`：一个由链表结构组成的双向阻塞队列。
  - 队列的头和尾都可以插入和移除元素，实现工作密取，方法名带 First 对头部操作，带 last 对尾部操作。



### 6 线程安全
**线程安全**：当多个线程访问某个类时，这个类始终都能表现出正确的行为。

线程安全有以下几种实现方式：
- 不可变（Immutable）的对象一定是线程安全的：final 关键字修饰的基本数据类型、String、枚举类型、Number 部分子类
- 互斥同步 Synchronized 和 ReentrantLock。
- 加锁和 CAS
- 安全发布
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
- 双检锁单例模式：线程安全



### 7 Java内存模型
Java 内存模型（Java Memory Model，JMM）试图屏蔽各种硬件和操作系统的内存访问差异，以实现让 Java 程序在各种平台下都能达到一致的内存访问效果。
- 主内存（Main Memory）与工作内存（Local Memory）
  - 所有的变量都存储在主内存中，每个线程还有自己的工作内存，工作内存存储在高速缓存或者寄存器中。
  - 线程只能直接操作工作内存中的变量，不同线程之间的变量值传递需要通过主内存来完成。
- Java 内存模型定义了 8 个操作来完成主内存和工作内存的交互操作
  - 作用于主内存的变量：read、write、lock、unlock
  - 作用于线程的工作内存的变量：load、use、assign、store
- 关键字 volatile 是 Java 虚拟机中提供的最轻量级的同步机制
  - volatile 类型的变量保证对所有线程的可见性。
  - volatile 类型的变量禁止指令重排序优化。
- 内存模型三大特性：原子性、可见性、有序性
- JVM 规定了先行发生原则，让一个操作无需控制就能先于另一个操作完成。

#### 7.1 JVM对synchronized的优化
- 自旋锁：让一个线程在请求一个共享数据的锁时执行忙循环（自旋）一段时间，如果在这段时间内能获得锁，就可以避免进入阻塞状态。
- 锁消除：对于被检测出不可能存在竞争的共享数据的锁进行消除。
- 锁粗化：如果虚拟机探测到由这样的一串零碎的操作都对同一个对象加锁，将会把加锁的范围扩展（粗化）到整个操作序列的外部。
- 轻量级锁：是相对于传统的重量级锁而言，它使用 CAS 操作来避免重量级锁使用互斥量的开销。
- 偏向锁：是偏向于让第一个获取锁对象的线程，这个线程在之后获取该锁就不再需要进行同步操作。

#### 7.2 建议
- 给线程起个有意义的名字；
- 缩小同步范围，从而减少锁争用；
- 多用同步工具少用 wait() 和 notify()；
- 使用 BlockingQueue 实现生产者消费者问题；
- 多用并发集合少用同步集合；
- 使用本地变量和不可变类来保证线程安全；
- 使用线程池而不是直接创建线程。
