---
title: Ubuntu 18.04之Gnome美化
date: 2019-05-14 21:33:57
tags: 
	- Ubuntu 2 Mac
	- Gnome
categories: Misc
---

![](my-ubuntu-config-2mac\a.png)

<!-- more -->

### 0x00 Mac主题

安装Tweak

> - sudo apt-get install gnome-tweak-tool

安装gnome-shell

> - sudo apt-get install chrome-gnome-shell

用浏览器打开https://extensions.gnome.org，点击页面的`Click here to install browser extension`按钮。

> - 点击页面extensions，启用`User Themes`扩展。
> - 下载并启动`blyr`扩展。
> - 下载并启动`netspeed`扩展：显示网络速度。
> - 下载并启动`coverflow alt-tab`：切换窗口的主题。
> - 下载并启动`hide dash x`：隐藏搜索时左边的Dock。

到https://www.gnome-look.org 下载MacOS 主题、图标、鼠标指针、GDM、Plymouth、字体

> - MacOS UI主题
> - Gnome-shell：按照`gnome-shell.css`文件渲染。
> - icons：应用图标。
> - GDM：锁屏时的主题。
> - Plymouth：开机启动动画。

将下载的主题文件（系统主题、gnome-shell主题）解压后放到用户家目录下的.themes目录下：`/home/.themes`。

将下载的图标文件和鼠标指针文件解压后放到用户家目录下的.icons目录下：`/home/.icons`。

把字体文件拷贝到~/.local/share/fonts目录下。

> 注：.themes和.icons需要自己创建。Ctrl+H可以显示目录下的隐藏文件。

之后，打开tweak选择主题、游标、图标、字体。。。。

设置窗口的按钮（最小化、最大化、关闭）在左边：

> Ubuntu：打开Tweak——Windows——Tilebar buttons——Placement——left
>
> Kali：打开Tweak——Windows Title——Tilebar buttons——Placement——left

壁纸网站（需科学上网）：https://wallpapersite.com/

### 0x01 Dock

卸载Ubuntu自带的Dock、

> - sudo apt remove gnome-shell-extension-ubuntu-dock

安装plank dock：

> - sudo apt-get install plank

设置开机启动：

> - `Win`键打开搜索 --> 搜索startup --> 打开startup application --> add --> command处输入plank即可。

plank主题：

> 主题下载地址：https://www.gnome-look.org/p/1201720/
>
> 将下载的文件放到：`~/.local/share/plank/theme/`目录下。
>
> 在plank的preference选用主题即可。

### 0x02 Zsh

查看系统已安装的shell有哪些：`cat /etc/shells`

查看系统当前shells：`echo $SHELL`

安装zsh：

> - apt-get install zsh

安装zsh配置框架oh-my-zsh：`wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O - | sh`

> - 在以 root 用户为前提下，oh-my-zsh 的安装目录：/root/.oh-my-zsh
> - 在以 root 用户为前提下，Zsh 的配置文件位置：/root/.zshrc
> - 为 root 用户设置 zsh 为系统默认 shell：`chsh -s /bin/zsh root`
> - 如果你要重新恢复到 bash：`chsh -s /bin/bash root`

zsh配置

主题配置：主题文件所在目录为`~/.oh-my-zsh/themes`。

> - ls ~/.oh-my-zsh/themes | wc -l
> - 主题介绍：https://github.com/robbyrussell/oh-my-zsh/wiki/Themes

修改主题：`vim ~/.zshrc`

> - 找到：ZSH_THEME="robbyrossell"改为自己喜欢的，默认是robbyrossell，想随机就使用random。

假设使用agnoster主题：

- ZSH_THEME="agnoster"
- 会出现箭头乱码，是因为没有安装powerline字体：https://github.com/powerline/fonts

> #克隆
> git clone https://github.com/powerline/fonts.git --depth=1
>
> #安装
> cd fonts
> ./install.sh
>
> #安装完成后可删除文件
> cd ..
> rm -rf fonts

插件设置：插件文件所在目录为`~/.oh-my-zsh/plugins`

> - ls ~/.oh-my-zsh/plugins | wc -l
> - 插件介绍：https://github.com/robbyrussell/oh-my-zsh/wiki/Plugins

启用插件：`vim ~/.zshrc`

> - 找到plugins=(git)，括号（）中即要要启用的插件，默认启用了git，多个用空格分开。
> - 修改启用插件后让配置文件生效：`source ~/.zshrc`

