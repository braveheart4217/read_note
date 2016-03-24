##Synchronized

1. 关于线程的理解
	* 线程的阻塞与挂起
		* 理解一:
			* 挂起是一种主动行为，因此恢复也应该要主动完成，而阻塞则是一种被动行为，是在等待事件或资源时任务的表现，你不知道他什么时候被阻塞(pend)，也就不能确切 的知道他什么时候恢复阻塞。而且挂起队列在操作系统里可以看成一个，而阻塞队列则是不同的事件或资源（如信号量）就有自己的队列
		* 理解二:
			* 阻塞（pend）就是任务释放CPU，其他任务可以运行，一般在等待某种资源或信号量的时候出现。挂起（suspend）不释放CPU，如果任务优先级高就永远轮不到其他任务运行，一般挂起用于程序调试中的条件中断，当出现某个条件的情况下挂起，然后进行单步调试
		* 理解三:
			* pend是task主动去等一个事件或消息.suspend是直接悬挂task,以后这个task和你没任何关系,任何task间的通信或者同步都和这个suspended task没任何关系了,除非你resume task
		* 理解四:
			* 任务调度是操作系统来实现的，任务调度时，直接忽略挂起状态的任务，但是会顾及处于pend下的任务，当pend下的任务等待的资源就绪后，就可以转为ready了。ready只需要等待CPU时间，当然，任务调度也占用开销，但是不大，可以忽略。可以这样理解，只要是挂起状态，操作系统就不在管理这个任务了
		* 理解五:
			* 挂起是主动的，一般需要用挂起函数进行操作，若没有resume的动作，则此任务一直不会ready。而阻塞是因为资源被其他任务抢占而处于休眠态。两者的表现方式都是从就绪态里“清掉”，即对应标志位清零，只不过实现方式不一样

2. Synchronized是java关键字，一种同步锁，他可以修饰：
	1.  修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象
	2.  修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象
	3.  修饰一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象
	4.  修饰一个类，其作用的范围是synchronized后面括号括起来的部分，作用主的对象是这个类的所有对象
	5.  note：
		*  当一个线程访问对象的一个synchronized(this)同步代码块时，另一个线程仍然可以访问该对象中的非synchronized(this)同步代码块
		 
				//obj 锁定的对象
   				synchronized(obj)
  				{
      				// todo
   				}
		* synchronized关键字不能继承
			* 在子类中复写父类中的synchronize方法时，子类中的方法并不是同步的
		* 在定义接口方法时不能使用synchronized关键字
		* 构造方法不能使用synchronized关键字，但可以使用synchronized代码块来进行同步
		* 当synchronize修饰一个static方法时，其作用对象是该类的所有对象
		* synchronize修饰一个类，作用对象也是该类的所有对象
			
				class ClassName {
   					public void method() {
      					synchronized(ClassName.class) {
         				// todo
      					}
   					}
				}


