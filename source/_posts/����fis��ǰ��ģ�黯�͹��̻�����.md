title: 基于fis的前端模块化和工程化方案
date: 2016-01-07 18:20:59
tags: [fis,前端,前端工程化,前端模块化]
---



## 前端构建工具
面对日益复杂的前端环境以及前端技术、node技术的高速发展，前端的开发也越来越工程化，体系化，也就出现了前端自动化构建维护工具。他们完成的任务目标基本是：

> * js，css,图片的自动压缩合并（图片也即是是自动sprite）
> * js，css,图片自动添加域名（静态文件的分布式部署）
> * js，css,图片自动添加md5或版本号（缓存管理）
> * 自动监听文件变化 （自动编译，方便调试）
> * sass/less/coffee等的自动编译 （方便扩展各种插件）
> * 支持amd/cmd的模块开发，可自动文件依赖 （js模块化开发）
> * 可以部署文件 （自动部署）
> * 网页自动刷新 （方便调试）
> * 实时编译 （方便调试）

成熟的前端构建工具也有很多，比如：Gulp.js，Grunt，webpark等等，其他构建工具本人使用不多，本文主要是对fis的前端工程化和模块化的使用做详细介绍


## 关于fis
fis是百度研发的一套前端构建工具，拥有丰富的脚手架和组件仓库。因为fis是本人接触的最早的前端构建工具，所以一直沿用到了至今。

		注： 本文针对的版本号是fis3

## fis的前端工程化和模块化基础插件
### 1.自动打包合并插件
#### fis配置
```js

//打包与css sprite基础配置
fis.match('::packager', {
    // npm install -g fis3-postpackager-loader
    postpackager: fis.plugin('loader', {
        resourceType: 'mod',
        obtainScript: true,
        allInOne: true,
        useInlineMap: true, // 资源映射表内嵌
    }),
    packager: fis.plugin('map', {//常用脚本和css的合并，此不参与到页面的自动合并
      useTrack: false,
      'pkg/base.js': ['/modules/jquery/*.js', '/modules/layer/*.js', '/modules/pizzalayer/*.js', '/modules/pizzatools/*.js'],
      'pkg/base-a.js': ['/widget/globle/*.js', '/modules/pizzaui/pizza.ui.js', '/site/common/common.js'],
      'pkg/base.css': ['/css/pizza.css', '/css/iconfont.css']
    }),
    spriter: fis.plugin('csssprites', {//css的雪碧合并配置
        layout: 'matrix',
        margin: '15'
    })
});

```
#### 作用
将html中分散的静态资源进行自动合并打包
#### 应用举例：
原始文件
```js
  <link rel="stylesheet" type="text/css" href="/css/pizza.css">
  <link rel="stylesheet" type="text/css" href="/css/iconfont.css">
  <link rel="stylesheet" type="text/css" href="/site/index/index/index.css">
```
转换后
```js
   <link type="text/css" rel="stylesheet" href="/pkg/base.css">
   <link rel="stylesheet" href="/pkg/view/home/article_index.html_aio.css" />
```
可以看到fis会自动合并多个css到同一个文件里，这个合并不仅仅适用于css，也同样适用与js，并且将自动把css文件放入header头，js放在body结束前，有了这个功能也就具备的模块化开发的大前提
### fis的include功能
#### fis配置
默认支持，无须插件
#### 作用
大家在使用模板引擎的时候，肯定是少不了include功能的， 即公共部分的文件引用。fis同样支持这个功能，而且借助与自动打包功能，include功能的作用也被放大的很多.(fis支持多级include)
#### 应用举例
##### 主模板文件
```js
<!DOCTYPE html>
<html>

<head>
  <meta name="description" content="">
  <meta name="keywords" content="">
  <link rel="stylesheet" type="text/css" href="/css/mcren.css">
  <link rel="stylesheet" type="text/css" href="/site/index/index/index.css">
  <script type="text/javascript" src="/lib/mod.js"></script>
</head>

<body>
  <link rel="import" href="/widget/header-small/header.html?__inline">
```
##### 次模版文件（header-small/header.html）
```js
<link rel="stylesheet" type="text/css" href="./header.css">
<div class="header">
	<div>
		<link rel="import" href="../loginstate/loginstate.html?__inline"><h1><a href="/">主页</a></h1>
	</div>
</div>
<script type="text/javascript">
    var jsfunc = require('jsfunc/jsfunc');
    jsfunc.init();
</script>
```
##### 转换后
```js

<!DOCTYPE html>
<html>

<head>
    <meta name="description" content="">
    <meta name="keywords" content="">
    <link type="text/css" rel="stylesheet" href="/pkg/mcren_7636e.css">
</head>

<body>
    <div class="header">
	  <div>
		  <div class="loginstate">
	        <a href="/space">左盐</a> | <a href="javascript:void(0);">退出</a><img src="member.jpg" id="avatar" />
          </div>
           <h1><a href="/">名场</a></h1>
	   </div>
    </div>

	<!--省略的html代码-->

<script type="text/javascript" src="/lib/mod.js"></script>
<script type="text/javascript">
    var jsfunc = require('jsfunc/jsfunc');
    jsfunc.init();
</script>
</body>

```
#### 说明
可以看到，经过fis处理后，本来应该分散的css被组合放进了head头里，js被放在了body结束前，模板的html代码也正常引入了进去。

