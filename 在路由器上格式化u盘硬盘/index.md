# 在路由器上格式化U盘硬盘



## 引言

本教程适用于梅林、padavan、LEDE（openwrt）等固件

以下具体方法都基于 ext4，NTFS 相关错误不做回答

使用ssh连接路由器，把U盘插到路由器上

我们需要在命令行进行以下4步操作：

<!--more-->

## 安装fdisk

一般梅林、Padavan 固件都会自带的，不用安装，如果没有则按照下面给出的命令
```
opkg update
opkg install fdisk
# 输出Configuring fdisk. 并且没有错误
# fdisk就安装好了
```


## 查看你的设备
```
fdisk -l 
# 这里先输出系统分区之类的不用管，外置设备一般在最后
Disk /dev/sda: 30.7 GB, 30752000000 bytes
64 heads, 32 sectors/track, 29327 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytes
Device Boot      Start         End      Blocks  Id System
/dev/sda1               2       29327    30029824  83 Linux
```
上面的信息注意看到和你的存储大小一样的设备，我的是 `/dev/sda`，在它里面有个 `/dev/sda1`的分区

##  删除分区、新建分区

先卸载 U 盘，如果提示 `No such file or directory` 没关系，说明本来就没挂载上
```
# padavan、梅林可以执行以下这个推出 usb
ejusb

# 其他固件，或者梅林使用以上命令无效，则可以使用这个命令卸载分区
umount /dev/sda1
```

然后分区
```
fdisk /dev/sda # 这是你的设备別打成分区

Welcome to fdisk (util-linux 2.29.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): d 
# 输入d回车，我只有一个分区，它自动选择了，如果你有多个分区，可以多次使用d
Selected partition 1
Partition 1 has been deleted.

Command (m for help): n # 输入n会车，创建分区
Partition type
p   primary (0 primary, 0 extended, 4 free)
e   extended (container for logical partitions)

Select (default p): p # 选择p
Partition number (1-4, default 1): # 输入 1 回车
First sector (2048-2065023, default 2048): # 回车
Last sector, +sectors or +size{K,M,G,T,P} (2048-2065023, default 2065023): # 回车
Created a new partition 1 of type 'Linux' and of size 1007.3 MiB.

Command (m for help): w # 输入 w 回车，保存并退出
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```
经过以上的操作，你可以用 `fdisk -l` 命令查看U盘上是否只有一个 Linux 分区
```
fdisk -l 
# 找到你的设备 可以看到ID为83就对了
Disk /dev/sda: 30.7 GB, 30752000000 bytes
64 heads, 32 sectors/track, 29327 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytes
Device Boot      Start         End      Blocks  Id System
/dev/sda1               2       29327    30029824  83 Linux
```

## 格式化分区

分区已经有了，现在开始格式化

用 `mkfs.ext4` 命令格式化，并且设置卷标为 onmp

**注意**: 如果下面的命令提示 `/dev/sda1 is mounted`，则需要先卸载 U 盘，和分区前卸载的方法一样
```
mkfs.ext4 -m 0 -L onmp /dev/sda1 
# 如果你的硬盘比较大，256G以上的话，是下面这个命令：
# mkfs.ext4 -m 0 -L ONMP -T largefile /dev/sda1

mke2fs 1.42.8 (20-Jun-2013)
Filesystem label=onmp
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
1880480 inodes, 7507456 blocks
0 blocks (0.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=0
230 block groups
32768 blocks per group, 32768 fragments per group
8176 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

这样，U盘就被格式化完了，拔插 U 盘可以重新挂载，或者你想用以下命令挂载也行
```
mkdir /mnt/onmp
mount -t ext4 /dev/sda1 /mnt/onmp
```

> 本文章发表于底噪博客 [https://zhih.me](https://zhih.me/) , 转载请注明




