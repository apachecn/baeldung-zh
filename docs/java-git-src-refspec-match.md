# “src refspec 不匹配任何”的问题

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-git-src-refspec-match>

## 1.概观

使用 [Git](/web/20221103221508/https://www.baeldung.com/git-guide) 是任何开发人员日常工作的重要部分。然而，在开始时，它可能是压倒性的，错误消息可能不明显。**当人们开始使用 Git 时，最常见的问题之一是`refspec:`** 的错误

```java
error: src refspec master does not match any 
error: failed to push some refs to 'https://github.com/profile/repository.git'
```

在本教程中，我们将了解这个问题的原因以及如何解决和减轻它。

## 2.问题的描述

我们中的许多人至少在控制台中看到过一次`refspec` 错误消息。推送到远程存储库时会出现此错误。让我们试着理解这一行的确切含义:

```java
error: src refspec master does not match any
```

**简单来说，这个错误信息告诉我们，我们没有想要推送的分支，这是导致这个错误的主要原因。**

## 3.经历这些步骤

当我们克隆一个未初始化的存储库并试图推送一个本地存储库时，可能会出现`refspec `错误。这就是 Git 服务如何解释建立本地存储库的。**以下是来自 GitHub 的步骤:**

```java
$ echo "# repository" >> README.md
$ git init
$ git add README.md
$ git commit -m "first commit"
$ git branch -M main
$ git remote add origin https://github.com/profile/repository.git
$ git push -u origin main
```

我们将在下面的段落中提到这些步骤。

## 4.推动一个不存在的分支

让我们一步一步地按照 GitHub 提供给我们的指令来做。

### 4.1 初始化存储库

第一行创建了一个`README.md `文件:

```java
$ echo "# repository" >> README.md
```

以下命令将初始化本地 Git 存储库:

```java
$ git init
```

该命令可能会发出以下消息:

```java
hint: Using 'master' as the name for the initial branch. This default branch name
hint: is subject to change. To configure the initial branch name to use in all
hint: of your new repositories, which will suppress this warning, call:
hint: 
hint: 	git config --global init.defaultBranch <name>
hint: 
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint: 
hint: 	git branch -m <name>
```

**在 2020 年 GitHub [将](https://web.archive.org/web/20221103221508/https://github.com/github/renaming)在一个新的存储库中创建的分支的默认名称从`“master”`更改为`“main.”` T5[git lab](https://web.archive.org/web/20221103221508/https://about.gitlab.com/blog/2021/03/10/new-git-default-branch-name/)上也发生了同样的变化。在 [GitHub](https://web.archive.org/web/20221103221508/https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-branches-in-your-repository/changing-the-default-branch) 、 [GitLab](https://web.archive.org/web/20221103221508/https://docs.gitlab.com/ee/user/project/repository/branches/default.html#change-the-default-branch-name-for-a-project) 和 [Git](https://web.archive.org/web/20221103221508/https://github.blog/2020-07-27-highlights-from-git-2-28/) 上仍然可以配置。**

### 4.2.第一次提交

随后的两个命令在我们的本地存储库中创建一个提交:

```java
$ git add README.md 
$ git commit -m "first commit"
```

### 4.3.重命名分支

有趣的事情发生在下面一行:

```java
$ git branch -M main
```

这一行负责将我们当前的本地分支重命名为`“main.”` ,这是因为提示消息中解释的原因。这一行将重命名我们的默认分支，以匹配我们的远程存储库中的默认分支名称。

GitHub 提供的步骤将包含平台上配置的默认名称。**然而，这种剩余成为`refspec`错误背后最常见的原因之一。**让我们看看，如果我们有一个使用`“master”`作为默认分支的本地存储库和一个使用`“main.”` 的远程存储库，会发生什么情况。对于这个例子，我们将跳过重命名步骤，直接设置我们的远程存储库:

```java
$ git remote add origin https://github.com/profile/repository.git
```

### 4.4.问题是

我们将在这条线上开始遇到问题:

```java
$ git push -u origin main
```

让我们回顾这一行并查阅[文档](https://web.archive.org/web/20221103221508/https://git-scm.com/docs/git-push)来理解发生了什么。**这一行将变更从一个分支(在本例中为`“main,”` )推送到我们在前一行中配置的远程存储库。**

这意味着本地存储库应该包含`“main”`分支。**然而，默认的本地分支名称被设置为`“master,”`，我们没有创建新的*“`main`”*分支或重命名`“master”`分支。**在这种情况下，Git 将无法找到要推送的*“`main`”*分支，我们将得到以下错误消息:

```java
error: src refspec main does not match any
error: failed to push some refs to 'origin'
```

现在这条信息更有意义了。如前所述，这个错误告诉我们没有`“main”` 分支。有几种方法可以解决这个问题。第一个是把我们现在的*`master`*分公司改名为`“main”` `:`

```java
$ git branch -M main
```

在重命名操作之后，我们可以重复 push 命令，它将没有问题地工作。同时，我们可以将想要推送的分支的名称从*、`main`、*更改为`master`，或者我们在本地存储库中默认使用的任何名称。以下命令将在远程存储库上创建一个“`master`”分支:

```java
$ git push -u origin master
```

最后，如果我们想在本地存储库中坚持使用"`master” `名称，在远程存储库中坚持使用`“main”`名称，我们可以使用以下命令显式设置上游分支:

```java
$ git push -u origin master:main
```

标志`-u`还将设置本地*`master`*分支和远程`main`分支之间的上游连接。这意味着下一次我们可以在不明确识别上游分支的情况下使用该命令:

`$ git push`

## 5.推送空存储库

这个问题的另一个原因是推送一个空的存储库。然而，背后的原因将是相同的——试图推动一个不存在的分支。让我们假设我们已经创建了一个新的存储库。这些分支被恰当地命名。我们添加了一个文件，但未提交:

```java
$ echo "# another-test-repo" >> README.md
$ git init
$ git add README.md
$ git branch -M main
$ git remote add origin https://github.com/profile/repository.git
$ git push -u origin main
```

**虽然我们在*`main`*分支上，但严格来说，并不存在。对于要在`.git/refs/heads,`下创建的分支，它应该包含至少一个提交。**让我们确保回购中的文件夹`.git/refs/heads `此时为空:

```java
$ ls .git/refs/heads 
```

这个命令应该显示一个空文件夹。因此，和前面的例子一样，我们试图推动一个不存在的分支。一次提交就能解决问题。它将创建一个分支，使推动变更成为可能:

```java
$ git commit -m "first commit"
$ git push -u origin master
```

## 6.结论

创建和初始化一个新的本地存储库并不是一项具有挑战性的任务。然而，跳过步骤或盲目遵循指示可能会导致错误。这些错误有时是不容易理解的，尤其是对于新的 Git 用户。

在这篇文章中，我们已经学习了如何处理`refspec`错误以及错误背后的原因。