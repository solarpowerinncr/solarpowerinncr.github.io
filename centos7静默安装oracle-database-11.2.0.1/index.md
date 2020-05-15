# Centos7静默安装oracle Database 11.2.0


<!--more-->



### 前言

oracle安装方式：GUI，silent mode，response mod
silent mode：把所有交互参数保存在response file里，启动Oracle Universal Installer时加 -silent 
使用response文件夹里的文件定义安装需要的变量
`db_install.rsp`：对应Oracle Configuration Manager registration
`dbca.rsp`：对应Oracle database configuration assistant
`netca.rsp`：对应Oracle net configuration assistant

静默安装ASM:
在创建使用Oracle ASM的数据库之前，必须运行root.sh脚本。因此，在静默方式安装期间，不能使用Oracle ASM作为数据库文件的存储选项来创建数据库。相反，您可以使用静默模式完成纯软件安装，然后在完成纯软件安装并运行root.sh脚本后，以静默模式运行Oracle Net Configuration Assistant和Database Configuration Assistant。

### 安装前准备工作

用root用户登录

#### 硬件需求

**内存：**
最小 1 GB，推荐 2 GB 以上
查看内存大小：`grep MemTotal /proc/meminfo`

**交换分区：**

> Note：
> 在Linux上，HugePages功能使用内存映射文件为大型页表分配不可交换的内存。如果启用了HugePages，则应在计算交换空间之前从可用的RAM中扣除分配给HugePages的内存。

| 可用内存   | 交换分区      |
| ---------- | ------------- |
| 1 GB~2 GB  | 1.5倍内存大小 |
| 2 GB~16 GB | 等同内存大小  |
| 超过16GB   | 16GB          |

查看交换分区大小：`grep SwapTotal /proc/meminfo`
查看内存和交换分区：`free`

**ASM:**
从Oracle Database 11g开始，自动内存管理功能需要更多的共享内存（`/dev/shm`）和文件描述符。共享内存的大小至少应大于该计算机上每个Oracle实例的`MEMORY_MAX_TARGET`和`MEMORY_TARGET`。
查看共享内存大小：`df -h /dev/shm/`

可以通过如下命令修改shm的大小：
`mount -t tmpfs shmfs -o size=8g /dev/shm`
为了系统重启也可以生效，需要修改`/etc/fstab`文件，添加如下内容：
`shmfs /dev/shm tmpfs size=8g 0 0`


> Note:
> 当启用`LOCK_SGA`或在Linux上使用HugePages时，不能使用`MEMORY_MAX_TARGET`和`MEMORY_TARGET`。

**硬盘：**
一、
 至少1GB可用空间，挂载在`/tmp`
查看`/tmp`大小：`df -h /tmp`

可用空间不够时，两种办法：

1. 删除`/tmp`下不必要的文件
2. 配置`oracle`用户的环境时，设置`TMP`和`TMPDIR`环境变量。
   1）创建一个临时目录，并在该目录上设置适当的权限：

```
$ sudo mkdir /mount_point/tmp
$ sudo chmod a+wr /mount_point/tmp
# exit
```

2）设置`TMP`和`TMPDIR`环境变量：

```
$ TMP=/mount_point/tmp
$ TMPDIR=/mount_point/tmp
$ export TMP TMPDIR
```

二、
Linux x86-64上安装所需的Software Files和Data Files的大小

| 安装类型           | Software Files要求（GB） |
| ------------------ | ------------------------ |
| Enterprise Edition | 4.7                      |
| Standard Edition   | 4.6                      |

| 安装类型           | Data Files要求（GB） |
| ------------------ | -------------------- |
| Enterprise Edition | 1.7                  |
| Standard Edition   | 1.5                  |

如果选择配置自动备份，则无论是安装在file system或OracleASM磁盘组上，快速恢复区域需要额外的磁盘空间。

#### 软件需求

**操作系统：**
`cat /proc/version`
Oracle Linux 7
Red Hat Enterprise Linux 7

