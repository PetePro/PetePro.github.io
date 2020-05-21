---
layout: post
title:  "【Java】Java 05 Java并发编程之生产者消费者"
crawlertitle: "生产者消费者"
summary: "Producer and Consumer"
date:   2020-03-15 10:00:00 +0800
categories: posts
tags: 'CS'
author: xusc
bg: "CS.jpg"
---

生产者消费者问题以及 Java 下的实现。

生产者生成一定量的数据放到缓冲区中，然后重复此过程；与此同时，消费者也在缓冲区消耗这些数据。生产者和消费者之间必须保持同步，要保证生产者不会在缓冲区满时放入数据，消费者也不会在缓冲区空时消耗数据。核心：保证资源在任意时刻至多被一个线程访问。

解决方法
- [1 synchronized](#1-synchronized)
- [2 JUC.Lock](#2-juclock)
- [3 BlockingQueue](#3-blockingqueue)
- [4 Semaphore](#4-semaphore)

### 1 synchronized
```java
public class PCSyn {
	private final int MAX_SIZE = 10;                      // 仓库容量
	private LinkedList<Object> list = new LinkedList<>(); // 仓库存储的载体

	public void produce() {
		synchronized (list) {
			while (list.size() + 1 > MAX_SIZE) {
				System.out.println("【生产者" + Thread.currentThread().getName() + "】仓库已满");
				try {
					list.wait();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			list.add(new Object());
			System.out.println("【生产者" + Thread.currentThread().getName() + "】生产一个产品，现库存" + list.size());
			list.notifyAll();
		}
	}

	public void consume() {
		synchronized (list) {
			while (list.size() == 0) {
				System.out.println("【消费者" + Thread.currentThread().getName() + "】仓库为空");
				try {
					list.wait();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			list.remove();
			System.out.println("【消费者" + Thread.currentThread().getName() + "】消费一个产品，现库存" + list.size());
			list.notifyAll();
		}
	}

	class Consumer implements Runnable {
		@Override
		public void run() {
			while (true) {
				try {
					Thread.sleep(3000);
					consume();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}

	class Producer implements Runnable {
		@Override
		public void run() {
			while (true) {
				try {
					Thread.sleep(1000);
					produce();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}

	public static void main(String[] args) {
		PCSyn pc = new PCSyn();
		Thread p = new Thread(pc.new Producer(), "p");
		Thread c = new Thread(pc.new Consumer(), "c");
		p.start();
		c.start();
	}
}
```

### 2 JUC.Lock
用 ReentrantLock 和 Condition 可以实现等待/通知模型，具有更大的灵活性。通过在 Lock 对象上调用 newCondition() 方法，将条件变量和一个锁对象进行绑定，进而控制并发程序访问竞争资源的安全。

```java
public class PCLock {
	private final int MAX_SIZE = 10;                            // 仓库最大存储量
	private LinkedList<Object> list = new LinkedList<Object>(); // 仓库存储的载体
	private final Lock lock = new ReentrantLock();              // 锁
	private final Condition full = lock.newCondition();         // 仓库满的条件变量
	private final Condition empty = lock.newCondition();        // 仓库空的条件变量

	public void produce() {
		lock.lock();
		while (list.size() + 1 > MAX_SIZE) {
			System.out.println("【生产者" + Thread.currentThread().getName() + "】仓库已满");
			try {
				full.await();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		list.add(new Object());
		System.out.println("【生产者" + Thread.currentThread().getName() + "】生产一个产品，现库存" + list.size());
		empty.signalAll();
		lock.unlock();
	}

	public void consume() {
		lock.lock();
		while (list.size() == 0) {
			System.out.println("【消费者" + Thread.currentThread().getName() + "】仓库为空");
			try {
				empty.await();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		list.remove();
		System.out.println("【消费者" + Thread.currentThread().getName() + "】消费一个产品，现库存" + list.size());
		full.signalAll();
		lock.unlock();
	}

	class Producer implements Runnable {
		@Override
		public void run() {
			while (true) {
				try {
					Thread.sleep(1000);
					produce();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}

	class Consumer implements Runnable {
		@Override
		public void run() {
			while (true) {
				try {
					Thread.sleep(3000);
					consume();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}

	public static void main(String[] args) {
		PCLock pc = new PCLock();
		Thread p = new Thread(pc.new Producer(), "p");
		Thread c = new Thread(pc.new Consumer(), "c");
		p.start();
		c.start();
	}
}
```

### 3 BlockingQueue
BlockingQueue 是一个已经在内部实现了同步的队列，实现方式采用的是 await() / signal() 方法。它可以在生成对象时指定容量大小，用于阻塞操作的是 put() 和 take() 方法。

可能会出现 put() 或 take() 和输出不匹配的情况，是由于它们之间没有同步造成的。

```java
public class PCBQ {
	private LinkedBlockingQueue<Object> list = new LinkedBlockingQueue<>(10);// 仓库存储的载体

	public void produce() {
		try {
			list.put(new Object());
			System.out.println("【生产者" + Thread.currentThread().getName() + "】生产一个产品，现库存" + list.size());
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}

	public void consume() {
		try {
			list.take();
			System.out.println("【消费者" + Thread.currentThread().getName() + "】消费了一个产品，现库存" + list.size());
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}

	class Producer implements Runnable {
		@Override
		public void run() {
			while (true) {
				try {
					Thread.sleep(1000);
					produce();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}

	class Consumer implements Runnable {
		@Override
		public void run() {
			while (true) {
				try {
					Thread.sleep(3000);
					consume();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}

	public static void main(String[] args) {
		PCBQ pc = new PCBQ();
		Thread p = new Thread(pc.new Producer());
		Thread c = new Thread(pc.new Consumer());
		p.start();
		c.start();
	}
}
```

### 4 Semaphore
Semaphore 是一种基于计数的信号量。它设定一个阈值，多个线程竞争获取许可信号，做完自己的申请后归还，超过阈值后，线程申请许可信号将会被阻塞。

创建计数为 1 的 Semaphore，将其作为一种类似互斥锁的机制，这也叫二元信号量，表示两种互斥状态。

```java
public class PCSema {
	private LinkedList<Object> list = new LinkedList<Object>(); // 仓库存储的载体
	final Semaphore notFull = new Semaphore(10);                // 仓库的最大容量
	final Semaphore notEmpty = new Semaphore(0);                // 将线程挂起，等待其他来触发
	final Semaphore mutex = new Semaphore(1);                   // 互斥锁

	public void produce() {
		try {
			notFull.acquire();
			mutex.acquire();
			list.add(new Object());
			System.out.println("【生产者" + Thread.currentThread().getName() + "】生产一个产品，现库存" + list.size());
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			mutex.release();
			notEmpty.release();
		}
	}

	public void consume() {
		try {
			notEmpty.acquire();
			mutex.acquire();
			list.remove();
			System.out.println("【消费者" + Thread.currentThread().getName() + "】消费一个产品，现库存" + list.size());
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			mutex.release();
			notFull.release();
		}
	}

	class Producer implements Runnable {
		@Override
		public void run() {
			while (true) {
				try {
					Thread.sleep(1000);
					produce();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}

	class Consumer implements Runnable {
		@Override
		public void run() {
			while (true) {
				try {
					Thread.sleep(3000);
					consume();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}

	public static void main(String[] args) {
		PCSema pc = new PCSema();
		Thread p = new Thread(pc.new Producer(), "p");
		Thread c = new Thread(pc.new Consumer(), "c");
		p.start();
		c.start();
	}
}
```

改编自 [CSDN](https://blog.csdn.net/ldx19980108/article/details/81707751)