---
title: 使用Jsoup爬取网站图片
date: 2017-05-25 20:40:50
tags: 小玩意儿
---

首先我们需要导入`jsoup.jar`才能够使用jsoup，jar包可以在官网下载：https://jsoup.org/download。

<!-- more -->

``` java
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.net.URL;
import java.net.URLConnection;
import java.util.UUID;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

public class JsoupTest {
    public static void main(String[] args) {
		String url="https://www.qiushibaike.com/imgrank/";
	    try {
			Document doc = Jsoup.connect(url).get();
			String title = doc.title();
			Elements img=doc.getElementsByTag("img");
			for (Element element : img) {
				getImage(element.attr("src"));
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	public static void getImage(String imgUrl){
		try {
			URL url;
			String u=imgUrl.split("[?]")[0];
			//判断是否为网站的静态资源图片
			if("/static".equals(imgUrl.substring(0,7))){
				url=new URL("https://www.qiushibaike.com/imgrank/"+u);
			}else{
				//链接前需要加上Http: 否则报错
				url=new URL("http:"+u);
			}
			URLConnection conn=url.openConnection();
			InputStream in=conn.getInputStream();
			//设置保存的路径及文件名
			File file=new File("d:\\迅雷下载\\"+UUID.randomUUID().toString()+".jpg");
			FileOutputStream out = new FileOutputStream(file);
			byte[] b=new byte[1024];
			int length;
			while((length=in.read(b))!=-1){
				out.write(b, 0, length);
			}
			in.close();
			out.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```
运行上面的代码，我们就可以轻松的获取到页面中的图片了，并且保存在了D盘的迅雷下载目录。