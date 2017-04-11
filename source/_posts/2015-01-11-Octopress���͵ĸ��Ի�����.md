---
title: "Octopress博客的个性化配置"
date: 2015-01-11 21:52:49 +0800
tags:
comments: true
categories: 其他技术
keywords: octopress, blog, github, 优化访问速度， 多说评论， 添加访客统计
---

本文主要讲述了对Octopress搭建的博客进行一些个性化的配置，主要包括以下几个方面：

* 优化提高博客的访问速度
* 设置链接在新窗口打开
* 配置首页文章以摘要形式展示
* 代码着色
* 添加侧边栏文章分类
* 添加多说评论系统
* 自动为图片添加URL前缀
* 添加访客统计

<!--more-->
原文链接：<http://tianweili.github.com/blog/2015/01/11/setup-octopress-blog/>

## 提高博客访问速度
因为“墙”的关系，所以Octopress建立以后会发现访问速度奇慢无比，竟然超过了40s。

![](http://7u2i08.com1.z0.glb.clouddn.com/setup-octopress-blog/call-octopress-blog-slowly.png)

仔细分析后我们发现其中都是一些被墙的请求报了404Error，所以导致访问博客巨慢无比，下面我们就一次阉割掉这些被墙的请求。T_T

### 替换Google JS公共库

Octopress默认使用的是Google的JS公共库地址，加载的过程无比的缓慢。因此我们要把它改为[百度的JS公共库](http://developer.baidu.com/wiki/index.php?title=docs/cplat/libs)，需要把`/source/_includes/head.html`文件中的Google公共库地址改为：

```xml
	<script src="//libs.baidu.com/jquery/1.7.2/jquery.min.js"></script>
```
### 去掉Twitter

从上图可以看出加载失败的还有twitter，这个也得给去掉。

把在根目录下的`_config.yml`文件中Twitter内容给注释掉。
```text
	# Twitter
	#twitter_user:
	#twitter_tweet_button: true
```

把`\source\_includes\after_footer.html`文件中的twitter内容给注释掉：

```html
	{% raw)<!--{% include twitter_sharing.html)-->{% endraw)
```

### 删除Google font

把在`\source\_includes\custom\head.html`中的Google font样式给删除：

```html
	<link href="//fonts.googleapis.com/css?family=PT+Serif:regular,italic,bold,bolditalic" rel="stylesheet" type="text/css">
	<link href="//fonts.googleapis.com/css?family=PT+Sans:regular,italic,bold,bolditalic" rel="stylesheet" type="text/css">
```

## 设置链接在新窗口打开
在博文中，如果点击链接直接在本窗口打开了，那么用户体验就不是很好。而markdown的标准语法是不支持链接在新窗口打开的，虽然可以通过在markdown中直接写html标签来解决这个问题，但是这与markdown的简洁书写特性不符。但是我们可以通过设置Octopress来达到这种效果，即在`\source\_includes\custom\head.html`文件中添加如下一段代码：
```html
	<script>
		function addBlankTargetForLinks () {
		  $('a[href^="http"]').each(function(){
			  $(this).attr('target', '_blank');
		  });
		}
	
		$(document).bind('DOMNodeInserted', function(event) {
		  addBlankTargetForLinks();
		});
	</script>
```
## 首页文章以摘要形式展示

1. 在文章对应的markdown文件中，在需要显示在首页的文字后面添加`<!--more-->`，执行rake generate后在首页上会看到只显示`<!—more—>`前面的文字，文字后面会显示`Read on`链接，点击后进入文字的详细页面;
2. 如果想将Read on修改为中文，可以修改_config.yml文件

```html
	#excerpt_link: "Read on &rarr;"  # "Continue reading" link text at the bottom of excerpted articles
	excerpt_link: "阅读全文&rarr;"  # "Continue reading" link text at the bottom of excerpted articles
```

## 代码着色
Octopress使用的是Pygments来进行代码着色的，使用方式也比较简单如下所示：

```java
//java code
```

[Pygments支持的语言列表](http://pygments.org/languages/)

### 修改代码生成css

当然你也可以修改Pygments生成的代码css样式。

Pygments默认提供了很多css样式，你可以在python shell中用下面命令列出当前pygments所支持的样式：

```bash
from pygments.styles import STYLE_MAP
STYLE_MAP.keys()
['manni', 'igor', 'xcode', 'vim', 'autumn', 'vs', 'rrt', 'native', 'perldoc', 'borland', 'tango', 'emacs', 'friendly', 'monokai', 'paraiso-dark', 'colorful', 'murphy', 'bw', 'pastie', 'paraiso-light', 'trac', 'default', 'fruity']
```
通过-S来选择，需要生成default的样式：

```bash
pygmentize -S default -f html > your/path/pygments.css
```


有时候Octopress会把我们想要展示的Ruby代码解析成HTML，如果只是想展示代码，而不让Octopress来解析，那么可以在代码前后加入`raw`和`endraw`代码。

## 添加侧边栏文章分类（category）
1.在`plugins`目录下创建`category_list_tag.rb`文件，内容如下：

```html
	module Jekyll 
	  class CategoryListTag < Liquid::Tag 
	    def render(context) 
	      html = "" 
	      categories = context.registers[:site].categories.keys 
	      categories.sort.each do |category| 
	        posts_in_category = context.registers[:site].categories[category].size 
	        category_dir = context.registers[:site].config['category_dir'] 
	        category_url = File.join(category_dir, category.gsub(/_|\P{Word}/, '-').gsub(/-{2,}/, '-').downcase) 
	        html << "<li class='category'><a href='/#{category_url}/'>#{category} (#{posts_in_category})</a></li>\n" 
	      end 
	      html 
	    end 
	  end 
	end
	
	Liquid::Template.register_tag('category_list', Jekyll::CategoryListTag)
```

2.添加`source/_includes/asides/category_list.html`文件，内容如下：
```html
	<section>
	  <h1>文章分类</h1>
	  <ul id="categories">
	    {% category_list)
	  </ul>
	</section>
```
3.修改`_config.yml`文件，在`default_asides`项中添加`asides/category_list.html`，值之间以逗号隔开，值的先后顺序代表了侧边栏展现的先后顺序。
```text
	default_asides: [asides/category_list.html, asides/recent_posts.html, asides/github.html, asides/delicious.html, asides/pinboard.html, asides/googleplus.html]
```

在侧边栏还可以添加其他组件，如微博、标签云等，添加方式和上面类似。

## 添加多说评论
Octopress默认自带了DISQUS，但是对于国内不是很好用。所以在经过考虑之后选择了国内比较流行的多说评论系统。
首先要去[多说网站注册](http://duoshuo.com/)，获取站点的`short_name`。

在`_config.yml`中添加
```html
	# duoshuo comments
	duoshuo_comments: true
	duoshuo_short_name: yourname
```
在`./source/_layouts/post.html`中的`disqus`代码

```html
	{% if site.disqus_short_name and page.comments == true)
	  <section>
	    <h1>Comments</h1>
	    <div id="disqus_thread" aria-live="polite">{% include post/disqus_thread.html)</div>
	  </section>
	{% endif)
```

下方添加多说评论模块：
```html
	{% if site.duoshuo_short_name and site.duoshuo_comments == true and page.comments == true)
	  <section>
	    <h1>Comments</h1>
	    <div id="comments" aria-live="polite">{% include post/duoshuo.html)</div>
	  </section>
	{% endif)
```
如果你希望一些单独的页面下方也放置评论功能，那么在`./source/_layouts/page.html`中也做如上修改。
然后创建一个`./source/_includes/post/duoshuo.html`文件，内容如下：

```html
	<!-- Duoshuo Comment BEGIN -->
	<div class="ds-thread" data-title="{% if site.titlecase){{ page.title | titlecase }}{% else){{ page.title }}{% endif)"></div>
	<script type="text/javascript">
	  var duoshuoQuery = {short_name:"{{ site.duoshuo_short_name }}"};
	  (function() {
	    var ds = document.createElement('script');
	    ds.type = 'text/javascript';ds.async = true;
	    ds.src = 'http://static.duoshuo.com/embed.js';
	    ds.charset = 'UTF-8';
	    (document.getElementsByTagName('head')[0] 
	    || document.getElementsByTagName('body')[0]).appendChild(ds);
	  })();
	</script>
	<!-- Duoshuo Comment END -->
```
最后再修改`_includes/article.html`文件，在
```html
	{% if site.disqus_short_name and page.comments != false and post.comments != false and site.disqus_show_comment_count == true)
	         | <a href="{% if index){{ root_url }}{{ post.url }}{% endif)#disqus_thread">Comments</a>
	{% endif)
```
下方添加下面代码：
```html
	{% if site.duoshuo_short_name and page.comments != false and post.comments != false and site.duoshuo_comments == true)
          | <a href="{% if index){{ root_url }}{{ post.url }}{% endif)#comments">Comments</a>
	{% endif)
```

## 自动为图片添加url前缀

我把图片资源都[放在了七牛云存储](https://portal.qiniu.com/)上，写博客时候就是用七牛的外链。但是这样有几个问题：

* 每次写博客插入图片外链地址时候都很麻烦，需要给每张图片都添加七牛外链地址url前缀；
* 如果以后更换了存储，那就麻烦了，需要依次编辑替换每个图片的url前缀

现在我们就使用一种灵活的方式来配置并自动生成图片的URL前缀：

首先修改`/plugins/image_tag.rb`文件，在`@img['class'].gsub!(/"/, '') if @img['class']`后添加下面一行代码：

```ruby
@img['src'] = Jekyll.configuration({})['static_file_prefix'] + @img['src'] if @img['src'][0] == '/'
```
然后再修改根目录下的`_config.yml`文件，添加如下配置：

```html
	# Add url prefix for image automatically
	static_file_prefix: http://7u2i08.com1.z0.glb.clouddn.com
```

最后我们在插入图片的时候要记住不能再使用Markdown语法来写了，要[使用Ocotpress自定义的IMG标签来插入图片](http://octopress.org/docs/plugins/image-tag/)。

本地预览先generate后preview，这样一来插入图片就灵活方便多了。

## 添加访客统计
本博客的访客统计系统使用的是Flag Counter，所以要[先去Flag Counter获取代码](http://www.flagcounter.com/)。

拿到代码后添加`.\source\_includes\custom\asides\flag_counter.html`文件：

```html
<section>
    <h1>访客统计</h1>
    <br/>
    <a href="http://s07.flagcounter.com/more/2SH"><img src="http://s07.flagcounter.com/count/2SH/bg_FFFFFF/txt_000000/border_CCCCCC/columns_2/maxflags_12/viewers_0/labels_0/pageviews_1/flags_0/" alt="Flag Counter" border="0"></a>
</section>
```
将页面添加到侧边栏，在`./_config.yml`配置文件中添加下面一行配置：

```html
	default_asides: [……, custom/asides/flag_counter.html]
```

最后添加控制开关，在`./_config.yml`配置文件中添加下面一行配置：
```html
	# Flag Counter
	flag_counter: true
```

## SEO

[百度站长工具](http://zhanzhang.baidu.com/site/index)

[百度统计](http://tongji.baidu.com/web/9700918/overview/sole?siteId=6181997)

[Google Analytics](https://www.google.com/analytics/web/?authuser=0#home/a58552615w92512090p96324524/)

[Google站长工具](https://www.google.com/webmasters/tools/home?hl=zh-CN&siteUrl=http://tianweili.github.io/&authuser=0)

---

作者：[李天炜](http://tianweili.github.com/)

原文链接：<http://tianweili.github.com/blog/2015/01/11/setup-octopress-blog/>

转载请注明作者和文章出处，谢谢。
