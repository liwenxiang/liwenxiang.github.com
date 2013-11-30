---
title: How to Analyze Java Thread Dumps(zz)
author: codeboy
layout: post
permalink: /2012/06/06/how-to-analyze-java-thread-dumps-cubrid-blog/
categories:
  - 程序语言小记录
  - 转载
tags:
  - java
---
<h1 class="postTitle" style="width: 727px; margin-top: 10px; margin-right: 0px; margin-bottom: 0px; margin-left: 0px; font-size: 34px; text-align: left; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif; line-height: normal;">
  <a style="color: #1e598e; text-decoration: none;" href="http://www.cubrid.org/blog/dev-platform/how-to-analyze-java-thread-dumps/">How to Analyze Java Thread Dumps</a>
</h1>

<p class="postMeta" style="font-family: Arial, sans-serif; color: #999999; line-height: 25px; font-size: 11px;">
  posted <span class="yymmdd" style="display: inline-block;" title="2012/02/15 @ 23:13">3 months ago</span> in <span class="category" style="display: inline-block; width: auto; margin-top: 0px; margin-right: 0px; margin-bottom: 15px; margin-left: 0px; font-weight: bold; color: #1e598e;">Dev Platform</span> category by <a class="author member_295970" style="color: #1e598e; text-decoration: none;" href="http://www.cubrid.org/blog/dev-platform/how-to-analyze-java-thread-dumps/#"><img style="border-width: initial; border-style: none;" title="Rank: 0/30, reputation score 1 (16% completed for the next level)" src="http://www.cubrid.org/modules/point/icons/default/0.gif" alt="[Level:0]" />Tae Jin Gu</a>
</p>

<div class="addthis_toolbox addthis_default_style" style="height: 35px; font-family: Arial, Helvetica, sans; line-height: normal;">
  <div id="___plusone_0" style="position: absolute; width: 100px; left: -10000px;" data-gwattr="size=medium:href=http%3A%2F%2Fwww.cubrid.org%2Fblog%2Fdev-platform%2Fhow-to-analyze-java-thread-dumps%2F:callback=_at_plusonecallback">
  </div>
</div>

