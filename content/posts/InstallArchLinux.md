---
title: VBox安装ArchLinux
date: 2017-09-17 16:26:32
tags: [Linux,ArchLinux,Arch,Vbox]
categories: ["Linux"]
---

### 准备软件
	下载Arch：https://www.archlinux.org/download/
	下载Vbox：https://www.virtualbox.org/wiki/Downloads

### 参考：
	http://blog.csdn.net/qq_25187981/article/details/75209051
	http://bbs.archlinuxcn.org/viewtopic.php?id=1037
	https://wiki.archlinux.org/index.php/Installation_guide
	https://wiki.archlinux.org/index.php/Installation_guide_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)

### 安装步骤
``` bash
ping www.baidu.com			#检查网络

timedatectl set-ntp true	#更新时间

fdisk -l					#查看磁盘名称

fdisk /dev/sda				#磁盘分区

#新建第一个分区，大小为1G，作为交换分区
# n		  ->新建
# p  	  ->选择分区类型为主分区
# l       ->选择分区扇区号
# 2048    ->设置分区开始的值
# 2099200 ->设置分区结束的值
#
# 创建第二个分区，剩下的空间
# n		  ->新建
# p  	  ->选择分区类型
# 2       ->选择分区扇区号
# 回车    ->设置分区开始的值
# 回车    ->设置分区结束的值
#
# 写入分区信息
# w

mkswap /dev/sda1	#格式化交换分区

swapon /dev/sda1	#挂载交换分区

mkfs.ext4 /dev/sda2	#格式化文件分区

mount /dev/sda2 /mnt	#挂载文件分区

# 将下载的源选择为中国
sed -i '/China/!{n;/Server/s/^/#/};t;n' /etc/pacman.d/mirrorlist

pacstrap /mnt base	# 下载安装基本的系统

genfstab -U /mnt >> /mnt/etc/fstab	# 设置自动挂载分区

arch-chroot /mnt	# 进入下载好的系统进行配置

# 设置系统时区
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

hwclock --systohc	# 同步时间

# 设置本地化，将en_US.UTF-8前的#号去掉
vi /etc/locale.gen

# 应用本地化配置
locale-gen

# 设置系统语言
echo LANG=en_US.UTF-8 > /etc/locale.conf

# 设置本机名称
echo ArchLinux > /etc/hostname

# 设置密码
passwd

# 更新包的信息
pacman -Ssy

# 下载Boot Loader包并安装
pacman -S grub

# 将Boot Loader安装到指定的硬盘
grub-install --target=i386-pc /dev/sda

# 创建Boot Loader配置
grub-mkconfig -o /boot/grub/grub.cfg

# 退出安装的系统
exit

# 卸载分区
umount -R /mnt

# 重新启动
reboot
```

### 重启进入系统后的工作
``` bash
ip a	#查询机器IP 

#开始联网
systemctl enable dhcpcd

#安装ssh
pacman -Sy openssh
#编辑文件 /ect/ssh/sshd_config 编辑如下并去掉 # 号
#PasswordAuthentication yes
#PermitRootLogin yes 	
#PermitEmptyPasswords no

```


