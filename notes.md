---
layout: page
title: Notes
---
### 1.解决fcitx黑框问题。

安装`compton`,拷贝配置文件`cp /etc/xdg/compton.conf ~/.config/compton.conf`,在个人配置文件中令`shadow = false`,在`rc.lua`中加入`awful.spawn.with_shell("compton -b")`

### 2.evil-mode下org-mode中`TAB`键不起作用问题。

`(evil-define-key 'normal org-mode-map (kbd "TAB") 'org-cycle)`

### 3.更新python3.7后youcompleteme报错

在vim中输入`:messages`查看错误信息，提示:
`ImportError: libpython3.6m.so.1.0: cannot open shared object file`

创建一个软连接解决了问题:
`sudo ln -s /usr/lib/libpython3.7m.so.1.0 /usr/lib/libpython3.6m.so.1.0`

或者更新下YCM插件。

### 4.UEFI启动下双硬盘双系统(windos\archlinux)grub引导问题。

把windos的EFI分区挂载在/mnt下，安装`os-prober` 重新生成grub配置，grub会自动检测到windos。

### 5.Archlinux自动挂载硬盘。  

编辑`/etc/fstab` 根据实际情况更改设备名和挂载点:

`/dev/sda1   /home/tianhongw/Hdd ext4    rw,noatime,noauto,x-systemd.automount,users 0 0`

### 6.Archlinux有线连接网络

需要启动两个服务，根据实际情况更改网卡名:

`dhcpcd@eno1.service`

`dhclient@eno1.service`

### 7.用docker运行jekyll

进入博客网站所在目录，运行:

`docker run --name blog -p 4000:4000 --volume="$PWD:/srv/jekyll" -it jekyll/jekyll jekyll serve`

### 8.日志命名技巧

`cmd.sh > log_file_name.$(date +\%Y\%m\%d_\%H\%M\%S).log 2>&1`
