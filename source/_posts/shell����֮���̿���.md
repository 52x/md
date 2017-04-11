title: shell命令之流程控制
date: 2016-05-06 17:07:00
tags: [linux,shell]
---

## 概述
在编写linux shell命令完成一些自动化的事情的时候，少不了关于流程的控制和条件判断，这里简单介绍几种简单的控制命令。

## 双and （&&）
```js
cd ../ && mocha
```

`&&` 命令表示，第二条命令只有在第一条命令执行成功后执行。

## 双竖线（||）
```js
cd ../ || mocha
```
`||` 命令表示，第二条命令只有在第一条命令执行不成功时执行。

## 分号（;）

```js
cp /tmp/t2 /tmp/t2.bak; echo "hello world"
```
顺序执行多条命令，当;号前的命令执行完（不管是否执行成功），才执行;后的命令。

## sleep、
```js
 sleep 20s
```
进程休眠一段时间，然后再进行

## 单and （&）
```js
npm start & cd ../
```
与`;` 类似，但不代表第一个命令执行结束

## 输入中端
```js
echo -e '\003'
```
模拟输出`Ctrl+C`

## 演示示例

```js
#步骤：
# 检查mocha是否已经安装，没有安装则安装mocha
# git clone项目所有源文件
# 切换到项目目录，安装pkg
# 进入到静态文件目录里，fis3构建静态文件
# 切换到项目根目录，启动端口监听
# 端口监听20s,然后模拟输出Ctrl+c,主要是确保bable编译完成
# 运行mocha单元测试
npm ls mocha || npm install mocha -g  && git clone https://github.com/zuoyanart/pizzaManage && cd pizzaManage && cnpm install && cd fis-static && fis3 release -d ../ && cd ../ && npm start & sleep 20s;echo -e '\003';cd app;mocha;

```