---
title: Git使用技巧之git pull
tags: Git
---

`git pull`命令的作用是，取回远程主机某个分支的更新，再与本地的指定分支合并。

```bash
[user@localhost]$ git pull <远程主机名> <远程分支名>:<本地分支名>
```

比如，取回origin主机的next分支，与本地的master分支合并，需要写成下面这样。

```bash
[user@localhost]$ git pull origin next:master
```

如果远程分支是与当前分支合并，则冒号后面的部分可以省略。

```bash
[user@localhost]$ git pull origin next
```

上面命令表示，取回origin/next分支，再与当前分支合并。实质上，这等同于先做`git fetch`，再做`git merge`。

```bash
[user@localhost]$ git fetch origin
[user@localhost]$ git merge origin/next
```

在某些场合，Git会自动在本地分支与远程分支之间，建立一种追踪关系(tracking)。比如，在`git clone`的时候，所有本地分支默认与远程主机的同名分支，建立追踪关系，也就是说，本地的master分支自动“追踪”`origin/master`分支。

Git也允许手动建立追踪关系。

```bash
[user@localhost]$ git branch --set-upstream master origin/next
```

上面命令指定master分支追踪`origin/next`分支。

如果当前分支与远程分支存在追踪关系，git pull就可以省略远程分支名。

```bash
[user@localhost]$ git pull origin
```

上面命令表示，本地的当前分支自动与对应的origin主机“追踪分支”(`remote-tracking branch`)进行合并。

如果当前分支只有一个追踪分支，连远程主机名都可以省略。

```bash
[user@localhost]$ git pull
```

上面命令表示，当前分支自动与唯一一个追踪分支进行合并。

如果合并需要采用rebase模式，可以使用–rebase选项。

```bash
[user@localhost]$ git pull --rebase <远程主机名> <远程分支名>:<本地分支名>
```