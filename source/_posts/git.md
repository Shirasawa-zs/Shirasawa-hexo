---
title: Git简记(很水,建议不看)
date: 2022-10-02 15:05:00
author: scy
img: https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20221002150824.jpg
top: false
hide: false
cover: false
toc: false
summary: 廖雪峰Git简记
categories: 其他开发加点
tags:
  - Git
---

## Git

摆烂的一天，不怎么想写代码，索性看看廖雪峰的Git顺便记录下一些有帮助的东西，以便于自己回忆和查找相关的Git命令

廖雪峰的官方网站:https://www.liaoxuefeng.com/wiki/896043488029600/896067074338496



#### **简介**

**个人理解**

Git是分布式版本控制系统，在我的理解之中，可以这样理解如果一个系统被git管理了，就相当于所有参与此系统的工作人员的所有修改记录都会被记录到git之中，并且保存每一个版本，当我们系统想要去推倒重建时，可以退回版本。当然如果我们参与系统不同模块构建的工作人员也可以通过git进行工作模块的合并，使之真正成为一个系统。



开发者

Linus，同时也是Linux开发者，git是他花了2 weeks的产品

CVS及SVN都是集中式的版本控制系统，而Git是分布式版本控制系统

大致看下截取的一段

集中式版本控制系统，版本库是集中存放在中央服务器的，而干活的时候，用的都是自己的电脑，所以要先从中央服务器取得最新的版本，然后开始干活，干完活了，再把自己的活推送给中央服务器。中央服务器就好比是一个图书馆，你要改一本书，必须先从图书馆借出来，然后回到家自己改，改完了，再放回图书馆。

分布式版本控制系统根本没有“中央服务器”，每个人的电脑上都是一个完整的版本库，这样，你工作的时候，就不需要联网了，因为版本库就在你自己的电脑上。既然每个人电脑上都有一个完整的版本库，那多个人如何协作呢？比方说你在自己电脑上改了文件A，你的同事也在他的电脑上改了文件A，这时，你们俩之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。在实际使用分布式版本控制系统的时候，其实很少在两人之间的电脑上推送版本库的修改，因为可能你们俩不在一个局域网内，两台电脑互相访问不了，也可能今天你的同事病了，他的电脑压根没有开机。因此，分布式版本控制系统通常也有一台充当“中央服务器”的电脑，但这个服务器的作用仅仅是用来方便“交换”大家的修改，没有它大家也一样干活，只是交换修改不方便而已。



**安装Git**

根据教程网站装下就行

安装后的配置

```xml
git config --global user.name "username"
git config --global user.emial "you email for github or others"
```



查看配置

```xml
git config user.name
git config user.email
```

出现你相关信息正确即可





**版本库**

你可以简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”

linux就用相关指令创建仓库，windows就图形化界面操作，在文件目录下输入如下(windows就git bush)

```xml
git init
```

`.git文件`这个目录是Git来跟踪管理版本库的，没事千万不要手动修改这个目录里面的文件，不然改乱了，就把Git仓库给破坏了

所有的版本变动都是针对文本文件，而图片、视频这些二进制文件，虽然也能由版本控制系统管理，但没法跟踪文件的变化，只能把二进制文件每次改动串起来，也就是只知道图片从100KB改成了120KB，但到底改了啥，版本控制系统不知道，也没法知道。

**千万不要使用Windows自带的记事本编辑任何文本文件**，他们自作聪明地在每个文件开头添加了0xefbbbf（十六进制）的字符，你会遇到很多不可思议的问题



**提交到仓库**

1.新建redme.md，内容如下,位置在.git痛击目录或者子目录

```xml
Git is version control system
Git is fress software
```

2.`git add`添加到仓库

```axml
git add readme.md

git add . //添加所有文件
```

3.用命令`git commit`告诉Git，把文件提交到仓库

```xml
git commit -m "first try to commit"
```

`-m`后面输入的是本次提交的说明，可以输入任意内容，当然最好是有意义的



ERROR

```xml
Q：输入git add readme.txt，得到错误：fatal: not a git repository (or any of the parent directories)。

A：Git命令必须在Git仓库目录内执行（git init除外），在仓库目录外执行是没有意义的。

Q：输入git add readme.txt，得到错误fatal: pathspec 'readme.txt' did not match any files。

A：添加某个文件时，该文件必须在当前目录下存在，用ls或者dir命令查看当前目录的文件，看看文件是否存在，或者是否写错了文件名。
```



