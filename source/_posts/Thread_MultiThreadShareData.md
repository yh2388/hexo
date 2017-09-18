---
title: 多个线程访问共享对象和数据的方式
date: 2017-07-08 20:40:50
tags: 多线程
---
#### 如果每个线程执行的代码相同
我们可以使用Runnable对象，这个Runnable对象中有那些共享的数据或对象。
例如，卖票系统就可以这么做。
``` java
public class Text {
    public static void main(String[] args) {
		ShareData data = new ShareData();
		for(int i=0;i<100;i++){
			new Thread(data).start();
		}
	}
}
class ShareData implements Runnable{
	private int count = 100;
	@Override
	public void run() {
		count--;
		System.out.println(Thread.currentThread().getName()+"购买了第"+count+"张票");
	}
}
```
<!-- more -->
#### 如果需要执行的代码不相同
比如，我们需要一个线程执行减操作，一个线程执行加操作，我们也可以有多种方案。

首先我们同样写一个存放共享数据的类,再将这个共享数据类交给不同的线程来处理它的不同方法：
``` java
class ShareData{
    private int j = 0;
	public synchronized void increment(){
		j++;
		System.out.println("inc:"+j);
	}
	
	public synchronized void decrement(){
		j--;
		System.out.println("dec:"+j);
	}
}
```
第一种我们可以进行如下的测试：
``` java
final ShareData data = new ShareData();
new Thread(new Runnable(){
	@Override
	public void run() {
		data.decrement();
	}
}).start();
new Thread(new Runnable(){
	@Override
	public void run() {
		data.increment();
	}
}).start();
```
再一种我们可以将两个不同的方法放入两个Runnable中,再进行处理:
``` java
class MyRunnable1 implements Runnable{
    private ShareData data;
	public MyRunnable1(ShareData1 data1){
		this.data = data1;
	}
	public void run() {
		data1.decrement();
	}
}

class MyRunnable2 implements Runnable{
	private ShareData data;
	public MyRunnable2(ShareData1 data1){
		this.data = data1;
	}
	public void run() {
		data1.increment();
	}
}
//测试代码
ShareData data = new ShareData();
new Thread(new MyRunnable1(data)).start();
new Thread(new MyRunnable2(data)).start();
```
以上，就是对多个线程访问共享对象和数据的方式的讨论。