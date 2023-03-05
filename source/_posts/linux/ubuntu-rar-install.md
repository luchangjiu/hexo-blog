---
title: ubuntu 安装rar 解压工具
date: 2022-04-17 20:27:03
tags: linux
---

# 环境

ubuntu 20.04

## 安装步骤

```shell

# 安装压缩工具
sudo apt-get install rar

# 卸载
sudo apt-get remove rar

# 安装解压工具
sudo apt-get install unrar

# 卸载
sudo apt-get reomve unrar

```

## 使用方法

```shell

# 压缩文件
# 示例 rar a 压缩文件名  压缩文件夹名称
rar a myrar.rar files

# 解压文件
# 示例 unrar e 解压文件  解压到文件夹名称
unrar e myrar.rar myrar

# 或者使用rar x 压缩文件 也可以解压
rar x myrar.rar

```
