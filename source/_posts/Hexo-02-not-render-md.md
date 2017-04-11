---
title: Hexo优化（2）：部署时保证README.md文件不被渲染
date: 2016-03-24 19:51:22
tags: Hexo
toc: true
---

其实确保README.md文件不被渲染也挺容易的，只要在博客根目录下的配置文件_config.yml中配置一下"skip_render"选项就行了，将不需要渲染的文件名称加入的其选项下就行了。

```bash
skip_render: README.md
```

## 参考
* [Hexo上传README.md文件](http://starsky.gitcafe.io/2015/12/31/Hexo%E4%B8%8A%E4%BC%A0README.md%E6%96%87%E4%BB%B6/)
