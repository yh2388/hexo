---
title: servlet中spring注解自动注入失败的问题
date: 2017-08-21 20:40:50
tags: spring
---

今天做一个在线测评的时候遇到了一个问题，那就是只使用spring框架的情况下，servlet中spring的
自动注入总是失败，在踩了各种坑后，终于找到了解决的办法。这边就直接贴链接了，因为实在是太累。
看到有人说使用xml配置的情况下事可以的，所以，等下次测试一下xml的情况下是否真的有用，再来更新。

链接：

1. http://blog.csdn.net/l1028386804/article/details/45696707
2. [这个链接太长，括起来](https://m.baidu.com/from=1000539d/bd_page_type=1/ssid=0/uid=0/pu=usm%401%2Csz%401320_2001%2Cta%40iphone_1_10.3_3_603/baiduid=D60F81A1F7CB29BBDB59EB8087B453E7/w=0_10_/t=iphone/l=3/tc?ref=www_iphone&lid=15128462209477335695&order=3&fm=alop&tj=www_normal_3_0_10_title&vit=osres&m=8&srd=1&cltj=cloud_title&asres=1&title=...管理的bean使用注解的方式注入到servlet中..._博客园&dict=30&w_qd=IlPT2AEptyoA_yimGVGrJyYgOVYPt9VnFQKDLCBTMjirnE3&sec=23385&di=bcb48549e764976c&bdenc=1&tch=124.317.271.594.1.214&nsrc=IlPT2AEptyoA_yixCFOxXnANedT62v3IEQGG_ytK1DK6mlrte4viZQRAWDbuLmyTGojxxWf0sqdFtXLR_m9o9Bp1qrIwdzZz&eqid=d1f31839b9d1580010000005599aaaba&wd=&clk_info=%7B"srcid"%3A"1599"%2C"tplname"%3A"www_normal"%2C"t"%3A1503308528384%2C"sig"%3A"53710"%2C"xpath"%3A"div-a-h3-em3"%7D&sfOpen=1)

再次感谢大佬！！！