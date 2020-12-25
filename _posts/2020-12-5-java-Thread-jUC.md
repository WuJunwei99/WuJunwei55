---
layout: post
title: "JUC并发编程"
date: 2020-12-05 15:27:54 +0800
categories: notes
tags: java
img: https://s1.ax1x.com/2020/07/12/U1oQN4.png
---
回顾线程进程；CopyOnWrite；常用的辅助类；读写锁；塞队列线程池(重点)；四大函数式接口（必需掌握）；Stream流式计算；ForkJoin；异步回调；深入理解CAS；原子引用；各种锁的理解


## 回顾线程进程

### 什么是JUC

在 Java 5.0 提供了 java.util.concurrent(简称JUC)包,在此包中增加了在并发编程中很常用的工具类,用于定义类似于线程的自定义子系统,包括线程池,异步 IO 和轻量级任务框架;还提供了设计用于多线程上下文中的 Collection 实现等;

![](https://s3.ax1x.com/2020/12/25/rfSvPH.png)

### 线程有几个状态

	public enum State { 
	
	// 新生 NEW,
	// 运行 RUNNABLE, 
	// 阻塞 BLOCKED, 
	// 等待，死死地等 WAITING, 
	// 超时等待 TIMED_WAITING, 
	// 终止 TERMINATED; 
	
	}

### wait/sleep 区别

* 来自不同的类
	* wait => Object
	* sleep => Thread
* 关于锁的释放
	* wait 会释放锁，sleep 睡觉了，抱着锁睡觉，不会释放！
* 使用的范围是不同的
	* wait必须在同步代码块中使用
	* sleep没有限制


### Lock锁（重点）

![](https://s3.ax1x.com/2020/12/24/r2WQzt.png)

公平锁：十分公平：可以先来后到

非公平锁：十分不公平：可以插队 （默认）

其中tryAcquire是公平锁和非公平锁实现的区别，下面的两种类型的锁的tryAcquire的实现，从中我们可以看出在公平锁中，每一次的tryAcquire都会检查CLH队列中是否仍有前驱的元素，如果仍然有那么继续等待，通过这种方式来保证先来先服务的原则；而非公平锁，首先是检查并设置锁的状态，这种方式会出现即使队列中有等待的线程，但是新的线程仍然会与排队线程中的对头线程竞争（但是排队的线程是先来先服务的），所以新的线程可能会抢占已经在排队的线程的锁，这样就无法保证先来先服务，但是已经等待的线程们是仍然保证先来先服务的，


### Synchronized 和 Lock 区别

1. Synchronized 内置的Java关键字，Lock 是一个Java类 
2. Synchronized 无法判断获取锁的状态，Lock 可以判断是否获取到了锁
3. Synchronized 会自动释放锁，lock 必须要手动释放锁！如果不释放锁，死锁
4. Synchronized 线程 1（获得锁，阻塞）、线程2（等待，傻傻的等）；Lock锁就不一定会等待下去；
5. Synchronized 可重入锁，不可以中断的，非公平；Lock ，可重入锁，可以 判断锁，非公平（可以自己设置）；
6. Synchronized 适合锁少量的代码同步问题，Lock 适合锁大量的同步代码！


### 生产者和消费者问题

面试的：单例模式、排序算法、生产者和消费者、死锁



	/**
	 * 线程之间的通信问题：生产者和消费者问题！  等待唤醒，通知唤醒
	 * 线程交替执行  A   B 操作同一个变量   num = 0
	 * A num+1
	 * B num-1
	 */
	public class A {
	    public static void main(String[] args) {
	        Data data = new Data();
	
	        new Thread(()->{
	            for (int i = 0; i < 10; i++) {
	                try {
	                    data.increment();
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                }
	            }
	        },"A").start();
	
	        new Thread(()->{
	            for (int i = 0; i < 10; i++) {
	                try {
	                    data.decrement();
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                }
	            }
	        },"B").start();
	
	        new Thread(()->{
	            for (int i = 0; i < 10; i++) {
	                try {
	                    data.increment();
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                }
	            }
	        },"C").start();
	
	
	        new Thread(()->{
	            for (int i = 0; i < 10; i++) {
	                try {
	                    data.decrement();
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                }
	            }
	        },"D").start();
	    }
	}
	
	// 判断等待，业务，通知
	class Data{ // 数字 资源类
	
	    private int number = 0;
	
	    //+1
	    public synchronized void increment() throws InterruptedException {
	        while (number!=0){  //0
	            // 等待
	            this.wait();
	        }
	        number++;
	        System.out.println(Thread.currentThread().getName()+"=>"+number);
	        // 通知其他线程，我+1完毕了
	        this.notifyAll();
	    }
	
	    //-1
	    public synchronized void decrement() throws InterruptedException {
	        while (number==0){ // 1
	            // 等待
	            this.wait();
	        }
	        number--;
	        System.out.println(Thread.currentThread().getName()+"=>"+number);
	        // 通知其他线程，我-1完毕了
	        this.notifyAll();
	    }
	
	}

![](https://s3.ax1x.com/2020/12/24/r2WKJA.png)

if 改为 while 判断

#### JUC版的生产者和消费者问题

![](https://s3.ax1x.com/2020/12/24/r2Wuid.png)

	// 判断等待，业务，通知
	class Data2{ // 数字 资源类
	
	    private int number = 0;
	
	    Lock lock = new ReentrantLock();
	    Condition condition = lock.newCondition();
	
	    //condition.await(); // 等待
	    //condition.signalAll(); // 唤醒全部
	    //+1
	    public void increment() throws InterruptedException {
	        lock.lock();
	        try {
	            // 业务代码
	            while (number!=0){  //0
	                // 等待
	                condition.await();
	            }
	            number++;
	            System.out.println(Thread.currentThread().getName()+"=>"+number);
	            // 通知其他线程，我+1完毕了
	            condition.signalAll();
	        } catch (Exception e) {
	            e.printStackTrace();
	        } finally {
	            lock.unlock();
	        }
	
	    }

Condition 精准的通知和唤醒线程

### 8锁现象

	/**
	 * 8锁，就是关于锁的8个问题
	 * 1、标准情况下，两个线程先打印 发短信还是 打电话？ 1/发短信  2/打电话	//发短信
	 * 1、sendSms延迟4秒，两个线程先打印 发短信还是 打电话？ 1/发短信  2/打电话	//发短信
	 */
	public class Test1 {
	    public static void main(String[] args) {
	        Phone phone = new Phone();
	
	        //锁的存在
	        new Thread(()->{
	            phone.sendSms();
	        },"A").start();
	
	        // 捕获
	        try {
	            TimeUnit.SECONDS.sleep(1);
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        }
	
	        new Thread(()->{
	            phone.call();
	        },"B").start();
	    }
	}
	
	class Phone{
	
	    // synchronized 锁的对象是方法的调用者！、
	    // 两个方法用的是同一个锁，谁先拿到谁执行！
	    public synchronized void sendSms(){
	        try {
	            TimeUnit.SECONDS.sleep(4);
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        }
	        System.out.println("发短信");
	    }
	
	    public synchronized void call(){
	        System.out.println("打电话");
	    }
	
	}

***

	/**
	 * 3、 增加了一个普通方法后！先执行发短信还是Hello？ 普通方法
	 * 4、 两个对象，两个同步方法， 发短信还是 打电话？ // 打电话
	 */
	public class Test2  {
	    public static void main(String[] args) {
	        // 两个对象，两个调用者，两把锁！
	        Phone2 phone1 = new Phone2();
	        Phone2 phone2 = new Phone2();
	
	        //锁的存在
	        new Thread(()->{
	            phone1.sendSms();
	        },"A").start();
	
	        // 捕获
	        try {
	            TimeUnit.SECONDS.sleep(1);
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        }
	
	        new Thread(()->{
	            phone2.call();
	        },"B").start();
	    }
	}
	
	class Phone2{
	
	    // synchronized 锁的对象是方法的调用者！
	    public synchronized void sendSms(){
	        try {
	            TimeUnit.SECONDS.sleep(4);
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        }
	        System.out.println("发短信");
	    }
	
	    public synchronized void call(){
	        System.out.println("打电话");
	    }
	
	    // 这里没有锁！不是同步方法，不受锁的影响
	    public void hello(){
	        System.out.println("hello");
	    }
	
	}

***

	/**
	 * 5、增加两个静态的同步方法，只有一个对象，先打印 发短信？打电话？发短信
	 * 6、两个对象！增加两个静态的同步方法， 先打印 发短信？打电话？发短信
	 */
	public class Test3  {
	    public static void main(String[] args) {
	        // 两个对象的Class类模板只有一个，static，锁的是Class
	        Phone3 phone1 = new Phone3();
	        Phone3 phone2 = new Phone3();
	
	        //锁的存在
	        new Thread(()->{
	            phone1.sendSms();
	        },"A").start();
	
	        // 捕获
	        try {
	            TimeUnit.SECONDS.sleep(1);
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        }
	
	        new Thread(()->{
	            phone2.call();
	        },"B").start();
	    }
	}
	
	// Phone3唯一的一个 Class 对象
	class Phone3{
	
	    // synchronized 锁的对象是方法的调用者！
	    // static 静态方法
	    // 类一加载就有了！锁的是Class
	    public static synchronized void sendSms(){
	        try {
	            TimeUnit.SECONDS.sleep(4);
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        }
	        System.out.println("发短信");
	    }
	
	    public static synchronized void call(){
	        System.out.println("打电话");
	    }
	
	
	}


***

	/**
	 * 1、1个静态的同步方法，1个普通的同步方法 ，一个对象，先打印 发短信？打电话？打电话
	 * 2、1个静态的同步方法，1个普通的同步方法 ，两个对象，先打印 发短信？打电话？打电话
	 */
	public class Test4  {
	    public static void main(String[] args) {
	        // 两个对象的Class类模板只有一个，static，锁的是Class
	        Phone4 phone1 = new Phone4();
	        Phone4 phone2 = new Phone4();
	        //锁的存在
	        new Thread(()->{
	            phone1.sendSms();
	        },"A").start();
	
	        // 捕获
	        try {
	            TimeUnit.SECONDS.sleep(1);
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        }
	
	        new Thread(()->{
	            phone2.call();
	        },"B").start();
	    }
	}
	
	// Phone3唯一的一个 Class 对象
	class Phone4{
	
	    // 静态的同步方法 锁的是 Class 类模板
	    public static synchronized void sendSms(){
	        try {
	            TimeUnit.SECONDS.sleep(4);
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        }
	        System.out.println("发短信");
	    }
	
	    // 普通的同步方法  锁的调用者
	    public synchronized void call(){
	        System.out.println("打电话");
	    }
	
	}

## CopyOnWrite

#### 集合类不安全


	// java.util.ConcurrentModificationException 并发修改异常！
	public class ListTest {
	    public static void main(String[] args) {
	        // 并发下 ArrayList 不安全的吗，Synchronized；
	        /**
	         * 解决方案；
	         * 1、List<String> list = new Vector<>();
	         * 2、List<String> list = Collections.synchronizedList(new ArrayList<>());
	         * 3、List<String> list = new CopyOnWriteArrayList<>()；
	         */
	        // CopyOnWrite 写入时复制  COW  计算机程序设计领域的一种优化策略；
	        // 多个线程调用的时候，list，读取的时候，固定的，写入（覆盖）
	        // 在写入的时候避免覆盖，造成数据问题！
	        // 读写分离
	        // CopyOnWriteArrayList  比 Vector Nb 在哪里？
	
	        List<String> list = new CopyOnWriteArrayList<>();
	
	        for (int i = 1; i <= 10; i++) {
	            new Thread(()->{
	                list.add(UUID.randomUUID().toString().substring(0,5));
	                System.out.println(list);
	            },String.valueOf(i)).start();
	        }
	
	    }
	}

![](https://s3.ax1x.com/2020/12/24/r2WMRI.md.png)

*** 

#### Set不安全

	import java.util.Collections;
	import java.util.HashSet;
	import java.util.Set;
	import java.util.UUID;
	import java.util.concurrent.CopyOnWriteArraySet;
	
	/**
	 * 同理可证 ： ConcurrentModificationException
	 * //1、Set<String> set = Collections.synchronizedSet(new HashSet<>());
	 * //2、
	 */
	public class SetTest {
	    public static void main(String[] args) {
	        Set<String> set = new HashSet<>();
	        // hashmap
	        // Set<String> set = Collections.synchronizedSet(new HashSet<>());
	        // Set<String> set = new CopyOnWriteArraySet<>();
	
	        for (int i = 1; i <=30 ; i++) {
	           new Thread(()->{
	               set.add(UUID.randomUUID().toString().substring(0,5));
	               System.out.println(set);
	           },String.valueOf(i)).start();
	        }
	
	    }
	}

#### Map 不安全
	
	import java.util.Collections;
	import java.util.HashMap;
	import java.util.Map;
	import java.util.UUID;
	import java.util.concurrent.ConcurrentHashMap;
	
	// ConcurrentModificationException
	public class MapTest {
	
	    public static void main(String[] args) {
	        // map 是这样用的吗？ 不是，工作中不用 HashMap
	        // 默认等价于什么？  new HashMap<>(16,0.75);
	        // Map<String, String> map = new HashMap<>();
	        // 唯一的一个家庭作业：研究ConcurrentHashMap的原理
	        Map<String, String> map = new ConcurrentHashMap<>();
	
	        for (int i = 1; i <=30; i++) {
	            new Thread(()->{
	                map.put(Thread.currentThread().getName(),UUID.randomUUID().toString().substring(0,5));
	                System.out.println(map);
	            },String.valueOf(i)).start();
	        }
	
	    }
	}



## 常用的辅助类(必会)

### CountDownLatch

![](https://s3.ax1x.com/2020/12/24/r2Wede.md.png)

	
	import java.util.concurrent.CountDownLatch;
	
	// 计数器
	public class CountDownLatchDemo {
	    public static void main(String[] args) throws InterruptedException {
	        // 总数是6，必须要执行任务的时候，再使用！
	        CountDownLatch countDownLatch = new CountDownLatch(6);
	
	        for (int i = 1; i <=6 ; i++) {
	            new Thread(()->{
	                System.out.println(Thread.currentThread().getName()+" Go out");
	                countDownLatch.countDown(); // 数量-1
	            },String.valueOf(i)).start();
	        }
	
	        countDownLatch.await(); // 等待计数器归零，然后再向下执行
	
	        System.out.println("Close Door");
	
	    }
	}

**原理：**

countDownLatch.countDown(); // 数量-1

countDownLatch.await(); // 等待计数器归零，然后再向下执行

每次有线程调用 countDown() 数量-1，假设计数器变为0，countDownLatch.await() 就会被唤醒，继续执行！


### CyclicBarrier

![](https://s3.ax1x.com/2020/12/24/r2WmIH.md.png)

	import java.util.concurrent.BrokenBarrierException;
	import java.util.concurrent.CyclicBarrier;
	
	public class CyclicBarrierDemo {
	    public static void main(String[] args) {
	        /**
	         * 集齐7颗龙珠召唤神龙
	         */
	        // 召唤龙珠的线程
	        CyclicBarrier cyclicBarrier = new CyclicBarrier(8,()->{
	            System.out.println("召唤神龙成功！");
	        });
	
	        for (int i = 1; i <=7 ; i++) {
	            final int temp = i;
	            // lambda能操作到 i 吗
	            new Thread(()->{
	                System.out.println(Thread.currentThread().getName()+"收集"+temp+"个龙珠");
	                try {
	                    cyclicBarrier.await(); // 等待
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                } catch (BrokenBarrierException e) {
	                    e.printStackTrace();
	                }
	            }).start();
	        }
	
	
	
	    }
	}

### Semaphore

Semaphore：信号量

6车---3个停车位置

	import java.util.concurrent.Semaphore;
	import java.util.concurrent.TimeUnit;
	
	public class SemaphoreDemo {
	    public static void main(String[] args) {
	        // 线程数量：停车位! 限流！
	        Semaphore semaphore = new Semaphore(3);
	
	        for (int i = 1; i <=6 ; i++) {
	            new Thread(()->{
	                // acquire() 得到
	                try {
	                    semaphore.acquire();
	                    System.out.println(Thread.currentThread().getName()+"抢到车位");
	                    TimeUnit.SECONDS.sleep(2);
	                    System.out.println(Thread.currentThread().getName()+"离开车位");
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                } finally {
	                    semaphore.release(); // release() 释放
	                }
	
	            },String.valueOf(i)).start();
	        }
	
	    }
	}

**原理：**

semaphore.acquire() 获得，假设如果已经满了，等待，等待被释放为止！

semaphore.release(); 释放，会将当前的信号量释放 + 1，然后唤醒等待的线程！

**作用：**

多个共享资源互斥的使用！并发限流，控制最大的线程数！

## 读写锁

ReadWriteLock

读的时候可以被多线程读，写的时候只能有一个线程去写。

A ReadWriteLock维护一对关联的locks ，一个用于只读操作，一个用于写入。 read lock可以由多个阅读器线程同时进行，只要没有作者。 write lock是独家的。 

所有ReadWriteLock实现必须保证的存储器同步效应writeLock操作（如在指定Lock接口）也保持相对于所述相关联的readLock 。 也就是说，一个线程成功获取读锁定将会看到在之前发布的写锁定所做的所有更新。 

读写锁允许访问共享数据时的并发性高于互斥锁所允许的并发性。 它利用了这样一个事实：一次只有一个线程（ 写入线程）可以修改共享数据，在许多情况下，任何数量的线程都可以同时读取数据（因此读取器线程）。 从理论上讲，通过使用读写锁允许的并发性增加将导致性能改进超过使用互斥锁。 实际上，并发性的增加只能在多处理器上完全实现，然后只有在共享数据的访问模式是合适的时才可以。 

读写锁是否会提高使用互斥锁的性能取决于数据被读取的频率与被修改的频率相比，读取和写入操作的持续时间以及数据的争用 - 即是，将尝试同时读取或写入数据的线程数。 例如，最初填充数据的集合，然后经常被修改的频繁搜索（例如某种目录）是使用读写锁的理想候选。 然而，如果更新变得频繁，那么数据的大部分时间将被专门锁定，并且并发性增加很少。 此外，如果读取操作太短，则读写锁定实现（其本身比互斥锁更复杂）的开销可以支配执行成本，特别是因为许多读写锁定实现仍将序列化所有线程通过小部分代码。 最终，只有剖析和测量将确定使用读写锁是否适合您的应用程序。 


### 使用

虽然读写锁的基本操作是直接的，但是执行必须做出许多策略决策，这可能会影响给定应用程序中读写锁定的有效性。 这些政策的例子包括： 

* 在写入器释放写入锁定时，确定在读取器和写入器都在等待时是否授予读取锁定或写入锁定。 作家偏好是常见的，因为写作预计会很短，很少见。 读者喜好不常见，因为如果读者经常和长期的预期，写作可能导致漫长的延迟。 公平的或“按顺序”的实现也是可能的。 
* 确定在读卡器处于活动状态并且写入器正在等待时请求读取锁定的读取器是否被授予读取锁定。 读者的偏好可以无限期地拖延作者，而对作者的偏好可以减少并发的潜力。 
* 确定锁是否可重入：一个具有写锁的线程是否可以重新获取？ 持有写锁可以获取读锁吗？ 读锁本身是否可重入？ 
* 写入锁可以降级到读锁，而不允许插入写者？ 读锁可以升级到写锁，优先于其他等待读者或作者吗？ 

在评估应用程序的给定实现的适用性时，应考虑所有这些问题。 

	/**
	 * 独占锁（写锁） 一次只能被一个线程占有
	 * 共享锁（读锁） 多个线程可以同时占有
	 * ReadWriteLock
	 * 读-读  可以共存！
	 * 读-写  不能共存！
	 * 写-写  不能共存！
	 */
		
	/**
	 * 自定义缓存
	 */
	class MyCache{
	
	
	    private volatile Map<String,Object> map = new HashMap<>();
	
	    // 读写锁： 更加细粒度的控制
	    private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
	
	
	    // 存，写入的时候，只希望同时只有一个线程写
	    public void put(String key,Object value){
	        readWriteLock.writeLock().lock();
	        try {
	            System.out.println(Thread.currentThread().getName()+"写入"+key);
	            map.put(key,value);
	            System.out.println(Thread.currentThread().getName()+"写入OK");
	        } catch (Exception e) {
	            e.printStackTrace();
	        } finally {
	            readWriteLock.writeLock().unlock();
	        }
	    }
	
	    // 取，读，所有人都可以读！
	    public void get(String key){
	        readWriteLock.readLock().lock();
	        try {
	            System.out.println(Thread.currentThread().getName()+"读取"+key);
	            Object o = map.get(key);
	            System.out.println(Thread.currentThread().getName()+"读取OK");
	        } catch (Exception e) {
	            e.printStackTrace();
	        } finally {
	            readWriteLock.readLock().unlock();
	        }
	    }
	}



	class Thread1 extends Thread{
		MyCache myCache = new MyCache();
		
		public Thread1(MyCache myCache){
			super();
			this.myCache = myCache;
		}
		
		public void run(){
	  	myCache.put(Thread.currentThread().getName(),Thread.currentThread().getName());
	  }
			
	}
	
	class Thread2 extends Thread{
		MyCache myCache = new MyCache();
	
		public Thread2(MyCache myCache){
			super();
			this.myCache = myCache;
		}
		
		
		public void run(){
			synchronized(this){
				myCache.get(Thread.currentThread().getName());
			} 
		}
	}
	
	public class ReadWriteLockDemo {
	    public static void main(String[] args) {
	    	MyCache myCache = new MyCache();
	        // 写入
	        for (int i = 1; i <= 5 ; i++) {
	            Thread1 t = new Thread1(myCache);
	            t.setName(String.valueOf(i));
	            t.start();
	        }
	//
	        // 读取
	        for (int i = 1; i <= 5 ; i++) {
	            Thread2 t = new Thread2(myCache);
	            t.setName(String.valueOf(i));
	            t.start();
	        }
	}

## 阻塞队列

![](https://s3.ax1x.com/2020/12/24/r2WZZD.md.png)

![](https://s3.ax1x.com/2020/12/24/r2WEqO.md.png)

![](https://s3.ax1x.com/2020/12/24/r2WkM6.md.png)

![](https://s3.ax1x.com/2020/12/24/r2WAsK.md.png)

### 使用情况

多线程并发处理，线程池！

### 四组API

![](https://s3.ax1x.com/2020/12/24/r2Wixx.md.png)

	/**
	 * 抛出异常
	 */
	public static void test1(){
	        // 队列的大小
	        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue<>(3);
	
	        System.out.println(blockingQueue.add("a"));
	        System.out.println(blockingQueue.add("b"));
	        System.out.println(blockingQueue.add("c"));
	        // IllegalStateException: Queue full 抛出异常！
	        // System.out.println(blockingQueue.add("d"));
	
	        System.out.println("=-===========");
	
	        System.out.println(blockingQueue.element()); // 查看队首元素是谁
	        System.out.println(blockingQueue.remove());
	
	
	        System.out.println(blockingQueue.remove());
	        System.out.println(blockingQueue.remove());
	
	        // java.util.NoSuchElementException 抛出异常！
	        // System.out.println(blockingQueue.remove());
	    }

***

    /**
     * 有返回值，没有异常
     */
    public static void test2(){
        // 队列的大小
        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue<>(3);

        System.out.println(blockingQueue.offer("a"));
        System.out.println(blockingQueue.offer("b"));
        System.out.println(blockingQueue.offer("c"));

        System.out.println(blockingQueue.peek());
        // System.out.println(blockingQueue.offer("d")); // false 不抛出异常！
        System.out.println("============================");
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll()); // null  不抛出异常！
    }

![](https://s3.ax1x.com/2020/12/24/r2W9i9.png)

***

    /**
     * 等待，阻塞（一直阻塞）
     */
    public static void test3() throws InterruptedException {
        // 队列的大小
        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue<>(3);

        // 一直阻塞
        blockingQueue.put("a");
        blockingQueue.put("b");
        blockingQueue.put("c");
        // blockingQueue.put("d"); // 队列没有位置了，一直阻塞
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take()); // 没有这个元素，一直阻塞

    }

![](https://s3.ax1x.com/2020/12/24/r2WCGR.md.png)

***


    /**
     * 等待，阻塞（等待超时）
     */
    public static void test4() throws InterruptedException {
        // 队列的大小
        ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue<>(3);

        blockingQueue.offer("a");
        blockingQueue.offer("b");
        blockingQueue.offer("c");
        // blockingQueue.offer("d",2,TimeUnit.SECONDS); // 等待超过2秒就退出
        System.out.println("===============");
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        blockingQueue.poll(2,TimeUnit.SECONDS); // 等待超过2秒就退出

    }

![](https://s3.ax1x.com/2020/12/24/r2WSIJ.md.png)

### SynchronousQueue 同步队列

没有容量，进去一个元素，必须等待取出来之后，才能再往里面放一个元素！


	import java.sql.Time;
	import java.util.concurrent.BlockingQueue;
	import java.util.concurrent.SynchronousQueue;
	import java.util.concurrent.TimeUnit;
	
	/**
	 * 同步队列
	 * 和其他的BlockingQueue 不一样， SynchronousQueue 不存储元素
	 * put了一个元素，必须从里面先take取出来，否则不能在put进去值！
	 */
	public class SynchronousQueueDemo {
	    public static void main(String[] args) {
	        BlockingQueue<String> blockingQueue = new SynchronousQueue<>(); // 同步队列
	
	        new Thread(()->{
	            try {
	                System.out.println(Thread.currentThread().getName()+" put 1");
	                blockingQueue.put("1");
	                System.out.println(Thread.currentThread().getName()+" put 2");
	                blockingQueue.put("2");
	                System.out.println(Thread.currentThread().getName()+" put 3");
	                blockingQueue.put("3");
	            } catch (InterruptedException e) {
	                e.printStackTrace();
	            }
	        },"T1").start();
	
	
	        new Thread(()->{
	            try {
	                TimeUnit.SECONDS.sleep(3);
	                System.out.println(Thread.currentThread().getName()+"=>"+blockingQueue.take());
	                TimeUnit.SECONDS.sleep(3);
	                System.out.println(Thread.currentThread().getName()+"=>"+blockingQueue.take());
	                TimeUnit.SECONDS.sleep(3);
	                System.out.println(Thread.currentThread().getName()+"=>"+blockingQueue.take());
	            } catch (InterruptedException e) {
	                e.printStackTrace();
	            }
	        },"T2").start();
	    }
	}

![](https://s3.ax1x.com/2020/12/24/r2Rza4.md.png)

## 线程池(重点)

线程池：三大方法、7大参数、4种拒绝策略

程序的运行，本质：占用系统的资源！ 优化资源的使用！=>池化技术

线程池、连接池、内存池、对象池///..... 创建、销毁。十分浪费资源

池化技术：事先准备好一些资源，有人要用，就来我这里拿，用完之后还给我。

### 线程池的好处:

1. 降低资源的消耗
2. 提高响应的速度
3. 方便管理

线程复用、可以控制最大并发数、管理线程

### 三大方法

![](https://s3.ax1x.com/2020/12/24/r2RxZF.md.png)
	
	// Executors 工具类、3大方法
	public class Demo01 {
		public static void main(String[] args) {
			myThread1 myThread = new myThread1();
			//ExecutorService threadPool = Executors.newSingleThreadExecutor();// 单个线程
			// ExecutorService threadPool = Executors.newFixedThreadPool(5); // 创建一个固定的线程池的大小
			 ExecutorService threadPool = Executors.newCachedThreadPool(); // 可伸缩的，遇强则强，遇弱则弱
			try {
				for (int i = 0; i < 100; i++) {
					// 使用了线程池之后，使用线程池来创建线程
					threadPool.execute(myThread);
				}
			} catch (Exception e) {
			e.printStackTrace();
			} finally {
			// 线程池用完，程序结束，关闭线程池
			threadPool.shutdown();
			}
		}
	}
	
	class myThread1 extends Thread{
		public void run(){
			System.out.println(Thread.currentThread().getName()+" ok");
		}
	}

![](https://s3.ax1x.com/2020/12/24/r2RjqU.md.png)

![](https://s3.ax1x.com/2020/12/24/r2RXrT.md.png)

### 7大参数


    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }


***

    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }


***
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }

***

    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }

***

    public ThreadPoolExecutor(int corePoolSize,// 核心线程池大小
                              int maximumPoolSize,// 最大核心线程池大小
                              long keepAliveTime,// 超时了没有人调用就会释放
                              TimeUnit unit,// 超时单位
                              BlockingQueue<Runnable> workQueue,// 阻塞队列
                              ThreadFactory threadFactory,// 线程工厂：创建线程的，一般不用动
                              RejectedExecutionHandler handler// 拒绝策略) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }

### 4种拒绝策略

![](https://s3.ax1x.com/2020/12/24/r2RHGn.md.png)

	/**
	* new ThreadPoolExecutor.AbortPolicy() // 银行满了，还有人进来，不处理这个人的，抛出异常
	* new ThreadPoolExecutor.CallerRunsPolicy() // 哪来的去哪里！
	* new ThreadPoolExecutor.DiscardPolicy() //队列满了，丢掉任务，不会抛出异常！
	* new ThreadPoolExecutor.DiscardOldestPolicy() //队列满了，尝试去和最早的竞争，也不会抛出异常！
	*/

### 池的最大的大小如何去设置

了解：IO密集型，CPU密集型：（调优）


### 自定义线程池

	
	// Executors 工具类、3大方法
	
	/**
	 * new ThreadPoolExecutor.AbortPolicy() // 银行满了，还有人进来，不处理这个人的，抛出异常
	 * new ThreadPoolExecutor.CallerRunsPolicy() // 哪来的去哪里！
	 * new ThreadPoolExecutor.DiscardPolicy() //队列满了，丢掉任务，不会抛出异常！
	 * new ThreadPoolExecutor.DiscardOldestPolicy() //队列满了，尝试去和最早的竞争，也不会抛出异常！
	 */
	public class Demo01 {
	    public static void main(String[] args) {
	        // 自定义线程池！工作 ThreadPoolExecutor
	
	        // 最大线程到底该如何定义
	        // 1、CPU 密集型，几核，就是几，可以保持CPU的效率最高！
	        // 2、IO  密集型   > 判断你程序中十分耗IO的线程，
	        // 程序   15个大型任务  io十分占用资源！
	
	        // 获取CPU的核数
	        System.out.println(Runtime.getRuntime().availableProcessors());
	
	        List  list = new ArrayList();
	
	        ExecutorService threadPool = new ThreadPoolExecutor(
	                2,
	                Runtime.getRuntime().availableProcessors(),
	                3,
	                TimeUnit.SECONDS,
	                new LinkedBlockingDeque<>(3),
	                Executors.defaultThreadFactory(),
	                new ThreadPoolExecutor.DiscardOldestPolicy());  //队列满了，尝试去和最早的竞争，也不会抛出异常！
	        try {
	            // 最大承载：Deque + max
	            // 超过 RejectedExecutionException
	            for (int i = 1; i <= 9; i++) {
	                // 使用了线程池之后，使用线程池来创建线程
	                threadPool.execute(()->{
	                    System.out.println(Thread.currentThread().getName()+" ok");
	                });
	            }
	
	        } catch (Exception e) {
	            e.printStackTrace();
	        } finally {
	            // 线程池用完，程序结束，关闭线程池
	            threadPool.shutdown();
	        }
	
	    }
	}


## 四大函数式接口（必需掌握）

![](https://s3.ax1x.com/2020/12/24/r2R7Ps.png)

lambda表达式、链式编程、函数式接口、Stream流式计算

函数式接口： 只有一个方法的接口

	@FunctionalInterface
	public interface Runnable {
		public abstract void run();
	}
	// 泛型、枚举、反射
	// lambda表达式、链式编程、函数式接口、Stream流式计算
	// 超级多FunctionalInterface
	// 简化编程模型，在新版本的框架底层大量应用！
	// foreach(消费者类的函数式接口)

### Function函数式接口


![](https://s3.ax1x.com/2020/12/24/r2ROMV.png)

	/**
	 * Function 函数型接口, 有一个输入参数，有一个输出
	 * 只要是 函数型接口 可以 用 lambda表达式简化
	 */
	public class Demo01 {
	    public static void main(String[] args) {
	        //
	//        Function<String,String> function = new Function<String,String>() {
	//            @Override
	//            public String apply(String str) {
	//                return str;
	//            }
	//        };
	
	        Function<String,String> function = str->{return str;};
	
	        System.out.println(function.apply("asd"));
	    }
	}

### 断定型接口：有一个输入参数，返回值只能是 布尔值！

![](https://s3.ax1x.com/2020/12/24/r2Rb2q.png)
	
	/**
	 * 断定型接口：有一个输入参数，返回值只能是 布尔值！
	 */
	public class Demo02 {
	    public static void main(String[] args) {
	        // 判断字符串是否为空
	//        Predicate<String> predicate = new Predicate<String>(){
	////            @Override
	////            public boolean test(String str) {
	////                return str.isEmpty();
	////            }
	////        };
	
	        Predicate<String> predicate = (str)->{return str.isEmpty(); };
	        System.out.println(predicate.test(""));
	
	    }
	}

### Consumer 消费型接口

![](https://s3.ax1x.com/2020/12/24/r2Rqx0.png)

	/**
	 * Consumer 消费型接口: 只有输入，没有返回值
	 */
	public class Demo03 {
	    public static void main(String[] args) {
	//        Consumer<String> consumer = new Consumer<String>() {
	//            @Override
	//            public void accept(String str) {
	//                System.out.println(str);
	//            }
	//        };
	        Consumer<String> consumer = (str)->{System.out.println(str);};
	        consumer.accept("sdadasd");
	
	    }
	}

### Supplier 供给型接口

![](https://s3.ax1x.com/2020/12/25/rfpCsP.png)

	import java.util.function.Supplier;
	
	/**
	 * Supplier 供给型接口 没有参数，只有返回值
	 */
	public class Demo04 {
	    public static void main(String[] args) {
	//        Supplier supplier = new Supplier<Integer>() {
	//            @Override
	//            public Integer get() {
	//                System.out.println("get()");
	//                return 1024;
	//            }
	//        };
	
	        Supplier supplier = ()->{ return 1024; };
	        System.out.println(supplier.get());
	    }
	}

## Stream流式计算

### 什么是Stream流式计算

大数据：存储 + 计算

集合、MySQL 本质就是存储东西的；

计算都应该交给流来操作！

![](https://s3.ax1x.com/2020/12/25/rfSLVO.png)

import java.util.Arrays;
import java.util.List;
	
	/**
	 * 题目要求：一分钟内完成此题，只能用一行代码实现！
	 * 现在有5个用户！筛选：
	 * 1、ID 必须是偶数
	 * 2、年龄必须大于23岁
	 * 3、用户名转为大写字母
	 * 4、用户名字母倒着排序
	 * 5、只输出一个用户！
	 */
	public class Test {
	    public static void main(String[] args) {
	        User u1 = new User(1,"a",21);
	        User u2 = new User(2,"b",22);
	        User u3 = new User(3,"c",23);
	        User u4 = new User(4,"d",24);
	        User u5 = new User(6,"e",25);
	        // 集合就是存储
	        List<User> list = Arrays.asList(u1, u2, u3, u4, u5);
	
	        // 计算交给Stream流
	        // lambda表达式、链式编程、函数式接口、Stream流式计算
	        list.stream()
	                .filter(u->{return u.getId()%2==0;})
	                .filter(u->{return u.getAge()>23;})
	                .map(u->{return u.getName().toUpperCase();})
	                .sorted((uu1,uu2)->{return uu2.compareTo(uu1);})
	                .limit(1)
	                .forEach(System.out::println);
	    }
	}

## ForkJoin

### 什么是 ForkJoin

ForkJoin 在 JDK 1.7 ， 并行执行任务！提高效率。大数据量！

大数据：Map Reduce （把大任务拆分为小任务）

![](https://s3.ax1x.com/2020/12/25/rfSOaD.png)

### ForkJoin 特点：

工作窃取

这个里面维护的都是双端队列

![](https://s3.ax1x.com/2020/12/25/rfSXIe.png)

![](https://s3.ax1x.com/2020/12/24/r2WDyV.png)

![](https://s3.ax1x.com/2020/12/24/r2WrLT.png)

	import java.util.concurrent.RecursiveTask;
	
	/**
	 * 求和计算的任务！
	 * 3000   6000（ForkJoin）  9000（Stream并行流）
	 * // 如何使用 forkjoin
	 * // 1、forkjoinPool 通过它来执行
	 * // 2、计算任务 forkjoinPool.execute(ForkJoinTask task)
	 * // 3. 计算类要继承 ForkJoinTask
	 */
	public class ForkJoinDemo extends RecursiveTask<Long> {
	
	    private Long start;  // 1
	    private Long end;    // 1990900000
	
	    // 临界值
	    private Long temp = 10000L;
	
	    public ForkJoinDemo(Long start, Long end) {
	        this.start = start;
	        this.end = end;
	    }
	
	    // 计算方法
	    @Override
	    protected Long compute() {
	        if ((end-start)<temp){
	            Long sum = 0L;
	            for (Long i = start; i <= end; i++) {
	                sum += i;
	            }
	            return sum;
	        }else { // forkjoin 递归
	            long middle = (start + end) / 2; // 中间值
	            ForkJoinDemo task1 = new ForkJoinDemo(start, middle);
	            task1.fork(); // 拆分任务，把任务压入线程队列
	            ForkJoinDemo task2 = new ForkJoinDemo(middle+1, end);
	            task2.fork(); // 拆分任务，把任务压入线程队列
	
	            return task1.join() + task2.join();
	        }
	    }
	}

***
	
	import java.util.concurrent.ExecutionException;
	import java.util.concurrent.ForkJoinPool;
	import java.util.concurrent.ForkJoinTask;
	import java.util.stream.DoubleStream;
	import java.util.stream.IntStream;
	import java.util.stream.LongStream;
	
	/**
	 * 同一个任务，别人效率高你几十倍！
	 */
	public class Test {
	    public static void main(String[] args) throws ExecutionException, InterruptedException {
	        // test1(); // 12224
	        // test2(); // 10038
	        // test3(); // 153
	    }
	
	    // 普通程序员
	    public static void test1(){
	        Long sum = 0L;
	        long start = System.currentTimeMillis();
	        for (Long i = 1L; i <= 10_0000_0000; i++) {
	            sum += i;
	        }
	        long end = System.currentTimeMillis();
	        System.out.println("sum="+sum+" 时间："+(end-start));
	    }
	
	    // 会使用ForkJoin
	    public static void test2() throws ExecutionException, InterruptedException {
	        long start = System.currentTimeMillis();
	
	        ForkJoinPool forkJoinPool = new ForkJoinPool();
	        ForkJoinTask<Long> task = new ForkJoinDemo(0L, 10_0000_0000L);
	        ForkJoinTask<Long> submit = forkJoinPool.submit(task);// 提交任务
	        Long sum = submit.get();
	
	        long end = System.currentTimeMillis();
	
	        System.out.println("sum="+sum+" 时间："+(end-start));
	    }
	
	    public static void test3(){
	        long start = System.currentTimeMillis();
	        // Stream并行流 ()  (]
	        long sum = LongStream.rangeClosed(0L, 10_0000_0000L).parallel().reduce(0, Long::sum);
	        long end = System.currentTimeMillis();
	        System.out.println("sum="+"时间："+(end-start));
	    }
	
	}

## 异步回调

Future 设计的初衷： 对将来的某个事件的结果进行建模

![](https://s3.ax1x.com/2020/12/25/rfpFZ8.png)

	import java.util.concurrent.CompletableFuture;
	import java.util.concurrent.ExecutionException;
	import java.util.concurrent.Future;
	import java.util.concurrent.TimeUnit;
	
	/**
	 * 异步调用： CompletableFuture
	 * // 异步执行
	 * // 成功回调
	 * // 失败回调
	 */
	public class Demo01 {
	    public static void main(String[] args) throws ExecutionException, InterruptedException {
	        // 没有返回值的 runAsync 异步回调
	//        CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(()->{
	//            try {
	//                TimeUnit.SECONDS.sleep(2);
	//            } catch (InterruptedException e) {
	//                e.printStackTrace();
	//            }
	//            System.out.println(Thread.currentThread().getName()+"runAsync=>Void");
	//        });
	//
	//        System.out.println("1111");
	//
	//        completableFuture.get(); // 获取阻塞执行结果
	
	        // 有返回值的 supplyAsync 异步回调
	        // ajax，成功和失败的回调
	        // 返回的是错误信息；
	        CompletableFuture<Integer> completableFuture = CompletableFuture.supplyAsync(()->{
	            System.out.println(Thread.currentThread().getName()+"supplyAsync=>Integer");
	            int i = 10/0;
	            return 1024;
	        });
	
	        System.out.println(completableFuture.whenComplete((t, u) -> {
	            System.out.println("t=>" + t); // 正常的返回结果
	            System.out.println("u=>" + u); // 错误信息：java.util.concurrent.CompletionException: java.lang.ArithmeticException: / by zero
	        }).exceptionally((e) -> {
	            System.out.println(e.getMessage());
	            return 233; // 可以获取到错误的返回结果
	        }).get());
	
	        /**
	         * succee Code 200
	         * error Code 404 500
	         */
	    }
	}

![](https://s3.ax1x.com/2020/12/25/rfSxGd.md.png)

## JMM

JMM ： Java内存模型，不存在的东西，概念！约定！

### 关于JMM的一些同步的约定：

1. 线程解锁前，必须把共享变量立刻刷回主存。
2. 线程加锁前，必须读取主存中的最新值到工作内存中！
3. 加锁和解锁是同一把锁

![](https://s3.ax1x.com/2020/12/25/rfpqln.md.png)



内存交互操作有8种，虚拟机实现必须保证每一个操作都是原子的，不可在分的（对于double和long类型的变量来说，load、store、read和write操作在某些平台上允许例外）

1. lock （锁定）：作用于主内存的变量，把一个变量标识为线程独占状态
2. unlock （解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定
3. read （读取）：作用于主内存变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用
4. load （载入）：作用于工作内存的变量，它把read操作从主存中变量放入工作内存中
5. use （使用）：作用于工作内存中的变量，它把工作内存中的变量传输给执行引擎，每当虚拟机遇到一个需要使用到变量的值，就会使用到这个指令
6. assign （赋值）：作用于工作内存中的变量，它把一个从执行引擎中接受到的值放入工作内存的变量副本中
7. store （存储）：作用于主内存中的变量，它把一个从工作内存中一个变量的值传送到主内存中，以便后续的write使用
8. write （写入）：作用于主内存中的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中

JMM对这八种指令的使用，制定了如下规则：

* 不允许read和load、store和write操作之一单独出现。即使用了read必须load，使用了store必须write
* 不允许线程丢弃他最近的assign操作，即工作变量的数据改变了之后，必须告知主存
* 不允许一个线程将没有assign的数据从工作内存同步回主内存
* 一个新的变量必须在主内存中诞生，不允许工作内存直接使用一个未被初始化的变量。就是怼变量实施use、store操作之前，必须经过assign和load操作
* 一个变量同一时间只有一个线程能对其进行lock。多次lock后，必须执行相同次数的unlock才能解锁
* 如果对一个变量进行lock操作，会清空所有工作内存中此变量的值，在执行引擎使用这个变量前，必须重新load或assign操作初始化变量的值
* 如果一个变量没有被lock，就不能对其进行unlock操作。也不能unlock一个被其他线程锁住的变量
* 对一个变量进行unlock操作之前，必须把此变量同步回主内存

问题： 程序不知道主内存的值已经被修改过了

![](https://s3.ax1x.com/2020/12/25/rfpbSs.md.png)

## Volatile

### 保证可见性

Volatile 是 Java 虚拟机提供**轻量级的同步机制**

1. 保证可见性
2. 不保证原子性
3. 禁止指令重排

	
	import java.util.concurrent.TimeUnit;
	
	public class JMMDemo {
	    // 不加 volatile 程序就会死循环！
	    // 加 volatile 可以保证可见性
	    private volatile static int num = 0;
	
	    public static void main(String[] args) { // main
	
	        new Thread(()->{ // 线程 1 对主内存的变化不知道的
	            while (num==0){
	
	            }
	        }).start();
	
	        try {
	            TimeUnit.SECONDS.sleep(1);
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        }
	
	        num = 1;
	        System.out.println(num);
	
	    }
	}

### 不保证原子性

原子性 : 不可分割

线程A在执行任务的时候，不能被打扰的，也不能被分割。要么同时成功，要么同时失败。

	
	// volatile 不保证原子性
	public class VDemo02 {
	
	    // volatile 不保证原子性
	    // 原子类的 Integer
	    private volatile static int num = 0;
	
	    public static void add(){
	        num++; // 不是一个原子性操作

	    }
	
	    public static void main(String[] args) {
	
	        //理论上num结果应该为 2 万
	        for (int i = 1; i <= 20; i++) {
	            new Thread(()->{
	                for (int j = 0; j < 1000 ; j++) {
	                    add();
	                }
	            }).start();
	        }
	
	        while (Thread.activeCount()>2){ // main  gc
	            Thread.yield();
	        }
	
	        System.out.println(Thread.currentThread().getName() + " " + num);
	
	
	    }
	}

如果不加 lock 和 synchronized ，怎么样保证原子性

![](https://s3.ax1x.com/2020/12/25/rfp7Wj.md.png)

使用原子类，解决 原子性问题

![](https://s3.ax1x.com/2020/12/25/rfpTYQ.png)

	import java.util.concurrent.atomic.AtomicInteger;
	
	// volatile 不保证原子性
	public class VDemo02 {
	
	    // volatile 不保证原子性
	    // 原子类的 Integer
	    private volatile static AtomicInteger num = new AtomicInteger();
	
	    public static void add(){
	        // num++; // 不是一个原子性操作
	        num.getAndIncrement(); // AtomicInteger + 1 方法， CAS
	    }
	
	    public static void main(String[] args) {
	
	        //理论上num结果应该为 2 万
	        for (int i = 1; i <= 20; i++) {
	            new Thread(()->{
	                for (int j = 0; j < 1000 ; j++) {
	                    add();
	                }
	            }).start();
	        }
	
	        while (Thread.activeCount()>2){ // main  gc
	            Thread.yield();
	        }
	
	        System.out.println(Thread.currentThread().getName() + " " + num);
	
	
	    }
	}

这些类的底层都直接和操作系统挂钩！在内存中修改值！Unsafe类是一个很特殊的存在！

### 指令重排

什么是 指令重排：你写的程序，计算机并不是按照你写的那样去执行的。

源代码-->编译器优化的重排--> 指令并行也可能会重排--> 内存系统也会重排---> 执行

处理器在进行指令重排的时候，考虑：数据之间的依赖性！

    int x = 1; // 1
    int y = 2; // 2
    x = x + 5; // 3
    y = x * x; // 4

我们所期望的：1234 但是可能执行的时候回变成 2134 1324

不可能是 4123！

可能造成影响的结果： a b x y 这四个值默认都是 0；

![](https://s3.ax1x.com/2020/12/25/rfpoFg.png)

![](https://s3.ax1x.com/2020/12/25/rfp5TS.png)

指令重排导致的诡异结果： x = 2；y = 1；

**volatile可以避免指令重排：**

内存屏障。CPU指令。作用：

1. 保证特定的操作的执行顺序！
2. 可以保证某些变量的内存可见性 （利用这些特性volatile实现了可见性）

![](https://s3.ax1x.com/2020/12/25/rfp4w8.png)

Volatile 是可以保持 可见性。不能保证原子性，由于**内存屏障**，可以保证避免指令重排的现象产生！

## 彻底玩转单例模式

饿汉式 DCL懒汉式，深究！

### DCL懒汉式

	// 懒汉式单例
	// 道高一尺，魔高一丈！
	public class LazyMan {
	
	    private static boolean qinjiang = false;
	
	    private LazyMan(){
	        synchronized (LazyMan.class){
	            if (qinjiang == false){
	                qinjiang = true;
	            }else {
	                throw new RuntimeException("不要试图使用反射破坏异常");
	            }
	        }
	    }
	
	    private volatile static LazyMan lazyMan;
	
	    // 双重检测锁模式的 懒汉式单例  DCL懒汉式
	    public static LazyMan getInstance(){
	        if (lazyMan==null){
	            synchronized (LazyMan.class){
	                if (lazyMan==null){
	                    lazyMan = new LazyMan(); // 不是一个原子性操作
	                }
	            }
	        }
	        return lazyMan;
	    }
	
	    // 反射！
	    public static void main(String[] args) throws Exception {
	//        LazyMan instance = LazyMan.getInstance();
	
	        Field qinjiang = LazyMan.class.getDeclaredField("qinjiang");
	        qinjiang.setAccessible(true);
	
	        Constructor<LazyMan> declaredConstructor = LazyMan.class.getDeclaredConstructor(null);
	        declaredConstructor.setAccessible(true);
	        LazyMan instance = declaredConstructor.newInstance();
	
	        qinjiang.set(instance,false);
	
	        LazyMan instance2 = declaredConstructor.newInstance();
	
	        System.out.println(instance);
	        System.out.println(instance2);
	    }
	
	}
	
	
	
	
	/**
	 * 1. 分配内存空间
	 * 2、执行构造方法，初始化对象
	 * 3、把这个对象指向这个空间
	 *
	 * 123
	 * 132 A
	 *     B // 此时lazyMan还没有完成构造
	 */

### 枚举

单例不安全，反射

	import java.lang.reflect.Constructor;
	import java.lang.reflect.InvocationTargetException;
	
	// enum 是一个什么？ 本身也是一个Class类
	public enum EnumSingle {
	
	    INSTANCE;
	
	    public EnumSingle getInstance(){
	        return INSTANCE;
	    }
	
	}
	
	class Test{
	
	    public static void main(String[] args) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
	        EnumSingle instance1 = EnumSingle.INSTANCE;
	        Constructor<EnumSingle> declaredConstructor = EnumSingle.class.getDeclaredConstructor(String.class,int.class);
	        declaredConstructor.setAccessible(true);
	        EnumSingle instance2 = declaredConstructor.newInstance();
	
	        // NoSuchMethodException: com.kuang.single.EnumSingle.<init>()
	        System.out.println(instance1);
	        System.out.println(instance2);
	
	    }
	
	}

![](https://s3.ax1x.com/2020/12/25/rfphef.png)

枚举类型的最终反编译源码:

![](https://s3.ax1x.com/2020/12/25/rfpRyt.png)

## 深入理解CAS

![](https://s3.ax1x.com/2020/12/25/rfpWOP.png)

### Unsafe类

![](https://s3.ax1x.com/2020/12/25/rfp2QI.png)

![](https://s3.ax1x.com/2020/12/25/rfpgSA.png)

CAS ： 比较当前工作内存中的值和主内存中的值，如果这个值是期望的，那么则执行操作！如果不是就一直循环！

#### 缺点

1. 循环会耗时
2. 一次性只能保证一个共享变量的原子性
3. ABA问题

### CAS ： ABA 问题（狸猫换太子）

![](https://s3.ax1x.com/2020/12/25/rfp6Wd.png)

![](https://s3.ax1x.com/2020/12/25/rfpyJH.png)

## 原子引用

解决ABA 问题，引入原子引用！ 对应的思想：乐观锁！

带版本号的原子操作

	import java.util.concurrent.TimeUnit;
	import java.util.concurrent.atomic.AtomicStampedReference;
	import java.util.concurrent.locks.Lock;
	import java.util.concurrent.locks.ReentrantLock;
	
	public class CASDemo {
	
	    //AtomicStampedReference 注意，如果泛型是一个包装类，注意对象的引用问题
	
	    // 正常在业务操作，这里面比较的都是一个个对象
	    static AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(1,1);
	
	    // CAS  compareAndSet : 比较并交换！
	    public static void main(String[] args) {
	
	        new Thread(()->{
	            int stamp = atomicStampedReference.getStamp(); // 获得版本号
	            System.out.println("a1=>"+stamp);
	
	            try {
	                TimeUnit.SECONDS.sleep(1);
	            } catch (InterruptedException e) {
	                e.printStackTrace();
	            }
	
	            Lock lock = new ReentrantLock(true);
	
	            atomicStampedReference.compareAndSet(1, 2,
	                    atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1);
	
	            System.out.println("a2=>"+atomicStampedReference.getStamp());
	
	
	            System.out.println(atomicStampedReference.compareAndSet(2, 1,
	                    atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1));
	
	            System.out.println("a3=>"+atomicStampedReference.getStamp());
	
	        },"a").start();
	
	
	        // 乐观锁的原理相同！
	        new Thread(()->{
	            int stamp = atomicStampedReference.getStamp(); // 获得版本号
	            System.out.println("b1=>"+stamp);
	
	            try {
	                TimeUnit.SECONDS.sleep(2);
	            } catch (InterruptedException e) {
	                e.printStackTrace();
	            }
	
	            System.out.println(atomicStampedReference.compareAndSet(1, 6,
	                    stamp, stamp + 1));
	
	            System.out.println("b2=>"+atomicStampedReference.getStamp());
	
	        },"b").start();
	
	    }
	}

注意：
Integer 使用了对象缓存机制，默认范围是 -128 ~ 127 ，推荐使用静态工厂方法 valueOf 获取对象实例，而不是 new，因为 valueOf 使用缓存，而 new 一定会创建新的对象分配新的内存空间；

![](https://s3.ax1x.com/2020/12/25/rfpsFe.png)

## 各种锁的理解

### 公平锁、非公平锁

公平锁： 非常公平， 不能够插队，必须先来后到！

非公平锁：非常不公平，可以插队 （默认都是非公平）

	public ReentrantLock() {
		sync = new NonfairSync();
	}
	public ReentrantLock(boolean fair) {
		sync = fair ? new FairSync() : new NonfairSync();
	}

### 可重入锁

可重入锁（递归锁）

![](https://s3.ax1x.com/2020/12/25/rfpBdO.png)

#### Synchronized
	
	
	import javax.sound.midi.Soundbank;
	
	// Synchronized
	public class Demo01 {
	    public static void main(String[] args) {
	        Phone phone = new Phone();
	
	        new Thread(()->{
	            phone.sms();
	        },"A").start();
	
	
	        new Thread(()->{
	            phone.sms();
	        },"B").start();
	    }
	}
	
	class Phone{
	
	    public synchronized void sms(){
	        System.out.println(Thread.currentThread().getName() + "sms");
	        call(); // 这里也有锁
	    }
	
	    public synchronized void call(){
	        System.out.println(Thread.currentThread().getName() + "call");
	    }
	}

![](https://s3.ax1x.com/2020/12/25/rfpDoD.md.png)


#### Lock 版
	
	import java.util.concurrent.locks.Lock;
	import java.util.concurrent.locks.ReentrantLock;
	
	public class Demo02 {
	    public static void main(String[] args) {
	        Phone2 phone = new Phone2();
	
	        new Thread(()->{
	            phone.sms();
	        },"A").start();
	
	
	        new Thread(()->{
	            phone.sms();
	        },"B").start();
	    }
	}
	
	class Phone2{
	    Lock lock = new ReentrantLock();
	
	    public void sms(){
	        lock.lock(); // 细节问题：lock.lock(); lock.unlock(); // lock 锁必须配对，否则就会死在里面
	        lock.lock();
	        try {
	            System.out.println(Thread.currentThread().getName() + "sms");
	            call(); // 这里也有锁
	        } catch (Exception e) {
	            e.printStackTrace();
	        } finally {
	            lock.unlock();
	            lock.unlock();
	        }
	
	    }
	
	    public void call(){
	
	        lock.lock();
	        try {
	            System.out.println(Thread.currentThread().getName() + "call");
	        } catch (Exception e) {
	            e.printStackTrace();
	        } finally {
	            lock.unlock();
	        }
	    }
	}

### 自旋锁

![](https://s3.ax1x.com/2020/12/25/rf98nP.png)


	import java.util.concurrent.atomic.AtomicReference;
	
	/**
	 * 自旋锁
	 */
	public class SpinlockDemo {
	
	    // int   0
	    // Thread  null
	    AtomicReference<Thread> atomicReference = new AtomicReference<>();
	
	    // 加锁
	    public void myLock(){
	        Thread thread = Thread.currentThread();
	        System.out.println(Thread.currentThread().getName() + "==> mylock");
	
	        // 自旋锁
	        while (!atomicReference.compareAndSet(null,thread)){
	
	        }
	    }
	
	
	    // 解锁
	    // 加锁
	    public void myUnLock(){
	        Thread thread = Thread.currentThread();
	        System.out.println(Thread.currentThread().getName() + "==> myUnlock");
	        atomicReference.compareAndSet(thread,null);
	    }
	
	
	
	}

***

测试

	import java.util.concurrent.TimeUnit;
	import java.util.concurrent.locks.ReentrantLock;
	
	public class TestSpinLock {
	    public static void main(String[] args) throws InterruptedException {
	//        ReentrantLock reentrantLock = new ReentrantLock();
	//        reentrantLock.lock();
	//        reentrantLock.unlock();
	
	        // 底层使用的自旋锁CAS
	        SpinlockDemo lock = new SpinlockDemo();
	
	
	        new Thread(()-> {
	            lock.myLock();
	
	            try {
	                TimeUnit.SECONDS.sleep(5);
	            } catch (Exception e) {
	                e.printStackTrace();
	            } finally {
	                lock.myUnLock();
	            }
	
	        },"T1").start();
	
	        TimeUnit.SECONDS.sleep(1);
	
	        new Thread(()-> {
	            lock.myLock();
	
	            try {
	                TimeUnit.SECONDS.sleep(1);
	            } catch (Exception e) {
	                e.printStackTrace();
	            } finally {
	                lock.myUnLock();
	            }
	
	        },"T2").start();
	
	    }
	}

![](https://s3.ax1x.com/2020/12/25/rf9tAS.png)

## 死锁

![](https://s3.ax1x.com/2020/12/25/rfp0eK.png)
	
	import com.sun.org.apache.xpath.internal.SourceTree;
	
	import java.util.concurrent.TimeUnit;
	
	public class DeadLockDemo {
	    public static void main(String[] args) {
	
	        String lockA = "lockA";
	        String lockB = "lockB";
	
	        new Thread(new MyThread(lockA, lockB), "T1").start();
	        new Thread(new MyThread(lockB, lockA), "T2").start();
	
	    }
	}
	
	
	class MyThread implements Runnable{
	
	    private String lockA;
	    private String lockB;
	
	    public MyThread(String lockA, String lockB) {
	        this.lockA = lockA;
	        this.lockB = lockB;
	    }
	
	    @Override
	    public void run() {
	        synchronized (lockA){
	            System.out.println(Thread.currentThread().getName() + "lock:"+lockA+"=>get"+lockB);
	
	            try {
	                TimeUnit.SECONDS.sleep(2);
	            } catch (InterruptedException e) {
	                e.printStackTrace();
	            }
	
	            synchronized (lockB){
	                System.out.println(Thread.currentThread().getName() + "lock:"+lockB+"=>get"+lockA);
	            }
	
	        }
	    }
	}


### 解决问题

#### 使用 jps -l 定位进程号


![](https://s3.ax1x.com/2020/12/25/rfpdL6.png)

#### 使用jstack进程号找到死锁问题

![](https://s3.ax1x.com/2020/12/25/rf9w1s.png)