#### 相关核心操作

版本回退

git log查看提交日志

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20221002170851.png)

回退版本

首先，Git必须知道当前版本是哪个版本，在Git中，用`HEAD`表示当前版本，也就是最新的提交`1094adb...`（注意我的提交ID和你的肯定不一样），上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个`^`比较容易数不过来，所以写成`HEAD~100`。

```XML
git reset --hard HEAD^
```

查看所有版本id

```XML
git reflog


Jack@LAPTOP-456PN4D3 MINGW64 /f/酸菜鱼/Test_Git (master)
$ git reflog
0e59a6a (HEAD -> master) HEAD@{0}: commit: Third
5dc3797 HEAD@{1}: commit: Second
fa01484 HEAD@{2}: commit (initial): first
```

返回指定版本

```xml
git reset --hard 1094a
```

老师总结

- `HEAD`指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令`git reset --hard commit_id`。
- 穿梭前，用`git log`可以查看提交历史，以便确定要回退到哪个版本。
- 要重返未来，用`git reflog`查看命令历史，以便确定要回到未来的哪个版本。

**工作区和暂存区概念**

![git-repo](https://www.liaoxuefeng.com/files/attachments/919020037470528/0)





**其他信息**

管理修改

Git跟踪并管理的是修改，而非文件，所以每次修改都需要先git add后再考虑commit



撤销修改

`git checkout -- file`可以丢弃工作区的修改

```xml
$ git checkout -- readme.txt
```

命令`git checkout -- readme.txt`意思就是，把`readme.txt`文件在工作区的修改全部撤销，这里有两种情况：

一种是`readme.txt`自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；

一种是`readme.txt`已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

总之，就是让这个文件回到最近一次`git commit`或`git add`时的状态。

用命令`git reset HEAD <file>`可以把暂存区的修改撤销掉（unstage），重新放回工作区：

```xml
$ git reset HEAD readme.txt
Unstaged changes after reset:
M	readme.txt
```

作者总结

场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git checkout -- file`。

场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令`git reset HEAD <file>`，就回到了场景1，第二步按场景1操作。

场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考[版本回退](https://www.liaoxuefeng.com/wiki/896043488029600/897013573512192)一节，不过前提是没有推送到远程库。



#### 远程仓库

1.github创建一个仓库

2.本地仓库关联远程仓库

```xml
 git remote add origin git@github.com:michaelliao/learngit.git
```

3.提交仓库

```java
git push -u origin master
```

4.删除远程仓库

```xml
$ git remote -v
origin  git@github.com:michaelliao/learn-git.git (fetch)
origin  git@github.com:michaelliao/learn-git.git (push)


git remote rm origin
```

5.克隆仓库

```xml
$ git clone git@github.com:michaelliao/gitskills.git
Cloning into 'gitskills'...
remote: Counting objects: 3, done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 3
Receiving objects: 100% (3/3), done.
```



#### 分支

查看分支：`git branch`

创建分支：`git branch <name>`

切换分支：`git checkout <name>`或者`git switch <name>`

创建+切换分支：`git checkout -b <name>`或者`git switch -c <name>`

合并某分支到当前分支：`git merge <name>`

删除分支：`git branch -d <name>`



创建并切换到新的`dev`分支，可以使用：

```
$ git switch -c dev
```

直接切换到已有的`master`分支，可以使用：

```
$ git switch master
```

![git-br-initial](https://www.liaoxuefeng.com/files/attachments/919022325462368/0)



![git-br-on-master](https://www.liaoxuefeng.com/files/attachments/919022533080576/0)





![git-br-rm](https://www.liaoxuefeng.com/files/attachments/919022479428512/0)

实现上图的合并

```XML
1$ git checkout -b dev
Switched to a new branch 'dev'

2$ git branch
* dev
  master

3$ git add readme.txt 
4$ git commit -m "branch test"
[dev b17d20e] branch test
 1 file changed, 1 insertion(+)

5$ git checkout master
Switched to branch 'master'


6$ git merge dev
Updating d46f35e..b17d20e
Fast-forward
 readme.txt | 1 +
 1 file changed, 1 insertion(


7$ git branch -d dev
Deleted branch dev (was b17d20e).

8$ git branch
* master
```

基本概念就这些，剩下的可以直接去看git实操