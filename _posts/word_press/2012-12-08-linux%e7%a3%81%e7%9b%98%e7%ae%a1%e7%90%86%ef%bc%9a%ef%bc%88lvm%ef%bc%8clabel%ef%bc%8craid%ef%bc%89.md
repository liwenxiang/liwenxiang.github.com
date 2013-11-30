---
title: linux磁盘管理：（lvm，label，raid）
author: codeboy
layout: post
permalink: /2012/12/08/linux%e7%a3%81%e7%9b%98%e7%ae%a1%e7%90%86%ef%bc%9a%ef%bc%88lvm%ef%bc%8clabel%ef%bc%8craid%ef%bc%89/
categories:
  - ubuntu配置
tags:
  - fstab
  - lvm
  - raid
  - 磁盘管理
---
周末没事看了下lvm的东西，在自己虚拟机上面实验了下，记录一下磁盘管理的东西

一、 lvm

把下面的物理磁盘虚拟成一个大的虚拟磁盘，可以做到动态的扩大磁盘空间而不需要重启。<!--more-->

[http://easwy.com/blog/archives/linux-lvm-learning-notes/][1]

[http://www.ibm.com/developerworks/cn/linux/filesystem/lvm/lvm-1/index.html][2]

&nbsp;

[<img style="background-image: none; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border: 0px;" title="vg" src="http://www.codeboy.name/wp-content/uploads/2012/12/vg_thumb.png" alt="vg" width="705" height="270" border="0" />][3]

新加的一块磁盘  
/dev/sdb  
1,对磁盘分区  
cfdisk /dev/sdb  
new  type里面选择8E  
2,创建pv  
pvcreate /dev/sdb  
3,创建vg  
vgcreate main /dev/sdb  
4,在vg上面创建逻辑卷  
lvcreate -L8G -n v_home main  
现在应该有了/dev/main/v_home了  
5，在/dev/main/v_home创建文件系统  
mkfs -t ext4 /dev/main/v_home  
6,挂载  
mkdir /data/  
mount /dev/main/v_home  /data/  
7, 编辑/etc/fstab 以使它包括新的 /home 项  
添加  
/dev/main/v_home   /data   ext4     defaults      2 2

完成  
重启机器，  
df -h  
/dev/main/v_home  2G &#8230; /home  
现在感觉/data/目录不够了，要增加1G。  
lvextend -L+1G /dev/main/lv_home  
resize2fs -f /dev/main/lv_home  
不需要重启机器  
df -h  
/dev/main/v_home   3G    &#8230; /home

&nbsp;

现在一块硬盘容量不够，再来一块硬盘；  
/dev/sdc  
分区  
cfdisk /dev/sdc  
pvcreate /dev/sdc  
把这块硬盘加到main的vg里面  
vgextend main /dev/sdc  
这样main的容量就是/dev/sdb和/dev/sdc的容量总和  
可用vgdisplay看下，容量已经加上去了

用的时间长了，想要把sdb磁盘卸载不用了  
把sdb移除之前，首先把上面的数据拷贝到其他磁盘上面，使用下面命令，数据会自动拷贝到另外的pv里面  
pvmove /dev/sdb  
如果要把sdb的数据指定移动到sdf中（比如sdf是新磁盘）  
pvmove /dev/sdb /dev/sdf  
从vg组里面移除这个pv  
vgreduce main /dev/sdb  
Or remove all empty physical volumes:  
移除这个vg里面的所有空pv  
vgreduce &#8211;all vg0  
最后删除这个pv  
pvremove /dev/sdb1

&nbsp;

二、 /etc/fstab文件中LABEL的用法

主要解决的是物理磁盘失效了，重新用一块新的磁盘挂上，使得/home/admin文件目录和/dev/sda1具体磁盘的直接关系脱离开

df -h是可用的磁盘，  
/dev下面是已经挂载的物理磁盘

假设之前/dev/sda1声明的LABEL为var

blkid显示

/dev/sda1: LABEL=”var”

/etc/fstab文件内容如下

LABEL=/home /home ext4 defaults 1 2  
LABEL=/var /var ext4 defaults 1 2

现在sda盘坏掉了，要换成另外一块盘sdb,

分区cfdisk /dev/sdb  
创建文件系统mkfs -t ext4 /dev/sdb1  
和标签关联起来e2label /dev/sdb1 var

mount –a

这样sdb盘就成了新的var目录，  
LABEL=/home           /home                 ext4    defaults        1 2  
LABEL=/var      /var            ext4    defaults        1 2

&nbsp;

三、使用raid

[http://www.bitscn.com/plus/view.php?aid=6926][4]

由于本文中会使用mdadm软件，而该软件一般情况下都会集成在Redhat linux中，所以可以直接使用。如果系统中没有安装可以到http://www.cse.unsw.edu.au/~neilb/source/mdadm来下载mdadm-1.8.1.tgz进行编译安装，也可以到http://www.cse.unsw.edu.au/~neilb/source/mdadm/rpm下载mdadm-1.8.1-1.i386.rpm直接安装。

第一步：以root用户登录系统，对磁盘进行分区。

#fdisk /dev/sdb

将设备/dev/sdb上的全部磁盘空间划分给一个主分区，建立/dev/sdb1分区，并修改分区的类型标识为fd（linux raid auto）,然后对剩余的磁盘做同样的操作。创/dev/sdb1,/dev/sdc1,/dev/sdd1三个分区。

第二步：创建RAID阵列

#madam -cv /dev/md0 -l1 -n2 -x1 /dev/sd{b,c,d}1

小提示：-C参数为创建阵列模式。/dev/md0为阵列的设备名称。-l1为阵列模式，可以选择0，1，4，5等多种不同的阵列模式，分别对应RAID0，RAID1，RAID4，RAID5。-n2为阵列中活动磁盘的数目，该数目加上备用磁盘的数目应该等于阵列中总的磁盘数目。-x1为阵列中备用磁盘的数目，因为我们是RAID1所以设置当前阵列中含有一块备用磁盘。/dev/sd{b,c,d}1为参与创建阵列的磁盘名称，阵列由三块磁盘组成，其中两块为镜象的活动磁盘，一块备用磁盘提供故障后的替换。

第三步：查看RAID阵列情况

创建RAID过程需要很长时间，因为磁盘要进行同步化操作，查看/proc/mdstat文件，该文件显示RAID的当前状态和同步完成所需要的时间。

#cat /proc/mdstat

系统会显示——

personalities:[raid1]  
read_ahead 1024 sectors  
event:1  
md0:active raid1 sdb1[0] sdc1[1] sdd1[2]  
18432000 blocks \[2/2\] \[UU\]  
unused devices:

出现上面的提示后就表示创建的RAID1已经可以使用了。

第四步：编辑阵列的配置文件

mdadm的配置文件主要提供人们日常管理，编辑这个文件可以让RAID更好的为我们工作，当然这个步骤不是必须的。不经过编辑配置文件也可以让RAID工作。

首先扫描系统中的全部阵列

#mdadm -detail -scan

扫描结果将显示阵列的名称，模式和磁盘名称，并且列出阵列的UUID号，UUID也同时存在于阵列的每个磁盘中，缺少该号码的磁盘是不能够参与阵列的组成的。

接下来编辑阵列的配置文件/etc/mdadm.conf文件，将扫描的显示结果按照文件规定的格式修改后添加到文件的末尾。

#vi /etc/mdadm.conf

添加以下内容到mdadm.conf文件中

device /dev/sdb1 /dev/sdc1 /dev/sdd1  
array /dev/md0 level=raid1 num-devices=2 uuid=2ed2ba37:d952280c:a5a9c282:a51b48da spare-group=group1

在配置文件中定义了阵列的名称和模式，还有阵列中活动磁盘的数目与名称，另外也定义了一个备用的磁盘组group1。

第五步：启动停止RAID1阵列

启动和停止RAID1阵列的命令非常简单。启动直接执行“mdadm -as /dev/md0”即可。执行mdadm -s /dev/md0将停止RAID1阵列。另外在rc.sysinit启动脚本文件中加入命令mdadm -as /dev/md0后将设置为阵列随系统启动而启动。

总结：配置RAID1的步骤相对RAID5来说不是很烦琐，不过在使用mdadm时应该注意就是不要在一块硬盘上划分多个分区，再将多个分区组成阵列，这种方式不但不能提高硬盘的访问速度，反而会降低整体系统的性能。正确的方法是将一块硬盘分成一个或多个分区，然后将多块不同硬盘的分区组成阵列。另外系统目录如/usr最好不要放在阵列中，因为一旦阵列出现问题系统将无法正常运行。

 [1]: http://easwy.com/blog/archives/linux-lvm-learning-notes/ "http://easwy.com/blog/archives/linux-lvm-learning-notes/"
 [2]: http://www.ibm.com/developerworks/cn/linux/filesystem/lvm/lvm-1/index.html "http://www.ibm.com/developerworks/cn/linux/filesystem/lvm/lvm-1/index.html"
 [3]: http://www.codeboy.name/wp-content/uploads/2012/12/vg.png
 [4]: http://www.bitscn.com/plus/view.php?aid=6926 "http://www.bitscn.com/plus/view.php?aid=6926"