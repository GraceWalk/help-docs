## 相关知识介绍
**Nginx (engine x)** 是一个基于C语言开发的高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。由俄罗斯的程序设计师 ***Igor Sysoev*** 所开发，官方测试**Nginx**能够支撑5万并发连接，并且CPU、内存等资源消耗却非常低，运行非常稳定。

在高连接并发的情况下，Nginx是Apache服务器不错的替代品。

官方下载地址：[官网下载](http://nginx.org/en/download.html)

本次搭建使用的版本，即当前最新稳定版：**nginx-1.16.1**。

##环境准备

+ 一台DC2云主机（CentOS7.6 x64）。
+ 一个EIP并绑定到云主机。

##搭建步骤

###安装依赖包

	yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel pcre-devel gcc

###安装并配置Nginx

1.下载Nginx安装包。

	wget http://nginx.org/download/nginx-1.16.1.tar.gz
	
2.解压缩。

	tar -zxvf nginx-1.16.1.tar.gz
	
3.进入安装目录并查看。

	cd nginx-1.16.1
![](https://docpics.s3.didiyunapi.com/nginx.service/3-2-3.png)

4.编译安装。

	./configure --prefix=/usr/local/nginx

![](https://docpics.s3.didiyunapi.com/nginx.service/3-2-4-1.png)

	make && make install

安装后的目录结构：

![](https://docpics.s3.didiyunapi.com/nginx.service/3-2-4-2.png)

5.验证安装是否成功。

	ln -s /usr/local/nginx/sbin/nginx /bin/
	nginx -t -c /usr/local/nginx/conf/nginx.conf

![](https://docpics.s3.didiyunapi.com/nginx.service/3-2-5.png)

6.启动Nginx服务。

	nginx -c /usr/local/nginx/conf/nginx.conf

7.查看进程。

	ps -ef|grep nginx|grep –v grep

![](https://docpics.s3.didiyunapi.com/nginx.service/3-2-7.png)

8.使用浏览器访问。

浏览器地址栏输入EIP地址，显示Nginx的欢迎界面，即为成功。

![](https://docpics.s3.didiyunapi.com/nginx.service/3-2-8.png)


##常用命令

	nginx –s reload   ##重新载入配置文件
	nginx –s reopen   ##重启
	nginx –s stop     ##停止


##常见问题

###启动失败，端口被占用

查看占用端口的程序。

	netstat –tunlp|grep 80
	ps –ef|grep $pid
	
###修改端口号

	vim /usr/local/nginx/conf/nginx.conf

![](https://docpics.s3.didiyunapi.com/nginx.service/5-2.png)

> 注意：修改完端口后，需要在控制台-安全组里放开对新端口的访问。