从Oracle Database 11g Release 2 (11.2)开始，Oracle Linux 4，Oracle Linux 5，Oracle Linux 6，Red Hat Enterprise Linux 4, Red Hat Enterprise Linux 5, and Red Hat Enterprise Linux 6支持 SE Linux 功能。

**内核:**
3.10.0-54.0.1.el7.x86_64 or later

**软件包：**

> Note：
> 从Oracle Database 11g Release 2 (11.2.0.2)开始，下表中列出的除`gcc-32bit-4.3`之外的所有32位软件包都不再需要在Linux x86-64上安装数据库，仅需要64位软件包。但是，对于11.2.0.2之前的任何Oracle Database 11g发行版，下表中列出的32位和64位软件包都是必需的。

对 Oracle Linux 7 和 Red Hat Enterprise Linux 7，以下软件包必须安装：

```bash
binutils-2.23.52.0.1-12.el7.x86_64 
compat-libcap1-1.10-3.el7.x86_64 
#compat-libstdc++-33-3.2.3-71.el7.i686
compat-libstdc++-33-3.2.3-71.el7.x86_64
gcc-4.8.2-3.el7.x86_64 
gcc-c++-4.8.2-3.el7.x86_64 
#glibc-2.17-36.el7.i686 
glibc-2.17-36.el7.x86_64 
#glibc-devel-2.17-36.el7.i686 
glibc-devel-2.17-36.el7.x86_64 
ksh
#libaio-0.3.109-9.el7.i686 
libaio-0.3.109-9.el7.x86_64 
#libaio-devel-0.3.109-9.el7.i686 
libaio-devel-0.3.109-9.el7.x86_64 
#libgcc-4.8.2-3.el7.i686 
libgcc-4.8.2-3.el7.x86_64 
#libstdc++-4.8.2-3.el7.i686 
libstdc++-4.8.2-3.el7.x86_64 
#libstdc++-devel-4.8.2-3.el7.i686 
libstdc++-devel-4.8.2-3.el7.x86_64 
#libXi-1.7.2-1.el7.i686 
libXi-1.7.2-1.el7.x86_64 
#libXtst-1.2.2-1.el7.i686 
libXtst-1.2.2-1.el7.x86_64 
make-3.82-19.el7.x86_64 
sysstat-10.1.5-1.el7.x86_64
```

可能需要安装：

```bash
#Oracle ODBC Drivers
unixODBC-2.3.1-6.el7.x86_64 or later
unixODBC-2.3.1-6.el7.i686 or later
unixODBC-devel-2.3.1-6.el7.x86_64 or later
unixODBC-devel-2.3.1-6.el7.i686 or later
#X11其他扩展库
libXext
#mksh是一个用于交互式和shell脚本的命令解释器
#它的命令语言是一种超集sh(C)shell语言，基本上兼容原来Kom shell
mksh.x86_64
```

11.2.0.2之前

```
yum -y install binutils compat-libcap1 compat-libstdc++-33 compat-libstdc++-33*i686 compat-libstdc++-33*.devel compat-libstdc++-33 compat-libstdc++-33*.devel gcc gcc-c++ glibc glibc*.i686 glibc-devel glibc-devel*.i686 ksh libaio libaio*.i686 libaio-devel libaio-devel*.devel libgcc libgcc*.i686 libstdc++ libstdc++*.i686 libstdc++-devel libstdc++-devel*.devel libXi libXi*.i686 libXtst libXtst*.i686 make sysstat unixODBC unixODBC*.i686 unixODBC-devel unixODBC-devel*.i686 elfutils-libelf-devel
```

11.2.0.2之后

```
yum install binutils compat-libcap1 compat-libstdc++-33 gcc gcc-c++ glibc glibc-devel ksh libaio libaio-devel libgcc libstdc++ libstdc++-devel libXi libXtst  make sysstat unixODBC unixODBC-devel
```

查看软件包是否安装：`rpm -q package_name`

