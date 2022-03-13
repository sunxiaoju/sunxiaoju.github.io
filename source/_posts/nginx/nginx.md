---
layout: post draft
title: nginx
date: 2022-03-13 15:34:40
tags: nginx学习
---


## nginx 入门

### nginx 简介

#### nginx概述 

#### nginx 特点

#### 正向代理
在客户端或者浏览器，
#### 反向代理
#### 负载均衡
单个服务器解决不了的问题，我们增加服务器数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的情况改为将请求分发到多个服务器上，将负载分发到不同的服务器，也就是我们所说的负载均衡
#### 动静分离
为了加快网站的解析速度，可以吧动态页面和静态界面由不同的服务器来解析，加快服务器的解析速度，降低原来单个服务器的压力


<!-- more -->

### nginx 安装--linux为例
- 安装地址： http://nginx.org

依赖：
- pcre
- zlib zlib-devel
- gcc-c++
- libtool
- openssl openssl-devel

将依赖安装到src目录下

#### 安装pcre依赖，
- 把安装的文件放到linux系统中 直接拖过去即可
- 解压文件  tar -xvf ***(压缩包文件)
- 进入解压之后目录，执行 ./configure
- 使用： make && make install    文件做编译，并安装
- 查看配置文件：pcre-config --version

#### 安装其他依赖
```
yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel
```

#### 安装nginx
mac电脑安装： https://www.jianshu.com/p/4f433d219ab7

- 把安装的文件放到linux系统中 直接拖过去即可
- 解压文件  tar -xvf ***(压缩包文件)
- 进入解压之后目录，执行 ./configure
- 使用： make && make install    文件做编译，并安装
- 安装成功之后 /usr 下会多出来一个文件夹local/nginx，在nginx中有sbin有启动脚本

### nginx 启动
```js
cd /usr/local/nginx/sbin/
./nginx
```
```js

```

增加端口号
```js
//查看防火墙开放端口号
filewall-cmd --list-all
// 设置开放端口号
sudo firewall-cmd --add-port-port=8001/tcp --permanent
// 设置完端口号中重启防火墙
firewall-cmd --reload
```


### nginx 常用命令
使用nginx操作命令必须进入到nginx的目录中: /usr/local/nginx/sbin

```js
//查看版本号
./nginx -v
// 启动
./nginx 
// 关闭
./nginx -s stop
// 重新加载nginx
./nginx -s reload
// 查看所有nginx 命令
./nginx -h
// 查看进程
pm -ef | grep nginx
```

### nginx 配置文件
- nginx 配置文件在什么位置
  /usr/local/nginx/conf/nginx.conf
- nginx 配置文件
  - 全局块
  从配置文件开始到events块之间的内容，主要会设置一些影响nginx服务器整体运行的配置指令
  主要包括配置运行的nginx服务器的用户（组），允许生成的worker process数，进程PID存放路径、日志存放路径和类型以及配置文件的引入等
  ```js
  #user  nobody;
  # nginx 服务器处理的并发数量，值越发处理并发量也越大
  worker_processes  1;
  #error_log  logs/error.log;
  #error_log  logs/error.log  notice;
  #error_log  logs/error.log  info;
  #pid        logs/nginx.pid;
  ```
  - events块
    涉及的质量主要影响nginx与用户的网络链接， 此部分内容对nginx的性能影响较大，在实际中应该灵活配置
    ```js
    events {
      // 支持的最大连接数
      worker_connections  1024;
    } 
    ```
  - http块: 包含 http 全局块，server块
    这里是nginx配置中最频繁的部分，代理。缓存和日志定义等绝大多数功能和第三方模块都配置再这里
    - http全局块: 配置的指令包括文件引入，mime-type定义、日志自定义、连接超时时间、单链接请求书上限等
    ```js
    http {
      include       mime.types;
      default_type  application/octet-stream;

      #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
      #                  '$status $body_bytes_sent "$http_referer" '
      #                  '"$http_user_agent" "$http_x_forwarded_for"';
      #access_log  logs/access.log  main;
      sendfile        on;
      #tcp_nopush     on;
      #keepalive_timeout  0;
      keepalive_timeout  65;
      #gzip  on;
    }
    ```
    - server块: 和虚拟主机有密切关系，包含，全局server和localtion
    ```js
    http {
      server {
        #nginx 目前监听的端口号
        listen       8080;
        # 主机名称
        server_name  localhost;
        # 请求拦截块 
        location / {
          root   html;
          index  index.html index.htm;
        }
      }
    }
    ```

#### nginx 配置实例 --- 反向代理
- 实现效果
 + 打开浏览器，在浏览器中输入地址：www.123.com 跳转到Linux系统的baidu.com主页中 

- 准备 ----- linux 中安装tomact
  - 安装openJDK -java运行环境
  - 把安装的文件放到linux系统中 直接拖过去即可
  - 解压文件  tar -xvf ***(压缩包文件)
  - 进入解压之后的文件中,然后进入/bin 文件下
  - .startup.sh  启动 tomact
  - 进入 cd logs
  - 查看启动日志 : tail -f catalina.out

- 对外开放8080端口号
```js
// 设置开放端口号
sudo firewall-cmd --add-port-port=8080/tcp --permanent
// 设置完端口号中重启防火墙
firewall-cmd --reload
```

- 实践
 1. 修改hosts文件，把 127.0.0.1 www.123.com 
 2. nginx进行请求转发 
 ```js
 location / {
   // 当访问服务访问的域名，指向百度
  // 不会修改访问url只会将原来的域名改为此路径
   proxy_pass  https://www.baidu.com
 }
 ```
