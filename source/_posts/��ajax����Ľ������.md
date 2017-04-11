title: 多ajax请求的各类解决方案（同步, 队列, cancel请求）的令一种解决方案
date: 2013-03-21 18:23:42
tags: [前端, ajax, 队列]
---

  最近在做地图和滚动条相关的功能：在绑定地图的拖拽，缩放和滚动条的滚动事件时，遇到ajax队列问题，即：当我把地图拖到我想要的省市或者滚动条滚动到我想要的位置过程中，会发送很多ajax请求，而这些请求到的数据对用户是无用的，只有最后一次才有用，并且还造成了很多服务器请求数的浪费。

  然后在园子里找到了[红草帽 * Arain](http://www.cnblogs.com/hongcaomao/ "红草帽 * Arain")的这篇文章：[多ajax请求的各类解决方案（同步, 队列, cancel请求）](http://www.cnblogs.com/hongcaomao/archive/2012/03/19/multi-ajaxrequest.html "多ajax请求的各类解决方案（同步, 队列, cancel请求）")，研究了一下发现此方法好是好，但没有解决http请求数的问题，因为就算是cancel掉请求，这个请求还是已经发送到了服务器(但并不绝对都是)，并且当ajax执行比较快的时候cancel掉已经是无用了(当然实际场景中发送ajax间隔时间都是很短的，所以影响基本也很小)。

测试代码：

```js
for(var i=1;i<10;i++){
     test(i);
 }

function test(i){
   setTimeout(function(){$.ajaxSingle($.extend(_settings,{
         type:"post",
        url:"PostPage.aspx?i="+new Date().getTime()+i,
        data:"second=2",
       anync:false,
       className:"post"
   }));},i*40);
}

```
测试结果：

![](/mdimg/21175810-293cecf67c5242ceb3e268570a8787b1.png)

可以看到间隔40ms发送请求就已经一大部分无法删除掉了(虽然实际场景更多的是发送ajax请求间隔很短)。

然后就觉得写那么多代码已经没必要了，简单的用setTimeout实现就可以了，而且能从根上杜绝以上两个问题发生。此方法也是在测试的时候发现的

```js
        var t;
        for(var i=1;i<10;i++){
            clearTimeout(t);
            t=setTimeout(function(){
                $.ajax({
                    type:"post",
                    url:"PostPage.aspx",
                    data:"second=1",
                    success:function(msg){

                    },
                    error:function(){

                    }
                });
            },100);
        }
```
这样子做的话：就不会因为时间间隔，执行速度等原因造成的请求删除不掉或者删除掉但服务器已经执行部分程序的情况，代码量少而且也很容易理解。

但是这也会有其他问题：比如用户等待数据的时候可能会偏长，这时候就要根据实际发送ajax请求的间隔来合理定义延迟时间(通常都不会长)。

另外一些场景，比如表单的提交，这时候还是灰掉按钮比较好。

以上是个人的一些想法，如方法有什么问题可以留言讨论。

--------
下面我把红草帽 * Arain文章也贴出来，方便查看


ajax带来很好的用户体验，于是一个稍微注重web系统使用ajax基本成为必然。当传统功能型web项目向用户体验型项目转变时，层出不穷的需求就来了。正如本篇所介绍的就是一个多个AJAX请求的情况下，如何利用jquery来处理几种case。

多个ajax请求同时发送，相互无依赖。
多个ajax请求相互依赖，必须有先后顺序。
多个请求被同时发送，只需要最后一个请求。
第1种case

应用场景： 这个场景很多，一个页面打开是多个区域同时请求后台得到各自的数据，没依赖，没顺序。
处理方案： 直接用jquery的ajax函数。这个用的非常多，这里从略，可看后面的代码中例子。

第2种case

应用场景： 多个ajax请求，需要顺序执行，后一个ajax请求的执行参数是前一个ajax的结果。例如： 用户登录后我们发送一次请求得到用户的应用ID，然后利用应用ID发送一次请求得到具体的应用内容（例子虽然不是太恰当，但基本就是这个意思了）。
处理方法： 
    1. 利用ajax参数async设置为false，进行同步操作。(这个方法只适合同域操作，跨域需使用下面两种方法)
    2. 利用ajax嵌套（这个同第1种情况）
    3. 利用队列进行操作

jquery ajax队列操作核心代码:

```js
(function ($) {
    var ajaxRequest = {};

    $.ajaxQueue = function (settings) {
        var options = $.extend({ className: 'DEFEARTNAME' }, $.ajaxSettings, settings);
        var _complete = options.complete;
        $.extend(options, {
            complete: function () {
                if (_complete)
                    _complete.apply(this, arguments);

                if ($(document).queue(options.className).length > 0) {
                    $(document).dequeue(options.className);
                } else {
                    ajaxRequest[options.className] = false;
                }
            }
        });

        $(document).queue(options.className, function () {
            $.ajax(options);
        });

        if ($(document).queue(options.className).length == 1 && !ajaxRequest[options.className]) {
            ajaxRequest[options.className] = true;
            $(document).dequeue(options.className);
        }
    };


})(jQuery);
```
 

第3中case

应用场景： 比较典型的是autocomplete控件的操作，这个我们可以使用第2种情况的处理方法，但我们可能只需要最后次按键后返回的结果，这样利用第2种处理方法未免有些浪费。
处理方法： 保留最后一次请求，cancel之前的请求。

```js
(function ($) {
var jqXhr = {}；
 $.ajaxSingle = function (settings) {
        var options = $.extend({ className: 'DEFEARTNAME' }, $.ajaxSettings, settings);

        if (jqXhr[options.className]) {
            jqXhr[options.className].abort();
        }

        jqXhr[options.className] = $.ajax(options);
    };
})(jQuery);
```

对于这些case都是在多个ajax请求，响应时间不能控制的情况。下面是完整Demo代码。

```js
(function ($) {
    var jqXhr = {},
        ajaxRequest = {};

    $.ajaxQueue = function (settings) {
        var options = $.extend({ className: 'DEFEARTNAME' }, $.ajaxSettings, settings);
        var _complete = options.complete;
        $.extend(options, {
            complete: function () {
                if (_complete)
                    _complete.apply(this, arguments);

                if ($(document).queue(options.className).length > 0) {
                    $(document).dequeue(options.className);
                } else {
                    ajaxRequest[options.className] = false;
                }
            }
        });

        $(document).queue(options.className, function () {
            $.ajax(options);
        });

        if ($(document).queue(options.className).length == 1 && !ajaxRequest[options.className]) {
            ajaxRequest[options.className] = true;
            $(document).dequeue(options.className);
        }
    };

    $.ajaxSingle = function (settings) {
        var options = $.extend({ className: 'DEFEARTNAME' }, $.ajaxSettings, settings);

        if (jqXhr[options.className]) {
            jqXhr[options.className].abort();
        }

        jqXhr[options.className] = $.ajax(options);
    };

})(jQuery);

var ajaxSleep = (function () {
    var _settings = {
        type: 'GET',
        cache: false,
        success: function (msg) {
            var thtml = $('#txtContainer').html();
            $('#txtContainer').html(thtml + "<br />" + msg);
        }
    };
    return {
        get: function (seconds, mode, isAsync) {
            var mode = mode || 'ajax',
                        isAsync = isAsync || false;

            $[mode]($.extend(_settings, {
                url: "ResponsePage.aspx?second=" + seconds,
                async: isAsync,
                className: 'GET'
            }));
        },
        post: function (seconds, mode, isAsync) {
            var mode = mode || 'ajax',
                        isAsync = isAsync || false;

            $[mode]($.extend(_settings, {
                type: 'POST',
                url: "PostPage.aspx",
                data: { second: seconds },
                async: isAsync,
                className: 'POST'
            }));
        }
    };
} ());

var launch = function (settings) {

    $('#txtContainer').html('');

    var mode = settings.mode,
                isAsync = settings.isAsync;

    ajaxSleep.get(12, mode, isAsync);
    ajaxSleep.get(10, mode, isAsync);
    ajaxSleep.get(8, mode, isAsync);

    ajaxSleep.post(6, mode, isAsync);
    ajaxSleep.post(4, mode, isAsync);
    ajaxSleep.post(2, mode, isAsync);
}

$(document).ready(function () {
    //第1种case
    $('#btnLaunchAsync').click(function () {
        launch({ isAsync: true });
    });

    //第2种case
    $('#btnLaunchSync').click(function () {
        launch({});
    });

    //第2种case
    $('#btnLaunchQueue').click(function () {
        launch({ mode: 'ajaxQueue', isAsync: true });
    });

    //第3种case
    $('#btnLaunchSingle').click(function () {
        launch({ mode: 'ajaxSingle', isAsync: true });
    });
});
```

```js
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

<html xmlns="http://www.w3.org/1999/xhtml">
<head id="Head1" runat="server">
    <title></title>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js" type="text/javascript"></script>
    <script type="text/javascript" src="js/default.js"></script>
</head>
<body>
    <form id="form1" runat="server">
    <input type="button" id="btnLaunchAsync" value="Launch Asynchronous Request" />
    <input type="button" id="btnLaunchSync" value="Launch Synchronous Request" />
    <input type="button" id="btnLaunchQueue" value="Launch Requested Queue" />
    <input type="button" id="btnLaunchSingle" value="Launch Single Request" />
    <div id="txtContainer"></div>
    </form>
</body>
</html>
```

```js
//ResponsePage.aspx
protected void Page_Load(object sender, EventArgs e)
        {
            int seconds = int.Parse(Request.QueryString["second"]);
            Thread.Sleep(seconds*1000);

            Response.Write("GET: selpt for "+ seconds.ToString() +" sec(s)");
        }

//PostPage.aspx
        protected void Page_Load(object sender, EventArgs e)
        {
            int seconds = int.Parse(Request.Form["second"]);

            Thread.Sleep(seconds * 1000); 

            Response.Write("POST: selpt for " + seconds.ToString() + " sec(s)");
        }

```