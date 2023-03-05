---
title: CentOS7下安装FastDFS
date: 2020-06-26 13:28:03
tags: linux
---

## 应用场景
当使用springcloud 微服务实现负载均衡时，遇到文件上传获取的难题，这时可以将文件放到FastDFS 服务上，只需要保存FastDFS返回的文件key 就好。

## 下载FastDFS安装依赖包
有兴趣可以去[作者大佬的博客](https://github.com/happyfish100)看看,给个Star啥的。
[fastdfs-6.06.tar.gz](https://github.com/happyfish100/fastdfs/archive/V6.06.tar.gz)
[libfastcommon-1.0.43.tar.gz](https://github.com/happyfish100/libfastcommon/archive/V1.0.43.tar.gz)
[fastdfs-nginx-module-1.22.tar.gz](https://github.com/happyfish100/fastdfs-nginx-module/archive/V1.22.tar.gz)

**下载命令**
```bash

# 安装编译依赖
yum -y install gcc 
yum -y install gcc-c++

# 进入/opt 文件夹
cd /opt
# 创建文件夹 softwares
mkdir softwares 
# 进入softwares 
cd softwares
# 下载fastdfs依赖文件
wget -O fastdfs-6.06.tar.gz https://github.com/happyfish100/fastdfs/archive/V6.06.tar.gz
wget -O libfastcommon-1.0.43.tar.gz https://github.com/happyfish100/libfastcommon/archive/V1.0.43.tar.gz
wget -O fastdfs-nginx-module-1.22.tar.gz https://github.com/happyfish100/fastdfs-nginx-module/archive/V1.22.tar.gz
```

访问时依赖nginx 所以也要下载nginx
[nginx 官网](http://nginx.org/en/download.html)

## 安装并配置
安装是有顺序的，先安装 libfastcommon再安装 fastdfs， 如果有需要用到nginx，就先安装nginx，再配置fastdfs-nginx-module 包
### 安装libfastcommon
```bash
# 进入 /opt/softwares文件夹
cd /opt/softwares

# 解压 libfastcommon-1.0.43.tar.gz 
tar -zxvf libfastcommon-1.0.43.tar.gz
# 进入解压好的文件夹里面
cd libfastcommon-1.0.43/
# 运行make.sh编译 
./make.sh
# 安装
./make.sh install
```
### 安装fastdfs
```bash
# 进入 /opt/softwares文件夹
cd /opt/softwares
# 解压
tar -zxvf fastdfs-6.06.tar.gz
# 进入解压好的文件夹里面
cd fastdfs-6.06
# 运行make.sh编译 
./make.sh
# 安装
./make.sh install
```
### 配置fastdfs
#### 创建数据文件夹
```bash
# 创建tracker 日志和数据文件存放的地址
mkdir -p /data/fastdfs/tracker
# 创建storage 日志和数据文件存放的地址
mkdir -p /data/fastdfs/storage
# 数据存放在store0 文件夹上
mkdir -p /data/fastdfs/storage/store0
```
#### 配置tracker.conf 
```bash
# 进入fastdfs 配置文件目录
cd /etc/fdfs 
# 查看配置文件
ls -lh
# 备份tracker.conf 
cp tracker.conf.sample tracker.conf
# 编辑tracker.conf
vi tracker.conf
# 配置 bash_path = /data/fastdfs/tracker
```
#### 配置storage.conf 和 storage_ids.conf
```bash
 # 进入fastdfs 配置文件目录
cd /etc/fdfs 
# 查看配置文件
ls -lh
# 备份storage.conf
cp storage.conf.sample storage.conf
# 编辑storage.conf
vi storage.conf
# group_name = group1 这里可以自定义group名称
# base_path = /data/fastdfs/storage 配置storage基础路径
# store_path = /data/fastdfs/storage/store0 配置数据存放路径
# tracker_server = 192.168.3.12:22122  这里不能写成127.0.0.1 ，由于只有一台tracker 服务所以写本机ip 就好了
# 备份storage_ids.conf
cp storage_ids.conf.sample storage_ids.conf
# 编辑 storage_ids.conf
vi storage_ids.conf
# 100001   group1  192.168.3.12 由于只有一个group1 所以保留一个，改成本机ip地址就好

```

## 启动服务
启动服务在`/etc/init.d`下

### 启动tracker服务
```bash
# 进入/etc/init.d
cd /etc/init.d
# 指定配置 文件启动 ， 可以不指定它会默认去找  tracker.conf
./fdfs_trackerd start  /etc/fdfs/tracker.conf
```

### 启动storage服务
```bash
# 进入/etc/init.d
cd /etc/init.d
# 指定配置 文件启动 ， 可以不指定它会默认去找  storage.conf
./fdfs_storaged start /etc/sdfs/storage.conf 
# 如果想在本机启动多个storage 可以复制多个storage.conf 改变名字和里面配置的端口参数,
# 然后通过指定配置文件启动的方式启动多个storage服务
# 比如 ./fdfs_storaged start /etc/sdfs/storage1.conf 
# ./fdfs_storaged start /etc/sdfs/storage2.conf 等，启动多个storage

```

## 安装nginx
### 先安装nginx 依赖包
```bash
yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel
```
### 下载nginx并解压
```bash
cd /opt/softwares/
wget -c https://nginx.org/download/nginx-1.16.1.tar.gz
tar -zxvf nginx-1.16.1.tar.gz
```
### 配置并安装nginx
```bash
# 进入解压后的目录
cd nginx-1.16.1 
# 执行默认配置
./configure
# 编译
make
# 安装
make install
# 可以使用&& 符号连起来一起运行  make && make install
```
### 启动nginx
nginx 安装路径在 `/usr/local/nginx` ，配置文件在conf 文件夹上
```bash
# 启动nginx 
/usr/local/nginx/sbin/nginx 
```

nginx 常用命令说明

| 命令 | 说明 |
| -- | -- |
| nginx -s stop | 停止 |
| nginx -s quit | 退出 |
| nginx -s reload | 重新读取配置并启动 |

## 配置FastDFS 的Nginx 模块

### 解压 fastdfs-nginx 模块
```bash
tar -zxvf  fastdfs-nginx-module-1.22.tar.gz
```
### 配置nginx
```bash
# 如果启动了nginx ，先停止nginx 
/usr/local/nginx/sbin/nginx -s quit

# 进入刚刚解压的nginx 目录
cd /opt/softwares/nginx-1.16.1

# 添加fastdfs-nginx模块到nginx 
./configure --add-module=../fastdfs-nginx-module-1.22/src
# 重新编译安装nginx
make && make install
```
### 查看是否安装fastdfs-nginx 
如果出现 configure arguments: --add-module=../fastdfs-nginx-module-1.22/src 说明配置成功了
```bash
/usr/local/nginx/sbin/nginx -V
```
### 编辑mod_fastdfs.conf配置文件
```bash
# 进入/opt/softwares/fastdfs-nginx-module-1.22/src
cd /opt/softwares/fastdfs-nginx-module-1.22/src

# 复制  mod_fastdfs.conf 到 /etc/fdfs 目录上
cp mod_fastdfs.conf /etc/fdfs/

# 进入fastdfs-6.06/conf 解压目录下
cd /opt/softwares/fastdfs-6.06/conf
# 复制  anti-steal.jpg http.conf mime.types 到 /etc/fdfs/上
cp anti-steal.jpg http.conf mime.types /etc/fdfs/

# 编辑 mod_fastdfs.conf 
vim /etc/fdfs/mod_fastdfs.conf
```
mod_fastdfs.conf 修改内容
> #配置tracker_server 服务器地址
> tracker_server=192.168.3.12:22122
> #需要和storage.conf 一致
> store_path0=/data/fastdfs/storage/store0
> #如果地址中包含group 前缀需要设置为true
> url_have_group_name=true

### 编辑nginx.conf 配置文件

```bash
vim /usr/loca/nginx/conf/nginx.conf
```
> #在server 下面新增
> #如果只有一个group1 配置
> location ~/group1/M00 {
    ngx_fastdfs_module;
}
> #如果多个group可以使用正则,配置多个group 的时候用到
> location ~/group([0-9])/M00 {
>    ngx_fastdfs_module;
> }

### 启动nginx 
```bash
/usr/local/nginx/sbin/nginx
```

## 查看服务是否启动
```bash
# 查看是否启动
ps aux | grep fdfs 
# 查看tracker端口是否正常
netstat -tlnp | grep 22122 
# 查看storage 端口是否正常
netstat -tlnp | grep 23000
# 查看nginx 端口是否正常
netstat -tlnp | grep 80
```

## 上传使用方式
[JAVA 示例]()
[C# 示例]()
