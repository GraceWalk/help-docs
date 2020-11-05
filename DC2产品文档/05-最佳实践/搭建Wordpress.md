## Wordpress介绍
Wordpress是使用PHP语言开发的博客平台，是一款个人博客系统，并逐步演化成一款内容管理系统软件，它是使用PHP语言和MySQL数据库开发的,用户可以在支持 PHP 和 MySQL数据库的服务器上使用自己的博客，也可以把 Wordpress当作一个内容管理系统（CMS）来使用。Wordpress有许多第三方开发的免费模板，安装方式简单易用。

## 环境准备

1. DC2服务器一台（包括公网IP），创建DC2服务器请参考：[创建DC2服务器](https://help.didiyun.com/hc/kb/article/1145869/)

2. Centos-7.x操作系统，远程访问DC2服务器请参考：[连接方式](https://help.didiyun.com/hc/kb/article/1134446/)

3. Nginx、Mysql、PHP环境。

## 三、安装Nginx服务

1. 使用yum方式安装Nginx服务。

		yum install -y nginx
	
 ![](https://docpics.s3.didiyunapi.com/wordpress/LNMP_1.png)
 
2. 安装完成后生成Nginx配置文件。
	
		mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak  #将自动生成的配置文件替换
		mv /etc/nginx/nginx.conf.default  /etc/nginx/nginx.conf  #将模版配置文件修改为加载配置文件
	
3. 启动Nginx服务并配置为随机启动。

		systemctl start nginx.service      #启动nginx 
		systemctl enable nginx.service     #将nginx设置为随机启动 
	
4. 使用DC2绑定的公网IP测试访问。

  ![](https://docpics.s3.didiyunapi.com/wordpress/LNMP_2.png)
 
## 安装PHP环境
1. 使用yum方式安装PHP环境和依赖。

		yum -y install  php-fpm   php php-mysql php-gd libjpeg* php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-bcmath php-mhash

2. 启动PHP服务并配置为随机启动。

		systemctl start php-fpm.service        #启动PHP服务
		systemctl enable php-fpm.service       #配置随机启动
3. 修改Nginx配置文件，使其加载PHP服务。

		vim /etc/nginx/nginx.conf  #添加index.php并修改PHP加载配置，完整配置文件如下
	
 ![](https://docpics.s3.didiyunapi.com/wordpress/LNMP_3.png)
 ![](https://docpics.s3.didiyunapi.com/wordpress/LNMP_4.png)
 
4. 重启Nginx和PHP服务。
	
		systemctl restart nginx.service
		systemctl restart php-fpm.service
5. 测试访问Nginx是否支持PHP语言。

		vim /usr/share/nginx/html/index.php
		<?php
		echo phpinfo();
		?>
 ![](https://docpics.s3.didiyunapi.com/wordpress/LNMP_5.png)
 
## 安装Mariadb服务
在Centos-7+版本后Mariadb代替Mysql服务（两者区别和介绍可参考[百度百科](https://baike.baidu.com/item/mariaDB/6466119?fr=aladdin)）。

1. 使用yum方式安装Mariadb服务。

	yum -y install mariadb mariadb-server
![](https://docpics.s3.didiyunapi.com/wordpress/LNMP_6.png)
	
2. 启动Mariadb服务。

		systemctl start mariadb
	
3. 配置访问密码和授权登录。

		mysql -u root #登录mariadb服务（默认没有密码）
		MariaDB [(none)]>  set password for root@localhost = password('Didiyun888'); #配置密码
		MariaDB [(none)]>  grant all privileges on *.* to 'root'@'%' identified by 'Didiyun888' with grant option; #授权远程访问
	
## 安装wordperss

1. 下载安装包并解压到指定目录。

		wget https://Wordpress.org/latest.tar.gz #下载最新版本
		tar -zxf Wordpress-5.3.2.tar.gz -C /usr/share/nginx/html/ #解压到web服务跟目录
	
2. 修改默认配置。

		cd Wordpress/
		cp wp-config-sample.php wp-config.php
		vim wp-config.php
 ![](https://docpics.s3.didiyunapi.com/wordpress/LNMP_7.png)
	
3. 打开浏览器输入http://IP/Wordpress/wp-admin/install.php 填写相关信息后就按照完成了。

 ![](https://docpics.s3.didiyunapi.com/wordpress/LNMP_8.png)


4. 关于更多Wordpress的使用请参考官网：[Wordpress](Wordpress.org)
