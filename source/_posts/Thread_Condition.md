---
title: Condition同步通信
date: 2017-07-11 20:40:50
tags: 多线程
---
Condition的功能类似于传统线程技术中的`Object.wait()`和`Object.notify()`。所以通过对比之前的
同步通信的代码，来了解`Condition`。

<!-- more -->

``` java
public class ConditionCommunication {
    public static void main(String[] args) {
		
		final Business business = new Business();
		new Thread(
				new Runnable() {
					@Override
					public void run() {
						for(int i=1;i<=50;i++){
							business.sub(i);
						}
					}
				}
		).start();
		for(int i=1;i<=50;i++){
			business.main(i);
		}
	}
	static class Business {
		Lock lock = new ReentrantLock();
		Condition condition = lock.newCondition();
		  private boolean bShouldSub = true;
		  public  void sub(int i){
			  lock.lock();
			  try{
				  while(!bShouldSub){
					  try {
						condition.await();
					} catch (Exception e) {
						e.printStackTrace();
					}
				  }
					for(int j=1;j<=10;j++){
						System.out.println("sub thread sequence of " + j + ",loop of " + i);
					}
				  bShouldSub = false;
				  condition.signal();
			  }finally{
				  lock.unlock();
			  }
		  }
		  public void main(int i){
			  lock.lock();
			  try{
				 while(bShouldSub){
				  		try {
							condition.await();
						} catch (Exception e) {
							e.printStackTrace();
						}
				  	}
					for(int j=1;j<=100;j++){
						System.out.println("main thread sequence of " + j + ",loop of " + i);
					}
					bShouldSub = true;
					condition.signal();
		  }finally{
			  lock.unlock();
		  }
	    }
	}
}
```
通过上面的代码我们可以看出，我们只是简单的对原先的`wait()`和`notify()`进行了替换，那么使用Condition又有什么好处呢？
于是，我们查看一下API，发现了API中有一段这样的代码：
``` java
class BoundedBuffer {
   final Lock lock = new ReentrantLock();
   final Condition notFull  = lock.newCondition(); 
   final Condition notEmpty = lock.newCondition(); 

   final Object[] items = new Object[100];
   int putptr, takeptr, count;
   public void put(Object x) throws InterruptedException {
     lock.lock();
     try {
       while (count == items.length) 
         notFull.await();
       items[putptr] = x; 
       if (++putptr == items.length) putptr = 0;
       ++count;
       notEmpty.signal();
     } finally {
       lock.unlock();
     }
   }
   public Object take() throws InterruptedException {
     lock.lock();
     try {
       while (count == 0) 
         notEmpty.await();
       Object x = items[takeptr]; 
       if (++takeptr == items.length) takeptr = 0;
       --count;
       notFull.signal();
       return x;
     } finally {
       lock.unlock();
     }
   } 
 }

```
这是一个缓冲区的示例，由这段代码我们可以看出，
如果试图在空的缓冲区上执行 take 操作，则在某一个项变得可用之前，线程将一直阻塞；
如果试图在满的缓冲区上执行 put 操作，则在有空间变得可用之前，线程将一直阻塞。

Condition 实现可以提供不同于 Object 监视器方法的行为和语义，比如受保证的通知排序，或者在执行通知时不需要保持一个锁。
如果某个实现提供了这样特殊的语义，则该实现必须记录这些语义。

注意，Condition 实例只是一些普通的对象，它们自身可以用作 synchronized 语句中的目标，
并且可以调用自己的 wait 和 notification 监视器方法。获取 Condition 实例的监视器锁或者使用其监视器方法，与获取和该 Condition 相关的 Lock 或使用其 waiting 和 signalling 方法没有什么特定的关系。
为了避免混淆，建议除了在其自身的实现中之外，切勿以这种方式使用 Condition 实例。