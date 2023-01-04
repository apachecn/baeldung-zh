# 如何在 Git 中找回丢失的东西

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/git-recover-dropped-stash>

## 1.概观

像`git stash`和`git stash pop`这样的命令用于搁置(隐藏)和恢复我们工作目录中的更改。在本教程中，我们将学习如何在 [Git](/web/20220909210247/https://www.baeldung.com/git-guide) 中找回丢失的东西。

## 2.将更改隐藏在工作目录中

对于我们的例子，假设我们已经分叉并克隆了一个 [Git 库](https://web.archive.org/web/20220909210247/https://github.com/eugenp/tutorials)。现在，让我们对`README.md` 文件做一些修改，只需在末尾添加新的一行，并检查我们的工作目录的状态:

```
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")
```

从这里，我们可以使用`git stash `命令暂时搁置我们的更改。

```
$ git stash
Saved working directory and index state WIP on master: a8088442db Updated pom.xml
```

现在，如果再做一次`git status `，我们会看到我们的工作目录是干净的。

```
$ git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

## 3.恢复隐藏的更改并查找哈希

让我们看看如何恢复隐藏的更改，并找到与隐藏提交相关联的散列。

### 3.1.将隐藏的更改恢复到工作目录中

我们可以像这样将隐藏的更改放回我们的工作目录:

```
$ git stash pop
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")
Dropped refs/[[email protected]](/web/20220909210247/https://www.baeldung.com/cdn-cgi/l/email-protection){0} (59861637f7b599d87cb7a1ff003f1b0212e8908e)
```

正如我们在最后一行 **`git stash pop`中看到的，不仅恢复了隐藏的更改，还删除了对相关提交的引用**。

### 3.2.当终端打开时定位散列

如果我们的终端仍然打开，我们可以很容易地定位到执行`git stash pop`后生成的 hash。在我们的例子中，最后一行显示的散列是`59861637f7b599d87cb7a1ff003f1b0212e8908e.`

### 3.3.在终端关闭后恢复散列

即使我们关闭了终端，我们仍然可以通过以下方式找到我们的哈希:

```
$ git fsck --no-reflog
Checking object directories: 100% (256/256), done.
Checking objects: 100% (302901/302901), done.
dangling commit 59861637f7b599d87cb7a1ff003f1b0212e8908e
```

我们现在可以看到被丢弃的存储的提交散列。

## 4.找回丢失的毒品

一旦应用了一个隐藏条目，我们通常就不需要它了。然而，可能会有这样一种情况，我们希望在删除一个隐藏条目后再回到它。例如，如果使用 **`git reset –hard HEAD` 将会从我们的工作目录中丢弃所有未提交的更改。**在这种情况下，我们可能希望召回一些早期隐藏的变更，即使它们已经被删除了。

### 4.1.使用哈希来恢复藏匿

使用悬空提交的散列，我们仍然有可能恢复这些更改:

```
$ git stash apply 59861637f7b599d87cb7a1ff003f1b0212e8908e
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")
```

我们可以看到，我们的工作目录恢复了之前隐藏的更改。

### 4.2.查找所有哈希提交

如果我们没有现成的散列，我们可以找到它:

```
git fsck --no-reflog | awk '/dangling commit/ {print $3}'
```

这里，我们将`–no-reflog `选项与`awk` 结合起来，只为我们过滤掉散列。

## 5.结论

在本文中，我们看到了`git stash`是如何工作的，以及当我们使用它时它是如何删除一个条目的。然后，我们看到了当我们知道一个被丢弃的条目的散列时，我们如何仍然可以使用它，以及如何找到 stash commit 的散列。