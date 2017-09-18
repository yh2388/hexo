---
title: 多线程的互斥问题
date: 2017-07-04 20:40:50
tags: 多线程
---
通常，我们在理解多线程互斥问题的时候会用银行转账这个例子来说明，道理也很简单，即当你在进行取钱操作的
时候，有人在这个时候向你汇款，那么就会出现账面余额与实际情况不符的问题。虽然这是小概率发生的事情，但是，这也是绝对不允许的事情。
现在，我们就通过几个小例子，来逐步说明线程互斥的问题。

<!-- more -->

#### 在未进行互斥处理时
当我们有两个或多个线程同时进行相同的操作时,我们可以写一个output方法，作为相同的操作
``` java
public void output(String name){
	int len = name.length();
	for(int i=0;i<len;i++){
		System.out.print(name.charAt(i));
	}
	System.out.println();
}
```
当两个线程同时运行时，就会出现问题了。
``` java
new Thread(new Runnable(){
	@Override
	public void run() {
		while(true){
			outputer.output("asdfghjkl");
		}
	}
}).start();
new Thread(new Runnable(){
	@Override
	public void run() {
		while(true){
			outputer.output("qwertyuiop");
		}
		
	}
}).start();
```
运行，我们可以很容易就找到控制台中这样的异常输出
``` java
aqwertysdfghjkl
uiop
qweasrtydfuighjklop

asdqwertfgyuiop
hjkl
asqwertyuiop
dfghjkl
```
很显然，这就不是我们所希望得到的结果。那么接下来我们开始讨论如何解决这样的问题发生。
#### 解决线程不互斥所产生的问题
我们对原output方法进行改写
``` java
public synchronized void output2(String name){
	int len = name.length();
	for(int i=0;i<len;i++){
			System.out.print(name.charAt(i));
	}
	System.out.println();
}

public static synchronized void output3(String name){
	int len = name.length();
	for(int i=0;i<len;i++){
			System.out.print(name.charAt(i));
	}
	System.out.println();
}
public void output4(String name){
	int len = name.length();
	synchronized (Outputer.class) 
	{
		for(int i=0;i<len;i++){
			System.out.print(name.charAt(i));
		}
		System.out.println();
	}
}
```
由代码我们可以看出，我们在方法的不同位置加上了`synchornized`关键字，但起到的作用是相同的，这样可以保证
线程运行的原子性。演示效果就不在列出了。这里特别注意的是`synchornized`需要控制的同一个对象才可以。