<div class="textyleContent">
  <div class="document_295971_295970 xe_content">
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      When there is an obstacle, or when a Java based Web application is running much slower than expected, we need to use<strong>thread dumps</strong>. If thread dumps feel like very complicated to you, this article may help you very much. Here I will explain what threads are in Java, their types, how they are created, how to manage them, how you can dump threads from a running application, and finally how you can analyze them and determine the bottleneck or blocking threads. This article is a result of long experience in Java application debugging.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      <!--more-->
    </p>
    
    <h2 style="width: 727px; margin-top: 21px; margin-right: 0px; margin-bottom: 21px; margin-left: 0px; font-size: 20px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      Java and Thread
    </h2>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      A web server uses tens to hundreds of threads to process a large number of concurrent users. If two or more threads utilize the same resources, a <em>contention</em> between the threads is inevitable, and sometimes deadlock occurs.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      <strong>Thread contention</strong> is a status in which one thread is waiting for a lock, held by another thread, to be lifted. Different threads frequently access shared resources on a web application. For example, to record a log, the thread trying to record the log must obtain a lock and access the shared resources.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      <strong>Deadlock</strong> is a special type of thread contention, in which two or more threads are waiting for the other threads to complete their tasks in order to complete their own tasks.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      Different issues can arise from thread contention. To analyze such issues, you need to use the <strong>thread dump</strong>. A thread dump will give you the information on the exact status of each thread.
    </p>
    
    <h2 style="width: 727px; margin-top: 21px; margin-right: 0px; margin-bottom: 21px; margin-left: 0px; font-size: 20px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      Background Information for Java Threads
    </h2>
    
    <h3 style="width: 727px; margin-top: 15px; margin-right: 0px; margin-bottom: 15px; margin-left: 0px; font-size: 14px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      Thread Synchronization
    </h3>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      A thread can be processed with other threads at the same time. In order to ensure compatibility when multiple threads are trying to use shared resources, one thread at a time should be allowed to access the shared resources by using <em>thread synchronization</em>.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      Thread synchronization on Java can be done using <strong>monitor</strong>. Every Java object has a single monitor. The monitor can be owned by only one thread. For a thread to own a monitor that is owned by a different thread, it needs to wait in the wait queue until the other thread releases its monitor.
    </p>
    
    <h3 style="width: 727px; margin-top: 15px; margin-right: 0px; margin-bottom: 15px; margin-left: 0px; font-size: 14px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      Thread Status
    </h3>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      In order to analyze a thread dump, you need to know the status of threads. The statuses of threads are stated on java.lang.Thread.State.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px; text-align: center;">
       <img class="iePngFix" style="border-width: initial; cursor: pointer; border-style: none;" title="Figure 1: Thread Status." src="http://www.cubrid.org/files/attach/images/220547/971/295/thread-state-diagram.png" alt="Figure 1: Thread Status." width="295" height="288" />
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px; text-align: center;">
      <strong>Figure 1: Thread Status.</strong>
    </p>
    
    <ul style="padding-left: 30px; margin-top: 10px; margin-right: 0px; margin-bottom: 20px; margin-left: 0px;">
      <li style="text-align: left; font-family: Arial, sans-serif; color: #474747; margin-top: 5px; margin-right: 0px; margin-bottom: 5px; margin-left: 0px;">
        <strong>NEW</strong>: The thread is created but has not been processed yet.
      </li>
      <li style="text-align: left; font-family: Arial, sans-serif; color: #474747; margin-top: 5px; margin-right: 0px; margin-bottom: 5px; margin-left: 0px;">
        <strong>RUNNABLE</strong>: The thread is occupying the CPU and processing a task. (It may be in <span style="display: inline-block; font-family: monospace;">WAITING</span> status due to the OS&#8217;s resource distribution.)
      </li>
      <li style="text-align: left; font-family: Arial, sans-serif; color: #474747; margin-top: 5px; margin-right: 0px; margin-bottom: 5px; margin-left: 0px;">
        <strong>BLOCKED</strong>: The thread is waiting for a different thread to release its lock in order to get the monitor lock.
      </li>
      <li style="text-align: left; font-family: Arial, sans-serif; color: #474747; margin-top: 5px; margin-right: 0px; margin-bottom: 5px; margin-left: 0px;">
        <strong>WAITING</strong>: The thread is waiting by using a <span style="display: inline-block; font-family: monospace;">wait, join</span> or <span style="display: inline-block; font-family: monospace;">park</span> method.
      </li>
      <li style="text-align: left; font-family: Arial, sans-serif; color: #474747; margin-top: 5px; margin-right: 0px; margin-bottom: 5px; margin-left: 0px;">
        <strong>TIMED_WAITING</strong>: The thread is waiting by using a <span style="display: inline-block; font-family: monospace;">sleep</span>, <span style="display: inline-block; font-family: monospace;">wait</span>, <span style="display: inline-block; font-family: monospace;">join</span> or <span style="display: inline-block; font-family: monospace;">park</span> method. (The difference from <span style="display: inline-block; font-family: monospace;">WAITING</span> is that the maximum waiting time is specified by the method parameter, and <em>WAITING</em> can be relieved by time as well as external changes.)
      </li>
    </ul>
    
    <h3 style="width: 727px; margin-top: 15px; margin-right: 0px; margin-bottom: 15px; margin-left: 0px; font-size: 14px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      Thread Types
    </h3>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      Java threads can be divided into two:
    </p>
    
    <ol style="padding-left: 30px;">
      <li style="text-align: left; font-family: Arial, sans-serif; color: #474747; margin-top: 5px; margin-right: 0px; margin-bottom: 5px; margin-left: 0px;">
        daemon threads;
      </li>
      <li style="text-align: left; font-family: Arial, sans-serif; color: #474747; margin-top: 5px; margin-right: 0px; margin-bottom: 5px; margin-left: 0px;">
        and non-daemon threads.
      </li>
    </ol>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      <strong>Daemon threads</strong> stop working when there are no other non-daemon threads. Even if you do not create any threads, the Java application will create several threads by default. Most of them are daemon threads, mainly for processing tasks such as garbage collection or JMX.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      A thread running the &#8216;<span style="display: inline-block; font-family: monospace;">static void main(String[] args)</span>’ method is created as a non-daemon thread, and when this thread stops working, all other daemon threads will stop as well. (The thread running this main method is called the <strong>VM thread in HotSpot VM</strong>.)
    </p>
    
    <h2 style="width: 727px; margin-top: 21px; margin-right: 0px; margin-bottom: 21px; margin-left: 0px; font-size: 20px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      Getting a Thread Dump
    </h2>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      We will introduce the three most commonly used methods. Note that there are many other ways to get a thread dump. A thread dump can only show the thread status at the time of measurement, so in order to see the change in thread status, it is recommended to extract them from 5 to 10 times with 5-second intervals.
    </p>
    
    <h3 style="width: 727px; margin-top: 15px; margin-right: 0px; margin-bottom: 15px; margin-left: 0px; font-size: 14px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      Getting a Thread Dump Using jstack
    </h3>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      In JDK 1.6 and higher, it is possible to get a thread dump on MS Windows using <strong>jstack</strong>.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      Use PID via jps to check the PID of the currently running Java application process.
    </p>
    
    <div>
      <div id="highlighter_776630" class="syntaxhighlighter  bash" style="width: 727px; margin-top: 1em !important; margin-right: 0px !important; margin-bottom: 1em !important; margin-left: 0px !important; position: relative !important; overflow-x: auto !important; overflow-y: auto !important; font-size: 1em !important; background-color: white !important;">
        <div class="toolbar" style="border-top-left-radius: 0px !important; border-top-right-radius: 0px !important; border-bottom-right-radius: 0px !important; border-bottom-left-radius: 0px !important; background-image: initial !important; background-attachment: initial !important; background-origin: initial !important; background-clip: initial !important; background-color: transparent !important; border-style: none !important; border-color: initial !important; border-image: initial !important; bottom: auto !important; float: none !important; height: 11px !important; left: auto !important; line-height: 1.1em !important; outline-width: 0px !important; outline-style: initial !important; outline-color: initial !important; overflow-x: visible !important; overflow-y: visible !important; position: absolute !important; right: 1px !important; text-align: left !important; top: 1px !important; vertical-align: baseline !important; width: 11px !important; box-sizing: content-box !important; font-family: Consolas, 'Bitstream Vera Sans Mono', 'Courier New', Courier, monospace !important; font-size: 10px !important; min-height: inherit !important; z-index: 10 !important; color: white !important; border-width: 0px !important; padding: 0px !important; margin: 0px !important;">
          <span style="display: inline-block;"><a class="toolbar_item command_help help" style="color: white !important; text-decoration: none !important; border-top-left-radius: 0px !important; border-top-right-radius: 0px !important; border-bottom-right-radius: 0px !important; border-bottom-left-radius: 0px !important; background-image: none !important; background-attachment: initial !important; background-origin: initial !important; background-clip: initial !important; background-color: initial !important; border-style: initial !important; border-color: initial !important; border-image: initial !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.1em !important; outline-width: 0px !important; outline-style: initial !important; outline-color: initial !important; overflow-x: visible !important; overflow-y: visible !important; padding-top: 1px !important; padding-right: 0px !important; padding-bottom: 0px !important; padding-left: 0px !important; position: static !important; right: auto !important; text-align: center !important; top: auto !important; vertical-align: baseline !important; width: auto !important; box-sizing: content-box !important; font-size: 1em !important; min-height: inherit !important; display: block !important; border-width: 0px !important; margin: 0px !important;" href="http://www.cubrid.org/blog/dev-platform/how-to-analyze-java-thread-dumps/#">?</a></span>
        </div>
        
        <div style="text-align: left;">
          <div>
            <code>[user@linux ~]$ ups -</code><code>v</code>
          </div>
          
          <div>
          </div>
          
          <div>
            <code>25780 RemoteTestRunner -Dfile.encoding=UTF-8</code>
          </div>
          
          <div>
            <code>25590 sub.rmi.registry.RegistryImpl 2999 -Dapplication.home=</code><code>/home1/user/java/jdk</code><code>.1.6.0_24 -Xms8m</code>
          </div>
          
          <div>
            <code>26300 sun.tools.jps.Jps -mlvV -Dapplication.home=</code><code>/home1/user/java/jdk</code><code>.1.6.0_24 -Xms8m</code>
          </div>
          
          <p>
            <span style="font-family: Consolas, 'Bitstream Vera Sans Mono', 'Courier New', Courier, monospace;"><span style="line-height: 14px;"><br /> </span></span>
          </p>
        </div>
      </div>
    </div>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      Use the extracted PID as the parameter of jstack to obtain a thread dump.
    </p>
    
    <div>
      <div id="highlighter_34751" class="syntaxhighlighter  bash" style="width: 727px; margin-top: 1em !important; margin-right: 0px !important; margin-bottom: 1em !important; margin-left: 0px !important; position: relative !important; overflow-x: auto !important; overflow-y: auto !important; font-size: 1em !important; background-color: white !important;">
        <div class="toolbar" style="border-top-left-radius: 0px !important; border-top-right-radius: 0px !important; border-bottom-right-radius: 0px !important; border-bottom-left-radius: 0px !important; background-image: initial !important; background-attachment: initial !important; background-origin: initial !important; background-clip: initial !important; background-color: transparent !important; border-style: none !important; border-color: initial !important; border-image: initial !important; bottom: auto !important; float: none !important; height: 11px !important; left: auto !important; line-height: 1.1em !important; outline-width: 0px !important; outline-style: initial !important; outline-color: initial !important; overflow-x: visible !important; overflow-y: visible !important; position: absolute !important; right: 1px !important; text-align: left !important; top: 1px !important; vertical-align: baseline !important; width: 11px !important; box-sizing: content-box !important; font-family: Consolas, 'Bitstream Vera Sans Mono', 'Courier New', Courier, monospace !important; font-size: 10px !important; min-height: inherit !important; z-index: 10 !important; color: white !important; border-width: 0px !important; padding: 0px !important; margin: 0px !important;">
          <span style="display: inline-block;"><a class="toolbar_item command_help help" style="color: white !important; text-decoration: none !important; border-top-left-radius: 0px !important; border-top-right-radius: 0px !important; border-bottom-right-radius: 0px !important; border-bottom-left-radius: 0px !important; background-image: none !important; background-attachment: initial !important; background-origin: initial !important; background-clip: initial !important; background-color: initial !important; border-style: initial !important; border-color: initial !important; border-image: initial !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.1em !important; outline-width: 0px !important; outline-style: initial !important; outline-color: initial !important; overflow-x: visible !important; overflow-y: visible !important; padding-top: 1px !important; padding-right: 0px !important; padding-bottom: 0px !important; padding-left: 0px !important; position: static !important; right: auto !important; text-align: center !important; top: auto !important; vertical-align: baseline !important; width: auto !important; box-sizing: content-box !important; font-size: 1em !important; min-height: inherit !important; display: block !important; border-width: 0px !important; margin: 0px !important;" href="http://www.cubrid.org/blog/dev-platform/how-to-analyze-java-thread-dumps/#">?</a></span>
        </div>
        
        <div style="text-align: left;">
          <span style="font-family: Consolas, 'Bitstream Vera Sans Mono', 'Courier New', Courier, monospace;"><span style="line-height: 14px;">[user@linux ~]$ jstack -f5824</span></span>
        </div>
      </div>
    </div>
    
    <h3 style="width: 727px; margin-top: 15px; margin-right: 0px; margin-bottom: 15px; margin-left: 0px; font-size: 14px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      A Thread Dump Using jVisualVM
    </h3>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      Generate a thread dump by using a program such as jVisualVM.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px; text-align: center;">
      <img class="iePngFix" style="border-width: initial; cursor: pointer; border-style: none;" title="Figure 2: &nbsp;A Thread Dump Using visualvm." src="http://www.cubrid.org/files/attach/images/220547/971/295/thread-dump-using-jvisualvm.png" alt="Figure 2: &nbsp;A Thread Dump Using visualvm." width="689" height="520" />
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px; text-align: center;">
      <strong>Figure 2:  A Thread Dump Using visualvm.</strong>
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      The task on the left indicates the list of currently running processes. Click on the process for which you want the information, and select the thread tab to check the thread information in real time. Click the <span style="display: inline-block; font-family: monospace;">Thread Dump</span> button on the top right corner to get the thread dump file.
    </p>
    
    <h3 style="width: 727px; margin-top: 15px; margin-right: 0px; margin-bottom: 15px; margin-left: 0px; font-size: 14px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      Generating in a Linux Terminal
    </h3>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      Obtain the process <span style="display: inline-block; font-family: monospace;">pid</span> by using <span style="display: inline-block; font-family: monospace;">ps -ef</span> command to check the pid of the currently running Java process.
    </p>
    
    <div>
      <div id="highlighter_819648" class="syntaxhighlighter  bash" style="width: 727px; margin-top: 1em !important; margin-right: 0px !important; margin-bottom: 1em !important; margin-left: 0px !important; position: relative !important; overflow-x: auto !important; overflow-y: auto !important; font-size: 1em !important; background-color: white !important;">
        <div class="toolbar" style="border-top-left-radius: 0px !important; border-top-right-radius: 0px !important; border-bottom-right-radius: 0px !important; border-bottom-left-radius: 0px !important; background-image: initial !important; background-attachment: initial !important; background-origin: initial !important; background-clip: initial !important; background-color: transparent !important; border-style: none !important; border-color: initial !important; border-image: initial !important; bottom: auto !important; float: none !important; height: 11px !important; left: auto !important; line-height: 1.1em !important; outline-width: 0px !important; outline-style: initial !important; outline-color: initial !important; overflow-x: visible !important; overflow-y: visible !important; position: absolute !important; right: 1px !important; text-align: left !important; top: 1px !important; vertical-align: baseline !important; width: 11px !important; box-sizing: content-box !important; font-family: Consolas, 'Bitstream Vera Sans Mono', 'Courier New', Courier, monospace !important; font-size: 10px !important; min-height: inherit !important; z-index: 10 !important; color: white !important; border-width: 0px !important; padding: 0px !important; margin: 0px !important;">
          <span style="display: inline-block;"><a class="toolbar_item command_help help" style="color: white !important; text-decoration: none !important; border-top-left-radius: 0px !important; border-top-right-radius: 0px !important; border-bottom-right-radius: 0px !important; border-bottom-left-radius: 0px !important; background-image: none !important; background-attachment: initial !important; background-origin: initial !important; background-clip: initial !important; background-color: initial !important; border-style: initial !important; border-color: initial !important; border-image: initial !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.1em !important; outline-width: 0px !important; outline-style: initial !important; outline-color: initial !important; overflow-x: visible !important; overflow-y: visible !important; padding-top: 1px !important; padding-right: 0px !important; padding-bottom: 0px !important; padding-left: 0px !important; position: static !important; right: auto !important; text-align: center !important; top: auto !important; vertical-align: baseline !important; width: auto !important; box-sizing: content-box !important; font-size: 1em !important; min-height: inherit !important; display: block !important; border-width: 0px !important; margin: 0px !important;" href="http://www.cubrid.org/blog/dev-platform/how-to-analyze-java-thread-dumps/#">?</a></span>
        </div>
        
        <div style="text-align: left;">
          <div>
            <code>[user@linux ~]$ </code><code>ps</code> <code>- ef | </code><code>grep</code> <code>java</code>
          </div>
          
          <div>
          </div>
          
          <div>
            <code>user      2477          1    0 Dec23 ?         00:10:45 ...</code>
          </div>
          
          <div>
            <code>user    25780 25361   0 15:02 pts</code><code>/3</code>    <code>00:00:02 .</code><code>/jstatd</code> <code>-J -Djava.security.policy=jstatd.all.policy -p 2999</code>
          </div>
          
          <div>
            <code>user    26335 25361   0 15:49 pts</code><code>/3</code>    <code>00:00:00 </code><code>grep</code> <code>java</code>
          </div>
          
          <p>
            <span style="font-family: Consolas, 'Bitstream Vera Sans Mono', 'Courier New', Courier, monospace;"><span style="line-height: 14px;"><br /> </span></span>
          </p>
        </div>
      </div>
    </div>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      Use the extracted pid as the parameter of <span style="display: inline-block; font-family: monospace;">kill –SIGQUIT(3)</span> to obtain a thread dump.
    </p>
    
    <h2 style="width: 727px; margin-top: 21px; margin-right: 0px; margin-bottom: 21px; margin-left: 0px; font-size: 20px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      Thread Information from the Thread Dump File
    </h2>
    
    <pre style="overflow-y: scroll; border-image: initial; border-width: 1px; border-color: #888888; border-style: solid; padding: 2px;"><span style="display: inline-block; background-color: #e4ff75;">"pool-1-thread-13"</span> <span style="display: inline-block; background-color: #a6ff4d;">prio=6</span> <span style="display: inline-block; background-color: #99dcff;">tid=0x000000000729a000 nid=0x2fb4</span> <span style="display: inline-block; background-color: #9334d8; color: #ffffff;">runnable [0x0000000007f0f000] java.lang.Thread.State: RUNNABLE</span>
              <span style="display: inline-block; background-color: #8e8e8e; color: #ffffff;"><span style="display: inline-block; color: #000000;">at java.net.SocketInputStream.socketRead0(Native Method) </span></span>
              <span style="display: inline-block; background-color: #8e8e8e; color: #ffffff;"><span style="display: inline-block; color: #000000;">at java.net.SocketInputStream.read(SocketInputStream.java:129) </span></span>
              <span style="display: inline-block; background-color: #8e8e8e; color: #ffffff;"><span style="display: inline-block; color: #000000;">at sun.nio.cs.StreamDecoder.readBytes(StreamDecoder.java:264) </span></span>
              <span style="display: inline-block; background-color: #8e8e8e; color: #ffffff;"><span style="display: inline-block; color: #000000;">at sun.nio.cs.StreamDecoder.implRead(StreamDecoder.java:306) </span></span>
              <span style="display: inline-block; background-color: #8e8e8e; color: #ffffff;"><span style="display: inline-block; color: #000000;">at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:158) </span></span>
              <span style="display: inline-block; background-color: #8e8e8e; color: #ffffff;"><span style="display: inline-block; color: #000000;">- locked &lt;0x0000000780b7e688&gt; (a java.io.InputStreamReader) </span></span>
              <span style="display: inline-block; background-color: #8e8e8e; color: #ffffff;"><span style="display: inline-block; color: #000000;">at java.io.InputStreamReader.read(InputStreamReader.java:167) </span></span>
              <span style="display: inline-block; background-color: #8e8e8e; color: #ffffff;"><span style="display: inline-block; color: #000000;">at java.io.BufferedReader.fill(BufferedReader.java:136) </span></span>
              <span style="display: inline-block; background-color: #8e8e8e; color: #ffffff;"><span style="display: inline-block; color: #000000;">at java.io.BufferedReader.readLine(BufferedReader.java:299) </span></span>
              <span style="display: inline-block; background-color: #8e8e8e; color: #ffffff;"><span style="display: inline-block; color: #000000;">- locked &lt;0x0000000780b7e688&gt; (a java.io.InputStreamReader) </span></span>
              <span style="display: inline-block; background-color: #8e8e8e; color: #ffffff;"><span style="display: inline-block; color: #000000;">at java.io.BufferedReader.readLine(BufferedReader.java:362)</span></span></pre>
    
    <ul style="padding-left: 30px; margin-top: 10px; margin-right: 0px; margin-bottom: 20px; margin-left: 0px;">
      <li style="text-align: left; font-family: Arial, sans-serif; color: #474747; margin-top: 5px; margin-right: 0px; margin-bottom: 5px; margin-left: 0px;">
        <span style="display: inline-block; background-color: #e4ff75; color: #000000;">Thread name</span>: When using Java.lang.Thread class to generate a thread, the thread will be named Thread-(Number), whereas when using java.util.concurrent.ThreadFactory class, it will be named pool-(number)-thread-(number).
      </li>
      <li style="text-align: left; font-family: Arial, sans-serif; color: #474747; margin-top: 5px; margin-right: 0px; margin-bottom: 5px; margin-left: 0px;">
        <span style="display: inline-block; background-color: #a6ff4d; color: #000000;">Priority</span>: Represents the priority of the threads.
      </li>
      <li style="text-align: left; font-family: Arial, sans-serif; color: #474747; margin-top: 5px; margin-right: 0px; margin-bottom: 5px; margin-left: 0px;">
        <span style="display: inline-block; background-color: #99dcff; color: #000000;">Thread ID</span>: Represents the unique ID for the threads. (Some useful information, including the CPU usage or memory usage of the thread, can be obtained by using thread ID.)
      </li>
      <li style="text-align: left; font-family: Arial, sans-serif; color: #474747; margin-top: 5px; margin-right: 0px; margin-bottom: 5px; margin-left: 0px;">
        <span style="display: inline-block; background-color: #9334d8; color: #ffffff;">Thread status</span>: Represents the status of the threads.
      </li>
      <li style="text-align: left; font-family: Arial, sans-serif; color: #474747; margin-top: 5px; margin-right: 0px; margin-bottom: 5px; margin-left: 0px;">
        <span style="display: inline-block; background-color: #8e8e8e; color: #000000;">Thread callstack</span>: Represents the call stack information of the threads.
      </li>
    </ul>
    
    <h2 style="width: 727px; margin-top: 21px; margin-right: 0px; margin-bottom: 21px; margin-left: 0px; font-size: 20px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      Thread Dump Patterns by Type
    </h2>
    
    <h3 style="width: 727px; margin-top: 15px; margin-right: 0px; margin-bottom: 15px; margin-left: 0px; font-size: 14px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      When Unable to Obtain a Lock (BLOCKED)
    </h3>
    
    <p>
      一个线程在Runnable,并且获得了一个锁，其他需要获取这个锁的都block住了
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      This is when the overall performance of the application slows down because a thread is occupying the lock and prevents other threads from obtaining it. In the following example, <span style="display: inline-block; background-color: #e4ff75; color: #000000;">BLOCKED_TEST pool-1-thread-1</span> thread is running with <<span style="display: inline-block; background-color: #e4ff75; color: #000000;"><span style="display: inline-block; background-color: #ffa700; color: #ffffff;">0x0000000780a000b0</span></span>> lock, while <span style="display: inline-block; background-color: #e4ff75; color: #000000;">BLOCKED_TEST pool-1-thread-2</span> and <span style="display: inline-block; background-color: #e4ff75; color: #000000;">BLOCKED_TEST pool-1-thread-3</span> threads are waiting to obtain <<span style="display: inline-block; background-color: #e4ff75; color: #000000;"><span style="display: inline-block; background-color: #ffa700; color: #ffffff;">0x0000000780a000b0</span></span>> lock.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px; text-align: center;">
      <img class="iePngFix" style="border-width: initial; cursor: pointer; border-style: none;" title="Figure 3: A thread blocking other threads" src="http://www.cubrid.org/files/attach/images/220547/971/295/when-unable-to-obtain-lock.png" alt="Figure 3: A thread blocking other threads" width="581" height="84" />
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px; text-align: center;">
      <strong>Figure 3: A thread blocking other threads.</strong>
    </p>
    
    <pre style="overflow-y: scroll; border-image: initial; border-width: 1px; border-color: #888888; border-style: solid; padding: 2px;">"BLOCKED_TEST pool-1-thread-1" prio=6 tid=0x0000000006904800 nid=0x28f4 runnable [0x000000000785f000]
   java.lang.Thread.State: <span style="display: inline-block; background-color: #a6ff4d;">RUNNABLE</span>
                at java.io.FileOutputStream.writeBytes(Native Method)
                at java.io.FileOutputStream.write(FileOutputStream.java:282)
                at java.io.BufferedOutputStream.flushBuffer(BufferedOutputStream.java:65)
                at java.io.BufferedOutputStream.flush(BufferedOutputStream.java:123)
                - locked &lt;0x0000000780a31778&gt; (a java.io.BufferedOutputStream)
                at java.io.PrintStream.write(PrintStream.java:432)
                - locked &lt;0x0000000780a04118&gt; (a java.io.PrintStream)
                at sun.nio.cs.StreamEncoder.writeBytes(StreamEncoder.java:202)
                at sun.nio.cs.StreamEncoder.implFlushBuffer(StreamEncoder.java:272)
                at sun.nio.cs.StreamEncoder.flushBuffer(StreamEncoder.java:85)
                - locked &lt;0x0000000780a040c0&gt; (a java.io.OutputStreamWriter)
                at java.io.OutputStreamWriter.flushBuffer(OutputStreamWriter.java:168)
                at java.io.PrintStream.newLine(PrintStream.java:496)
                - locked &lt;0x0000000780a04118&gt; (a java.io.PrintStream)
                at java.io.PrintStream.println(PrintStream.java:687)
                - locked &lt;0x0000000780a04118&gt; (a java.io.PrintStream)
                at com.nbp.theplatform.threaddump.ThreadBlockedState.monitorLock(ThreadBlockedState.java:44)
                - locked &lt;<span style="display: inline-block; background-color: #e4ff75;"><span style="display: inline-block; background-color: #ffa700; color: #ffffff;">0x0000000780a000b0</span></span>&gt; (a com.nbp.theplatform.threaddump.ThreadBlockedState)
                at com.nbp.theplatform.threaddump.ThreadBlockedState$1.run(ThreadBlockedState.java:7)
                at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:886)
                at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:908)
                at java.lang.Thread.run(Thread.java:662)

   Locked ownable synchronizers:
                - &lt;0x0000000780a31758&gt; (a java.util.concurrent.locks.ReentrantLock$NonfairSync)

