---
title: 多线程的同步通信问题
date: 2017-07-06 20:40:50
tags: 多线程
---
在多个线程之间，也会有同步通信的需求，这里，我们通过一个面试题来帮助理解线程的同步通信问题。

该问题是这样的：主线程循环输出100次后，子线程循环输出10次，如此交替往复地进行50次。
通过分析题目可知，这里就需要用到同步通信了，在线程中有几个方法可以实现，即`wait()`、`notify()`、`notifyAll()`。

<!-- more -->

接下来我们就通过实际的代码来解答这个面试题吧。
``` java
public class TraditionalThreadCommunication {
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
        //main()可以作为主线程
		for(int i=1;i<=50;i++){
			business.main(i);
		}
	}
}
   class Business {
	  private boolean bShouldSub = true;
	  public synchronized void sub(int i){
		  while(!bShouldSub){
			  try {
				this.wait();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		  }
			for(int j=1;j<=10;j++){
				System.out.println("sub thread sequence of " + j + ",loop of " + i);
			}
		  bShouldSub = false;
		  this.notify();
	  }
	  public synchronized void main(int i){
		  	while(bShouldSub){
		  		try {
					this.wait();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
		  	}
			for(int j=1;j<=100;j++){
				System.out.println("main thread sequence of " + j + ",loop of " + i);
			}
			bShouldSub = true;
			this.notify();
	}
}
```
由上述代码我们可以看出，通过两个线程所执行的方法交替的进行`wait()`和`notify()`,便达到了我们想要的结果，可以说，这就是一个简单的
线程同步通信了。

另外，再说明一个问题，就是有时候线程会存在伪唤醒的情况，即没有被通知唤醒的情况下，唤醒了。因此，使用`while()`进行判断更为健壮。