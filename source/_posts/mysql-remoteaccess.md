---
title: MySql开启允许远程访问
date: 2017-08-04 20:40:50
tags: MySql
---
## 为什么需要为MySql开启允许远程访问
1.MySql-Server 出于安全方面考虑只允许本机(localhost, 127.0.0.1)来连接访问. 这对于 Web-Server 与 MySql-Server 都在同一台服务器上的网站架构来说是没有问题的. 但随着网站流量的增加, 后期服务器架构可能会将 Web-Server 与 MySql-Server 分别放在独立的服务器上, 以便得到更大性能的提升, 此时 MySql-Server 就要修改成允许 Web-Server 进行远程连接.

2.不用每次都登到服务器去添加修改表，只要用图形化界面即可远程管理。

<!-- more -->
向Mac下优秀的markdown编辑器mou致敬

### 我们可以按照下面的步骤修改
1.登陆MySql，切换到mysql库
``` sql
use mysql --切换到mysql db
```
2.查看现有用户、密码及允许链接的主机
``` sql
select user,password,host from user where user='root'; --正常情况下只有一个root用户
```
3.如果只有一个root用户且允许访问的主机为本机时，我们再可以添加一个root用户数据
``` sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;

```
这样，我们就将远程访问的权限分配给了任意的主机，且通过密码‘password’登录。
3.我们也可以通过update更新root的host信息，但并不推荐
``` sql
UPDATE user SET Host='192.168.1.100' WHERE User='root' AND Host='localhost' LIMIT 1;
```
4.最后，刷新刚刚修改的权限信息，否则不起作用。
``` sql
flush privileges;
```
### 忘记root密码时修改密码
1.跳过权限检查进入mysql。
在mysql的安装目录下执行一下命令
``` shell
mysqld --defaults-file="C:\hostadmin\mysql\my.ini" --console --skip-grant-tables
mysql -u root -p #需要输入密码时直接按回车跳过
```
2，登录mysql后，切换到user db，修改root的密码
``` sql
update user set password=PASSWORD('123456') where user='root';
flush privileges;
```
这样，我们就可以解决因为忘记root密码而无法登陆mysql的问题了

##### 参考：http://www.cnblogs.com/easyzikai/archive/2012/06/17/2552357.html