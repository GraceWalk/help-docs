##扩容概述

随着业务增长，您可能需要扩容（即增加容量）云盘以存放更多的数据，可以顺序执行以下两个步骤完成扩容（注：如果云盘里已经有数据，请先打个快照备份数据，以防止因误操作而导致已有的数据损坏）。

##在控制台上操作扩容云盘。

1. 登录 控制台。

2. 在页面顶部导航栏点击存储，并在顶部下面的二级标签选择“云盘（EBS）”，进入我的云盘列表。

3. 选择一个云盘并点击 “...” 选择 "扩容" 。

4. 在弹出的扩容框中选择扩容到的容量，点击确认。

5. 在控制台上扩容完毕后，分以下情况：

* 如果云盘已经挂载到 DC2 实例，则需要登录实例完成扩容。

* 如果云盘还未挂载到 DC2 实例，则需先挂载云盘，然后根据不同情况操作：

*  全新云盘，并未格式化过，则可经过[格式化云盘]()后直接使用。

* 云盘之前格式化过，则需要登录实例完成扩容。



##登录DC2实例完成扩容

本步骤以具有一个分区 vdb1 的云盘 vdb 的 centos -7.3 DC2 实例为例介绍实例内扩容步骤。

1. 登录 DC2。
2. 使用 lsblk 查找需要扩容的云盘。

	```
	dc2-user@10-254-99-45:~$ lsblk
	NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
	sr0 11:0 1 1024M 0 rom
	vda 253:0 0 20G 0 disk
	└─vda1 253:1 0 20G 0 part /
	vdb 253:16 0 50G 0 disk
	└─vdb1 253:17 0 5G 0 part /data
	```

	vdb 是我们需要扩容的云盘，并已经在控制台做过扩容操作。

3. 卸载 vdb1。

	```
	dc2-user@ip:~$ sudo umount /data
	```

	没有任何输出表示卸载成功

4. 记录分区的 uuid 以便之后步骤中使用。

	```
	dc2-user@ip:~$ ls -al /dev/disk/by-uuid/ | grep vdb1
	lrwxrwxrwx 1 root root  10 Feb 22 17:52 6f24c2f5-b646-4e4e-	a0dc-63b72929453e -> ../../vdb1
	```

