---
title: 编译vim，加入gdb debug功能
author: codeboy
layout: post
permalink: /2012/12/14/%e7%bc%96%e8%af%91vim%ef%bc%8c%e5%8a%a0%e5%85%a5gdb-debug%e5%8a%9f%e8%83%bd/
categories:
  - vim
tags:
  - vim
---
下载vim7.2源代码

[ftp://ftp.vim.org/pub/vim/unix/vim-7.3.tar.bz2][1]

下载vimgdb的patch代码

<http://nchc.dl.sourceforge.net/project/clewn/vimGdb/vimgdb72-1.14/vimgdb72-1.14.tar.gz>

解压

tar xjf vim-7.2.tar.bz2   
tar xzf vimgdb72-1.14.tar.gz

patch -d vim72 &#8211;backup -p0 < vimgdb/vim72.diff

cd vim72/src

编辑Makefile文件

打开

CONF\_OPT\_GUI = &#8211;disable-gui

CONF\_OPT\_FEAT = &#8211;with-features=huge

CONF\_OPT\_INPUT = &#8211;enable-xim

CONF\_OPT\_MULTIBYTE = &#8211;enable-multibyte

CONF\_OPT\_GDB = &#8211;enable-gdb

首先测试下make test，可能有如下错误

checking for tgetent()&#8230; configure: error: NOT FOUND!   
You need to install a terminal library: for example ncurses.   
Or specify the name of the library with &#8211;with-tlib.

需要安装libncurses5-dev，

安装之后再make test，如果可以了

make clean ，

make CFLAGS="-O2 -D\_FORTIFY\_SOURCE=1" (没加的话安装之后每次使用vim都是显示\*\\*\* buffer overflow detected \*\**: vim terminated)

sudo make install

把vimgdb下面的vimgdb_runtime.tgz解压到.vim目录

打开vim， 

：helptag ~/.vim/doc/

: help vimgdb

 [1]: ftp://ftp.vim.org/pub/vim/unix/vim-7.3.tar.bz2 "ftp://ftp.vim.org/pub/vim/unix/vim-7.3.tar.bz2"