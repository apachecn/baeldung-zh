# 如何在 Git 中获取当前分支名称

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/git-current-branch-name>

## 1.概观

[Git](/web/20220803043636/https://www.baeldung.com/git-guide) 已经成为业界流行且广泛使用的版本控制系统。通常，当我们使用 Git 存储库时，我们使用分支。

在本教程中，我们将探索如何获得我们目前正在工作的分支名称。

## 2.问题简介

首先，让我们准备一个名为`myRepo`的 Git 存储库:

```java
$ git branch -a
* feature
  master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
```

如 [`git` `branch`](/web/20220803043636/https://www.baeldung.com/git-guide#11-git-branching) 命令的输出所示，我们在`myRepo`有两个本地分支，现在，当前检出的分支是`feature`分支，因为在`feature`前面有一个“*”字符。

有时，我们可能只想得到当前的分支名称，而不想得到整个分支列表。例如，如果我们在存储库中有许多本地分支，我们可能会得到一个很长的分支列表。或者，当我们编写一些脚本来自动化一些过程时，我们只想获得分支名称并将其传递给其他命令。

那么接下来，我们来看看如何快速获取当前分支“`feature`”。

## 3.使用`git symbolic-ref`命令

当前分支信息由`.git/HEAD`文件存储或链接，这取决于 Git 版本。例如，在这台机器上，`.git/HEAD`文件包含:

```java
$ cat .git/HEAD
ref: refs/heads/feature 
```

所以，它指向了一个`symbolic ref`。**[`git symbolic-ref`](https://web.archive.org/web/20220803043636/https://git-scm.com/docs/git-symbolic-ref)命令允许我们读取和修改符号参考**。我们可以使用这个命令获得简短的当前分支名称:

```java
$ git symbolic-ref --short HEAD
feature 
```

## 4.使用`git rev-parse`命令

**从 Git 1.7 版本开始，我们也可以使用 [`git rev-parse`](https://web.archive.org/web/20220803043636/https://git-scm.com/docs/git-rev-parse) 命令来获取当前的分支名称**:

```java
$ git rev-parse --abbrev-ref HEAD
feature 
```

## 5.使用`git name-rev`命令

Git 的 [`git name-rev`](https://web.archive.org/web/20220803043636/https://git-scm.com/docs/git-name-rev) 命令可以找到给定版本的符号名称。因此，为了得到当前的分支名称，我们可以读取 rev `HEAD`的名称:

```java
$ git name-rev --name-only HEAD
feature
```

如上面的输出所示，`git name-rev`命令也打印当前的分支名称。我们应该注意到**我们在命令中添加了`–name-only`选项，以抑制输出中的 rev 信息`(HEAD)`。**

## 6.使用`git branch`命令

我们知道，如果我们启动没有任何选项的`git branch`命令，Git 将打印所有本地分支，并将当前分支放在第一行，在名称前面有一个“*”字符:

```java
$ git branch
* feature
  master 
```

因此，我们可以解析`git branch`命令的输出来获得分支名称。让我们看一个组合`git branch`和 [`sed`](/web/20220803043636/https://www.baeldung.com/linux/sed-editor) 命令的例子:

```java
$ git branch | sed -n '1{s/^* *//;p}'
feature 
```

实际上，**从 2.22 版本开始，Git 在`git branch`命令**中引入了`–show-current`选项，这样我们可以直接得到当前分支的名称:

```java
$ git branch --show-current
feature 
```

显然，这个命令比解析`git branch`命令的输出要好。

## 7.结论

在这篇简短的文章中，我们介绍了几种在 Git 存储库中获取当前分支名称的方法。