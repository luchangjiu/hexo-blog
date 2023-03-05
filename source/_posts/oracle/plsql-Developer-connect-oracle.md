---
title: PL/SQL Developer 远程链接 Oracle 数据库
date: 2016-12-12 12:13:23
tags: oracle
---

# 说明
本机无需安装Oracle ， 只需要安装PL/SQL 和Oracle 驱动 即可远程链接Oracle 数据库 。

## 第1步 下载工具
1. [pl/sql 工具官网](https://www.allroundautomations.com/)
2. [oracle 驱动包 官网](http://www.oracle.com/technetwork/topics/winx64soft-089540.html)
![alt oracle 驱动下载图](/images/archives/20161212115533951.png)

## 第2步 先安装pl/sql 
安装过程 略

## 第3步 解压oracle 驱动
3.1 解压 instantclient_12_1  放到指定目录  
3.2 在instantclient_12_1 里面 新建文件夹 NETWORK/ADMIN/
3.3 再新建一个文件 tnsnames.ora （这里注意tnsnames.org 是固定写法）
文件内容配置oracle 链接信息
![alt tnsnames.org 文件配置](/images/archives/20161212120245848.png)

## 第4步 环境配置
4.1 进入windows 环境变量配置
4.2 配置信息如下：  
ORACLE_HOME = F:\instantclient_12_1 //指定的目录  
TNS_ADMIN = F:\instantclient_12_1\NETWORK\ADMIN //配置 tsn文件路径  
NLS_LANG = SIMPLIFIED CHINESE_CHINA.ZHS16GBK //配置编码  
在path 最后添加 %ORACLE_HOME%  

## 第5步 配置PL/SQL
5.1 打开pl/sql 菜单 Tools –> Preferences 如图： 
![alt pl/sql 配置图](/images/archives/20161212121018344.png)

## 最后一步 重启PL/SQL
6.1 重启pl/sql 查看databases 是否有tnsnames 的链接 ，有的话就配置成功了

