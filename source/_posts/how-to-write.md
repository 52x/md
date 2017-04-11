title: Hexo luocman 主题的文章书写规范
date: 2015-02-28 13:51:42
tags:
- 知识管理
- 教程
category: Hexo 
description: 将一些 Hexo luocman 主题的文章的书写规范和技巧总结一下。
---
将一些 Hexo luocman 主题的文章的书写规范和技巧总结一下。

## NEW 命令
创建一篇文章最简单的方法就是使用 new 命令。该命令的格式如下：

```
hexo new [layout] "title"
```
您可以在命令中指定文章的布局（layout），默认为 post，可以通过修改 _config.yml 中的 default_layout 参数来指定默认布局。

##Front-matter
Front-matter 是文件最上方以 --- 分隔的区域，用于指定个别文件的变量。以下是预先定义的参数，您可在模板中使用这些参数值并加以利用。

| 参数 | 描述 | 默认值 |
| - | - | - |
| layout |布局 | |
| title | 标题 | |
| date | 建立日期 | 文件建立时的日期 |
| update | 更新日期 | 文件更新时的日期 |
| comments | 开启文章评论功能 | true |
| tags | 标签（不适用于分页） | |
| categories | 分类（不适用于分页） | |
| permalink | 覆盖文章网址 | |
| toc | 文章目录是否启用 | true |
| list_number | 目录自动编号功能是否启用 | true |

##布局
Hexo 有三种默认布局：post、page 和 draft，它们分别对应不同的路径，而您自定义的其他布局和 post 相同，都将储存到 source/_posts 文件夹。除了自带的三种默认布局以外 luocman 还有 photo 和 link 两种布局。

##草稿
刚刚提到了 Hexo 的一种特殊布局：draft，这种布局在建立时会被保存到 `source/_drafts` 文件夹，您可通过 publish 命令将草稿移动到 `source/_posts` 文件夹，该命令的使用方式与 new 十分类似，您也可在命令中指定 layout 来指定布局。

```
hexo publish [layout] <title>
```

草稿默认不会显示在页面中，您可在执行时加上 - -draft 参数，或是把 render_drafts 参数设为 true 来预览草稿。
##模板
在新建文章时，Hexo 会根据 scaffolds 文件夹内相对应的文件来建立文件。所以每个人可以根据自己的使用习惯修改模板中的参数，我的模板参数如下：

注意：代码块的 `(( ))` 替换为大括号。

post:

```
layout: (( layout ))
title: (( title ))
toc: true
date: (( date ))
tags:
category:
description:
---
```

photo:

```
layout: (( layout ))
title: (( title ))
toc: false
date: (( date ))
tags:
category:
description:
---
```

draft:

```
title: (( title ))
toc: true
tags:
category:
description:
---
```

link:

```
layout: (( layout ))
title: (( title ))
date: (( date ))
tags:
category:
description:
---
```


##分类和标签

只有文章支持分类和标签，您可以在 Front-matter 中设置。在其他系统中，分类和标签听起来很接近，但是在 Hexo 中两者有着明显的差别：分类具有顺序性和层次性，也就是说 Foo, Bar 不等于 Bar, Foo；而标签没有顺序和层次。

```
categories:
- Diary
tags:
- PS3
- Games
```

##Markdown 语法

### 1. 斜体和粗体

使用 \* 和 \** 表示斜体和粗体。

示例：

这是 *斜体*，这是 **粗体**。

### 2. 分级标题

使用 === 表示一级标题，使用 --- 表示二级标题。

示例：

```
这是一个一级标题
============================

这是一个二级标题
--------------------------------------------------

### 这是一个三级标题
```

你也可以选择在行首加井号表示不同级别的标题 (H1-H6)，例如：# H1, ## H2, ### H3，#### H4。

### 3. 链接

使用 \[描述](链接地址) 为文字增加外链接。

示例：

这是去往 [本人博客](http://luoluodafang.info) 的链接。
<!-- more -->
### 4. 无序列表

使用 *，+，- 表示无序列表。

示例：

- 无序列表项 一
- 无序列表项 二
- 无序列表项 三

### 5. 有序列表

使用数字和点表示有序列表。

示例：

1. 有序列表项 一
2. 有序列表项 二
3. 有序列表项 三

### 6. 文字引用

使用 > 表示文字引用。

示例：

> 野火烧不尽，春风吹又生。

### 7. 行内代码块

使用 \`代码` 表示行内代码块。

示例：

让我们聊聊 `html`。

### 8.  代码块

使用 四个缩进空格 表示代码块。

示例：

    这是一个代码块，此行左侧有四个不可见的空格。
也可以使用 ``` 将代码包裹的方式。

### 9.  插入图像

使用 \!\[描述](图片链接地址) 插入图像。

示例：

![我的头像](http://7u2qla.com1.z0.glb.clouddn.com/head.jpg)

### 10. 表格

\| 表头表头表头表头 | 表头表头表头表头 | 表头表头表头表头 |
\| - | :-  | -: |
\| 内容 | 内容 | 内容 |
\| 内容 | 内容 | 内容 |

冒号位置表示对齐方式，默认左对齐。

示例：

| 表头表头表头表头 | 表头表头表头表头 | 表头表头表头表头 |
| - | :- | -: |
| 内容 | 内容 | 内容 |
| 内容 | 内容 | 内容 |

### 11. 禁止 Markdown 语法解释

在想要禁止 Markdown 语法解释的前方加反斜杠 `\` 即可。

##参考

[Hexo文档](http://hexo.io/zh-cn/docs/)