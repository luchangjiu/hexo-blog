---
title: oracle 在linux 系统的备份脚本
date: 2022-04-27 12:13:23
tags: oracle
---

# 说明

centos7 上oracle 数据库的备份脚本

## 依赖

centos7 上安装了oracle 数据库且安装的用户是oracle
且oracle 用户的环境可以使用oracle 相关的命令，如sqlplus 等命令.


## 创建oracle 的备份目录

```sql

-- 查询已经存在的directory

select * from  dba_directories;

-- 创建一个名为 oracle_dbbk 的目录 并指向 /data/oracle_dbbk 路径
create or replace directory oracle_dbbk  as '/data/oracle_dbbk';


-- 使用数据库管理员帐号授权可操作directory 的帐号权限

grant read, write on directory oracle_dbbk to 你自己的帐号;

-- 删除目录
drop directory oracle_dbbk;

```

## shell脚本代码

```shell

#!/bin/sh

# 加载当前 oracle 帐号的环境变量
source /home/oracle/.bash_profile

# 获取当前时间

CURDATE=$(date '+%Y%m%d%H%M%S')

# 备份目录
backup_directory=/data/oracle_dbbk


# 备份文件名的前缀
data_file_prefix=orcl_bak_

# 拼接备份文件dmp 的文件名 加上当前的时间
BACKUP_FILE_NAME=$data_file_prefix$CURDATE.dmp	
LOG_BACKUP_FILE_NAME=$data_file_prefix$CURDATE.log	

# 拼接日志文件log 的文件名 加上当前的时间
BACKUP_FILE=$backup_directory/$BACKUP_FILE_NAME
LOG_BACKUP_FILE=$backup_directory/$LOG_BACKUP_FILE_NAME

echo $BACKUP_FILE
echo $LOG_BACKUP_FILE

# 一个删除5 天前的的备份数据文件
delete_prev_data_file() 
{
    premonth=$(date -d"5 days ago" '+%Y%m%d')
    rm -rf $backup_directory/$data_file_prefix$premonth* || true
}
# 进入到备份目录
cd $backup_directory 


# 设置oracle 的导出命令
bak_exec="expdp oracle数据库帐号/密码 directory=oracle_dbbk dumpfile=$BACKUP_FILE_NAME logfile=$LOG_BACKUP_FILE_NAME schemas=数据库帐号名 job_name=exp_user_schema"

# 执行oracle expdp 导出命令
$bak_exec

echo 'delete pre feils  ..... '

# 调用删除前 5 天的函数

delete_prev_data_file


# 下面是可选项 将已经生成的备份文件通过ssh 命令  发送到存储数据的服务器
# 前提是 两台机器需要通过ssh 密钥验证才能免密码传输 
# sendToDiskServer="scp $BACKUP_FILE xxk@192.168.1.121:/e:/oracle_bak/files ";
# $sendToDiskServer

exit 0 


```
## 设置定时任务

这里设置的是凌晨1 点同步的任务

使用 <code> crontab -e </code> 命令，然后录入
<code> 00 01 * * *  /data/oracle_bakup.sh </code>

录入完成后输入 :wq 退出编辑即可

