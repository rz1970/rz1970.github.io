---
title: Git使用技巧之git remote
tags: Git
---

为了便于管理，Git要求每个远程主机都必须指定一个主机名。`git remote`命令就用于管理主机名。不带选项的时候，`git remote`命令列出所有远程主机。

```bash
$ git remote
origin
```

使用-v选项，可以参看远程主机的网址。

```bash
$ git remote -v
origin  git@github.com:jquery/jquery.git (fetch)
origin  git@github.com:jquery/jquery.git (push)
```

上面命令表示，当前只有一台远程主机，叫做origin，以及它的网址。

克隆版本库的时候，所使用的远程主机自动被Git命名为origin。如果想用其他的主机名，需要用`git clone`命令的`-o`选项指定。

```bash
$ git clone -o jQuery https://github.com/jquery/jquery.git
$ git remote
jQuery
```

上面命令表示，克隆的时候，指定远程主机叫做jQuery。

`git remote show`命令加上主机名，可以查看该主机的详细信息。

```bash
git remote show <主机名>
```

`git remote add`命令用于添加远程主机。

```bash
git remote add <主机名> <网址>
```

`git remote rm`命令用于删除远程主机。

```bash
git remote rm <主机名>
```

`git remote rename`命令用于远程主机的改名。

```bash
git remote rename <原主机名> <新主机名>
```