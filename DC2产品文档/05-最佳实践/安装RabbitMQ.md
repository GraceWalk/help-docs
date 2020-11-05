
## RabbitMQ简介
RabbitMQ是一个开源的AMQP实现，服务器端用Erlang语言编写，支持多种客户端，如：Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP等，支持AJAX。用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。

## 环境准备

1. DC2服务器一台（包括公网IP），创建DC2服务器请参考：[创建DC2服务器](https://help.didiyun.com/hc/kb/article/1145869/)

2. Centos-7.x操作系统，远程访问DC2服务器请参考：[连接方式](https://help.didiyun.com/hc/kb/article/1134446/)

	
3. 由于RabbitMQ是基于erlang语言开发的，所以必须先安装erlang。
	
## 安装erlang环境
1. 安装依赖包。

		yum -y install gcc glibc-devel make ncurses-devel openssl-devel xmlto perl wget gtk2-devel binutils-devel
 ![](https://docpics.s3.didiyunapi.com/MQ/MQ1.png)
	
2. 下载erlang并安装（官网下载会比较慢，我们将下载好的安装包放到了滴滴云S3服务中，以便提升下载速度，如果需要其他版本可以移步到：www.erlang.org/downloads）。

		wget https://docpics.s3.didiyunapi.com/MQ/otp_src_22.0.tar.gz #下载
		tar -zxf otp_src_22.0.tar.gz -C /usr/local/ #解压
		cd /usr/local/otp_src_22.0 #切换到安装目录
		mkdir ../erlang #创建安装目录
		./configure --prefix=/usr/local/erlang #执行安装命令
		make install #安装
	在安装过程中如遇到以下错误，忽略即可。
 ![](https://docpics.s3.didiyunapi.com/MQ/MQ2.png)
3. 配置环境。

		echo 'export PATH=$PATH:/usr/local/erlang/bin' >> /etc/profile #添加环境变量
		source /etc/profile #重载环境变量
		erl #测试进入erl环境，如图
 ![](https://docpics.s3.didiyunapi.com/MQ/MQ3.png)
	
## 安装RabbitMQ
1. 下载安装包(github下载较慢，我们将下载好的安装包放到了滴滴云S3服务中，以便提升下载速度)。

		wget https://docpics.s3.didiyunapi.com/MQ/rabbitmq-server-generic-unix-3.7.15.tar.xz

2. 解压安装包。

		/bin/xz -d rabbitmq-server-generic-unix-3.7.15.tar.xz #第一次解压
		tar -xf rabbitmq-server-generic-unix-3.7.15.tar -C /usr/local/ #第二次解压
		mv /usr/local/rabbitmq_server-3.7.15  /usr/local/rabbitmq #重命名
3. 配置环境。

		echo 'export PATH=$PATH:/usr/local/rabbitmq/sbin' >> /etc/profile  #添加环境变量
		source /etc/profile #重载环境变量
		mkdir /etc/rabbitmq #创建配置目录

4. 启动RabbitMQ服务。

		rabbitmq-server -detached #启动
		rabbitmqctl stop #停止
		rabbitmqctl status #查询状态
	
5. 开启Web插件。

		rabbitmq-plugins enable rabbitmq_management
	
6. 打开浏览器测试访问公网IP(确保安全组开放15672端口)。

 ![](https://docpics.s3.didiyunapi.com/MQ/MQ4.png)

7. 用户管理。

		rabbitmqctl list_users #查看所有用户
		rabbitmqctl add_user didiyun didiyun@888 #添加用户
		rabbitmqctl set_permissions -p "/" didiyun ".*" ".*" ".*" #配置权限
		rabbitmqctl list_user_permissions didiyun #查看用户权限
		rabbitmqctl set_user_tags didiyun administrator #为用户设置tag
		rabbitmqctl delete_user guest #为了保证安全，建议删除默认用户guest
	
8. 配置完用户后重启服务并登录成功。

		rabbitmqctl stop #停止
		rabbitmq-server -detached #启动

   ![](https://docpics.s3.didiyunapi.com/MQ/MQ5.png)






	
	



	
	
	
	
