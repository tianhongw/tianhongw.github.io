---
layout: post
title: awesomewm + urxvt 一个日常使用Linux的轻量环境方案
date: 2018-05-31
tags: 
    - awesomewm
    - urxvt
description: 简单介绍平铺式窗口管理器awesome和一个轻量的终端模拟器urxvt.
---

## awesome window manager
---

### 简介
awesome是一个高度可定制的用于X下的平铺式窗口管理器，
使用Lua进行扩展。关于窗口管理器(window manager)和桌面环境(desktop environment)的区别，
可以简单理解为窗口管理器是桌面环境的子集，窗口管理器负责绘制窗口的边框、最大/小化窗口等工作，桌面环境则使用窗口管理器管理窗口，并且提供一系列软件构成一个完整的工作环境。
举例来说，xfce4属于桌面环境，它默认使用xfwm4作为窗口管理器。毫无疑问你可以在日常使用中选择只安装窗口管理器，甚至两者都不安装在tty下“裸奔”，
当然这种情况大多存在于服务器上。现在电脑配置有了很大提升，对于使用桌面环境来说绰绰有余，如果你想尝试更轻量的窗口管理器，那么awesome是个不错的选择。
什么是平铺式窗口管理器？区别于以往我们使用的浮动式窗口管理器，所谓的平铺是指窗口之间不会相互重叠而是自动的调整大小刚好铺满整个屏幕。

### 一些名词
**状态栏**(status bar): 第一次启动awesome后屏幕上方类似于windows任务栏的东西。  
**主菜单**(main menu): 状态栏最左侧有个图标，点击打开就是awesome的主菜单。  
**标签**(tag): 紧接着主菜单的是阿拉伯数字1-9，这是awesome的标签，可以在`rc.lua`中配置，改变标签的名字、数目等。  
**布局**(layout): awesome支持多种布局，比如：
- 平铺(tile) 所有的窗口都平铺显示，布满整个屏幕。平铺模式下屏幕被分为master和stacking两个区域。master中的窗口包含了需要最多关注的窗口(通常这表示master中的窗口会占据更大的屏幕空间)，而stacking区域中包含了其它窗口。
- 浮动(floating) 每个窗口都可以自由的移动和调整大小，就好像普通的窗口管理器一样。
- 最大化(max) 每个窗口全屏显示。
- 放大(magnifier) 当前工作窗口显示在屏幕中央，占据大部分面积，剩下的窗口都处在stacking区域，并放到当前窗口的后面。

**布局切换器**：状态栏最右侧的是布局切换器，显示当前标签的布局，点击可以切换布局，快捷键为`modkey + 空格键`。

### 配置
安装awesome后，首先要设置它为你的默认窗口管理器，可以通过登录管理器或者添加`exec awesome`到你的启动脚本(比如`~/.xinitrc`)。
awesome是开箱即用的(out of box)，默认配置文件位于`~/.config/awesome`，如果文件不存在，创建一个并复制默认的配置文件到该目录中。
默认的配置文件是`/etc/xdg/awesome/rc.lua`。关于awesome更多的配置，参考[archwiki](https://wiki.archlinux.org/index.php/Awesome)或者[gentoowiki](https://wiki.gentoo.org/wiki/Awesome)，在文章结尾也给出了我正在使用的配置。

### 快捷键
贴一个常用的快捷键列表，可以在`rc.lua`中配置更多快捷键。**M**代表`modkey`，默认为`mod4`，即`win`键。**S**代表`shift`键，**C**代表`ctrl`键。

|按键|功能|
|:---|:---|
|M-r|运行命令|
|M-x|运行lua代码|
|M-Enter|打开terminal emulator|
|M-w|打开主菜单|
|M-S-q|退出awesomewm|
|M-C-r|重启awesomewm|
|M-m|最大化窗口|
|M-n|最小化窗口|
|M-C-n|恢复窗口|
|M-f|设置当前窗口全屏|
|M-S-c|关闭当前窗口|
|M-Left|上一个tag|
|M-Right|下一个tag|
|M-1…9|切换到tag1…tag9|
|M-C-j|切换到下一个显示器|
|M-C-k|切换到上一个显示器|
|M-Esc|回到上一个tag|
|M-S-j|将当前窗口与下一个窗口交换位置|
|M-S-k|将当前窗口与前一个窗口交换位置|
|M-o|把当前程序发送到下一个显示器中|
|M-h|减少5%的主视窗区的高和宽|
|M-l|增加5%的主视窗区的高和宽|
|M-S-h|增加一个主视窗区|
|M-S-l|减少一个主视窗区|
|M-Space|切换下一种布局|
|M-S-Space|切换到上一种布局|


## 终端模拟器urxvt
---

### 特点
- 多标签功能。通过perl扩展，urxvt支持标签功能，当然这可以通过使用tmux替代。
- 提供C/S模式。`urxvtd`启动一个守护进程，`urxvtc`启动客户端，以达到节省系统资源的目的。同样的，目前电脑配置教高的情况下，正常启动就好了。
- 多语言多字体支持。
- 支持perl扩展

### 配置
urxvt使用用户的Xresources配置。Xresources是用户级别的配置文件，通常位于`～/.Xresources`，使用它来配置X应用客户端的各种参数。比如
- 设置终端偏好(例如终端颜色)。
- 设置DPI，字体大小、字体抗锯齿，和其他X下的字体设置。
- 设置底层X应用的偏好(比如xorg-xclock,xpdf,rxvt-unicode)

关于复制粘贴操作有两种方法：一是通过快捷键：复制`alt-ctrl-c`，粘贴`atl-ctrl-v`，此外，Linux下普遍使用的图形界面均为X11，
而X11支持一种独特的复制粘贴方式，即如果你在另一个程序比如文本编辑器中使用鼠标拖动来高亮一段文字后，不用进行任何操作，
此时选中的内容已经复制到剪贴板中了，随后在rxvt中单击鼠标中键即可将选中内容粘贴到其中，如果鼠标没有中键，
可以同时按下左右键以达到同样的效果，还可以使用Shift+Insert组合键来完成粘贴；反之，从rxvt中向外复制内容同样如此。

## 尾
---
按例上图![](/assets/images/awesomewm-and-urxvt.png)
附上我的配置[https://github.com/Trytwice/dotfiles](https://github.com/Trytwice/dotfiles)

## 参考
---
- [https://wiki.archlinux.org/index.php/Awesome](https://wiki.archlinux.org/index.php/Awesome)
- [https://wiki.gentoo.org/wiki/Awesome](https://wiki.gentoo.org/wiki/Awesome)
- [http://wiki.ubuntu.org.cn/Awesome](http://wiki.ubuntu.org.cn/Awesome)
- [https://wiki.archlinux.org/index.php/rxvt-unicode](https://wiki.archlinux.org/index.php/rxvt-unicode)
- [https://wiki.gentoo.org/wiki/Rxvt-unicode](https://wiki.gentoo.org/wiki/Rxvt-unicode)

