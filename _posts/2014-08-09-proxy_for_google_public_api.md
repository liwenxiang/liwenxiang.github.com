---
layout: post
title: "解决因谷歌公共js,font库访问不了，造成网站加载缓慢"
description: ""
category: "web"
tags: []
---

# 本地解决因谷歌公共js,font库访问不了，网站加载缓慢问题
## 问题产生原因
国外很多网站使用了google的公共js和font库，导致在国内用户在访问这些网站的时候，js和字体加载不出来，画面一直是白屏
## 解决方案
360提供了一个Google js和font库的镜像cdn库，但是这需要网站的站长来改，对于浏览网页的人来说还是没办法，
### 主要思想
在本地提供一个转发，将Google的请求都转到360去。
### 软件安装
1. /etc/hosts文件增加如下内容,将Google请求劫持到本地

    127.0.0.1 ajax.googleapis.com
    127.0.0.1 fonts.googleapis.com

2. 本地安装nginx, 将浏览转发到360, nginx配置如下, 主要是`proxy_set_header HOST ajax.useso.com`这行

    server {
        listen 80;
        server_name ajax.googleapis.com;
    
        location / {
            proxy_set_header HOST ajax.useso.com;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass   http://ajax.useso.com;
        }
        allow 127.0.0.1;
        deny all;
    }
    server {
        listen 80;
        server_name fonts.googleapis.com;
    
        location / {
            proxy_set_header HOST fonts.useso.com;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass   http://fonts.useso.com;
        }
        allow 127.0.0.1;
        deny all;
    }

## 后记
对于XX站点，都可以通过本地hosts文件配合nginx转发，来将访问墙外流量转发到墙内镜像来解决。

{% include JB/setup %}

