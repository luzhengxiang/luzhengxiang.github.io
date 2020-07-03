---
title: nginx的安装以及简单配置
date: 2020-07-01 10:26:37
tags: nginx linux 
---


# 在linux下安装

## yum自动安装
``` shell
yum install -y nginx

```
可以直接nginx来执行命令，

## 手动安装
``` shell
//一键安装上面四个依赖
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel

//创建一个文件夹
cd /usr/local
mkdir nginx
cd nginx
//下载tar包
wget http://nginx.org/download/nginx-1.13.7.tar.gz
tar -xvf nginx-1.13.7.tar.gz

//进入nginx目录
cd /usr/local/nginx
//进入目录
cd nginx-1.13.7
//执行命令
./configure
// ./configure --with-stream
//执行make命令
make
//执行make install命令
make install

```
手动安装完的需要去/usr/local/nginx/sbin 下操作nginx


# nginx 简单命令

``` shell
快速停止或关闭Nginx：nginx -s stop
正常停止或关闭Nginx：nginx -s quit
配置文件修改重装载命令：nginx -s reload
启动 nginx
查看nginx版本 nginx -v

```



# 实际场景配置

## 负载均衡 + 简单路径请求代理
```

#user  nobody;
worker_processes  1;

events {
    worker_connections  1024;
}

http {
   
   upstream  linuxidc{        #定义upstream名字，下面会引用
        server localhost:8080 weight=5;        #指定后端服务器地址
        server localhost:8081 weight=5;        #指定后端服务器地址
    }


    server {
        listen       8089;

        location /hclp/ {
            proxy_pass http://localhost:8082/;
        }

        location / {
            proxy_pass http://linuxidc;        #引用upstream
        }

    }

    include servers/*;
}

```