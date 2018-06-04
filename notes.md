---
layout: page
title: Notes
---
1. 解决fcitx黑框问题。  
安装`compton`,拷贝配置文件`cp /etc/xdg/compton.conf ~/.config/compton.conf`,在个人配置文件中令`shadow = false`,在`rc.lua`中加入`awful.spawn.with_shell("compton -b")`
