---
title: HEXO中运行自己代码的方法
date: 2017-02-15 13:07:02
updated: 2017-02-15 13:07:02
tags: [hexo]
categories: [Hexo]
keywords: [hexo,scripts,plugin]
---

# 引子

最近学习使用hexo，修改模板过程中，总有些想法想要自己做，所以想起来看看他们插件是怎么写的。

找了两个小的看了，没有发现怎么调用的（大雾）

找了找资料，发现只要是在node_modules中的 以hexo-开头的，就会自动的被调用。

![以hexo-开头的模块](2017-02-15-hexo-run-self-code\startwithhexo.png)

# 两个办法

还有其他办法么？查阅API发现，想扩展代码，有两个办法，即在 Hexo 中有两种形式的插件：

**脚本**（Scripts）和**插件**（Packages） 这里沿用了[官网中文翻译](https://hexo.io/zh-cn/docs/plugins.html)的叫法。

## 脚本（Scripts）

按官网建议，如果代码简单，建议编写脚本，只要放到scripts文件夹即可。
官网没有举例，实际操作一下。
在博客项目的根目录，新建一个文件夹scripts。
放一个文件index.js，文件名是我随手起得index，不固定。

内容：

```javascript
'use strict';
const chalk = require('chalk');
var hexo = hexo != undefined ? hexo : {};
console.log("Scripts test,log no.1", __dirname)

hexo.on("generateBefore", function(){
    console.log(chalk.green("Scripts generateBefore.", __dirname))
});
hexo.on("generateAfter", function(){
    console.log(chalk.green("Scripts generateAfter.", __dirname))
});
//注意， 这里不要直接hexo.on，检测不到，我看了源码才知道，这个事件依赖于box
hexo.source.on("processAfter", function(file){
    console.log(file);
    console.log(chalk.green("Scripts test,processAfter.", __dirname))
});

hexo.source.on("processBefore", function(file){
    console.log(file);
    console.log(chalk.green("Scripts test,processBefore.", __dirname))
});
hexo.theme.on("processAfter", function(file){
    console.log(file);
    console.log(chalk.cyan("Scripts test,processAfter.", __dirname))
});

hexo.theme.on("processBefore", function(file){
    console.log(file);
    console.log(chalk.cyan("Scripts test,processBefore.", __dirname))
});

```

实际输出类似于：

![输出情况](2017-02-15-hexo-run-self-code\scriptsoutput.png)

## 插件（Packages）

效果和代码放在scripts中是一样的。不过是先执行的scripts中的，后执行的packages中的。

### 注意事项

- 插件是要放在`node_module`s文件夹中的
- 文件夹必须以`hexo-`开头
- 需要符合package的定义，也就是至少要有一个`package.json`，一个执行文件
- `package.json` 中至少要包含 `name`, `version`, `main` 属性
- 感觉通用可以[发布](https://hexo.io/zh-cn/docs/plugins.html#发布)给更多人使用。



