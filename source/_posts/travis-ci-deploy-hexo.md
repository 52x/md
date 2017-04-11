title: 使用 Travis-CI 生成发布 Hexo 博客
date: 2015-11-27 6:51 PM
categories: 
tags: [hexo,github,git]
---

本篇文章将指导你如何使用Travis-CI自动部署hexo生成的静态博客到github上。

<!--more-->

## 背景介绍

在最新的github中，如果想要你的github repo可以作为博客，需要命名为类似 `zhaochunqi.github.io`这样的。
在这里，我在此repo下建立了两个完全不同的分支，source和master分支。

source分支放置hexo的一些源文件，master分支放置静态博客资源。

**我们需要做的就是获取到source分支的改动，自动deploy到master分支**

所以步骤就是： 获取改动->deploy

## travis-CI

travis-CI([https://travis-ci.org/](https://travis-ci.org/))是集成测试工具，目前只支持github。

在travis-CI中对`zhaochunqi.github.io`开启自动构建，配置如下：
![](http://harchiko.qiniudn.com/Screen%20Shot%202015-11-28%20at%205.37.05%20AM.png)
（建议开启Build only if .travis.yml is prezent.)

## 配置 `.travis.yml`

### 获取`github`权限

两种方法 

	1. SSH
	2. github token

这里采用的是github token的方法，（SSH配置较为复杂）

点击头像->settings-> Personal access tokens 新建一个token,**会获取到一串字符**，记录下下面需要用到。
![](http://harchiko.qiniudn.com/Screen%20Shot%202015-11-28%20at%206.48.06%20AM.png)

### 将token配置到travis-CI

#### 安装Travis-CI命令行工具

```bash
gem install travis
```

### 登陆 Travis CI

需要输入Github账号和密码

```bash
travis login --auto
```
加密环境变量（token）

```bash
travis encrypt GH_OWNER=super_secret
```
此命令会写入加密后的环境变量到.travis.yml中.


这样，使用`https://${GH_OAUTH_TOKEN}@github.com/${GH_OWNER}/${GH_PROJECT_NAME}`即可进行正常的push和pull了。



最后贴一下我的配置[29 Nov 2016 更新了 cache，加快构建速度]

```yaml
language: node_js
sudo: false
node_js: 
   - "4.4.4"

# Travis-CI Caching
cache:
  directories:
    - node_modules


env:
  global:
    - GH_OWNER: zhaochunqi
    - GH_PROJECT_NAME: zhaochunqi.github.io
    
branches:
  only:
    - source
    
# Build Lifecycle
install:
  - npm install

script:
  - git clone -b master https://${GH_OAUTH_TOKEN}@github.com/${GH_OWNER}/${GH_PROJECT_NAME} public
  - hexo g
  - git config --global user.name "zhaochunqi"
  - git config --global user.email git@zhaochunqi.com
  - cd public
  - git add --all
  - git commit -m "$(curl -s http://whatthecommit.com/ | awk '/<p>/ {sub("<p>", ""); print }')"
  - git remote set-url --add --push origin https://${GH_OAUTH_TOKEN}@github.com/${GH_OWNER}/${GH_PROJECT_NAME}
  - git remote set-url --add --push origin https://${coding_account}:${coding_passwd}@git.coding.net/harchiko/harchiko.git
  - git push -f origin master > /dev/null 2>&1
```

这里面除了global这里其他都不难懂，看一下travis的生命周期就全明白了。