> 另外大家可以看到我模块的划分，header模块的html是引入css的，也就是说模块与模块之间以及模块和主体页面之间的css都是独立的，这样充分解耦，可以有效的解决css的维护问题，

### ejs分析能力
#### fis配置
```js
//npm install fis-parser-ejs
fis.match("**/*.ejs", {
        parser: fis.plugin('ejs'),
        isJsLike: true,
        release: false
    })
```
#### 作用
大家知道，写js打印html字符串到页面的时候，如果在js里面串接html字符串是一种很难受的体验，所以fis也就有了这个插件，这个插件可以给js方法添加引用ejs模板的能力，这里ejs的使用方法和原生的完全兼容
#### 应用举例
##### js函数
```js
function ejsEx() {
	var tpl = __inline('comment.ejs');
	var s = tpl({
		title: "我的ejs模板"
	});
	console.log(s);
}
```
#### ejs模板
```js
<h1><%= title%></h1>
```
#### 输出
```js
<h1>我的ejs模板</h1>
```
### 母版页支持
#### fis配置
```js
//npm install fis-parser-ejs
//page下面的页面发布时去掉page文件夹
//这里其实是使用了swig的母版页功能，然后通过fis编译成html页面的
fis.match(/^\/view\/(.*)$/i, {
        parser: fis.plugin('swigt'),
        useCache: false,
        release: '/$&'
    })
```
#### 作用
母版页的使用场景在后台管理页面里特别常见，在后台里总是头尾和左侧管理菜单栏是不变的，其他位置在变化，如果有母版页功能支持，那么开发这些页面就变得轻而易举
#### 应用举例
##### html母板
```js

<!DOCTYPE html>
<html>

<head>
    <link rel="import" href="/widget/meta/meta.html?__inline">
    <link rel="stylesheet" href="/site/common/common.css">
    <link rel="stylesheet" href="/site/home/base.css" >
    {% block css %}{% endblock %}
    <title>{% block title %}{% endblock %}</title>
</head>
<body>
	<div id="header">
		<h1><i class="icon-pizza"></i>Pizza Admin</h1>
	</div>
	<div id="user-nav">
		<ul>
			<li><i class="icon-person"></i><span id="userinfo"></span></li>
			<li><a href="javascript:void(0);" id="loginout"><i class="icon-loginout"></i>退出</a></li>
		</ul>
	</div>
	<div id="search">
		<input type="search" id="searchkw">
	</div>
	<div id="sidebar">
		<ul>
			<li><a href="/" class=""><i class="icon-home"></i>主页</a></li>
			<li>
				<a href="#" class="submenu"><i class="icon-article"></i>信息管理</a>
				<ul>
					<li><a href="/tree">节点管理</a></li>
					<li><a href="/article">文章管理</a></li>
					<li><a href="/module">模块管理</a></li>
				</ul>
			</li>
			<li>
        <a href="#" class="submenu"><i class="icon-setting"></i>系统管理</a>
        <ul>
					<li><a href="/user">管理员管理</a></li>
					<!-- <li><a href="/article">系统设置</a></li>
					<li><a href="/module"></a></li> -->
				</ul>
      </li>
		</ul>
	</div>
	<div id="main">
		{% block content %}{%endblock%}
	</div>
	<div class="row-fluid"></div>
	<script type="text/javascript">
  var globleConfig = require('globle/globle');
  globleConfig.init();
	var common = require('common/common');
	common.init();
	</script>
    {% block js %}{%endblock%}
</body>

</html>

```
##### html模板
```js
{% extends "../master/index.html" %} {% block css%} {% endblock %} {% block title %} 文章管理 {% endblock %}
<!---->
{% block content %}
<div class="menu">
  <label class="checkgroup">
    <input type="checkbox" id="checkall" name="checkall"><label for="checkall" class="check-all"></label>全选
  </label>|<a href="/home/user/edit">添加</a>|<em class="pass">冻结</em>|<em class="remove">删除</em>
</div>
<ul class="list" id="list">
</ul>
{% endblock %}
<!---->
{% block js %}
<script type="text/javascript">
  var user = require('home/user/user');
  user.init();
</script>
{% endblock %}

```
#### 生成
```js
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
<meta name="renderer" content="webkit">
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
<meta name="viewport" content="initial-scale=1, maximum-scale=1, user-scalable=no, minimal-ui">     
    <title> 文章管理 </title>

    <link rel="stylesheet" media="screen" charset="utf-8" href="/static/pkg/base.css" />
    <link rel="stylesheet" href="/static/pkg/view/home/user_index.html_aio.css" />
</head>
<body>
	<div id="header">
		<h1><i class="icon-pizza"></i>Pizza Admin</h1>
	</div>
	<div id="user-nav">
		<ul>
			<li><i class="icon-person"></i><span id="userinfo"></span></li>
			<li><a href="javascript:void(0);" id="loginout"><i class="icon-loginout"></i>退出</a></li>
		</ul>
	</div>
	<div id="search">
		<input type="search" id="searchkw">
	</div>
	<div id="sidebar">
		<ul>
			<li><a href="/" class=""><i class="icon-home"></i>主页</a></li>
			<li>
				<a href="#" class="submenu"><i class="icon-article"></i>信息管理</a>
				<ul>
					<li><a href="/tree">节点管理</a></li>
					<li><a href="/article">文章管理</a></li>
					<li><a href="/module">模块管理</a></li>
				</ul>
			</li>
			<li>
        <a href="#" class="submenu"><i class="icon-setting"></i>系统管理</a>
        <ul>
					<li><a href="/user">管理员管理</a></li>
					<!-- <li><a href="/article">系统设置</a></li>
					<li><a href="/module"></a></li> -->
				</ul>
      </li>
		</ul>
	</div>
	<div id="main">
		
<div class="menu">
  <label class="checkgroup">
    <input type="checkbox" id="checkall" name="checkall"><label for="checkall" class="check-all"></label>全选
  </label>|<a href="/home/user/edit">添加</a>|<em class="pass">冻结</em>|<em class="remove">删除</em>
</div>
<ul class="list" id="list">
</ul>

	</div>
	<div class="row-fluid"></div>

<script charset="utf-8" src="/static/lib/mod.js"></script>
<script type="text/javascript" src="/static/pkg/base.js"></script>
<script type="text/javascript" src="/static/pkg/base-a.js"></script>
<script type="text/javascript" src="/static/site/home/user/user.js"></script>
<script type="text/javascript">
  var globleConfig = require('globle/globle');
  globleConfig.init();
	var common = require('common/common');
	common.init();
	

  var user = require('home/user/user');
  user.init();
</script>
<script type="text/javascript" charset="utf-8" src="http://192.168.1.134:8132/livereload.js"></script></body>

</html>

```
## 目录结构
上面已经介绍了fis的模块化基础能力，现在开始实践fis的模块化开发能力。当然fis的amd/cmd，域名添加等等基础功能，这里没有叙述，您可以自行上官网查看学习
> 根目录
>> css //less生成的css文件
>> font //字体图标文件
>> img //公共图片
>> less 
>> lib //系统类库
>> modules //常用的module
>>> jquery
>>> laydate
>>> layer
>>> pizzalayer
>>> pizzatools //网站工具类
>>> pizzaui //ui组件
>>> store //本地化插件
>>> xss //xss过滤插件

