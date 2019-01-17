---
title: Windows Server数据中心版(无UI) 安装Honolulu管理中心
date: 2019-01-17T11:20:37+08:00
tags: [Windows,Honolulu]
categories: ["Windows"]
---

### 前提说明。
	+ 下文中的名称介绍
		+ **服务器** 指的是 **被控制端**
		+ **使用端** 指的是 **使用的机子(一般是自己电脑)**
	
	+ Honolulu 可安装在服务器中，也可安装在使用端中

### 准备工作
 - 准备一台 Windows Server数据中心版(无UI) 的服务器，并开通3389远程链接
 - 如是 腾讯云、阿里云 等云服务器，请在web面板上放行 入方向的5985、5986 端口

### 服务器(被控制端)配置
1、打开远程后，在cmd窗口输入以下命令
``` bash
#运行 PowerShell
PowerShell

#打开 PowerShell 的远程链接权限
Enable-PSRemoting -Force
Set-NetFirewallRule -Name "WINRM-HTTP-In-TCP-PUBLIC" -RemoteAddress Any
```

### 安装 Honolulu (Windows Admin Center)
可安装在 服务器 中，也可安装在 使用端 中，请根据需求选择
软件下载页面为：[Honolulu 下载页面](https://docs.microsoft.com/en-us/windows-server/manage/windows-admin-center/understand/windows-admin-center)
软件下载地址为：[Honolulu 下载地址](https://aka.ms/WACDownload)
以下命令，不管安装在那个端都通用(要在PowerShell中运行)
``` bash
#授权信任IP
Set-Item WSMan:localhost\client\trustedhosts -value * -Force

#下载软件
wget -Uri https://download.microsoft.com/download/1/0/5/1059800B-F375-451C-B37E-758FFC7C8C8B/WindowsAdminCenter1809.5.msi -UseBasicParsing -OutFile c:\Honolulu.msi

#打开软件安装，安装的时候注意自己填写的端口，以后打开管理界面也是这端口
./Honolulu.msi
```
到现在可以用 IP+端口 的形式直接打开网页进行访问管理了.

Ps:假设连接的时候需要输入账号密码，且多次输入不正常，可以试一下修改密码后再链接
``` bash
# CMD 下运行,假设 administrator 密码改为 Hais1992
net user administrator Hais1992
```

