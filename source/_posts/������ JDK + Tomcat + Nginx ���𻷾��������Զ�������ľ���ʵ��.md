title: 服务器 JDK + Tomcat + Nginx 部署环境整体搭建及自动化部署的具体实现
date: 2017-01-19 11:39:44
permalink: linux
tags:
- linux
- 部署
- 服务器
categories:
- linux

---
本篇文章主要围绕一个问题：本地开发完项目后如何简单部署到服务器？
这个问题太宽泛了，因此，提出了以下几个要点：
1. 第一次怎么部署
2. 增量开发后怎么`简单`部署?
3. 服务器环境整体如何规划？各个项目怎么存放，日志是否集中存放 ？项目各个项目运行环境与部署怎么分开？。。。
4. 如何用脚本自动化部署？
5. 如何用几句话归纳部署？

经过整篇文章，我想会有一定的答案。


文章的各个章节并不是按照解决问题的顺序写的，而是按照环境部署、项目配置、自动化部署一步步递进的，完成服务器 JDK + Tomcat + Nginx 部署环境整体搭建及自动化部署的具体实现。

## 1.服务器环境整体规划
在本篇文章的部署方式中，会涉及以下4个总目录

	1. 项目目录 /opt/web
	2. 日志目录 /opt/log
	3. JDK/Tomcat/Nginx等安装环境的父目录  /opt/soft
	4. 项目应用war包上传到服务器的目录 /root/deploy

## 2.运行环境安装
### 2.1 JDK
1. 官网下载 JDK8，解压到/opt/soft ，解压文件夹为jdk1.8.0_11
2. 配置 JDK 环境变量
 在`/etc/profile`文件末尾添加以下配置 ：
```bash
#jdk enviroment
JAVA_HOME=/opt/soft/jdk1.8.0_11/
PATH=$JAVA_HOME/bin:$PATH
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JAVA_HOME
export PATH
export CLASSPATH 
```
 保存退出。终端执行命令` source /etc/profile`  使配置生效  
3. 验证 JDK 配置
 终端执行命令：`java -version`
```bash
ubuntu# java -version
java version "1.8.0_111"
Java(TM) SE Runtime Environment (build 1.8.0_111-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.111-b14, mixed mode)
```
> **JDK 配置成功！！！**

### 2.2 Tomcat
1. 官网下载tomcat，我这里下载的是apache-tomcat-7.0.73，解压到/opt/soft ，解压文件夹为apache-tomcat-7.0.73
2. 进入tomcat目录下的bin目录下，执行tomcat启动脚本（tomcat依赖jdk，应先确定jdk已经安装）
```bash
ubuntu# sh startup.sh 
Using CATALINA_BASE:   /opt/soft/apache-tomcat-7.0.73
Using CATALINA_HOME:   /opt/soft/apache-tomcat-7.0.73
Using CATALINA_TMPDIR: /opt/soft/apache-tomcat-7.0.73/temp
Using JRE_HOME:        /opt/soft/jdk1.8.0_11/
Using CLASSPATH:       /opt/soft/apache-tomcat-7.0.73/bin/bootstrap.jar:/opt/soft/apache-tomcat-7.0.73/bin/tomcat-juli.jar
Tomcat started.
ubuntu# 

```
出现以上信息证明tomcat已经成功启动，打开 http://ip:8080 或 [http:/localhost:8080](http://localhost:8080),如果能看到tomcat系统界面，说明已经安装成功，可以进行下一步了。
 
 **常见命令**
>启动tomcat ：sh startup.sh
关闭tomcat  ：sh shutdown.sh
查看tomcat运行进程 ： ps aux | grep tomcat 或 ps aux | grep java 或ps -ef | grep java  或 ps -ef | grep tomcat
查看端口使用情况   netstat –apn 或 netstat -tunlp  ，可以结合 grep ，如 netstat -apn | grep (端口号)  或 netstat -apn | grep java
其中最后一下为pid/program name ，如果端口被占用， 可以进一步使用命令：ps -ef  | grep (pid) 查看，可以查出是哪个进程占用了端口号，可以kill portId 杀死进程。

### 2.3 Nginx
 这里演示的系统为`Ubuntu`

Ubuntu平台编译环境安装
```bash
apt-get install build-essentail
apt-get install libtool
```
安装Nginx依赖包 OpenSSL、zlib、 pcre
这三个包可以直接在终端下载，如果网速慢，建议在本地下载再上传到服务器。
安装Nginx依赖包
```bash
apt-get install libpcre3 libpcre3-dev zlib1g-dev libssl-dev build-essential
```
选择下载保存目录 cd /opt/soft
下载OpenSSL
```bash
wget ftp://ftp.openssl.org/source/openssl-1.1.0c.tar.gz
```
下载zlib
```bash
wget http://zlib.net/zlib128.zip
```
下载pcre
```bash
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.39.tar.gz
```
安装OpenSSL
```bash
tar zxvf openssl-1.0.0c.tar.gz
cd openssl-1.0.0c/
./config --prefix=/opt/soft --openssldir=/opt/soft/ssl
make && make install

./config shared --prefix=/opt/soft --openssldir=/opt/soft/ssl
make clean
make && make install
```
安装zlib库
```bash
unzip zlib128.zip
cd zlib128/
./configure --prefix=/opt/soft
make && make install
```
安装pcre库
```bash
 tar zxvf pcre-8.39.tar.gz
cd pcre-8.39/
./configure --prefix=/opt/soft
make && make install
```
 查看pcre版本
