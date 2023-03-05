---
title: 使用Putty 链接 linux 系统
date: 2020-06-22 22:47:05
tags: windows
---

## 说明 
使用putty 轻量级ssh 软件链接 linux shell 终端和互传文件。

## 下载文件

[下载 putty 软件 ](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
按照自己电脑的操作系统下载 ，可以下载  `putty-64bit-0.73-installer.msi` 安装使用，也可以直接下载`putty.zip` 解压直接使用， 由于本人不想安装直接下载`putty.zip` 文件。 下载后解压即可。
解压后如图： 
![alt puttylist](/images/archives/putty-list.png)

##  使用PUTTY.EXE链接终端
双击打开PUTTY.EXE录入ip 地址和端口然后点击open 打开终端输入用户名和密码即可
如图： 
![alt putty-exe](/images/archives/putty-exe.png)

## 使用PSCP.EXE传输文件
打开cmd 命令， cd 定位到PSCP.EXE的文件夹上

**上传命令**
```bash
# scp 当前windows 文件  linux用户名@IP地址:/上传目录
# 示例 将 test.txt 上传到linux /opt 文件夹上
scp D:\\test.txt root@192.168.3.12:/opt
```
**下载命令**
```bash
# scp  linux用户名@IP地址:/文件地址 windows 下载目录
# 示例 将 linux 上的test.txt下载到当前D盘
scp root@192.168.3.12:/opt/test.txt D:\\
```

//todo 其余参数解释和使用待补充

## 使用PSFTP.EXE传输文件
双击打开PSFTP.EXE 文件
**输入命令**
```bash
# open ip 地址 回车 然后输入用户名和密码登陆
open 192.168.3.12
# cd /opt 切换到指定的操作目录
cd /opt
# lcd d:\\ 切换本地windows操作的目录
lcd D:\\
# put 上传文件名
put test.txt
# get 下载文件名
get test2.txt
```

//todo 其余参数解释和使用待补充

## 配置环境变量
1. 为方便配置环境变量打开`控制面板\系统和安全\系统`打开高级系统设置点击环境变量
2. 点击新建环境变量
3. 变量名`PUTTY_HOME`, 变量值是putty 文件的目录比如我的：`S:\softwares\common\putty`
4. 在path系统变量后面追加 `;%PUTTY_HOME%`
5. 重新打开cmd 就可以在任意目录访问scp， putty，psftp 等命令了。 方便文件直接上传下载
