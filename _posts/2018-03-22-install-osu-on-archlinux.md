---
layout: post
title: Install osu! on archlinux
date: 2018-03-22
tags:
    - archlinux
    - osu
description: The way to play osu! on archlinux.
---

## Wine
-----------------
关于wine需要搞清楚`WINEPREFIX`这个概念，`WINEPREFIX`相当于一个windows的系统目录，也就是说你安装的所有windows库、运行软件时需要的依赖都在这个文件夹中。
wine的强大之处在于你可以创建多个这样的系统目录，相当于你同时拥有多个系统。64位系统运行wine时默认的是64位windows环境，可以通过`WINARCH`和`WINEPREFIX`这
两个变量来指定系统位数和系统目录的位置。比如：
```
WINEARCH=win32 WINEPREFIX=~/win32 winecfg
WINEARCH=win64 WINEPREFIX=~/win64 winecfg
```
这样就创建了32位和64位windows系统，系统目录分别位于`~/win32`和`~/win64`
- 安装一个32位程序`WINEARCH=win32 WINEPREFIX=~/win32 wine 32installer.exe`
- 运行一个32位程序`WINEARCH=win32 WINEPREFIX=~/win32 wine 32.exe`
- 安装一个64位程序`WINEARCH=win64 WINEPREFIX=~/win64 wine 64installer.exe`
- 运行一个64位程序`WINEARCH=win64 WINEPREFIX=~/win64 wine 64.exe`

## 安装osu!
------------------
```
sudo pacman -S wine wine_gecko wine-mono winetricks #wine
sudo pacman -S lib32-alsa-lib lib32-libpulse lib32-openal lib32-mpg123 #64位关于声音可能需要的库
WINEARCH=win64 WINEPREFIX=~/win64 winecfg
WINEARCH=win64 WINEPREFIX=~/win64 winetricks dotnet40 #osu依赖库
```

然后去osu官网[下载](https://osu.ppy.sh/p/download)安装文件，放在你想要安装osu的位置，执行

```
WINEARCH=win64 WINEPREFIX=~/win64 wine osu\!install.exe`
```

安装就到此结束了，其它的windows程序安装过程类似，需要的windows库不同罢了。遗留的问题是我这样安装之后
osu一直提示我验证帐号，导致无法登录，只能离线打图了，这个问题待解决。
**update:**这个问题解决了，具体参考<https://osu.ppy.sh/forum/t/658601>
## 尾
-----------------------
为了启动方便，可以在.zshrc里加入一些alias。比如:

```
alias osu="wine ~/win64/drive_c/users/kotiyasanae/Downloads/osu!/osu\!.exe >/tmp/out.file 2>&1 &"
```

之所以直接用`wine`是因为在.zshrc里加入了：

```
export WINEPREFIX=~/win64
export WINEARCH=win64
```

这样就不用每次指定前缀了。
## 参考
----------------------------------
- [ArchWiki](https://wiki.archlinux.org/index.php/Wine_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#.E5.A3.B0.E9.9F.B3)
