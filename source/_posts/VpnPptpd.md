---
title: linux 控制台 连接 vpn (PPTPD)
date: 2015-05-10 12:52
tags: [Linux,VPN,PPTPD]
---
其实在ubuntu里面很简单。

1、安装 pptpd

``` bash
$ sudo apt-get install pptp-linux
```

2、就可以直接拨号连接了（下面中文改为相应的参数）

``` bash
sudo pptpsetup --create 名称 --server 地址 --username 账号 --password 密码 --encrypt --start
```

–create后的是创建的连接名称，可以为任意名称;
–server后接的是vpn服务器的IP;
–username是用户名
–password是密码，在这也可以没这个参数，命令稍后会自动询问。这样可以保证账号安全
–encrypt 是表示需要加密，不必指定加密方式，命令会读取配置文件中的加密方式
–start是表示创建连接完后马上连接，如果你不想连，就不写

如果创建好了连接–start选项或者是下次再想连接时，输入命令 “sudo pon 名称”  就可以了，然后 “sudo poff”命令可以 断开当前连接。