title: Git 同时推送到两个不同的 remote
date: 2016-05-08 8:50 PM
categories: ""
tags: [git,]
---

近期遇到需要将git推送到不同的git在线服务商的问题，所以查找了下资料.

<!--more-->

在最新版本的git中，可以这么做:

### 配置
```bash
git remote set-url --add --push origin git://original/repo.git
git remote set-url --add --push origin git://another/repo.git
```

### 效果图

![http://harchiko.qiniudn.com/Screen%20Shot%202016-05-08%20at%208.48.19%20PM.png](http://harchiko.qiniudn.com/Screen%20Shot%202016-05-08%20at%208.48.19%20PM.png)

### 参考链接

[http://stackoverflow.com/questions/14290113/git-pushing-code-to-two-remotes](http://stackoverflow.com/questions/14290113/git-pushing-code-to-two-remotes)