"BLOCKED_TEST pool-1-thread-2" prio=6 tid=0x0000000007673800 nid=0x260c waiting for monitor entry [0x0000000008abf000]
   java.lang.Thread.State: <span style="display: inline-block; background-color: #ff0000; color: #ffffff;">BLOCKED</span> (on object monitor)
                at com.nbp.theplatform.threaddump.ThreadBlockedState.monitorLock(ThreadBlockedState.java:43)
                - waiting to lock &lt;<span style="display: inline-block; background-color: #e4ff75;"><span style="display: inline-block; background-color: #ffa700; color: #ffffff;">0x0000000780a000b0</span></span>&gt; (a com.nbp.theplatform.threaddump.ThreadBlockedState)
                at com.nbp.theplatform.threaddump.ThreadBlockedState$2.run(ThreadBlockedState.java:26)
                at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:886)
                at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:908)
                at java.lang.Thread.run(Thread.java:662)

   Locked ownable synchronizers:
                - &lt;0x0000000780b0c6a0&gt; (a java.util.concurrent.locks.ReentrantLock$NonfairSync)

"BLOCKED_TEST pool-1-thread-3" prio=6 tid=0x00000000074f5800 nid=0x1994 waiting for monitor entry [0x0000000008bbf000]
   java.lang.Thread.State: <span style="display: inline-block; background-color: #ff0000; color: #ffffff;">BLOCKED</span> (on object monitor)
                at com.nbp.theplatform.threaddump.ThreadBlockedState.monitorLock(ThreadBlockedState.java:42)
                - waiting to lock &lt;<span style="display: inline-block; background-color: #e4ff75;"><span style="display: inline-block; background-color: #ffa700; color: #ffffff;">0x0000000780a000b0</span></span>&gt; (a com.nbp.theplatform.threaddump.ThreadBlockedState)
                at com.nbp.theplatform.threaddump.ThreadBlockedState$3.run(ThreadBlockedState.java:34)
                at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:886
                at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:908)
                at java.lang.Thread.run(Thread.java:662)

   Locked ownable synchronizers:
                - &lt;0x0000000780b0e1b8&gt; (a java.util.concurrent.locks.ReentrantLock$NonfairSync)</pre>
    
    <h3 style="width: 727px; margin-top: 15px; margin-right: 0px; margin-bottom: 15px; margin-left: 0px; font-size: 14px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      When in Deadlock Status
    </h3>
    
    <p>
      经典死锁，A等B,B等C，C等A
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      This is when <em>thread A</em> needs to obtain <em>thread B</em>&#8216;s lock to continue its task, while <em>thread B</em> needs to obtain <em>thread A</em>&#8216;s lock to continue its task. In the thread dump, you can see that <span style="display: inline-block; background-color: #e4ff75; color: #000000;">DEADLOCK_TEST-1 thread</span> has <span style="display: inline-block; background-color: #ffa700; color: #ffffff;">0x00000007d58f5e48</span> lock, and is trying to obtain <span style="display: inline-block; background-color: #ffa700; color: #ffffff;"><span style="display: inline-block; background-color: #99dcff; color: #000000;">0x00000007d58f5e60</span></span> lock. You can also see that <span style="display: inline-block; background-color: #e4ff75; color: #000000;">DEADLOCK_TEST-2 thread</span> has <span style="display: inline-block; background-color: #ffa700; color: #ffffff;"><span style="display: inline-block; background-color: #99dcff; color: #000000;">0x00000007d58f5e60</span></span> lock, and is trying to obtain <span style="display: inline-block; background-color: #ffa700; color: #ffffff;"><span style="display: inline-block; background-color: #a6ff4d; color: #000000;">0x00000007d58f5e78</span></span> lock. Also, <span style="display: inline-block; background-color: #e4ff75; color: #000000;">DEADLOCK_TEST-3 thread</span> has <span style="display: inline-block; background-color: #ffa700; color: #ffffff;"><span style="display: inline-block; background-color: #a6ff4d; color: #000000;">0x00000007d58f5e78</span></span> lock, and is trying to obtain <span style="display: inline-block; background-color: #ffa700; color: #ffffff;">0x00000007d58f5e48</span> lock. As you can see, each thread is waiting to obtain another thread&#8217;s lock, and this status will not change until one thread discards its lock.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px; text-align: center;">
      <img class="iePngFix" style="border-width: initial; cursor: pointer; border-style: none;" title="Figure 4: Threads in a Deadlock status" src="http://www.cubrid.org/files/attach/images/220547/971/295/when-in-deadlock-status.png" alt="Figure 4: Threads in a Deadlock status" width="580" height="91" />
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px; text-align: center;">
      <strong>Figure 4: Threads in a Deadlock status.</strong>
    </p>
    
    <pre style="overflow-y: scroll; border-image: initial; border-width: 1px; border-color: #888888; border-style: solid; padding: 2px;">"DEADLOCK_TEST-1" daemon prio=6 tid=0x000000000690f800 nid=0x1820 waiting for monitor entry [0x000000000805f000]
   java.lang.Thread.State: <span style="display: inline-block; background-color: #ff0000; color: #ffffff;">BLOCKED</span> (on object monitor)
                at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.goMonitorDeadlock(ThreadDeadLockState.java:197)
                - waiting to lock &lt;<span style="display: inline-block; background-color: #99dcff;">0x00000007d58f5e60</span>&gt; (a com.nbp.theplatform.threaddump.ThreadDeadLockState$Monitor)
                at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.monitorOurLock(ThreadDeadLockState.java:182)
                - locked &lt;<span style="display: inline-block; background-color: #ffa700; color: #ffffff;">0x00000007d58f5e48</span>&gt; (a com.nbp.theplatform.threaddump.ThreadDeadLockState$Monitor)
                at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.run(ThreadDeadLockState.java:135)

   Locked ownable synchronizers:
                - None

