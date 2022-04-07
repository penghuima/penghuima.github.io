---
layout: post
title: "Git 从入门到精通"
categories: Git
tags: Git
---

* content
{:toc}


## Git 配置多个github账户

- 首先我们需要在`C:\Users\用户名\.ssh`（windows用户）下，生成两对 `ssh key` (公钥、私钥)

- 然后在 `ssh` 目录下创建 `config` 文件并配置内容如下

  ```bash
  # user one
  Host penghuima   #Host myhost（这里是自定义的host简称，以后连接远程服务器就可以用命令ssh myhost）
  HostName github.com  #HostName 主机名可用ip也可以是域名(如:github.com或者bitbucket.org)
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa
  
  # user two 
  Host nudtpcm
  HostName github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa_two
  ```

- 在GitHub 账户中部署添加 `ssh` 公钥

- 远程测试

  ```bash
  ssh -T git@penghuima  #在config文件里配置host的命名
  ssh -T git@nudtpcm
  ```

<!--more-->

### 正确使用说明

由于是配置多个github账户，因此我们在commit的时候需要注意签名设置，以及push的时候权限等问题。

#### commit的时候签名切换

​	为每个克隆下来的仓库设置局部的用户名和邮箱

```bash
# 取消全局 用户名/邮箱 配置
git config --global --unset user.name
git config --global --unset user.email

# 单独为每个repo设置 用户名/邮箱
git config user.name "penghuima" ; git config user.email "2992426202@qq.com"
git config user.name "nudtpcm" ; git config user.email "maph9916@163.com"
```

#### push的时候权限等问题

​	使用 ssh 方式克隆代码 

- 原来的写法

  ```bash
  git clone git@github.com: one的用户名/learngit.git
  ```

