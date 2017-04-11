title: Nginx配置网站适配PC和手机
date: 2016-01-07 15:28:34
tags: [nginx,移动端,mobile]
---

现在的网页开发一般都会适配与移动端，也即是响应式网页开发，这其中也有很多的ui框架可以帮助开发者完成自动适配（比如bootstrap）。但是我个人一般不采用这种方式，因为我觉的这种框架在移动端表现的过重了，而且响应式设计会影响到PC端web的设计思路，对比起来我还是选择开发两套系统的方式来走移动端适配。

自动判断是加载移动端页面还是pc端页面的通用做法就是通过User-Agent来做判断。

## 判断客户端的设备类型

首先国外有一套开源的通过User-Agent区分PC和手机的解决方案并且长期维护，可以直接使用：http://detectmobilebrowsers.com/


由于本文使用Nginx，只要在网站上下载Nginx配置即可。

下载的文件大致是这样：

```js

set $mobile_rewrite do_not_perform;

if ($http_user_agent ~* "(android|bb\d+|meego).+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge |maemo|midp|mmp|mobile.+firefox|netfront|opera m(ob|in)i|palm( os)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows ce|xda|xiino") {
  set $mobile_rewrite perform;
}

if ($http_user_agent ~* "^(1207|6310|6590|3gso|4thp|50[1-6]i|770s|802s|a wa|abac|ac(er|oo|s\-)|ai(ko|rn)|al(av|ca|co)|amoi|an(ex|ny|yw)|aptu|ar(ch|go)|as(te|us)|attw|au(di|\-m|r |s )|avan|be(ck|ll|nq)|bi(lb|rd)|bl(ac|az)|br(e|v)w|bumb|bw\-(n|u)|c55\/|capi|ccwa|cdm\-|cell|chtm|cldc|cmd\-|co(mp|nd)|craw|da(it|ll|ng)|dbte|dc\-s|devi|dica|dmob|do(c|p)o|ds(12|\-d)|el(49|ai)|em(l2|ul)|er(ic|k0)|esl8|ez([4-7]0|os|wa|ze)|fetc|fly(\-|_)|g1 u|g560|gene|gf\-5|g\-mo|go(\.w|od)|gr(ad|un)|haie|hcit|hd\-(m|p|t)|hei\-|hi(pt|ta)|hp( i|ip)|hs\-c|ht(c(\-| |_|a|g|p|s|t)|tp)|hu(aw|tc)|i\-(20|go|ma)|i230|iac( |\-|\/)|ibro|idea|ig01|ikom|im1k|inno|ipaq|iris|ja(t|v)a|jbro|jemu|jigs|kddi|keji|kgt( |\/)|klon|kpt |kwc\-|kyo(c|k)|le(no|xi)|lg( g|\/(k|l|u)|50|54|\-[a-w])|libw|lynx|m1\-w|m3ga|m50\/|ma(te|ui|xo)|mc(01|21|ca)|m\-cr|me(rc|ri)|mi(o8|oa|ts)|mmef|mo(01|02|bi|de|do|t(\-| |o|v)|zz)|mt(50|p1|v )|mwbp|mywa|n10[0-2]|n20[2-3]|n30(0|2)|n50(0|2|5)|n7(0(0|1)|10)|ne((c|m)\-|on|tf|wf|wg|wt)|nok(6|i)|nzph|o2im|op(ti|wv)|oran|owg1|p800|pan(a|d|t)|pdxg|pg(13|\-([1-8]|c))|phil|pire|pl(ay|uc)|pn\-2|po(ck|rt|se)|prox|psio|pt\-g|qa\-a|qc(07|12|21|32|60|\-[2-7]|i\-)|qtek|r380|r600|raks|rim9|ro(ve|zo)|s55\/|sa(ge|ma|mm|ms|ny|va)|sc(01|h\-|oo|p\-)|sdk\/|se(c(\-|0|1)|47|mc|nd|ri)|sgh\-|shar|sie(\-|m)|sk\-0|sl(45|id)|sm(al|ar|b3|it|t5)|so(ft|ny)|sp(01|h\-|v\-|v )|sy(01|mb)|t2(18|50)|t6(00|10|18)|ta(gt|lk)|tcl\-|tdg\-|tel(i|m)|tim\-|t\-mo|to(pl|sh)|ts(70|m\-|m3|m5)|tx\-9|up(\.b|g1|si)|utst|v400|v750|veri|vi(rg|te)|vk(40|5[0-3]|\-v)|vm40|voda|vulc|vx(52|53|60|61|70|80|81|83|85|98)|w3c(\-| )|webc|whit|wi(g |nc|nw)|wmlb|wonu|x700|yas\-|your|zeto|zte\-)") {
  set $mobile_rewrite perform;
}

## 条件判断，自行修改为自己的
if ($mobile_rewrite = perform) {
  rewrite ^ http://detectmobilebrowser.com/mobile redirect;
  break;
}


```

## nginx配置文件

```js
location / { #网站访问
     proxy_pass http://node_pc_app; # pc端
     if ($mobile_rewrite = perform) {
         proxy_pass http://node_mobile_app;  #手机版
      }
      proxy_redirect off;
      proxy_set_header X-Real-IP $remote_addr;
 }

location ~.*\.(gif|jpg|jpeg|png|bmp|swf|js|css|ejs|htm|html|txt) { #静态文件
    root   /pc/staic/; # pc端
    if ($mobile_rewrite = perform) {#手机版
       root   /mobile/static/;
    }
    index  index.html index.htm;
    expires max;
}

```

## user-agent的补充方案
因为user-agent的覆盖率是有限的，并不能保证所有的访问都可以通过user-agent准确判断，这时候我们可以添加一个cookie来加强判断，通常的做法是页面底部放一个手机版和pc版的切换跳转连接，并且点击的时候添加cookie，这样服务器端再判断是否存在cookie即可

nginx配置如下

```js
if ($http_cookie ~ 'gotopc=true') {  
    set $mobile_rewrite do_not_perform;  
} 
```

## 说明
使用 http://detectmobilebrowsers.com/ 提供的user-agent判断是不包含pad的，因为pad的分辨率其实是可以达到pc端访问的分辨率的，所以没有添加判断，如果您也需要适配pad端，则添加pad的user-agent即可