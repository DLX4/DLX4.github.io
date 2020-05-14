---
title: 用100块钱搭建一个网站
tags:
  - wordpress
url: 70.html
id: 70
categories:
  - 生活随笔
date: 2019-10-13 16:50:03
---

网站搭建方案：基于腾讯乞丐版云服务器 \+ wordpress + vue酷炫主题。

第一步：购买一台腾讯云服务器，配置（1核1GB），centos操作系统。拿到服务器之后，按照腾讯云的教程[手动搭建wordpress个人站点](https://cloud.tencent.com/document/product/213/8044)，整个过程没有什么坑，最后我们能访问刚刚搭建的wordpress（默认主题），可以点点点，创建文章，评论等。

第二步：给wordpress换一个酷炫的主题。我挑选了一款用vue写的主题，按照主题[作者提供的教程](https://www.xuanmo.xin/details/2987)配置一下。

首先要安装“WP REST API”插件，由于现在（20191012）已经下架了，可以从github上下载这个插件的[源码](https://github.com/WP-API/WP-API)，可以直接下载zip，然后在wordpress后台手动安装插件，上传zip文件即可。

![](http://106.54.113.128/wordpress/wp-content/uploads/2019/10/image.png)

安装好rest插件之后前端可以用rest接口访问wordpress的业务数据。安装插件之后还需要配置nginx服务器，要执行阅读rest插件的[开发文档](https://developer.wordpress.org/rest-api/)。

location /wordpress/ {  
try_files $uri $uri/ /wordpress/index.php?$args;  
}

正确配置好rest插件后，[测试](http://domain/wp-json)下rest接口是否能返回json数据。

第二个要注意的地方，在服务器上编译运行主题项目时，由于腾讯云服务器乞丐版配置过低，会报错（错误码137 内存不足），这时候需要给服务器配置swap（实际测试至少要再配置300M，稳妥一点可以配置1G）。主题运行起来以后，发现从外网不能访问3000端口，因为主题监听的是localhost:3000端口，因此需要在nginx配置代理。

server {  
listen 80;  
root /usr/share/nginx/html;  
server_name localhost;  
location / {  
#index index.php index.html index.htm;  
proxy_pass http://127.0.0.1:3000;  
}

location /wordpress/ {  
try_files $uri $uri/ /wordpress/index.php?$args;  
}

这样就大功告成了。最后可以在腾讯云上买一个域名，需要申请备案。