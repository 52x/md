---
title: 记一起hexo-asset-image插件未起作用的处理
date: 2017-02-21 15:37:00
updated: 2017-02-21 15:37:00
tags: [hexo]
categories: Hexo
keywords: [hexo,image,asset]
description: hexo-asset-image插件未起作用的处理
---

# 起---问题的出现

最近在尝试着用Hexo来写Blog，习惯用的编辑器是Typora，之前一直还比较顺，就是用图片的时候，不喜欢hexo自带的语法，其实所有的hexo自带的语法我都没有用:first_quarter_moon_with_face:

之前，所有的图片放在了source下的images文件夹中，不过担心数量一多不容易发现，所以根据网上搜索的结果，引入了hexo-asset-image插件。这样本地使用应该方便了结果出来的是如下的框框： 

![说明](2017-02-21-hexo-asset-image-not-work\fail.png)

本地引用图片是采用这样的语法：`![说明](2017-02-21-hexo-asset-image-not-work\fail.png)`可是还是不行啊。于是开始了研究之旅

# 承---问题的原因

打开node_modules\hexo-asset-image，发现很简单，就一个js文件，一共才50行，一行行读下来。分析一下

```
var cheerio = require('cheerio');
```

这一行是引入了cheerio库，类似于jquery的库，用来处理dom非常的nice。

```
hexo.extend.filter.register('after_post_render', function(data){
```

这一行是在文章处理完以后，进入过滤器，处理页面的。

```
var linkArray = link.split('/').filter(function(elem){
  return elem != '';
});
var srcArray = src.split('/').filter(function(elem){
  return elem != '';
});
if(linkArray[linkArray.length - 1] == srcArray[0])
  srcArray.shift();
```

这一段，是处理核心，主要是取出link 和 src的地址，进行比较，符合规则则去除多余的生成的地址。我碰到的问题就出现在这里了。

 在我的_config.yml中。

```
permalink: :year/:month/:title/
new_post_name: :year-:month-:day-:title.md # File name of new posts
```

这两个的定义可以方便我按日期查找post的，结果这里因为自定义了，所以不被支持。

列出来看看：

```
link: 2017/02/hexo-asset-image-not-work/ 
linkArray: [ '2017', '02', 'hexo-asset-image-not-work' ]

src: 2017-02-21-hexo-asset-image-not-work/fail.png 
srcArray: [ '2017-02-21-hexo-asset-image-not-work', 'fail.png' ]
```

可以看出来，linkArray[linkArray.length - 1] 不等于 srcArray[0]

# 转--问题的解决

问题发现了，要处理一下呗，直接禁用了插件

在script的文件中新建了一个index.js

复制hexo-asset-image的内容，稍微修改点，如下

```javascript
'use strict';
let cheerio = require('cheerio');

// http://stackoverflow.com/questions/14480345/how-to-get-the-nth-occurrence-in-a-string
function getPosition(str, m, i) {
    return str.split(m, i).join(m).length;
}

hexo.extend.filter.register('after_post_render', function (data) {
  //为提高效率，非post直接被我给忽略掉了。。  
  	if (data.layout !== "post") {
        return;
    }
    let config = hexo.config;
    if (config.post_asset_folder) {
        let link = data.permalink;
        let slug = data.slug;	//新增加的，取文章的slug
        let beginPos = getPosition(link, '/', 3) + 1;
        // In hexo 3.1.1, the permalink of "about" page is like ".../about/index.html".
        let endPos = link.lastIndexOf('/') + 1;
        link = link.substring(beginPos, endPos);

        let toprocess = ['excerpt', 'more', 'content'];
        for (let i = 0; i < toprocess.length; i++) {
            let key = toprocess[i];

            let $ = cheerio.load(data[key], {
                ignoreWhitespace: false,
                xmlMode: false,
                lowerCaseTags: false,
                decodeEntities: false
            });

            $('img').each(function () {
                // For windows style path, we replace '\' to '/'.
                let src = $(this).attr('src').replace('\\', '/');
                if (!/http[s]*.*|\/\/.*/.test(src)) {
                    // For "about" page, the first part of "src" can't be removed.
                    // In addition, to support multi-level local directory.
                    let linkArray = link.split('/').filter(function (elem) {
                        return elem != '';
                    });
                    let srcArray = src.split('/').filter(function (elem) {
                        return elem != '';
                    });

                    if (linkArray[linkArray.length - 1] == srcArray[0]) {
                        srcArray.shift();
                        //以下的是新增的，判断，当src和link都存在slug的时候，移除src中的第一项。
                    }else if((link.indexOf(slug)>-1)&&(src.indexOf(slug)>-1)){
                        srcArray.shift();
                    }
                    src = srcArray.join('/');
                    $(this).attr('src', '/' + link + src);
                }
            });
            data[key] = $.html();
        }
    }
});
```

新增的一段，就是取出来文章的slug作比较，这个slug生成请参考 `hexo/lib/plugins/console/new.js`

这样，判断当src和link都存在slug的时候，移除src中的第一项。 问题解决。

# 合--写了起承转，感觉非得写个合才行啊

问题解决了，又学到了一点东西，感觉很Nice。。萌萌哒(⊙o⊙)…



# 续

- 又去hexo-asset-image的作者的页面去看了一下，他最新的修改把条件去掉了，我也想过，不过那样，之前文章中，使用的../images/xx图片就无法访问了。未取这种方式。