3. Synchronized 原理
	* JVM规范规定JVM基于进入和退出 `Monitor` 对象来实现方法同步和代码块同步
	* 代码块同步是使用 `monitorenter` 和 `monitorexit` 指令实现，而方法同步是使用另外一种方式实现，细节在JVM规范里并没有详细说明，但是方法的同步同样可以使用这两个指令来实现。`monitorenter `指令是在编译后插入到同步代码块的开始位置，而`monitorexit` 是插入到方法结束处和异常处， JVM要保证每个 `monitorenter` 必须有对应的 `monitorexit` 与之配对。任何对象都有一个 `monitor` 与之关联，当且一个 `monitor` 被持有后，它将处于锁定状态。线程执行到 `monitorenter` 指令时，将会尝试获取对象所对应的 `monitor` 的所有权，即尝试获得对象的锁
	* 锁存在Java对象头里
		* 如果对象是数组类型，则虚拟机用3个Word（字宽）存储对象头，如果对象是非数组类型，则用2字宽存储对象头。在32位虚拟机中，一字宽等于四字节，即32bit

			|长度|内容|说明|
			|----|-----:|----|
			| 32/64 bit | Mark Word |存储对象的hashCode或锁的信息
		| 32/64 bit| Class Metadata | 存贮到对象类型数据的指针
		| 32/64 bit | Array Length | 数组的长度(如果当前对象是数组)
		
		* Java对象头里的 Mark Word 里默认存贮对象的hashCode，分年代标记和锁标记位，默认存贮如下
		
		 	||25bit|4vit|1bit(是否是偏向锁)|2bit(锁标识位)|
			|---|---|--|--:|---:|
			|无锁状态 | 对象的hashCode | 对象分代年龄|0|01

	
		* 运行期间 Mark Word存储的数据随着锁的标志位的变化而变化
			
			![运行时数据格式](http://i.imgur.com/TD5sQv4.png)
		* 锁的升级
			* Java1.6中的锁一共有四种状态，无锁状态，偏向锁状态，轻量级锁状态和重量级锁状态，它会随着竞争情况逐渐升级
			* 锁可以升级但不能降级，意味着偏向锁升级成轻量级锁后不能降级成偏向锁。这种锁升级却不能降级的策略，目的是为了提高获得锁和释放锁的效率
				* 偏向锁
					* 当一个线程访问同步块并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程ID，以后该线程在进入和退出同步块时不需要花费CAS操作来加锁和解锁，而只需简单的测试一下对象头的Mark Word里是否存储着指向当前线程的偏向锁，如果测试成功，表示线程已经获得了锁，如果测试失败，则需要再测试下Mark Word中偏向锁的标识是否设置成1（表示当前是偏向锁），如果没有设置，则使用CAS竞争锁，如果设置了，则尝试使用CAS将对象头的偏向锁指向当前线程
					* 偏向锁的撤销
						* 偏向锁使用了一种等到竞争出现才释放锁的机制，所以当其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁
						* 偏向锁的撤销，需要等待全局安全点（在这个时间点上没有字节码正在执行），它会首先暂停拥有偏向锁的线程，然后检查持有偏向锁的线程是否活着，如果线程不处于活动状态，则将对象头设置成无锁状态，如果线程仍然活着，拥有偏向锁的栈会被执行，遍历偏向对象的锁记录，栈中的锁记录和对象头的Mark Word要么重新偏向于其他线程，要么恢复到无锁或者标记对象不适合作为偏向锁，最后唤醒暂停的线程
						* [偏向锁的撤销流程](http://i.imgur.com/EUg4NM9.png)
				* 轻量级锁
					* 轻量级锁的加锁
						* 线程在执行同步块之前，JVM会先在当前线程的栈桢中创建用于存储锁记录的空间，并将对象头中的Mark Word复制到锁记录(Displaced Mark Word)中。然后线程尝试使用CAS将对象头中的Mark Word替换为指向锁记录的指针。如果成功，当前线程获得锁，如果失败，表示其他线程竞争锁，当前线程便尝试使用自旋来获取锁
					* 轻量锁的解锁
						* 轻量级解锁时，会使用原子的CAS操作来将Displaced Mark Word替换回到对象头，如果成功，则表示没有竞争发生。如果失败，表示当前锁存在竞争，锁就会膨胀成重量级锁。
						* 因为自旋会消耗CPU，为了避免无用的自旋（比如获得锁的线程被阻塞住了），一旦锁升级成重量级锁，就不会再恢复到轻量级锁状态。当锁处于这个状态下，其他线程试图获取锁时，都会被阻塞住，当持有锁的线程释放锁之后会唤醒这些线程，被唤醒的线程就会进行新一轮的夺锁之争
						*  [轻量级锁膨胀流程](http://ifeve.com/wp-content/uploads/2012/10/%E8%BD%BB%E9%87%8F%E7%BA%A7%E9%94%81.png)、
##Java wait/notify


##并发数据结构

1. volatile
	1. 就是使用乐观锁加锁的变量
2. 共享预期的修改
	* 用来替换拷贝和修改整个数据结构，一个线程可以共享它们对共享数据结构预期的修改。一个线程向对修改某个数据结构的过程变成了下面这样(实际上就像加锁了)
		* 检查是否另一个线程已经提交了对这个数据结构提交了修改
		* 如果没有其他线程提交了一个预期的修改，创建一个预期的修改，然后向这个数据结构提交预期的修改
		* 执行对共享数据结构的修改
		* 移除对这个预期的修改的引用，向其它线程发送信号，告诉它们这个预期的修改已经被执行
	* 为了避免一个已经提交的预期修改可以锁住共享数据结构，一个已经提交的预期修改必须包含足够的信息让其他线程来完成这次修改。因此，如果一个提交了预期修改的线程从未完成这次修改，其他线程可以在它的支持下完成这次修改，保证这个共享数据结构对其他线程可用
	* 流程如下：
		
		![非阻塞算法执行流程](http://i.imgur.com/FWOcEPI.png)

	* 修改必须被当做一个或多个 CAS 操作来执行。因此，如果两个线程尝试去完成同一个预期修改，仅有一个线程可以所有的 CAS 操作。一旦一条 CAS 操作完成后，再次企图完成这个 CAS 操作都不会“得逞”
	* A-B-A 问题需要使用额外的 标记(版本) 来解决
	* 一个非阻塞算法模板
	
				import java.util.concurrent.atomic.AtomicBoolean;
				import java.util.concurrent.atomic.AtomicStampedReference;
		
				public class NonblockingTemplate{
		    		public static class IntendedModification{
		        		public AtomicBoolean completed = new AtomicBoolean(false);
		    		}
		
		    	private AtomicStampedReference<IntendedModification> ongoinMod = new AtomicStampedReference<IntendedModification>(null, 0);
		    	//declare the state of the data structure here.
		
		    	public void modify(){
		        	while(!attemptModifyASR());
		    	}
		
		    	public boolean attemptModifyASR(){
		        	boolean modified = false;
		
		        	IntendedMOdification currentlyOngoingMod = ongoingMod.getReference();
		        	int stamp = ongoingMod.getStamp();
		
		        	if(currentlyOngoingMod == null){
		            	//copy data structure - for use
		            	//in intended modification
		
		            	//prepare intended modification
		            	IntendedModification newMod = new IntendModification();
		
		            	boolean modSubmitted = ongoingMod.compareAndSet(null, newMod, stamp, stamp + 1);
		
		            	if(modSubmitted){
		            	     //complete modification via a series of compare-and-swap operations.
		            	    //note: other threads may assist in completing the compare-and-swap
		            	    // operations, so some CAS may fail
		            	    modified = true;
		            	}
		        	}else{
		             	//attempt to complete ongoing modification, so the data structure is freed up
		            	//to allow access from this thread.
		            	modified = false;
		        	}
		
		        	return modified;
		    	}
			}


参考资料
	
1. [ Synchronize 使用](http://blog.csdn.net/luoweifu/article/details/46613015)
2. [ Wait/Notify 使用](http://blog.csdn.net/luoweifu/article/details/46664809)
3. [Java SE1.6中的Synchronized](http://ifeve.com/java-synchronized/)