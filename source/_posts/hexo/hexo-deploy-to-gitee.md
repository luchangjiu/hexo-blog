---
title: hexo 博客部署到gitee 上
date: 2020-05-30 22:16:58
tags: Hexo
---

# 说明
以下操作在ubuntu 系统上完成。

## 准备工作

* gitee 帐号信息
* ssh
* git
* hexo

## 配置git 的用户信息
```bash
git config --global user.name "gitee 用户名"
git config --global user.mail "你的邮箱"
```

## 生成ssh密钥

```bash
ssh-keygen -t rsa -C "邮箱地址"
# 连续按三下回车 后 密钥信息将会生成在你的主目录下的.ssh 文件夹上面

# 进入.ssh 目录下面
cd /home/你的帐号名/.ssh
# 查看密钥信息
cat id_rsa.pub

# 添加密钥信息到ssh 上
ssh-add id_rsa

```

## 将生成的密钥信息配置到gitee 上
1. 登陆[码云](https://gitee.com)
2. 进入个人设置
3. 找到安全设置--> SSH公钥
4. 填写标题,然后将刚才 cat id_rsa.pub 这个文件里面的内容复制到 公钥输入框上
5. 点击提交即可

## 测试是否链接成功
``` bash
# 执行命令
:ssh -T  git@gitee.com

# 返回以下结果表示成功

Hi 用户名! You've successfully authenticated, but GITEE.COM does not provide shell access.

```

## 进入到hexo 文件夹上
1. 打开_config.yml 文件

**在底下找到deploy 节点 并修改你的git地址
修改时注意保证你的仓库是空的，否则hexo 上传文件时会将你原有的文件清空**
```bash

deploy:
  type: git
  repo: git@gitee.com:你的用户名/仓库名.git
  branch: master

```
2. 安装hexo git 
```bash
npm install hexo-deployer-git --save
```

3. 上传到gitee 仓库上

```bash

hexo clean # 清除缓存
hexo g # 生成
hexo d # 部署

```

## 部署成功后进入gitee 仓库

1. 点击服务 --> Gitee Pages
2. 进入该页面后需要手动点击更新按钮才会部署到静态页面上
3. 点击更新按钮后等待几分钟就可以访问了

