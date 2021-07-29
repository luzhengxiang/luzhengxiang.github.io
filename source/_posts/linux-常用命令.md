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

tail -f 实时查看日志文件 tail -f 日志文件log
tail - 100f 实时查看日志文件 后一百行
tail -f -n 100 catalina.out linux查看日志后100行

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

copy -r aaaa bbbb  //复制aaaa 到bbbb

```


## 定时任务

``` shell

crontab -help

crontab -l  查看系统定时任务


```


## 系统性能

``` shell
#查看内存使用情况
free -h
# Mem：表示物理内存统计
# total：表示物理内存总量(total = used + free)
# used：表示总计分配给缓存（包含buffers 与cache ）使用的数量，但其中可能部分缓存并未实际使用。
# free：未被分配的内存。shared：共享内存。
# buffers：系统分配但未被使用的buffers 数量。cached：系统分配但未被使用的cache 数量。
# -/+ buffers/cache：表示物理内存的缓存统计
# used2：也就是第一行中的used – buffers-cached 也是实际使用的内存总量。 //used2为第二行
# free2= buffers1 + cached1 + free1 //free2为第二行、buffers1等为第一行
# free2：未被使用的buffers 与cache 和未被分配的内存之和，这就是系统当前实际可用内存。
# Swap：表示硬盘上交换分区的使用情况

# 查看磁盘使用情况
df -h

# 查看磁盘使用情况
du -sh * 

#调正服务器时间
ntpdate ntp.api.bz



```