---
layout: post draft
title: nginx config
date: 2022-03-16 21:34:40
tags: nginx学习
---

### nginx  配置
#### express
- nginx缓存功能
nginx通过配置，可以告知浏览器，返回数量的有效时间，浏览器可以根据数据的有效时间，确定是否应该到服务器请求，如果数据没有超过有效期，就使用浏览器缓存的数据。
优点：可以让用户快速获取使用数据，减少带宽压力
```js
#缓存图片文件
{
  server {
    location ~ \.(jepg|jpg|png)$ {
      # 1d: 一天  1h: 1小时 
      express 1d:
    
    }
  }
}
```
#### gzip
压缩资源通过网络发送的大小就更加节省资源，带宽展示变小。启用压缩机制，为了能够快速访问到资源。
web服务器进行压缩，浏览器需要进行解压缩操作，目前市场上大部分浏览器式支持gzip压缩的

```js
{
  keepalive-timeout 60;
  # 开启gzip
  gzip on;

  # 启用gzip压缩的最小文件；小于设置值的文件将不会被压缩
  gzip_min_length 1k;

  # gzip 压缩级别 1-10 
  gzip_comp_level 2;

  # 进行压缩的文件类型。
  gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;

  # 是否在http header中添加Vary: Accept-Encoding，建议开启
  gzip_vary on;
}
```