# 使用删除跟踪的文件。gitignore

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/git-remove-tracked-files-gitignore>

## 1.介绍

我们知道， [`.gitignore`](https://web.archive.org/web/20221106183604/https://git-scm.com/docs/gitignore) 文件防止将来未被跟踪的文件被添加到 git 索引中。换句话说，git 仍然会跟踪当前跟踪的任何文件。

在本教程中，我们将探索在将跟踪的文件添加到`.gitignore` 后，**从 git 索引中移除它们的不同可能性。**

## 2.删除单个文件

为了删除单个文件，我们首先必须将文件名添加到`.gitignore`中，然后运行`git rm`命令，之后是提交:

```java
git rm --cached <filename>
git commit -m "<Message>"
```

第一个命令从索引中删除文件并存放更改，而第二个命令将更改提交到分支。

## 3.移除文件夹

我们可以通过首先将文件夹名称添加到`.gitignore`并运行`git`命令来删除整个文件夹:

```java
git rm --cached -r <folder>
git commit -m "<Message>"
```

**注意命令中添加的`-r`，如果没有它，命令将因**而失败:

```java
fatal: not removing 'folder' recursively without -r.
```

## 4.删除所有忽略的文件

在这里，我们将删除当前在`.gitignore`中被忽略的所有文件:

```java
git rm -r --cached .
git add .
git commit -m "Removes all .gitignore files and folders"
```

第一个命令从索引中删除所有文件。第二个命令重新添加所有文件，不包括`.gitignore`中的文件，最后一个命令提交更改。在这三个命令之后，`.gitignore`中的所有文件都将从索引中删除。

## 5.结论

在本文中，我们探讨了从 git 索引中删除跟踪文件的三种不同方法。

上面的动作**不会从我们的机器**上移除物理文件，但是**会在其他开发者的机器**上移除文件。