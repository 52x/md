---
title: 开发者日常使用的 Git 命令
date: 2014-05-22 16:31:13
tags:
  - Git
categories:
  - Tools
  - VCS
---

原文链接： [Everyday GIT With 20 Commands Or So](https://www.kernel.org/pub/software/scm/git/docs/everyday.html)   翻译： [ 伯乐在线 ](http://blog.jobbole.com) - [cjpan](http://blog.jobbole.com/author/cjpan/)
译文链接： <http://blog.jobbole.com/54184/> （下文在原译文基础上有修正补充）

<span style="color:red;">原文（英）最后更新时间：Last updated 2013-02-15 15:53:17 UTC</span>

这些命令分四种类型：①不需要和其他开发者协作的独立开发者，会经常用到 `git init`、`git show branch`、`git commit` 等命令；②需要和其他人协作的开发者，会常用到 `git clone`、`git push`、`git pull`、`git format patch`；③在项目中负责接收其他开发者发来更新的核心开发者，会常用到 `git am`、`git pull`、`git format patch`、`git revert`、`git push`；④ 代码仓库管理员常用 `git daemon`、`git shell` ……

对于任何想做提交的人来说，甚至对于某位单独工作的人来说，【个人开发者（单独开发）】部分命令都是必不可少的。如果你和别人一起工作，你也会需要【个人开发者（参与者）】部分列出的命令。

除了上述的部分，担当【集成人员】角色的人需要知道更多命令。【代码库管理】命令帮助系统管理员负责管理，以及向git代码库提交内容。
<!--more-->

## 个人开发者（单独开发）Individual Developer (Standalone)

单独的个人开发者不会与他人交换修补程序，只用到下列命令，独自在单独的代码库上工作：

*   [git-init(1)](https://www.kernel.org/pub/software/scm/git/docs/git-init.html) 创建新代码库。to create a new repository.
*   [git-show-branch(1)](https://www.kernel.org/pub/software/scm/git/docs/git-show-branch.html) 查看你在哪里。to see where you are.
*   [git-log(1)](https://www.kernel.org/pub/software/scm/git/docs/git-log.html) 查看发生过什么。to see what happened.
*   [git-checkout(1)](https://www.kernel.org/pub/software/scm/git/docs/git-checkout.html)和[git-branch(1)](https://www.kernel.org/pub/software/scm/git/docs/git-branch.html) 切换分支。to switch branches.
*   [git-add(1)](https://www.kernel.org/pub/software/scm/git/docs/git-add.html) 管理索引文件。to manage the index file.
*   [git-diff(1)](https://www.kernel.org/pub/software/scm/git/docs/git-diff.html)和[git-status(1)](https://www.kernel.org/pub/software/scm/git/docs/git-status.html) 查看你正在做什么。to see what you are in the middle of doing.
*   [git-commit(1)](https://www.kernel.org/pub/software/scm/git/docs/git-commit.html) 将推进当前分支。to advance the current branch.
*   [git-reset(1)](https://www.kernel.org/pub/software/scm/git/docs/git-reset.html)和[git-checkout(1)](https://www.kernel.org/pub/software/scm/git/docs/git-checkout.html) （带路径名参数）放弃（撤销）修改。to undo changes.
*   [git-merge(1)](https://www.kernel.org/pub/software/scm/git/docs/git-merge.html) 合并本地分支。to merge between local branches.
*   [git-rebase(1)](https://www.kernel.org/pub/software/scm/git/docs/git-rebase.html) 维护主题分支。to maintain topic branches.
*   [git-tag(1)](https://www.kernel.org/pub/software/scm/git/docs/git-tag.html) 给已知点打标签。to mark known point.

### 实例 Examples

**用Tar包作为一个新代码库的起始点**
Use a tarball as a starting point for a new repository.
```bash
$ tar zxf frotz.tar.gz
$ cd frotz
$ git init
$ git add . <1>
$ git commit -m "import of frotz source tree."
$ git tag v2.43 <2>
```

1.  添加现目录下的所有文件。
2.  打一个轻量的无注释的标签。

**创建一个主题分支并开发**
Create a topic branch and develop.
```bash
$ git checkout -b alsa-audio <1>
$ edit/compile/test
$ git checkout -- curses/ux_audio_oss.c <2>
$ git add curses/ux_audio_alsa.c <3>
$ edit/compile/test
$ git diff HEAD <4>
$ git commit -a -s <5>
$ edit/compile/test
$ git reset --soft HEAD^ <6>
$ edit/compile/test
$ git diff ORIG_HEAD <7>
$ git commit -a -c ORIG_HEAD <8>
$ git checkout master <9>
$ git merge alsa-audio <10>
$ git log --since='3 days ago' <11>
$ git log v2.43.. curses/ <12>
```

1.  创建(create)一个主题分支。
2.  还原(revert)你在`curses/ux_audio_oss.c`文件里搞砸了的修改。
3.  如果你添加一个新文件，你需要告诉git；之后，如果你使用_`git commit -a`_， 删除和修改就会被(git)捕获。
4.  查看你正在提交什么修改。
5.  提交所有的你已测试文件，包括你的签名。
6.  撤回最后一次提交，保留工作区（working tree）内容。
7.  查看自从上一个不成熟提交后的修改。
8.  沿用原先写过的（注释）信息，重做在之前步骤中撤销了的提交。
9.  切换到主干（master）分支。
10.  合并一个主题分支到主分支。
11.  回顾提交记录；其他限制输出的形式也可以合并包含（combined and include）： _`--max-count=10`_(显示10个提交)，_`--until=2005-12-10`_ 等。
12.  只查看从_`v2.43`_标签开始以来影响到_`curses/`_目录的修改。

## 个人开发者（参与开发）Individual Developer (Participant)

作为一个团体项目里的参与角色，开发人员需要学习如何与他人沟通，除了那些单独开发者需要掌握的命令以外，还要使用这些命令。

*   [git-clone(1)](https://www.kernel.org/pub/software/scm/git/docs/git-clone.html) 从上游代码库填充你的本地代码库。
from the upstream to prime your local repository.
*   [git-pull(1)](https://www.kernel.org/pub/software/scm/git/docs/git-pull.html)和[git-fetch(1)](https://www.kernel.org/pub/software/scm/git/docs/git-fetch.html) 从“origin”得到最新的上游代码库。
from "origin" to keep up-to-date with the upstream.
*   [git-push(1)](https://www.kernel.org/pub/software/scm/git/docs/git-push.html) 共享代码库，如果你采用cvs风格的代码库工作流的话。
to shared repository, if you adopt CVS style shared repository workflow.
*   [git-format-patch(1)](https://www.kernel.org/pub/software/scm/git/docs/git-format-patch.html) 准备e-mail提交，如果你使用Linux内核风格的公共论坛工作流的话。
to prepare e-mail submission, if you adopt Linux kernel-style public forum workflow.

### 实例 Examples

**复制上游代码库并在其上工作，提交修改到上游代码库。**
Clone the upstream and work on it. Feed changes to upstream.
```bash
$ git clone git://git.kernel.org/pub/scm/.../torvalds/linux-2.6 my2.6
$ cd my2.6
$ edit/compile/test; git commit -a -s <1>
$ git format-patch origin <2>
$ git pull <3>
$ git log -p ORIG_HEAD.. arch/i386 include/asm-i386 <4>
$ git pull git://git.kernel.org/pub/.../jgarzik/libata-dev.git ALL <5>
$ git reset --hard ORIG_HEAD <6>
$ git gc <7>
$ git fetch --tags <8>
```

1.  按需重复。(repeat as needed)
2.  从你的分支中提取补丁文件，用于电子邮件提交。
3.  _`git pull`_命令默认从“_`origin`_”里取得内容并合并到当前的分支中去。
4.  在`pull`之后，立即查看在上游仓库中自上次检出之后提交的修改（日志），仅查看关注的部分。
5.  从一个指定代码库的一个指定分支获取内容并合并。
6.  撤销（revert）`pull`操作。
7.  从撤销的`pull`操作中回收残存的对象。（代码库清理）
8.  不时地，从_`origin`_中获取官方的标签，并保存于_`.git/refs/tags/`_。

**推送到另一个代码库**
Push into another repository.
```bash
satellite$ git clone mothership:frotz frotz <1>
satellite$ cd frotz
satellite$ git config --get-regexp '^(remote|branch)\.' <2>
remote.origin.url mothership:frotz
remote.origin.fetch refs/heads/*:refs/remotes/origin/*
branch.master.remote origin
branch.master.merge refs/heads/master
satellite$ git config remote.origin.push \
           master:refs/remotes/satellite/master <3>
satellite$ edit/compile/test/commit
satellite$ git push origin <4>

mothership$ cd frotz
mothership$ git checkout master
mothership$ git merge satellite/master <5>
```

1.  `mothership`机器的`home`目录下有一个`frotz`代码库；复制它（clone from it）以在`satellite`机器上开始一个代码库。
2.  复制（`clone`）操作默认设定这些配置变量。它安排_`git pull`_去抓取并保存`mothership`机器上的分支到本地的_`remotes/origin/*`_ 跟踪分支。
3.  安排_`git push`_去推送本地的主（_`master`_）分支到`mothership`机器的_`remotes/satellite/master`_分支
4.  `push`操作会在`mothership`机器的_`remotes/satellite/master`_的远程跟踪分支上存储我们的工作。你可以用此作为一个备份方法。
5.  在`mothership`机器上，将`satellite`机器上已完成的工作合并到`master`分支中。

**分支的特定标签**
Branch off of a specific tag.
```bash
$ git checkout -b private2.6.14 v2.6.14 <1>
$ edit/compile/test; git commit -a
$ git checkout master
$ git format-patch -k -m --stdout v2.6.14..private2.6.14 |
  git am -3 -k <2>
```

1.  创建一个私有分支，基于熟知（但稍许过时的）标签。
2.  在还没有正式“合并（merging）”的情况下，向前移植`private2.6.14`分支所有的修改到`master`分支上。

## 集成人员 Integrator

在一个团队项目中担任集成者的是一名相当重要的人员，他/她接收其他人的修改，评审并集成然后发布最终结果供其他人使用；除了参与者需要的那些命令之外，还需要使用以下这些命令。

*   [git-am(1)](https://www.kernel.org/pub/software/scm/git/docs/git-am.html) 采用你的贡献者邮寄过来的补丁文件。
to apply patches e-mailed in from your contributors.*   [git-pull(1)](https://www.kernel.org/pub/software/scm/git/docs/git-pull.html) 从你的可信任的助手处合并内容。
to merge from your trusted lieutenants.
*   [git-format-patch(1)](https://www.kernel.org/pub/software/scm/git/docs/git-format-patch.html) 准备并向你的贡献者发送建议选项。
to prepare and send suggested alternative to contributors.*   [git-revert(1)](https://www.kernel.org/pub/software/scm/git/docs/git-revert.html) 撤销不好的提交。
to undo botched commits.*   [git-push(1)](https://www.kernel.org/pub/software/scm/git/docs/git-push.html) 发布最新的内容。
to publish the bleeding edge.

### 实例 Examples

**我典型的GIT一天**
My typical GIT day.
```bash
$ git status <1>
$ git show-branch <2>
$ mailx <3>
& s 2 3 4 5 ./+to-apply
& s 7 8 ./+hold-linus
& q
$ git checkout -b topic/one master
$ git am -3 -i -s -u ./+to-apply <4>
$ compile/test
$ git checkout -b hold/linus && git am -3 -i -s -u ./+hold-linus <5>
$ git checkout topic/one && git rebase master <6>
$ git checkout pu && git reset --hard next <7>
$ git merge topic/one topic/two && git merge hold/linus <8>
$ git checkout maint
$ git cherry-pick master~4 <9>
$ compile/test
$ git tag -s -m "GIT 0.99.9x" v0.99.9x <10>
$ git fetch ko && git show-branch master maint 'tags/ko-*' <11>
$ git push ko <12>
$ git push ko v0.99.9x <13>
```

1.  查看我正在做什么，如果有的话。
2.  查看我拥有的主题分支，并考虑它们的完成度。
3.  读邮件，保存合适的，并且保存那些尚未完成的。
4.  采用它们，交互式地，带着我的签名。
5.  按需创建主题分支，还是由我签名采用。
6.  为内部的还未合并到主分支，也没有作为稳定分支的一部分公开的主题分支重定基线。
7.  从接下来开始，每次都重置_pu_。
8.  合并仍然在料理中的主题分支
9.  向后移植（backport）极其重要的修正。
10.  创建一个签名的标签。
11.  确保我不会意外将主分支回滚到我已经推出来的内容。简写的_`ko`_指向我在kernel.org上已有的代码库里，看起来像这样：
```bash
$ cat .git/remotes/ko
URL: kernel.org:/pub/scm/git/git.git
Pull: master:refs/tags/ko-master
Pull: next:refs/tags/ko-next
Pull: maint:refs/tags/ko-maint
Push: master
Push: next
Push: +pu
Push: maint
```
在从_`git show-branch`_的输出里，主分支（_`master`_）应该包含_`ko-master`_所有的内容，并且_`next`_应该包含_`ko-next`_所有的内容。
12\.  推送最新内容
13\.  推送标签

## 代码库管理 Repository Administration

代码库管理员使用下列工具来设置及维护开发者对代码库的访问。

*   [git-daemon(1)](https://www.kernel.org/pub/software/scm/git/docs/git-daemon.html) 允许匿名者从代码库下载。
to allow anonymous download from repository.*   [git-shell(1)](https://www.kernel.org/pub/software/scm/git/docs/git-shell.html) 可以被用作为限制登录shell，用于共享中央代码库的用户。
can be used as a _restricted login shell_ for shared central repository users.
[update hook howto](https://www.kernel.org/pub/software/scm/git/docs/howto/update-hook-example.txt) 有一个很好的管理共享中央代码库的实例。

### 实例 Examples

**我们假设下面的内容均在/etc/services目录下**
We assume the following in /etc/services
```bash
$ grep 9418 /etc/services
git             9418/tcp                # Git Version Control System
```

**从inetd中运行git-daemon来服务于/pub/scm**
Run git-daemon to serve /pub/scm from inetd.
```bash
$ grep git /etc/inetd.conf
git     stream  tcp     nowait  nobody \
  /usr/bin/git-daemon git-daemon --inetd --export-all /pub/scm
```
实际的配置应该在1行里。

**从xinetd运行git-daemon来服务于/pub/scm**
Run git-daemon to serve /pub/scm from xinetd.
```bash
$ cat /etc/xinetd.d/git-daemon
# default: off
# description: The git server offers access to git repositories
service git
{
        disable = no
        type            = UNLISTED
        port            = 9418
        socket_type     = stream
        wait            = no
        user            = nobody
        server          = /usr/bin/git-daemon
        server_args     = --inetd --export-all --base-path=/pub/scm
        log_on_failure  += USERID
}
```
检查`xinetd(8)`文档并设置，这个文档来自于Fedora系统。其他也许会不一样。

**授予开发者只推/拉访问操作权限**
Give push/pull only access to developers.
```bash
$ grep git /etc/passwd <1>
alice:x:1000:1000::/home/alice:/usr/bin/git-shell
bob:x:1001:1001::/home/bob:/usr/bin/git-shell
cindy:x:1002:1002::/home/cindy:/usr/bin/git-shell
david:x:1003:1003::/home/david:/usr/bin/git-shell
$ grep git /etc/shells <2>
/usr/bin/git-shell
```

1.  登录`shell`被设置到`/usr/bin/git-shell`, 不允许_`git push`_和_`git pull`_以外的任何操作。用户需要获得一个访问此机器的`ssh`权限。
2.  在许多发布版本中，`/etc/shells`需要列出作为一个登录`shell`所使用的程序。

**CVS风格的共享代码库**
CVS-style shared repository.
```bash
$ grep git /etc/group <1>
git:x:9418:alice,bob,cindy,david
$ cd /home/devo.git
$ ls -l <2>
  lrwxrwxrwx   1 david git    17 Dec  4 22:40 HEAD -> refs/heads/master
  drwxrwsr-x   2 david git  4096 Dec  4 22:40 branches
  -rw-rw-r--   1 david git    84 Dec  4 22:40 config
  -rw-rw-r--   1 david git    58 Dec  4 22:40 description
  drwxrwsr-x   2 david git  4096 Dec  4 22:40 hooks
  -rw-rw-r--   1 david git 37504 Dec  4 22:40 index
  drwxrwsr-x   2 david git  4096 Dec  4 22:40 info
  drwxrwsr-x   4 david git  4096 Dec  4 22:40 objects
  drwxrwsr-x   4 david git  4096 Nov  7 14:58 refs
  drwxrwsr-x   2 david git  4096 Dec  4 22:40 remotes
$ ls -l hooks/update <3>
  -r-xr-xr-x   1 david git  3536 Dec  4 22:40 update
$ cat info/allowed-users <4>
refs/heads/master       alice\|cindy
refs/heads/doc-update   bob
refs/tags/v[0-9]*       david
```

1.  把开发者置于同一`git`组中。
2.  将共享代码库配为组可写。
3.  使用 Documentation/howto/ 中Carl的`update-hook`实例作为分支策略控制。
4.  alice和cindy可以推送到主分支（master），只有bob可以推送进`doc-update`。david是发布经理，并且是唯一可以创建并推送版本标签的人。

**支持dumb协议传送的HTTP服务器**
HTTP server to support dumb protocol transfer.
```bash
dev$ git update-server-info <1>
dev$ ftp user@isp.example.com <2>
ftp> cp -r .git /home/user/myproject.git
```

1.  确保你的 `info/refs` 和 `objects/info/packs` 是最新的。
2.  上传到由你的ISP托管的公共HTTP服务器。
