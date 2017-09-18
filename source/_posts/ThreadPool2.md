---
title: 线程池技术（二）
date: 2017-07-10 20:40:50
tags: 多线程
---
`Callable`与`Future`的功能与有点类似于回调函数，我们写一段代码来记录一下。
``` java
ExecutorService threadPool =  Executors.newSingleThreadExecutor();
Future<String> future =
	threadPool.submit(
		new Callable<String>() {
			public String call() throws Exception {
				Thread.sleep(2000);
				return "hello";
			};
		}
);
System.out.println("等待结果");
try {
	System.out.println("拿到结果" + future.get());
}catch (Exception e) {
	e.printStackTrace();
}
```
我们实现了一个功能，即通过`future.get()`来获取到`Callable`执行完毕后的返回结果。

另外，我们还有一种方法，简而言之就是获取到完成后的结果，谁先完成就获取谁。示例代码如下：
``` java
ExecutorService threadPool =  Executors.newFixedThreadPool(10);
CompletionService<Integer> completionService = new ExecutorCompletionService<Integer>(threadPool);
for(int i=1;i<=10;i++){
	 final int seq = i;
	completionService.submit(new Callable<Integer>() {
		@Override
		public Integer call() throws Exception {
			Thread.sleep(new Random().nextInt(5000));
			return seq;
		}
	});
}
for(int i=0;i<10;i++){
	try {
		System.out.println(completionService.take().get());
	}catch (Exception e) {
		e.printStackTrace();
	}
}
```
使用这两种技术时，`threadPool`都不再调用`execute()`,而是调用`submit()`。
执行后的结果就不再赘述了。