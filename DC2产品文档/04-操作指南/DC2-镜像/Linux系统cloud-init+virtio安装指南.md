为保证运行该镜像的云服务器DC2能成功完成初始化配置，建议您在制作Linux类型自定义镜像时，在源服务器上安装cloud-init以及virtio。本文介绍安装社区版cloud-init的操作步骤。

##安装cloud-init

在云环境的虚拟机的创建过程中，需要cloud-init工具对虚拟机进行初始化配置，因此要使用自定义镜像，一定要确保原系统中已安装cloud-init， 如未安装，请执行以下步骤。

使用Python3安装cloud-init，本文介绍操作系统为Centos7.6，使用python3.7。

* 安装依赖包

> yum install  zlib-devel bzip2-devel libffi-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make -y

* 下载Python源码包

> wget -P /usr/local https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tgz

* 解压并安装

>cd /usr/local/
 
>tar -zxvf Python-3.7.0.tgz 

>cd Python-3.7.0/ 

>./configure 

>make && make install

* 创建软链

>mv /usr/bin/python /usr/bin/python.bak ln -s /usr/local/bin/python3 /usr/bin/python ln -s /usr/local/bin/pip3 /usr/bin/pip

* 确认python为python3.7

> [root@localhost bin]# python 
Python 3.7.0 (default, Feb 26 2020, 23:52:03) [GCC 4.8.5 20150623 (Red Hat 4.8.5-39)] on linux Type "help", "copyright", "credits" or "license" for more information. >>>

* Yum依赖python2，修改yum依赖

> vi /usr/libexec/urlgrabber-ext-down 

> vi /usr/bin/yum

* 下载cloud-init源码包

> wget -P /usr/local https://launchpad.net/cloud-init/trunk/17.1/+download/cloud-init-17.1.tar.gz

* 解压并安装

>cd /usr/local 

>tar -zxvf cloud-init-17.1.tar.gz 

>cd cloud-init-17.1 

>pip install -r requirements.txt


>> 根据不同操作系统，安装过程中可能会提示缺少相关模块或库文件，使用pip或yum安装即可。

* 安装cloud-utils组件

> yum install cloud-utils-growpart –y

* 安装cloud-init

> python setup.py build python setup.py install --init-system systemd

* 设置cloud-init命令软链

> ln -s /usr/local/bin/cloud-init /usr/bin/cloud-init

* 将 /lib/systemd/system/cloud-init-local.service 文件替换为如下内容：

>[Unit]

>Description=Initial cloud-init job (pre-networking) DefaultDependencies=no 

>Wants=network-pre.target 

>After=systemd-remount-fs.service 

>Requires=dbus.socket 

>After=dbus.socket 

>Before=NetworkManager.service network.service Before=network-pre.target 

>Before=shutdown.target 

>Before=firewalld.target 

>Conflicts=shutdown.target 

>RequiresMountsFor=/var/lib/cloud ConditionPathExists=!/etc/cloud/cloud-init.disabled ConditionKernelCommandLine=!cloud-init=disabled   

>[Service] 

>Type=oneshot 

>ExecStartPre=/bin/mkdir -p /run/cloud-init ExecStartPre=/sbin/restorecon /run/cloud-init ExecStartPre=/usr/bin/touch /run/cloud-init/enabled ExecStart=/usr/bin/cloud-init init --local ExecStart=/bin/touch /run/cloud-init/network-config-ready RemainAfterExit=yes TimeoutSec=0   # Output needs to appear in instance console output StandardOutput=journal+console 

>[Install] WantedBy=multi-user.target

* 将 /lib/systemd/system/cloud-init.service 文件替换为以下内容：


>[Unit] Description=Initial cloud-init job (metadata service crawler) Wants=cloud-init-local.service 

>Wants=sshd-keygen.service 

>Wants=sshd.service 

>After=cloud-init-local.service After=NetworkManager.service network.service Before=network-online.target 

>Before=sshd-keygen.service 

>Before=sshd.service 

>Before=systemd-user-sessions.service ConditionPathExists=!/etc/cloud/cloud-init.disabled ConditionKernelCommandLine=!cloud-init=disabled   

>[Service] 

>Type=oneshot 

>ExecStart=/usr/bin/cloud-init init 

>RemainAfterExit=yes 

>TimeoutSec=0   # Output needs to appear in instance console output StandardOutput=journal+console   

>[Install]     

>WantedBy=multi-user.target

* 设置 cloud-init 服务开机自启动

> systemctl enable cloud-init-local.service systemctl start cloud-init-local.service systemctl enable cloud-init.service systemctl start cloud-init.service systemctl enable cloud-config.service systemctl start cloud-config.service systemctl enable cloud-final.service systemctl start cloud-final.service

>注：执行完此步骤后请确认密码登录验证方式是否开启。

>vim /etc/ssh/sshd_config

##安装virtio

* 检查服务器内核是否以后virtio模块：

> grep -i virtio /boot/config-$(uname -r)

  1. 如果没有VIRTIO_BLK及VIRTIO_NET的信息说明该操作系统没有virtio内核模块，需要编译安装virtio驱动。如果看到以下输出：
  
  > 重点关注
  
  >CONFIG_VIRTID__BLK=m
  
  >CONFIG_VIRTID__NET=m

 2. 如果CONFIG_VIRTIO_BLK 参数和CONFIG_VIRTIO_NET 参数取值为 y，说明virtio已经被编译到内核中，并不是以模块形式存在，跳过后续步骤，virtio安装可以结束。
 
 3. 如果CONFIG_VIRTIO_BLKm 参数和CONFIG_VIRTIO_NET 参数取值为 m，执行以下步骤。
 
 * 确认virtio驱动是否包含在内存文件系统initramfs或者initrd中，根据不通操作系统执行以下命令：
 
 1. CentOS 6/CentOS 7/RedHat 6/RedHat 7 操作系统：
  
  >lsinitrd /boot/initramfs-$(uname -r).img | grep virtio

 2. RedHat 5/CentOS 5 操作系统：

  >mkdir -p /tmp/initrd && cd /tmp/initrd zcat /boot/initrd-$(uname -r).img | cpio –idmv find . -name "virtio*"
  
 3. Debian/Ubuntu 操作系统：
 
 >lsinitramfs /boot/initrd.img-$(uname -r) | grep virtio

* 如果输出为空，则需要根据不同操作系统，执行以下步骤修复内存文件系统：

* CentOS 6/CentOS 7/RedHat 6/RedHat 7 操作系统：

> cp /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak mkinitrd -f --with=virtio_blk --with=virtio_pci /boot/initramfs-$(uname -r).img\           $(uname -r)

* RedHat 5/CentOS 5 操作系统：

> cp /boot/initrd-$(uname -r).img /boot/initrd-$(uname -r).img.bak   mkinitrd -f --with=virtio_blk --with=virtio_pci /boot/initrd-$(uname -r).img $(uname    -r)

* Debian/Ubuntu 操作系统

>echo -e "virtio_pci\nvirtio_blk" >> /etc/initramfs-tools/modules update-initramfs  -u

*至此，cloud-init和virtio安装完毕，可以将自定义镜像导入滴滴云。









