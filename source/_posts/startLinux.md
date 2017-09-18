---
title: 初入Linux
date: 2017-07-03 20:40:50
tags: linux
---
### 终于开始了Linux的学习，纪念一下
因为是初学，之前也没怎么接触过linux，因此，这里简单记录几个常用的命令。
### 1.环境搭建
这里我使用的linux版本是centos 7。在虚拟机中安装好系统后，出于降低学习成本的考虑，我安装了图形界面，提高友好度，熟悉了一个上午后，开始使用CRT工具作为我的操作终端。
<!-- more -->
#### 1.1 首先，需要安装X，命令如下
``` shell
yum groupinstall "X Window System"
```
#### 1.2 检查一下我们已经安装的软件以及可以安装的软件
``` shell
yum grouplist
```
#### 1.3 选择要安装的图形界面软件
我选择的是安装GNOME，找到列表中对应的名字，一定要注意 名称必须对应 不同版本的centOS的软件名可能不同 其他Linux系统类似
否则会出现No packages in any requested group available to install or update 的错误。
``` shell
yum groupinstall "GNOME desktop"
```
安装完成后，在登陆系统后使用startx命令就可以进入图形界面了，之后，我惊奇的发现，GNOME安装好后，基本的一些
开发环境都已经配置完成了，这也算是linux对于开发友好的一种体现吧。

### 2.常用命令
#### 2.1 安装命令
对于centos等redhat系统，有yum命令，而一般linux系统，都可以使用rpm进行软件的安装与卸载
``` shell
rpm  -ivh  your-package.rpm #安装软件
rpm  -e --nodeps your-package.rpm #卸载软件
```
#### 2.2 文件查看
1. cat 查看小文件
2. less  -Nm 查看大文件并显示行号与百分比，Enter查看下一行，空格键查看下一页，q退出查看
3. tail -20  查看文件的最后几行
4. tail -f  filename 动态查看文件，如log文件

### 3.端口开放
#### 3.1 linux访问端口开放
``` shell
/sbin/iptables -I INPUT -p tcp --dport 8080 -j ACCEPT;
/etc/rc.d/init.d/iptables save #将修改信息保存到路由表
```
#### 3.2 centos 7访问端口开放
``` shell
firewall-cmd --zone=public --add-port=3306/tcp --permanent #--permanent是永久生效
firewall-cmd --reload #重启生效
```
#### 3.3 查询已开放的端口
``` shell
firewall-cmd --query-port=8080/tcp
```

以上，就是我在接触linux之初，所遇到的一些问题，其中还进行了一些应用软件的安装配置，由于网上有大量信息，而且操作过程中也没有遇到
太多问题，因此就不再做记录。