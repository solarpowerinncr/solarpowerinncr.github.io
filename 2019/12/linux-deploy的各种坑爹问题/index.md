# Linux Deploy的各种坑爹问题




首先就是因为和安卓公用内核，所以安卓6.0及以下的内核太旧了好多软件不能用，比如docker
然后不知为啥chroot环境下不能设置system，导致大多数需要后台的软件跑不起来

不过跑个caddy搭个网盘还是可以的

<!--more-->

## 配置linux deploy

### 设置里：

- [x] 锁定wifi 
- [x] cpu唤醒
- [x] 开机自启动

PATH变量：填busybox里的安装地址，比如`/system/xbin`

- [x] 更新环境

### 配置里：
架构：`arm64`

源地址：`http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports`(换来换去还是清华源最好用，不过不要用https因为还没安装apt-transport-https )

挂载：`/storage/emulated/0:/mnt/storage`

启用SSH服务器

然后就安装->启动，OK

## FileBrowser的配置

### 安装及配置
安装：`curl -fsSL https://filebrowser.xyz/get.sh | bash`

创建配置数据库：`filebrowser -d /etc/filebrowser.db config init`

设置监听地址：`filebrowser -d /etc/filebrowser.db config set --address 0.0.0.0`

设置监听端口：`filebrowser -d /etc/filebrowser.db config set --port 8088`

设置语言环境：`filebrowser -d /etc/filebrowser.db config set --locale zh-cn`

设置日志位置：`filebrowser -d /etc/filebrowser.db config set --log /var/log/filebrowser.log`

设置根目录：`filebrowser -d /etc/filebrowser.db config set --root /path/to/dir`

添加一个用户：`filebrowser -d /etc/filebrowser.db users add root password --perm.admin`，其中的`root`和`password`分别是用户名和密码，根据自己的需求更改。

有关更多配置的选项，可以参考官方文档：[https://docs.filebrowser.xyz/](https://docs.filebrowser.xyz/)

&nbsp;

配置修改好以后，就可以启动 FileBrowser 了，使用`-d`参数指定配置数据库路径。示例：`filebrowser -d /etc/filebrowser.db`

可以写个启动脚本
```
cd /mnt/storage/
vim startes.sh
 
#!/bin/bash
nohup filebrowser -d /etc/filebrowser.db >/dev/null 2>&1 &

chmod +x startes.sh
sh startes.sh
```
启动成功就可以使用浏览器访问 File Browser 了，在浏览器输入 IP:端口，示例：http://192.168.1.1:8088

然后会看到 FileBrowser 的登陆界面，用刚刚创建的用户登陆。

登陆以后，默认会看到 FileBrowser 设置的根目录下的文件，需要更改一下当前用户的文件夹位置。

点击 [设置] → [用户设置] → 编辑用户 admin → 将目录范围改为你想要显示的文件夹，例如：/mnt → 修改完成后点击最下方的保存即可。
这样，FileBrowser 的基本安装和配置就搞定了。

### 后台运行

第一种是 nohup 大法：

运行：`nohup filebrowser -d /etc/filebrowser.db >/dev/null 2>&1 &`

停止运行：`kill -9 $(pidof filebrowser)`

开机启动：`sed -i '/exit 0/i\nohup filebrowser -d \/etc\/filebrowser.db >\/dev\/null 2>&1 &' /etc/rc.local`

或者

```bash
vim /etc/rc.local
##添加
sh /mnt/storage/startes.sh
 
chmod +x /etc/rc.local
```

取消开机启动：`sed -i '/nohup filebrowser -d \/etc\/filebrowser.db >\/dev\/null 2>&1 &/d' /etc/rc.local`

&nbsp;

第二种是 systemd 大法：

首先下载 FileBrowser 的 service 文件：`curl https://cdn.mivm.cn/www.mivm.cn/archives/filebrowser/filebrowser.service -o /lib/systemd/system/filebrowser.service`

如果你的运行命令不是`/usr/local/bin/filebrowser -d /etc/filebrowser.db`，需要对 service 文件进行修改，将文件的 ExecStart 改为你的运行命令，更改完成后需要输入`systemctl daemon-reload`。

运行：`systemctl start filebrowser.service`

停止运行：`systemctl stop filebrowser.service`

开机启动：`systemctl enable filebrowser.service`

取消开机启动：`systemctl disable filebrowser.service`

查看运行状态：`systemctl status filebrowser.service`

### HTTPS
File Browser 2.0 起开始内建 HTTPS 支持，只需要配置 SSL 证书即可。

配置 SSL：`filebrowser -d /etc/filebrowser.db config set --cert example.com.crt --key example.com.key`，其中example.com.crt和example.com.key分别是 SSL 证书和密钥路径，根据自身情况进行更改。配置完 SSL 后，只可以使用 HTTPS 访问，不可以使用 HTTP。

取消 SSL：`filebrowser -d /etc/filebrowser.db config set --cert "" --key ""`

当然，你也可以使用 Nginx 等 Web 服务器对 File Browser 进行反向代理，以达到 HTTPS 访问的目的。

还有就是使用 Caddy，这是一个开源、支持 HTTP/2 的 Web 服务器，它的一个显著特点就是默认启用 HTTPS 访问，会自己申请 SSL 证书，同时支持大量的插件，File Browser 就可以作为其插件运行。