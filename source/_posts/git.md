---
title: Git命令
mathjax: true
categories:
  - 教程
tags: Git
date: 2020-10-31 12:00:43
---



---

> [Git](https://en.wikipedia.org/wiki/Git)是一个开源的分布式版本控制系统，用于在软件开发过程中跟踪源代码的变化。
>
> 本文参考：
>
> - [Git教程-廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/896043488029600)
>
> - [Git官方文档](https://git-scm.com/docs)
>
> - [Git-cheatsheet](https://gitee.com/liaoxuefeng/learn-java/raw/master/teach/git-cheatsheet.pdf)

# 基础指令

在相关文件目录下，右键打开Git，其中Git Bash Here为命令行窗口，Git GUI Here为GUI界面，这里主要是针对命令行窗口。

```
$ mkdir learngit	# 创建learngit文件夹     $ torch xx.txt  表示创建xx.txt文件
$ cd learngit	# 进入文件夹
$ git init    # 将当前文件夹初始化为Git仓库
```

```
$ git add <file>  #　添加特定文件
$ git add .     # 添加所有改动的文件
$ git commit -m '提交说明'	# 提交当前改动，提交说明要写
$ git push origin master  # 推送到远程库的origin master分支，默认为origin master
$ git status   # 查看当前状态，比如有哪些未提交文件等
$ git diff 		# 查看具体修改了什么内容，即查看working tree和index的区别
$ git diff --cached    # 查看暂存区和最后一次commit的差异
$ git diff HEAD		# 查看工作区和最后一次commit的差异
```

# 工作区和暂存区

- **工作区**(working directory)是本地电脑看到的一个目录，比如learngit文件夹，或者说是项目文件夹。
- 工作区中有.git隐藏文件夹，为Git的版本库，其中包括最重要的**暂存区**(stage/index)。Git会自动创建一个分支master，以及指向master的指针HEAD
- add操作是将文件添加到暂存区，commit操作将暂存区的所有内容提交到当前分支(仓库)，push操作将本地仓库文件推送到远程仓库。

# 版本回退

```
$ git log		# 查看提交记录
$ git log --pretty=oneline		# 简化查看提交记录信息，只显示commit id和提交说明
$ git restore --hard HEAD^  # 回退到上个commit版本，两个^^则表示回退到上上个版本，~n表示n个版本
$ git restore --hard <commit id>  # 回到指定id的版本，id只需要前几位字符，能区分出不同commit即可
$ git reflog		# 查看历史commit id变化情况，方便找到想要回到的版本
$ git log --graph --pretty=oneline --abbrew-commit	# 用图形显示提交记录
```

# 管理修改

> Git管理的是修改，而不是文件

使用add命令添加文件后，如果再次修改，此时提交是不包括第二次修改的。也就是说，每次修改之后都要add之后才能将修改提交到仓库。

# 撤销修改

想要撤销修改的话，要根据不同的情况采取不同的操作：

- **没有git add 时**，使用`git restore --<file>`命令，file修改后没有放到暂存区，使用此命令回到版本库状态；或者修改了已经添加到暂存区的文件，修改为还没有add，使用此命令可以回退到第一次添加时的状态。总之，restore 操作让文件回到最近一次commit或者add时的状态。
- **已经git add到暂存区**，使用`git restore HEAD <file>`，将暂存区的修改撤销，重新放回工作区
- **已经commit到版本库**，使用`git restore --hard HEAD^`回退版本，或者使用回退到指定commit id
- **已经push到远程仓库**，没救了:(

# 删除文件

使用`rm <file>`指令或者其他方法删除文件后，工作区和仓库文件不一致，这时：

- 如果确定是删除文件，则继续下一步操作即可，`git commit -m 'xxx'`指令，提交删除，会删除仓库文件
- 如果是误删，此时仓库还有文件，使用`git restore --<file>`恢复即可

# 添加远程库

在github上创建完仓库(repository)之后，在本地配置好之后，使用push指令可以将本地仓库到远程仓库。前提要求是本地仓库是git仓库，即有init初始化处理、commit等操作。此外，还需要Git配置好SSH环境，参考[教程](https://git-scm.com/book/zh/v2/%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E7%9A%84-Git-%E7%94%9F%E6%88%90-SSH-%E5%85%AC%E9%92%A5)。

本地修改首先提交到本地仓库(无需Internet)，然后push到远程：

```
$ git remote add origin git@github.com:<path>   	# 第一次推送时，需要先关联远程库
$ git push -u origin master     # 第一次push，需要添加-u参数
$ git push 或者git push origin master  # 如果不是第一次push，则无需-u参数，也可以推送到指定分支
```



# 从远程库克隆

SSH方式：

```
$ git clone git@github.com:<path>    # 后面的地址可以在github上直接复制
```

http方式：

```
$ git clone https://github.com/<path>   # 同样，地址可以从github直接复制      
```

> http方式的速度比较慢，ssh方式速度比较快

# 分支管理

## 创建与分支合并

```
$ git branch   # 查看分支
$ git branch <name>   # 创建分支
$ git checkout <name> 或 git switch <name>  # 切换分支
$ git checkout -b <name> 或 git switch -c <name>   # 创建并切换分支
$ git merge <name>   # 合并指定分支到当前分支
$ git branch -d <name>  # 删除指定分支
```

> Git合并、删除、创建分支的速度非常快，鼓励用分支完成任务再合并，这样过程更安全

## 分支管理策略

分支合并时，Git优先用Fastforward模式，这种模式下，删除分支后，会丢掉分支信息。

如果强制禁用Fastforward模式，Git会在merge时生成新的commit，并且在分支历史记录可以看到分支信息：

```
$ git merge --no-ff -m 'commit information' <brach name>
# 因为no-ff合并要创建新的commit，所以使用-m参数加上提交说明
```

团队开发中，master分支仅用于发布新版本，日常开发在各自的分支上。

>2020.10.1起，github默认主分支改名为main，不再是master

## bug分支

当在dev分支工作时，需要临时修复master上一个bug，需要创建临时bug分支，修复、合并，删除分支。

如果dev分支的工作未完成，可以使用`$ git stash`命令保留现场。

```
$ git stash apply stash@{xxx}   # 有多个stash是，恢复指定的stash，stash不删除
$ git stash pop   # 恢复stash同时删掉stash内容
$ git stash list   # 查看stash列表
$ git cherry-pick <commit id>  # 用于将某个提交复制到当前分支
```

## Feature分支

项目中开发新功能/特征，最好用新的分支，每个功能一个feature分支，在上面开发，完成后，合并，最后删除feature分支。

如果feature分支还未合并，就需要删除掉，需要使用-D参数强行删除：

```
$ git branch -D <branch-name>   # 强行删除一个指定的未merge的分支
```

## 多人协作

多人协作的工作模式：

- 用`$ git push origin <branch-name>`推送自己的修改

- 如果推送失败，则因为远程分支比自己的分支更新，需要使用`$ git pull`命令合并，此命令会自动合并，如果pull请求之后提示‘no stracking information’，说明本地分支和远程分支的链接关系没有创建，使用以下命令：

  ```
  $ git branch --set-upstream-to=origin/dev <本地branch>
  # 设置本地分支和远程origin/dev分支的链接
  ```

- 如果合并有冲突，则需要解决冲突，然后提交，push

- 如果没有冲突或者解决掉冲突后，则使用`$ git push origin <branch name>`推送到远程指定分支

命令总结：

```
$ git remote   # 查看远程信息
$ git remote -v  # 查看远程库的详细信息
$ git switch -c <branch-name> origin/<remote-branch-name>  # 从本地创建和远程对应的分支
$ git branch --set-upstream-to=origin/dev <本地branch>  # 将本地分支与远程origin/dev分支链接
```

## Rebase

```
$ git rebase    # 把分叉的提交历史“整理”成一条直线，看上去更直观，缺点是本地的分叉提交会被修改
```

# 标签管理

标签(tag)也是版本库的一个快照，是指向某个commit的指针，tag比直接使用commit更方便。因此，可以使用tag作为版本号。

## 创建标签

```
$ git tag v1.0  # 给当前最新的commit添加标签v1.0
$ git tag <tag-name> <commit-id>  # 给特定的commit添加标签
$ git tag    # 查看所有标签
$ git show v1.0   # 显示标签v1.0的详细信息
$ git tag -a <tag-name> -m '说明' <commit-id>		# 为特定commit添加标签并添加说明信息
  # 这里的-a 用来指定标签名，-m指定说明文字
```

标签不会随着push推送到远程仓库，使用以下命令将tag信息push到远程：

```
$ git push origin <tagname>   # 推送单个标签
$ git push origin --tags     # 推送所有标签
```

## 操作标签

```
$ git tag -d <tag-name>		# 删除一个指定的本地标签
$ git push origin :refs/tags/<tag-name>    # 删除远程仓库的指定标签
```

