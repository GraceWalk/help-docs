##相关知识介绍

**FTP(File transfer protocol)**文件传输协议，用于在服务器和客户端之间进行文件的传输。基于客户端/服务器模式，默认使用20、21端口，其中端口20（数据端口）用于数据传输，端口21（命令端口）用于接受客户端发出的ftp命令与参数。FTP协议的传输拓扑如下图所示。

![](https://docpics.s3.didiyunapi.com/ftp.service/0.png)

+ FTP协议有两种工作模式：
	- 主动模式：FTP服务器主动向客户端发起连接。
	- 被动模式：FTP服务器等待客户端发起连接请求（默认工作模式）。
	
+ vsftpd允许用户以三种认证模式登录到FTP服务器上，分别是：
	- 匿名用户模式：任何人都可以无需密码直接登录到服务器。
	- 本地用户模式：通过Linux系统本地在账号密码进行认证。
	- 虚拟用户模式：专门为ftp服务建立的用户。

##环境准备
+ 一台滴滴云主机DC2（含EIP）作为FTP服务器。

+ 创建DC2服务器请参考：[DC2创建](https://help.didiyun.com/hc/kb/article/1145869/)

+ 操作系统：CentOS-7.6 64bit。

##搭建步骤

###登录DC2，查看是否安装了vsftpd服务

`rpm –q vsftpd`

![](https://docpics.s3.didiyunapi.com/ftp.service/1.png)

###安装vsftpd服务

`yum –y install vsftpd`

![](https://docpics.s3.didiyunapi.com/ftp.service/2-1.png)

再次检查是否安装成功：`rpm –q vsftpd`

![](https://docpics.s3.didiyunapi.com/ftp.service/2-2.png)

###启动ftp服务并设置为开机自启动

```
systemctl start vsftpd
systemctl enable vsftpd
systemctl status vsftpd

```

![](https://docpics.s3.didiyunapi.com/ftp.service/3.png)

###创建虚拟宿主用户

```
useradd didiyun-virtuser -s /sbin/nologin
echo "Didiyun1234" | passwd didiyun-virtuser --stdin

```
![](https://docpics.s3.didiyunapi.com/ftp.service/4.png)

> 注意，密码长度和复杂的要符合要求，否则会报错。

###备份并编辑配置文件/etc/vsftpd/vsftpd.conf

```
mv /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.bak
vim /etc/vsftpd/vsftpd.conf

```
配置文件内容及说明：

```
# 是否允许匿名登陆
anonymous_enable=NO

# 是否允许本地用户登陆
local_enable=YES

# 是否具备写入权限
write_enable=NO

# 上传文件的权限掩码
local_umask=022

# 是否进行日志记录
xferlog_enable=YES
xferlog_std_format=YES
xferlog_file=/var/log/vsftpd.log

# FTP开启数据传输端口20
connect_from_port_20=YES

# 会话超时时间/秒
idle_session_timeout=600

# 数据传输连接超时时间
data_connection_timeout=120

# 是否允许进行异步传输
async_abor_enable=YES

# 请求欢迎信息
ftpd_banner=Welcome to FTP service on DC2.

# linux中FTP默认情况下允许用户从ftp主目录切换到linux系统的其他目录
# 可开启chroot_list_enable功能，通过列表文件，进行用户限制
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd/chroot_deny

# 是否允许FTP用户使用"ls -R"命令，该命令会对系统造成较大的负担
ls_recurse_enable=NO

# 设定vsftpd服务工作在standalone模式，可理解为拥有自己的守护进程
listen=YES

# 设定pam服务下的vsftpd验证配置文件名，pam验证配置文件位于/etc/pam.d/目录下
pam_service_name=vsftpd

# 是否启用userlist列表限制功能
userlist_enable=YES

# 是否支持TCPwrappers
tcp_wrappers=YES

# 是否进行反向域名解析
reverse_lookup_enable=NO

# 启用虚拟用户功能
guest_enable=YES

# 指定虚拟的宿主用户
guest_username=didiyun-virtuser

# 虚拟用户权限符合它们的宿主用户
virtual_use_local_privs=YES

# 指定虚拟用户的配置文件目录
user_config_dir=/etc/vsftpd/vconf

```

###创建pam服务识别的虚拟账户文件（虚拟用户：tt,密码：didiyun123）

```
mkdir /etc/vsftpd/vconf
cd /etc/vsftpd/vconf/
echo -e "tt\ndidiyun123" > password
db_load -T -t hash -f password{,.db}

```
###配置vsftpd的pam账号认证文件，添加如下两行。

```
auth       sufficient   /lib64/security/pam_userdb.so    db=/etc/vsftpd/vconf/password
account    sufficient   /lib64/security/pam_userdb.so    db=/etc/vsftpd/vconf/password

```

![](https://docpics.s3.didiyunapi.com/ftp.service/7.png)

###配置共享目录并配置虚拟用户的个人配置文件

```
mkdir /ftpdata
chown -R didiyun-virtuser:didiyun-virtuser /ftpdata/
vim /etc/vsftpd/vconf/tt

```
内容及说明：

```
#指定虚拟用户仓库的具路径
local_root=/ftpdata

#允许写的操作
write_enable=YES

#不允许下载操作
download_enable=NO

#设定并发客户端的访问数量
max_clients=100

#设定客户端的最大线程数
max_per_ip=10

#设定用户的最大传输速率，单位b/s
local_max_rate=102400

```

###登录控制台，修改安全组，增加两条入方向ftp端口规则

![](https://docpics.s3.didiyunapi.com/ftp.service/9.png)

###创建测试文件并安装客户端测试

```
ls -l / > /ftpdata/testfile.list
yun –y install ftp
ftp <eip>

```

