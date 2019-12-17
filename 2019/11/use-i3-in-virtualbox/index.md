# 在virtualbox里折腾i3wm




选用了一个作死的版本：ubuntu mini ISO，安装的时候记得选国内源，~~时区现在还没调回来~~

<!--more-->

## 安装i3
sudo apt install i3 （i3wm i3status i3lock i3bar）
## 安装xinit
sudo apt install xinit
输入startx打开桌面
如果希望有回落环境，可以考虑安装 Xfce ( 只是核心包)，这仍然会使膨胀的最小程度：
sudo apt install xfce4
## VB安装增强功能
1.mount /dev/cdrom  /mnt
2.cd /mnt
3../VBoxLinx**.run
ubuntu mini里居然没有装gcc make perl git！！
还有build-essential
## Ubuntu自定义桌面分辨率
[https://www.jianshu.com/p/d278d0d24748](https://www.jianshu.com/p/d278d0d24748)
:xrandr（查看当前分辨率和所有分辨率选项）
然后再在上面数你要设置的分辨率是第几行，比如我要设置1280*960，是在第七行
:xrandr -s 7 就这样，设置好了

~~安装ly登陆管理器
[https://github.com/cylgom/ly](https://github.com/cylgom/ly)
没看懂makefile怎么工作的，但是记得安装依赖库`libpam0g-dev`和`libxcb1-dev`~~

## 中文输入法
```shell
sudo apt-get install fcitx fcitx-sunpinyin fcitx-configtool 
sudo apt-get install im-config fcitx-frontend-gtk2 fcitx-frontend-gtk3
im-config
```
根据提示设置默认输入法
在`.bashrc`中加入
```shell
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```
在i3config中加入`exec fcitx`
待解决：为啥fcitx自启动和ly不能兼容？
安装搜狗输入法
```shell
sudo dpkg -i sogoupinyin_2.2.0.0108_amd64.deb
sudo apt-get  -f install #解决依赖
sudo dpkg -i sogoupinyin_2.2.0.0108_amd64.deb
```

## 安装字体
```shell
sudo cp ~/Desktop/font/*.ttf /usr/local/share/fonts
#然后，改变权限
sudo chmod 744 /usr/share/fonts/winFonts/*.ttf
#开始安装
cd /usr/share/fonts/winFonts/
sudo mkfontscale  #创建字体的fonts.scale文件，它用来控制字体旋转缩放
sudo mkfontdir  #创建字体的fonts.dir文件，它用来控制字体粗斜体产生
sudo fc-cache -fv * #建立字体缓存信息，也就是让系统认识雅黑
```
修改dmenu的字体
```
修改 .config/i3/config 中找到 
bindsym $mod+d exec dmenu_run
添加参数
bindsym $mod+d exec dmenu_run -fn monaco
// 以Monaco字体为例
使用 $mod+shift+r 应用配置
```
## gtk修改主题
使用lxappearance

## 安装mpd
### 安装alsa
```shell
sudo apt install libasound2 libasound2-plugins alsa-base alsa-utils alsa-oss
cat /proc/asound/version #打印当前版本
alsamixer #调音量
```
### 安装pulseaudio
```shell
sudo apt-get install pulseaudio pulseaudio-utils pavucontrol
pavucontrol #调音量
```
### 安装pamixer
```shell
sudo apt install libpulse-dev libboost-program-options-dev
git clone https://github.com/cdemoulins/pamixer.git
cd pamixer
make
ln -s pamixer ~/.local/bin/pamixer
pacmd list-sinks | grep name #查看声卡名称
```
### 安装mpd并配置：
[Archwiki](https://wiki.archlinux.org/index.php/Music_Player_Daemon_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E5%91%BD%E4%BB%A4%E8%A1%8C%E4%B8%8B)
**我在这里才发现.xinitc和.xprofile这俩玩意我完全没搞懂！！到底怎么启动用户下的文件**
**全局配置会自动使用/etc/mpd.conf，这里使用本地配置，启动需要手动mpd ~.config/mpd/mpd.conf，为了启动需要先disable掉全局的systemctl mpd.serevice&mpd.socket**
```shell
sudo apt install mpd mpc
sudo systemctl disable mpd.service
mkdir ~/.config/mpd
cd .config/mpd/
cp /usr/share/doc/mpd/mpdconf.example.gz  mpd.conf.gz
gunzip mpd.conf.gz 

# mpd.conf
music_directory     "~/music"
playlist_directory      "~/.config/mpd/playlists"
db_file         "~/.config/mpd/database"
log_file            "~/.config/mpd/log"
pid_file            "~/.config/mpd/pid"
state_file          "~/.config/mpd/state"
sticker_file            "~/.config/mpd/sticker.sql"
bind_to_address     "localhost"
port                "6600"

input {
        plugin "curl"
#       proxy "proxy.isp.com:8080"
#       proxy_user "user"
#       proxy_password "password"
}

audio_output {  #配合ncmpcpp用的
    type                    "fifo"
    name                    "my_fifo"
    path                    "/tmp/mpd.fifo"
    format                  "44100:16:2"
}

audio_output {
        type            "pulse"
        name            "pulse audio"
        mixer_type       "software"
}
#这段貌似是关于自动更新，不用也行反正我没用
log_level            "verbose"
auto_update    "yes"
auto_update_depth "3"
follow_outside_symlinks    "yes"
follow_inside_symlinks        "yes"

...
```
创建所有上述配置中提及的文件和目录
```shell
mkdir ~/.config/mpd/playlists
mkdir ~/music
touch ~/.config/mpd/{database,log,pid,state,sticker.sql}
mpd --kill
mpd ~/.config/mpd/mpd.conf #重启配置
#使用mpc客户端
mpc update
mpc add sleep.mp3 
mpc add http://relay3.slayradio.org:8000
mpc play
```
开机自启动
添加`exec --no-startup-id mpd ~/.config/mpd/mpd.conf`到`.config/i3/config`
mpc常用参数：[https://www.jianshu.com/p/2d5884a6d317](https://www.jianshu.com/p/2d5884a6d317)



## 安装compton
```shell
sudo apt install compton
nano ~/.config/compton.conf #创建配置文件
```
在`.config/i3/config`里加入`exec --no-startup-id compton -b`

## 安装feh
```shell
sudo apt install feh
#在i3config中加入
exec feh --bg-scale /path/to/image.file
```
feh浏览图片功能[看这里](https://www.cnblogs.com/dylancao/p/9524516.html)

## 显卡驱动
[配置显卡驱动](https://wiki.manjaro.org/index.php?title=Configure_Graphics_Cards#How_check_driver)
[配置nvidia闭源显卡驱动](https://link.jianshu.com/?t=https%3A%2F%2Fwiki.manjaro.org%2Findex.php%3Ftitle%3DConfigure_NVIDIA_%28non-free%29_settings_and_load_them_on_Startup)


## 使用py3status装饰i3bar
```shell
sudo apt-get install py3status
sudo apt-get install fonts-font-awesome
sudo cp /etc/i3status.conf  ~/.config/i3status/config #初始化配置
chown sherry:sherry ~/.config/i3status/config
```
在i3bar中加入`status_command py3status -c ~/.config/i3status/config`
例子：
```
bar {
        font pango:Noto Sans,FontAwesome Regular  11
        status_command py3status

colors {
    background #3C3C3C
    statusline #FFFFFF
    separator  #525252

    focused_workspace  #629BC6 #285577 #FFFFFF
    active_workspace   #333333 #3C3C3C #FFFFFF
    inactive_workspace #333333 #3C3C3C #888888
    urgent_workspace   #2F343A #904A00 #FFFFFF
    binding_mode       #3C3C3C #904A00 #FFFFFF
  }
}
```
i3status配置：https://py3status.readthedocs.io/en/latest/configuration.html
awesome icon复制：[https://fontawesome.com/cheatsheet?from=io](https://fontawesome.com/cheatsheet?from=io)
在线主题颜色编辑：[https://thomashunter.name/i3-configurator/](https://thomashunter.name/i3-configurator/)
模块依赖的程序：xbacklight

## 安装wicd
先安装提示器

1. xfce4-notifyd

```shell
sudo apt remove dunst
sudo apt install xfce4-notifyd
#手动运行
/usr/lib/x86_64-linux-gnu/xfce4/notifyd/xfce4-notifyd
#后台运行
systemctl --user start xfce4-notifyd
#配置
xfce4-notifyd-config
```
2. dunst

```shell
//安装
sudo apt install dunst 
//配置文件
wget https://raw.githubusercontent.com/dunst-project/dunst/master/dunstrc
mkdir -p ~/.config/dunst/
mv dunstrc  ~/.config/dunst/ 
//修改systemd文件
sudo nano /usr/lib/system/user/dunst.service
//在 Service项 添加 Environment=DISPLAY=:0
systemctl --user daemon-reload
systemctl --user start dunst
```

安装wicd
```shell
sudo apt install wicd python-wicd wicd-cli wicd-curses wicd-daemon wicd-gtk
#禁用各种网络管理守护进程，包括network, dhcdbd, 和 networkmanager：
systemctl stop netcfg
systemctl stop dhcpcd@.service
systemctl stop NetworkManager.service
systemctl disable netcfg
systemctl disable dhcpcd@.service
systemctl disable NetworkManager.service

sudo systemctl enable wicd.service
sudo gpasswd -a $USERNAME users
sudo systemctl start wicd
```
可以访问wicd的用户组可能不是 users. 检查`/etc/dbus-1/system.d/wicd.conf`中指定的用户组,并将你的用户加入该组。
可以把wicd加入i3config：`exec --no-startup-id wicd-client`


## rofi
```shell
sudo apt install rofi
mkdir ~/.config/rofi
nano ~/.config/rofi/config
#输入以下内容：
rofi.combi-modi:    window,drun,ssh,run
rofi.theme:         solarized
rofi.font:          Noto Sans 11
rofi.modi:          combi
...

#在i3config中加入
bindsym $mod+d exec --no-startup-id rofi -show combi

```
修改桌面图标：https://www.jianshu.com/p/f25465ca5e73
## i3lock



## 软件
[超级好用的pdf阅读器推荐：Zathura](https://www.douban.com/group/topic/22778587/?type=like)
[学术文章写作利器: TeXmacs介绍](http://x-wei.github.io/TeXmacs_intro.html)
Powerline
Conky