"DEADLOCK_TEST-2" daemon prio=6 tid=0x0000000006858800 nid=0x17b8 waiting for monitor entry [0x000000000815f000]
   java.lang.Thread.State: <span style="display: inline-block; background-color: #ff0000; color: #ffffff;">BLOCKED</span> (on object monitor)
                at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.goMonitorDeadlock(ThreadDeadLockState.java:197)
                - waiting to lock &lt;<span style="display: inline-block; background-color: #a6ff4d;">0x00000007d58f5e78</span>&gt; (a com.nbp.theplatform.threaddump.ThreadDeadLockState$Monitor)
                at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.monitorOurLock(ThreadDeadLockState.java:182)
                - locked &lt;<span style="display: inline-block; background-color: #99dcff;">0x00000007d58f5e60</span>&gt; (a com.nbp.theplatform.threaddump.ThreadDeadLockState$Monitor)
                at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.run(ThreadDeadLockState.java:135)

   Locked ownable synchronizers:
                - None

"DEADLOCK_TEST-3" daemon prio=6 tid=0x0000000006859000 nid=0x25dc waiting for monitor entry [0x000000000825f000]
   java.lang.Thread.State: <span style="display: inline-block; background-color: #ff0000; color: #ffffff;">BLOCKED</span> (on object monitor)
                at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.goMonitorDeadlock(ThreadDeadLockState.java:197)
                - waiting to lock &lt;<span style="display: inline-block; background-color: #ffa700; color: #ffffff;">0x00000007d58f5e48</span>&gt; (a com.nbp.theplatform.threaddump.ThreadDeadLockState$Monitor)
                at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.monitorOurLock(ThreadDeadLockState.java:182)
                - locked &lt;<span style="display: inline-block; background-color: #a6ff4d;">0x00000007d58f5e78</span>&gt; (a com.nbp.theplatform.threaddump.ThreadDeadLockState$Monitor)
                at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.run(ThreadDeadLockState.java:135)

   Locked ownable synchronizers:
                - None</pre>
    
    <h3 style="width: 727px; margin-top: 15px; margin-right: 0px; margin-bottom: 15px; margin-left: 0px; font-size: 14px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      When Continuously Waiting to Receive Messages from a Remote Server
    </h3>
    
    <p>
      Socket等待读数据，显示的是Runnable，但其实是出于一个网络IO等待过程中。
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      The thread appears to be normal, since its state keeps showing as <span style="display: inline-block; background-color: #a6ff4d; color: #000000;"><span style="display: inline-block; background-color: #9334d8; color: #ffffff;">RUNNABLE</span></span>. However, when you align the thread dumps chronologically, you can see that <span style="display: inline-block; font-family: monospace;">socketReadThread thread</span> is waiting infinitely to read the socket.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px; text-align: center;">
      <img class="iePngFix" style="border-width: initial; cursor: pointer; border-style: none;" title="Figure 5: Continuous Waiting Status" src="http://www.cubrid.org/files/attach/images/220547/971/295/when-continuosly-waiting-to-receive-message-from-remote-server.png" alt="Figure 5: Continuous Waiting Status" width="578" height="33" />
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px; text-align: center;">
      <strong>Figure 5: Continuous Waiting Status.</strong>
    </p>
    
    <pre style="overflow-y: scroll; border-image: initial; border-width: 1px; border-color: #888888; border-style: solid; padding: 2px;">"socketReadThread" prio=6 tid=0x0000000006a0d800 nid=0x1b40 runnable [0x00000000089ef000]
   java.lang.Thread.State: <span style="display: inline-block; background-color: #9334d8; color: #ffffff;">RUNNABLE</span>
                at java.net.SocketInputStream.socketRead0(Native Method)
                at java.net.SocketInputStream.read(SocketInputStream.java:129)
                at sun.nio.cs.StreamDecoder.readBytes(StreamDecoder.java:264)
                at sun.nio.cs.StreamDecoder.implRead(StreamDecoder.java:306)
                at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:158)
                - locked &lt;0x00000007d78a2230&gt; (a java.io.InputStreamReader)
                at sun.nio.cs.StreamDecoder.read0(StreamDecoder.java:107)
                - locked &lt;0x00000007d78a2230&gt; (a java.io.InputStreamReader)
                at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:93)
                at java.io.InputStreamReader.read(InputStreamReader.java:151)
                at com.nbp.theplatform.threaddump.ThreadSocketReadState$1.run(ThreadSocketReadState.java:27)
                at java.lang.Thread.run(Thread.java:662)</pre>
    
    <h3 style="width: 727px; margin-top: 15px; margin-right: 0px; margin-bottom: 15px; margin-left: 0px; font-size: 14px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      When Waiting
    </h3>
    
    <p>
      在线上发现最多的，阻塞队列的取数据，都取不出来，线程状态全部是WAITING，确认方法，确认代码开的线程数，统计这个WAITING状态，如果线程数和WAINTING数目一致，说明队列都取不出来，队列死在这了，重启程序吧。。
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      The thread is maintaining <span style="display: inline-block; background-color: #e4ff75; color: #000000;">WAIT</span> status. In the thread dump, <span style="display: inline-block; font-family: monospace;">IoWaitThread thread</span> keeps waiting to receive a message from <span style="display: inline-block; font-family: monospace;">LinkedBlockingQueue</span>. If there continues to be no message for LinkedBlockingQueue, then the thread status will not change.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px; text-align: center;">
      <img class="iePngFix" style="border-width: initial; cursor: pointer; border-style: none;" title="Figure 6: Waiting status" src="http://www.cubrid.org/files/attach/images/220547/971/295/when-waiting.png" alt="Figure 6: Waiting status" width="580" height="35" />
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px; text-align: center;">
      <strong>Figure 6: Waiting status.</strong>
    </p>
    
    <pre style="overflow-y: scroll; border-image: initial; border-width: 1px; border-color: #888888; border-style: solid; padding: 2px;">"IoWaitThread" prio=6 tid=0x0000000007334800 nid=0x2b3c waiting on condition [0x000000000893f000]
   java.lang.Thread.State: <span style="display: inline-block; background-color: #e4ff75;">WAITING</span> (parking)
                at sun.misc.Unsafe.park(Native Method)
                - parking to wait for  &lt;0x00000007d5c45850&gt; (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
                at java.util.concurrent.locks.LockSupport.park(LockSupport.java:156)
                at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:1987)
                at java.util.concurrent.LinkedBlockingDeque.takeFirst(LinkedBlockingDeque.java:440)
                at java.util.concurrent.LinkedBlockingDeque.take(LinkedBlockingDeque.java:629)
                at com.nbp.theplatform.threaddump.ThreadIoWaitState$IoWaitHandler2.run(ThreadIoWaitState.java:89)
                at java.lang.Thread.run(Thread.java:662)</pre>
    
    <h3 style="width: 727px; margin-top: 15px; margin-right: 0px; margin-bottom: 15px; margin-left: 0px; font-size: 14px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      When Thread Resources Cannot be Organized Normally
    </h3>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      Unnecessary threads will pile up when thread resources cannot be organized normally. If this occurs, it is recommended to monitor the thread organization process or check the conditions for thread termination.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px; text-align: center;">
      <img class="iePngFix" style="border-width: initial; cursor: pointer; border-style: none;" title="Figure 7: Unorganized Threads" src="http://www.cubrid.org/files/attach/images/220547/971/295/when-thread-resources-cannot-be-organized-normally.png" alt="Figure 7: Unorganized Threads" width="678" height="193" />
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px; text-align: center;">
      <strong>Figure 7: Unorganized Threads.</strong>
    </p>
    
    <h2 style="width: 727px; margin-top: 21px; margin-right: 0px; margin-bottom: 21px; margin-left: 0px; font-size: 20px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      How to Solve Problems by Using Thread Dump
    </h2>
    
    <h3 style="width: 727px; margin-top: 15px; margin-right: 0px; margin-bottom: 15px; margin-left: 0px; font-size: 14px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      Example 1: When the CPU Usage is Abnormally High
    </h3>
    
    <ol style="padding-left: 30px;">
      <li style="text-align: left; font-family: Arial, sans-serif; color: #474747; margin-top: 5px; margin-right: 0px; margin-bottom: 5px; margin-left: 0px;">
        <span style="color: #474747; font-family: Arial, sans-serif;"><span style="line-height: normal;">Extract the thread that has the highest CPU usage.</span></span> <div>
          <div id="highlighter_213245" class="syntaxhighlighter  bash " style="width: 697px; margin-top: 1em !important; margin-right: 0px !important; margin-bottom: 1em !important; margin-left: 0px !important; position: relative !important; overflow-x: auto !important; overflow-y: auto !important; font-size: 1em !important; background-color: white !important;">
            <div class="toolbar" style="border-top-left-radius: 0px !important; border-top-right-radius: 0px !important; border-bottom-right-radius: 0px !important; border-bottom-left-radius: 0px !important; background-image: initial !important; background-attachment: initial !important; background-origin: initial !important; background-clip: initial !important; background-color: transparent !important; border-style: none !important; border-color: initial !important; border-image: initial !important; bottom: auto !important; float: none !important; height: 11px !important; left: auto !important; line-height: 1.1em !important; outline-width: 0px !important; outline-style: initial !important; outline-color: initial !important; overflow-x: visible !important; overflow-y: visible !important; position: absolute !important; right: 1px !important; top: 1px !important; vertical-align: baseline !important; width: 11px !important; box-sizing: content-box !important; font-family: Consolas, 'Bitstream Vera Sans Mono', 'Courier New', Courier, monospace !important; font-size: 10px !important; min-height: inherit !important; z-index: 10 !important; color: white !important; border-width: 0px !important; padding: 0px !important; margin: 0px !important;">
              <span style="display: inline-block;"><a class="toolbar_item command_help help" style="color: white !important; text-decoration: none !important; border-top-left-radius: 0px !important; border-top-right-radius: 0px !important; border-bottom-right-radius: 0px !important; border-bottom-left-radius: 0px !important; background-image: none !important; background-attachment: initial !important; background-origin: initial !important; background-clip: initial !important; background-color: initial !important; border-style: initial !important; border-color: initial !important; border-image: initial !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.1em !important; outline-width: 0px !important; outline-style: initial !important; outline-color: initial !important; overflow-x: visible !important; overflow-y: visible !important; padding-top: 1px !important; padding-right: 0px !important; padding-bottom: 0px !important; padding-left: 0px !important; position: static !important; right: auto !important; text-align: center !important; top: auto !important; vertical-align: baseline !important; width: auto !important; box-sizing: content-box !important; font-size: 1em !important; min-height: inherit !important; display: block !important; border-width: 0px !important; margin: 0px !important;" href="http://www.cubrid.org/blog/dev-platform/how-to-analyze-java-thread-dumps/#">?</a></span>
            </div>
            
            <div>
              <code>[user@linux ~]$ </code><code>ps</code> <code>-mo pid.lwp.stime.</code><code>time</code><code>.cpu -C java</code>
            </div>
            
            <div>
            </div>
            
            <div>
              <code>     </code><code>PID         LWP          STIME                  TIME        %CPU</code>
            </div>
            
            <div>
              <code>10029               -         Dec07          00:02:02           99.5</code>
            </div>
            
            <div>
              <code>         </code><code>-       10039        Dec07          00:00:00              0.1</code>
            </div>
            
            <div>
              <code>         </code><code>-       10040        Dec07          00:00:00           95.5</code>
            </div>
            
            <p>
              <span style="font-family: Consolas, 'Bitstream Vera Sans Mono', 'Courier New', Courier, monospace;"><span style="line-height: 14px;"><br /> </span></span>
            </p>
          </div>
        </div>
        
        <p style="color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
          From the application, find out which thread is using the CPU the most.
        </p>
        
        <p style="color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
          Acquire the <em>Light Weight Process</em> (LWP) that uses the CPU the most and convert its unique number (10039) into a hexadecimal number (0&#215;2737).
        </p>
      </li>
      
      <li style="text-align: left; font-family: Arial, sans-serif; color: #474747; margin-top: 5px; margin-right: 0px; margin-bottom: 5px; margin-left: 0px;">
        <p style="color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
          After acquiring the thread dump, check the thread&#8217;s action.
        </p>
        
        <p style="color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
          Extract the thread dump of an application with a PID of 10029, then find the thread with an <span style="display: inline-block; font-family: monospace;">nid of 0&#215;2737</span>.
        </p>
        
        <p style="color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
          ＣＰＵ太高，通过lwp找到最高的CPU对应的PID，或者通过top找到进程号，然后top -Hp 进程号，找到最高的PID,转为二进制，在dump出来的东西里搜索到tid相等的状态，确认在什么函数里一直run不停，
        </p>
        
        <p style="color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
          这个应该多Dump几次，确认是否都在一个地方run
        </p>
        
        <pre style="overflow-y: scroll; border-image: initial; border-width: 1px; border-color: #888888; border-style: solid; padding: 2px;">"NioProcessor-2" prio=10 tid=0x0a8d2800 nid=0x2737 runnable [0x49aa5000]
java.lang.Thread.State: RUNNABLE
                at sun.nio.ch.EPollArrayWrapper.epollWait(Native Method)
                at sun.nio.ch.EPollArrayWrapper.poll(EPollArrayWrapper.java:210)
                at sun.nio.ch.EPollSelectorImpl.doSelect(EPollSelectorImpl.java:65)
                at sun.nio.ch.SelectorImpl.lockAndDoSelect(SelectorImpl.java:69)
                - locked &lt;0x74c52678&gt; (a sun.nio.ch.Util$1)
                - locked &lt;0x74c52668&gt; (a java.util.Collections$UnmodifiableSet)
                - locked &lt;0x74c501b0&gt; (a sun.nio.ch.EPollSelectorImpl)
                at sun.nio.ch.SelectorImpl.select(SelectorImpl.java:80)
                at external.org.apache.mina.transport.socket.nio.NioProcessor.select(NioProcessor.java:65)
                at external.org.apache.mina.common.AbstractPollingIoProcessor$Worker.run(AbstractPollingIoProcessor.java:708)
                at external.org.apache.mina.util.NamePreservingRunnable.run(NamePreservingRunnable.java:51)
                at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:886)
                at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:908)
                at java.lang.Thread.run(Thread.java:662)</pre>
        
        <p style="color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
          Extract thread dumps several times every hour, and check the status change of the threads to determine the problem.
        </p>
      </li>
    </ol>
    
    <h3 style="width: 727px; margin-top: 15px; margin-right: 0px; margin-bottom: 15px; margin-left: 0px; font-size: 14px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      Example 2: When the Processing Performance is Abnormally Slow
    </h3>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      After acquiring thread dumps several times, find the list of threads with <span style="display: inline-block; background-color: #ff0000; color: #ffffff;">BLOCKED</span> status.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      性能太差，处理速度变慢，很多线程都在等待一个锁，只有一个线程得到了然后它在得到锁之后的处理过程很慢，拖慢了所有程序，
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      通常有两种情况引发：
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      1，大家都在抢资源，但是资源不够，所以很多线程抢不到，都block住，等待。（不确定是不是这么翻译，理解的大意是这样）
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      2，在一个锁里面的处理函数有一个超时操作，超时设置的太长，一直在等待超时，又不释放锁，导致其他线程都在等待。
    </p>
    
    <pre style="overflow-y: scroll; border-image: initial; border-width: 1px; border-color: #888888; border-style: solid; padding: 2px;">" DB-Processor-13" daemon prio=5 tid=0x003edf98 nid=0xca waiting for monitor entry [0x000000000825f000]
