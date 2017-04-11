layout: post
title: 简单的 Git 入门
date: 2015-2-13 22:31:00
tags: 
- 教程
- 知识管理
- Git
category: Hexo 
description: 在使用 Hexo 作为博客程序的过程中使用了 Git 这个分布式版本控制系统。现在已经初步入门了。之前一直都没有深入去研究，但是今天上午突然想要把所有的东西都同步到某网盘上以免本地数据出现差错导致博客从头再来。

---

##前言

在使用 Hexo 作为博客程序的过程中使用了 Git 这个分布式版本控制系统。现在已经初步入门了。之前一直都没有深入去研究，但是今天上午突然想要把所有的东西都同步到某网盘上以免本地数据出现差错导致博客从头再来。思来想去还是放弃了将整个博客程序目录放到 OneDrive 下了，其原因一是整个文件夹的文件非常散碎个数非常多原因二是博客调试时会频繁删除生成文件。之后便选择了更加正宗的版本控制系统 Git 。因本人对码代码并非专业也无需团队协作所以本人只是用 Git 配合 GitHub 把一些需要备份的东西备份到云端防止文件丢失。
 
##配置 Git 以及 GitHub
 
###下载并安装 Git 
[下载地址](http://git-scm.com/download/)        
###配置 SSH Key
* 检查 SSH Key
在任意文件夹右键，找到 Git Bash ,打开 Git Bash ，输入如下命令

```
cd ~/.SSH
```

如果提示：No such file or directory 说明你是第一次使用 Git 。
* 生成新的 SSH Key

```
ssh-keygen -t rsa -C "邮件地址@youremail.com"
```
注意 : 此处的邮箱地址，你可以输入自己的邮箱地址；此处的「-C」的是大写的「C」。
提示：

```
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/your_user_directory/.ssh/id_rsa):
```
回车就可以。
然后系统会要你输入密码：
```
Enter passphrase (empty for no passphrase):<输入加密串>
Enter same passphrase again:<再次输入加密串>
```
在回车中会提示你输入一个密码，这个密码会在你提交项目时使用，如果为空的话提交项目时则不用输入。
这个设置是防止别人往你的项目里提交内容。

注意 ：输入密码的时候没有 * 字样的，你直接输入就可以了。

密码提交完成后会提示你已经成功生成 SSH 并且告诉你保存的位置以及 Key 的相关信息。

###添加SSH Key到 GitHub

在本机设置 SSH Key 之后，需要添加到 GitHub 上，以完成SSH链接的设置。
* 打开本地 `C:\User \你的用户名.shh\id_rsa.pub` 文件(windows操作系统)。此文件里面内容为刚才生成的密钥。
如果看不到这个文件，你需要设置显示隐藏文件。准确的复制这个文件的内容（用记事本打开即可），才能保证设置的成功。
* 登陆 Github 系统。点击右上角的 Account Settings—->SSH Public keys —-> add another public keys。
* 把你本地生成的密钥复制到里面（key文本框中）， 点击 add key 就ok了。

###测试 Git 是否成功链接 GitHub 

```
ssh -T git@github.com
```

如果是下面的反馈：

```
The authenticity of host 'github.com (207.97.227.239)' can't be established.
RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)?
```

不要紧张，输入 yes 就好，然后会看到：

```
Hi cnfeat! You've successfully authenticated, but GitHub does not provide shell access.
```

###设置用户信息
现在你已经可以通过 SSH 链接到 GitHub 了，还有一些个人信息需要完善的。
Git 会根据用户的名字和邮箱来记录提交。 GitHub 也是用这些信息来做权限的处理，
输入下面的代码进行个人信息的设置，把名称和邮箱替换成你自己的，名字必须是你的真名，而不是 GitHub 的昵称。
（此处的姓名是作为记录提交信息的，是为了方便识别提交代码的人的）

```
git config --global user.name "yourame"//用户名
git config --global user.email  "yourame@gmail.com"//填写自己的邮箱
```

##克隆一个远程代码库或者是生成一个空的库

###克隆远程库

```
git clone git@github.com:yourname/yourrespositoryname.git
```

上面用的是 SSH 协议，也可以使用 HTTPS 协议，如果使用 HTTPS 协议必须每次提交口令。

###生成一个空的库

到你想要作为 Git 仓库的文件夹下执行如下命令。
如果你使用 Windows 系统，为了避免遇到各种莫名其妙的问题，请确保目录名（包括父目录）不包含中文。

```
git init
```

成功后会在当前目录下生成一个 `.git` 的目录。这个目录是 Git 来跟踪管理版本库的，
没事千万不要手动修改这个目录里面的文件，不然改乱了，就把 Git 仓库给破坏了。

##管理修改

工作区（Working Directory）：就是你在电脑里能看到的目录。

版本库（Repository）：工作区有一个隐藏目录.git，这个不算工作区，而是Git的版本库。

暂存区（stage或者叫index）: Git的版本库里存了很多东西，其中最重要的就是称为 stage（或者叫 index）的暂存区，
还有Git为我们自动创建的第一个分支 master，以及指向 master 的一个指针叫 HEAD。

* 查看状态：使用 `git status` 命令。通过反馈的信息可以看到目前的状态。
* 添加到暂存区： 使用 `git add filename` 命令可以将文件或文件夹添加到暂存区。
* 提交： 使用 `git commit -m "msg"` 命令将暂存区提交到 `master` 。
* 查看修改： 使用 `git diff HEAD -- filename` 命令可以查看工作区和版本库里面最新版本的区别。
* 丢弃工作区的修改： 使用 `git checkout -- file` 命令可以丢弃工作区的修改，回到 `add` 或 `commit` 时的状态。
* 将暂存区回退到工作区: 使用 `git reset HEAD file` 命令可以将暂存区回退到工作区。参数 `HEAD` 是最新的意思。
* 查看所有版本：使用 `git log` 命令查看所有已提交的版本。
* 版本回退： 使用 `git reset --hard HEAD^` 回退到上一版本。 `HEAD^^` 参数表示上上版本； `HEAD~n` 参数表示 n 个版本；
这个参数也可以用 `commit id` 来替代。

另外：
 `reset` 命令有3种方式：
* `git reset --mixed` ：此为默认方式，不带任何参数的 `git reset` ，即时这种方式，它回退到某个版本，只保留源码，回退 `commit` 和 `index` 信息。
* `git reset --soft` ：回退到某个版本，只回退了 `commit` 的信息，不会恢复到 `index file` 一级。如果还要提交，直接 `commit`即可。
* `git reset --hard` ：彻底回退到某个版本，本地的源码也会变为上一个版本的内容。

每次修改，如果不 `add` 到暂存区，那就不会加入到 `commit` 中。

##分支管理

* 查看分支： `git branch`
* 创建分支： `git branch <name>`
* 切换分支： `git checkout <name>`
* 创建+切换分支： `git checkout -b <name>`
* 合并某分支到当前分支： `git merge <name>`
* 删除分支： `git branch -d <name>`

当 Git 无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。

另外：
用 `git log --graph` 命令可以看到分支合并图。

合并分支时，加上 `--no-ff` 参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，
而 `fast forward` 合并就看不出来曾经做过合并。

##标签管理

* 命令 `git tag <name>` 用于新建一个标签，默认为 `HEAD` ，也可以指定一个 `commit id` ；
* 命令 `git tag -a <tagname> -m "blablabla..."` 可以指定标签信息；
* 命令 `git tag -s <tagname> -m "blablabla..."` 可以用PGP签名标签；
* 命令 `git tag` 可以查看所有标签。


* 命令 `git push origin <tagname>` 可以推送一个本地标签；
* 命令 `git push origin --tags` 可以推送全部未推送过的本地标签；
* 命令 `git tag -d <tagname>` 可以删除一个本地标签；
* 命令 `git push origin :refs/tags/<tagname>` 可以删除一个远程标签。

##参考资料

[Git 教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
[Git版本恢复命令reset ](http://blog.csdn.net/xsckernel/article/details/9021225)