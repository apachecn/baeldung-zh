# 如何修复 Git“拒绝合并不相关的历史”

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/git-merge-unrelated-histories-error>

## 1.概观

在 [Git](/web/20221023090055/https://www.baeldung.com/git-guide) 中，存在[分支](/web/20221023090055/https://www.baeldung.com/git-guide#11-git-branching)没有共同历史基础的情况。因此，如果我们试图合并它们，我们会得到`“refusing to merge unrelated histories”`错误。在本教程中，我们将讨论如何修复这个错误，以及如何在未来的项目中避免这个错误。

## 2.为什么分支可以有不相关的历史？

让我们看看分支拥有不相关历史的场景。拥有一个不相关的历史库的最常见的原因是为了开始彼此独立的分支。例如，如果我们在本地机器上启动一个新的 Git 项目，然后将它连接到一个远程 GitHub 分支，这些分支将有不同的历史基础。

唯一的例外是其中一个分支没有提交。在这种情况下，它们合并应该没有问题。否则，我们将得到如下例所示的`“refusing to merge unrelated histories”`:

```java
$ git pull origin main
...
fatal: refusing to merge unrelated histories 
```

正如我们所看到的，我们不能使用 [`git pull`](/web/20221023090055/https://www.baeldung.com/git-guide#103-git-pull--update-and-apply-at-once) 命令来合并具有不常见历史的分支。

## 3.如何修复错误

为了修复上面的错误，我们需要在 `git pull <remote> <branch>` 命令后使用选项`–allow-unrelated-histories` ，其中

*   `<remote>`是远程存储库的 URL 还是它的简称`origin`
*   `<branch>`是我们想要合并的分支名称

例如`:`

```java
$ git pull origin main --allow-unrelated-histories
```

`–allow-unrelated-histories`选项将告诉 Git，我们允许合并没有共同历史基础的分支，然后 Git 将完成合并，不会出现错误。**我们应该注意到，对于 Git 2.9 或更老的版本，这个选项不是必需的，我们的合并不需要它也能工作。**要检查 Git 版本，我们可以使用 [`git –version`](https://web.archive.org/web/20221023090055/https://git-scm.com/search/results?search=git%20version) 命令。

## 4.如何避免将来的错误

通常，独立于远程存储库创建本地存储库分支并不是最佳实践。更可靠的方法是使用 [`git clone`](/web/20221023090055/https://www.baeldung.com/git-guide#2-git-clone---clone-an-external-repository) 命令将远程存储库下载到本地机器，如下所示:

```java
$ git clone <repo_url>
```

这样，我们从远程服务器复制存储库，并且提交历史库对于远程和本地分支保持不变。

## 5.结论

在本教程中，我们讨论了当 Git 拒绝合并不相关的历史时如何合并分支。我们首先看一下这个错误什么时候会发生。然后，我们使用`–allow-unrelated-histories`选项来修复它。最后，我们学会了如何在未来的项目中避免这种错误。