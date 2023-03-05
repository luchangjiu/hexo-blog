---
title: window 安装FreeSSHD 实现与linux 文件互传
date: 2020-06-25 21:56:01
tags: windows
---

## 应用场景
在搭建 Windows Server 服务器时有时候需要在linux 系统上直接发送文件或者通过终端直接链接过来， 所以需要windows 系统支持ssh 。
## 下载FreeSSHD
下载地址`http://www.freesshd.com/?ctt=download` 下载[freeSSHd.exe](http://www.freesshd.com/freeSSHd.exe)

## 安装
双击运行`freeSSHd.exe` 然后一直next 直至完成就好了。

## 配置
在桌面双击FreeSSHd快捷方式， 然后在右下角找到启动的FreeSSHd启动程序，并打开它。

### 设置登陆用户
点击Users 的Tab 页如图： 
![alt freesshd01](/images/archives/freesshd-install-img01.png)
添加登陆用户
![alt freesshd02](/images/archives/freesshd-install-img02.png)
这里我配置了两个可以登陆的用户，一个使用windows 授权方式登陆，一个是我自定义的root 账号
![alt freesshd03](/images/archives/freesshd-install-img03.png)
![alt freesshd03](/images/archives/freesshd-install-img04.png)

### 配置日志监听
点击Logging 的Tab页如图:
配置日志输入
![alt freesshd03](/images/archives/freesshd-install-img05.png)
设置好点击应用即可

### 配置登陆授权方式
点击 Authentication 的Tab页如图： 
![alt freesshd03](/images/archives/freesshd-install-img06.png)
设置好点击应用即可

### 配置SSH 信息
点击SSH的Tab 页如图：
![alt freesshd03](/images/archives/freesshd-install-img07.png)
设置好点击应用即可
根据设置的端口去防火墙入站规则设置开启， 我这里用的是默认22 端口。

### 配置SFTP 上传文件路径
点击SFTP的Tab 页如图： 
![alt freesshd03](/images/archives/freesshd-install-img08.png)

## 开启服务

打开cmd 命令行输入命令：
```bash
# 重新启动 FreeSSHDService服务
net stop FreeSSHDService
net start FreeSSHDService
```
或者打开services.msc找到windows服务FreeSSHDService点击重新启动就好如图： 
![alt freesshd03](/images/archives/freesshd-install-img10.png)

## 测试是否链接成功
使用putty 的ssh 方式链接或者在 linux 系统上直接使用ssh 链接和发送文件测试就好。