5. 使用 fdisk 扩容 vdb1 分区。

	* 运行 fdisk 进入磁盘分区。

		```
		dc2-user@ip:~$ sudo fdisk /dev/vdb
		Welcome to fdisk (util-linux 2.23.2).

		Changes will remain in memory only, until you decide to write them.
		Be careful before using the write command.

		Command (m for help):
```
	* 输入 p，打印分区表。

		```
		Command (? for help): p

		Disk /dev/vdb: 53.7 GB, 53687091200 bytes, 104857600 sectors
		Units = sectors of 1 * 512 = 512 bytes
		Sector size (logical/physical): 512 bytes / 512 bytes
		I/O size (minimum/optimal): 512 bytes / 512 bytes
		Disk label type: dos
		Disk identifier: 0x1e7b3827

   		Device Boot      Start         End      Blocks   Id  System
   		/dev/vdb1            2048    10485759     5241856   83  Linux
		```

		把打印的分区表内容复制下来保存，之后会用到，由分区表可以看出，磁盘大小为 		53.7 GB，说明控制台的操作有效果，但是分区 1 只有 5GB 大小，因此，我们接下		来要先扩展分区 1 的大小。
		
	* 输入 d 并按回车，删除原分区，删除分区不会造成分区内数据丢失。

		```
		Command (m for help): d
		Selected partition 1
		Partition 1 is deleted

		Command (m for help):
		```

	* 输入 n 并按回车，创建新的分区。

	* 如果之前只有 1 个分区，如本例，只需在选择分区开始扇区，结束扇区时使用默认值，就可以扩展分区 1 的大小。

		```
		Command (? for help): n
		Partition type:
   		p   primary (0 primary, 0 extended, 4 free)
   		e   extendedSelect (default p):
		Using default response p
		Partition number (1-4, default 1):First sector (2048-104857599, 		default 2048):
		Using default value 2048
		Last sector, +sectors or +size{K,M,G} (2048-104857599, default 		104857599):
		Using default value 104857599
		Partition 1 of type Linux and of size 50 GiB is set
		Command (m for help):
		```

	* 如果之前有多个分区，则在每一步，输入您在 b 步骤记录的分区表对应的值，如各分区的起始扇区，结束扇区，对于最后一个分区的结束扇区，使用默认值，以扩展最后一个分区的大小。

		```
		Command (m for help): n
		Partition type:
   		p   primary (0 primary, 0 extended, 4 free)
   		e   extendedSelect (default p):
		Using default response p
		Partition number (1-4, default 1):First sector (2048-104857599, default 		2048): 2048
		Last sector, +sectors or +size{K,M,G} (2048-104857599, default 		104857599): 8191
		Partition 1 of type Linux and of size 3 MiB is set

		Command (m for help): n
		Partition type:
   		p   primary (1 primary, 0 extended, 3 free)
   		e   extended
		Select (default p):Using default response p
		Partition number (2-4, default 2):First sector (8192-104857599, default 		8192): 8192
		Last sector, +sectors or +size{K,M,G} (8192-104857599, default 		104857599):
		Using default value 104857599
		Partition 2 of type Linux and of size 50 GiB is set
		Command (m for help):
		```

		这里，假设之前有两个分区，第一个分区是起始扇区 2048，结束扇区 8191。第二个分区结束扇		区使用默认值 104857566(104857566*512 bytes=50GB)，以扩展其大小。


	* 输入 wq，保存修改结果并退出。

		```
		Command (m for help): wq
		The partition table has been altered!
		Calling ioctl() to re-read partition table.
		Syncing disks.
		```


	检查文件系统是否有报错。

	* 查看文件系统类型，根据文件系统类型，执行 b 或 c 操作。

		```
		dc2-user@ip:~$ sudo file -sL /dev/vdb1
		/dev/vdb1: SGI XFS filesystem data (blksz 4096, inosz 512, v2 dirs)
		```

		由输出可见，文件系统类型为 xfs。

	* xfs 文件系统。

		```
		dc2-user@ip:~$ sudo xfs_repair /dev/vdb1
		Phase 1 - find and verify superblock...
		Phase 2 - using internal log
		- zero log...
		- scan filesystem freespace and inode maps...
		- found root inode chunk
		Phase 3 - for each AG...
		- scan and clear agi unlinked lists...
		- process known inodes and perform inode discovery...
		- agno = 0
		- agno = 1
		- agno = 2
		- agno = 3
		- process newly discovered inodes...
		Phase 4 - check for duplicate blocks...
		- setting up duplicate extent list...
		- check for inodes claiming duplicate blocks...
		- agno = 0
		- agno = 1
		- agno = 2
		- agno = 3
		Phase 5 - rebuild AG headers and trees...
		- reset superblock...
		Phase 6 - check inode connectivity...
		- resetting contents of realtime bitmap and summary inodes
		- traversing filesystem ...
		- traversal finished ...
		- moving disconnected inodes to lost+found ...
		Phase 7 - verify and correct link counts...
		done
		```

		注：如果系统中没有 `xfs_repair` 命令，则需要安装 `xfsprogs` 包。

	* ext3 或 ext4 文件系统。

		```
		dc2-user@ip:~$ sudo e2fsck -f /dev/vdc
		e2fsck 1.42.13 (17-May-2015)
		Pass 1: Checking inodes, blocks, and sizes
		Pass 2: Checking directory structure
		Pass 3: Checking directory connectivity
		Pass 4: Checking reference counts
		Pass 5: Checking group summary information
		/dev/vdc: 12/327680 files (0.0% non-contiguous), 55903/1310720 blocks
		```



	* 挂载分区。

		*  如果磁盘之前已经照`格式化云盘`操作，曾经修改 fstab 配置过 vdb1 的挂载，则只需如下操作：**

		```
		dc2-user@ip:~$ sudo mount -a
		```
		* 否则，照着 `格式化云盘` 第 5 步，修改 fstab 以将 vdb1 挂载到系统中。


	* 根据磁盘分区上的文件系统不同，使用不同的工具扩容文件系统。

		* 查看文件系统类型。

		```
		dc2-user@ip:~$ sudo file -sL /dev/vdb1
		/dev/vdb1: SGI XFS filesystem data (blksz 4096, inosz 512, v2 dirs)
		```

		由输出可见，文件系统类型为 xfs。

		* 根据文件系统类型选择以下步骤：

		* xfs 文件系统。

		```
		dc2-user@ip:~$ df -h
		Filesystem Size Used Avail Use% Mounted on
		udev 488M 0 488M 0% /dev
		tmpfs 100M 5.7M 94M 6% /run
		/dev/vda1 20G 1.2G 19G 6% /
		tmpfs 497M 0 497M 0% /dev/shm
		tmpfs 5.0M 0 5.0M 0% /run/lock
		tmpfs 497M 0 497M 0% /sys/fs/cgroup
		tmpfs 100M 0 100M 0% /run/user/1001
		/dev/vdb1 5.0G 33M 5.0G 1% /data

		dc2-user@ip:~$ sudo xfs_growfs -d /data
		meta-data=/dev/vdb1 isize=512 agcount=4, agsize=327615 blks
		= sectsz=512 attr=2, projid32bit=1
		= crc=1 finobt=1 spinodes=0
		data = bsize=4096 blocks=1310459, imaxpct=25
		= sunit=0 swidth=0 blks
		naming =version 2 bsize=4096 ascii-ci=0 ftype=1
		log =internal bsize=4096 blocks=2560, version=2
		= sectsz=512 sunit=0 blks, lazy-count=1
		realtime =none extsz=4096 blocks=0, rtextents=0
		data blocks changed from 1310459 to 13106939
		```

		* ext3 或 ext4。

		```
		dc2-user@ip:~$ sudo resize2fs /dev/vdc
		resize2fs 1.42.13 (17-May-2015)
		Resizing the filesystem on /dev/vdc to 13107200 (4k) blocks.
		The filesystem on /dev/vdc is now 13107200 (4k) blocks long.
		```



		* 运行 `df -h` 确认扩容成功。
		
		```
		dc2-user@ip:~$ df -h
		Filesystem Size Used Avail Use% Mounted on
		udev 488M 0 488M 0% /dev
		tmpfs 100M 5.7M 94M 6% /run
		/dev/vda1 20G 1.2G 19G 6% /
		tmpfs 497M 0 497M 0% /dev/shm
		tmpfs 5.0M 0 5.0M 0% /run/lock
		tmpfs 497M 0 497M 0% /sys/fs/cgroup
		tmpfs 100M 0 100M 0% /run/user/1001
		/dev/vdb1 50G 34M 50G 1% /data
		/dev/vdc 50G 17M 47G 1% /mnt
		```
		
		其中 vdb 是 xfs，vdc 是 ext4。

