---
title: Debian开机自动挂载U盘
date: 2019-01-14T10:50:37+08:00
tags: [Debian,Linux]
categories: ["Linux"]
---

查看U盘(移动硬盘)的分区UUID
``` bash
#查看所有
ls -l /dev/disk/by-uuid/ 

#查看指定
blkid /dev/sda1
```

根据UUID增加挂载分区,挂载到 /mnt/harddisk 文件夹
``` bash
#编辑启动文件
nano /etc/fstab

#在最后增加以下行数
UUID=A8987C2D987BF7E0 /mnt/harddisk auto defaults,noatime,rw,nofail,x-systemd.automount 0 0
```