java.lang.Thread.State: <span style="display: inline-block; background-color: #ff0000; color: #ffffff;">BLOCKED</span> (on object monitor)
                at beans.ConnectionPool.getConnection(ConnectionPool.java:102)
                - waiting to lock &lt;<span style="display: inline-block; background-color: #e4ff75;">0xe0375410</span>&gt; (a beans.ConnectionPool)
                at beans.cus.ServiceCnt.getTodayCount(ServiceCnt.java:111)
                at beans.cus.ServiceCnt.insertCount(ServiceCnt.java:43)

"DB-Processor-14" daemon prio=5 tid=0x003edf98 nid=0xca waiting for monitor entry [0x000000000825f020]
java.lang.Thread.State: <span style="display: inline-block; background-color: #ff0000; color: #ffffff;">BLOCKED</span> (on object monitor)
                at beans.ConnectionPool.getConnection(ConnectionPool.java:102)
                - waiting to lock &lt;<span style="display: inline-block; background-color: #e4ff75;">0xe0375410</span>&gt; (a beans.ConnectionPool)
                at beans.cus.ServiceCnt.getTodayCount(ServiceCnt.java:111)
                at beans.cus.ServiceCnt.insertCount(ServiceCnt.java:43)

" DB-Processor-3" daemon prio=5 tid=0x00928248 nid=0x8b waiting for monitor entry [0x000000000825d080]
java.lang.Thread.State: <span style="display: inline-block; background-color: #9334d8; color: #ffffff;">RUNNABLE</span>
                at oracle.jdbc.driver.OracleConnection.isClosed(OracleConnection.java:570)
                - waiting to lock &lt;0xe03ba2e0&gt; (a oracle.jdbc.driver.OracleConnection)
                at beans.ConnectionPool.getConnection(ConnectionPool.java:112)
                - locked &lt;0xe0386580&gt; (a java.util.Vector)
                - locked &lt;0xe0375410&gt; (a beans.ConnectionPool)
                at beans.cus.Cue_1700c.GetNationList(Cue_1700c.java:66)
                at org.apache.jsp.cue_1700c_jsp._jspService(cue_1700c_jsp.java:120)</pre>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      Acquire the list of threads with <span style="display: inline-block; background-color: #ff0000; color: #ffffff;">BLOCKED</span> status after getting the thread dumps several times.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      If the threads are BLOCKED, extract the threads related to the lock that the threads are trying to obtain.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      Through the thread dump, you can confirm that the thread status stays <span style="display: inline-block; background-color: #ff0000; color: #ffffff;">BLOCKED</span> because <<span style="display: inline-block; background-color: #e4ff75; color: #000000;">0xe0375410</span>> lock could not be obtained. This problem can be solved by analyzing stack trace from the thread currently holding the lock.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      There are two reasons why the above pattern frequently appears in applications using DBMS. The first reason is <strong>inadequate configurations</strong>. Despite the fact that the threads are still working, they cannot show their best performance because the configurations for DBCP and the like are not adequate. If you extract thread dumps multiple times and compare them, you will often see that some of the threads that were BLOCKED previously are in a different state.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      The second reason is the <strong>abnormal connection</strong>. When the connection with DBMS stays abnormal, the threads wait until the time is out. In this case, even after extracting the thread dumps several times and comparing them, you will see that the threads related to DBMS are still in a BLOCKED state. By adequately changing the values, such as the timeout value, you can shorten the time in which the problem occurs.
    </p>
    
    <h2 style="width: 727px; margin-top: 21px; margin-right: 0px; margin-bottom: 21px; margin-left: 0px; font-size: 20px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      Coding for Easy Thread Dump
    </h2>
    
    <h3 style="width: 727px; margin-top: 15px; margin-right: 0px; margin-bottom: 15px; margin-left: 0px; font-size: 14px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
      Naming Threads
    </h3>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      When a thread is created using <span style="display: inline-block; font-family: monospace;">java.lang.Thread</span> object, the thread will be named <span style="display: inline-block; font-family: monospace;">Thread-(Number)</span>. When a thread is created using <span style="display: inline-block; font-family: monospace;">java.util.concurrent.DefaultThreadFactory</span> object, the thread will be named <span style="display: inline-block; font-family: monospace;">pool-(Number)-thread-(Number)</span>. When analyzing tens to thousands of threads for an application, if all the threads still have their default names, analyzing them becomes very difficult, because it is difficult to distinguish the threads to be analyzed.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      Therefore, you are recommended to develop the habit of naming the threads whenever a new thread is created.
    </p>
    
    <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
      When you create a thread using <span style="display: inline-block; font-family: monospace;">java.lang.Thread</span>, you can give the thread a custom name by using the creator parameter.
    </p>
    
    <div>
      <div id="highlighter_164522" class="syntaxhighlighter  java" style="width: 727px; margin-top: 1em !important; margin-right: 0px !important; margin-bottom: 1em !important; margin-left: 0px !important; position: relative !important; overflow-x: auto !important; overflow-y: auto !important; font-size: 1em !important; background-color: white !important;">
        <div class="toolbar" style="border-top-left-radius: 0px !important; border-top-right-radius: 0px !important; border-bottom-right-radius: 0px !important; border-bottom-left-radius: 0px !important; background-image: initial !important; background-attachment: initial !important; background-origin: initial !important; background-clip: initial !important; background-color: transparent !important; border-style: none !important; border-color: initial !important; border-image: initial !important; bottom: auto !important; float: none !important; height: 11px !important; left: auto !important; line-height: 1.1em !important; outline-width: 0px !important; outline-style: initial !important; outline-color: initial !important; overflow-x: visible !important; overflow-y: visible !important; position: absolute !important; right: 1px !important; text-align: left !important; top: 1px !important; vertical-align: baseline !important; width: 11px !important; box-sizing: content-box !important; font-family: Consolas, 'Bitstream Vera Sans Mono', 'Courier New', Courier, monospace !important; font-size: 10px !important; min-height: inherit !important; z-index: 10 !important; color: white !important; border-width: 0px !important; padding: 0px !important; margin: 0px !important;">
          <span style="display: inline-block;"><a class="toolbar_item command_help help" style="color: white !important; text-decoration: none !important; border-top-left-radius: 0px !important; border-top-right-radius: 0px !important; border-bottom-right-radius: 0px !important; border-bottom-left-radius: 0px !important; background-image: none !important; background-attachment: initial !important; background-origin: initial !important; background-clip: initial !important; background-color: initial !important; border-style: initial !important; border-color: initial !important; border-image: initial !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.1em !important; outline-width: 0px !important; outline-style: initial !important; outline-color: initial !important; overflow-x: visible !important; overflow-y: visible !important; padding-top: 1px !important; padding-right: 0px !important; padding-bottom: 0px !important; padding-left: 0px !important; position: static !important; right: auto !important; text-align: center !important; top: auto !important; vertical-align: baseline !important; width: auto !important; box-sizing: content-box !important; font-size: 1em !important; min-height: inherit !important; display: block !important; border-width: 0px !important; margin: 0px !important;" href="http://www.cubrid.org/blog/dev-platform/how-to-analyze-java-thread-dumps/#">?</a></span>
        </div>
        
        <div style="text-align: left;">
          <pre>public Thread(Runnable target, String name);