对于zsh没有自带的插件，可以将其下载放到：~/.oh-my-zsh/custom/plugins目录下，然后再在配置文件中启用。

例如：

> #安装外部插件：zsh-syntax-highlighting，作用是zsh语法高亮。
> cd  ~/.oh-my-zsh/custom/plugins
> git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
>
> #安装zsh-autosuggestions
> cd  ~/.oh-my-zsh/custom/plugins
> git clone https://github.com/zsh-users/zsh-autosuggestions.git

### 0x03 conky

安装conky：

> - sudo apt-get install conky-all

运行conky时，conky默认会使用`~/.conkyrc`作为配置文件。

conky主题：

> - https://www.deviantart.com/customization/skins/linuxutil/applications/conky/newest/?offset=0
> - https://www.deviantart.com/custom-linux/gallery/39357745/Conky-Themes
> - 肥宅：https://www.deviantart.com/justhumans404/art/Natsuki-conky-ddlc-721176434>
> - 简约：https://github.com/mr-mierzejewski/conkyrc

### 0x04 基本工具软件安装

1.安装shadowsocks-qt5

> 说明：shadowsocks-qt5是ubuntu上一个可视化的版本
>
> Ubuntu 18.04出现无法安装ss-qt5：https://blog.csdn.net/a912952381/article/details/81172971

```
sudo add-apt-repository ppa:hzwhuang/ss-qt5
sudo apt-get update
sudo apt-get install shadowsocks-qt5
```

2.安装搜狗输入法：

> https://jingyan.baidu.com/article/e75057f20d9521ebc91a890a.html
>
> https://www.jianshu.com/p/01ffac435ea8

3.安装Chrome：

> https://www.google.cn/intl/zh-CN/chrome/

4.安装JAVA

> - 安装Oracle Java：https://tecadmin.net/install-oracle-java-11-ubuntu-18-04-bionic/
> - 安装OpenJDK：`sudo apt-get install openjdk-8-jdk`
>
> 注：
>
> ​    Oracle Java是Oracle官方的Java版本，但apt方式只能安装Java 11。OpenJDK Java是开源的Java版本。
>
> ​    当系统安装了多个Java版本时，可以使用命令切换默认使用的Java：`sudo update-alternatives –config java`
>
> ​    使用`java -version`查看当前系统默认使用的Java版本信息。

5.Misc：

```
sudo apt install 

proxychains            #代理工具
meld                   #文件比较工具
amule                  #电驴下载工具
transmission           #种子下载工具
ttf-wqy-microhei       #字体
mtr                    #Traceroute工具（GUI）
whois                  #域名信息查询工具
git                    #版本管理工具
curl                   #命令行HTTP请求工具
obs-studio             #录屏、直播软件
unrar unrar-free       #rar压缩包解压
woeusb				   #刻录工具
ascii                  #显示ascii表
unicode                #字符对应的Unicode编码
axel                   #命令行下载工具（类似wget）
screefetch             #打印banner 
ubuntu-restricted-extras #视频解码器包
libavcodec-extra         #视频解码相关
libdvd-pkg               #dvd格式视频解码器
```

6.安装Tmux

> - https://www.cnblogs.com/wangqiguo/p/8905081.html
> - https://www.cnblogs.com/kevingrace/p/6496899.html
> - https://www.cnblogs.com/maoxiaolv/p/5526602.html

7.Motrix：开源、跨平台，支持下载 HTTP、FTP、BT、磁力链、百度网盘。

> - https://github.com/agalwood/Motrix/releases/tag/v1.3.8
> - https://motrix.app/

### 0x05 常用快捷键

```
Win + a          #显示所有已安装的程序

PrtSc            #全屏截屏
Shift + PrtSc    #截屏选定范围


Ctrl + Alt + t      #打开一个终端
Ctrl + Shift + t    #在已打开终端情况下，会打开一个新的tab终端
Ctrl + Shift + n    #在已打开终端情况下，会打开一个新的独立窗口的终端
Ctrl + Shift + q    #关闭该终端，所有Tab终端都将被关闭
Ctrl + Shift + w    #只关闭当前tab终端。

Ctrl + d            #显示桌面

Ctrl + t            #打开新页签
Ctrl + q/w          #关闭程序/页签
Win + 上下左右方向键  #调整程序窗口位置

Win + h             #最小化当前程序窗口

Win + l             #锁屏
```



