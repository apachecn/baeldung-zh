# Git 中假设不变和跳过工作树的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/git-assume-unchanged-skip-worktree>

## 1.概观

当我们想要手动操作 [Git](/web/20220524050201/https://www.baeldung.com/git-guide) staging 区域中的文件时，我们使用 [`git update-index`](/web/20220524050201/https://www.baeldung.com/Program%20Files/Git/mingw64/share/doc/git-doc/git-update-index.html) 。该命令支持两个经常被误用的选项:`–assume-unchanged`和`–skip-worktree`。

在本教程中，我们将了解这两个选项的不同之处，并为每个选项提供一个用例。

## 2.`assume-unchanged`选项有什么作用？

`–assume-unchanged`选项告诉 Git 暂时假设一个被跟踪的文件在工作树中没有被修改。因此，所做的更改不会反映在临时区域中:

```java
$ git update-index --assume-unchanged assumeunchanged.txt
```

我们可以用`[git ls-files](/web/20220524050201/https://www.baeldung.com/Program%20Files/Git/mingw64/share/doc/git-doc/git-ls-files.html)`来验证文件状态:

```java
$ git ls-files -v
$ h assumeunchanged.txt 
```

这里，`h` 标签表示`assume-unchanged.txt`被标记了`assumed-unchanged`选项`.`

尽管主要用于这个目的， [assume-unchanged 选项并不意味着忽略对跟踪文件](https://web.archive.org/web/20220524050201/https://github.com/git/git/commit/936d2c9301e41a84a374b98f92777e00d321a2ea)的更改。**它是为检查一组文件是否被修改的代价很高的情况而设计的。**如果我们想要优化慢速文件系统上的资源使用，会发生什么:git 忽略了对目标文件的任何检查，并且不会比较它在工作目录和索引中的版本。

每当目标文件在索引中的条目发生变化时，此功能就会丢失。当文件在上游被更改时，可能会发生这种情况。要取消设置该选项，我们可以使用`–no-assume-unchanged`:

```java
$ git update-index --no-assume-unchanged assumeunchanged.txt
```

## 3.`skip-worktree`选项有什么作用？

`–skip-worktree`选项忽略已经跟踪的文件中未提交的更改。不管在工作树中做了什么修改，git 将总是使用 staging 区域中的文件内容和属性。**当我们想要添加本地更改到一个文件而不把它们推送到上游**时，这很有用:

```java
$ git update-index --skip-worktree skipworktree.txt
```

我们可以验证文件状态:

```java
$ git ls-files -v
$ S skipworktree.txt 
```

这里，`S` 表示 `skip-worktree.txt`标有`skip-worktree`选项`.`

当文件在索引中改变时，即，如果文件在上游被改变并且我们提取它，该选项自动取消设置。

`–no-skip-worktree` 用于取消设置该选项。如果标记了错误的文件，或者情况发生了变化，以前跳过的文件不应再被忽略，这将非常有用:

```java
$ git update-index --no-skip-worktree skipworktree.txt
```

## 4.选项之间的差异

### 4.1.分支交换

当一个文件有`–skip-worktree`选项时，签出一个分支是没有问题的。但是，`–assume-unchanged`会引发一个错误:

```java
$ git checkout  another-branch
error: Your local changes to the following files would be overwritten by checkout:
        assumeunchanged.txt
Please commit your changes or stash them before you switch branches.
Aborting
```

我们可以取消选项来克服这种情况:

```java
$ git update-index --no-assume-unchanged assumeunchanged.txt 
$ git checkout another-branch 
Switched to branch 'another-branch'
```

### 4.2.优先

当两者都置位时，`–skip-worktree`优先于— `assume-unchanged`位。让我们尝试在一个文件上设置这两个选项:

```java
$ git update-index --assume-unchanged --skip-work-tree worktree-assumeunchanged.txt
```

文件的状态确认了`skip-wortkree`的优先级:

```java
$ git ls-files -v
$ S worktree-assumeunchanged.txt
```

## 5.结论

在本文中，我们讨论了 Git 的— `assume-unchanged`和— `skip-worktree`选项在用法上的区别。我们还讨论了它们的优先级以及它们如何与本地和上游分支机构进行交互。