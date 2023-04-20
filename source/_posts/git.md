---
title: Git简记
date: 2022-4-14 15:05:00
author: scy
top: false
hide: false
cover: false
toc: false
summary: Git简记
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









# 1.二次总结

## 1概念介绍

1.GIT介绍和主要作用

GIT是一个开源的分布式版本控制系统

作用是代码共享,回溯版本,追踪信息

GIT和SVN区别 -> 面试题

 

2.GIT概念

工作区(.git不属于工作区)，（缓存区，本地仓库）（版本库），远程仓库

1. 工作区：就是你平时存放项目代码的地方。

位置：一个文件夹通过git init设置成一个git可以管理的文件夹时，这个文件夹里的内

容（除去.git文件夹）就是工作区

2. 仓库区（或版本库），就是安全存放数据的位置，这里面有你提交到所有版本的数据。其中HEAD指向最 新放入仓库的版本。就是工作区有一个隐藏目录.git它不算工作区，而是Git的版本库

3. 暂存区：英文叫 stage 或 index。是用来暂时存放工作区中修改的内容，可以理解为一个中转站。

1）位置：一般存放在 .git 目录下的 index 文件（.git/index）中，所以我们把暂存区有时也叫 作索引（index）。 

2）只是一个文件

3）包含在版本库中

4）为什么需要暂存区：

1.如果没有暂存区，如果想要提交文件就需要一个个修改然后，提交，比较麻烦，但是有了暂存区 就可以一次性将所需要的文件从暂存区 直接修改后提交。 

2.如果没有暂存区，你修改的文件只可以立刻保存到版本库中，但是这样很容易对别人的工作造成影响

4. Head :指向最新放入仓库的版本 

5. master:是我们的主分支。当我们git init后，并不会立刻产生分支。而是我们添加了一个文件，并 git add, git commit后这时我们查看分支情况便可以看到master分支了。是本地仓库一部分。 

6. objects:是git对象库，是用来存储各种创建的对象以及内容. 

7. 远程仓库:托管代码的服务器，常用github 码云 gitlab

 

3.Git安装

依照网上教程无脑安装

 

 

## 2.具体操作

1. GIT本地操作-初始化工作区

1.1初始化当前的文件夹为工作区：git init

1.2如何查看文件状态：git status -> 红色代表当前没有提交到缓存区

1.3进入文件编辑模式 linux系统

vim文件名，然后按字i(前) a(后) o(下一行)插入数据，然后按esc退出吗，最后按:wq保存退出。

1.4查看文件内容：cat文件名(linux系统)

 

 

2. GIT本地操作add与commit

2.1工作区提交缓存区git add readme.txt ->发现这时文件变成绿色，可以提交到本地仓库

2.2缓存区提交本地仓库git commit -m第一次提交'后面提交的内容规范后面会有总结

2.3第一次提交需要填写如下信息

命令git config --global user.email '123456@qq.com' -> 说明:指定邮箱 

命令:git config --global user.name 'suoge' -> 说明:指定操作者

2.4其他总结

添加多个文件 git add [file1] [file2] ...

添加指定目录到暂存区，包括子目录 git add [dir]

添加当前目录下的所有文件到暂存区，不包括被删除的文件 git add .

add时，一个个文件加比较麻烦，可以用下面的命令将所有变动的文件同步至暂存区（新增，修改，删，除）-> git add -A

下面的命令是将所有修改和删除的文件同步至暂存区，不包括新增文件 -> git add -u

 

3. GIT本地操作-差异比较

3.1工作区暂存区比较 -> git diff readme.txt

3,2工作区本地库比较 -> git diff HEAD readme.txt 

3.3暂存区本地库比较 -> git diff --cached readme.txt 

 

4. GIT本地操作-版本回退

4.0每一个版本的前面，都有一长串随即数字，这是每次提交的commit id

4.1查看当前提交日志 -> git log

4.2查看当前提交日志，且显示当前分支的当前版本所在位置 -> git log –decorate

4.3回退到之前版本 -> git reset --hard HEAD^回退一个版本( git reset --hard HEAD~100回退100版本)，此时使用git log发现只有此版本之前的版本

