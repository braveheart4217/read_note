##java.util.concurrent
Concurrent 包是Java中解决并发问题的包

* TimeUnit
	* 包括所有的时间单位
* CopyOnWriteArrayList
	* 开发人员往往求助于使用同步的ArrayList，来替代创建全新的数组副本，然而这也是一个成本较高的选择(必须使用同步操作保证一致性)
	* CopyOnWriteArrayList适用于 读取频繁，但很少有写操作的集合，JavaDoc将其定义为一份ArrayList的线程安全的变体，在这个变体中所有易变操作（添加，设置等）可以通过复制全新的数组来实现
* BlockingQueue
	* 此接口本质上是一个queue，对于此queue的读取，插入操作会被同步直到资源可用
	* BlockingQueue 解决了如何将一个线程收集的项“传递”给另一线程用于处理的问题，串行化的解决方式

* ConcurrentMap
* SynchronizeQueues
	* 这是一个阻塞队列，每个插入操作必须等待另一个线程的对应移除操作，反之亦然
	* SynchronousQueue 是之前提过的 BlockingQueue 的又一实现，它给我们提供了在线程之间交换单一元素的极轻量级方法

* Semaphore
	* 支持信号量的操作
		
			Semaphore available = new Semaphore(3); // init
			available.acquire();// get semaphore
			available.release();// realease
* CountDownLatch
	* 如果 `Semaphore` 是允许一次进入一个线程的并发性类，那么 `CountDownLatch` 就像是赛马场的起跑门栅。此类持有所有空闲线程，**直到满足特定条件**，这时它将会一次释放所有这些线程
	* 初始的时候可以申请 n个 **门闩** ，调用`await()`函数的线程会阻塞直到门闩的数量为0,调用`countDown()`会使门闩的数量减一

* Executor
	* `Executor` 使您不必亲自对 `Thread` 对象执行 new 就能够创建新线程
	
			Executor exec = getAnExecutorFromSomeplace();
			exec.execute(new Runnable() { ... });
	*	 使用 `Executor` 的主要缺陷在于：工厂必须来自某个位置。不幸的是，与 CLR 不同，JVM 没有附带一个标准的 VM 级线程池
	*	 `ExecutorService` -- 随时可以使用
		*	 `ExecutorService`将线程启动工厂建模为一个可集中控制的服务。例如，无需每执行一项任务就调用一次 `execute()`，`ExecutorService` 可以接受一组任务并返回一个表示每项任务的未来结果的未来列表


* ScheduledExecutorServices
	* 让某些任务按照确定的方式执行，比如以确定的时间间隔或在特定时间执行给定的任务,

			//心跳服务模拟代码
			public class Ping
			{
    			public static void main(String[] args)
    			{
        			ScheduledExecutorService ses =
           			Executors.newScheduledThreadPool(1);
        			Runnable pinger = new Runnable() {
            		public void run() {
                	System.out.println("PING!");
            		}
        		};
        		ses.scheduleAtFixedRate(pinger, 5, 5, TimeUnit.SECONDS);
    		}

* Timeout 方法
	* 为阻塞操作设置一个具体的超时值（以避免死锁）的能力是 `java.util.concurrent` 库相比起早期并发特性的一大进步

[关于 java.util.concurrent 您不知道的5件事Part2](http://www.ibm.com/developerworks/cn/java/j-5things5.html)

[关于 java.util.concurrent 您不知道的5件事Part1](https://www.ibm.com/developerworks/cn/java/j-5things4.html)

[javaDoc On CountDownLatch](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/CountDownLatch.html#countDown())