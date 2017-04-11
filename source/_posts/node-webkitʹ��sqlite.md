title: node-webkit中使用sqlite3
date: 2014-01-22 15:29:14
tags: [noedjs,sqlite,前端]
---

sqlite3的官方文档提到：nodejs和node-webkit的ABI不同，所以默认的安装方式：

```js
npm install sqlite3
```
安装的sqlite3是无法使用的，需要重新编译。

#### 编译方法：

##### windows下：

配置编译环境：

> 安装python，据说nodejs的编译建议使用的的版本是2.6+，但不推荐3.0，所以本人也就不去深究了，本人使用的是2.7的版本

> VC++编译器，包含在VS2010中（VC++ 2010 Express亦可）

> 安装nw-gyp ，使用命令：npm install nw-gyp -g

##### 编译步骤

1，安装sqlte3，使用命令：npm install sqlite3

2，在cmd中切换到sqlite3所在的目录，cd ./node_module/sqlite3

3，输入命令：nw-gyp rebuild --target=0.8.4  (0.8.4为node-webkit的版本号)

4，把./build/Release/node-sqlite3.node 复制到 ./lib/binding/Release/node-v11-win32-ia32/ 下，如果文件夹不存在请手动创建

注意：本人在编译的时候遇到Python不是内部命令的错误，设置了path也不行，故本人在编译的时候多加了一个命令：set PATH=%PATH%;C:\Python27，把此语句放在第三部执行即可

##### MAC的编译：

省去 “windows编译中配置编译环境”中的1，2步外，剩下的都一致


注：真爱生命，远离windows

 