# 从 Git 的提交历史中删除一个大文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/git-remove-file-commit-history>

## 1.概观

在本教程中，我们将学习如何使用各种工具从 git 存储库的提交历史中删除大文件。

## 2。使用`git filter-branch`

这是最常用的方法，它帮助我们重写提交分支的历史。

例如，假设我们错误地将一个 blob 文件放到了一个项目文件夹中，在删除它之后，我们仍然会在 g it 历史中注意到该文件:

```java
$ git log --graph --full-history --all --pretty=format:"%h%x09%d%x20%s"
* 9e87646        (HEAD -> master) blob file removed
* 2583677        blob file
* 34ea256        my first commit
```

我们可以通过使用以下命令重写树及其内容，从 git 历史中删除 blob 文件:

```java
$ git filter-branch --tree-filter 'rm -f blob.txt' HEAD
```

这里， `rm` 选项 从树中删除文件。此外， `-f` 选项防止命令在文件不在我们项目的其他提交目录中时失败。如果没有`-f` 选项，当我们的项目中有多个目录时，这个命令可能会失败。

这里是我们运行命令后的 git 日志:

```java
* 8f39d86        (HEAD -> master) blob file removed
* e99a81d        blob file
| * 9e87646      (refs/original/refs/heads/master) blob file removed
| * 2583677      blob file
|/  
* 34ea256        my first commit 
```

我们可以用提交历史的 SHA1 键替换头部，以最小化重写。

我们的 git 日志仍然包含对被删除文件的引用。我们可以通过更新我们的回购协议来删除引用:

```java
$ git update-ref -d refs/original/refs/heads/master 
```

`-d` 选项在验证已命名的 ref 仍然包含旧值后，删除该 ref。

我们需要记录我们的引用在存储库中发生了变化:

```java
$ git reflog expire --expire=now --all
```

`expire`子命令删除旧的引用日志条目。

最后，我们需要清理和优化我们的回购:

```java
$ git gc --prune=now
```

`–prune=now`选项修剪松散的对象，而不考虑它们的年龄。

运行命令后，这里是我们的 git 日志:

```java
* 6f49d86        (HEAD -> master) my first commit 
```

我们可以看到，引用已经被删除。

或者，我们可以运行:

```java
$ git filter-branch --index filter 'git rm --cached --ignore-unmatched blob.txt' HEAD
```

这与 `tree-filter,` 的工作方式完全一样，但速度更快，因为它只重写了索引，即工作目录。子命令 `–ignore-unmatched` 防止命令在文件从我们项目的其他提交目录中丢失时失败。

我们应该注意，这种使用两个不同命令的方法在删除大文件时会很慢。

## 3。使用`git-filter-repo`

