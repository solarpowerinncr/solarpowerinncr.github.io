# Linuximint初始化



## 开机黑屏
用recovery模式进，改驱动为nvidia闭源驱动

## 输入法
1. 安装fcitx框架 切换：ctrl+space
2. 安装搜狗拼音,下载后双击，安装完毕后注销再登录

<!--more-->

## 删除楷体
```bash
sudo apt remove fonts-arphic-ukai fonts-arphic-uming
```
## 换源
同过软件源换源，再打开`/etc/apt/sources.list.d/official-package-repositories.list`换security项

## 字体发虚
```bash
sudo apt-get install language-selector-*  
#重启boot
```

## 修改firefox默认缩放比
在firefox网址栏打开about:config，搜索layout.css.devPixelsPerPx，修改为1.2（120%）

## 安装v2ray
按照`sudo bash go.sh`安装，配置`config.json` 重启`service v2ray start`

## 全局代理
```
sudo apt-get install proxychains4
修改 /etc/proxychains4.conf
proxychains4 google-chrome
```
下载chrome、网易云、百度云、guake

查看环境变量： 
`export` 
`echo $all_proxy`

## 换用gedit并解决中文乱码
```bash
sudo apt-get install dconf-tools
```
修改xed的配置`/org/x/editor/preferences/encodings`
Candidate Encodings的值替换为`['GB18030', 'UTF-8', 'CURRENT', 'ISO-8859-15', 'UTF-16']`

## 安装主题
1. adapta
```bash
sudo apt install software-properties-common
sudo add-apt-repository ppa:tista/adapta
sudo apt-get update
sudo apt-get install adapta-gtk-theme
```
2. arc
```bash
sudo apt-get install arc-theme
```
安装后在设置->窗口管理器选择主题

## 安装图标
1. papirus
```bash
sudo add-apt-repository ppa:papirus/papirus
sudo apt-get update
sudo apt-get install papirus-icon-theme
```
2. 安装Hardcode-Tray
```bash
sudo add-apt-repository ppa:papirus/hardcode-tray
sudo apt update
sudo apt install sni-qt sni-qt:i386 hardcode-tray
```

## 开机自动挂载硬盘
```bash
#查看硬盘信息
sudo blkid
sudo nano /etc/fstab
#在末尾添加以下内容
UUID=B8D0FF63D0FF25F2 [/media/sherry/C](/media/sherry/C) ntfs defaults 0 1
UUID=A87C9D797C9D42CC [/media/sherry/D](/media/sherry/D) ntfs defaults 0 1
UUID=76D4C71BD4C6DD0D [/media/sherry/G](/media/sherry/G) ntfs defaults 0 1
UUID=FA4EE32C4EE2DFFD [/media/sherry/F](/media/sherry/F) ntfs defaults 0 1
#boot重启
```

