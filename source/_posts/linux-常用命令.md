---
title: linux 常用命令
date: 2019-06-12 00:37:09
tags: linux 
---

# 工作中常用的命令记录

## 查询

``` shell
lsof -i:端口号 查看端口被什么进程占用

netstat -tunlp 查看端口使用情况。

ps -ef|grep xxx 查询xxx的进程使用情况

文件中根据关键字搜索
Cat filename | grep XXX |grep XXX
例如：
cat zhang8online3.0.log |grep 50641658625520

文件中根据关键字统计行数
场景：查询接口调用次数
cat localhost_access_log.2018-06-06.txt |grep getverifycode |wc -l
例如：
cat localhost_access_log.2018-06-07.txt |grep getverifycode |wc -l

查询中使用正则匹配
场景：查询耗时长的接口。
cat zhang8online3.0.log | grep -E "耗时：[0-9]{4,}"

df -h 查询 磁盘使用情况

```


## 操作命令

``` shell

往文件中写值，覆盖文件内容。在不删除文件的情况下清空文件内容
echo "" > ZH8.online.trade.log

chmod 777 文件名   文件给权限

解压tar包
tar -xvf file.tar //解压 tar包
tar -xzvf file.tar.gz //解压tar.gz
tar -xjvf file.tar.bz2   //解压 tar.bz2
tar -xZvf file.tar.Z   //解压tar.Z
unrar e file.rar //解压rar
unzip file.zip //解压zip



ctrl+u //删除光标前所有的内容

```


## 定时任务

``` shell

crontab -help

crontab -l  查看系统定时任务



```