另一种方法是使用`git-filter-repo`命令。它是一个第三方插件，比其他方法更简单、更快。此外，这也是 git 官方[文档](https://web.archive.org/web/20221031104216/https://git-scm.com/docs/git-filter-branch)中推荐的解决方案。

### 3.1。安装

**最低要求 python3 > = 3.5，git>= 2 . 22 . 0；有些特性需要 git 2.24.0 或更高版本**。

我们将在我们的 Linux 机器上安装`git-filter-repo`。对于 Windows 安装指南，我们可以参考[文档](https://web.archive.org/web/20221031104216/https://github.com/newren/git-filter-repo#how-do-i-install-it)。

首先，我们将使用以下命令安装`python-pip`和`git-filter-repo` :

```java
$ sudo apt install python3-pip
$ pip install --user git-filter-repo
```

另外，我们可以使用下面的命令来安装`git-filter-repo` :

```java
# Add to bashrc.
export PATH="${HOME}/bin:${PATH}"

mkdir -p ~/bin
wget -O ~/bin/git-filter-repo https://raw.githubusercontent.com/newren/git-filter-repo/7b3e714b94a6e5b9f478cb981c7f560ef3f36506/git-filter-repo
chmod +x ~/bin/git-filter-repo
```

### 3.2。移除文件

让我们运行命令来检查我们的 git 日志:

```java
$ git log --graph --full-history --all --pretty=format:"%h%x09%d%x20%s"
* ee36517        (HEAD -> master) blob.txt removed
* a480073        project folder
```

接下来我们要分析我们的回购:

```java
$ git filter-repo --analyze
Processed 5 blob sizes
Processed 2 commits
Writing reports to .git/filter-repo/analysis...done.
```

这将生成一个报告我们回购状态的目录。可以在`.git/filter-repo/analysis.` T 找到该报告。他的信息可以帮助确定在后续运行中要过滤的内容。它还可以帮助我们确定之前的过滤命令是否真的达到了我们的目的。

然后，让我们使用选项`–path-match,`运行这个命令，它有助于指定要包含在过滤历史中的文件:

```java
$ git filter-repo --force --invert-paths --path-match blob.txt
```

这是我们新的 git 日志:

```java
* 8940776        (HEAD -> master) project folder 
```

**执行后，会改变修改后提交的提交哈希。**

## 4。使用 BRG 回购清洁剂

另一个很棒的选择是 [BRG 回购清理器](https://web.archive.org/web/20221031104216/https://rtyley.github.io/bfg-repo-cleaner/)，这是一个用 Java 编写的第三方插件。

**它比`git filter-branch` 进场更快。此外，它还有利于删除大文件、密码、凭证和其他私人数据**。

假设我们想要删除大于 200MB 的 blob 文件。这个附加组件使它很容易做到这一点:

```java
$ java -jar bfg.jar --strip-blob-bigger-than 200M my-repo.git
```

然后，让我们运行这个命令来清除无效数据:

```java
$ git gc --prune=now --aggressive
```

## 5。使用`git-rebase`

我们需要 git 日志中的 SHA1 密钥来使用这种方法:

```java
$ git log --graph --full-history --all --pretty=format:"%h%x09%d%x20%s"
* 535f7ea        (HEAD -> master) blob file removed
* 8bffdfa        blob file
* 5bac30b        index.html
```

我们的目标是从提交历史中删除 blob 文件。因此，我们将使用我们想要删除的条目之前的条目的历史中的 SHA1 键。

通过这个命令，我们进入了一个交互式的 rebase:

```java
$ git rebase -i 5bac30b
```

这将打开我们的`nano`编辑器显示:

```java
pick 535f7ea blob file removed
pick 8bffdfa blob file 

# Rebase 5bac30b..535f7ea onto 535f7ea (2 command)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .  create a merge commit using the original merge commit's
# .  message (or the oneline, if no original merge commit was
# .  specified). Use -c <commit> to reword the commit message. 
```

现在，我们将通过删除文本“ `pick 535f7ea blob file removed` ”来对此进行修改。这有助于我们改变提交历史并删除我们之前删除的历史。

然后我们保存文件并退出编辑器，这将我们带到终端并显示以下消息:

```java
interactive rebase in progress; onto 535f7ea
Last command done (1 command done):
pick 535f7ea blob file removed
No commands remaining.
You are currently rebasing branch 'master' on '535f7ea'.
(all conflicts fixed: run "git rebase --continue") 
```

最后，让我们继续 rebase 操作:

```java
$ git rebase --continue
Successfully rebased and updated refs/heads/master.
```

然后，我们可以验证我们的提交历史:

```java
$ git log --graph --full-history --all --pretty=format:"%h%x09%d%x20%s"
* 5bac30b        (HEAD -> master) index.html
```

**我们要注意，这种方法没有`git-filter-repo`** 那么快。

## 6.结论

在本文中，我们学习了从 git 存储库的提交历史中删除大文件的不同方法。我们还看到，根据 git 文档，推荐使用`git filter-repo`,因为与其他方法相比，它速度更快，缺点更少。