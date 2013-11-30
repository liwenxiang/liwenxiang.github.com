---
title: 在eclipse中使用maven，构建项目
author: codeboy
layout: post
permalink: /2012/09/09/%e5%9c%a8eclipse%e4%b8%ad%e4%bd%bf%e7%94%a8maven%ef%bc%8c%e6%9e%84%e5%bb%ba%e9%a1%b9%e7%9b%ae/
categories:
  - 安装配置
tags:
  - eclipse
  - mavne
---
一直对公司的maven怎么搞的不清楚，放假两天把maven\_define\_guide_中文版看了下，对一些基本概念有了些了解，刚刚好解决周日碰到的一个maven过不了的问题。现在记录下怎么在eclipse中使用maven的过程。  
<!--more-->

  
1，基本依赖，大家都懂得，jdk，eclipse

2，安装maven的eclipse插件，m2e  地址[http://m2eclipse.sonatype.org/sites/m2e/][1]，安装之后需要在Windows->Java->Installed JREs将默认的jre改为jdk的jre，具体就是把原来的jre删掉，重新添加一个，添加的时候JRE home选C:\Program Files\Java\jdk1.7.0_03

3，默认的maven去访问外网，速度实在是太慢。。。，安装一个artifactory，作为中间代理，下载，另一个是可以把自己的东西传上去，

3.1，下载artifactory安装包，解压，到bin目录管理员权限执行InstallService.bat，然后再windowns管理中，把这个服务启动。

3.2，artifactory默认的库设置已经够用了，没仔细研究，分了N各组，然后这些组再聚成几个虚拟的组，我们直接使用虚拟组最方便，用到的几个虚拟组是libs-release plugins-release libs-snapshot plugins-snapshot ，然后再C:\Users\thinkpad\.m2\  下新建settings.xml文件

> 内容如下
> 
> <settings>  
> <profiles>  
> <profile>  
> <id>artifactory</id>  
> <repositories>  
> <repository>  
> <id>central</id>  
> <url><http://localhost:8081/artifactory/libs-release/></url>  
> <snapshots>  
> <enabled>false</enabled>  
> </snapshots>  
> </repository>  
> <repository>  
> <id>snapshots</id>  
> <url><http://localhost:8081/artifactory/libs-snapshot/></url>  
> <releases>  
> <enabled>false</enabled>  
> </releases>  
> </repository>  
> </repositories>  
> <pluginRepositories>  
> <pluginRepository>  
> <id>central</id>  
> <url><http://localhost:8081/artifactory/plugins-release/></url>  
> <snapshots>  
> <enabled>false</enabled>  
> </snapshots>  
> </pluginRepository>  
> <pluginRepository>  
> <id>snapshots</id>  
> <url><http://localhost:8081/artifactory/plugins-snapshot/></url>  
> <releases>  
> <enabled>false</enabled>  
> </releases>  
> </pluginRepository>  
> </pluginRepositories>  
> </profile>  
> </profiles>  
> <activeProfiles>  
> <activeProfile>artifactory</activeProfile>  
> </activeProfiles>
> 
> <servers>  
> <server>  
> <id>releases</id>  
> <username>admin</username>  
> <password>password</password>  
> </server>  
> <server>  
> <id>snapshots</id>  
> <username>admin</username>  
> <password>password</password>  
> </server>  
> </servers>
> 
> </settings>
> 
> serve配置的是项目deploy的时候上传的路径和用户名密码

3，在eclispe中新建一个项目测试下。

在新建选maven MavenProject,接着会有选Archetype的界面，这个地方在Filter中找quickstart，地下就能找出quickstart的模板了。选好之后，填好，group Id,artifact Id,Version,就好了，默认建立出来的项目pom.xml文件如下

> <project xmlns=&#8221;[http://maven.apache.org/POM/4.0.0&#8243;][2] xmlns:xsi=&#8221;[http://www.w3.org/2001/XMLSchema-instance&#8221;][3]  
> xsi:schemaLocation=&#8221;<http://maven.apache.org/POM/4.0.0> [http://maven.apache.org/xsd/maven-4.0.0.xsd&#8221;][4]>  
> <modelVersion>4.0.0</modelVersion>
> 
> <groupId>name.codeboy</groupId>  
> <artifactId>qucikstart\_maven\_test</artifactId>  
> <version>0.0.1-SNAPSHOT</version>  
> <packaging>jar</packaging>
> 
> <name>qucikstart\_maven\_test</name>  
> <url>www.codeboy.name</url>
> 
> <properties>  
> <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  
> </properties>
> 
> <distributionManagement>  
> <repository>  
> <id>releases</id>  
> <name>quickstart-releases</name>  
> <url>>http://localhost:8081/artifactory/libs-release-local</url>  
> </repository>  
> <snapshotRepository>  
> <id>snapshots</id>  
> <name>quickstart-snapshots</name>  
> <url>http://localhost:8081/artifactory/libs-snapshot-local</url>  
> </snapshotRepository>  
> </distributionManagement>
> 
> <dependencies>  
> <dependency>  
> <groupId>junit</groupId>  
> <artifactId>junit</artifactId>  
> <version>3.8.1</version>  
> <scope>test</scope>  
> </dependency>  
> </dependencies>  
> </project>

然后run->mvn deploy,在控制台确认请求的是localhost，并且成功上传

distributionManagement配置是表示deploy的时候上传的地方。id必须和settings.xml文件中server中的id配置相同

&nbsp;

ps：eclispe下的varpper简直就是神器啊，具有vi的大部分功能，而且快捷键没有和eclipse冲突，不需要手动去改什么配置，而且可以随时切换到传统编辑器。

就是在查找的时候输入/然后再粘贴就粘到文本里去了，而且u没办法还原，比较麻烦，不是查找，比较麻烦

 [1]: http://m2eclipse.sonatype.org/sites/m2e/ "http://m2eclipse.sonatype.org/sites/m2e/"
 [2]: http://maven.apache.org/POM/4.0.0"
 [3]: http://www.w3.org/2001/XMLSchema-instance"
 [4]: http://maven.apache.org/xsd/maven-4.0.0.xsd"