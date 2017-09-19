---
title: Spark基础
date: 2017-09-18 20:40:50
tags: Spark
---
#### Spark概述   
Apache Spark是一个快速且通用的大规模数据处理的引擎。使用Scala编写，Scala是一种函数式的编程语言。
它提供了一种交互式的命令行，用于学习和数据探索，分为Python和Scala两种。可以通过Python、Scala或Java
来编写Spark应用程序，来进行大规模的数据处理。
##### Spark Shell   
Spark Shell 提供了交互式的，通过读取/评估/打印三者循环的处理模式，来进行数据的探索，就像使用命令行
一样方便
##### Spark Context
每一个Spark应用程序都必须要有一个SparkContext，它是SparkAPI的主要入口点。

在启动Spark Shell后，我们会在终端中看到一段这样的描述：`Spark context available as sc`。
这是Spark Shell提供的一个预配置的SparkContext，称为`sc`。

<!-- more -->

#### RDD (Resilient Distributed Dataset)
弹性分布式数据集：

    1. 弹性的：如果存在内存中的数据丢失了，可以被重新创建
    2. 分布式：跨集群处理
    3. 数据集：初始数据可来自于一个文件，也可以通过编程方式创建

RDD是Spark中最基本的数据单元

##### Creating an RDD
我们可以通过三种方式来创建一个RDD：

    1. 从一个文件或一个文件集
    2. 从内存中的数据
    3. 从另一个RDD
    
基于文件创建一个RDD：
``` java
val mydata=sc.textFile("purplecow.txt")
mydata.count()
```

##### RDD Operations
RDD的操作有两种，一种是`Action`，它会返回值，另一个是`Transformations`，它会基于当前的RDD来定义一个
新的RDD。
###### RDD Operations:Actions
常见的Action有`count()`、`take(n)`、`collect()`和`saveAsTextFile()`,使用它们来实现对RDD的一些基本操作，
例如统计RDD中元素的个数，获取一定数量的元素，获取全部以及将RDD保存为textFile格式的文件。
###### RDD Operations:Transformations
Transformations会从现有的一个RDD中创建出一个新的RDD，我们要知道的是数据在RDD中是不会被改变的，
可根据需要修改数据并按顺序来转换成新的RDD。

常见的Transformations有：

`map(function)`：通过在一个RDD的每条记录上执行一个函数来创建一个新的RDD。

`fileter(function)`：根据一个布尔函数，筛选或过滤原RDD中的每一条记录，从而创建一个
``` java
mydate.map(line=>line.toUpperCase).filter(line=>line.startsWith("I"))
```

#### Lazy Execution
懒执行的概念与Hibernate框架的懒加载相似。在Hibernate中，session的load()方法具有懒加载的特性，其作用是
降低程序与数据库之间的交互频率，通过load()查询某一条数据的时候并不会直接将这条数据以指定对象的
形式来返回，而是在你真正需要使用该对象里面的一些属性的时候才会去数据库访问并得到数据，因此这样的方式可
以提高程序的运行效率。

所有的Transformations操作都使用了lazy，它们不会计算结果，只是记录dataset的转换操作。因此，在执行Action
操作前数据在RDD中是不会被处理的。

Transformations操作也可以连在一起，作为一个链式的操作进行执行。

参考：http://www.jianshu.com/p/b60dfb856312

###### RDD Lineage and toDebugString
Spark会维护每个RDD的`lineage`关系，即它所依赖的以前的RDD与其的关系。
执行下面的代码：
``` scala
val mydata_filt =sc.textFile("purplecow.txt").map(line => line.toUpperCase()).filter(line => line.startsWith("I"))
mydata_filt.toDebugString
```
我们可以看到输出结果为：
``` scala
(1) MapPartitionsRDD[5] at filter at <console>:27 []
 |  MapPartitionsRDD[4] at map at <console>:27 []
 |  purplecow.txt MapPartitionsRDD[3] at textFile at <console>:27 []
 |  purplecow.txt HadoopRDD[2] at textFile at <console>:27 []
```
通过这些输出的结果，我们可以清晰的看出当前RDD与之前的RDD的各种依赖关系。

###### Pipelining
在可能的情况下，Spark会顺序执行Transformations操作，因此不会存储数据。

在后续的练习中出现了数据被存储下来的情况，RDD 的 Transformation 函数中,又分为窄依赖(narrow dependency)
和宽依赖(wide dependency)的操作。窄依赖跟宽依赖的区别是是否发生 shuffle(洗牌) 操作。宽依赖会发生 shuffle 操作。
窄依赖是子 RDD的各个分片(partition)不依赖于其他分片,能够独立计算得到结果,宽依赖指子 RDD 的各个分片会依赖于父RDD 
的多个分片,所以会造成父 RDD 的各个分片在集群中重新分片。窄依赖中，RDD中的各个数据元素之间不存在依赖，可以在集群
的各个内存中独立计算，也就是并行。而进行宽依赖后的操作，例如groupBy,为了计算相同 key 下的元素个数,需要把相同 key 
的元素聚集到同一个 partition 下,所以造成了数据在内存中的重新分布,即 shuffle 操作.shuffle 操作是 spark 中最耗时的
操作,应尽量避免不必要的 shuffle。