public Thread(ThreadGroup group, String name);
public Thread(ThreadGroup group, Runnable target, String name);
public Thread(ThreadGroup group, Runnable target, String name, long stackSize);</pre>
          
          <p>
            <code>stackSize);</code>
          </p>
        </div>
        
        <p>
          <span style="font-family: Consolas, 'Bitstream Vera Sans Mono', 'Courier New', Courier, monospace;"><span style="line-height: 14px;"><br /> </span></span>
        </p>
      </div>
    </div>
  </div>
  
  <p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
    When you create a thread using <span style="display: inline-block; font-family: monospace;">java.util.concurrent.ThreadFactory</span>, you can name it by generating your own <span style="display: inline-block; font-family: monospace;">ThreadFactory</span>. If you do not need special functionalities, then you can use <span style="display: inline-block; font-family: monospace;">MyThreadFactory</span> as described below:
  </p>
  
  <pre>import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.atomic.AtomicInteger;

public class MyThreadFactory implements ThreadFactory {
  private static final ConcurrentHashMap POOL_NUMBER =
                                                       new ConcurrentHashMap();
  private final ThreadGroup group;
  private final AtomicInteger threadNumber = new AtomicInteger(1);
  private final String namePrefix;

  public MyThreadFactory(String threadPoolName) {

      if (threadPoolName == null) {
          throw new NullPointerException("threadPoolName");
      }
            POOL_NUMBER.putIfAbsent(threadPoolName, new AtomicInteger());

      SecurityManager securityManager = System.getSecurityManager();
      group = (securityManager != null) ? securityManager.getThreadGroup() :
                                                    Thread.currentThread().getThreadGroup();

      AtomicInteger poolCount = POOL_NUMBER.get(threadPoolName);

      if (poolCount == null) {
            namePrefix = threadPoolName + " pool-00-thread-";
      } else {
            namePrefix = threadPoolName + " pool-" + poolCount.getAndIncrement() + "-thread-";
      }
  }

