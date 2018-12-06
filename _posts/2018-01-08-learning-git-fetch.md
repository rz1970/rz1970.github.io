---
title: Git使用技巧之git fetch
category: Git
tags: Git
---

一旦远程主机的版本库有了更新(Git术语叫做commit)，需要将这些更新取回本地，这时就要用到`git fetch`命令。
```
$ git fetch <远程主机名>
```
上面命令将某个远程主机的更新，全部取回本地。默认情况下，`git fetch`取回所有分支(`branch`)的更新。如果只想取回特定分支的更新，可以指定分支名。
```
$ git fetch <远程主机名> <分支名>
```
比如，取回origin主机的master分支。
```
$ git fetch origin master
```
所取回的更新，在本地主机上要用`“远程主机名/分支名”`的形式读取。比如origin主机的master，就要用`origin/master`读取。

`git branch`命令的`-r`选项，可以用来查看远程分支，`-a`选项查看所有分支。
```
$ git branch -r
origin/master

$ git branch -a
* master
  remotes/origin/master
```
上面命令表示，本地主机的当前分支是master，远程分支是origin/master。

取回远程主机的更新以后，可以在它的基础上，使用`git checkout`命令创建一个新的分支。
```
$ git checkout -b newBrach origin/master
```
上面命令表示，在`origin/master`的基础上，创建一个新分支。
此外，也可以使用`git merge`命令或者`git rebase`命令，在本地分支上合并远程分支。
```
$ git merge origin/master
$ git rebase origin/master
```
上面命令表示在当前分支上，合并origin/master。