# Linux-raid-
一、Raid定义

RAID,全称Redundant Array ofInexpensive Disks,中文名为廉价磁盘冗余阵列。RAID可分为软RAID和硬RAID,软RAID是通过软件实现多块硬盘冗余的。而硬RAID是一般通过RAID卡来实现RAID的。前者配置简单，管理也比较灵活。对于中小企业来说不失为一最佳选择。硬RAID往往花费比较贵。不过，在性能方面具有一定优势。

RAID分类

RAID可分为以下几种，做个表格认识下：

RAID 0 ：存取速度最快 没有容错

RAID 1 ：完全容错 成本高，硬盘使用率低。

RAID 3 ：写入性能最好 没有多任务功能

RAID 5 ：具备多任务及容错功能 写入时有overhead

RAID 0+1 ：速度快、完全容错 成本高

 

二、Linux RAID 5 实验详解

（注：以下linux命令中如果提示权限不够，请在命令前加sudo）



1、添加4块硬盘

可以用虚拟机设置出4块硬盘出来。在虚拟机上添加硬盘，一直采用默认设置。

添加后，linux重启才能识别

可以用fdisk –l命令查看到

添加的四块硬盘分别为

/dev/sdb

/dev/sdc

/dev/sdd

/dev/sde。



2、分区

比如对/dev/sdb分区

fdisk /dev/sdb

Device contains neither a valid DOS partition table, norSun, SGI or OSF disklabel

Building a new DOS disklabel. Changes will remain in memoryonly,

until you decide to write them. After that, of course, theprevious

content won’t be recoverable.

Warning: invalid flag 0×0000 of partition table 4 will becorrected by w（rite）

Command （m for help）： n #按n创建新分区

Command action

e extended

p primary partition （1-4） #输入p 选择创建主分区

p

Partition number （1-4）： 1 #输入 1 创建第一个主分区

First cylinder （1-130, default 1）： #直接回车，选择分区开始柱面这里就从 1 开始

Using default value 1

Last cylinder or +size or +sizeM or +sizeK （1-102, default 130）：

Using default value 130

Command （m for help）： w #然后输入w写盘

The partition table has been altered!

Calling ioctl（） to re-readpartition table.

Syncing disks.

其它分区照这样做全部分出一个区出来。下面命令显示所有分区信息：

 fdisk –l

会看到出现/dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1

3、创建RAID

安装mdadm软件

sudo apt install mdadm

 mdadm –create /dev/md0 –level=5 -n3 /dev/sdb1 /dev/sdc1 /dev/sdd1

#意思是创建RAID设备名为md0, 级别为RAID 5,采用了三块磁盘，分别是/dev/sdb1 /dev/sdc1 /dev/sdd1

mdadm: array /dev/md0 started.

OK,初步建立了RAID了，我们看下具体情况吧。

mdadm –detail /dev/md0

4 RAID的启动方法（这一步在实验时可以跳过）

   RAID的启动有两种方法，一种是指定RAID设备和RAID成员的办法来启动RAID，另一种办法是通过加载RAID默认的配置文件来启动。

第一种方法：   

语法：

   mdadm -A RAID设备   RAID成员

  注：

   -A 同 –assemble ，意思是激活一个已经存在的RAID；

   RAID设备 ，就是/dev/md0 或/dev/md1 …… 根据你所创建的RAID设备为准；

   RAID成员，就是你要启动的RAID，其下属设备有哪些，要一个一个的列出来，中间以空格分开；

  举例：

比如我要启动刚才建立的RAID，设备是/dev/md0，其下有成员是 /dev/sdb1和/dev/sdc1和/dev/sde1；所以我要用下面的办法；

    mdadm  -A  /dev/md0  /dev/sdb1 /dev/sdc1 /dev/sde1



第二种方法：

为了让RAID开机启动，需要编辑RIAD配置文件。默认名字为mdadm.conf,这个文件默认是不存在的，要自己建立。该配置文件存在的主要作用是系统启动的时候能够自动加载软RAID，同时也方便日后管理。

mdadm –detail –scan >/etc/mdadm.conf

[root@localhost ~]# cat /etc/mdadm.conf

ARRAY /dev/md0 level=raid5 num-devices=3 UUID=e62a8ca6:2033f8a1:f333e527:78b0278a

devices=/dev/sdb1,/dev/sdc1,/dev/sdd1

#默认格式是不正确的，需要做以下方式的修改：

vi /etc/mdadm.conf

cat /etc/mdadm.conf

devices /dev/sda1,/dev/sdb1,/dev/sdc1,/dev/sdd1

ARRAY /dev/md0 level=raid5 num-devices=3 UUID=e62a8ca6:2033f8a1:f333e527:78b0278a

5、将/dev/md0创建文件系统

mkfs -t ext4 /dev/md0

6、挂载/dev/md0到系统中去，我们实验是否可用：

mkdir /mdadm

mount /dev/md0 /mdadm/

cd /mdadm/

ls

cd /etc/services .

ls

7、如果我要移除一块坏的硬盘或添加一块硬盘呢？

#删除一块硬盘

mdadm /dev/md0 -f/dev/sdc1

 

mdadm /dev/md0–remove /dev/sdc1

mdadm: hot removed /dev/sdc1

cat /proc/mdstat

Personalities : [raid5]

md0 : active raid5 sdd1[2] sdb1[1] sda1[0]

2088192 blocks level 5, 64k chunk, algorithm 2 [3/3] [UUU]

unused devices: <none>

#增加一块硬盘

mdadm /dev/md0 –add /dev/sdc1

mdadm: hot added /dev/sdc1

cat /proc/mdstat

Personalities : [raid5]

md0 : active raid5 sdc1[3] sdd1[2] sdb1[1] sda1[0]
