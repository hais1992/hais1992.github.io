---
title: 给Linux 增加sudo命令
date: 2015-02-01 23:52
tags: Linux
---
习惯了 sudo apt-get update ， 然后 某天，发现没有了 sudo 感觉有点不爽快了。

然后第一感觉就是使用 root 用户安装 sudo 包

``` bash
$ apt-get install sudo
```

安装完后我们新建个用户"hais"，并给他加入sudo的权限

``` bash
$ adduser hais     #添加一个hais用户
$ passwd hais     #给hais用户设置密码
$ mkdir /home/hais      #给hais用户创建一个文件夹
$ chown hais:hais /home/hais        #给hais设置home文件
```

创建好了之后，开始配置sudo

``` bash
$ chmod u+w /etc/sudoers         #修改文件给可写
$ nano /etc/sudoers               #修改文件 在 'root ALL=(ALL) ALL' 后增加一行 “hais ALL=(ALL) ALL”
$ chmod u-w /etc/sudoers         #恢复文件权限
```

然后就OK了， 可以愉快的使用 sudo 了
