---
layout: post
title: "Manjaro 安装与使用教程"
subtitle: "欢迎使用Linux系统"
date: 2019-01-31 01:10:00
author: "Echcz"
header-img: "img/in-post/2019-01-31-manjaro-install-use.jpg"
catalog: true
tags:
  - "Manjaro"
  - "Linux"
---

{% raw %}
## Manjaro 介绍

Manjaro是一款基于 Arch Linux 、对用户友好、全球排名第1的Linux发行版。其继承了 Arch Linux 的所有优点：

* 滚动更新可以使软件保持最新
* AUR软件仓库有着世界上最齐全的Linux软件
* 丰富的wiki和活跃的社区让所有问题都可以快速得到满意的答案

并且同时注重用户友好性和可用性。

Manjaro提供32位和64位版本，适合新手以及经验丰富的Linux用户。对新手提供了一个用户友好的安装程序，并且系统本身设计为完全“直接开箱”，包括：

* 预安装的桌面环境
* 预安装的图形应用程序，轻松安装软件和更新您的系统
* 预先安装的编解码器播放多媒体文件

不仅如此，Manjaro还拥有自己的一些额外的特性：

* 从自己独立的存储库中汲取软件，以确保提供完全测试过的稳定的软件包。这些存储库还包含非Arch提供的软件包
* 提供本发行版特有的工具，如Manjaro硬件检测（mhwd）实用程序和Manjaro设置管理器（msm）
* 为系统自动安装必要的软件（例如显卡驱动程序）
* 轻松安装和使用多个内核

可以说，Manjaro 绝对是一头猛兽： Manjaro 提供了Arch操作系统的所有优势。不仅快速、强大、始终保持最新，还特别强调了对新手及高级用户的稳定性、用户友好性和可用性。

## 使用U盘安装 Manjaro

### 安装前的准备