#### 验证UDP和TCP内核参数

设置TCP / IP临时端口范围参数，以为预期的服务器工作负载提供足够的临时端口。确保将下限范围设置为9000或更高，以避免Oracle和其他服务器端口通常使用的已注册端口范围内的端口。如果该范围的下限值大于9000，且该范围足够大以应付预期的工作负荷，则可以忽略有关临时端口范围的OUI警告。
检查当前的临时端口范围：`cat /proc/sys/net/ipv4/ip_local_port_range`
永久修改参数：

```
vim /etc/sysctl.conf
...
net.ipv4.ip_local_port_range = 9000 65500
...
#重启网络
/etc/rc.d/init.d/network restart
```

#### 安装cvuqdisk.rpm

没有`cvuqdisk`，Cluster Verification Utility（CVU）找不到共享磁盘，并且在运行CVU时收到“Package cvuqdisk not installed”错误。

> Note:
> 可以修改文件`oracle_home1/cv/admin/cvu_config`禁用CVU共享磁盘检查：`CV_RAW_CHECK_ENABLED =FALSE`
> 在此示例中，`oracle_home1`是数据库安装所在的Oracle主目录。

1. 在安装包的rpm目录中找到cvuqdisk RPM软件包。如果安装了Oracle Grid Infrastructure，则位于`oracle_home1/cv/rpm`目录中。
2. 登录root用户
3. 查看是否已安装cvuqdisk：`# rpm -qi cvuqdisk`
   如果存在已安装版本，卸载：`# rpm -e cvuqdisk`
4. 将环境变量`CVUQDISK_GRP`设置为拥有`cvuqdisk`的组，通常是`oinstall`，例如：`# CVUQDISK_GRP=oinstall; export CVUQDISK_GRP`
5. 在保存`cvuqdisk`的目录下，安装：`rpm -iv cvuqdisk-1.0.7-1.rpm`

#### 确认主机名解析

能ping通：`ping centos`

#### 禁用Transparent HugePages

从Red Hat Enterprise Linux 6开始，Transparent HugePages默认启用。但是，Transparent HugePages可能会导致内存分配延迟，因为内存是动态分配的。
Oracle建议您使用标准的HugePages来增强性能。
查看Transparent HugePages是否启用：`cat /sys/kernel/mm/transparent_hugepage/enabled`
默认情况下,状态为always，需要调整为never

关闭Transparent HugePages：

1. 修改`etc/default/grub`文件，添加`transparent_hugepage=never`

```
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto spectre_v2=retpoline rhgb quiet transparent_hugepage=never"
GRUB_DISABLE_RECOVERY="true"
```

2. 执行生效命令`grub2-mkconfig -o /boot/grub2/grub.cfg`
3. 重启reboot

#### 创建所需的系统组和用户

安装Oracle数据库，需要以下本地系统组和用户：

