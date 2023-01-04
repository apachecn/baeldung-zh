# 撤销 git 重置基础指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/git-undo-rebase>

## 1.概观

git rebase 是编写干净的代码提交历史的推荐最佳实践，特别是对于多开发人员的代码库。手动完成这个操作后，我们可能会意识到我们想要返回到初始状态。

在本教程中，我们将**探索一些撤销 git rebase** 操作的技术。

## 2.设置

让我们**创建一个测试平台来模拟一个多分支**的多开发者代码库。我们可以假设,`development`分支是项目的唯一真实来源，每个开发人员都使用它来使用特定于功能的分支处理特定的功能:

![](img/a566459d48e29569048b33e22ac2bc6b.png)现在，假设我们已经为项目准备好了上述版本，让我们检查一下`feature2`分支:

```
$ git branch --show-current
feature2
```

最后，让我们看看`feature1`和`feature2`分支的代码提交历史:

```
$ git log feature1
commit e5e9afbbd82e136fc20957d47d05e72a38d8d10d
Author: Tapan Avasthi <[[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date:   Sun Jul 31 16:27:22 2022 +0530

    Add feature-1

commit 033306a06895a4034b681afa912683a81dd17fed
Author: Tapan Avasthi <[[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date:   Sun Jul 31 16:27:22 2022 +0530

    Add .gitignore file

$ git log feature2
commit 9cec4652f34f346e293b19a52b258d9d9a49092e
Author: Tapan Avasthi <[[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date:   Sun Jul 31 16:27:22 2022 +0530

    Add feature-2

commit 033306a06895a4034b681afa912683a81dd17fed
Author: Tapan Avasthi <[[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date:   Sun Jul 31 16:27:22 2022 +0530

    Add .gitignore file
```

在接下来的部分中，我们将重复使用这个基础场景来做一个`git rebase`，然后一次应用一个方法来撤销 rebase 操作。

## 3.使用`ORIG_HEAD`

让我们以一个简单的场景开始，检查`feature2`分支的当前提交:

```
$ git log HEAD
commit 728ceb3219cc5010eae5840c992072cac7a5da00 (HEAD -> feature2)
Author: Tapan Avasthi <[[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date:   Sun Jul 31 16:46:56 2022 +0530

    Add feature-2

commit 6ed8a4d2a961fdfc4d5e4c7c00b221ed6f283bf4 (development)
Author: Tapan Avasthi <[[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date:   Sun Jul 31 16:46:56 2022 +0530

    Add .gitignore file
```

现在，让我们将`feature2 `分支放在`feature1`分支之上:

```
$ git rebase feature1 
```

完成重置基础操作后，让我们看看`HEAD`参考:

```
$ git log HEAD
commit 9d38b792d0c9a8d0cd8e517fcb2ca5260989cc4a
Author: Tapan Avasthi <[[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date:   Sun Jul 31 16:46:56 2022 +0530

    Add feature-2

commit 1641870338662a016d5c8a17ef5cada0309f107e
Author: Tapan Avasthi <[[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date:   Sun Jul 31 16:46:56 2022 +0530

    Add feature-1

commit 6ed8a4d2a961fdfc4d5e4c7c00b221ed6f283bf4
Author: Tapan Avasthi <[[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date:   Sun Jul 31 16:46:56 2022 +0530

    Add .gitignore file 
```

此外，我们可以验证`ORIG_HEAD`仍然指向`728ceb3219cc5010eae5840c992072cac7a5da00`提交:

```
$ git log ORIG_HEAD
commit 728ceb3219cc5010eae5840c992072cac7a5da00
Author: Tapan Avasthi <[[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date:   Sun Jul 31 16:46:56 2022 +0530

    Add feature-2

commit 6ed8a4d2a961fdfc4d5e4c7c00b221ed6f283bf4
Author: Tapan Avasthi <[[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date:   Sun Jul 31 16:46:56 2022 +0530

    Add .gitignore file 
```

最后，**我们用`ORIG_HEAD`引用**做一个`reset`:

```
$ git reset --hard ORIG_HEAD
$ git log HEAD -1
commit 728ceb3219cc5010eae5840c992072cac7a5da00
Author: Tapan Avasthi <[[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date:   Sun Jul 31 16:46:56 2022 +0530

    Add feature-2 
```

就是这样！在`ORIG_HEAD`的帮助下，我们成功地恢复了重置基础操作。

## 4.使用`git reflog`

同样，让我们从一个全新的场景设置开始:

```
$ git log HEAD
commit 07b98ef156732ba41e2cbeef7939b5bcc9c364bb
Author: Tapan Avasthi <[[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date:   Sun Jul 31 17:53:35 2022 +0530

    Add feature-2

commit d6c52eb601e3ba11d65e7cb6e99ec6ac6018e272
Author: Tapan Avasthi <[[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date:   Sun Jul 31 17:53:35 2022 +0530

    Add .gitignore file 
```

现在，让我们重新排序并检查提交历史记录:

```
$ git rebase feature1
$ git log HEAD
commit b6ea25bf83ade2caca5ed92f6c5e5e6a3cb2ca7b
Author: Tapan Avasthi <[[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection)al>
Date:   Sun Jul 31 17:53:35 2022 +0530

    Add feature-2

commit d2cabe48747699758e2b14e76fb2ebebfc49acb1
Author: Tapan Avasthi <[[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date:   Sun Jul 31 17:53:35 2022 +0530

    Add feature-1

commit d6c52eb601e3ba11d65e7cb6e99ec6ac6018e272
Author: Tapan Avasthi <[[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date:   Sun Jul 31 17:53:35 2022 +0530

    Add .gitignore file 
```

接下来，让**使用 [`git reflog`](https://web.archive.org/web/20220923112234/https://git-scm.com/docs/git-reflog) 命令在粒度级别检查事件记录**:

```
$ git reflog
b6ea25b [[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection){0}: rebase (continue) (finish): returning to refs/heads/feature2
b6ea25b [[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection){1}: rebase (continue): Add feature-2
d2cabe4 [[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection){2}: rebase (start): checkout feature1
07b98ef [[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection){3}: commit: Add feature-2
d6c52eb [[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection){4}: checkout: moving from feature1 to feature2
d2cabe4 [[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection){5}: commit: Add feature-1
d6c52eb [[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection){6}: checkout: moving from development to feature1
d6c52eb [[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection){7}: Branch: renamed refs/heads/master to refs/heads/development
d6c52eb [[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection){9}: commit (initial): Add .gitignore file 
```

我们可以注意到，git 在内部以粒度级别维护引用，其中在 rebase 操作之前,`HEAD`的位置由`[[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection){3}`引用表示。

因此，作为最后一步，让我们通过执行`git reset`来恢复之前的状态:

```
$ git reset --hard [[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection){3}
$ git log HEAD
commit 07b98ef156732ba41e2cbeef7939b5bcc9c364bb
Author: Tapan Avasthi <[[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date:   Sun Jul 31 17:53:35 2022 +0530

    Add feature-2

commit d6c52eb601e3ba11d65e7cb6e99ec6ac6018e272
Author: Tapan Avasthi <[[email protected]](/web/20220923112234/https://www.baeldung.com/cdn-cgi/l/email-protection)>
Date:   Sun Jul 31 17:53:35 2022 +0530

    Add .gitignore file 
```

太好了！我们也成功地学会了这种方法。但是，**这种方式使用了 git 的一些底层细节，只有高级 git 用户才应该使用。**

## 5.结论

在本文中，我们使用了一个 git 存储库的测试场景，并学习了两种撤销`git rebase`操作的流行技术。