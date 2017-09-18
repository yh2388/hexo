---
title: Java发送简单邮件
date: 2017-05-11 20:40:50
tags: 小玩意儿
---
发送邮件是我们日常生活中经常会做的事情，那么，如何使用Java发送一封邮件呢？我写了一个这样的小demo，用来实现Java发送邮件，一起来看一下吧。

<!-- more -->

``` java
import java.security.GeneralSecurityException;
import java.util.Properties;

import javax.mail.Address;
import javax.mail.Message;
import javax.mail.MessagingException;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.AddressException;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;

import com.sun.mail.util.MailSSLSocketFactory;

public class JavaMailTest {
    public static void main(String[] args) throws AddressException, MessagingException, GeneralSecurityException {
		Properties prop = new Properties();
        //邮件相关配置；服务器地址、端口号等
        prop.setProperty("mail.transport.protocol", "smtp");
        prop.setProperty("mail.smtp.host", "smtp.qq.com");
        prop.setProperty("mail.smtp.auth", "true");
        prop.put("mail.smtp.port","465");
        prop.setProperty("mail.debug", "true");
        MailSSLSocketFactory sf = new MailSSLSocketFactory();
        sf.setTrustAllHosts(true);
        prop.put("mail.smtp.ssl.enable", "true");
        prop.put("mail.smtp.ssl.socketFactory", sf);
        //创建会话
        Session session = Session.getInstance(prop);
        //填写信封写信
        Message msg = new MimeMessage(session);
        msg.setFrom(new InternetAddress("178317391@qq.com"));
        //设置邮件标题
        msg.setSubject("拉拉拉啊啦啦啦");
        //设置邮件内容（可使用HTML标签）
        msg.setText("123235412341234234");
        //验证用户名密码发送邮件
        Transport transport = session.getTransport();
        transport.connect("178317391@qq.com","zehdfovpffcicabb");
        //使用sendMessage，防止tx误认为垃圾邮件
        transport.sendMessage(msg,new Address[]{new InternetAddress("1049097603@qq.com")});
	}
}
```
通过上面的代码，我们就可以发送一封很简单的邮件了。这里有一个要注意的地方就是，邮箱账号必须要开启`SMTP`服务。

我们上面用到了一个jar包，即`JavaMail`，可以去官网或者其他正规渠道下载获取。当然，我们可以同样使用这个jar包来开发发送复杂邮件的工具，
具体的网上已经有很多教程了，这里就不再写了。