>> site //非模块的css，js
>> views //网页模板文件 
>> widget //模块
>>> head //头部模块
>>> footer //底部模块
>>> nav //菜单模块

>> fis-conf.js //fis配置文件

因为我的技术构架是： 前端 + nodejs + rest api，所以使用这种目录结构，用户可根据自己的项目目录自由更改。

### 说明
#### modules目录
modules都是工具类，只包含js，
目录结构为：
```js
---根目录
----modules
-----jquery
------jquery-1.11.3.min.js
```
引用方式为：require('文件夹名字')
```js
require('jquery')
```

#### widget目录
widget目录都是网站模块，可包含js,css,html,图片
目录结构为：
```js
---根目录
----widget
-----loginstate
------loginstate.less
------loginstate.js
------loginstate.html
------loginstate.jpg
```
引用方式为：
```js
<link rel="import" href="/widget/loginstate/loginstate.html?__inline">
```
## 一个模块的代码示例
### loginstate.html代码
```js
<link rel="stylesheet" type="text/css" href="loginstate.css">
<div class="loginstate">
	<a href="/space">左盐</a> | <a href="javascript:void(0);">退出</a><img src="member.jpg" id="avatar" />
</div>
<script type="text/javascript">
	var loginstate = require('loginstate/loginstate');
	loginstate.init();
</script>
```
### loginstate.js代码
```js
var $  = require('jquery');
var tools = require('pizzatools');
require('pizzalayer');
var loginState = new function() {
	var _self = this;
	/**
	 * 打印登录状态
	 * @return {[type]} [description]
	 */
	_self.init = function() {
		var s = '';
		var id = tools.getCookie('id');
		if(id == '0' ) {//未登录
			s = '<a href="/login?f='+document.location.href+'" class="btn btn-primary">登录</a>&nbsp;&nbsp;&nbsp;&nbsp;<a href="/reg" class="btn btn-primary">注册</a>';
		}
		else {
			s = '<a href="/space">' + tools.getCookie('nickname') + '</a> | <a href="javascript:void(0);" onclick="loginstate.loginout();">退出</a><img src="' + tools.siteData.url.avatar + tools.getCookie('avatar') + '!v1" id="avatar" />';
		}
		$('.loginstate').html(s);
	}
	/**
	 * 退出登录
	 * @return {[type]} [description]
	 */
	_self.loginout = function() {
		$.ajax({
			type:'GET',
			url:'/index/loginout',
			success: function(msg) {
				var url = document.location.href.split('/');
				if(url[3] == 'space') {
					document.location.href = '/';
				}
				else {
					document.location.reload();
				}
			}
		})
	}
}
module.exports = loginState;
```
### index.html代码
```js
<div class="header">
	<div>
		<link rel="import" href="/widget/loginstate/loginstate.html?__inline"><h1><a href="/">主页</a></h1>
	</div>
</div>

```

