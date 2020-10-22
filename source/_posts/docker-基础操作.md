---
title: docker 基础操作
date: 2020-06-16 11:01:45
tags: docker 
---

# docker常用命令
```

docker ps 查看启动的docker容器列表
docker ps -a 查看所有容器列表
docker rm -f 1e560fca3906 删除容器
docker stop 容器id 关闭容器
docker start 容器id 启动一个关闭的容器
docker restart <容器 ID> 重启
docker attach 1e560fca3906    进入容器
docker exec -it 243c32535da7 /bin/bash 进入容器
退出已经进入的容器：exit。   
attach进入容器退出会导致容器关闭，exec进入容器使用exit退出不会导致容器关闭。

docker images 列出镜像列表
docker pull ubuntu:13.10  下载镜像
docker search ubuntu    搜索镜像
docker rmi hello-world 删除镜像


docker save -o nacos-server.tar nacos/nacos-server:latest
将本地镜像保存成文件

docker load --input nacos-server.tar 加载文件中的镜像
docker load < busybox.tar.gz 加载文件中的镜像


容器内 安装vim编辑器
apt-get update
apt-get install vim


```

# 在linux安装docker

``` 
# 安装yum-utils：
yum install -y yum-utils device-mapper-persistent-data lvm2
# 为yum源添加docker仓库位置：
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 安装docker：
yum install docker-ce
# 启动docker：
systemctl start docker
```

# docker环境 mysql安装 
``` shell
#下载mysql5.7的docker镜像：
docker pull mysql:5.7

#使用docker命令启动：
docker run -p 3306:3306 --name mysql \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root  \
-d mysql:5.7

#参数说明
-p 3306:3306：将容器的3306端口映射到主机的3306端口
-v /mydata/mysql/conf:/etc/mysql：将配置文件夹挂在到主机
-v /mydata/mysql/log:/var/log/mysql：将日志文件夹挂载到主机
-v /mydata/mysql/data:/var/lib/mysql/：将数据文件夹挂载到主机
-e MYSQL_ROOT_PASSWORD=root：初始化root用户的密码


# 进入运行mysql的docker容器：
docker exec -it mysql /bin/bash

# 使用mysql命令打开客户端：
mysql -uroot -proot --default-character-set=utf8

```

# docker安装redis
``` shell
# 下载redis3.2的docker镜像：
docker pull redis:3.2

# 使用docker命令启动：
docker run -p 6379:6379 --name redis \
-v /mydata/redis/data:/data \
-d redis:3.2 redis-server --appendonly yes

# 进入redis容器使用redis-cli命令进行连接：
docker exec -it redis redis-cli

```

# 容器内操作
## 容器内安装组件
因为容器一般都是最简单安装，很多组件没有，需要进入容器后手动安装
```
apt-get update
apt-get install vim
apt-get install yum
```
