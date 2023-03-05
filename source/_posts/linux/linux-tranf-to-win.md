---
title: linux 使用ssh免密向window 传输数据
date: 2022-04-27 21:56:01
tags: linux
---

# 说明

最近遇到 数据库云服务器是linux 系统，但是本地备份的物理机系统是win7 的，
这时需要从linux 服务器上拷贝数据并且需要免密传输（方便写shell 脚本）.


## 依赖
CenterOS 7 自带ssh 服务
Windows7 默认你已经安装了OpenSSH 服务(没安装可以先去安装了)

## 使用ssh 生成密钥

在linux 服务器端生成rsa密钥

输入以下命令后，出现的提示可以根据提示录入地址或者配置密码，我这里直接都是按回车，使用默认的就好
```shell

ssh-keygen -t rsa

```

## 将生成的密钥复制到windows 系统上

在linux 系统上使用scp 命令复制过去
我这里使用的是root 帐号生成的rsa密钥默认在 /root/.ssh 目录下

如果你不是使用root 帐号，那么默认生成的rsa 密钥在 /home/你的帐号/.ssh 目录下
```shell

# 这里传输到windows d 盘目录
scp /root/.ssh/id_rsa.pub  Administrator@你的windowip地址:/d:/

```

## 修改windows openssh 配置文件

进入 c://ProgramData//ssh 打开文件 sshd_config

找到节点 
PubkeyAuthentication yes   将这行的注释取消
然后注释掉下面两行
Match Group admiistrators
    AuthorizedKeysFile __PROGRADDATA__/ssh/administrators_authorized_keys

## 配置windows openssh 的密钥文件

进入C://用户//你当前的用户名//.ssh 

然后将刚刚linux 上传到D盘的id_rsa.pub 文件复制到 .ssh 目录上并改名为 authorized_keys

注意： 如果没有.ssh 文件可以使用cmd 命令行 mkdir .ssh 创建


配置后记得重启openssh 相关的服务.


## 测试是否成功

切换回我们的linux 
输入以下命令

```shell

ssh Administrator@你的windows IP 地址

```

如果不需要密码能直接链接那么就表示配置成功了，可以编写shell 脚本ssh 命令了
