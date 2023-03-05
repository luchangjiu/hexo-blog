---
title: Redis 模糊删除key 
date: 2022-03-22 02:15:02
tags: redis
---

# 前言

有时候遇到想直接在服务端批量删除key,或者通过模糊查询key 来删除redis 的数据，那可以使用下面的脚本来实现。
本次操作在Ubuntu 系统操作的


## 编写删除命令的lua脚本
redis-cli 可以通过 解析 lua 脚本来执行信息，我们可以利用这点来实现模糊删除key。

```bash
# 创建 lua 脚本

vim del.lua
# 编写以下代码

local key = KEYS[1]
local list = redis.call("keys", key)
for i, v in ipairs(list) do
    redis.call("del", v);
end

```

## 调用脚本删除


```bash
# 使用redis-cli 命令来解析刚刚编写的lua ，并传入需要模糊删除的key信息

redis-cli -n 0 --eval /redis/script/del.lua "*需要模糊删除的key*"

```
嫌每次都要输入一大串redis-cli 的也可以再编写一个shell 脚本. 然后执行它就好了。
