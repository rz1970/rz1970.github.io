# Git使用的8个小技巧

## 使用-p选择性添加
当你想提交内容时，你可以通过使用 `git commit -am` 来选择所有文件或使用 `git add file` 来添加特定文件。然而，有时候你可能想只添加文件的一部分来提交。你可以用 `git add -p` 交互性地选择哪些你想提交的部分。

在选择完你所想要提交的区块后，只需要做一个 `git commit`（没有 -a），这样只会提交选中的部分。同样可以使用 `git checkout -p` 来选择需要恢复的部分。添加后，你可以使用 `git diff --cached` 来查看差异。

## 交互式的重建基准(Interactive Rebase)
如果你在一个分支上工作，同时进行了一些提交(commit)，以用来压缩合并（squash）或者删除一个提交（commit）以及这个提交的恢复， 你可以做一个交互式的重建基准（rebase），用来重新组织提交。

为了做到这点，你需要运行命令 `git rebase -i <commit>`，这里的<commit>是你想要重写之前的一个提交（commit）的sha1值。接下来，它将在你的编辑器（在$EDITOR环境变量，或者git配置里面指定的编辑器）上面打开一些指令，用来变更提交（commit）历史，你可以选择压缩合并（将两次提交合并为一次新的提交），重写（变更提交信息），编辑甚至删除一个提交（commit）。

请注意这改变了历史信息，因此，如果你提交了这个变更，你将不得不再一次强制提交（push），所以，绝不要在主分支，或者有其他人（除你之外）在使用的分支上面做这个操作。

## 储藏（Stashing）
如果你正忙于什么事情，你必须更改文本去修复其他问题，去`git stash`查到储藏在当前中的更改。然而，过一段时间后，你可能就会忘记关于这个已储藏的变更。因此，我试着去保持一个储藏0（就像收件箱0如果没有储藏的情况）规则。每一次我储藏一个美元信号出现在我的输出，并且我通过`git stash show -p`检查，还可以通过`git stash pop`弹出或者通过`git stash clear`丢弃。

## 全局gitignore
在项目的根目录，你可以通过文件.gitignore来指定git需要忽略的文件。但是，如果存在git需要忽略的文件，同时，你又是唯一创建这个文件的人（如vim的 bkp文件，编辑器或者操作系统生成的一些文件，如OSX生成的.DS_Store文件），你可以在配置文件中指定全局的gitignore文件，它和工程中的.gitignore文件使用一样的语法格式。
```
git config --global core.excludesfile '/Users/Ryan/.gitignore'
```
### 空格警告
我不得不承认：有时候我忘记代码尾部的一些空格。但是，通常我不会提交它们，因为我使用了这个选项：`git config --global apply.whitespace 'warn'`。每次我通过`git add -p`增加文件中的一大块代码，同时，这个代码块尾部包含空格，由于git给出警告提示，因此我可以在提交前修正它们。

### 自动重建代码基准（Auto setup rebase）
另一个很酷的技巧是自动重建代码基准（auto setup rebase）。如果你有一个分支，并做了一些commit但并未push。同时，其他人也在这个分支上进行了commit和push。当你pull的时候，git会创建一个commit来合并你的commit到上游（upstream）commit。由于这个commit毫无意义，我更倾向于，在pull时，通过配置来自动重建代码基准（auto setup rebase）：`branch.autosetuprebase=always`。设置之后，每个pull操作，git都会尝试在当前版本的上游（upsteam）分支重新使用你的commit。

### 更好的日志(logging)
你是否尝试过在一个分支中找一个特定的提交啊？ git log 给我们提供了一些基本的信息，不过你使用下面的命令会得到更多有用的信息：
```
git log --graph --decorate --pretty=oneline --abbrev-commit
```
+ --graph 会在各个提交之间打印出线条，这些线条可以展示出分支之间的关系。
+ --decorate 显示出分支处在哪一次提交上。
+ --pretty=oneline 只是在一行中显示 sha1 和 提交的注释(译者将title一词应对到更精确的注释)
+ --abbrev-commit 用开始的7个sha1字符代替整个sha1（他在你的仓库中是唯一的）。

### 改写提交的注释
如果你在提交代码的时候注释不能准确的描述当前提交，或者你写了错别字。你可以使用 `git commit --amend` 来改写你已经提交的注释。 他允许你在命令行中通过 -m 选项来指定新注释或直接打开系统默认编辑器让你来编辑新的注释。另外你还可以把一些新的变化加入到上一次提交中。请记住该操作和 Interactive Rebase 一样，他会改变提交历史。如果你已经把你改动的这次提交push了，那么你需要强制（force）push这次变化。

Reference: http://blog.rlmflores.me/git/2015/03/31/8-small-git-tips/