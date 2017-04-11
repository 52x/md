title: 演示：HTML5应用之文件拖拽上传
date: 2015-08-018 23:22:44
permalink: dropPic
tags:
- java
- html5
categories:
- java

---
一个HTML5拖拽上传的小例子。
HTML5可以实现文件的拖拽上传与下载。但是目前技术有限，只能操作单个文件或拖拽多个文件通过循环遍历实现多个文件上传下载，至于文件夹的操作，由于技术水平有限，暂时还没有找到资料。

需要注意的知识点有三个
1. 拖拽事件的监听。取消浏览器默认的监听行为，重新添加拖拽监听事件。

2. `formdata`的操作。通过`e.dataTransfer.files`获取文件对象，通过`formdata`实现表单的数据处理。

3. `XMLHttpRequest`的操作。主要通过send方法发送到服务器。


```html
<!DOCTYPE HTML>
<html>
<head>
<meta charset="gbk">
<title>演示：HTML5应用之文件拖拽上传</title>
<script type="text/javascript" src="http://www.helloweba.com/demo/js/jquery-1.7.2.min.js"></script>
<style type="text/css">
.demo{width:500px; margin:50px auto}
#drop_area{width:100%; height:100px; border:3px dashed silver; line-height:100px; text-align:center; font-size:36px; color:#d3d3d3}
#preview{width:500px; overflow:hidden}
</style>
</head>

<body>


<div id="main">
<div class="demo">
<div id="drop_area">将图片拖拽到此区域</div>
<div id="preview"></div>
</div>
</div>

<script type="text/javascript">
$(function(){
//阻止浏览器默认行。
$(document).on({
dragleave:function(e){ //拖离
e.preventDefault();
},
drop:function(e){ //拖后放
e.preventDefault();
},
dragenter:function(e){ //拖进
e.preventDefault();
},
dragover:function(e){ //拖来拖去
e.preventDefault();
}
});

//================上传的实现
var box = document.getElementById('drop_area'); //拖拽区域
debugger
box.addEventListener("drop",function(e){
e.preventDefault(); //取消默认浏览器拖拽效果
var fileList = e.dataTransfer.files; //获取文件对象
//检测是否是拖拽文件到页面的操作
if(fileList.length == 0){
return false;
}
//检测文件是不是图片
if(fileList[0].type.indexOf('image') === -1){
alert("您拖的不是图片！");
return false;
}

//拖拉图片到浏览器，可以实现预览功能
var img = window.webkitURL.createObjectURL(fileList[0]);
var filename = fileList[0].name; //图片名称
var filesize = Math.floor((fileList[0].size)/1024); 
if(filesize>500){
alert("上传大小不能超过500K.");
return false;
}
//alert(filesize);
var str = "<img src='"+img+"'><p>图片名称："+filename+"</p><p>大小："+filesize+"KB</p>";
$("#preview").html(str);

//上传
xhr = new XMLHttpRequest();
xhr.open("post", "upload.php", true);
xhr.setRequestHeader("X-Requested-With", "XMLHttpRequest");

var fd = new FormData();
fd.append('mypic', fileList[0]);

xhr.send(fd);


},false);
});
</script>

</body>
</html>
```