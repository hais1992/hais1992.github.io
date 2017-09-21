---
title: 使用Ubuntu16.04编译SudaMod
date: 2017-07-28 10:03:38
tags: [Linux,Ubuntu,编译,Android,SudaMod]
---
##### 注：文章参考 https://github.com/wzv5/android_device_zuk_z2_plus/wiki/ 进行修改整理。

### 准备工作
> * 电脑磁盘空余150G以上
> * 电脑运存大于4G
> * 推荐使用Ubuntu16.04
> * 安装系统时 swap 推荐设置10G
> * 推荐设置 /tmp 空间为10G
> * 系统默认使用 Python2
		
### 1、分区调整(如安装系统时已设置swap比较大的可不设置)
``` bash
sudo dd if=/dev/zero of=/swapfile.img bs=1M count=10240
sudo chmod 600 /swapfile.img
sudo mkswap /swapfile.img
sudo swapon /swapfile.img
#编辑 /etc/fstab 在最后加入一行
/swapfile.img	swap	swap	defaults			0	0
tmpfs			/tmp	tmpfs	defaults,size=10G	0	0
#调整完成后可使用 free -h 查看
```

### 2、安装编译工具
``` bash
sudo apt-get update && sudo apt-get install git-core gnupg flex bison gperf libsdl1.2-dev libesd0-dev libwxgtk3.0-dev squashfs-tools build-essential zip curl libncurses5-dev zlib1g-dev openjdk-8-jre openjdk-8-jdk pngcrush schedtool libxml2 libxml2-utils xsltproc lzop libc6-dev schedtool g++-multilib lib32z1-dev lib32ncurses5-dev gcc-multilib maven tmux screen w3m ncftp liblz4-tool pngquant bc
#编译过程中缺少依赖rsync,使用sudo apt-get install rsync解决
```

### 3、由于同步Android源码需要翻墙，可以使用修改hosts的方法翻墙
``` bash
sudo wget https://raw.githubusercontent.com/sy618/hosts/master/FQ -O /etc/hosts
```

### 4、安装repo
``` bash
#下载安装
sudo wget https://storage.googleapis.com/git-repo-downloads/repo -O /usr/bin/repo
sudo chmod +x /usr/bin/repo
```

### 5、创建源码目录并进入
``` bash
mkdir -p ~/Android/SudaMod && cd ~/Android/SudaMod
```

### 6、下载配置清单文件
``` bash
#下载核心官方的清单文件
repo init -u git://github.com/SudaMod/android.git -b sm-3.1
#配置个人自定义的清单文件，包含device、vendor、kernel
mkdir -p .repo/local_manifests && cd .repo/local_manifests
wget https://raw.githubusercontent.com/wzv5/android_device_zuk_z2_plus/sm-3.1/z2_sm_manifest.xml
```

### 7、开始同步源码
``` bash
cd ~/Android/SudaMod
repo sync -c -f -j$( nproc --all ) --force-sync --no-clone-bundle --no-tags
```

### 8、同步完成后，创建一个编译环境的脚本"sudamodBuild.sh"输入以下内容
``` bash

#!/bin/sh
# 改为自己的源码路径
BASEPATH=~/Android/DirtyUnicorns
# 创建 ccache 缓存目录
mkdir -p $BASEPATH/ccache_dir
export USE_CCACHE=1
export CCACHE_DIR=$BASEPATH/ccache_dir
export JACK_SERVER_VM_ARGUMENTS="-Xmx4096M -Dfile.encoding=UTF-8 -XX:+TieredCompilation"
export SDCLANG=true
export SDCLANG_PATH=$BASEPATH/prebuilts/snapdragon/llvm-3.8/bin
export SDCLANG_LTO_DEFS=$BASEPATH/device/qcom/common/sdllvm-lto-defs.mk
cd $BASEPATH
make clean
make clobber
./prebuilts/sdk/tools/jack-admin kill-server
wget https://raw.githubusercontent.com/sy618/hosts/master/FQ -O ./system/core/rootdir/etc/hosts
./prebuilts/misc/linux-x86/ccache/ccache -M 50G
source build/envsetup.sh
breakfast z2_plus
mka bacon -j$( nproc --all )


```
保存并退出后，给脚本执行权限
``` bash
chmod +x sudamodBuild.sh
```

### 9、开始编译
``` bash
#执行刚刚的脚本进入编译环境
source sudamodBuild.sh
#开始编译
brunch z2_plus
#编译结算后执行命令释放资源
croot
./prebuilts/sdk/tools/jack-admin kill-server
#执行清理，便于下次编译
croot
make clean
```


## 其它说明
> * 如果要收集编译日记，则编译的时候执行“brunch z2_plus | tee ~/sudalog.txt”
> * 后期更新的话就 “repo sync” 同步上游源码，再次编译就可以了
> * 制作OTA包 
	``` bash
	#每次编译完后进入目录，把 sm_z2_plus-target_files-xxx.zip 文件备份
	~/Android/SudaMod/out/target/product/z2_plus/obj/PACKAGING/target_files_intermediates
	#当包有两个后，就可以使用命令制作增量包了
	croot
	./build/tools/releasetools/ota_from_target_files -i sm_z2_plus-target_files-old.zip sm_z2_plus-target_files-new.zip ota.zip
	```
	
> * 添加主题引擎，编辑清单文件 ~/Android/SudaMod/.repo/local_manifests 增加以下两行内容，后执行 "repo sync" 同步
	``` bash
	<project name="sudamod/android_vendor_extra-1" path="vendor/extra" remote="github" revision="cm-14.1-rootless"/>
	<project name="substratum/interfacer" path="packages/services/ThemeInterfacer" remote="github" revision="n-rootless"/>
	
	#每次同步源码都需要重新打补丁 " ./vendor/extra/patch.sh "
	#加入主题管理器，创建 prebuilt 目录，在其中放入主题管理器 Substratum.apk 文件，编辑 sm.mk 文件，在最后加入以下代码后保存即可
	PRODUCT_COPY_FILES += \$(LOCAL_PATH)/prebuilt/Substratum.apk:/system/app/Substratum/Substratum.apk
	```

> * 编译的时候偶尔卡住不动或者有提示“Compiling SDK Stubs with Jack”，的可以改一下jack的配置文件
	``` bash
	#修改 ./prebuilts/sdk/tools/jack-admin，找到
	JACK_SERVER_COMMAND="java -XX:MaxJavaStackTraceDepth=-1 -Djava.io.tmpdir=$TMPDIR $JACK_SERVER_VM_ARGUMENTS -cp $LAUNCHER_JAR $LAUNCHER_NAME"
	#改为
	JACK_SERVER_COMMAND="java -XX:MaxJavaStackTraceDepth=-1 -Djava.io.tmpdir=$TMPDIR $JACK_SERVER_VM_ARGUMENTS -Xmx4096M -cp $LAUNCHER_JAR $LAUNCHER_NAME"
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