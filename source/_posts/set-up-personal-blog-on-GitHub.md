---
title: GitHub上搭建个人博客
date: 2016-03-28 16:01:08
tags:
 - blog
 - GitHub
categories:
 - blog
---

### 准备 ###

GitHub账号、ubuntu系统

### 安装git ###

```
sudo apt-get install git
```

配置账户信息

```
git config --global user.email "you@example.com" # 你在GitHub上注册的email
git config --global user.name "Your Name"
```

check this: [https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git)

### 安装NodeJS ###

```
wget -qO- https://raw.github.com/creationix/nvm/master/install.sh | sh
```

安装完成后，`重启终端`并执行下列命令。

```
nvm install 4
npm install -g hexo-cli
```

check this: [https://hexo.io/zh-cn/docs/index.html](https://hexo.io/zh-cn/docs/index.html)

### 建站 ###

```
hexo init hexo # if error, check this: https://github.com/hexojs/hexo/issues/580
cd hexo
npm install
```

check this: [https://hexo.io/zh-cn/docs/setup.html](https://hexo.io/zh-cn/docs/setup.html)

### 安装NexT主题 ###

```
cd hexo
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

check this: [http://theme-next.iissnan.com/getting-started.html](http://theme-next.iissnan.com/getting-started.html)

### 创建GitHub仓库 ###

仓库名必须为：`你的GitHub账号名.github.io`
得到生成的仓库的链接，如：https://github.com/aeiouaoeiuv/aeiouaoeiuv.github.io.git

### hexo部署 ###

先修改`_config.yml`，修改如下：

```
deploy:
  type: git
  repo: https://github.com/aeiouaoeiuv/aeiouaoeiuv.github.io.git
```

然后执行如下命令：


```
cd hexo
npm install hexo-deployer-git --save
hexo deploy
```

输入账号和密码，稍等片刻即可通过 `你的账号名.github.io` 访问博客了。

check this: [https://hexo.io/zh-cn/docs/deployment.html#Git](https://hexo.io/zh-cn/docs/deployment.html#Git)

### 换电脑怎么办？ ###

在GitHub上新建一个repository，名为 `hexo` ，拷贝该url
在ubuntu上clone该repository，如： `git clone https://github.com/aeiouaoeiuv/hexo.git hexo.git`
然后把你的 `hexo` 里所有文件拷贝至该 `hexo.git` 目录，如： `cp -rf hexo/* hexo.git`

然后执行如下命令：

```
cd hexo.git
rm -rf themes/next/.git # 注意删掉此.git目录，否则next目录不会上传
git add .
git commit -m "comments"
git push origin master
```

### 新电脑操作步骤 ###

```
git clone https://github.com/aeiouaoeiuv/hexo.git hexo.git
cd hexo.git
npm install # 没有npm？那就看上面的安装NodeJS吧。
```

### 以后写文章步骤 ###

```
cd hexo.git
hexo clean
rm -rf db.json .deploy_git public # 这两个文件夹没用，public是每次hexo生成的静态博客文件，无需同步。
git add .
git commit -m "comments"
git push origin master
hexo deploy
```

git官方文档：https://git-scm.com/book/zh/v2
hexo官方文档：https://hexo.io/zh-cn/docs/
NexT主题官方文档：http://theme-next.iissnan.com/getting-started.html
