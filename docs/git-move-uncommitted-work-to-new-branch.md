# 将现有的、未提交的工作转移到 Git 中的新分支

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/git-move-uncommitted-work-to-new-branch>

## 1.概观

Git 是当今非常流行的版本控制系统。

在这个快速教程中，我们将探索如何将现有的但是未提交的变更转移到一个新的分支。

## 2.问题简介

首先，让我们考虑一下向 Git 托管项目添加新特性的典型工作流程:

*   创建一个新的特性分支，比如说`feature`，然后切换到那个分支
*   实现该特性，并将其提交给我们的本地存储库
*   推送到远程存储库的特性分支，并创建一个拉请求
*   其他队友审核后，新的变更可以合并到`master`或`release`分支

然而，有时，我们已经开始做出改变，但是忘记了创建一个新的特性分支并切换到它。结果，当我们要提交我们的变更时，我们可能意识到我们在错误的分支——例如，`master`分支。

因此，我们需要创建一个新的特性分支，并将未提交的工作转移到新的分支。此外，`master`分支不应该被修改。

一个例子可以很快解释这种情况。假设我们有一个名为`myRepo`的 Git 存储库:

```java
$ git branch
* master

$ git status
On branch master
nothing to commit, working tree clean
```

正如我们从上面的输出中看到的，我们目前在`master`分支上。还有，工作树是干净的。

接下来，让我们做一些改变:

```java
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   Readme.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	a-new-file.txt

no changes added to commit (use "git add" and/or "git commit -a") 
```

如上面的输出所示，我们添加了一个新文件`a-new-file.txt`，并更改了`Readme.md`的内容。现在，我们意识到这项工作应该交给一个特色部门，而不是`master`部门。

接下来，让我们看看如何将更改移动到一个新的分支，并保持`master`不变。

## 3.使用`git checkout`命令

`git checkout -b <BranchName>`命令将创建一个新的分支并切换到它。此外，该命令将**保持当前分支不变，并将所有未提交的更改带到新分支**。

接下来，让我们在我们的`myRepo`项目上测试`git checkout`命令:

```java
$ git branch
* master

$ git co -b feature1
Switched to a new branch 'feature1'

$ git status
On branch feature1
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   Readme.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	a-new-file.txt

no changes added to commit (use "git add" and/or "git commit -a") 
```

如上面的命令所示，我们已经创建了`feature1`分支，并将所有未提交的变更从`master`转移到`feature1`。接下来，让我们准备并提交更改:

```java
$ git add . && git commit -m'implemented feature1'
[feature1 2ffc161] implemented feature1
 2 files changed, 2 insertions(+)
 create mode 100644 a-new-file.txt

$ git log --abbrev-commit feature1
commit 2ffc161 (HEAD -> feature1)
Author: ...
Date:   ...
    implemented feature1

commit b009ddf (master)
Author: ...
Date:   ...
    init commit 
```

现在，让我们切换回`master`分支，检查我们是否没有改变它:

```java
$ git checkout master
Switched to branch 'master'
$ git status
On branch master
nothing to commit, working tree clean
$ git log --abbrev-commit master
commit b009ddf (HEAD -> master)
Author: ...
Date:   ...
    init commit
```

正如我们在输出中看到的，在`master`分支上没有局部变化。此外，在`master`上也没有新的提交。

## 4.使用`git switch`命令

众所周知，Git 的`checkout`命令就像一把瑞士军刀。同一个命令可以做很多不同种类的操作，比如恢复工作树文件、切换分支、创建分支、移动头等等。`checkout`命令的使用已经超载了。

因此，Git 从 2.23 版本开始引入了 [`git switch`](https://web.archive.org/web/20220611090101/https://git-scm.com/docs/git-switch) 命令，以清除`checkout`命令过载使用带来的一些混乱。顾名思义，`git switch`允许我们在分支之间切换。此外，**我们可以使用`-C`选项创建一个新的分支，并一次性切换到它**。它的工作原理与`git checkout -b`命令非常相似。

接下来，让我们对`myRepo`项目进行与`git checkout -b` 相同的测试:

```java
$ git branch
  feature1
* master

$ git status
On branch master
Changes not staged for commit:
  (use "git add/rm ...)
  (use "git restore ...)
	deleted:    Readme.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	ReadmeNew.md

... 
```

正如我们在上面的输出中看到的，我们目前在`master`分支上。这一次，我们删除了文件`Readme.md`并添加了一个新的`ReadmeNew.md`文件。

接下来，让我们使用`git` `switch`命令将这些未提交的变更移动到一个名为`feature2`的新分支:

```java
$ git switch -C feature2
Switched to a new branch 'feature2'

$ git status
On branch feature2
Changes not staged for commit:
  (use "git add/rm ...)
  (use "git restore ...)
        deleted: Readme.md
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        ReadmeNew.md
...
$ git add . && git commit -m 'feature2 is done'
[feature2 6cd5933] feature2 is done
1 file changed, 0 insertions(+), 0 deletions(-)
rename Readme.md => ReadmeNew.md (100%)
$ git log --abbrev-commit feature2
commit 6cd5933 (HEAD -> feature2)
Author: ...
Date:   ...
      feature2 is done
commit b009ddf (master)
Author: ...
Date:   ...
      init commit 
```

正如我们在上面的输出中看到的，`git` `switch -C` 创建了一个新的分支`feature2` ，并把我们带到了`feature2`。此外，所有未提交的变更已经从`master`转移到`feature2`分支。然后，我们将变更提交到`feature2`分支。

接下来，让我们切换回`master`分支，检查它是否未被修改:

```java
$ git switch master
Switched to branch 'master'

$ git status
On branch master
nothing to commit, working tree clean
$ ls -1 Readme.md 
Readme.md

$ git log --abbrev-commit master
commit b009ddf (HEAD -> master)
Author: ...
Date:   ...
    init commit 
```

正如我们所见，在`master`分支上，我们之前对工作树文件所做的所有更改都已经恢复。比如被移除的文件`Readme.md`又回来了。此外，`git log`命令显示在`master`上没有新的提交。

## 5.结论

在本文中，我们介绍了几种将未提交的变更转移到新的 Git 分支的快速方法。这两个命令都很容易使用:

*   `git checkout -b <NEW_BRANCH>`
*   `git switch -C <NEW_BRANCH>`