由此前端模块化已经全部完成


## 所需插件
* fis3-postpackager-loader
* fis-parser-ejs
* fis-parser-swigt

```js
npm install -g 以上插件即可
```

## fis-conf.js
结合我的项目，本人的配置文件如下。下载本配置文件并且安装好fis后，可以直接使用。

```js
//由于使用了bower，有很多非必须资源。通过set project.files对象指定需要编译的文件夹和引用的资源
// fis.set('project.files', ['page/**', 'map.json', 'modules/**', 'lib']);
fis.set('project.ignore', ['*.bat', '*.rar', 'node_modules/**', 'fis-conf.js', "package.json"]);

fis.set('statics', '/www/static'); //static目录
fis.set('url', '/static');

//FIS modjs模块化方案，您也可以选择amd/commonjs等
fis.hook('commonjs', {
    mod: 'amd'
});

/*************************目录规范*****************************/
fis.match("**/*", {
        release: '${statics}/$&'
    })
    .match("**/*.ejs", {
        parser: fis.plugin('ejs'),
        isJsLike: true,
        release: false
    })
    //modules下面都是模块化资源
    .match(/^\/modules\/([^\/]+)\/(.*)\.(js)$/i, {
        isMod: true,
        id: '$1', //id支持简写，去掉modules和.js后缀中间的部分
        release: '${statics}/$&',
        url: '${url}/$&',
        //optimizer: fis.plugin('uglify-js')
    })
    //page下面的页面发布时去掉page文件夹
    .match(/^\/view\/(.*)$/i, {
        parser: fis.plugin('swigt'),
        useCache: false,
        release: '/$&'
    })
    .match(/^\/(widget|site)\/(.*)\.(js)$/i, {
      isMod:true,
      id: '$2',
       url: '${url}/$&'
    })
    .match(/^\/widget\/kindeditor-4.1.10\/(.*)\.(js)$/i, {
      isMod:false,
       url: '${url}/$&'
    })
    //less的mixin文件无需发布
    .match(/^(.*)mixin\.less$/i, {
        release: false
    })
    //前端模板,当做类js文件处理，可以识别__inline, __uri等资源定位标识
    .match("**/*.tmpl", {
        isJsLike: true,
        release: false
    }).match("**/*", {
        url: '/static$&'
    })
    //页面模板不用编译缓存
    .match(/.*\.(html|jsp|tpl|vm|htm|asp|aspx|php)$/, {
        useCache: false
    });



//打包与css sprite基础配置
fis.match('::packager', {
    // npm install [-g] fis3-postpackager-loader
    postpackager: fis.plugin('loader', {
        resourceType: 'mod',
        obtainScript: true,
        allInOne: true,
        useInlineMap: true, // 资源映射表内嵌
    }),
    packager: fis.plugin('map', {
      useTrack: false,
      'pkg/base.js': ['/modules/jquery/*.js', '/modules/layer/*.js', '/modules/pizzalayer/*.js', '/modules/pizzatools/*.js'],
      'pkg/base-a.js': ['/widget/globle/*.js', '/modules/pizzaui/pizza.ui.js', '/site/common/common.js'],
      'pkg/base.css': ['/css/pizza.css', '/css/iconfont.css']
    }),
    spriter: fis.plugin('csssprites', {
        layout: 'matrix',
        margin: '15'
    })
});

```