+ The Oracle Inventory group (typically, `oinstall`)
+ The OSDBA group (typically, `dba`)
+ The Oracle software owner (typically, `oracle`)
+ The OSOPER group (optional. Typically, `oper`)
  [创建任务角色划分操作系统权限组、用户和目录参考](https://www.oracle.com/technetwork/cn/articles/hunter-rac11gr2-iscsi-2-092412-zhs.html#13)

查看Oracle Inventory group是否存在：

```
more /etc/oraInst.loc
```

如果oraInst.loc存在，那么此命令的输出类似于以下内容：

```
inventory_loc=/u01/app/oraInventory
inst_group=oinstall
```

`inventory_loc`显示Oracle Central Inventory的位置
`inst_group`显示Oracle Inventory group的名称

如不存在，创建`oinstall`和`dba`用户组：

```
/usr/sbin/groupadd oinstall
/usr/sbin/groupadd dba
```

创建`oracle`用户：

```
/usr/sbin/useradd -g oinstall -G dba oracle
```

将oinstall指定为primary group，并将dba指定为secondary group。

设置`oracle`的密码：

```
passwd oracle
```

#### 配置内核参数

最小值（如果参数的当前值大于此表中列出的值，则不更改）：

| Parameter           | Value                                                        | File                                   |
| ------------------- | ------------------------------------------------------------ | -------------------------------------- |
| semmsl              | 250                                                          | /proc/sys/kernel/sem                   |
| semmns              | 32000                                                        |                                        |
| semopm              | 100                                                          |                                        |
| semmni              | 128                                                          |                                        |
| shmall              | 2097152                                                      | /proc/sys/kernel/shmall                |
| shmmax              | 最小: 536870912；最大: 比物理内存小1byte；推荐：大于等于物理内存一半 | /proc/sys/kernel/shmmax                |
| shmmni              | 4096                                                         | /proc/sys/kernel/shmmni                |
| file-max            | 6815744                                                      | /proc/sys/fs/file-max                  |
| aio-max-nr          | 1048576 注意：该值限制了并发的未完成请求，应设置为避免I/O子系统故障 | /proc/sys/fs/aio-max-nr                |
| ip_local_port_range | 最小：9000 最大：65500                                       | /proc/sys/net/ipv4/ip_local_port_range |
| rmem_default        | 262144                                                       | /proc/sys/net/core/rmem_default        |
| rmem_max            | 4194304                                                      | /proc/sys/net/core/rmem_max            |
| wmem_default        | 262144                                                       | /proc/sys/net/core/wmem_default        |
| wmem_max            | 1048576                                                      | /proc/sys/net/core/wmem_max            |

[查看当前参数值](https://docs.oracle.com/cd/E11882_01/install.112/e47689/pre_install.htm#LADBI1188)

**更改参数：**

1. 创建或编辑`/etc/sysctl.conf`文件，添加或编辑类似于以下行：

```
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 2097152
kernel.shmmax = 4294967295
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
```

> Note:
> 对于信号量参数（kernel.sem），必须指定所有四个值。如果任何当前值大于最小值，指定较大的值。

2. 更改内核参数的当前值：
   `/sbin/sysctl -p`
   对比输出值以验证正确。如果值不正确，编辑`/etc/sysctl.conf`，并再次执行。

3. 输入`/sbin/sysctl -a`确认设置正确

4. 重启或`sysctl -p`


#### 资源限制需求

installation owner资源限制的建议范围

| Resource Shell Limit                           | Resource | Soft Limit        | Hard Limit                              |
| ---------------------------------------------- | -------- | ----------------- | --------------------------------------- |
| Open file descriptors                          | nofile   | at least 1024     | at least 65536                          |
| Number of processes available to a single user | nproc    | at least 2047     | at least 16384                          |
| Size of the stack, segment of the process      | stack    | at least 10240 KB | at least 10240 KB, and at most 32768 KB |

**查看资源限制：**

1. 登录安装用户`oracle`
2. 检查file descriptor的软限制和硬限制，确保结果在建议范围内：

```
$ ulimit -Sn
1024
$ ulimit -Hn
4096

```

3. 检查用户可用进程数的软限制和硬限制，确保结果在建议范围内：

```
$ ulimit -Su
4096
$ ulimit -Hu
7268

```

4. 检查stack的软限制，确保结果在建议范围内：

```
$ ulimit -Ss
8192
$ ulimit -Hs
unlimited
```

如有必要，为安装用户更新`/etc/security/limits.conf`配置文件中的资源限制。

```
nano /etc/security/limits.conf
...
oracle           soft    nproc           2047
oracle           hard    nproc           16384
oracle           soft    nofile          1024
oracle           hard    nofile          65536
oracle           soft    stack           10240
```

> Note：
> 如果`grid`或`oracle`用户已登录，则在注销并重新登录之后，limits.conf文件中的更改才会生效。

#### 创建所需目录

创建类似于以下名字的目录，并为其指定正确的所有者，组和权限：

+ The Oracle base directory
+ An optional Oracle data file directory
  The Oracle base directory必须具有3GB的可用磁盘空间，如果选择不创建单独的Oracle data file directory，则必须具有4GB的可用磁盘空间（不推荐）。
  创建Oracle base directory：

1. 查看所有挂载点

```
df -k
```

2. 选择满足条件的挂载点
3. 创建子目录，并为其设置适当的所有者，组和权限：

```
mkdir -p /home/oracle/app/
chown -R oracle:oinstall /home/oracle/app/
chmod -R 775 /home/oracle/app/
```

默认Oracle Inventory Directory：`ORACLE_BASE/../oraInventory`
默认Oracle Home Directory：`ORACLE_BASE/product/11.1.0/dbhome_1`

#### 设置oracle用户的环境变量

编辑 .bash_profile，添加：

```
umask 022
#oracle基目录
export ORACLE_BASE=/home/oracle/app
#oracle主目录
export ORACLE_HOME=$ORACLE_BASE/oracle/product/11.2.0/dbhome_1
#SID数据库名
export ORACLE_SID=orcl
export PATH=$PATH:$HOME/bin:$ORACLE_HOME/bin
#export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/usr/lib
```

生效：`source .bash_profile `

> Note:
> 如果设置了ORACLE_HOME环境变量，那么Oracle Universal Installer将使用它指定的值作为Oracle主目录的默认路径。如果存在ORACLE_BASE环境变量，则Oracle建议您取消设置ORACLE_HOME环境变量，并选择Oracle Universal Installer建议的默认路径。

查看环境变量：

```
$ umask
$ env | more
```

### 静默安装

> Note:
> 一份完整的响应文件包含数据库管理帐户和OSDBA组用户（自动备份必需）的密码。确保只有Oracle软件所有者用户才能查看或修改响应文件，或考虑在安装成功后删除它们。

1. 切换oracle用户
2. 如果要完成响应文件模式的安装，请设置`DISPLAY`环境变量。
3. unzip linux.x64_11gR2_database_1of2.zip
   unzip linux.x64_11gR2_database_1of2.zip

4. 修改`db_install.rsp`

```
oracle.install.option=INSTALL_DB_SWONLY
ORACLE_HOSTNAME=localhost
UNIX_GROUP_NAME=oinstall
INVENTORY_LOCATION=/home/oracle/app/oraInventory
SELECTED_LANGUAGES=en,zh_CN
ORACLE_HOME=/home/oracle/app/product/11.2.0/dbhome_1 
ORACLE_BASE=/home/oracle/app
oracle.install.db.InstallEdition=EE
oracle.install.db.isCustomInstall=true
oracle.install.db.DBA_GROUP=dba
oracle.install.db.OPER_GROUP=oinstall
oracle.install.db.config.starterdb.type=GENERAL_PURPOSE
oracle.install.db.config.starterdb.globalDBName=orcl
oracle.install.db.config.starterdb.SID=orcl
oracle.install.db.config.starterdb.characterSet=ZHS16GBK
oracle.install.db.config.starterdb.memoryOption=true（自动内存管理）
oracle.install.db.config.starterdb.memoryLimit=1800（根据实际内存设置）
oracle.install.db.config.starterdb.installExampleSchemas=false
oracle.install.db.config.starterdb.enableSecuritySettings=true
oracle.install.db.config.starterdb.password.ALL=123456
oracle.install.db.config.starterdb.control=DB_CONTROL
oracle.install.db.config.starterdb.dbcontrol.enableEmailNotification=false
DECLINE_SECURITY_UPDATES=true
```

4. 运行Oracle Universal Installer

```
/directory_path/runInstaller [-silent] [-noconfig] \
 -responseFile responsefilename
```

+ `-silent`  静默安装
+ `-noconfig` 禁止在安装过程中运行配置助手，而是执行仅软件安装。
+ `responsefilename` 响应文件的绝对路径
  例：

```
./runInstaller -silent -responseFile /home/oracle/database/response/db_install.rsp
```

[WARING]可暂时忽略，此时安装程序仍在后台进行，如果出现[FATAL]，则安装程序已经停止了。
安装需要一会儿，如果想看安装进度，可以再打开一个窗口，输出会话日志

```
tail -f 日志文件
```

5. 要确定静默方式安装是成功还是失败，请参考以下日志文件：
   `/oraInventory_location/logs/silentInstalldate_time.log`

两个错误：

+ 调用 makefile '/home/oracle/app/product/11.2.0/dbhome_1/ctx/lib/ins_ctx.mk' 的目标 'install' 时出错
  解决方法：经检查，libstdc++3.3.3 已正确安装，也就是/usr/lib64/libstdc++.so.5.0.7 /usr/lib64/libstdc++.so.5两个文件，发现可能是这两个文件有问题，换成mandriva的 libstdc++5-3.3.6-4mdv2009.0.x86_64即可，也可以用redhat上的。
+ ins_emagent.mk 链接错误
  解决方法：
  修改` $ORACLE_HOME/sysman/lib/ins_emagent.mk`
  找到 `$(MK_EMAGENT_NMECTL)`
  改成:
  `$(MK_EMAGENT_NMECTL) -lnnz11`

6. 安装失败后清理
   卸载工具（deinstall）在安装介质中可用，安装后在Oracle主目录中可用。它位于`$ORACLE_HOME/deinstall`目录中。

```
deinstall -home complete path of Oracle home [-silent] [-checkonly] [-local] [-paramfile complete path of input parameter property file] [-params name1=value
name2=value . . .] [-o complete path of directory for saving files]
[-tmpdir complete path of temporary directory to use]
[-logdir complete path of log directory to use] [-help]
```

[参数设置](https://docs.oracle.com/cd/E11882_01/install.112/e47689/remove_oracle_sw.htm#LADBI1334)

7. 安装完成后如果有类似如下提示，说明安装完成

```
以下配置脚本需要以 "root" 用户的身份执行。
   #!/bin/sh
   #要运行的 Root 脚本
 /home/oracle/app/oraInventory/orainstRoot.sh
/home/oracle/app/oracle/product/11.2.0/db_1/root.sh
  要执行配置脚本, 请执行以下操作:
     1. 打开一个终端窗口
     2. 以 "root" 身份登录
     3. 运行脚本
     4. 返回此窗口并按 "Enter" 键继续
  Successfully Setup Software.
```

8. 使用root用户执行脚本

```
sudo su
sh /home/oracle/app/oraInventory/orainstRoot.sh
sh /home/oracle/app/product/11.2.0/db_1/root.sh
```

### 配置监听程序

1. 切换oracle用户

```
export ORACLE_BASE=/home/oracle/app
export ORACLE_HOME=$ORACLE_BASE/oracle/product/11.2.0/dbhome_1
#配置DISPLAY环境变量：export DISPLAY=本机IP:0.0
export DISPLAY=192.168.123.82:0.0
```

2. 以静默模式运行Net Configuration Assistant

```
$ORACLE_HOME/bin/netca -silent -responsefile /home/oracle/database/response/netca.rsp

#输出结果：
正在对命令行参数进行语法分析:
参数"silent" = true
参数"responsefile" = /home/oracle/database/response/netca.rsp
完成对命令行参数进行语法分析。
Oracle Net Services 配置:
完成概要文件配置。
Oracle Net 监听程序启动:
    正在运行监听程序控制:
      /home/oracle/app/product/11.2.0/dbhome_1/bin/lsnrctl start LISTENER
    监听程序控制完成。
    监听程序已成功启动。
监听程序配置完成。
成功完成 Oracle Net Services 配置。退出代码是0
```

成功运行后，在`$ORACLE_HOME/network/admin`目录下生成`sqlnet.ora`和`listener.ora`两个文件。
完成后通过命令`netstat -tnpl | grep 1521`可以查看到1521端口已开

### 静默创建数据库

> See Also：
> "Oracle ASM Configuration Assistant Command-Line Interface" section in [Oracle Automatic Storage Management Administrator's Guide](https://docs.oracle.com/cd/E11882_01/server.112/e18951/toc.htm)
> ，提供了有关以非交互方式运行Oracle ASMCA的信息

1. 编辑响应文件`response/dbca.rsp`

```
[GENERAL]
RESPONSEFILE_VERSION = "11.2.0"
OPERATION_TYPE = "createDatabase"
[CREATEDATABASE]
GDBNAME = "orcl"
SID = "orcl"
TEMPLATENAME = "General_Purpose.dbc"
#SYSPASSWORD = "oracle"
#SYSTEMPASSWORD = "oracle"
#SYSMANPASSWORD = "oracle"
#DBSNMPPASSWORD = "oracle"
#DATAFILEDESTINATION =/home/oracle/oradata
RECOVERYAREADESTINATION=/data/app/oracle/fast_recovery_area
CHARACTERSET = "ZHS16GBK"
NATIONALCHARACTERSET= "AL16UTF16"
AUTOMATICMEMORYMANAGEMENT = "TRUE"
TOTALMEMORY = "1800"
```

2. 执行静默建库

```
$ORACLE_HOME/bin/dbca -silent -sysPassword joyin -systemPassword joyin -responseFile /home/oracle/database/response/dbca.rsp 
```

执行过程如下

```
复制数据库文件
1% 已完成
3% 已完成
11% 已完成
18% 已完成
26% 已完成
37% 已完成
正在创建并启动 Oracle 实例
40% 已完成
45% 已完成
50% 已完成
55% 已完成
56% 已完成
60% 已完成
62% 已完成
正在进行数据库创建
66% 已完成
70% 已完成
73% 已完成
85% 已完成
96% 已完成
100% 已完成
有关详细信息, 请参阅日志文件 "/home/oracle/app/cfgtoollogs/dbca/orcl/orcl.log"。
```

查看进程

```
ps -ef | grep ora_ | grep -v grep
# 执行结果
oracle   25717     1  0 12:24 ?        00:00:00 ora_pmon_orcl
oracle   25719     1  0 12:24 ?        00:00:01 ora_vktm_orcl
oracle   25724     1  0 12:24 ?        00:00:00 ora_gen0_orcl
oracle   25726     1  0 12:24 ?        00:00:00 ora_diag_orcl
oracle   25728     1  0 12:24 ?        00:00:00 ora_dbrm_orcl
oracle   25730     1  0 12:24 ?        00:00:00 ora_psp0_orcl
oracle   25732     1  0 12:24 ?        00:00:00 ora_dia0_orcl
oracle   25734     1  0 12:24 ?        00:00:00 ora_mman_orcl
oracle   25736     1  0 12:24 ?        00:00:00 ora_dbw0_orcl
oracle   25738     1  0 12:24 ?        00:00:00 ora_lgwr_orcl
oracle   25740     1  0 12:24 ?        00:00:00 ora_ckpt_orcl
oracle   25742     1  0 12:24 ?        00:00:00 ora_smon_orcl
oracle   25744     1  0 12:24 ?        00:00:00 ora_reco_orcl
oracle   25746     1  0 12:24 ?        00:00:00 ora_mmon_orcl
oracle   25748     1  0 12:24 ?        00:00:00 ora_mmnl_orcl
oracle   25750     1  0 12:24 ?        00:00:00 ora_d000_orcl
oracle   25752     1  0 12:24 ?        00:00:00 ora_s000_orcl
oracle   25809     1  0 12:24 ?        00:00:00 ora_qmnc_orcl
oracle   25828     1  0 12:24 ?        00:00:00 ora_cjq0_orcl
oracle   25834     1  0 12:24 ?        00:00:00 ora_q000_orcl
oracle   25836     1  0 12:24 ?        00:00:00 ora_q001_orcl
oracle   26071     1  0 12:29 ?        00:00:00 ora_smco_orcl
```

查看监听状态

```
$ORACLE_HOME/bin/lsnrctl status
# 执行结果
LSNRCTL for Linux: Version 11.2.0.1.0 - Production on 05-FEB-2020 12:32:23

Copyright (c) 1991, 2009, Oracle.  All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=IPC)(KEY=EXTPROC1521)))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 11.2.0.1.0 - Production
Start Date                05-FEB-2020 11:08:57
Uptime                    0 days 1 hr. 23 min. 25 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Parameter File   /home/oracle/app/product/11.2.0/dbhome_1/network/admin/listener.ora
Listener Log File         /home/oracle/app/diag/tnslsnr/centos/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=centos.lan)(PORT=1521)))
Services Summary...
Service "orcl" has 1 instance(s).
  Instance "orcl", status READY, has 1 handler(s) for this service...
Service "orclXDB" has 1 instance(s).
  Instance "orcl", status READY, has 1 handler(s) for this service...
The command completed successfully
```

登录数据库：

```
su oracle
sqlplus / as sysdba
```

查看数据库版本

```
SQL> select * from v$version;
```

### 设置Oracle开机启动

修改`$ORACLE_HOME/bin/dbstart`

```
ORACLE_HOME_LISTNER=$ORACLE_HOME
```

修改`$ORACLE_HOME/bin/dbshut`

```
ORACLE_HOME_LISTNER=$ORACLE_HOME
```

修改`/etc/oratab`
格式如下：
`$ORACLE_SID:$ORACLE_HOME:<N|Y>`
例：

```
orcl:/home/oracle/app/product/11.2.0/dbhome_1:Y
```

新建文件`/etc/rc.d/init.d/oracle`

```
#! /bin/bash
# oracle: Start/Stop Oracle Database 11g R2
#
# chkconfig: 345 90 10
# description: The Oracle Database is an Object-Relational Database Management System.
#
# processname: oracle
. /etc/rc.d/init.d/functions
LOCKFILE=/var/lock/subsys/oracle
ORACLE_HOME=/data/app/oracle/product/11.2.0
ORACLE_USER=oracle
case "$1" in
'start')
   if [ -f $LOCKFILE ]; then
      echo $0 already running.
      exit 1
   fi
   echo -n $"Starting Oracle Database:"
   su - $ORACLE_USER -c "$ORACLE_HOME/bin/lsnrctl start"
   su - $ORACLE_USER -c "$ORACLE_HOME/bin/dbstart $ORACLE_HOME"
   su - $ORACLE_USER -c "$ORACLE_HOME/bin/emctl start dbconsole"
   touch $LOCKFILE
   ;;
'stop')
   if [ ! -f $LOCKFILE ]; then
      echo $0 already stopping.
      exit 1
   fi
   echo -n $"Stopping Oracle Database:"
   su - $ORACLE_USER -c "$ORACLE_HOME/bin/lsnrctl stop"
   su - $ORACLE_USER -c "$ORACLE_HOME/bin/dbshut"
   su - $ORACLE_USER -c "$ORACLE_HOME/bin/emctl stop dbconsole"
   rm -f $LOCKFILE
   ;;
'restart')
   $0 stop
   $0 start
   ;;
'status')
   if [ -f $LOCKFILE ]; then
      echo $0 started.
      else
      echo $0 stopped.
   fi
   ;;
*)
   echo "Usage: $0 [start|stop|status]"
   exit 1
esac
exit 0
```

更改权限：`chmod +x /etc/init.d/oracle`
开机启动oracle：`chkconfig oracle on`
停止oracle：`service oracle stop`
开启oracle：`service oracle start`
启动监听：`lsnrctl status`
启动数据库：`SQL> statup`

### 防火墙配置放开Oracle的端口

```bash
firewall-cmd --zone=public --add-port=1521/tcp --permanent
firewall-cmd --reload
```
