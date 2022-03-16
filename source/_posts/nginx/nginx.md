---
layout: post draft
title: nginx
date: 2022-03-13 15:34:40
tags: nginx学习
---


## nginx 入门
#### nginx 特点
#### 正向代理
正向代理，意思是一个位于客户端和原始服务器(origin server)之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。客户端才能使用正向代理。
#### 反向代理
反向代理（Reverse Proxy）方式是指以代理服务器来接受Internet上的连接请求，然后将请求转发给内部网络上的服务器；并将从服务器上得到的结果返回给Internet上请求连接的客户端，
此时代理服务器对外就表现为一个服务器。
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

#### nginx 配置实例 --- 负载均衡
nginx分配服务器策略
- 轮询（默认）： 每个请求按照时间顺序逐一分配到不用的后端服务器，如果后端服务器domn掉，能自动剔除
```js
// 负载均衡名字
stream myServer {
  server 192.168.10.1:8080;
  server 192.168.10.1:8081;
}

location / {
  proxy_pass  https://myServer
}
```
- weight ： 代表权重默认为1 ，权重越高被分配的客户端就越多,按比例分配
```js
// 负载均衡名字
stream myServer {
  server 192.168.10.1:8080 weight=5;
  server 192.168.10.1:8081 weight=10;
}

location / {
  proxy_pass  https://myServer
}
```

- ip_hash：每个请求按照访问ip的hash结果分配，这样每个方可固定访问一个后端服务器（服务器设置session时，会用到）
```js
// 负载均衡名字
stream myServer {
  ip_hash
  server 192.168.10.1:8080;
  server 192.168.10.1:8081;
}

location / {
  proxy_pass  https://myServer
}
```

- fair(第三方)： 按照后端服务器的响应时间来分配请求，响应时间短的有限分配
```js
// 负载均衡名字
stream myServer {
  server 192.168.10.1:8080;
  server 192.168.10.1:8081;
  fair;
}

location / {
  proxy_pass  https://myServer
}
```

#### nginx 配置实例--- 动静分离
nginx动静分离简单的来说就是把动态和静态请求分开，不能理解成制单成的吧动态界面和静态界面物理分离，严格意义上来说应该是动态请求和静态请求分离开，可以理解成nginx处理静态请求，tacmat来处理请求

在服务创建data文件夹，在data中分别创建 www和image文件夹

```js
// 访问： http://***/www/index.html 可以直接访问到data下的www文件夹中
location /www/ {
  root /data/;
  index index.html index.htm;
}
// 访问： http://***/image/01.jpg 可以直接访问到data下的image
location /image/ {
  root /data/;
  // 列出当前文件夹中的目录内容
  autoindex on;
}
```

#### nginx 高可用

- 服务器上安装keepalived
```js
// 安装
yum install keepalived -y
// 查看是否安装
rpm -q -a keepalived
```
- 在etc中生成一个目录 keepalived, 有一个文件keepalived.conf

##### 配置
```js
// 脚本文件 检测服务是否可用


vrrp_instanve VI_1 {
  state MASTER   # MATER 主服务器  BACKUP 备份服务器
  interface ens22 # 网卡
  virtual_router_id 51 # 主、备服务器必须相同
  priority 100 # 主、备机取不同的优先级，主机值较大。，备份机较小
  advert_int 1 
  authentication {
    auth_type PASS
    auth_pass 1111
  }
  virtual_ipaddress {
    192.168.17.50 # VRRP H虚拟主机
  }
}
```

### nginx 原理解析

设置worker数量： 一个worker可以吧一个cpu发挥到极致，worker数跟cpu数量向东

- 连接数 worker-connection
  - 一个请求，占用多个worker几个连接数：  2个或者4个 静态资源：2  反向代理链接java：4
  - nginx 有一个master，有四个worker，每个worker支持最大的连接数据1024，支持最大并发数是多少？
    + 静态访问 worker_connection*worker_processes/2
    + http 反向代理 worker_connection*worker_processes/4