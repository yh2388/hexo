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
![图片]()

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
懒执行的概念与Hibernate框架的烂加载相似。在Hibernate中，session的load()方法具有懒加载的特性，其作用是
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
在后面的练习中发现，处理RDD时，当执行join、reduceByKey等操作后，数据会被存储下来，再次执行Action操作时，
就会有Skipped的情况出现。
** 在测试时，发现似乎与var/val声明值无关，单纯的具有依赖关系的RDD执行Action操作时，数据不会被保存。
Transformations操作之记录操作的记录，而不保存数据。


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
4. 懒执行：
5. Spark使用了函数使编程。






