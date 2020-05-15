# Rime输入法配置


## 安装
ubuntu:
```
sudo apt-get install fcitx fcitx-sunpinyin fcitx-configtool 
sudo apt-get install im-config fcitx-frontend-gtk2 fcitx-frontend-gtk3
im-config
```

<!--more-->

arch:

```
pacman -S fcitx fcitx-im fcitx-configtool
```

## 安装中文字体
```
sudo pacman -S ttf-dejavu wqy-microhei
```

## 添加对 gtk、qt的支持
 nano ~/.xprofile
```
#export LC_ALL=“zh_CN.UTF-8”
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

重启即可使用fcitx，快速切换中英输入可用快捷键：ctrl+space

## 基本配置
以下提到的配置文件皆放置或创建在`~/.config/fcitx/rime`文件夹下。

在`~/.config/fcitx/rime/`中新建`default.custom.yaml`

**注意缩进！**

```yaml
# default.custom.yaml
# save it to:
#   ~/.config/ibus/rime  (linux)
#   ~/Library/Rime       (macos)
#   %APPDATA%\Rime       (windows)

patch:
  schema_list:
    - schema: luna_pinyin_simp          # 朙月拼音
    - schema: double_pinyin        # 自然碼雙拼

  "ascii_composer/switch_key":
    Caps_Lock: noop
    Shift_L: commit_code # 左Shift上屏编码并切换为英文状态，inline_ascii 设[>
    Shift_R: inline_ascii # 不上屏，字符转变为英文输入，按Enter键英文字符上[>

  "punctuator/full_shape":
    "/": "/"
  "punctuator/half_shape":
    "/": "/"

  "menu/page_size": 9
```

## 皮肤
### ibus-rime
ibus-rime的皮肤配置写入如下文件就好

`~/.config/ibus/rime/default.custom.yaml`

记录一个配置皮肤的网页工具：https://rime.netlify.com/

### fcitx-rime
下载皮肤配置文件并复制到目录

```
/usr/share/fcitx/skin   ##全局设置
~/.config/fcitx/skin    #特定用户设置
```

推荐皮肤：

1.fcitx-skin-material

archlinux下可以直接使用`pacman -S fcitx-skin-material`进行安装。

2.mantis

3.fcitx-artwork
```bash
git clone https://github.com/fcitx/fcitx-artwork
cd fcitx-artwork
cp -r skin ~/.config/fcitx/
```

## 扩展词库
RIME的默认词库并没有多大，因为作者希望用户在与 RIME 磨合的过程中自己积累用户词库，确实自己养起来的词库更顺手，但养词库的过程多少还是有点痛苦的，好在RIME支持扩展词库。

这里使用的是 [xiaoTaoist](https://github.com/xiaoTaoist) 制作的词库扩展包

[GitHub 地址](https://github.com/xiaoTaoist/rime-dict)  [下载地址](https://github.com/Mogeko/blog-commits/releases/download/031/rime-dict.zip)

**使用方法**

将解压出来的所以文件复制到`~/.config/fcitx/rime`文件夹下，将`luna_pinyin.custom.yaml`重命名为`luna_pinyin_simp.custom.yaml`

这个扩展词库的`luna_pinyin.custom.yaml`中也包含了模糊拼音的功能，按注释开启即可。

右下角图标->重新部署，检查是否运行成功

## 同步
打开`~/.config/fcitx/rime`下的`installation.yaml`文件，添加或修改内容参考如下：
```
installation_id: "manjaro-i3" # 可以自定义 id 用来区分不同的电脑上的配置
sync_dir: "~/RimeSync" # 自定义同步文件夹位置，可以将其加入到同步盘中用于同步
```
然后，部署 -> 同步，你的用户配置、用户词库等都会被放在你配置的同步文件地址里。

如果你换了新的电脑，只需要将同步文件拷贝过去，然后配置一下 installation.yaml 文件，执行 部署 -> 同步 -> 部署 ，你的 用户配置、用户词库 都回来了。

> 参考：
> 
> [GNU/Linux 输入法折腾笔记 (RIME)](https://mogeko.me/2018/031/)
> 
> [Rime 输入法配置记录](https://10101.io/2019/01/30/rime-configuration)
> 
> [在 arch linux 下 配置 fcitx-rime 输入法](https://kelvin.mbioq.com/configuring-fcitxrime-input-method-under-arch-linux.html)
