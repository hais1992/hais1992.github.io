---
title: 使用ArchLinux编译LineageOS
date: 2017-09-17 10:03:38
tags: [Linux,ArchLinux,编译,Android,LineageOS]
categories: ["Linux"]
---

### 准备工作
> * 电脑磁盘空余150G以上
> * 电脑运存大于4G
> * 安装系统时 swap 推荐设置10G
> * 推荐设置 /tmp 空间为10G
> * 系统默认使用 Python2
		
### 1、分区调整(如安装系统时已设置swap比较大的可不设置)
``` bash
dd if=/dev/zero of=/swapfile.img bs=1M count=10240
chmod 600 /swapfile.img
mkswap /swapfile.img
swapon /swapfile.img
#编辑 /etc/fstab 在最后加入一行
/swapfile.img	swap	swap	defaults			0	0
tmpfs			/tmp	tmpfs	defaults,size=10G	0	0
#调整完成后可使用 free -h 查看
```

### 2、准备工作
``` bash
# 编辑文件/etc/pacman.conf，最后面添加lib32和AUR支持
[multilib]
Include = /etc/pacman.d/mirrorlist
[archlinuxcn]
SigLevel = Optional TrustAll
Server = http://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch

# 编辑文件/etc/pacman.d/mirrorlist
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
Server = http://mirrors.163.com/archlinux/$repo/os/$arch
Server = http://mirror.bit.edu.cn/archlinux/$repo/os/$arch

#安装软件包
pacman -Syy gcc gnupg flex bison gperf sdl wxgtk squashfs-tools curl ncurses zlib schedtool perl-switch zip unzip libxslt bc jdk8-openjdk base-devel git-core repo wget imagemagick base-devel git-core pngcrush libxml2 lzop maven tmux screen w3m ncftp

#AUR安装 ncurses ,参考 http://blog.csdn.net/gddxz_zhouhao/article/details/53466977
yaourt -S ncurses5-compat-libs

#做软连接 python
ln /usr/bin/python2 /usr/bin/python

# 编辑文件/etc/yaourtrc 去掉 # AURURL 的注释，加入aur镜像地址，修改为
AURURL="https://aur.tuna.tsinghua.edu.cn"
```

### 3、由于同步Android源码需要翻墙，可以使用修改hosts的方法翻墙
``` bash
#下载hosts翻墙
wget https://raw.githubusercontent.com/sy618/hosts/master/FQ -O /etc/hosts

#使用清华大学的repo源
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'
```

### 4、创建源码目录并进入
``` bash
mkdir -p ~/Android/LineageOS && cd ~/Android/LineageOS
```

### 6、下载配置清单文件
``` bash
#下载核心官方的清单文件
repo init -u git://github.com/LineageOS/android.git -b cm-14.1
#配置个人自定义的清单文件，根据机型和系统寻找device、vendor、kernel
mkdir -p .repo/local_manifests && cd .repo/local_manifests
wget https://raw.githubusercontent.com/hais1992/android_device_xiaomi_mido/hais-cm14.1/local_manifests_mido.xml
```

### 7、开始同步源码
``` bash
#修改使用清华大学的aosp源码
nano .repo/manifest.xml 
#把第四行的 https://android.googlesource.com 改为 https://aosp.tuna.tsinghua.edu.cn 或 git://mirrors.ustc.edu.cn/aosp/
cd ~/Android/LineageOS
repo sync -c -f -j$(nproc --all) --force-sync --no-clone-bundle --no-tags
```

### 8、同步完成后，创建一个编译环境的脚本"LineageOSBuild.sh"输入以下内容
``` bash

#!/bin/sh
# 改为自己的源码路径
BASEPATH=~/Android/LineageOS
# 创建 ccache 缓存目录
mkdir -p $BASEPATH/ccache_dir
export USE_CCACHE=1
export CCACHE_DIR=$BASEPATH/ccache_dir
export JACK_SERVER_VM_ARGUMENTS="-Xmx4096M -Dfile.encoding=UTF-8 -XX:+TieredCompilation"
export SDCLANG=true
export SDCLANG_PATH=$BASEPATH/prebuilts/snapdragon/llvm-3.8/bin
export SDCLANG_LTO_DEFS=$BASEPATH/device/qcom/common/sdllvm-lto-defs.mk
export WITH_SU=true
cd $BASEPATH
make clean
make clobber
./prebuilts/sdk/tools/jack-admin kill-server
wget https://raw.githubusercontent.com/sy618/hosts/master/FQ -O ./system/core/rootdir/etc/hosts
./prebuilts/misc/linux-x86/ccache/ccache -M 50G
source build/envsetup.sh
breakfast mido
mka bacon -j$( nproc --all )


```
保存并退出后，给脚本执行权限
``` bash
chmod +x LineageOSBuild.sh
```

### 9、开始编译
``` bash
#执行刚刚的脚本进入编译环境
source sudamodBuild.sh
#编译结算后执行命令释放资源
croot
./prebuilts/sdk/tools/jack-admin kill-server
#执行清理，便于下次编译
croot
make clean
```


## 其它说明
> * 如果要收集编译日记，则编译的时候执行“brunch xiaomi_mido | tee ~/log.txt”
> * 后期更新的话就 “repo sync” 同步上游源码，再次编译就可以了
> * 制作OTA包 
	``` bash
	#每次编译完后进入目录，把 sm_z2_plus-target_files-xxx.zip 文件备份
	~/Android/LineageOS/out/target/product/z2_plus/obj/PACKAGING/target_files_intermediates
	#当包有两个后，就可以使用命令制作增量包了
	croot
	./build/tools/releasetools/ota_from_target_files -i sm_z2_plus-target_files-old.zip sm_z2_plus-target_files-new.zip ota.zip
	```
	
> * 如果是使用远程机器进行编译，为了预防网络不稳定重连ssh的情况，可以使用 screen 来进行编译
	``` bash
	sudo apt-get install screen
	#新建一个名为 hais 的 screen
	screen -S hais
	#显示一个名为 hais 的 screen
	screen -r -d hais
	#显示所有正在运行的 screen
	screen -ls
	```
	
	
### 参考网站：
    https://github.com/wzv5/android_device_zuk_z2_plus/wiki/
	https://hais.pw/2017/07/28/UbuntuBuildSudaMod/
	https://mirror.tuna.tsinghua.edu.cn/help/AOSP/
	https://lug.ustc.edu.cn/wiki/mirrors/help/aosp
	https://mirrors.tuna.tsinghua.edu.cn/help/git-repo/
	https://github.com/sy618/hosts
	https://www.isthnew.com/build-lineageos/
	http://blog.csdn.net/gddxz_zhouhao/article/details/53466977
	