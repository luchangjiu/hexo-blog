---
title: 怎样创建hexo 的博客
date: 2020-05-30 22:16:58
tags: Hexo
---

# 步骤一
打开命令行（我这里用的是windows 系统 用cmd 命令行） 
```bash 
$ hexo new "这里输入一个文件的名称（一般用英文）"
```
# 步骤二
在部署目录 <code>hexo/source/_posts/</code> 文件夹下找到刚刚的文件以<code>.md</code> 结尾，然后对它进行编辑(需要熟悉markdown 语法)

# 步骤三
编辑完成后 输入命令
```bash
$ hexo clean    ##清空缓存信息
$ hexo generate ## 根据.md 格式的文件生成hexo 需要的html 静态网页
$ hexo deploy   ## 发布到GitHub上
```