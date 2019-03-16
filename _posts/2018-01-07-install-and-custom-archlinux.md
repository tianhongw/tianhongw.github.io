---
layout: post
title: ArchLinux折腾小记
date: 2018-01-07
tags:
    - archlinux
description: 安装archlinux，安装后的美化，以及一些软件的配置。
---

## Why ArchLinux
--------------------
### windows让我变得很“懒”
安装windows，你只需要插入做好的启动盘，然后鼠标点点点，就可以装好一个系统，可以说确实很”人性化“，然而后面隐藏了很多知识却被忽略了，
比如UEFI和BIOS是什么？两者有什么区别？分区是怎么回事？为什么windows有C盘，A盘和B盘呢？这些问题对于使用windows的人来说不需要了解也能安装和使用windows。windows自带桌面环境，
缺少驱动也有驱动精灵帮你一键搞定。另一点，windows下流氓软件数不胜数，广告满天飞，装软件不小心的话可能捆绑一大堆软件。
还有个重要的原因是我有很多时间都花在游戏上了，种种原因让我放弃windows。
### Linux版本
第一次尝试Linux是Ubuntu，Ubuntu帮用户做了很多贴心的工作，良好的驱动支持，巨大的软件仓库，丰富的论坛资源，对于第一次接触Linux的小白很友好，使用了一段时间后，
觉得Ubuntu也有些缺点，安装系统后琳琅满目的软件似乎有点windows的味道，`apt-get`的依赖关系太复杂，还有版本升级的一些问题。所以我决定换个更~~装X~~适合我的Linux。
在网上搜索之后，Gentoo和Arch似乎是个不错的选择，考虑到我的Linux知识还很薄弱，所以选择了ArchLinux。
<!--more-->
## 安装
-----------------------------
### 分区
相比于MBR分区，GPT分区更灵活、更先进，而且GPT分区更适合于UEFI系统，所以针对我的电脑，我选择UEFI+GPT，UEFI引导要求硬盘必需有[EFI系统分区](https://wiki.archlinux.org/index.php/EFI_System_Partition)，最终我的分区方案是：

|分区|挂载点|大小|
|:---|:---|---:|
|EFI系统分区|/mnt/boot|512M|
|----
|/|/mnt|100G(Wiki推荐15-20G)|
|----
|/home|/mnt/home|剩下所有硬盘空间|

swap分区不是必需的，而且在系统安装完成后，通过建立[交换文件](https://wiki.archlinux.org/index.php/Swap)，达到同样的目的。
假设是一块新的硬盘或者想重新分区，以root身份：
```
gdisk -l    #查看设备，比如我的硬盘被系统识别为/dev/sda
gdisk /dev/sda
```
### 建立分区：
1.  使用o命令建立一个新的空 GUID 分区表。
2. 使用n 命令创建一个新的分区。
3. 保证 2048 扇区对齐，gdisk默认从2048扇区开始。
4. 使用 +x{M,G} 的格式指定分区大小为 x MB 或 GB。
5. 选择分区类型 ID。（EFI分区代码：ef00，其余采用默认值8300）
6. 其他分区的处理方式类似。
7. 使用 w 命令将分区表写入磁盘并退出。
8. 将新分区格式化为文件系统。

### 格式化分区：
```shell
mkfs.fat -F32 /dev/sd1  #格式化EFI分区
mkfs.ext4 /dev/sda2 #格式化/dev/sda2
mkfs.ext4 /dev/sda3 #格式化/dev/sda3
```
### 挂载
```shell
mount /dev/sda2 /mnt    #首先将根分区挂载到/mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot   #重要：把EFI分区挂载到/mnt/boot
mkdir /mnt/home
mount /dev/sda3 /mnt/home
```
### 安装
具体的安装步骤参考[WIKI](https://wiki.archlinux.org/index.php/Installation_guide)。
```shell
pacstrap -i /mnt base base-devel
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc --utc
vi /etc/locale.gen  #移除en_US.UTF-8 UTF-8、zh_CN.UTF-8 UTF-8、zh_TW.UTF-8 UTF-8前面的注释符号
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
echo myhostname > /etc/hostname     #设置主机名
pacman -S iw  wpa_supplicant dialog
passwd  #设置密码
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub
grub-mkconfig -o /boot/grub/grub.cfg
exit
umount -R /mnt
reboot
```
## 安装后的工作和美化
--------------------
### 创建用户
```shell
useradd -m -g users -G wheel -s /bin/zsh username
passwd username
```
为了使普通用户使用`sudo`也能执行系统命令，并且不需要每次输入密码，切换到root，执行
```shell
visudo
```
取消`%wheel ALL=(ALL) NOPASSWD: ALL`前面的注释。
### 图形界面
为了使用图形界面，安装一些必要的软件包和驱动。
```shell
sudo pacman -S xorg
sudo pacman -S xf86-video-intel    #intel核显驱动
```
### 桌面环境
Linux下常见的桌面环境比如说GNOME和KDE，还有其他很多轻量级的桌面环境，其中xfce4是我比较喜欢的一款，可以配置得很漂亮。
```shell
sudo pacman -S xfce4 xfce4-goodies
```
### 终端主题
根据个人喜好，在[这里](https://github.com/netzverweigerer/xfce4-terminal-colorschemes)可以选择不同终端主题。
### 开启AUR
```
sudo vi /etc/pacman.conf
```
选择清华源，在末尾添加：
```
[archlinuxcn]
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```
之后安装archlinuxcn-keyring包导入GPG key.AUR的包管理可以选择yaourt或者yay。
如果你想运行一些32位程序，编辑/etc/pacman.conf，取消注释:
```
[multilib]
Include = /etc/pacman.d/mirrorlist
```
### 美化
#### 1.字体
Source Code Pro 是Adobe开发的一款开源字体，非常适合阅读代码。
```
sudo pacman -S adobe-source-code-pro-fonts
sudo pacman -S ttf-dejavu wqy-zenhei
```
#### 2.pulseaudio
```
yaourt pulseaudio    #选择pavucontrol和pulseaudio-alsa
```
#### 3.whisker
```
yaourt whisker    #选择xfce4-whiskermenu-plugin
```
之后添加pulseaudio和whisker menu到panel上.
#### 4.docky
```
sudo pacman -S docky
```
docky用来替代xfce4自带的panel。
#### 5.窗口透明度
右键`Applications->Settings->Window Manager Tweaks->Compositor`设置窗口透明度。
#### 6.编辑panel
编辑panel添加一些新的item，比如:Audio Mixer、DateTime、Weather Update等。
#### 7.主题、icon
```
yaourt numix-frost
yaourt paper-icon-theme-git
```
可以在[xfce-look](https://www.xfce-look.org/)找到更多主题。手动安装主题的方法是把主题文件copy到`/usr/share/icons/或/usr/share/themes/`中，然后右键`Applications->Settings->Apperance`设置主题。
#### 8.Background
把图片copy到`/usr/share/backgrounds/xfce/`避免不小心删掉。
#### 9.快捷键
右键`Applications->Settings->Keyboard`设置快捷键。例如设置打开终端快捷键:`xfce4-terminal --maximize --command tmux   Ctrl+Alt+T`。
#### 10.终端透明
打开终端`Edit->Preferences->Apperance`将Background设置为Transparent background，透明度设置为0.75。
#### 11.conky
```
sudo pacman -S conky conky-manager
```
在[这里](https://www.deviantart.com/?section=&global=1&q=conky+theme&offset=0)可以找到许多conky主题。
#### 12.List of installed packages(备份从官方仓库获取的软件列表)
```
comm -23 <(pacman -Qeq|sort) <(pacman -Qmq|sort) > pkglist
# to install packages from pkglist.txt, run:
pacman -S - < pkglist.txt
```
## 软件（不定时更新）
-----------------------------
### 1.vim
```
sudo pacman -S vim
```
配置文件可以用[spf13-vim](https://github.com/spf13/spf13-vim)
不知道什么原因（可能是网络导致），下载spf13-vim后用vim打开某些文件会出现:**E411: highlight group not found: Normal**，而且错误提示的红色高亮特别刺眼。检查后才发现vimrc内默认使用的是`solarized`配色，但是`~/.vim/bundle/`里没有`vim-colors-solarized`文件，装了这个插件后解决了上面提到的问题。update:已经放弃使用spf13的配置，推荐根据自己的需求来配置。
### 2.zsh
```
sudo pacman -S zsh
```
配置文件可以用[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)
### 3.shadowsocks
```
sudo pacman -S shadowsocks
sudo cp /etc/shadowsocks/config.json /etc/shadowsocks/foo.json
```
编辑foo.json，需要修改3处：
```
{
    "server":"my_server_ip",    #你的服务器ip
    "server_port":8388,         #你的shadowsocks服务端口
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",    #密码
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false,
    "workers": 1,
    "prefer_ipv6": false
}
```
设置开机运行shadowsocks服务:
```
systemctl start shadowsocks@foo
systemctl enable shadowsocks@foo
```
Chrome浏览器配合SwitchyOmega插件科学上网。
### 4.输入法
```
sudo pacman -S fcitx fcitx-googlepinyin
```
### 5.zathura PDF阅读器
```
sudo pacman -S zathura zathura-pdf-poppler

```
### 6.视频/音乐播放器
```
sudo pacman -S mpv
sudo pacman -S netease-cloud-music
```
### 7.IRC客户端
```
sudo pacman -S weechat
/server add freenode chat.freenode.net  #添加server
/set irc.server.freenode.autoconnect on #自动连接到freenode
/set irc.server.freenode.command "/msg nickserv identify youpasswd" #自动认证
/set irc.server.freenode.autojoin "#channel1,#channel2" #自动加入群组
```
### 8.Markdown编辑器
[Marker](https://github.com/fabiocolacio/Marker)
## 尾
---------------------------
最后附一张简单配置好的ArchLinux:
![](/assets/images/install-and-custom-archlinux.png)
