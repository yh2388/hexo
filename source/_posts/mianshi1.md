---
title: 面试中遇到的不足（一）
date: 2017-08-18 10:40:50
tags: 面试中的坑
---

面试时遇到的一些问题，有之前没接触过的，也有自己基础不够扎实的所导致的问题，在这里做一个总结，查漏补缺，明确自己的不足。
##### 问题一
问：springMVC中的@RequestMapping中的value能否相同？

答：答案是可以的，经过网上资料的查阅，发现value值是可以相同的。那么我们需要考虑的是，当有相同的URL时，springMVC如何进行映射。对于这个问题，在我知道答案后，我想到了数据库联合主键。
的确，http协议请求时，除了URL，还有一个请求方法如`GET`、`POST`、`PUT`、`DELETE`等，通过URL与请求方法组合的形式，就能够达到和数据库联合主键一样的效果。因此，我们就可以明白，当不同的@RequestMapping中的value相同时，method不同时，springMVC同样可以将我们的请求映射到对应的controller的方法。

既然被问到了这方面相关的问题，那就好好总结一下吧。下面就总结一下几个我暂时还不怎么用到的方法。

<!-- more -->

###### 1.数据绑定
``` java
@RequestMapping(value="/departments")
public String findDepatment(
    @RequestParam("departmentId") String departmentId){
    System.out.println("Find department with ID:"+departmentId);
    return "someResult";
}
```
形如这样的访问形式：`/departments?departmentId=23`就可以触发访问findDepatment方法了。

###### 2.RESTful风格
RESTful风格有两种形式，第一种如下：
``` java
@RequestMapping(value="/departments/{departmentId}")
public String findDepatment(
    @PathVariable String departmentId){
    System.out.println("Find department with ID:"+departmentId);
    return "someResult";
}
```
形如RESTful风格的地址访问，比如：`/departments/23`，其中用@PathVariable接收RESTful风格的参数。

第二种形式如下：
``` java
@RequestMapping(value="/departments/{departmentId}")
public String findDepatment(
    @PathVariable("departmentId") String someDepartmentId){
    System.out.println("Find department with ID:"+someDepartmentId);
    return "someResult";
}
```
这个有点不同，就是接收形如`/departments/23`的URL访问，把23作为传入的departmetnId,，但是在实际的方法findDepatmentAlternative中，使用 
`@PathVariable("departmentId") String someDepartmentId`，将其绑定为 
someDepartmentId,所以这里someDepartmentId为23。这种方法也看起来更加严谨一些。

我们再拓展一下，令一个url中绑定多个数据：
``` java
@RequestMapping(value="/departments/{departmentId}/employees/{employeeId}")
public String findDepatment(
    @PathVariable("departmentId") String departmentId
    @PathVariable("employeeId") String employeeId
    ){
    System.out.println("Find employee with ID:"+employeeId+
                        "from department:"+departmentId);
    return "someResult";
}
```
###### 3.正则表达式匹配
``` java
@RequestMapping(value="/{textualPart:[a-z-]+}.{numericPart:[\\d]+}")
public String regularExpression(
    @PathVariable String textualPart,
    @PathVariable String numericPart){
    System.out.println("Textual part: " + textualPart + 
      ", numeric part: " + numericPart);
    return "someResult";
}
```
比如如下的URL：`/sometext.123`，则输出： Textual part: sometext, numeric part: 123.

##### 问题二
问：Thread的start()与run()相关，如果只运行run()是否可以？

答：只作为一个普通的类的方法进行执行，而没有创建一个新的线程，调用start()来创建一个新的线程，并执行run()。

##### 问题三
问：包装类的作用。之前表达的不够全面，在这里再总结一下。

答：

1. 实现基本类型之间的转换。
2. 在实际使用时，有些地方需要传一个Object的对象，显然传入基本类型是不行的。这时就可以传入一个包装类进去。
还有一点要提的是，实际上我们在平常使时，往往传入基本类型也没有问题，比如equals(Object obj),我们可以看到
他需要传入的参数是Object，但`"1".equals("1")`也可以正常运行，这里就涉及到了，基本类型的自动装箱了，
这就是设计包装类所带来的好处。
3. 将基本类型转换成包装类，使其变成一个对象，那么，我们就可以赋予包装类很多的方法去操作了。原本以为这些包装类还可以被继承，但是
通过自己实际测试并查看了源码后发现，这些类都是`final`所修饰的类，因此不能被继承。
``` java
public final class Double extends Number implements Comparable<Double> {
	//Double类
}
```

对于面试官所说的BigDecimal的精度丢失问题，可以参考这骗文章：http://www.cnblogs.com/chenssy/archive/2012/09/09/2677279.html
，这里就不再赘述了。

以上，就是我对我面试到的回答不好的部分问题的总结。如果有不对的地方，欢迎批评指正！QQ：178317391

