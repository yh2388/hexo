---
title: Lock与读写锁
date: 2017-07-11 20:40:50
tags: 多线程
---

Lock比传统线程模型中的synchronized方式更加面向对象，与生活中的锁类似，锁本身也应该是一个对象。
两个线程执行的代码片段要实现同步互斥的效果，他们必须用同一个Locak对象。
锁是上在代表要操作的资源的类的内部方法中，而不是线程代码中。下面我们写一段代码来展示：

<!-- more -->

``` java
class Outputer{
    Lock lock = new ReentrantLock();
	public void output(String name){
		int len = name.length();
		lock.lock();
		try{
			for(int i=0;i<len;i++){
				System.out.print(name.charAt(i));
			}
			System.out.println();
		}finally{
			lock.unlock();
		}
	}
}
public class LockTest {
    public static void main(String[] args) {
		final Outputer outputer = new Outputer();
		new Thread(new Runnable(){
			@Override
			public void run() {
				while(true){
					outputer.output("zhangsan");
				}
				
			}
		}).start();
		new Thread(new Runnable(){
			@Override
			public void run() {
				while(true){
					outputer.output("wangwu");
				}
				
			}
		}).start();
	}
}
```
这段代码起到的效果与传统线程技术中的`synchronized`是一样的。

接下来，再说一下读写锁。它分为读锁和写锁，多个读锁不互斥，多个写锁互斥，这就跟我们进行数据库操作的时候很类似了，我们执行`update`等修改操作时，会有锁表的情况。
读写锁是由jvm控制的，我们只要上好锁就可以了。接下来我们就通过一个小例子来说明：
``` java
public class CacheDemo {
    //表示需要缓存的数据
	private Map<String, Object> cache = new HashMap<String, Object>();
	private ReadWriteLock rwl = new ReentrantReadWriteLock();
	public  Object getData(String key){
		rwl.readLock().lock();
		Object value = null;
		try{
			value = cache.get(key);
			if(value == null){
				rwl.readLock().unlock();
				rwl.writeLock().lock();
				try{
					if(value==null){
						value = "aaaa";//实际是queryDB();
					}
				}finally{
					rwl.writeLock().unlock();
				}
				rwl.readLock().lock();
			}
		}finally{
			rwl.readLock().unlock();
		}
		return value;
	}
}
```
上面是一个简单的缓存伪代码，通过这段代码我们可以清楚的知道读写锁的作用。保证了多人写的时候的数据安全与效率。