```bash
pcre-config --version
```
安装Nginx
1下载 Nginx，下载地址：[http://nginx.org/download/nginx-1.6.2.tar.gz](http://nginx.org/download/nginx-1.6.2.tar.gz)
```bash
wget http://nginx.org/download/nginx-1.6.2.tar.gz
```
2解压安装包 
```bash
tar zxvf nginx-1.6.2.tar.gz
```
3进入安装包目录
```bash
cd nginx-1.6.2
```
4编译安装
```bash
[root@bogon nginx-1.6.2]# ./configure --prefix=/opt/soft/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/opt/soft/src/pcre-8.39
[root@bogon nginx-1.6.2]# make
[root@bogon nginx-1.6.2]# make install
```
5查看Nginx版本
```bash
/opt/soft/nginx/sbin/nginx -v
nginx version: nginx/1.6.2
```

6启动nginx
```bash
/opt/soft/nginx/sbin/nginx
```
>到此，nginx安装成功

**Nginx常用命令**
>/opt/soft/nginx/sbin/nginx -v                   #查看 Nginx版本
/opt/soft/nginx/sbin/nginx                       #启动 Nginx
/opt/soft/nginx/sbin/nginx -t                   # 检查配置文件ngnix.conf的正确性
/opt/soft/nginx/sbin/nginx -s reload        #重新载入配置文件
/opt/soft/nginx/sbin/nginx -s reopen       #重启 Nginx
/opt/soft/nginx/sbin/nginx -s stop           #停止 Nginx


	jdk\tomcat\nginx 安装完毕，查看服务器目录
/opt/soft 目录下文件结构
```bash
 ➜  apache-tomcat-7.0.73 ll /opt/soft       
总用量 8.4M
drwxr-xr-x  9 root root 4.0K 12月 27 09:25 apache-tomcat-7.0.73
drwxr-xr-x  2 root root 4.0K 12月 28 15:44 bin
drwxr-xr-x  3 root root 4.0K 12月 28 15:38 include
drwxr-xr-x  8 uucp  143 4.0K 6月  17  2014 jdk1.8.0_11
drwxr-xr-x  4 root root 4.0K 12月 28 15:44 lib
drwxr-xr-x 11 root root 4.0K 12月 28 16:22 nginx
drwxr-xr-x  9 1001 1001 4.0K 12月 28 16:14 nginx-1.6.2
-rwxr-xr-x  1 root root 786K 12月 28 16:13 nginx-1.6.2.tar.gz
drwxr-xr-x 19 root root 4.0K 12月 28 16:05 openssl-1.1.0c
-rwxr-xr-x  1 root root 5.0M 12月 28 15:34 openssl-1.1.0c.tar.gz
drwxr-xr-x  9 1169 1169  12K 12月 28 16:15 pcre-8.39
-rwxr-xr-x  1 root root 2.0M 12月 28 15:23 pcre-8.39.tar.gz
drwxr-xr-x  4 root root 4.0K 12月 28 15:32 share
drwxr-xr-x  5 root root 4.0K 12月 28 15:44 ssl
drwxr-xr-x 14 root root 4.0K 12月 28 15:30 zlib-1.2.8
-rwxr-xr-x  1 root root 679K 12月 28 15:28 zlib128.zip
```

## 3. Tomcat多实例多应用部署
### 3.1. 部署应用分类
`单实例单应用`：一个tomcat目录，webapps目录下只跑一个war包，最基本的部署方式。
`单实例多应用`：一个tomcat目录，webapps目录下跑多个war包。 启动、关闭 tomcat对所有war包起作用。
`多实例单应用`：多个tomcat目录，每个tomcat只跑一个应用，并且是同一个应用。通过修改tomcat端口配合nginx进行负载均衡。
`多实例多应用`：把一个tomcat分为2部分，一部分为公用的bin目录与lib目录，其余文件夹作为第二部分，可以复制多次，因此tomcat只有一个，但是实例可以有多个，每个实例在单独部署，与单实例多应用相似。

>1. 一个tomcat，充分利用资源，对tomcat的更新维护也很方便
>2. 多个实例，各个实例互不影响
>3. 各个项目相互独立，启动关闭不影响
>4. 部署容易，启动快速
### 3.2 目录结构认识
 一个刚解压出来的tomcat打包文件应该有以下几个目录
 bin:主要存放脚本文件，例如比较常用的windows和linux系统中启动和关闭脚本
 conf:主要存放配置文件，其中最重要的两个配置文件是server.xml和web.xml
 lib:主要存放tomcat运行所依赖的包
 logs:主要存放运行时产生的日志文件，例如catalina.{date}.log等
 temp:存放tomcat运行时产生的临时文件，例如开启了hibernate缓存的应用程序，会在该目录下生成一些文件
 webapps:部署web应用程序的默认目录
 work:主要存放由JSP文件生成的servlet（java文件以及最终编译生成的class文件）
### 3.3 多实例原理
tomcat中两个比较重要的概念（通常也是两个系统变量）——CATALINA_HOME和CATALINA_BASE：
 * CATALINA_HOME：即指向Tomcat安装路径的系统变量
 * CATALINA_BASE：即指向活跃配置路径的系统变量

通过设置这两个变量，就可以将tomcat的安装目录和工作目录分离，从而实现tomcat多实例的部署。

Tomcat官方文档指出，CATALINA_HOME路径的路径下只需要包含bin和lib目录，这也就是支持tomcat软件运行的目录，而CATALINA_BASE设置的路径可以包括上述所有目录，不过其中bin和lib目录并不是必需的，缺省时会使用CATALINA_HOME中的bin和conf。如此，我们就可以使用一个tomcat安装目录部署多个tomcat实例，这样的好处在于方便升级，就可以在不影响tomcat实例的前提下，替换掉CATALINA_HOME指定的tomcat安装目录。

简而言之：
* CATALINA_HOME：bin      lib
* CATALINA_BASE：conf      webapps      temp      logs      work

> **server.xml**

这里部署多实例要修改的两个端口：
* Server Port：该端口用于监听关闭tomcat的shutdown命令，默认为8005
* Connector Port：该端口用于监听HTTP的请求，默认为8080

简单的多实例只需要保证多实例中的Server Port和Connect Port不同即可。

另外要修改的是Host配置，Host就是所谓的虚拟主机，对应包含了一个或者多个web应用程序。其中：

* name： 虚拟主机的名称，一台主机表示了完全限定的域名或IP地址，默认为localhost，同时也是唯一的host，进入tomcat的所有http请求都会映射到该主机上
* appBase：web应用程序目录的路径，可以是CATALINA_HOME的相对路径，也可以写成绝对路径，默认情况下为$CATALINA_HOME/webapps
* unpackWARs： 表示是否自动解压war包
* autoDeploy：所谓的热部署，即在tomcat正在运行的情况下，如果有新的war加入，则会立即执行部署操作
* 另外再介绍一个Host中的属性—deployOnStartup：表示tomcat启动时是否自动部署appBase目录下所有的Web应用程序，默认为true。这个属性和autoDeploy会产生两次部署的“副作用”：一次是tomcat启动时就开始部署，第二次就是autoDeploy引起的热部署。因此最好将autoDeploy置为false

最后再说明一下Context的配置，它出现在Host配置内，一个Context的配置就代表了一个web应用程序，如果配置多应用程序，就需要在Host下配置多个Context。其中：

* path：表示访问入口，例如，path=”/abc”，则访问localhost:8080/abc时，就可以访问该Context对应的应用程序。如果path=””，则直接用localhost:8080就可以访问。
* docBase：表示应用程序的解包目录或者war文件路径，是Host的appBase配置目录的相对路径，也可以是直接写成绝对路径，但是不要将appBase的值，作为docBase配置路径的前缀，例如appBase=”somedir”，docBase=”somedir-someapp.war”，这样的配置会导致部署错误。

### 3.4 多实例多应用部署
假设一个完好的可以运行的web应用程序已经打包成test.war，放在/root/deploy/test目录下，其中包含一个简单的index.jsp
```xml
<html>
<body>
<h2>Hello World!</h2>
</body>
</html>
```
#### 创建tomcat实例
之前我们已经在/opt/soft/下成功安装了tomcat
```bash
ubuntu# ls /opt/soft/apache-tomcat-7.0.73
bin  conf  lib    LICENSE  logs  NOTICE  RELEASE-NOTES  RUNNING.txt  temp  webapps  work
```

我们的实例目录放在/opt/web/test 下
复制相关实例文件夹到该目录下
```bash
ubuntu# ls /opt/web/test
conf  debug.sh	logs  restart.sh  run.sh  stop.sh  webapps  work
```
其中run.sh 、stop.sh、debug.sh、restart.sh 脚本文件本不属于tomcat实例的，是为了方便启动、停止tomcat实例而创建的脚本

#### 创建启停脚本
run.sh 脚本文件内容：
```bash
#!/bin/sh
TOMCAT_HOME=/opt/soft/apache-tomcat-7.0.73
CATALINA_BASE=$(cd "$(dirname $0)"; pwd)
CATALINA_PID=$CATALINA_BASE/tomcat.pid
JAVA_HOME=/opt/soft/jdk1.8.0_11/
CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/apr/lib
PATH=$PATH:$JAVA_HOME/bin

#路径为【/opt/web/xxx】时，项目名则为【xxx】
PROJECT_NAME=`echo $CATALINA_BASE | awk -F "/" '{print $NF}'`

#检测temp目录是否存在，不存在则创建
if [ ! -d "$CATALINA_BASE/temp" ]; then
    mkdir "$CATALINA_BASE/temp"
fi

#日志目录名
PROJECT_LOG=/opt/log/$PROJECT_NAME/catalina.out

JAVA_OPTS="-server -Xms32m -Xmx512m -Xmn32m -Xss1024K -XX:PermSize=32m -XX:MaxPermSize=64m -XX:ParallelGCThreads=32 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:SurvivorRatio=4 -XX:MaxTenuringThreshold=10 -XX:CMSInitiatingOccupancyFraction=80"

export TOMCAT_HOME CATALINA_BASE CATALINA_PID JAVA_HOME CLASSPATH PATH JAVA_OPTS PROJECT_NAME PROJECT_LOG LD_LIBRARY_PATH

proj_dir=$CATALINA_BASE
if [ -n "$proj_dir" ]
then
        ps aux | grep "$proj_dir" | grep -v "grep" | grep -v "$CATALINA_BASE/\(start.sh\|stop.sh\|restart.sh\|logs/catalina.out\)" > /dev/null
        status=$?

        if  [ $status -eq 0 ] && [ -s $CATALINA_PID  ]
        then
                echo "The $proj_dir tomcat is running and the pid_file is exist !"
                exit 1
        elif [ $status -eq 0 ] || [ -s $CATALINA_PID  ]
        then
                echo "kill one"
                ps aux | grep "$proj_dir" | grep -v "grep" | grep -v "$CATALINA_BASE/\(start.sh\|stop.sh\|restart.sh\|logs/catalina.out\)" | awk '{print $2}' | xargs -i kill -9 {}
                > $CATALINA_PID
                $TOMCAT_HOME/bin/catalina.sh start
        else
                $TOMCAT_HOME/bin/catalina.sh start
        fi

else
        echo "start failt!!!"
        exit 1
fi
```

stop.sh脚本内容
```bash
#!/bin/sh
TOMCAT_HOME=/opt/soft/apache-tomcat-7.0.73
#CATALINA_BASE=/opt/web/test
CATALINA_BASE=$(cd "$(dirname $0)"; pwd)
CATALINA_PID=$CATALINA_BASE/tomcat.pid
JAVA_HOME=/opt/soft/jdk1.8.0_11/
CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
PATH=$PATH:$JAVA_HOME/bin

#路径为【/opt/web/xxx】时，项目名则为【xxx】
PROJECT_NAME=`echo $CATALINA_BASE | awk -F "/" '{print $NF}'`
echo $PROJECT_NAME

#日志目录名
PROJECT_LOG=/opt/log/$PROJECT_NAME/

export TOMCAT_HOME CATALINA_BASE CATALINA_PID JAVA_HOME CLASSPATH PATH JAVA_OPTS PROJECT_NAME PROJECT_LOG

proj_dir=$CATALINA_BASE
$TOMCAT_HOME/bin/catalina.sh stop
sleep 5
if [ -n "$proj_dir" ]
then
        ps aux | grep "$proj_dir" | grep -v "grep" | grep -v "$CATALINA_BASE/\(start.sh\|stop.sh\|restart.sh\|logs/catalina.out\)" > /dev/null
        status=$?
        if [ $status -eq 0 ] || [ -s $CATALINA_PID  ]
        then
                echo "kill it"
                ps aux | grep "$proj_dir" | grep -v "grep" | grep -v "$CATALINA_BASE/\(start.sh\|stop.sh\|restart.sh\|logs/catalina.out\)" |  awk '{print $2}' | xargs -i kill -9 {}
                > $CATALINA_PID
        fi
else
        echo "stop failt!!!"
        exit 1
fi
```
debug.sh 脚本内容
```bash
#!/bin/sh
TOMCAT_HOME=/opt/soft/apache-tomcat-7.0.73
CATALINA_BASE=$(cd "$(dirname $0)"; pwd)
CATALINA_PID=$CATALINA_BASE/tomcat.pid
JAVA_HOME=/opt/soft/jdk1.8.0_11/
CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/apr/lib
PATH=$PATH:$JAVA_HOME/bin

#路径为【/opt/web/xxx】时，项目名则为【xxx】
PROJECT_NAME=`echo $CATALINA_BASE | awk -F "/" '{print $NF}'`

#日志目录名
PROJECT_LOG=/opt/log/$PROJECT_NAME/catalina.out

JAVA_OPTS="-server -Xms32m -Xmx512m -Xmn32m -Xss1024K -XX:PermSize=32m -XX:MaxPermSize=64m -XX:ParallelGCThreads=32 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:SurvivorRatio=4 -XX:MaxTenuringThreshold=10 -XX:CMSInitiatingOccupancyFraction=80"

export TOMCAT_HOME CATALINA_BASE CATALINA_PID JAVA_HOME CLASSPATH PATH JAVA_OPTS PROJECT_NAME PROJECT_LOG LD_LIBRARY_PATH

proj_dir=$CATALINA_BASE
if [[ -n "$proj_dir" ]]
then
        ps aux | grep "$proj_dir" | grep -v "grep" | grep -v "$CATALINA_BASE/\(start.sh\|stop.sh\|restart.sh\|logs/catalina.out\)" > /dev/null
        status=$?

        if  [[ $status -eq 0 ]] && [[ -s $CATALINA_PID  ]]
        then
                echo "The $proj_dir tomcat is running and the pid_file is exist !"
                exit 1
        elif [[ $status -eq 0 ]] || [[ -s $CATALINA_PID  ]]
        then
                echo "kill one"
                ps aux | grep "$proj_dir" | grep -v "grep" | grep -v "$CATALINA_BASE/\(start.sh\|stop.sh\|restart.sh\|logs/catalina.out\)" | awk '{print $2}' | xargs -i kill -9 {}
                > $CATALINA_PID
                $TOMCAT_HOME/bin/catalina.sh jpda start
        else
                $TOMCAT_HOME/bin/catalina.sh jpda start
        fi
else
        echo "start failt!!!"
        exit 1
fi
```

restart.sh 脚本内容
```bash
#!/bin/sh
TOMCAT_HOME=/opt/soft/apache-tomcat-7.0.73
CATALINA_BASE=$(cd "$(dirname $0)"; pwd)
CATALINA_PID=$CATALINA_BASE/tomcat.pid
JAVA_HOME=/opt/soft/jdk/1.8.11/jdk1.8.0_11
CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
PATH=$PATH:$JAVA_HOME/bin
JAVA_OPTS="-server -Xms1g -Xmx1g -Xmn512m -Xss1024K -XX:PermSize=256m -XX:MaxPermSize=512m -XX:ParallelGCThreads=8 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:SurvivorRatio=4 -XX:MaxTenuringThreshold=10 -XX:CMSInitiatingOccupancyFraction=80"

#路径为【/opt/web/xxx】时，项目名则为【xxx】
PROJECT_NAME=`echo $CATALINA_BASE | awk -F "/" '{print $NF}'`

#日志目录名
PROJECT_LOG=/opt/log/$PROJECT_NAME/catalina.out

export TOMCAT_HOME CATALINA_BASE CATALINA_PID JAVA_HOME CLASSPATH PATH JAVA_OPTS PROJECT_NAME PROJECT_LOG

sh $CATALINA_BASE/stop.sh
sh $CATALINA_BASE/start.sh
```

#### 修改conf文件夹下的server.xml
把对应的Host节点修改为下面的内容，并修改相关的路径
```xml
<Host name="localhost"  appBase="/opt/web/test/webapps" autoDeploy="false" deployOnStartup="false">
     <Context path="" docBase="/opt/web/test/webapps" crossContext="true" allowLinking="true"reloadable="false">
         <JarScanner scanAllDirectories="true" />
         <Valve className="org.apache.catalina.valves.AccessLogValve" directory="/opt/web/test/logs/" prefix="gh_access_log." suffix=".log" pattern="%a %A %t %S %b %r %s %{Referer}i %{User-Agent}i %D" resolveHosts="false" />
     </Context>
</Host>
```

除此之外，可选的修改Server的端口号以及Connector的端口号。
一个实例就这样完成，以后每次想要部署一个项目，就可以简单的复制这个实例文件夹，修改一下配置，就行了。

注意：复制tomcat实例的时候webapps下面可能已经存在其他项目的文件，我们要清空webapps，然后把test.war解压到webapps下面，而不是直接放war包。

我们再创建一个项目为test1
#### 现在我们的目录结构如下:
```bash
ubuntu# ls test test1
test:
conf  debug.sh	logs  restart.sh  run.sh  stop.sh  webapps  work

test1:
conf  debug.sh	logs  restart.sh  run.sh  stop.sh  webapps  work
```

#### 启动tomcat多实例
```bash
ubuntu# test/run.sh
Using CATALINA_BASE:   /opt/web/test
Using CATALINA_HOME:   /opt/soft/apache-tomcat-7.0.73
Using CATALINA_TMPDIR: /opt/web/test/temp
Using JRE_HOME:        /opt/soft/jdk1.8.0_11/
Using CLASSPATH:       /opt/soft/apache-tomcat-7.0.73/bin/bootstrap.jar:/opt/soft/apache-tomcat-7.0.73/bin/tomcat-juli.jar
Using CATALINA_PID:    /opt/web/test/tomcat.pid
Tomcat started.
ubuntu# test1/run.sh
Using CATALINA_BASE:   /opt/web/test1
Using CATALINA_HOME:   /opt/soft/apache-tomcat-7.0.73
Using CATALINA_TMPDIR: /opt/web/test1/temp
Using JRE_HOME:        /opt/soft/jdk1.8.0_11/
Using CLASSPATH:       /opt/soft/apache-tomcat-7.0.73/bin/bootstrap.jar:/opt/soft/apache-tomcat-7.0.73/bin/tomcat-juli.jar
Using CATALINA_PID:    /opt/web/test1/tomcat.pid
Tomcat started.
ubuntu# netstat -tunlp
激活Internet连接 (仅服务器)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.1.1:53            0.0.0.0:*               LISTEN      7237/dnsmasq   
tcp6       0      0 :::8080                 :::*                    LISTEN      10182/java     
tcp6       0      0 127.0.0.1:8005          :::*                    LISTEN      10258/java     
tcp6       0      0 :::8070                 :::*                    LISTEN      10258/java     
tcp6       0      0 127.0.0.1:8006          :::*                    LISTEN      10182/java     
tcp6       0      0 :::8009                 :::*                    LISTEN      10182/java     
udp        0      0 0.0.0.0:57782           0.0.0.0:*                           7062/avahi-daemon:
udp        0      0 0.0.0.0:49141           0.0.0.0:*                           7237/dnsmasq   
udp        0      0 0.0.0.0:41976           0.0.0.0:*                           7237/dnsmasq   
udp        0      0 127.0.1.1:53            0.0.0.0:*                           7237/dnsmasq   
udp        0      0 0.0.0.0:68              0.0.0.0:*                           9541/dhclient   
udp        0      0 0.0.0.0:48730           0.0.0.0:*                           7237/dnsmasq   
udp        0      0 0.0.0.0:631             0.0.0.0:*                           7120/cups-browsed
udp        0      0 0.0.0.0:47809           0.0.0.0:*                           7237/dnsmasq   
udp        0      0 0.0.0.0:5353            0.0.0.0:*                           7062/avahi-daemon:
udp        0      0 0.0.0.0:36600           0.0.0.0:*                           7237/dnsmasq   
udp6       0      0 :::46484                :::*                                7062/avahi-daemon:
udp6       0      0 :::5353                 :::*                                7062/avahi-daemon:
ubuntu#
```

可以看到8081，8082，8083等其他端口都已经启动，现在可以使用curl命令进行访问
```bash
ubuntu# curl localhost:8080

<html>
<head>
    <meta charset="utf-8">
</head>
<body>
<h2>Hello World!</h2>
<h2>Test 项目!</h2>
</body>
</html>
ubuntu# curl localhost:8070

<html>
<head>
    <meta charset="utf-8">
</head>
<body>
<h2>Hello World!</h2>
<h2>Test 项目!</h2>
</body>
</html>
ubuntu#
```
多实例部署成功，以后可以简单的复制其中一个实例，修改相关配置即可。

## 4. Nginx配置
本篇文章重点不在Nginx,简单带过。

在配置文件nginx.conf下的http节点中，引入了两行代码

```profile
http
        {
                include       mime.types;
                default_type  application/octet-stream;

	            ......
	            ......
	            ......

                #在以下两个文件具体配置
                include /usr/local/nginx/conf/conf.d/upstream.conf;
                include /usr/local/nginx/conf/conf.d/ld.conf;
}
```
因此每个项目站点的具体配置是在`upstream.conf `与`ld.conf`中配置的。

其中`ld.conf`下配置的是具体站点的server 站点映射关系
```profile

......
other server
......

server  {
    listen  80;
    server_name  www.dongxiaoxia.xyz;
    server_name_in_redirect off;
    client_header_timeout  30;
    client_body_timeout  30;
    charset  UTF-8;
    client_max_body_size  10000k;
    allow  all;

  location / {
        index   index.html index.php index.jsp index.htm;
        proxy_pass      http://test_pool;
        proxy_redirect  off;
        proxy_set_header Host $host;
        proxy_set_header CURRENT_URL http://$Host$request_uri;
        proxy_set_header X-USER_IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
  access_log  /opt/web/test/logs/test.log  main;
}

......
......
```

其中`upstream.conf`下配置的是具体站点的 pool 映射关系

```profile

......
other pool
......

upstream test_pool {
        server 127.0.0.1:8303 weight=15 max_fails=2 fail_timeout=30s;
}

......
......
```
需要注意的是，在修改了Nginx配置文件后，一定要执行`/opt/soft/nginx/sbin/nginx -s reload `重新加载配置 


## 5. 自动化部署脚本
 服务器每次部署都要拷贝war包到服务器，停止服务器，再开启服务器，不太方便；而且服务器的配置文件应该相对于开发环境不太容易变动，或者开发人员根本无法得知服务器配置信息，因此带来另两个不好的原因，war包整体覆盖会有问题，war包体积太大。因此本地打包war包时要去掉相应文件， 服务器也要相应处理。 

假设场景：本地编译打包项目war包。假如项目名为test，war包为test.war，部署时要求去除日志文件、项目配置文件、以及依赖的外部jar包。

因此在项目根目录下创建`deploy.bat` 脚本，内容为：
```bash
@echo off

echo ...............................................................................................................................................................
echo This shell script does not put lib directory into package. If you want to, please just upload 'test.war' in target directory.
echo ...............................................................................................................................................................

set SVN_ROOT=D:\svn\test
set TARGET_PATH=%SVN_ROOT%\target\test
set DEPLOY=%SVN_ROOT%\deploy

rd /s /q %DEPLOY%
md %DEPLOY%
echo %TARGET_PATH%
xcopy %TARGET_PATH%\* %DEPLOY%\ /s /e
rd /s /q %DEPLOY%\WEB-INF\lib
del /f /s /q %DEPLOY%\WEB-INF\classes\test.properties
del /f /s /q %DEPLOY%\WEB-INF\classes\log4j.properties

cd /d %DEPLOY%

jar -cvf test.war *

rd /s /q %DEPLOY%\META-INF
rd /s /q %DEPLOY%\WEB-INF
rd /s /q %DEPLOY%\static
del /s /q %DEPLOY%\index.jsp

pause
```

这个脚本用来去掉lib下依赖的jar包，test.proterties，log4j.properties,再重新压缩为war包，去掉lib下的jar包，是因为项目的依赖包太大了，传输时很费时间，而jar包在部署过后很可能不会变动。
 test.properties与log4j.properties 为项目的配置文件，开发环境与服务器的配置肯定是不一样的，因此不能直接覆盖掉，而是全量部署时放在项目上，以后增量部署的时候不覆盖即可。

1.本地install为test.war，如果为增量部署还要执行一下deploy脚本。
2.把deploy过的war包通过Filezilla等传输到服务器指定的部署目录，我们这里服务器的下项目部署文件夹为/root/deploy，根据项目名创建一个文件夹test，因此我们的test项目的部署路径为/root/deploy/test，文件夹下有我们的test.war文件。
3.在/root/deploy/test文件夹下，会创建一个`deploy.sh` 脚本文件，与客户端的deploy.bat 相呼应。
```bash
#!/bin/sh
deploy_file_dir=/root/deploy/test
deploy_dir=/opt/web/test
cd ${deploy_dir}/deploy
rm -rf ./*
jar xvf ${deploy_file_dir}/test.war
cp -rf ./* ${deploy_dir}/webapps/
cd ../
sh stop.sh
sh run.sh
```

查看脚本内容，我们可以得知deploy.sh 脚本的功能
1.由于客户端传送过来的可能不会是全量test.war包，因此不能简单的把test.war包直接放到、opt/web/test/webapps目录下，这样会覆盖掉服务器的项目配置，也会导致部分文件丢失。只能解压test.war包 把全部内容复制到webapps下，存在就覆盖，而不存在的也不会删掉webapps下存在的。
2.执行stop.sh 停止test项目，因为test项目之前可能已经启动了。
3.执行run.sh 启动项目。


到此为止，我们已经用了脚本代替一部分手动操作，但还是不够完善，比如项目实例还是要自己手动搭建，启动脚本也要手动编写，下面提供了一键部署tomcat实例项目环境的脚本`build.sh`，最大化利用脚本，减少手动操作。(这里没有添加Nginx配置，所以有关Nginx的配置还要手动添加，并且重新加载！)
```bash
#!/bin/sh
#一键部署Tomcat实例项目环境

#项目名称
PROJECT_NAME=www.dongxiaoxia.xyz
#tomcat关闭端口
SERVER_PORT=8006
#Http监听端口
HTTP_PORT=8080
##################以上三个参数需要修改############################
#项目基础路径
PROJECT_BASE_PATH=/opt/web/
#日志基础路径
LOG_BASE_PATH=/opt/log/
#部署基础路径
DEPLOY_BASE_PATH=/root/deploy/
#Java安装路径
JAVA_HOME=/opt/soft/jdk1.8.0_11/
#Tomcat安装路径
CATALINA_HOME=/opt/soft/apache-tomcat-7.0.73
#Tomcat活跃配置路径,也就是项目部署路径
CATALINA_BASE=$PROJECT_BASE_PATH$PROJECT_NAME
#tomcat server.xml配置文件路径
SERVER_XML=$CATALINA_BASE/conf/server.xml
#########################################################################################
echo 'Java安装路径:'$JAVA_HOME
echo 'Tomcat安装路径:'$CATALINA_HOME
echo 'Tomcat项目实例路径:'$CATALINA_BASE
echo '项目war包部署上传路径:'$DEPLOY_BASE_PATH$PROJECT_NAME
echo 'tomcat server.xml配置文件路径:'$SERVER_XML
echo '项目日志路径:'$LOG_BASE_PATH$PROJECT_NAME
echo 'tomcat关闭端口:'$SERVER_PORT
echo 'Http监听端口:'$HTTP_PORT

#################创建tomcat实例 修改server.xml配置##########################

#复制tomcat实例所需文件夹
echo '复制tomcat实例所需文件夹'

if [ ! -d "$CATALINA_BASE" ]; then
mkdir -p "$CATALINA_BASE"
fi

cd $CATALINA_HOME
cp -r conf $CATALINA_BASE
mkdir $CATALINA_BASE/webapps
mkdir $CATALINA_BASE/work
mkdir $CATALINA_BASE/logs

echo '修改server.xml配置'
# 修改tomcat关闭端口
sed -i 22s@8005@${SERVER_PORT}@ $SERVER_XML
# 修改Http监听端口
sed -i 71s@8080@${HTTP_PORT}@ $SERVER_XML
# 统一web应用程序的路径
sed -i '125s@appBase="webapps"@appBase="'$CATALINA_BASE'/webapps"@' $SERVER_XML
# 关闭自动部署
#sed -i '126s@autoDeploy="true"@autoDeploy="false"@' $SERVER_XML
sed -i '141a<Host name="localhost"  appBase="'$CATALINA_BASE'/webapps" autoDeploy="false" deployOnStartup="false">\n     <Context path="" docBase="'$CATALINA_BASE'/webapps" crossContext="true" allowLinking="true" reloadable="false">\n         <JarScanner scanAllDirectories="true" />\n         <Valve className="org.apache.catalina.valves.AccessLogValve" directory="'$CATALINA_BASE'/logs/" prefix="gh_access_log." suffix=".log" pattern="%a %A %t %S %b %r %s %{Referer}i %{User-Agent}i %D" resolveHosts="false" />\n     </Context>\n</Host>
' $SERVER_XML

sed -i '125,141d' $SERVER_XML

#######################创建run.sh脚本文件###################################
echo '创建run.sh脚本文件'
cd $CATALINA_BASE
touch run.sh
echo '#!/bin/sh' >> run.sh
echo 'TOMCAT_HOME='$CATALINA_HOME >> run.sh
#echo 'CATALINA_BASE='$CATALINA_BASE >> run.sh
echo 'CATALINA_BASE=$(cd "$(dirname $0)"; pwd)' >> run.sh
echo 'CATALINA_PID=$CATALINA_BASE/tomcat.pid' >> run.sh
echo 'JAVA_HOME='$JAVA_HOME >> run.sh
echo 'CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib' >> run.sh
echo 'LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/apr/lib' >> run.sh
echo 'PATH=$PATH:$JAVA_HOME/bin' >> run.sh
echo '' >> run.sh
echo '#路径为【/opt/web/xxx】时，项目名则为【xxx】' >> run.sh
echo "PROJECT_NAME=\`echo \$CATALINA_BASE | awk -F \"/\" '{print \$NF}'\`" >> run.sh
echo '' >> run.sh
echo '#检测temp目录是否存在，不存在则创建' >> run.sh
echo 'if [ ! -d "$CATALINA_BASE/temp" ]; then' >> run.sh
echo '    mkdir "$CATALINA_BASE/temp"' >> run.sh
echo 'fi' >> run.sh
echo '' >> run.sh
echo '#日志目录名' >> run.sh
echo 'PROJECT_LOG='$LOG_BASE_PATH'$PROJECT_NAME/catalina.out' >> run.sh
echo '' >> run.sh
echo 'JAVA_OPTS="-server -Xms32m -Xmx512m -Xmn32m -Xss1024K -XX:PermSize=32m -XX:MaxPermSize=64m -XX:ParallelGCThreads=32 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:SurvivorRatio=4 -XX:MaxTenuringThreshold=10 -XX:CMSInitiatingOccupancyFraction=80"' >> run.sh
echo '' >> run.sh
echo 'export TOMCAT_HOME CATALINA_BASE CATALINA_PID JAVA_HOME CLASSPATH PATH JAVA_OPTS PROJECT_NAME PROJECT_LOG LD_LIBRARY_PATH' >> run.sh
echo '' >> run.sh
echo 'proj_dir=$CATALINA_BASE' >> run.sh
echo 'if [ -n "$proj_dir" ]' >> run.sh
echo 'then' >> run.sh
echo "        ps aux | grep \"\$proj_dir\" | grep -v \"grep\" | grep -v \"\$CATALINA_BASE/\(start.sh\|stop.sh\|restart.sh\|logs/catalina.out\)\" > /dev/null" >> run.sh
echo '        status=$?' >> run.sh
echo '' >> run.sh
echo '        if  [ $status -eq 0 ] && [ -s $CATALINA_PID  ]' >> run.sh
echo '        then' >> run.sh
echo '                echo "The $proj_dir tomcat is running and the pid_file is exist !"' >> run.sh
echo '                exit 1' >> run.sh
echo '        elif [ $status -eq 0 ] || [ -s $CATALINA_PID  ]' >> run.sh
echo '        then' >> run.sh
echo '                echo "kill one"' >> run.sh
echo "                ps aux | grep \"\$proj_dir\" | grep -v \"grep\" | grep -v \"\$CATALINA_BASE/\(start.sh\|stop.sh\|restart.sh\|logs/catalina.out\)\" | awk '{print \$2}' | xargs -i kill -9 {}" >> run.sh
echo '                > $CATALINA_PID' >> run.sh
echo '                $TOMCAT_HOME/bin/catalina.sh start' >> run.sh
echo '        else' >> run.sh
echo '                $TOMCAT_HOME/bin/catalina.sh start' >> run.sh
echo '        fi' >> run.sh
echo '' >> run.sh
echo 'else' >> run.sh
echo '        echo "start failt!!!"' >> run.sh
echo '        exit 1' >> run.sh
echo 'fi' >> run.sh

#######################创建stop.sh脚本文件###################################
echo '创建stop.sh脚本文件'
cd $CATALINA_BASE
touch stop.sh

echo '#!/bin/sh' >> stop.sh
echo 'TOMCAT_HOME='$CATALINA_HOME >> stop.sh
echo '#CATALINA_BASE='$CATALINA_BASE >> stop.sh
echo 'CATALINA_BASE=$(cd "$(dirname $0)"; pwd)' >> stop.sh
echo 'CATALINA_PID=$CATALINA_BASE/tomcat.pid' >> stop.sh
echo 'JAVA_HOME='$JAVA_HOME >> stop.sh
echo 'CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib' >> stop.sh
echo 'PATH=$PATH:$JAVA_HOME/bin' >> stop.sh
echo '' >> stop.sh
echo '#路径为【/opt/web/xxx】时，项目名则为【xxx】' >> stop.sh
echo "PROJECT_NAME=\`echo \$CATALINA_BASE | awk -F \"/\" '{print \$NF}'\`" >> stop.sh
echo 'echo $PROJECT_NAME' >> stop.sh
echo '' >> stop.sh
echo '#日志目录名' >> stop.sh
echo 'PROJECT_LOG='$LOG_BASE_PATH'$PROJECT_NAME/' >> stop.sh
echo '' >> stop.sh
echo 'export TOMCAT_HOME CATALINA_BASE CATALINA_PID JAVA_HOME CLASSPATH PATH JAVA_OPTS PROJECT_NAME PROJECT_LOG' >> stop.sh
echo '' >> stop.sh
echo 'proj_dir=$CATALINA_BASE' >> stop.sh
echo '$TOMCAT_HOME/bin/catalina.sh stop' >> stop.sh
echo 'sleep 5' >> stop.sh
echo 'if [ -n "$proj_dir" ]' >> stop.sh
echo 'then' >> stop.sh
echo "        ps aux | grep \"\$proj_dir\" | grep -v \"grep\" | grep -v \"\$CATALINA_BASE/\(start.sh\|stop.sh\|restart.sh\|logs/catalina.out\)\" > /dev/null" >> stop.sh
echo '        status=$?' >> stop.sh
echo '        if [ $status -eq 0 ] || [ -s $CATALINA_PID  ]' >> stop.sh
echo '        then' >> stop.sh
echo '                echo "kill it"' >> stop.sh
echo "                ps aux | grep \"\$proj_dir\" | grep -v \"grep\" | grep -v \"\$CATALINA_BASE/\(start.sh\|stop.sh\|restart.sh\|logs/catalina.out\)\" |  awk '{print \$2}' | xargs -i kill -9 {}" >> stop.sh
echo '                > $CATALINA_PID' >> stop.sh
echo '        fi' >> stop.sh
echo 'else' >> stop.sh
echo '        echo "stop failt!!!"' >> stop.sh
echo '        exit 1' >> stop.sh
echo 'fi' >> stop.sh

#######################创建debug.sh脚本文件###################################
echo '创建debug.sh脚本文件'
cd $CATALINA_BASE
touch debug.sh

echo '#!/bin/sh' >> debug.sh
echo 'TOMCAT_HOME='$CATALINA_HOME >> debug.sh
echo 'CATALINA_BASE=$(cd "$(dirname $0)"; pwd)' >> debug.sh
echo 'CATALINA_PID=$CATALINA_BASE/tomcat.pid' >> debug.sh
echo 'JAVA_HOME='$JAVA_HOME >> debug.sh
echo 'CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib' >> debug.sh
echo 'LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/apr/lib' >> debug.sh
echo 'PATH=$PATH:$JAVA_HOME/bin' >> debug.sh
echo '' >> debug.sh
echo '#路径为【/opt/web/xxx】时，项目名则为【xxx】' >> debug.sh
echo "PROJECT_NAME=\`echo \$CATALINA_BASE | awk -F \"/\" '{print \$NF}'\`" >> debug.sh
echo '' >> debug.sh
echo '#日志目录名' >> debug.sh
echo 'PROJECT_LOG='$LOG_BASE_PATH'$PROJECT_NAME/catalina.out' >> debug.sh
echo '' >> debug.sh
echo 'JAVA_OPTS="-server -Xms32m -Xmx512m -Xmn32m -Xss1024K -XX:PermSize=32m -XX:MaxPermSize=64m -XX:ParallelGCThreads=32 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:SurvivorRatio=4 -XX:MaxTenuringThreshold=10 -XX:CMSInitiatingOccupancyFraction=80"' >> debug.sh
echo '' >> debug.sh
echo 'export TOMCAT_HOME CATALINA_BASE CATALINA_PID JAVA_HOME CLASSPATH PATH JAVA_OPTS PROJECT_NAME PROJECT_LOG LD_LIBRARY_PATH' >> debug.sh
echo '' >> debug.sh
echo 'proj_dir=$CATALINA_BASE' >> debug.sh
echo 'if [[ -n "$proj_dir" ]]' >> debug.sh
echo 'then' >> debug.sh
echo '        ps aux | grep "$proj_dir" | grep -v "grep" | grep -v "$CATALINA_BASE/\(start.sh\|stop.sh\|restart.sh\|logs/catalina.out\)" > /dev/null' >> debug.sh
echo '        status=$?' >> debug.sh
echo '' >> debug.sh
echo '        if  [[ $status -eq 0 ]] && [[ -s $CATALINA_PID  ]]' >> debug.sh
echo '        then' >> debug.sh
echo '                echo "The $proj_dir tomcat is running and the pid_file is exist !"' >> debug.sh
echo '                exit 1' >> debug.sh
echo '        elif [[ $status -eq 0 ]] || [[ -s $CATALINA_PID  ]]' >> debug.sh
echo '        then' >> debug.sh
echo '                echo "kill one"' >> debug.sh
echo "                ps aux | grep \"\$proj_dir\" | grep -v \"grep\" | grep -v \"\$CATALINA_BASE/\(start.sh\|stop.sh\|restart.sh\|logs/catalina.out\)\" | awk '{print \$2}' | xargs -i kill -9 {}" >> debug.sh
echo '                > $CATALINA_PID' >> debug.sh
echo '                $TOMCAT_HOME/bin/catalina.sh jpda start' >> debug.sh
echo '        else' >> debug.sh
echo '                $TOMCAT_HOME/bin/catalina.sh jpda start' >> debug.sh
echo '        fi' >> debug.sh
echo 'else' >> debug.sh
echo '        echo "start failt!!!"' >> debug.sh
echo '        exit 1' >> debug.sh
echo 'fi' >> debug.sh

#######################创建restart.sh脚本文件###################################
echo '创建restart.sh脚本文件'
cd $CATALINA_BASE
touch restart.sh

echo '#!/bin/sh' >> restart.sh
echo 'TOMCAT_HOME='$CATALINA_HOME >> restart.sh
echo 'CATALINA_BASE=$(cd "$(dirname $0)"; pwd)' >> restart.sh
echo 'CATALINA_PID=$CATALINA_BASE/tomcat.pid' >> restart.sh
echo 'JAVA_HOME=/opt/soft/jdk/1.8.11/jdk1.8.0_11' >> restart.sh
echo 'CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib' >> restart.sh
echo 'PATH=$PATH:$JAVA_HOME/bin' >> restart.sh
echo 'JAVA_OPTS="-server -Xms1g -Xmx1g -Xmn512m -Xss1024K -XX:PermSize=256m -XX:MaxPermSize=512m -XX:ParallelGCThreads=8 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:SurvivorRatio=4 -XX:MaxTenuringThreshold=10 -XX:CMSInitiatingOccupancyFraction=80"' >> restart.sh
echo '' >> restart.sh
echo '#路径为【/opt/web/xxx】时，项目名则为【xxx】' >> restart.sh
echo "PROJECT_NAME=\`echo \$CATALINA_BASE | awk -F \"/\" '{print \$NF}'\`" >> restart.sh
echo '' >> restart.sh
echo '#日志目录名' >> restart.sh
echo 'PROJECT_LOG='$LOG_BASE_PATH'$PROJECT_NAME/catalina.out' >> restart.sh
echo '' >> restart.sh
echo 'export TOMCAT_HOME CATALINA_BASE CATALINA_PID JAVA_HOME CLASSPATH PATH JAVA_OPTS PROJECT_NAME PROJECT_LOG' >> restart.sh
echo '' >> restart.sh
echo 'sh $CATALINA_BASE/stop.sh' >> restart.sh
echo 'sh $CATALINA_BASE/start.sh' >> restart.sh
#######################创建deploy.sh脚本文件###################################
echo '创建deploy.sh脚本文件'

if [ ! -d "$DEPLOY_BASE_PATH$PROJECT_NAME" ]; then
  mkdir -p "$DEPLOY_BASE_PATH$PROJECT_NAME"
fi

cd $DEPLOY_BASE_PATH$PROJECT_NAME
touch deploy.sh

echo '#!/bin/sh' >> deploy.sh
echo "deploy_file_dir=$DEPLOY_BASE_PATH$PROJECT_NAME" >> deploy.sh
echo "deploy_dir=$CATALINA_BASE" >> deploy.sh
echo "if [ ! -d "\${deploy_dir}/deploy" ]; then" >> deploy.sh
echo "  mkdir -p "\${deploy_dir}/deploy"" >> deploy.sh
echo "fi" >> deploy.sh
echo 'cd ${deploy_dir}/deploy' >> deploy.sh
echo 'rm -rf ./*' >> deploy.sh
echo 'jar xvf ${deploy_file_dir}/'$PROJECT_NAME'.war' >> deploy.sh
echo 'cp -rf ./* ${deploy_dir}/webapps/' >> deploy.sh
echo 'cd ../' >> deploy.sh
echo 'sh stop.sh' >> deploy.sh
echo 'sh run.sh' >> deploy.sh
#########################################################################################
echo 'tomcat项目实例创建成功!'
echo "请把war包放置到$DEPLOY_BASE_PATH$PROJECT_NAME目录下，执行$DEPLOY_BASE_PATH$PROJECT_NAME/deploy脚本启动项目."
echo "或者直接解压war包到$CATALINA_BASE/webapps下。执行$CATALINA_BASE/run脚本启动项目."
```

## 6. 部署总结
### 为什么
本篇文章都是为了解决文章开头的问题及后续问题：本地开发完项目后如何简单部署到服务器？

### 怎么做
1. 服务器项目部署规范化，相关软件规范化，统一部署，集中管理。
2. 用脚本替代手动操作。
3. 部署文档清晰易懂。

### 怎么样
**优点**
1. tomcat多实例部署，启动、关闭一个项目站点不影响其他站点，没有耦合。
2. 对添加开放，对删除关闭。
要添加一个新的项目站点，只需简单的执行一下脚本就能够实现，不会影响其他站点配置。
3. 第一次部署及后续增量部署都极其简单、容易操作、全程脚本化。
4. 整个服务器规范化，各个项目部署路径、相关软件路径、日志路径清晰明了。
5. 项目日志集中管理，方便后续统计分析。 

**劣势**
 1. 要求部署人员具备一定的脚本语法基础，能够理解相关的脚本原理。
 2. 脚本的创建是直接添加内容，如果之前已经存在相关的文件，没有采取替换而是追加，因此要确定相关路径上的文件是否正确。

### 几句话说明白部署

**部署环境整体规划**
1.  项目目录 /opt/web
2.  日志目录 /opt/log
3.  jdk/tomcat/nginx等目录 安装环境 /opt/soft
4.  项目应用war包上传到服务器的目录 /root/deploy

假设现在有3个项目 test test1 test2
```bash
ubuntu# ls /opt/web /root/deploy/ /opt/soft /opt/log
/opt/log:
test  test1  test2

/opt/soft:
apache-tomcat-7.0.73  include	   lib	  nginx-1.6.2	      openssl-1.1.0c	     pcre-8.39	       share  zlib-1.2.8
bin		      jdk1.8.0_11  nginx  nginx-1.6.2.tar.gz  openssl-1.1.0c.tar.gz  pcre-8.39.tar.gz  ssl    zlib128.zip

/opt/web:
build.sh  test	test1  test2

/root/deploy/:
test  test1  test2
ubuntu# 
```

**全量部署(第一次部署)**
1. **执行`build.sh`脚本一键部署项目**
2. 配置并重新加载Nginx代理配置
3. 开发环境install打包war包传输到服务器deploy目录
4. 执行`deploy.sh`脚本，打开项目日志查看启动情况
                          
**增量部署**
1. 开发环境install打包war包,**执行开发环境`deploy.sh(mac)/deploy.bat(windows)`脚本**，传输到服务器deploy目录 
4. 执行`deploy.sh`脚本，打开项目日志查看启动情况