4.4查看所有操作 -> git reflog

4.5版本号 回退到指定版本 -> git reset --hard 版本号

 

5. GIT本地操作-修改撤消

5.1在你提交缓存区前，你突然发现这个修改是有问题的，你打算恢复到原来的样子

撤销工作区修改 -> git checkout 文件名称

撤销缓存区到工作区 -> git reset HEAD readme.txt

## 3.分支操作

提交的本质：记录仓库中一组文件的变化信息（增、删、改）

分支是什么：分支就是多次提交串起来的一条线 

!![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230416171625.png)

 

1   分支操作-分支创建与切换

1.1创建dev分支 -> git branch dev

1.2切换dev分支 -> git checkout dev

1.3查看分支 git branch

1.4 当前在master分支，创建并转到dev分支，在dev分支进行修改文件并提交到缓存区推到本地仓库，dev分支显示文件内容已经被修改，转换到master分支，发现master分支的文件内容未被修改

 

\2.  分支操作-分支合并与删除

2.1合并dev分支 git merge dev -> git merge 分支名

2.2 查看分支情况 git branch

2.3 删除dev分支 -> git branch -d 分支名

 

# 4.GIT远程仓库介绍

Github/Gitee/Gitlib

码云 -> 随便找个教程注册就行

 

 

 

 

 

# 5．GIT远程仓库操作-关联、拉取、推送、克隆

0. 

*本地分支和远程分支的关联关系 ->* git branch -v 

*查看远程仓库* -> git remote -v 

 

1. 我们需要先建立本地仓库 与远程仓库的关系

git remote add origin 远程仓库地址 关联远程仓库

 

2. 拉取：从码云仓库拉取到本地仓库

 《在推送代码前必须先拉取代码，否则无法推送本地仓库代码到码云仓库 》

git pull origin master --allow-unrelated-histories 

首次拉取需要添加:--allow-unrelated-histories 

git pull 后续拉取

 

3. 推送：本地仓库推送到码云仓库

首次推送 -> git push -u origin master

后续推送 -> git push 

强制push -> git push -u origin master **-f**

 

4.克隆

克隆到本地 -git clone https://gitee.com/tiansuo123456/itheim-heima141.git

 

 

# 6. .IDEA中使用GIT-集成GIT

1.额，我一般创建仓库，本地关联远程仓库，然后add, commit, push解决

 

红色 -> 没添加到缓存区

蓝色 -> 没添加到本地仓库

黑色 -> 同步成功

 

2.重点应该是回退版本

文件 -> git -> commit file -> log下的历史提交记录选择 -> Reset开头的 -> 选择hard -> 版本回退成功

 

3.工作区撤销

当我们在工作区编辑代码时候，希望撤销未提交本地仓库的代码时候,在Git中右键 Rollback

4.在IDEA中完成分支创建、合并、删除操作

首先克隆文件，软件右侧手动创建新分支，并切换到这个分支进行编码，编码成功后提交到本地仓库，切换成master分支。在当前分支合并新分支（选择新分支合并）删除新分支

 

5.版本冲突

但是当多个用户对同一个文件交叉修改的时候就尴尬了。A修改完提交一个，B修改完又提交一个，这个 时候A修改完提交，问题来了，如果A提交成功，那么就相当于忽略了B提交的内容。这个时候就要强制 你去处理一下这个问题，这就是我们所说的冲突问题。

比如仓库代码和本地代码冲突

解决冲突：X放弃 >>采用，在主干代码进行选择后，重新提交

 

 

7. IDEA git拉取项目时报 No tracked branch configured for branch master or the branch doesn't exist的提示

选择本地工作空间右键 输入git branch --set-upstream-to=origin/master，再次拉去合并

 

 

8. 多人共享git项目

分为项目，用户，组

新建用户，并加入组中，根据组分配权限，创建并关联项目。

组长对master分支进行保护，组长创建dev分支，实际开发中，组员在dev分支开发，开发完毕由组长进行合并。设置master分支不被允许合并，设置其他分支可以合并和推送。

VCS关联地址。组员切换到dev分支，并在dev分支编码提交到本地仓库。组长在master分支合并。