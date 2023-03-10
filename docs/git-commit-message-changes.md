# 如何修改 Git 提交消息

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/git-commit-message-changes>

## 1.概观

在本教程中，我们将看到如何修改一个 [Git](/web/20220810165030/https://www.baeldung.com/git-guide) commit 消息，无论它是最近的提交还是旧的提交。

## 2.修改最近的提交消息

我们将从最简单的情况开始。让我们构建一个简单的提交，它的提交消息中有一个错别字:

```java
$ touch file1
$ git add file1
$ git commit -m "Ading file1"
[articles/BAEL-5627-how-to-modify-git-commit-message 3e9ac2dbcd] Ading file1
1 file changed, 0 insertions(+), 0 deletions(-)
create mode 100644 file1 
```

现在，让我们确认在最近的提交消息中存在输入错误，并注意提交的散列:

```java
$ git log -1
commit 3e9ac2dbcdde562e50c5064b288f5b3fa23f39da (HEAD -> articles/BAEL-5627-how-to-modify-git-commit-message)
Author: baeldung <[[email protected]](/web/20220810165030/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date: Tue Jun 21 21:53:12 2022 +0200

 Ading file1
```

**为了修正打字错误，我们将使用[修改](https://web.archive.org/web/20220810165030/https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History)选项:**

```java
$ git commit --amend -m "Adding file1"
[articles/BAEL-5627-how-to-modify-git-commit-message 66dfa06796] Adding file1
Date: Tue Jun 21 21:53:12 2022 +0200
1 file changed, 0 insertions(+), 0 deletions(-)
create mode 100644 file1
```

同样，让我们显示最近的提交:

```java
$ git log -1
commit 66dfa067969f941eef5304a6fbcd5b22d0ba6c2b (HEAD -> articles/BAEL-5627-how-to-modify-git-commit-message)
Author: baeldung <[[email protected]](/web/20220810165030/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date: Tue Jun 21 21:53:12 2022 +0200

 Adding file1
```

我们现在可以确认在最近的消息中拼写错误得到了修复，但也注意到提交散列已经更改。从技术上讲，我们并没有改变我们的提交，而是用一个新的替换了它。

## 3.改写旧的提交消息

现在让我们添加两个新的提交，这样错别字就不会出现在最近的提交中，而是出现在较早的提交中:

```java
$ touch file2
$ git add file2
$ git commit -m "Ading file2"
[articles/BAEL-5627-how-to-modify-git-commit-message ffb7a68bf6] Ading file2
1 file changed, 0 insertions(+), 0 deletions(-)
create mode 100644 file2
$ touch file3
$ git add file3
$ git commit -m "Adding file3"
[articles/BAEL-5627-how-to-modify-git-commit-message 517193e1e9] Adding file3
1 file changed, 0 insertions(+), 0 deletions(-)
create mode 100644 fil3 
```

让我们检查一下刚刚添加的两个提交:

```java
$ git log -2
commit 517193e1e99c784efd48086f955fcdbc3110d097 (HEAD -> articles/BAEL-5627-how-to-modify-git-commit-message)
Author: baeldung <[[email protected]](/web/20220810165030/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date: Tue Jun 21 22:04:56 2022 +0200

 Adding file3

commit ffb7a68bf63c7da9bd0b261ebb9b2ca548aa1333
Author: baeldung <[[email protected]](/web/20220810165030/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date: Tue Jun 21 22:02:59 2022 +0200

 Ading file2
```

**Git 的`amend`选项只适用于最近的提交**，所以这次我们不能用它来修复打字错误。

相反，我们将使用`rebase`。

### 3.1.开始一个交互式的 Rebase

为了修复旧的提交消息，让我们通过运行下面的命令来继续进行所谓的[交互重定基础](https://web.archive.org/web/20220810165030/https://git-scm.com/docs/git-rebase):

```java
$ git rebase -i HEAD~2
hint: Waiting for your editor to close the file...
```

这里意味着我们将重新审视最近的两次提交。

这将在您的机器上打开与 Git 相关联的文本编辑器[,并在编辑器中填充所有可在重置过程中使用的命令，包括:](https://web.archive.org/web/20220810165030/https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration)

```java
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
```

正如我们所见，我们可以在重置提交时做各种事情，比如通过 [`squash`命令](/web/20220810165030/https://www.baeldung.com/ops/git-squash-commits)将两个提交合并成一个。

这里，**我们想要重写提交消息，所以我们将使用`reword`命令。**

### 3.2.改写提交消息

编辑器中的前两行包含以下文本:

```java
pick ffb7a68bf6 Ading file2
pick 517193e1e9 Adding file3
```

注意，在这个视图中，提交是从最早到最近排列的，这与我们使用 `[git log](https://web.archive.org/web/20220810165030/https://git-scm.com/docs/git-log)` 相反。

让我们将第一行改为使用`reword`命令，而不是`pick`命令；我们将把`pick`留给第二次提交，因为我们希望保持消息原样:

```java
reword ffb7a68bf6 Ading file2
pick 517193e1e9 Adding file3
```

如果我们想在一个 rebase 中更改这两个消息，我们可以简单地将这两行上的命令更改为`reword`。

现在，**我们还没有更改提交消息**。所以让我们**保存我们的文件并关闭文本编辑器**，这让 Git 知道我们已经完成了我们的重置指令。

Git 现在将处理 rebase 命令，当它需要我们的交互时提示我们。由于我们告诉 Git 进行第一次提交，它将使用第一次提交的内容重新打开文本编辑器。第一行包含提交消息:

```java
Ading file2
```

下面几行的注释描述了`reword`操作将如何工作。

让我们编辑提交消息，方法是将第一行修改为“添加文件 2”，保存文件，然后关闭编辑器。

Git 将更新我们的提交消息，然后完成其余的 rebase 指令。我们来确认一下 Git 的工作:

```java
$ git log -2
commit 421d446d77d4824360b516e2f274a7c5299d6498 (HEAD -> articles/BAEL-5627-how-to-modify-git-commit-message)
Author: baeldung <[[email protected]](/web/20220810165030/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date: Tue Jun 21 22:04:56 2022 +0200

 Adding file3

commit a6624ee55fdb9a6a2446fbe6c6fb8fe3bc4bd456
Author: baeldung <[[email protected]](/web/20220810165030/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date: Tue Jun 21 22:02:59 2022 +0200

 Adding file2
```

另外，请注意，这些提交的提交散列已经更改，就像我们之前修改的提交一样。我们所有的原始提交都被新的替换了。

## 4.推动您重写的提交

此时，我们的分支上有三个提交:一个修改的提交和两个重定基础的提交。它们都有不同于我们最初提交的提交散列。

只要原始提交没有被推送到任何远程存储库，我们就能够正常地[推](https://web.archive.org/web/20220810165030/https://git-scm.com/docs/git-push)。

然而，假设我们在替换之前已经提交了任何有缺陷的提交。在这种情况下，远程存储库将拒绝我们的新推送，因为我们的本地提交历史不再与存储库的历史兼容。

因此，**如果我们想要推送修正过的(但之前被推送过的)提交，我们将不得不使用[`force`选项](https://web.archive.org/web/20220810165030/https://git-scm.com/docs/git-push) :**

```java
$ git push --force
```

应该小心使用这个命令，因为它会用我们的更改覆盖远程分支，如果其他人正在使用该分支，这可能会导致问题。

## 5.结论

在本文中，我们已经看到了如何编辑提交消息，无论是最后一条消息还是旧消息。我们还看到了如何将变更的提交推送到拥有原始提交的存储库中，注意到这应该小心进行。