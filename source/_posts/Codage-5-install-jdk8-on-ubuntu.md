title: Install jdk8 on Ubuntu 14.04
date: 2015-09-24 21:39:03
categories: Codage|编程
tags: [Ubuntu, jdk, Linux, vi, vim]
---
JDK is an important tool which consists the base of Java programming as well as supports many other programming IDEs.
This article introduces the installation method of Oracle jdk 8 on Ubuntu 14.04.
<!-- more --> 

## Download Linux jdk 8u60
You could either download the `.tar.gz` package from [oracle website](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html). 
The default download path (not from command terminal) is `/Home/Downloads/`.

## Decompress the jdk file
Create a folder under `/usr/lib`. 
``` bash
$ mkdir -p /usr/lib/jvm
```
If your access is denied, that means you need to use root to realize that.
You can either:
``` bash
$ sudo -i
```
or change directly the right your username group as root:
``` bash
$ sudo gedit /etc/passwd
```
In the file poped up, find the line with your username, for instance, your username is "blabla" then find " blabla:x:1000:1000:crystal:/home/linuxidc:/bin/bash".
Modify the two 1000 to two 0: "blabla:x:0:0:crystal:/home/linuxidc:/bin/bash". This method is not recommanded since it's a bit dangerous.

You will find that the start symbol of command line is "#" instead of "$" and you can create the folder again.

Then type the following code (suppose we don't modify the user right and the downloaded file is saved under /Home/):
``` bash
$ sudo mv jdk-8u60-linux-x64.tar.gz /usr/lib/jvm
$ cd /usr/lib/jvm
$ sudo tar -xzvf jdk-8u60-linux-x64.tar.gz
$ sudo ln -s jdk1.8.0_60 java-8
```

## Set environment variable
``` bash
$ vi ~/.bashrc
```
If you feel sufferred using vi, try to install vim then use it.
``` bash
$ sudo apt-get remove vim-common
$ sudo apt-get install vim
$ vim ~/.bashrc
```
Press `i` to switch to insert mode and enter the info below:
``` bash
export JAVA_HOME=/usr/lib/jvm/java-8
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```
Press `Esc` to quit insert mode. Enter `:w!` to save the changes and enter `:q` to quit vi.
Source the file and activate the changes.
``` bash
$ source ~/.bashrc
```
## Set default jdk version
The default version of jdk in some OS is OpenJDK. So you need to specify the default to be jdk from Oracle.
``` bash
$ sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-8/bin/java 300
$ sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/java-8/bin/javac 300
$ sudo update-alternatives --config java
```
In Ubuntu 14.04 OpenJDK is not preloaded, so I don't need to do this step.

## Test and configure
Check if the installation is successful.
``` bash
$ java -version
java version "1.8.0_60"
Java(TM) SE Runtime Environment (build 1.8.0_60-b27)
Java HotSpot(TM) 64-Bit Server VM (build 25.60-b23, mixed mode)
```
The info returned proves that we've successfully installed Oracle jdk 8u60.

------------------------------------------

## Reference:
1. [Ubuntu 14.04 安装 JDK 8，ubuntu14.04](http://www.bkjia.com/xtzh/881605.html)
2. [vi in Linux](http://blog.csdn.net/leonardo811/article/details/8133835)
3. [解决ubuntu中vi不能正常使用方向键与退格键的问题](http://hongzhguan.iteye.com/blog/1479563)