  public Thread newThread(Runnable runnable) {
      Thread thread = new Thread(group, runnable, namePrefix + threadNumber.getAndIncrement(), 0);

      if (thread.isDaemon()) {
            thread.setDaemon(false);
      }

      if (thread.getPriority() != Thread.NORM_PRIORITY) {
            thread.setPriority(Thread.NORM_PRIORITY);
      }

      return thread;
  }
}</pre>
  
  <div id="highlighter_198842" class="syntaxhighlighter  java " style="width: 727px; margin-top: 1em !important; margin-right: 0px !important; margin-bottom: 1em !important; margin-left: 0px !important; position: relative !important; overflow-x: auto !important; overflow-y: auto !important; font-size: 1em !important; background-color: white !important;">
    <div class="toolbar" style="border-top-left-radius: 0px !important; border-top-right-radius: 0px !important; border-bottom-right-radius: 0px !important; border-bottom-left-radius: 0px !important; background-image: initial !important; background-attachment: initial !important; background-origin: initial !important; background-clip: initial !important; background-color: transparent !important; border-style: none !important; border-color: initial !important; border-image: initial !important; bottom: auto !important; float: none !important; height: 11px !important; left: auto !important; line-height: 1.1em !important; outline-width: 0px !important; outline-style: initial !important; outline-color: initial !important; overflow-x: visible !important; overflow-y: visible !important; position: absolute !important; right: 1px !important; text-align: left !important; top: 1px !important; vertical-align: baseline !important; width: 11px !important; box-sizing: content-box !important; font-family: Consolas, 'Bitstream Vera Sans Mono', 'Courier New', Courier, monospace !important; font-size: 10px !important; min-height: inherit !important; z-index: 10 !important; color: white !important; border-width: 0px !important; padding: 0px !important; margin: 0px !important;">
      <span style="display: inline-block;"><a class="toolbar_item command_help help" style="color: white !important; text-decoration: none !important; border-top-left-radius: 0px !important; border-top-right-radius: 0px !important; border-bottom-right-radius: 0px !important; border-bottom-left-radius: 0px !important; background-image: none !important; background-attachment: initial !important; background-origin: initial !important; background-clip: initial !important; background-color: initial !important; border-style: initial !important; border-color: initial !important; border-image: initial !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.1em !important; outline-width: 0px !important; outline-style: initial !important; outline-color: initial !important; overflow-x: visible !important; overflow-y: visible !important; padding-top: 1px !important; padding-right: 0px !important; padding-bottom: 0px !important; padding-left: 0px !important; position: static !important; right: auto !important; text-align: center !important; top: auto !important; vertical-align: baseline !important; width: auto !important; box-sizing: content-box !important; font-size: 1em !important; min-height: inherit !important; display: block !important; border-width: 0px !important; margin: 0px !important;" href="http://www.cubrid.org/blog/dev-platform/how-to-analyze-java-thread-dumps/#">?</a></span>
    </div>
    
    <div style="text-align: left;">
      <span style="font-family: Consolas, 'Bitstream Vera Sans Mono', 'Courier New', Courier, monospace;"><span style="line-height: 14px;"><br /> </span></span>
    </div>
  </div>
