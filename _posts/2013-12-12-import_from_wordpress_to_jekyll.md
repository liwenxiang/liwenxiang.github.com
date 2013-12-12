---
layout: post
title: "使用exitwp从Wordpress倒入文章到jekyll"
description: ""
category: "web"
tags: ["建站"]
---
1. git clone git://github.com/thomasf/exitwp.git
2. sudo pip install --upgrade  -r pip_requirements.txt
>可能需要创建文件~/.pip/pip.conf内容如下  
>[global]  
>index-url = http://pypi.v2ex.com/simple  
>
3. 更改config.yaml中的`target_format: html`这个是因为默认转换出md格式，可能又太多错误，还不如html正确性高
4. python exitwp.py
5. 将build目录下面的_post/下的文件拷贝到自己的_posts/目录下面就可以了


{% include JB/setup %}
