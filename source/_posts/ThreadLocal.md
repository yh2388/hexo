---
title: ThreadLocal实现线程范围内的共享变量
date: 2017-07-04 20:40:50
tags: 多线程
---

同样，我们拿银行转账的操作举例。

对于转出账户的余额减少，转入账户余额的增加，这两个操作必须在同一个事物中完成，他们必须使用相同的数据库连接对象，
转入和转出操作的代码分别是两个不同账户对象的方法。

<!-- more -->

接下来，我们进行一个类的设计，来验证同一个线程内变量共享。
首先我们使用`ThreaLocal`来设计一个用于存放共同操作金额的类。
``` java
class MyThreadScopeData{
    private static ThreadLocal<MyThreadScopeData> map = new ThreadLocal<MyThreadScopeData>();
	private int balance;
	private MyThreadScopeData(){}
    //由于测试的本身就是在同一线程下，因此此处不在添加synchronized关键字
	public static MyThreadScopeData getThreadInstance(){
		//此处获取的实例为ThreadLocal中的MyThreadScopeData实例，因此，当线程改变时，这个实例也就变为null了
		MyThreadScopeData instance = map.get();
		if(instance == null){
			instance = new MyThreadScopeData();
			map.set(instance);
		}
		return instance;
	}
	public int getBalance() {
		return balance;
	}
	public void setBalance(int balance) {
		this.balance = balance;
	}
}

```
然后，我们简易的设计一个Accoount类。
``` java
class Account{
    private int balance=10000 ;//账户原金额
	public void in(){
		MyThreadScopeData myData = MyThreadScopeData.getThreadInstance();
		int currentBalance = (balance+myData.getBalance());
		System.out.println("转入账户  " + Thread.currentThread().getName() 
				+ "当前余额: " + currentBalance );
		balance=currentBalance;
	}
	public void out(){
		MyThreadScopeData myData = MyThreadScopeData.getThreadInstance();
		int currentBalance = (balance-myData.getBalance());
		System.out.println("转出账户 " + Thread.currentThread().getName() 
				+"当前余额: " + currentBalance);
		balance=currentBalance;
	}	
}
```
最后编写测试类。
``` java
class ThreadLocalTest {
    private static ThreadLocal<MyThreadScopeData> myThreadScopeData = new ThreadLocal<MyThreadScopeData>();
	public static void main(String[] args) {
		for(int i=0;i<2;i++){
			new Thread(new Runnable(){
				@Override
				public void run() {
					int data = (int) (Math.random()*1000);//要操作的金额
					System.out.println("操作的金额："+data);
					//保证得到的操作金额相同相同
					MyThreadScopeData.getThreadInstance().setBalance(data);
					new Account().in();
					new Account().out();
				}
			}).start();
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}
```
最终的结果如下：
``` java
操作的金额：299
转入账户  Thread-0当前余额: 10299
转出账户 Thread-0当前余额: 9701
操作的金额：937
转入账户  Thread-1当前余额: 10937
转出账户 Thread-1当前余额: 9063
```
我们模拟了两次转入转出操作，使用ThreadLocal后，都可以在当前线程范围内获取到所要的共同数据，实现了
同一线程内的数据共享。显然，这不是通过传递参数实现的。

另外要说明的是，ThreadLocal只能处理一个对象，当我需要获取多个对象或属性时，我们可以将所需的对象或者
属性进一步的封装。
#### 思考：不同的线程与ThreadLocal的关系
经过反复的测试与思考，得出来在MyThreadScopeData类第七行注释的结论。因此，我们也就可以理解，ThreadLocal是相对于
每个线程的。比如，当一个请求是一个线程时，那么我们也可以说ThreadLocal是这个请求的容器。