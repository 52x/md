---
layout: post
title: "Linux下使用nexus搭建maven私服"
date: 2015-03-17 17:26:57 +0800
tags:
comments: true
categories: [java]
keywords: [nexus, maven, 私服, java]
---

有个maven私服可以很方便地管理我们的jar包和发布构建到远程仓库，本文就介绍了如何在linux下一步步使用nexus搭建maven私服。

<!--more-->
原文链接：

<http://tianweili.github.io/blog/2015/03/17/linux-nexus-maven-private-server/>


## 下载安装

最新nexus下载地址：<http://www.sonatype.org/nexus/go>

解压后会在同级目录中，出现两个文件夹：`nexus-oss-webapp-1.8.0`和`sonatype-work`，前者包含了nexus的运行环境和应用程序，后者包含了你自己的配置和数据。

```bash
$ mkdir nexus
$ tar xzvf /home/jili/nexus-2.7.0-05-bundle.tar.gz
$ ls
nexus-2.7.0-05  sonatype-work
```

## 启动nexus

```bash
$ cd bin/
$ ls
jsw  nexus  nexus.bat
$ ./nexus
Usage: ./nexus { console | start | stop | restart | status | dump }
$ ./nexus start
Starting Nexus OSS...
Started Nexus OSS.
```

查看控制台：

```bash
$ ./nexus console
```

显示未启动成功，报错如下：

```bash
$ ./nexus console
Running Nexus OSS...
wrapper  | --> Wrapper Started as Console
wrapper  | Launching a JVM...
wrapper  | JVM exited while loading the application.
jvm 1    | Exception in thread "main" java.lang.UnsupportedClassVersionError: org/sonatype/nexus/bootstrap/jsw/JswLauncher : Unsupported major.minor version 51.0
jvm 1    |      at java.lang.ClassLoader.defineClass1(Native Method)
jvm 1    |      at java.lang.ClassLoader.defineClassCond(ClassLoader.java:632)
jvm 1    |      at java.lang.ClassLoader.defineClass(ClassLoader.java:616)
jvm 1    |      at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:141)
jvm 1    |      at java.net.URLClassLoader.defineClass(URLClassLoader.java:283)
jvm 1    |      at java.net.URLClassLoader.access$000(URLClassLoader.java:58)
jvm 1    |      at java.net.URLClassLoader$1.run(URLClassLoader.java:197)
jvm 1    |      at java.security.AccessController.doPrivileged(Native Method)
jvm 1    |      at java.net.URLClassLoader.findClass(URLClassLoader.java:190)
jvm 1    |      at java.lang.ClassLoader.loadClass(ClassLoader.java:307)
jvm 1    |      at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:301)
jvm 1    |      at java.lang.ClassLoader.loadClass(ClassLoader.java:248)
jvm 1    | Could not find the main class: org.sonatype.nexus.bootstrap.jsw.JswLauncher.  Program will exit.
wrapper  | Reloading Wrapper configuration...
wrapper  | Launching a JVM...
wrapper  | JVM exited while loading the application.
.
.
.
jvm 5    | Exception in thread "main" java.lang.UnsupportedClassVersionError: org/sonatype/nexus/bootstrap/jsw/JswLauncher : Unsupported major.minor version 51.0
jvm 5    |      at java.lang.ClassLoader.defineClass1(Native Method)
jvm 5    |      at java.lang.ClassLoader.defineClassCond(ClassLoader.java:632)
jvm 5    |      at java.lang.ClassLoader.defineClass(ClassLoader.java:616)
jvm 5    |      at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:141)
jvm 5    |      at java.net.URLClassLoader.defineClass(URLClassLoader.java:283)
jvm 5    |      at java.net.URLClassLoader.access$000(URLClassLoader.java:58)
jvm 5    |      at java.net.URLClassLoader$1.run(URLClassLoader.java:197)
jvm 5    |      at java.security.AccessController.doPrivileged(Native Method)
jvm 5    |      at java.net.URLClassLoader.findClass(URLClassLoader.java:190)
jvm 5    |      at java.lang.ClassLoader.loadClass(ClassLoader.java:307)
jvm 5    |      at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:301)
jvm 5    |      at java.lang.ClassLoader.loadClass(ClassLoader.java:248)
jvm 5    | Could not find the main class: org.sonatype.nexus.bootstrap.jsw.JswLauncher.  Program will exit.
wrapper  | There were 5 failed launches in a row, each lasting less than 300 seconds.  Giving up.
wrapper  |   There may be a configuration problem: please check the logs.
wrapper  | <-- Wrapper Stopped

```

原因：查找原因是JDK版本过低造成的，升级到最新的JDK7或者使用`nexus-2.4-bundle.tar.gz`版本JDK6会支持.

Nexus所有版本下载地址：<http://www.sonatype.org/nexus/archived>

下载Nexus2.4重来

