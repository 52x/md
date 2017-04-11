title: 'golang 的编译安装以及supervisord部署'
date: 2015-12-24 13:14:33
tags: golang
---

#### go的编译
go的编译相对很简单，只需要一个命令即可完成,进入项目开发路径，输入
```js
go install <appName>
```
即可在bin文件夹下生成可执行文件 appName。此文件可直接运行。
备注：生成的可执行文件只包含go的程序文件，配置文件，views文件等需要拷贝过来，这样才能组成一个完整的运行程序。

#### go的部署
###### supervisord安装
```js
yum install setuptools //先安装工具
yum install supervisor
```
##### 修改配置文件
```js
vi /etc/supervisord.conf
```
将最后一行的代码改为
```js
files = /etc/supervisord.conf.d/*.conf
```
##### 新增文件并编写配置
```js
vi /etc/supervisord.conf.d/appname.conf
```
```js
[program:appname]
user=root
command=/data/host/liudu/go/bin/appname
autostart=true
startsecs=10
stdout_logfile=/data/host/liudu/golog/appname.log //此文件需手动创建
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=10
stdout_capture_maxbytes=1MB
stderr_logfile=/data/host/liudu/golog/appname.log
stderr_logfile_maxbytes=1MB
stderr_logfile_backups=10
stderr_capture_maxbytes=1MB
stopsignal=INT
[supervisord]                        
```

```js
command：表示运行的命令，填入完整的路径即可。
autostart：表示是否跟随supervisor一起启动。
autorestart：如果该程序挂了，是否重新启动。
stdout_logfile：终端标准输出重定向文件。
stderr_logfile：终端错误输出重定向文件。
```

##### 启动服务
```js
supervisord -c /etc/supervisord.conf.d/renmaiApi/conf
```

##### supervisord 管理
* supervisord，初始启动Supervisord，启动、管理配置中设置的进程。
* supervisorctl stop programxxx，停止某一个进程(programxxx)，programxxx为[program:appname]里配置的值，这个示例就是appname。
* supervisorctl start programxxx，启动某个进程
* supervisorctl restart programxxx，重启某个进程
* supervisorctl stop groupworker: ，重启所有属于名为groupworker这个分组的进程(start,restart同理)
* supervisorctl stop all，停止全部进程，注：start、restart、stop都不会载入最新的配置文件。
* supervisorctl reload，载入最新的配置文件，停止原有进程并按新的配置启动、管理所有进程。
* supervisorctl update，根据最新的配置文件，启动新配置或有改动的进程，配置没有改动的进程不会受影响而重启。
> 注意：显示用stop停止掉的进程，用reload或者update都不会自动重启。


##### 错误记录
```js
Error: Another program is already listening on a port that one of our HTTP servers is configured to use.  Shut this program down first before starting supervisord.
For help, use /usr/bin/supervisord -h
```
解决办法

```js
find / -name supervisor.sock
unlink /***/supervisor.sock

```