1. 下载 Manjaro ISO 镜像。[Manjaro 官方下载页](https://manjaro.org/download/) 提供了多种不同桌面环境的版本，选择自己的喜欢的进行下载。对于国内下载慢的同学可以到 [USTC 镜像站点](http://mirrors.ustc.edu.cn/manjaro-cd/) 下载或使用种子下载。

2. (Windows用户) 使用 [Rufus](https://rufus.ie/) 制作 Manjaro 启动U盘，注意：需要使用`DD模式`。如果是Linux用户，可以使用DD命令进行制作，教程可以参考 [CSDN-时梦-Linux下使用dd命令制作启动盘](https://blog.csdn.net/qq_37968132/article/details/82918925) 。注意： 这步会清除U盘里的内容，请先备份你的U盘里的数据。

3. 将启动盘插入电脑，开机时选择U盘启动，进入 Manjaro 安装界面。

### 安装 Manjaro

Manjaro 提供了相当友好的安装界面，安装过程并不困难，只是有以下注意点:

* 在开始界面时，如遇到时区问题，请将 tz(时区) = Asia/Shanghai。
* 在开始界面时，推荐将driver(驱动) = nofree (使用闭源驱动)。
* 如想安装 Windows 10(已安装) + Manjaro 双系统，请选择手动分区。由于 Windows10 本身的引导分区就是 ESP 分区，所以到分区的界面，只需要将 /boot/efi (标记是boot和esp)挂载到 ESP 分区(也就是约100M大小的分区)即可。
* 在安装时请保持网络连接。

## 配置 Manjaro

### 配置国内源

``` shell
~ $ sudo pacman-mirrors -i -c China -m rank # 可以多选，推荐勾选 USTC 源
```

### 升级系统

``` shell
~ $ sudo pacman -Syy && sudo pacman -Syu
```

### 添加 Archlinuxcn 源

``` shell
~ sudo mousepad /etc/pacman.conf
# 在文件末尾添加以下内容
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch

# 安装 archlinuxcn签名钥匙，用于导入 GPG key，否则key验证失败，将导致无法安装软件
~ $ sudo pacman -Syy && sudo pacman -S archlinuxcn-keyring
```

### 配置 I+N 双显卡驱动

Manjaro 自己安装的驱动有问题，独显会锁死60FPS，需要自己重装

先到 设置 > Manjaro Settings Manager > 硬件设定中卸载原有驱动，再执行以下命令：

``` shell
~ $ sudo pacman -S virtualgl lib32-virtualgl lib32-primus primus # 安装依赖
~ $ sudo mhwd -f -i pci video-hybrid-intel-nvidia-bumblebee # 请使用此命令安装驱动，不要使用`硬件设定`
~ $ sudo systemctl enable bumblebeed.service # 允许服务
~ $ sudo gpasswd -a $USER bumblebee # 添加当前用户到 bumblebee 用户组
~ $ reboot # 重启
```

> *注意：此项配置可能导致电脑黑屏，如不幸遭遇，请自行百度*

### 配置时间和日期

在 设置 > Manjaro Settings Manager > 时间和日期 中勾选 `自动设置时间和日期` 和 `本地时区的硬件时钟`，并且注意时区应是 `Asia/Shanghai`，国家应是 `China`

### 安装配置 Git

``` shell
~ $ sudo pacman -S git # 系统默认已安装，如果没有 Git 请执行此命令
~ $ git config --global user.name "YourName" # 设置 git 用户名
~ $ git config --global user.email "YourEmail@example.com" # 设置 git 用户邮箱
```

### 生成 SSH key

``` shell
~ $ ssh-keygen -t rsa -C "YourEmail@example.com" # -C 指定注释，用于方便用户标识，推荐使用你的邮箱地址
# 接着程序会提示：请输入密语(passphrase)，空表示没有密语。空就行了，直接按回车
# 接着程序会提示：请输入密码(password)，空表示没有密码。空就行了，直接按回车
# 接着程序会提示：请确认密码，再输一次密码后按回车
```

这将在 `~/.ssh` 目录下生成 `id_rsa`(私钥) 和 `id_rsa.pub`(公钥)，可以将公钥发布给支持公私钥进行身份认证的站点(比如 Github)，进行身份认证

### 安装 yay

yay 是 AUR 助手，兼容 pacman，以后可以使用 yay 安装软件

``` shell
~ $ sudo pacman -S yay
```

### 语言设定

在 设置 > Manjaro Settings Manager > 本地化设定 加入 `zh_CN.UTF-8`，在 详细设置 中将所有项目的显示语言都设为 `zh_CN.UTF-8`

在 设置 > Manjaro Setting Manager > 语言包 中安装语言包

在 设置 > Manjaro Notifier Settings 中勾选 `检查缺少的语言包`。如果需要安装语言包，请安装

### 安装中文输入法

这里推荐中州韻输入法，关于中州韻的使用教程请访问 [中州韻官网](https://rime.im/)

``` shell
~ $ yay -S fcitx-im # 安装fcitx 选择全部安装
~ $ yay -S fcitx-configtool # fcitx 配置界面
~ $ yay -S fcitx-rime # 安装中州韻输入法
# yay -S fcitx-sogoupinyin # 当然也可以安装搜狗拼音输入法了
```

安装完之后需要配置环境，以启用中文输入，重启后生效：

``` shell
sudo mousepad ~/.xprofile # 编辑 .xprofile 文件
# 在文件中加入以下代码
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

### 修改 Home 目录下的用户目录名为英文

``` shell
~ $ mousepad ~/.config/user-dirs.dirs
# 修改为以下内容
XDG_DESKTOP_DIR="$HOME/Desktop"
XDG_DOWNLOAD_DIR="$HOME/Download"
XDG_TEMPLATES_DIR="$HOME/Templates"
XDG_PUBLICSHARE_DIR="$HOME/Public"
XDG_DOCUMENTS_DIR="$HOME/Documents"
XDG_MUSIC_DIR="$HOME/Music"
XDG_PICTURES_DIR="$HOME/Pictures"
XDG_VIDEOS_DIR="$HOME/Videos"
# 再将 Home 目录下的用户目录名改为对应的英文名
~ $ cd ~
~ $ mv 公共 Public
~ $ mv 模板 Templates
~ $ mv 视频 Videos
~ $ mv 图片 Pictures
~ $ mv 文档 Documents
~ $ mv 下载 Download
~ $ mv 音乐 Music
~ $ mv 桌面 Desktop
```

### 中文字体安装

``` shell
~ $ yay -S --noconfirm wqy-microhei && fc-cache -fv
# 其它文泉驿字体
~ $  sudo pacman -S wqy-microhei-lite
~ $  sudo pacman -S wqy-bitmapfont
~ $  sudo pacman -S wqy-zenhei
# 选用字体
~ $  sudo pacman -S adobe-source-han-sans-cn-fonts
~ $  sudo pacman -S adobe-source-han-serif-cn-fonts
~ $  sudo pacman -S noto-fonts-cjk
```

### 配置应用程序启动器，桌面，面板

应用程序启动器菜单(Windows 下对应开始菜单项目)位于 /usr/share/applications ， /usr/local/share/applications 和 ~/.local/share/applications 目录，修改其中的文件即可更改。也可以 右击应用程序启动器(Windows 下对应程序快捷方式) > 编辑应用程序启动器进行修改(这是无法删除的)

桌面图标可以通过在 ~/Desktop 目录下添加 .desktop 文件添加，或 右击应用程序启动器 > 添加到桌面 添加

右击应用程序启动器 > 添加到面板 添加到面板(Windows 下对应任务栏)，右击 面板上的图标可以对图标进行修改

### 快捷键设定

**应用程序启动快捷键**设定请到 设置 > 键盘 > 应用程序快捷键 中设定。关于应用程序启动的命令可以通过 右击应用程序启动器 > 编辑应用程序启动器 查得

**窗口行为快捷键**设定请到 设置 > 窗口管理器 > 键盘 中设定

**输入法快捷键**设定请 打开`Fcitx配置` > 全局配置 进入配置，如需更细致的设定，请勾选`显示高级选项`

## 常用软件安装

软件安装可以使用 `sudo pacman -S` 或 `yay -S` 命令 或使用 `pamac-manager`(`添加或删除软件`) 程序

推荐打开 `pamac-manager` 的 AUR 支持： 首选项 > AUR > 启用 AUR 支持

以下是常用软件列表：

| 名称 | 程序名 | 说明 | 备注 |
|:--:|:-- |:-- |:-- |
| zsh | zsh | 超好用的 shell | [文档](https://github.com/robbyrussell/oh-my-zsh/wiki) ; [segmentfault-MichaelXoX-zsh+on-my-zsh配置教程指南](https://segmentfault.com/a/1190000013612471) |
| 红移 | redshift | 在指定时间段自动调节屏幕色温 | [Archlinux-wiki-Redshift](https://wiki.archlinux.org/index.php/Redshift) |
| Catfish | catfish | 多功能文件搜索工具 |  |
| Grsync | grsync | 文件/文件夹同步软件 |  |
| shutter | shutter | 功能强大截屏软件 |  |
| GoldenDict | goldendict | 词典 | [中文词典下载](http://download.huzheng.org/zh_CN/) |
| Shadowsocks-Qt5 | shadowsocks-qt5 | Sock v5 客户端，用于配置代理 |  |
| google-chrome | google-chrome | 谷歌浏览器 |  |
| Thunderbird | thunderbird | 邮件或新闻阅读软件 | 系统默认已安装 |
| aria2 | aria2 | 下载器，支持 HTTP(S)，FTP，SFTP，BitTorrent和Metalink 协议 | [Aria2 Manual](https://aria2.github.io/manual/en/html/aria2c.html) ; [Archlinux-wiki-aria2](https://wiki.archlinux.org/index.php/aria2) ; [简书-zephyrous-Aria2配置详解](https://www.jianshu.com/p/6adf79d29add)  |
| uGet | uget | 强大的下载软件，比迅雷更强，推荐搭配 aria2 使用 | uGet 使用 aria2：编辑 > 设置 > 插件 > 插件匹配顺序：aria2; URI：http://localhost:6800/jsonrpc；参数：--enable-rpc=true -D --disable-ipv6 --check-certificate=false |
| FileZilla | filezilla | FTP，FTPS，SFTP 文件上传下载器 |  |
| GUN 图像处理程序 | gimp | 相当于 Windows 下的 Photoshop | [官方文档](https://docs.gimp.org/2.10/zh_CN/) |
| Peek | peek | 简单易用的屏幕录制器 |  |
| VLC | vlc | 多媒体播放器，支持超多格式 |  |
| 网易云音乐(非官方) | electron-netease-cloud-music | 比官方版更好的网易云音乐客户端 |  |
| masterpdfeditor | masterpdfeditor | PDF阅读编辑器 |  |
| VSCode | visual-studio-code-bin | 免费开源的现代化轻量级代理编辑器，微软的良心之作 |  |
| LibreOffice | libreoffice-fresh | 相当于 Windows 下的 MS Office |  |
| IDEA | intellij-idea-community-edition | 强大的 IDE，不用多说了吧 | 有钱(注册码)的朋友也可以安装旗舰版：intellij-idea-ultimate-edition |
| xmind-zen | xmind-zen |  |  |
| dbeaver | dbeaver | 通用数据库客户端，支持多个平台及多种数据库 |  |
| TIM | deepin.com.qq.office | 通过 `Wine` 移植过来的 |  |
| 微信(非官方) | electronic-wechat | 比官方版更好的微信客户端 |  |

## 参考链接

* 学习 Linux 可以访问 [鸟哥的 Linux 私房菜](http://linux.vbird.org/)
* 更多关于 Manjaro 可以访问 [Manjaro 官网](https://manjaro.org/)
* Manjaro 是 Archlinux 的发行版，想要用好他，你当然需要访问 [Archlinux-wiki](https://wiki.archlinux.org/)

{% endraw %}