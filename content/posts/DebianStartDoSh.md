---
title: Debian开机自动执行文件
date: 2019-01-14T11:20:37+08:00
tags: [Debian,Linux]
categories: ["Linux"]
---

创建启动sh脚本文件,输入以下内容
``` bash
sleep 20    #延时执行
#下面是要执行的信息
sh /mnt/harddisk/stopMysql.sh && sh /mnt/harddisk/startAria2.sh
```

把启动脚本增加到启动文件
``` bash
nano /etc/rc.local

#在 exit 0 前面增加以下一行
nohup /root/openMachineStart.sh &
```