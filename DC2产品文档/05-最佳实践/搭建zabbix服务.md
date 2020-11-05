## 相关知识介绍

**Zabbix**是一个基于web界面的企业级开源监控软件，Zabbix服务器需要LAMP环境或LNMP环境，提供分布式系统监控与网络监视功能。具备主机的性能监控，网络设备性能监控，数据库性能监控，多种告警方式，详细报表、图表的绘制等功能。监测对象可以是Linux或Windows服务器，也可以是路由器、交换机等网络设备，通过SNMP、Zabbix Agent、Ping、端口监视等方法提供对远程网络服务器等监控、数据收集等功能。


## 搭建环境和安装方式

+ 一台滴滴云主机+EIP。

+ CentOS-7.6+ MariaDB-5.5+Zabbix-4.4+httpd-2.4.6。

+ 安装方式：通过RPM包。


## 搭建Zabbix服务器

1.安装httpd+php服务，并设置为开机自启动（一般默认已安装)。

```
yum -y install httpd
systemctl start httpd
systemctl status httpd
systemctl enable httpd
yum –y install php php-mysql
```

![](https://docpics.s3.didiyunapi.com/zabbix.service/1-1.png)
![](https://docpics.s3.didiyunapi.com/zabbix.service/1-2.png)

2.修改timezone，否则后期检查会不通过。

	vim /etc/httpd/conf.d/zabbix.conf
	
解除***php_value date.timezone***的注释，并将值改为***Asia/Shanghai。***

然后，重启httpd:

	systemctl restart httpd

![](https://docpics.s3.didiyunapi.com/zabbix.service/2-1.png)

3.安装MariaDB数据库。

	yum install -y mariadb mariadb-server
![](https://docpics.s3.didiyunapi.com/zabbix.service/3-1.png)

4.启动数据库并设置开机自启动。

```
systemctl enable mariadb 
systemctl start mariadb
systemctl status mariadb
```
![](https://docpics.s3.didiyunapi.com/zabbix.service/4-1.png)
![](https://docpics.s3.didiyunapi.com/zabbix.service/4-2.png)

设置数据库密码：```mysqladmin -u root -p password```
回车+回车+输入新密码+确认新密码(此处设置密码为：123456)。

![](https://docpics.s3.didiyunapi.com/zabbix.service/4-3.png)

5.创建Zabbix使用的数据库。

	create database zabbix character set utf8 collate utf8_bin;
	grant all privileges on zabbix.* to zabbix@localhost identified by '123456';
	
导入初始架构和数据，系统将提示您输入新创建的密码(即123456)。

	zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix

6.为Zabbix server配置数据库编辑配置文件。

	vim /etc/zabbix/zabbix_server.conf
	设置：DBPassword=123456

7.安装Zabbix repository。

	rpm -Uvh https://repo.zabbix.com/zabbix/4.4/rhel/7/x86_64/zabbix-release-4.4-1.el7.noarch.rpm

![](https://docpics.s3.didiyunapi.com/zabbix.service/7-1.png)

8.安装Zabbix server，Web前端，agent。

	yum install zabbix-server-mysql zabbix-web-mysql zabbix-agent
官网yum源下载失败（官方源在国外，由于网络问题导致）。

![](https://docpics.s3.didiyunapi.com/zabbix.service/8-1.png)

更换源：（阿里、腾讯的源都可以）。
	
	vim /etc/yum.repos.d/zabbix.repo
	阿里：https://mirrors.aliyun.com/zabbix/zabbix/4.4/rhel/7/x86_64/
	腾讯：https://mirrors.cloud.tencent.com/zabbix/zabbix/4.4/rhel/7/x86_64/

9.连接到新安装的Zabbix前端，浏览器输入：<http://EIP/zabbix> (确保安全组放开入方向80端口)。


![](https://docpics.s3.didiyunapi.com/zabbix.service/9-1.png)

10.配置数据库，密码即之前设置的123456。

![](https://docpics.s3.didiyunapi.com/zabbix.service/10-1.png)

11.Zabbix服务器的名称，可选。

![](https://docpics.s3.didiyunapi.com/zabbix.service/11-1.png)

安装信息汇总：

![](https://docpics.s3.didiyunapi.com/zabbix.service/11-2.png)

开始安装：

![](https://docpics.s3.didiyunapi.com/zabbix.service/11-3.png)

12.进入登录界面：默认用户名Admin，密码：zabbix。

![](https://docpics.s3.didiyunapi.com/zabbix.service/12-1.png)

登录成功！

![](https://docpics.s3.didiyunapi.com/zabbix.service/12-2.png)

13.默认是英文，点击右上角的人形图标，选择[Language]-[Chinese(zh_CN)]，然后点击下方的Update按钮。

![](https://docpics.s3.didiyunapi.com/zabbix.service/13-1.png)

好了，现在就是中文界面了！

![](https://docpics.s3.didiyunapi.com/zabbix.service/13-2.png)














 





