```bash
$ ls
nexus-2.4.0-09  sonatype-work
$ cd nexus-2.4.0-09/bin/
$ ls
jsw  nexus  nexus.bat
$ ./nexus
Usage: ./nexus { console | start | stop | restart | status | dump }
$ ./nexus start
Starting Nexus OSS...
Started Nexus OSS.
$ ./nexus console
Running Nexus OSS...
Nexus OSS is already running.
```

控制台显示启动成功。

查看nexus日志：

```bash
$ cd nexus-2.4.0-09/logs
$ ls
wrapper.log
$ tail -f wrapper.log

```

## 配置nexus

访问网址：<http://yourhostname:8081/nexus>

![](http://7u2i08.com1.z0.glb.clouddn.com/java/nexus-maven-1.png)

右上角以admin登陆，默认用户名/密码：admin/admin123。

![](http://7u2i08.com1.z0.glb.clouddn.com/java/nexus-maven-2.png)

3rd party、Snapshots、Releases这三个，分别用来保存第三方jar、项目组内部的快照、项目组内部的发布版.

## 手动添加第三方jar

将第三方的jar上传到nexus上面：

![](http://7u2i08.com1.z0.glb.clouddn.com/java/nexus-maven-3.png)

![](http://7u2i08.com1.z0.glb.clouddn.com/java/nexus-maven-4.png)

点击Upload Artifact(s)按钮提交后即上传。

查看上传的jar包如下：

![](http://7u2i08.com1.z0.glb.clouddn.com/java/nexus-maven-5.png)

在项目中使用私服的jar包配置pom.xml如下：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.vclk.mkt.crawler</groupId>
	<artifactId>MarketingCrawler</artifactId>
	<packaging>jar</packaging>
	<version>0.3</version>
	<name>MarketingCrawler</name>
	<url>http://maven.apache.org</url>

	<!-- 仓库地址 -->
	<repositories>
		<repository>
			<id>nexus</id>
			<name>Team Nexus Repository</name>
			<url>http://yourhostname:8081/nexus/content/groups/public</url>
		</repository>
	</repositories>
	
	<!-- 插件地址 -->
	<pluginRepositories>
		<pluginRepository>
			<id>nexus</id>
			<name>Team Nexus Repository</name>
			<url>http://yourhostname:8081/nexus/content/groups/public</url>
		</pluginRepository>
	</pluginRepositories>

	<!-- jar -->
	<dependencies>
		<dependency>
			<groupId>de.innosystec</groupId>
			<artifactId>java-unrar</artifactId>
			<version>0.5</version>
		</dependency>
	</dependencies>
</project>

```

Maven在项目根目录下执行mvn eclipse:eclipse命令时，所依赖的jar包都会从私服中下载到本地并关联上项目，私服中没有就会从网络上下载到私服，本地再从私服下载。

![](http://7u2i08.com1.z0.glb.clouddn.com/java/nexus-maven-6.png)

## 自动发布构件到远程仓库

在工程的pom.xml中添加：

```xml
<distributionManagement>
	<repository>
		<id>nexus-releases</id>
		<url>http://yourhostname:8081/nexus/content/repositories/releases/</url>
	</repository>
	<snapshotRepository>
		<id>nexus-snapshots</id>
		<url>http://yourhostname:8081/nexus/content/repositories/snapshots/</url>
	</snapshotRepository>
</distributionManagement>

```

进入maven的安装目录apache-maven-3.1.1\conf目录下，向settings.xml配置文件中的<servers>语句块中添加如下所示：

```xml
<servers>
	<server>
		<id>nexus-releases</id>
		<username>admin</username>
		<password>admin123</password>
	</server>
	<server>
		<id>nexus-snapshots</id>
		<username>admin</username>
		<password>admin123</password>
	</server>
</servers>
```

进入windows命令行，在工程所在目录下执行

```bash
mvn deploy
```

所部署的包就自动上传到了nexus安装目录下的`/maven/nexus/sonatype-work/nexus/storage/releases/com/vclk/mkt/crawler/MarketingCrawler/0.3`目录

## nexus仓库中各目录介绍

项目中的各种jar包和项目快照等都放在`/nexus/sonatype-work/nexus/storage/`目录下，在这个目录下包括以下各种目录和存放相应文件。

`/nexus/sonatype-work/nexus/storage/central` - 用于放置maven从中央仓库中下载下来的项目pom.xml中配置到的相关jar包；

`/nexus/sonatype-work/nexus/storage/thirdparty` - 用于放置自己手动上传的第三方jar包；

`/nexus/sonatype-work/nexus/storage/releases` - 用于放置项目deploy后的发布版。

---

作者：[李天炜](http://tianweili.github.io/)

原文链接：<http://tianweili.github.io/blog/2015/03/17/linux-nexus-maven-private-server/>

转载请注明作者和文章出处，谢谢。