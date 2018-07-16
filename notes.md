---
layout: page
title: Notes
---
1. 解决fcitx黑框问题。  
安装`compton`,拷贝配置文件`cp /etc/xdg/compton.conf ~/.config/compton.conf`,在个人配置文件中令`shadow = false`,在`rc.lua`中加入`awful.spawn.with_shell("compton -b")`  
2. evil-mode下org-mode中`TAB`键不起作用问题。  
`(evil-define-key 'normal org-mode-map (kbd "TAB") 'org-cycle)`