宽依赖主要有两个过程: shuffle write 和 shuffle fetch. 类似 Hadoop 的 Map 和 Reduce 阶段.shuffle write 将 ShuffleMapTask 
任务产生的中间结果缓存到内存中, shuffle fetch 获得 ShuffleMapTask 缓存的中间结果进行 ShuffleReduceTask 计算,
这个过程容易造成OutOfMemory. 

shuffle 过程内存分配使用 ShuffleMemoryManager 类管理,会针对每个 Task 分配内存,Task 任务完成后通过 Executor 释放空间.
早期的内存分配机制使用公平分配,即不同 Task 分配的内存是一样的,但是这样容易造成内存需求过多的 Task 的 OutOfMemory, 
从而造成多余的 磁盘 IO 过程,影响整体的效率这里可以把 Task 理解成不同 key 的数据对应一个 Task.(例:某一个 key 下的数据
明显偏多,但因为大家内存都一样,这一个 key 的数据就容易 OutOfMemory).1.5版以后 Task 共用一个内存池,内存池的大小默认为 JVM 
最大运行时内存容量的16%,分配机制如下::假如有 N 个 Task,ShuffleMemoryManager 保证每个 Task 溢出之前至少可以申请到1/2N 内存,
且至多申请到1/N,N 为当前活动的 shuffle Task 数,因为N 是一直变化的,所以 manager 会一直追踪 Task 数的变化,重新计算队列中
的1/N 和1/2N.但是这样仍然容易造成内存需要多的 Task 任务溢出,所以最近有很多相关的研究是针对 shuffle 过程内存优化的. 

上面描述了shuffle过程内存分配的问题，但是，当shuffle的结果量非常大，而内存不够时，要不就失败，要不就用老办法把内存中的数据
移到磁盘上放着。Spark意识到在处理数据规模远远大于内存空间时所带来的不足，引入了一个具有外部排序的方案。Shuffle过来的数据先
放在内存中，当内存中存储的<key, value>对超过1000并且内存使用超过70%时，判断节点上可用内存如果还足够，则把内存缓冲区大小翻倍，
如果可用内存不再够了，则把内存中的<key, value>对排序然后写到磁盘文件中。最后把内存缓冲区中的数据排序之后和那些磁盘文件组成一
个最小堆，每次从最小堆中读取最小的数据，这个和MapReduce中的merge过程类似。

参考：
	1. http://blog.csdn.net/pzw_0612/article/details/53150004
	2. http://blog.csdn.net/databatman/article/details/53023818
	3. http://www.cnblogs.com/jxhd1/p/6528540.html
#### Functional Programming in Spark
Spark很大程度上依赖于函数式编程的概念：函数是程序的最基本单位，仅有输入和输出，没有状态和其他的影响。

关键概念：将函数作为另一个函数的输入；匿名函数。

###### Passing Functions as Parameters
许多RDD的操作使用函数作为参数，下面有一段伪代码，表示将函数fn应用到RDD中的每个记录：
``` scala
RDD {
    map(fn(x)) {
        foreach record in rdd
        emit fn(record)
    }
}
```
下面有一个通过定义一个函数，将该函数作为参数传入map中的例子：
``` scala
def toUpper(s: String): String ={ s.toUpperCase }
val mydata = sc.textFile("purplecow.txt")
mydata.map(toUpper).take(2)
```
###### Anonymous Functions
匿名函数是没有标识符的内联函数，那些代码量少或只执行一次的函数，我们就可以将其写为匿名函数。
很多语言支持匿名函数，例如Python、Scala、Java8。下面是一个Scala匿名函数操作的例子：
``` scala
mydata.map(line => line.toUpperCase()).take(2)
//Scala允许在匿名参数使用下划线(_)代替
//mydata.map(_.toUpperCase()).take(2)
```
在java中，由于java8提供了新的语法特性，因此在编写匿名函数时，相较于之前的版本会更加简洁。

``` java
//java7的写法
JavaRDD<String> mydata = sc.textFile("file");
JavaRDD<String> mydata_uc = mydata.map(new Function<String,String>() {
    @Override
    public String call(String s) {
        return (s.toUpperCase());
    }
});
//java8的写法
JavaRDD<String> lines = sc.textFile("file");
JavaRDD<String> lines_uc = lines.map(line -> line.toUpperCase());
```

#### Essential Points
对以上内容做一个总结：

1. Spark可以通过Spark Shell交互式地使用，支持Python或Scala语言。
2. RDDs(弹性分布式数据集)是Spark中的一个关键概念。
3. RDD操作
    * Transformations操作基于现有的RDD来创建一个新的RDD。
    * Action操作会返回RDD的一个值。
4. 懒执行。
5. Spark使用了函数使编程。






