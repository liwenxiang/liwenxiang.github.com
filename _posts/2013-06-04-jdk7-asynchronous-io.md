---
title: jdk7 Asynchronous I/O
author: codeboy
layout: post
permalink: /2013/06/04/jdk7-asynchronous-io/
categories:
  - java
tags:
  - java
  - jdk7
  - nio2
---
AIO参考（[参考链接][1]）

主要思想是：read(buffer, callback)函数立即返回，当keneral完成ＩＯ操作之后，会调用回调函数，同时数据已经写入此buffer中了。

在java/nio/channels/下的几个AsynchronousXXXChannel有完成此功能：

实现AIO的两种方式：

1，Future read(ByteBuffer dst）；调用立即返回，然后可作为future.get()来操作，或者把future传到Executor中去。

2，read(ByteBuffer dst, CompletionHandler handler); 调用立即返回，当keneral完成IO操作，将数据拷贝到dst中之后，会调用CompletionHandler(在后台重新启动一个线程，不会block主线程)  
对于第二中回调函数的方式，调用CompletionHandler的实现方式分为两种：

1，Fixed thread pool

> thread pool中每个线程都在等待是否有自己的IO事件，当等到的时候，把数据拷贝到Buffer中，处理CompletionHandler ，处理结束在回到thread pool中继续等待
> 
> [<img style="background-image: none; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="QQ截图20130605002812" alt="QQ截图20130605002812" src="http://www.codeboy.name/wp-content/uploads/2013/06/QQ20130605002812_thumb.png" width="353" height="148" border="0" />][2]
> 
> 可能问题：如果CompletionHandler 函数会阻塞，那么可能造成thread pool没有线程可用来从keneral中拷贝数据。

2，Cached or custom thread pool

由keneral内部不可见的线程池来等待IO事件，当等到的时候，keneral内部拷贝数据到buffer中，然后从thread pool中取得一个thread，让这个thread 来处理CompletionHandler，处理结束之后把这个thread再放回thread pool

[<img style="background-image: none; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="QQ截图20130605003210" alt="QQ截图20130605003210" src="http://www.codeboy.name/wp-content/uploads/2013/06/QQ20130605003210_thumb.png" width="338" height="143" border="0" />][3]

可能问题：同上，CompletionHandler 函数不要阻塞，否则可能造成keneral拷贝了数据到buffer中，但是没有thread可以处理，这样可能造成OOM；另外，如果同时有大量的数据到来，会把keneral撑爆OOM

参考（<http://openjdk.java.net/projects/nio/presentations/TS-4222.pdf>）

 [1]: http://www.codeboy.name/2012/06/19/linux-%E4%B8%8A%E5%8F%AF%E4%BB%A5%E4%BD%BF%E7%94%A8%E7%9A%84%E4%B8%8D%E5%90%8C-io-%E6%A8%A1%E5%9E%8B%EF%BC%88zz%EF%BC%89/
 [2]: http://www.codeboy.name/wp-content/uploads/2013/06/QQ20130605002812.png
 [3]: http://www.codeboy.name/wp-content/uploads/2013/06/QQ20130605003210.png