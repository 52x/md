title: Hexo博客优化 Part II
date: 2015-10-02 22:34:40
categories: Hexo|博客技术
tags: [Hexo, blog, git]
---

不了解`css`,`js`和`swig`，重新看了一次主题使用介绍，又大致过了一遍`hexo`和`NexT`的基本配置文件，继续对这个hexo博客进行配置。
<!-- more -->

## 配置文件 _config.yml
在重新看这个配置文件之前已经Google了一大圈，发现普通自建博客的解决方案，Hexo都提供了最简洁的方案，Hexo已经尽最大可能降低了配置的时间成本。这个特点集中体现在站点配置文件或者主题的配置文件`_config.yml`文件中。以下修改均发生在主题文件夹的配置文件中。

### 代码高亮主题 Code Highlight Theme
Hexo中代码高亮的方案取自Github的资源[tommorow主题](https://github.com/chriskempson/tomorrow-theme)。默认为`tomorrow`白色底纹的高亮设定，配置文件中代码为`normal`， 我把它改成`night eighties`

```
highlight_theme: night eighties
```

### 站点地图 Sitemap
在git bash中输入
``` bash
$ npm install hexo-generator-sitemap --save
$ npm install hexo-generator-baidu-sitemap --save
```
生成`sitemap.xml`和`baidusitemap.xml`两份站点地图文件(其实我到现在还不知道，这两份文件放在哪儿了,然而位置不重要)。
这时候在浏览器地址栏输入`博客地址/sitemap.xml`(如，我输入: papacochon.com/sitemap.xml)，应该可以返回以下截图中的内容，否则说明sitemap没有成功创建。
![sitemap](/img/sitemap_screenshot.png)

同时配置文件中添加：
```
sitemap:
  path: sitemap.xml

baidusitemap:
  pth: baidusitemap.xml
```

可去[百度](http://zhanzhang.baidu.com/site/index?action=add)和[360](http://www.sousuoyinqingtijiao.com/360/)提交搜索结果(后者未验证)。如果有梯子，也可以去谷歌提交一下。谷歌度娘均需要验证，顺着提示的验证方法去做。
提示：如果选用添加HTML标签的验证方法，需要将标签添加在以下文件中：
E:\Hexo\themes\next\layout\_partials\head.swig
举个栗子(前三行是原有的，后两行是添加的标签)：
```
<meta charset="UTF-8"/>
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1"/>
<meta name="google-site-verification" content="XXXXXXXXXXX" />
<meta name="baidu-site-verification" content="XXXXXXXXXXX" />
```
验证成功后，如有需要，在相应的提交站点的位置输入`sitemap.xml`即可，不再赘述。

### Tab上的图标 favicon
我很好奇，访问一个站点或者博客的时候，经常可以在那个页面的tab上看到个性化的图标。查了查，原来这图标是有名字的，名叫`favicon`！可是我想要设置一个我的favicon的热情立即被一堆复杂的`html`/`css`/`ejs`/blabla的攻略给浇灭了。然而hexo还是很体贴的提供了一个简洁的方案。将`favicon.ico`放在站点根目录的source文件夹下，然后在配置文件中设置(其实默认已经设定好了，就等你放`.ico`文件了)，什么？没有`favicon.ico`怎么办？去搜"favicon generator"能找出好多网站提供文件格式转换服务。
这是我的favicon：
![stormtrooper](/img/stormtrooper.png)

## 边栏头像 Sidebar Profile
侧边栏默认的头像是一个矩形框，把它改成圆哒  ╮(╯▽╰)╭
**注意**:上传的头像最好是一个正方形，不然变形以后就会变成椭圆。。。
先找到对应路径下的文件E:\Hexo\themes\next\source\css\_common\_section\sidebar.styl
然后做一下修改(添加的部分用`start`和`end`标记出来了)：
```
.site-author-image {
  display: block;
  margin: 0 auto;
  max-width: 96px;
  height: auto;
  border: 2px solid #333;
  padding: 2px;
  
  /* start*/
  border-radius: 90%
  webkit-transition: 1.4s all;
  moz-transition: 1.4s all;
  ms-transition: 1.4s all;
  transition: 1.4s all;
  /* end */
}
```
(sorry上面这段code是抄的，然而已经忘了是从哪位大神的博客上剽窃的了，看到请见谅。。。)

## 字体设置 Fonts
[官方的文档](http://theme-next.iissnan.com/)中说修改这个文件就行(请对应到自己的站点文件夹)：
E:\Hexo\themes\next\source\css\_variables\custom.styl 
于是我就兴冲冲的改了：
```
$font-family-headings = "Microsoft YaHei", Cambria, STZhongsong, Calibri // 标题，修改成你期望的字体族
$font-family-base = Georgia, STFangsong, "Microsoft YaHei", "Courrier New" // 修改成你期望的字体族
```
然而并没有生效。。。
于是又折腾折腾找到了这个神奇的文件：
E:\Hexo\themes\next\source\css\_variables\base.styl
这里面你可以看到一堆字体、字号、颜色的基础配置。
注意观察那些`$`符号，有玄机哦~扯远了，我厚颜无耻地把字体的基础配置也改了。
```
// Font families.
$font-family-sans-serif   = "Avenir Next", Avenir, Tahoma, Vendana, sans-serif, Cambria
$font-family-serif        = "PT Serif", Cambria, Georgia, "Times New Roman", serif
$font-family-monospace    = "Courier New", "PT Mono", Consolas, Monaco, Menlo, monospace
$font-family-chinese      = "STFangsong", "STZhongsong", "Hiragino Sans GB", "Microsoft YaHei"
$font-family-base         = Cambria, "Century Gothic", $font-family-chinese, sans-serif
$font-family-headings     = Cambria, Georgia, $font-family-chinese, "Times New Roman", serif
$font-family-posts        = $font-family-base
```
记不得原先是啥样子了，总之这是我理想的配置，仅供参考。

全部设定完成以后，在git bash里重新部署一遍。
```
hexo clean
hexo d -g
```
啊哈，大功告成。

----------------

其实还有很多想改的，比如上传图片自带边框，还不知道怎么去掉这些border。。。慢慢来。。。
 (￣o￣) . z Z

[Hexo博客优化 Part I 在这里](http://papacochon/2015/10/02/hexo-2_set-the-theme/)
