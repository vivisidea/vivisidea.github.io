---
title: 'ubuntu - the symbol "grub_xputs" not found'
author: vivi
layout: post
permalink: /posts/112.html
categories:
  - 技术记录
tags:
  - grub
  - ubuntu
---
ubuntu 被玩坏了，更新重启之后出现

> the symbol "grub_xputs" not found

错误，比较雷的是这个 grub rescue 连 help 都没有。。还好有个 iPad 可以上网，找到如下解决方法

```
grub rescue> ls <-- 注0
grub rescue> set root=(hdx, y) <-- 注1
grub rescue> insmod /boot/grub/linux16.mod
grub rescue> linux16 /vmlinuz root=/dev/sda ro <-- 注2
grub rescue> initrd16 /initrd.img
grub rescue> boot
```

- 注0： 大概会输出 `(hd0) (hd0,1) (hd0,5)` 为下一步准备的，没什么用 
- 注1： 确定 root 分区，方法是分别尝试执行 `ls (hdx, y)/` 看到出来的是 `/` 下的文件列表就是了 
- 注2： 这里的 sda 是对应的硬盘，如果是 usb 可能会是 sdb 之类的

进入系统之后的处理

- [找个工具省事](https://help.ubuntu.com/community/Boot-Repair)
- 自己执行命令，重新安装 grub（自己未测，不知道是否还需要其他步骤）

```
sudo grub-install /dev/sda <-- 选择自己对应的硬盘设备
```

参考

[http://ubuntuforums.org/showthread.php?t=1195275](http://ubuntuforums.org/showthread.php?t=1195275)

[http://bit.ly/w2qhCc](http://bit.ly/w2qhCc)

以上记录。
