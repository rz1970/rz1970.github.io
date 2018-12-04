# Git使用技巧之git stash

`git stash`可以用于暂存代码。比如需要切换一个`branch`去做其他的事情，但是当前又有一些代码没有`commit`。你显然也不会想要把这些修改`checkout`。该命令就是为了解决这一个问题。

使用`git stash`保存当前的工作现场，那么就可以切换到其他分支进行工作，或者在当前分支上完成其他紧急的工作，比如修订一个bug测试提交。
```
$ git stash
```
保存当前的工作进度。会分别对暂存区和工作区的状态进行保存。
```
$ git stash list
```
显示进度列表。此命令显然暗示了`git stash`可以多次保存工作进度，并用在恢复时候选择。
```
$ git stash pop [--index] [<stash>]
```
如果不使用任何参数，会恢复最新保存的工作进度，并将恢复的工作进度从存储的工作进度列表中清除。

如果提供`<stash>`参数（来自`git stash list`显示的列表），则从该`<stash>`中恢复。恢复完毕 也将从进度列表中删除`<stash>`。选项`--index`除了恢复工作区的文件外，还尝试恢复暂存区。这也就是为什么在本章一开始恢复进度的时候显示的状态和保存进度前的略有不同。
```
$ git stash [save [--patch] [-k|--[no]keep-index] [-q|--quiet] [<message>]]
```
这条命令实际上是第一条`git stash`命令的完整版。即如果需要在保存工作进度的时候使用指定的说明，必须使用如下格式：
```
$ git stash save “message...”
```
使用参数`--patch`会显示工作区和HEAD的差异，通过对差异文件的编辑决定在进度中最终要保存的工作区的内容，通过编辑差异文件可以在进度中排除无关内容。使用`-k`或者`--keep-index`参数，在保存进度后不会将暂存区重置。默认会将暂存区和工  作区强制重置。
```
$ git stash apply [--index] [<stash>]
```
除了不删除恢复的进度之外，其余和`git stash pop`命令一样。
```
$ git stash drop [<stash>]
```
删除一个存储的进度。默认删除最新的进度。
```
$ git stash clear
```
删除所有存储的进度。
```
$ git stash branch <branchname> <stash>
```
基于进度创建分支。