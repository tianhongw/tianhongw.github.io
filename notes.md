---
layout: page
title: Notes
---
1. 解决fcitx黑框问题。
安装`compton`,拷贝配置文件`cp /etc/xdg/compton.conf ~/.config/compton.conf`,在个人配置文件中令`shadow = false`,在`rc.lua`中加入`awful.spawn.with_shell("compton -b")`
2. evil-mode下org-mode中`TAB`键不起作用问题。
`(evil-define-key 'normal org-mode-map (kbd "TAB") 'org-cycle)`
3. 更新python3.7后youcompleteme报错
在vim中输入`:messages`查看错误信息，提示:
`ImportError: libpython3.6m.so.1.0: cannot open shared object file`
创建一个软连接解决了问题:
`sudo ln -s /usr/lib/libpython3.7m.so.1.0 /usr/lib/libpython3.6m.so.1.0`
或者更新下YCM插件。
4. UEFI启动下双硬盘双系统(windos\archlinux)grub引导问题。
把windos的EFI分区挂载在/mnt下，安装`os-prober` 重新生成grub配置，grub会自动检测到windos。
