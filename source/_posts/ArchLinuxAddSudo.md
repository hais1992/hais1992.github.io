---
title: 给 ArchLinux 增加sudo命令
date: 2017-09-21 10:03:38
tags: [Linux,ArchLinux,sudo]
---

习惯了 sudo apt-get update ， 然后 某天，发现没有了 sudo 感觉有点不爽快了。



安装Sudo
``` bash
pacman -S sudo
```


安装完后我们新建个用户，并给他加入sudo的权限
``` bash
useradd hais     #添加一个hais用户,如果是ubuntu命令为 adduser
passwd hais     #给hais用户设置密码
mkdir /home/hais      #给hais用户创建一个文件夹
chown hais:hais /home/hais        #给hais设置home文件
```


创建好了之后，开始配置sudo
``` bash
chmod u+w /etc/sudoers         #修改文件给可写
vim /etc/sudoers               #修改文件 在 'root ALL=(ALL) ALL' 后增加一行 “hais ALL=(ALL) ALL”
chmod u-w /etc/sudoers         #恢复文件权限
```

然后就OK了， 可以愉快的使用 sudo 了