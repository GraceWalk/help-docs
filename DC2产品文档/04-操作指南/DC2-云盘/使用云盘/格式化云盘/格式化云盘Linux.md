##格式化云盘Linux

全新云盘需要格式化后才能使用，需按如下步骤操作。

*  centos-7.3 DC2 实例为例，其他发行版可能命令有细微差别。

##操作步骤

 1. 登录DC2。
 
 2. 查看挂载之前DC2的块设备列表。
![image.png](http://help.didiyun.com/attachments/download/4722049/0015a9cbbb4217c353b77115b379e55/?filename=image.png)

3. 查看挂载之后DC2的块设备列表。
![image.png](http://help.didiyun.com/attachments/download/4722078/0015a9cbc3d9977e0519d7ff16bd144/?filename=image.png)
​	vdb 即是新挂载的云盘。
4. 在云盘上创建第一个分区。
 
 
* 对设备运行 fdisk，进入分区程程序。

	```
	[dc2-user@10-254-136-43 ~]$ sudo fdisk /dev/vdb
	Welcome to fdisk (util-linux 2.23.2). 
	
	Changes will remain in memory only, until you decide to write them.Be careful before using the write command. 
	
	Device does not contain a recognized partition table
	Building a new DOS disklabel with disk identifier 0xc14bbbbb. 
	Command (m for help):
	```

* 输入 n，并按回车，创建新分区。
	
	

	```
	Command (m for help): n
	Partition type:   
	    p   primary (0 primary, 0 extended, 4 free)
	    e   extended
	Select (default p):
	```



* 输入 p，并按回车。只要创建 4 个以下的分区，就选 p。


* 选择创建的分区号，因为我们是创建第一个分区，所以用默认值就行，直接按回车，按回车后显示如下：

	```
	Select (default p): p
	Partition number (1-4, default 1):
	First sector (2048-10485759, default 2048):
	```



* 输入分区的开始位置所在的扇区号。因为分区表需要占一定的空间，另外分区开始的位置需要对齐到一定的字	节，而默认值 2048 把这些都考虑进去了，所以用默认值即可，直接按回车，按回车后显示如下：

	```
	First sector (2048-10485759, default 2048):
	Using default value 2048
	Last sector, +sectors or +size{K,M,G} (2048-10485759, default 10485759):
	```


* 输入分区的结束位置所在的扇区号。因为我们只创建一个分区，所以让分区占满整个磁盘空间，分区结束需设在磁盘最后一个扇区，继续用默认值，直接按回车，按回车后显示如下:

	```
	Last sector, +sectors or +size{K,M,G} (2048-10485759, default 10485759):
	Using default value 10485759
	Partition 1 of type Linux and of size 5 GiB is set
	
	Command (m for help):
	```


* 输入 wq，把分区表写入磁盘，并退出。
	
	```
	Command (m for help): wq
	The partition table has been altered!
	
	Calling ioctl() to re-read partition table.
	Syncing disks.
	```
 
 * lsblk 确认分区成功。
​
​
	
	```
	[dc2-user@ip ~]$ lsblk
	NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
	sr0 11:0 1 1024M 0 rom
	vda 253:0 0 20G 0 disk
	└─vda1 253:1 0 20G 0 part /
	vdb 253:16 0 5G 0 disk
	└─vdb1 253:17 0 5G 0 part
	```

* 如果输出中包含 vdb1，说明分区成功。

* 用下面命令在新分区上创建一个 xfs 文件系统。根据您的需求，可把下面命令中 xfs 换成 ext4或ext3 以创建 ext4 或 ext3 文件系统。如果磁盘容量较大，创建文件系统可能需要较长时间，请耐心等待。

	```
	dc2-user@ip:~$ sudo mkfs.xfs /dev/vdb1
	meta-data=/dev/vdb1 isize=512 agcount=4, agsize=327615 blks
	= sectsz=512 attr=2, projid32bit=1
	= crc=1 finobt=1, sparse=0
	data = bsize=4096 blocks=1310459, imaxpct=25
	= sunit=0 swidth=0 blks
	naming =version 2 bsize=4096 ascii-ci=0 ftype=1
	log =internal log bsize=4096 blocks=2560, version=2
	= sectsz=512 sunit=0 blks, lazy-count=1
	realtime =none extsz=4096 blocks=0, rtextents=0
	```



	将磁盘挂载到 linux 某个目录下，以使得它可以访问。

* 创建一个磁盘需要挂载的目录，我们假设挂载到 /data 下。

	```
	dc2-user@ip:~$ sudo mkdir /data
	```



* 查看磁盘分区的 UUID，以备挂载时使用。

	```
	dc2-user@ip:~$ ls -al /dev/disk/by-uuid | grep vdb1
	lrwxrwxrwx 1 root root  10 Feb 22 17:52 6f24c2f5-b646-4e4e-a0dc-63b72929453e -> ../../vdb1
	```

	这里 `6f24c2f5-b646-4e4e-a0dc-63b72929453e` 就是 vdb1 分区的 UUID。



* 备份 fstab 文件。

	```
	dc2-user@10-254-99-45:~$ sudo cp /etc/fstab /etc/fstab.orig
	```



* 编辑 fstab 文件，添加挂载项。
	
	```
	dc2-user@10-254-99-45:~$ sudo vim /etc/fstab
	dc2-user@10-254-99-45:~$ cat /etc/fstab
	LABEL=cloudimg-rootfs / ext4 defaults 0 0
	UUID=6f24c2f5-b646-4e4e-a0dc-63b72929453e /data xfs defaults 0 0
	```

	其中 `UUID` 那一行是这次操作添加的，这里 `6f24c2f5-b646-4e4e-	a0dc-63b72929453e` 就是在 b 步骤得到的 vdb1 的分区 UUID。



* 运行下列命令使 fstab 文件生效。

	```
	dc2-user@ip:~$ sudo mount -a
	dc2-user@ip:~$
	```

	没有任何输出代表挂载成功，如果报错，则需调查原因并修正 fstab 文件，	如果无法确定错误原因，您总是可以用以下命令回滚 fstab 文件。

	```
	dc2-user@ip:~$ sudo mv /etc/fstab.orig /etc/fstab
	```



* 修改挂载目录的拥有者以便他人可以访问。

	```
	dc2-user@ip:~$ sudo chown dc2-user /data
	```
	
**结果：**然后就可以在 /data 目录下创建文件，读写文件了。
