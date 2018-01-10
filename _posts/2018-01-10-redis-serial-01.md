---
layout: post
title:  redis系列 01 - redis与相关技术
tag: redis
date: 2018-01-10 13:22:50 +09:00
---


### redis 简介

Redis是一个使用ANSI C编写的开源、支持网络、基于内存、可选持久性的日志型,键值对存储数据库 [参考维基百科Redis定义](https://zh.wikipedia.org/wiki/Redis)

* 支持多语言
* 键值存储, 非关系型数据库
* 持久化, 主要存储在内存中, 使用硬盘快照与日志的形式持久化
* 网络同步, 支持主从同步, 消息的发布/订阅
* 性能非常高, 基于内存的读写. `读的速度是110000次/s,写的速度是81000次/s`

## redis 相关技术的比较

* 数据类型更丰富: `Redis`的值,不仅限于字符串, 还可以包含更复杂的 **抽象数据类型**，
`字符串列表`
`无序不重复的字符串集合`
`有序不重复的字符串集合`
`键、值都为字符串的哈希表`,
并且`Redis`支持不同无序、有序的列表，无序、有序的集合间的交集、并集等高级服务器端 **原子操作**.

* 持久性差异: `Redis`是一个内存数据库，但在磁盘数据库上是持久的，重启后可以再次加载, 因此它代表了一个不同的权衡，在这种情况下，在不能大于存储器(内存)的数据集的限制下实现 **非常高的写和读速度**。
内存数据库的另一个优点是，它与磁盘上的相同数据结构相比，复杂数据结构在内存中存储表示更容易操作。 因此，`Redis`可以做*很少的内部复杂性*。

* 数据备份: 支持`master-slave`模式的数据备份。
* 丰富的功能:  消息队列`publish/subscribe`, 通知, `key`过期等等特性

## redis的安装

### 1. Ubuntu 系统
 
* `Ubuntu`使用`apt-get`安装 `redis`

```shell
sudo apt-get update 
sudo apt-get install redis-server
```

* 启动命令

```
redis-server
```

* 检查是否启动

```
redis-cli
``` 

* `redis.conf` 配置文件

### 2. Linux 系统

* 比如 `CentOS` 使用 `wget` 安装

```
wget http://download.redis.io/releases/redis-2.8.17.tar.gz
tar xzf redis-2.8.17.tar.gz
cd redis-2.8.17
make
```
编译后的`redis`服务程序`redis-server`,还有用于测试的客户端程序`redis-cli`

* 启动`src`目录下服务器

```
cd src
./redis-server
```

* `redis.conf` 默认配置文件 , 启动命令后面跟上配置文件, 可以改变默认配置

* 客户端程序`redis-cli`, 可以进行测试

### 3. Mac OS 系统
* `Mac` 使用 `homebrew` 安装 `redis`

```
brew install redis
```

* 成功标志

```
To have launchd start redis now and restart at login:
  brew services start redis
Or, if you don't want/need a background service you can just run:
  redis-server /usr/local/etc/redis.conf
==> Summary
🍺  /usr/local/Cellar/redis/4.0.6: 13 files, 2.8MB
```

* 启动

```
redis-server /usr/local/etc/redis.conf
```

* `redis.conf` 配置文件

## 应用场景

* 高速缓存
* 消息队列 , 功能简单


### 相关链接

* [redis 官网](http://www.redis.io/)
* [维基百科 Redis 定义](https://zh.wikipedia.org/wiki/Redis)
* [易百教程 Redis](http://www.yiibai.com/redis/redis_quick_guide.html)
* [RUNOOB.COM Redis](http://www.runoob.com/redis/redis-intro.html)