- 现在的写法

  ```bash
  git clone git@penghuima: one的用户名/learngit.git  #@后面跟 HOST 命名
  git clone git@nudtpcm: two的用户名/learngit.git
  ```

  **必须要这样 ssh 克隆，不然push的时候会遇到权限问题，而无法成功**

  > [403错误解决方案](https://blog.csdn.net/qiannz/article/details/121118042)
  >
  > [想要弹出输入用户名、密码（填token序列）窗口](https://www.jianshu.com/p/912fe8c95908)：`git config --global --unset credential.helper`
  >
  > [阻止github每次push都要输入账号密码](https://blog.csdn.net/weixin_44523860/article/details/122389041)：`git config --global credential.helper store`

参考

> [一台电脑上的git同时使用两个github账户](https://blog.csdn.net/qq_33254766/article/details/122941664)
>
> [Git配置多个github账号](https://blog.51cto.com/u_4160094/2912836)

## Git 图解

> [原文](http://marklodato.github.io/visual-git-guide/index-zh-cn.html)
>
> [git-recipes](https://github.com/geeeeeeeeek/git-recipes)

### 基本用法
![image-20220405232642216](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220405232642.png)

上面的四条命令在工作目录、stage 缓存(也叫做索引)和 commit 历史之间复制文件。

* `git add files` 把工作目录中的文件加入 stage 缓存
* `git commit` 把 stage 缓存生成一次 commit，并加入 commit 历史
* `git reset -- files` 撤销最后一次 `git add files`，你也可以用 `git reset` 撤销所有 stage 缓存文件
* `git checkout -- files` 把文件从 stage 缓存复制到工作目录，用来丢弃本地修改

你可以用 `git reset -p`、`git checkout -p` 或 `git add -p` 进入交互模式，也可以跳过 stage 缓存直接从  commit历史取出文件或者直接提交代码。

![enter image description here](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220405232720.png)

* `git commit -a ` 相当于运行 `git add` 把所有当前目录下的文件加入 stage 缓存再运行 `git commit`。
* `git commit files` 进行一次包含最后一次提交加上工作目录中文件快照的提交，并且文件被添加到 stage 缓存。
* `git checkout HEAD -- files` 回滚到复制最后一次提交。

### 约定

后文中以下面的形式使用图片：

![enter image description here](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220405232755.png)

绿色的5位字符表示提交的 ID，分别指向父节点。分支用橙色显示，分别指向特定的提交。当前分支由附在其上的 `HEAD` 标识。

这张图片里显示最后 5 次提交，`ed489` 是最新提交。 `main` 分支指向此次提交，另一个 `stable` 分支指向祖父提交节点。

### 命令详解

#### Diff

有许多种方法查看两次提交之间的变动，下面是其中一些例子。

![enter image description here](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220405232844.png)

#### Commit

提交时，Git 用 stage 缓存中的文件创建一个新的提交，并把此时的节点设为父节点。然后把当前分支指向新的提交节点。下图中，当前分支是 `main`。

在运行命令之前，`main` 指向 `ed489`，提交后，`main` 指向新的节点`f0cec` 并以 `ed489` 作为父节点。
![image-20220405233101740](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220405233101.png)

即便当前分支是某次提交的祖父节点，Git 会同样操作。下图中，在 `main` 分支的祖父节点 `stable` 分支进行一次提交，生成了 `1800b`。

**这样，`stable `分支就不再是 `main` 分支的祖父节点。此时，[merge](#merge) (合并) 或者 [rebase](#rebase) （衍合）是必须的。**
![image-20220405233606063](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220405233606.png)

如果想更改一次提交，使用 `git commit --amend`。Git 会使用与当前提交相同的父节点进行一次新提交，旧的提交会被取消。
![image-20220405233736337](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220405233736.png)

另一个例子是[分离HEAD提交](#detached)，在后面的章节中介绍。

#### Checkout

**`git checkout` 命令用于从历史提交（或者 stage 缓存）中拷贝文件到工作目录，也可用于切换分支。**

当给定某个文件名（或者打开 `-p` 选项，或者文件名和-p选项同时打开）时，Git 会从指定的提交中拷贝文件到 stage 缓存和工作目录。比如，`git checkout HEAD~ foo.c` 会将提交节点 `HEAD~`（即当前提交节点的父节点）中的 `foo.c` 复制到工作目录并且加到 stage 缓存中。如果命令中没有指定提交节点，则会从 stage 缓存中拷贝内容。注意当前分支不会发生变化。

![enter image description here](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220405234039.png)

当不指定文件名，而是给出一个（本地）分支时，那么 `_HEAD_` 标识会移动到那个分支（也就是说，我们「切换」到那个分支了），然后 stage 缓存和工作目录中的内容会和 `HEAD` 对应的提交节点一致。新提交节点（下图中的 `a47c3`）中的所有文件都会被复制（到 stage 缓存和工作目录中）；只存在于老的提交节点（`ed489`）中的文件会被删除；不属于上述两者的文件会被忽略，不受影响。

![enter image description here](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220405234211.png)
如果既没有指定文件名，也没有指定分支名，而是一个标签、远程分支、SHA-1 值或者是像 `main~3` 类似的东西，就得到一个匿名分支，称作 `detached HEAD`（被分离的 `HEAD` 标识）。这样可以很方便地在历史版本之间互相切换。比如说你想要编译 1.6.6.1 版本的 Git，你可以运行 `git checkout v1.6.6.1`（这是一个标签，而非分支名），编译，安装，然后切换回另一个分支，比如说 `git checkout mian`。然而，当提交操作涉及到「分离的 HEAD」时，其行为会略有不同，详情见在[下面](#detached)。

![enter image description here](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220405234314.png)

#### detached HEAD(分离的HEAD)

> HEAD 标识处于分离状态时的提交操作

当 `HEAD` 处于分离状态（不依附于任一分支）时，提交操作可以正常进行，但是不会更新任何已命名的分支。你可以认为这是在更新一个匿名分支。

![enter image description here](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220405234433.png)

一旦此后你切换到别的分支，比如说 `_master_`，那么这个提交节点（可能）再也不会被引用到，然后就会被丢弃掉了。注意这个命令之后就不会有东西引用 `_2eecb_`。
![enter image description here](http://marklodato.github.io/visual-git-guide/checkout-after-detached.svg)
但是，如果你想保存这个状态，可以用命令 `git checkout -b name` 来创建一个新的分支。

![enter image description here](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220405234454.png)

#### Reset

`git reset` 命令把当前分支指向另一个位置，并且有选择的变动工作目录和索引。也用来在从历史commit历史中复制文件到索引，而不动工作目录。

如果不给选项，那么当前分支指向到那个提交。如果用 `--hard` 选项，那么工作目录也更新，如果用 `--soft` 选项，那么都不变。

![enter image description here](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220405234612.png)

如果没有给出提交点的版本号，那么默认用 `HEAD`。这样，分支指向不变，但是索引会回滚到最后一次提交，如果用 `--hard` 选项，工作目录也同样。

![enter image description here](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220405234701.png)

如果给了文件名(或者 `-p` 选项), 那么工作效果和带文件名的[checkout](#checkout)差不多，除了索引被更新。

![enter image description here](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220405234735.png)

#### Merge

`git merge` 命令把不同分支合并起来。合并前，索引必须和当前提交相同。如果另一个分支是当前提交的祖父节点，那么合并命令将什么也不做。

另一种情况是如果当前提交是另一个分支的祖父节点，就导致 **`fast-forward` 合并**。指向只是简单的移动，并生成一个新的提交。

![image-20220406001325219](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220406001325.png)

否则就是一次真正的合并。**默认把当前提交（`ed489` 如下所示）和另一个提交（`33104`）以及他们的共同祖父节点（`b325c`）进行一次[三方合并](http://en.wikipedia.org/wiki/Three-way_merge)**。结果是先保存当前目录和索引，然后和父节点 `33104` 一起做一次新提交。

![image-20220406002840572](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220406002840.png)

#### Cherry-Pick

`git cherry-pick` 命令「复制」一个提交节点并在当前分支做一次完全一样的新提交。

![enter image description here](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220406002857.png)

#### Rebase

`git rebase` 是合并命令的另一种选择。合并把两个父分支合并进行一次提交，提交历史不是线性的。rebase 在当前分支上重演另一个分支的历史，提交历史是线性的。

本质上，这是线性化的自动的 [cherry-pick](#cherry-pick)。

![enter image description here](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220406215755.png)

上面的命令都在 `topic` 分支中进行，而不是 `main` 分支，在 `main` 分支上重演，并且把分支指向新的节点。注意旧提交没有被引用，将被回收。

要限制回滚范围，使用 `--onto` 选项。下面的命令在 `main` 分支上重演当前分支从 `169a6` 以来的最近几个提交，即 `2c33a`。

![enter image description here](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220406215827.png)

同样有 `git rebase --interactive` 让你更方便的完成一些复杂操作，比如丢弃、重排、修改、合并提交。

## 奇淫技巧

### checkout检出提交 || 检出文件

`git checkout` 这个命令有三个不同的作用：**检出文件**、**检出提交**和**检出分支**。

- 检出提交和检出分支会使工作目录和这个提交/分支完全匹配。你可以用它来查看项目之前的状态，而不改变当前的状态。
- 检出文件使你能够查看某个特定文件的旧版本，而工作目录中剩下的文件不变。

一旦你回到 master 分支之后，你可以使用 `git revert` 或 `git reset` 来回滚任何不想要的更改。

#### 检出文件

如果你只对某个文件感兴趣，你也可以用 `git checkout` 来获取它的一个旧版本。比如说，如果你只想从之前的提交中查看 `hello.py` 文件，你可以使用下面的命令：

```
git checkout a1e8fb5 hello.py
```

记住，和检出提交不同，这里 *确实* 会影响你项目的当前状态。旧的文件版本会显示为「需要提交的更改」，允许你回滚到文件之前的版本。如果你不想保留旧的版本，你可以用下面的命令检出到最近的版本：

```
git checkout HEAD hello.py
```

> - 版本控制系统背后的思想就是「安全」地储存项目的拷贝，这样你永远不用担心什么时候不可复原地破坏了你的代码库。当你建立了项目历史之后，`git checkout` 是一种便捷的方式，来将保存的快照「加载」到你的开发机器上去。
> - 在另一方面，检出旧文件不影响你仓库的当前状态。你可以在新的快照中像其他文件一样重新提交旧版本。所以，在效果上，`git checkout` 的这个用法可以用来将单个文件回滚到旧版本 。

#### 检出分支

> 最佳实践：每次切换分支前  当前分支一定得是干净的（已提交状态）
>
> 坑：在切换分支时， 如果当前分支上有未暂存的修改（第一次）或者有未提交的暂存（第一次）
>
> ​      分支可以切换成功  但这种操作可能会污染其它分支的三个地方
>
> HEAD
>
> 暂存区
>
> 工作目录

远程分支有一个特别的属性，在你**检出**时自动进入分离 HEAD 状态。Git 这么做是出于不能直接在这些分支上进行操作的原因, 你必须在别的地方完成你的工作, （更新了远程分支之后）再用远程分享你的工作成果。

如果检出远程分支会怎么样呢？

```bash
git checkout o/main; git commit
```

![image-20220404165332516](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220404165339.png)

**正如你所见，Git 变成了分离 HEAD 状态，当添加新的提交时 `o/main` 也不会更新。这是因为 `o/main` 只有在远程仓库中相应的分支更新了以后才会更新。**

### git status

`git status` 命令显示工作目录和缓存区的状态。你可以看到哪些更改被缓存了，哪些还没有，以及哪些还未被 Git 追踪。status 的输出 *不会* 告诉你任何已提交到项目历史的信息。如果你想看的话，应该使用 `git log` 命令。

```
git status
```

列出已缓存、未缓存、未追踪的文件。

### git add 之后想撤销

- git restore --staged <文件/.> 是将暂存区的文件从暂存区撤出，但不会更改文件的内容。

  > git restore 指令使得在工作空间但是不在暂存区的文件撤销更改(内容恢复到没修改之前的状态)

- git reset HEAD  (默认是 --mixed  )从暂存区的文件从暂存区撤出，但不会更改文件的内容，查看状态会提示未暂存（stated）

  > git reset <文件> 直接添加文件名撤销

- git reset --hard HEAD 从暂存区和工作目录中回滚，本地文件的修改也会回复到没修改之前的状态

### 分支合并（merge/rebase）

#### rebase

- ```bash
  git rebase <branch1>
  ```

  将当前分支 rebase 到 `branch1` 后面，**但是，rebase 为原分支上每一个提交创建一个新的提交，重写了项目历史**

- ```bash
  git rebase <branch1> <branch2>
  ```

  将分支2的提交衔接在分支1的后面，这会使所有分支变成线性的，并且衔接之后，对应分支指向原先对应的提交

如果你还没有完全熟悉 `git rebase`，你还可以在一个临时分支中执行 rebase。这样的话，如果你意外地弄乱了你 feature 分支的历史，你还可以查看原来的分支然后重试。

比如说：

```
git checkout feature
git checkout -b temporary-branch
git rebase -i master
# [清理目录]
git checkout master
git merge temporary-branch
```

#### merge

- ```bash
  git merge <branch1>
  ```

  将当前分支与分支1合并，并（HEAD，当前分支）指向新生成的提交

### 交互式rebese (整理自己私有仓库的提交)

#### 合并多个提交

> [git rebase -i 合并多次提交](https://www.jianshu.com/p/201a56ffe9a4)

```bash
git rebase -i <commmitID> #任意选择一个基，整理这个基之后的所有提交
```

它会打开一个文本编辑器，显示所有将被移动的提交：

```bash
pick 33d5b7a Message for commit #1
pick 9480b3d Message for commit #2
pick 5c67e61 Message for commit #3
```

这个列表定义了 rebase 将被执行后分支会是什么样的。更改 `pick` 命令或者重新排序，这个分支的历史就能如你所愿了。比如说，如果你想合并多个提交

```bash
pick 33d5b7a Message for commit #1
s 9480b3d Message for commit #2
s 5c67e61 Message for commit #3

#这里有几个使用说明（前面字母是缩写）：
#p，pick：使用该次提交
#r，reword：使用该次提交，但重新编辑提交信息
#e，edit：使用该次提交，但停止到该次提交
#s，squash：将该commit和前一个commit合并
#f，fixup：将该commit和前一个commit合并，但不保留该提交的注释信息
#x，exec：执行shell命令
#d，drop：丢弃该commit  
```

保存后关闭文件，Git 会根据你的指令来执行 rebase

> 如果想要放弃当前 rebase 操作，用 `git rebase --abort`
>
> 文件冲突后，使用 git rebase --continue继续

## 不要重设公共历史

### 不要使用 git reset 修改公共历史 

当有 `commit` 之后的提交被推送到公共仓库后，你绝不应该使用 `git reset`。发布一个提交之后，你必须假设其他开发者会依赖于它。

移除一个其他团队成员在上面继续开发的提交在协作时会引发严重的问题。当他们试着和你的仓库同步时，他们会发现项目历史的一部分突然消失了。下面的序列展示了如果你尝试重设公共提交时会发生什么。`origin/master` 是你本地 `master` 分支对应的中央仓库中的分支。

![image-20220406164937522](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220406164944.png)

![image-20220406165012431](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220406165012.png)

![image-20220406165030573](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220406165030.png)

**一旦你在重设之后又增加了新的提交，Git 会认为你的本地历史已经和 `origin/master` 分叉了**，同步你的仓库时的合并提交（`merge commit`）会使你的同事困惑。

重点是，确保你只对本地的修改使用 `git reset`，而不是公共更改。如果你需要修复一个公共提交，`git revert` 命令正是被设计来做这个的。

### 谨慎使用 git commit --amend

其实 **git commit --amend** 之所以 `push` 的时候失败我想就是上面的原因吧，需要强制 `push`

reset中提到永远不要重设和其他开发者共享的提交。对于修复也是一样(git commit --amend)：永远不要修复一个已经推送到公共仓库中的提交。

修复过的提交事实上是全新的提交，之前的提交会被移除出项目历史。这和重设公共快照的后果是一样的。如果你修复了其他开发者在之后继续开发的一个提交，看上去他们的工作基础从项目历史中消失了一样。对于在这上面的开发者来说这是很困惑的，而且很难恢复。

### 不要 rebase 公共历史

例子：同时使用 git rebase 和 git merge 来保持线性的项目历史。这是一个确认你的合并都是快速向前的方法。

```bash
git checkout -b new-feature master # 开始新的功能分支
# 编辑文件
git commit -a -m "Start developing a feature"
```

在 feature 分支开发了一半的时候，我们意识到项目中有一个安全漏洞:

```bash
git checkout -b hotfix master   # 基于master分支创建一个快速修复分支
# 编辑文件
git commit -a -m "Fix security hole"
# 合并回master
git checkout master
git merge hotfix    #因为hotfix包含main的全部提交，因此这是一个快速合并，将main指向hotfix
git branch -d hotfix
```

将 hotfix 分支并回之后 master，我们有了一个分叉的项目历史。我们用 rebase 整合 feature 分支以获得线性的历史，而不是使用普通的 git merge。

```bash
git checkout new-feature
git rebase master   #一定是要将别的分支rebase到main分支上，而不是将main分支rebase到别的分支，公共提交是不要改变的
#有冲突解决冲突，new-feature分支上的提交会复制一份，衔接在main分支后面
```

它将 new-feature 分支移到了 master 分支的末端，现在我们可以在 master 上进行标准的快速向前合并了:

```bash
git checkout master
git merge new-feature  #快速合并其它分支
```

###  远程分支

远程分支和本地分支一样，只不过它们代表这些提交来自于其他人的仓库。你可以像查看本地分支一样查看远程分支，但你会处于分离 HEAD 状态（就像查看旧的提交时一样）。你可以把它们视作只读的分支。如果想要查看远程分支，只需要向 `git branch` 命令传入 `-r` 参数。远程分支拥有 remote 的前缀，所以你不会将它们和本地分支混起来。比如，下面的代码片段显示了从 origin 拉取之后，你可能想要查看的分支：

```
git branch -r
# origin/master
# origin/develop
# origin/some-feature
```

同样，你可以用寻常的 `git checkout` 和 `git log` 命令来查看这些分支。如果你接受远程分支包含的更改，你可以使用 `git merge` 将它并入本地分支。

#### git fetch 

当你希望查看其他人的工作进展时，你需要 fetch。**fetch 下来的内容表示为一个远程分支**，因此不会影响你的本地开发。这是一个安全的方式，在整合进你的本地仓库之前，检查那些提交。

这个例子回顾了同步本地和远程仓库 `master` 分支的常见工作流：

```bash
git fetch origin
```

> git fetch <remote> <branch> 只会拉取指定的分支

它会显示会被下载的分支：

```
a1e8fb5..45e66a4 master -> origin/master
a1e8fb5..9e8ab1c develop -> origin/develop
* [new branch] some-feature -> origin/some-feature
```

在下图中，远程分支中的提交显示为方块，而不是圆圈。正如你所见，`git fetch` 让你看到了另一个仓库完整的分支结构。

![image-20220406215740307](https://cdn.jsdelivr.net/gh/penghuima/ImageBed@master/img/blog_file/PicGo-Github-ImgBed20220406215740.png)



执行

```bash
git merge origin/master
```

origin/master 和 master 分支现在指向了同一个提交，你已经和上游的更新保持了同步。

#### git pull

在基于 Git 的协作工作流中，将上游更改合并到你的本地仓库是一个常见的工作。我们已经知道应该使用 `git fetch`，然后是 `git merge`，但是 `git pull` 将这两个命令合二为一。

用法

```
git pull <remote>
```

拉取当前分支对应的远程副本中的更改，并立即并入本地副本。效果和 `git fetch` 后接 `git merge origin/.` 一致。

```
git pull --rebase <remote>
```

和上一个命令相同，但使用 `git rebase` 合并远程分支和本地分支，而不是使用 `git merge`。

> `--rebase` 标记可以用来保证线性的项目历史，防止合并提交（merge commits）的产生。很多开发者倾向于使用 rebase 而不是 merge，因为「我想要把我的更改放在其他人完成的工作之后」
>
> 事实上，使用 `--rebase` 的 pull 的工作流是如此普遍
>
> 你可以直接在配置项中设置它：（我还没有设置）
>
> ```
> git config --global branch.autosetuprebase always # In git < 1.7.9
> git config --global pull.rebase true              # In git >= 1.7.9
> ```
>
> 在运行这个命令之后，所有的 `git pull` 命令将使用 `git rebase` 而不是 `git merge`

#### git push 

```
git push <remote> <branch>
```

将指定的分支推送到 `<remote>` 上，包括所有需要的提交和提交对象。它会在目标仓库中创建一个本地分支。**为了防止你覆盖已有的提交，如果会导致目标仓库非快速向前合并时，Git 不允许你 push**。

==当你运行 `git push origin master` 发布更改时发生的事情。注意，`git push` 和在远程仓库内部运行 `git merge master` 事实上是一样的。(在远程仓库内，remote/master 合并 master)==

> **强制推送**
>
> Git 为了防止你覆盖中央仓库的历史，会拒绝你会导致非快速向前合并的推送请求。所以，如果远程历史和你本地历史已经分叉，你需要将远程分支 pull 下来，在本地合并后再尝试推送。
>
> `--force` 这个标记覆盖了这个行为，让远程仓库的分支符合你的本地分支，删除你上次 pull 之后可能的上游更改。只有当你意识到你刚刚共享的提交不正确，**并用 `git commit --amend` 或者交互式 rebase 修复之后，你才需要用到强制推送**。但是，你必须绝对确定在你使用 `--force` 标记前你的同事们都没有 pull 这些提交。

下面的例子描述了将本地提交推送到中央仓库的一些标准做法。首先，确保你本地的 `master` 和中央仓库的副本是一致的，提前 fetch 中央仓库的副本并在上面 rebase。交互式 rebase 同样是共享之前清理提交的好机会。接下来，`git push` 命令将你本地 `master` 分支上的所有提交发送给中央仓库.

```bash
git checkout master
git fetch origin master
git rebase -i origin/master    #上面这两步可以合为一个 git pull --rebase origin/master
# Squash commits, fix up commit messages etc.
git push origin master
```

因为我们已经确信本地的 `master` 分支是最新的，它应该导致快速向前的合并，`git push` 不应该抛出非快速向前之类的问题。

### pull request

Pull Request 是开发者使用 GitHub 进行协作的利器。这个功能为用户提供了友好的页面，让提议的更改在并入官方项目之前，可以得到充分的讨论。

有如下三种工作流 ：基于功能分支的工作流、gitflow工作流、fork工作流，这里重点讲一下fork工作流

#### fork工作流

首先fork完项目之后，创建本地仓库

```
git clone git clone git@penghuima: one的用户名/repo.git
```

**Fork 工作流需要两个远程连接**

- **一个是中央仓库**
- **另一个是开发者个人的服务端仓库。**

**你可以给这些远端取任何名字，约定的做法是将 `origin` 作为你 fork 后的仓库的远端（运行 `git clone` 是会自动创建）和 `upstream` 作为官方项目。**

```
git remote add upstream https://github.com/maintainer/repo
```

你需要使用上面的命令来创建上游仓库的远程连接。它使得你轻易地保持本地仓库和官方仓库的进展同步。

**开发者和中央仓库保持同步**

```bash
git pull upstream master <分支> --rebase  #待验证命令正确与否   都不是同一个分支啊
```

## 查看分支图

```bash
git log --oneline --graph --all  --decorate  #貌似只需要前三个参数就好了
#或者使用这个 gitk 图形界面命令来输出所有分支信息
gitk --all
```

## 从理论到实践

把 [learnGitBranching](https://github.com/pcottle/learnGitBranching) 的题刷一遍就可以了，这是一个很棒的网站



**相关引用**

> [tutorials](https://www.atlassian.com/git/tutorials)
>
> [git-recipes](https://github.com/geeeeeeeeek/git-recipes)
>
> [git-tips](https://github.com/521xueweihan/git-tips)
>
> *[Git 小白教程](https://rogerdudler.github.io/git-guide/index.zh.html)*
>
> [learnGitBranching](https://github.com/pcottle/learnGitBranching)
>
> [learnGitBranching答案](https://blog.csdn.net/qq_34519487/article/details/107882290)
>
> [一台电脑上的git同时使用两个github账户](https://blog.csdn.net/qq_33254766/article/details/122941664)
>
> [Git配置多个github账号](https://blog.51cto.com/u_4160094/2912836)
>
> [git rebase -i 合并多次提交](https://www.jianshu.com/p/201a56ffe9a4)

