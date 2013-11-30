---
title: 修改默认配置使得php页面错误信息显示出来(zz)
author: codeboy
layout: post
permalink: /2012/05/14/%e4%bf%ae%e6%94%b9%e9%bb%98%e8%ae%a4%e9%85%8d%e7%bd%ae%e4%bd%bf%e5%be%97php%e9%a1%b5%e9%9d%a2%e9%94%99%e8%af%af%e4%bf%a1%e6%81%af%e6%98%be%e7%a4%ba%e5%87%ba%e6%9d%a5zz/
categories:
  - php
  - ubuntu配置
tags:
  - apache
  - php
---
使用PHP Apache编程，在缺省设置下，PHP编码错误是不会提示的，这对于开发来说，是很不方便的。可以使用以下步骤打开出错提示：  
1. 打开php.ini文件。  
以我的ubuntu为例，这个文件在： /etc/php5/apache2 目录下。  
2. 搜索并修改下行，把Off值改成On  
display_errors = Off  
3. 搜索下行  
error\_reporting = E\_ALL & ~E_NOTICE  
或者搜索：  
error\_reporting = E\_ALL & ~E_DEPRECATED  
修改为  
error\_reporting = E\_ALL | E_STRICT  
4. 修改Apache的 httpd.conf，  
以我的 Ubuntu 为例， 这个文件在：/etc/apache2/ 目录下，这是一个空白文件。  
添加以下两行：  
php\_flag display\_errors on  
php\_value error\_reporting 2039  
5. 重启Apache，就OK了。  
重启命令： ：sudo /etc/init.d/apache2 restart

转载自<http://www.phptogether.com/archives/7491>