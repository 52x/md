---
title: Hexo 标签插件
date: 2016-10-01 21:40:05
tags:
  - Hexo
categories:
  - Tools
  - Hexo
---

标签插件和 Front-matter 中的标签不同，它们是用于在文章中快速插入特定内容的插件。

## 引用块

在文章中插入引言，可包含作者、来源和标题。
```
{% blockquote [author[, source]] [link] [source_link_title] %}
content
{% endblockquote %}
```
<!--more-->
### 示例

**没有提供参数，则只输出普通的blockquote**
```
{% blockquote %}
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Pellentesque hendrerit lacus ut purus iaculis feugiat. Sed nec tempor elit, quis aliquam neque. Curabitur sed diam eget dolor fermentum semper at eu lorem.
{% endblockquote %}
```
{% blockquote %}
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Pellentesque hendrerit lacus ut purus iaculis feugiat. Sed nec tempor elit, quis aliquam neque. Curabitur sed diam eget dolor fermentum semper at eu lorem.
{% endblockquote %}

**引用书上的句子**
```
{% blockquote David Levithan, Wide Awake %}
Do not just seek happiness for yourself. Seek happiness for all. Through kindness. Through mercy.
{% endblockquote %}
```
{% blockquote David Levithan, Wide Awake %}
Do not just seek happiness for yourself. Seek happiness for all. Through kindness. Through mercy.
{% endblockquote %}

**引用Twitter**
```
{% blockquote @DevDocs https://twitter.com/devdocs/status/356095192085962752 %}
NEW: DevDocs now comes with syntax highlighting. http://devdocs.io
{% endblockquote %}
```
{% blockquote @DevDocs https://twitter.com/devdocs/status/356095192085962752 %}
NEW: DevDocs now comes with syntax highlighting. http://devdocs.io
{% endblockquote %}

**引用网络上的文章**
```
{% blockquote Seth Godin http://sethgodin.typepad.com/seths_blog/2009/07/welcome-to-island-marketing.html Welcome to Island Marketing %}
Every interaction is both precious and an opportunity to delight.
{% endblockquote %}
```
{% blockquote Seth Godin http://sethgodin.typepad.com/seths_blog/2009/07/welcome-to-island-marketing.html Welcome to Island Marketing %}
Every interaction is both precious and an opportunity to delight.
{% endblockquote %}

## 代码块

在文章中插入代码
```
{% codeblock [title] [lang:language] [url] [link text] %}
code snippet
{% endcodeblock %}
```

### 示例

**普通的代码块**
```
{% codeblock %}
alert('Hello World!');
{% endcodeblock %}
```
{% codeblock %}
alert('Hello World!');
{% endcodeblock %}

**指定语言**
```
{% codeblock lang:objc %}
[rectangle setX: 10 y: 10 width: 20 height: 20];
{% endcodeblock %}
```
{% codeblock lang:objc %}
[rectangle setX: 10 y: 10 width: 20 height: 20];
{% endcodeblock %}

**附加说明**
```
{% codeblock Array.map %}
array.map(callback[, thisArg])
{% endcodeblock %}
```
{% codeblock Array.map %}
array.map(callback[, thisArg])
{% endcodeblock %}

**附加说明和网址**
```
{% codeblock \_.compact http://underscorejs.org/#compact Underscore.js %}
\_.compact([0, 1, false, 2, '', 3]);
=> [1, 2, 3]
{% endcodeblock %}
```
{% codeblock \_.compact http://underscorejs.org/#compact Underscore.js %}
\_.compact([0, 1, false, 2, '', 3]);
=> [1, 2, 3]
{% endcodeblock %}

## 其他标签

### 引用文章

引用其他文章的链接。
```
{% post_path slug %}
{% post_link slug [title] %}
```

**示例**
```
{% post_path hexo-commands %}
```
{% post_path hexo-commands %}

```
{% post_link hexo-commands %}
```
{% post_link hexo-commands %}

### 引用资源

引用文章的资源
```
{% asset_path slug %}
{% asset_img slug [title] %}
{% asset_link slug [title] %}
```

### Raw

如果您想在文章中插入 Swig 标签，可以尝试使用 Raw 标签，以免发生解析异常。
```
{% raw %}
content
{% endraw %}
```
