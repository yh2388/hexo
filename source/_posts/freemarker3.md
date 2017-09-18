---
title: FreeMarker模板开发语句
date: 2017-08-17 20:40:50
tags: FreeMarker
---

最简单的模板是普通  HTML  文件（或者是其他任何文本文件—FreeMarker  本身不属于HTML）。当客户端访问页面时，FreeMarker 要发送 HTML 代码至客户端浏览器端显示。如果想要页面动起来，就要在 HTML 中放置能被FreeMarker所解析的特殊部分。 
    
${…}：FreeMarker 将会输出真实的值来替换花括号内的表达式，这样的表达式被称为interpolations 插值，可以参考第之前示例的内容。

FTL tags 标签（FreeMarker  模板的语言标签）：FTL 标签和 HTML 标签有一点相似，但是它们是FreeMarker的指令而且是不会直接输出出来的东西。这些标签的使用一般以符号`#`开头。（用户自定义的 FTL 标签使用`@`符号来代替`#`，但这是更高级的主题内容了，后面会详细地讨论） 
   
Comments 注释：FreeMarker 的注释和 HTML 的注释相似，但是它用`<#--`和`-->`来分隔的。任何介于这两个分隔符（包含分隔符本身）之间内容会被FreeMarker忽略，就不会输出出来了。 

其他任何不是  FTL  标签，插值或注释的内容将被视为静态文本，这些东西就不会被FreeMarker 所解析，会被按照原样输出出来。 

directives指令：就是所指的FTL标签。这些指令在HTML的标签（如`<table>`和`</table>`）和HTML元素（如table元素）中的关系是相同的。（如果现在你还不能区分它们，那么把“FTL标签”和“指令”看做是同义词即可。）

<!-- more -->

##### if指令

我们接着之前的第一个程序，在Map中put一个随机数：`root.put("random", new Random().nextInt(100));`
然后，开始进行测试：

* `${user}是<#if user=="老高">我们的老师</#if><#-- 简单示例 -->`
* `<#if num0 gt 18> 及格<#else>不及格！</#if><#-- if else 测试 -->`
* `<#if random gte 90>优秀！<#elseif random gte 80>良好！<#else>一般！</#if><#-- if else if else 测试 -->`

上述语句很容易看懂，因此结果就不再贴出了。

##### list指令
我们首先增加一个简单的Address类，然后创建如下的List对象，并放入map中：

``` java
List list = new ArrayList();
list.add(new Address("中国","北京"));
list.add(new Address("中国","上海"));
list.add(new Address("美国","纽约"));
root.put("lst", list);
```
测试list指令：
`<#list lst as dizhi ><b>${dizhi.country}</b> <br/></#list>`
控制台输出结果为：`<b>中国</b> <br/>``<b>中国</b> <br/>``<b>美国</b> <br/>`

##### include指令
一个非常简单的指令，与jsp相似:`<#include "included.txt" />`

##### 自定义指令(macro指令)
定义：`<#macro m1>   <#--定义指令m1 --><b>aaabbbccc</b><b>dddeeefff</b></#macro>`

调用：`<@m1 /><@m1 />  <#--调用上面的宏指令 -->`

定义带参的宏指令：
`<#macro m2 a b c >${a}--${b}--${c}</#macro>`

调用带参的宏指令：`<@m2 a="老高" b="老张" c="老马" />`

###### nested指令：
`<#macro border><#nested></#macro>`

调用：`<@border >nested中要显示的内容内容！</@border>` ，
我们很好理解的就是，调用后，`<#nested>`会被替换为你调用时所写的内容。

##### 命名空间
当运行 FTL 模板时，就会有使用`assign`和`macro`指令创建的变量的集合（可能是空的），
可以从前一章节来看如何使用它们。像这样的变量集合被称为`namespace`命名空间。
在简单的情况下可以只使用一个命名空间，称之为`main namespace`主命名空间。
因为通常只使用本页上的命名空间，所以就没有意识到这点。 
如果想创建可以重复使用的宏，函数和其他变量的集合，通常用术语来说就是引用
`library`库。使用多个命名空间是必然的。只要考虑你在一些项目中，或者想和他人共享使用的时候，
你是否有一个很大的宏的集合。但要确保库中没有宏（或其他变量）名和数据模型中变量同名，
而且也不能和模板中引用其他库中的变量同名。通常来说，变量因为名称冲突也会相互冲突。
所以要为每个库中的变量使用不同的命名空间。

首先创建一个b.ftl文件，内容如下：
`<#macro copyright date>${date}</#macro> <#assign mail = "bjsxt@163.com">`

然后，我们在a.ftl中引用：
`<#import "b.ftl" as bb  />`
`<@bb.copyright date="2010-2011" />`
`${bb.mail}`
`<#assign mail="my@163.com"  />`
`${mail}`
`<#assign mail="my@163.com" in bb  />`
`${bb.mail}`

控制台输出结果：
`2010-2011` `bjsxt@163.com` `my@163.com` `my@163.com`

##### 命名空间命名规则

如果你为Example公司工作，它们拥有www.example.com网的主页，你的工作是开发
一个部件库，那么要引入你所写的 FTL 的路径应该是： 
`/lib/example.com/widget.ftl`

注意到`www`已经被省略了。第三次路径分割后的部分可以包含子目录，可以像下面这
样写： `/lib/example.com/commons/string.ftl` 
一个重要的规则就是路径不应该包含大写字母，为了分隔词语，使用下划线`_`，就像
`wml_form`（而不是 `wmlForm`）。 

如果你的工作不是为公司或组织开发库，也要注意，你应该使用项目主页的 URL，比如
`/lib/example.sourceforge.net/example.ftl`或`/lib/geocities.com/jsmith/example.ftl`。