</div>

<h3 style="width: 727px; margin-top: 15px; margin-right: 0px; margin-bottom: 15px; margin-left: 0px; font-size: 14px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
  Obtaining More Detailed Information by Using MBean
</h3>

<p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
  You can obtain <span style="display: inline-block; font-family: monospace;">ThreadInfo</span> objects using <span style="display: inline-block; font-family: monospace;">MBean</span>. You can also obtain more information that would be difficult to acquire via thread dumps, by using ThreadInfo.
</p>

<pre>ThreadMXBean mxBean = ManagementFactory.getThreadMXBean();
long[] threadIds = mxBean.getAllThreadIds();
ThreadInfo[] threadInfos = mxBean.getThreadInfo(threadIds);
for (ThreadInfo threadInfo : threadInfos) {
  System.out.println(
      threadInfo.getThreadName());
  System.out.println(
      threadInfo.getBlockedCount());
  System.out.println(
      threadInfo.getBlockedTime());
  System.out.println(
      threadInfo.getWaitedCount());
  System.out.println(
      threadInfo.getWaitedTime());
}</pre>

<div style="font-family: Georgia, 'Times New Roman', Times, serif; line-height: normal;">
  <div id="highlighter_981305" class="syntaxhighlighter  java " style="width: 727px; margin-top: 1em !important; margin-right: 0px !important; margin-bottom: 1em !important; margin-left: 0px !important; position: relative !important; overflow-x: auto !important; overflow-y: auto !important; font-size: 1em !important; background-color: white !important;">
    <div class="toolbar" style="border-top-left-radius: 0px !important; border-top-right-radius: 0px !important; border-bottom-right-radius: 0px !important; border-bottom-left-radius: 0px !important; background-image: initial !important; background-attachment: initial !important; background-origin: initial !important; background-clip: initial !important; background-color: transparent !important; border-style: none !important; border-color: initial !important; border-image: initial !important; bottom: auto !important; float: none !important; height: 11px !important; left: auto !important; line-height: 1.1em !important; outline-width: 0px !important; outline-style: initial !important; outline-color: initial !important; overflow-x: visible !important; overflow-y: visible !important; position: absolute !important; right: 1px !important; text-align: left !important; top: 1px !important; vertical-align: baseline !important; width: 11px !important; box-sizing: content-box !important; font-family: Consolas, 'Bitstream Vera Sans Mono', 'Courier New', Courier, monospace !important; font-size: 10px !important; min-height: inherit !important; z-index: 10 !important; color: white !important; border-width: 0px !important; padding: 0px !important; margin: 0px !important;">
      <span style="display: inline-block;"><a class="toolbar_item command_help help" style="color: white !important; text-decoration: none !important; border-top-left-radius: 0px !important; border-top-right-radius: 0px !important; border-bottom-right-radius: 0px !important; border-bottom-left-radius: 0px !important; background-image: none !important; background-attachment: initial !important; background-origin: initial !important; background-clip: initial !important; background-color: initial !important; border-style: initial !important; border-color: initial !important; border-image: initial !important; bottom: auto !important; float: none !important; height: auto !important; left: auto !important; line-height: 1.1em !important; outline-width: 0px !important; outline-style: initial !important; outline-color: initial !important; overflow-x: visible !important; overflow-y: visible !important; padding-top: 1px !important; padding-right: 0px !important; padding-bottom: 0px !important; padding-left: 0px !important; position: static !important; right: auto !important; text-align: center !important; top: auto !important; vertical-align: baseline !important; width: auto !important; box-sizing: content-box !important; font-size: 1em !important; min-height: inherit !important; display: block !important; border-width: 0px !important; margin: 0px !important;" href="http://www.cubrid.org/blog/dev-platform/how-to-analyze-java-thread-dumps/#">?</a></span>
    </div>
  </div>
</div>

<p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
  You can acquire the amount of time that the threads <span style="display: inline-block; background-color: #e4ff75; color: #000000;">WAITed</span> or were <span style="display: inline-block; background-color: #ff0000; color: #ffffff;">BLOCKED</span> by using the method in ThreadInfo, and by using this you can also obtain the list of threads that have been inactive for an abnormally long period of time.
</p>

<h2 style="width: 727px; margin-top: 21px; margin-right: 0px; margin-bottom: 21px; margin-left: 0px; font-size: 20px; font-family: 'Myriad Pro', 'lucida grande', 'lucida sans unicode', arial, verdana, sans-serif;">
  In Conclusion
</h2>

<p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
  In this article I was concerned that for developers with a lot of experience in multi-thread programming, this material may be common knowledge, whereas for less experienced developers, I felt that I was skipping straight to thread dumps, without providing enough background information about the thread activities. This was because of my lack of knowledge, as I was not able to explain the thread activities in a clear yet concise manner. I sincerely hope that this article will prove helpful for many developers.
</p>

<p style="font-family: Arial, sans-serif; color: #555555; line-height: 25px; margin-top: 7px; margin-right: 0px; margin-bottom: 7px; margin-left: 0px;">
  By Tae Jin Gu, Software Engineer at Web Platform Development Lab, NHN Corporation.
</p>

[How to Analyze Java Thread Dumps | CUBRID Blog][1].

 [1]: http://www.cubrid.org/blog/dev-platform/how-to-analyze-java-thread-dumps/