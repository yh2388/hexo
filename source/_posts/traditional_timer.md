---
title: 传统多线程技术之定时器
date: 2017-06-04 20:40:50
tags: 多线程
---
定时器是个非常有趣的东西，因为它的叫法，所以我想到了做一个定时炸弹的模拟

首先。最基本的一个定时炸弹，我们设定时间为十秒钟以后爆炸，代码如下：
``` java
new Timer().schedule(new TimerTask() {
	@Override
	public void run() {
		System.out.println("bombing!");
	}
}, 10000);
```
当然，我们还可以牛逼一点的搞一个连环炸，只要在TimerTask中再加一个参数即可。
``` java
//首次爆炸结束后，每隔一秒钟炸一次
new Timer().schedule(new TimerTask() {
    @Override
	public void run() {
		System.out.println("bombing!");
	}
}, 10000,1000);
```
这样，我们的一个简易的定时炸弹就做好了。

但是，你是否会觉得这样太过于简单呢，所以，我们还可以这样玩。
``` java
public class TraditionalTimerTest {
    private static int count = 0;
	public static void main(String[] args) {
		class MyTimerTask extends TimerTask{
			@Override
			public void run() {
				count = (count+1)%2;
				System.out.println("bombing!");
				new Timer().schedule(new MyTimerTask(),2000+2000*count);
			}
		}
		new Timer().schedule(new MyTimerTask(), 2000);
		while(true){
			System.out.println(new Date().getSeconds());
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
}
```
上面所达到的效果就是炸弹在两秒和四秒的间隔交替爆炸。怎么样，很有趣吧。在实际工作中的话，这种定时的作用应该能够有所应用，所以我就把他记录下来了。
