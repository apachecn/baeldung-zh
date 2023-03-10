# 本地和远程删除 Git 分支

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/git-delete-branch-locally-remotely>

## 1.概观

[Git](/web/20221003155508/https://www.baeldung.com/git-guide) 作为版本控制系统在业界已经被广泛使用。此外，Git 分支是我们日常开发过程的一部分。

在本教程中，我们将探索如何删除 Git 分支。

## 2.Git 存储库的准备

为了更容易地处理如何删除 Git 分支，让我们首先准备一个 Git 存储库作为例子。

首先，让我们从 GitHub 克隆`myRepo`库(`https://github.com/sk1418/myRepo`)进行测试:

```java
$ git clone [[email protected]](/web/20221003155508/https://www.baeldung.com/cdn-cgi/l/email-protection):sk1418/myRepo.git
Cloning into 'myRepo'...
...
remote: Total 6 (delta 0), reused 3 (delta 0), pack-reused 0
Receiving objects: 100% (6/6), done
```

其次，让我们进入本地`myRepo`目录并检查分支:

```java
$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master 
```

正如我们从上面的输出中看到的，目前我们在`myRepo`存储库中只有一个`master`分支。另外，**`master`分支是`myRepo`的默认分支。**

接下来，让我们创建一些分支，并展示如何本地和远程删除分支。在本教程中，我们将重点关注在命令行中删除分支。

## 3.删除本地分支

让我们先来看看删除一个本地分支。

**Git 的 [`git branch`](/web/20221003155508/https://www.baeldung.com/git-guide#111-git-branch--manage-branches) 命令有两个删除本地分支的选项:`-d`和`-D`** 。

接下来，让我们仔细看看它们，通过一个例子来了解这两个选项的区别。

### 3.1.使用`-d`选项删除本地分支

首先，让我们尝试创建一个本地分支:

```java
$ git checkout -b feature
Switched to a new branch 'feature'
```

接下来，让我们使用`-d`选项删除`feature`分支:

```java
$ git branch -d feature
error: Cannot delete branch 'feature' checked out at '/tmp/test/myRepo'
```

哎呀，正如我们所看到的，我们收到了一条错误消息。这是因为我们目前在`feature`分支:

```java
$ git branch
* feature
  master
```

**换句话说，我们不能删除当前检出的分支。**所以，让我们切换到`master`分支，再次发出命令:

```java
$ git checkout master
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
$ git branch -d feature
Deleted branch feature (was 3aac499)

$ git branch -a
* master
remotes/origin/HEAD -> origin/master
remotes/origin/master
```

如我们所见，我们已经成功删除了本地`feature`分支。

### 3.2.使用`-D`选项删除本地分支

首先，让我们再次创建`feature`分支。但这一次，我们将做一些更改并提交:

```java
$ git checkout -b feature
Switched to a new branch 'feature'

# ... modify the README.md file ...
$ echo "new feature" >> README.md
$ git status
On branch feature
Changes not staged for commit:
...
	modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")

$ git ci -am'add "feature" to the readme'
[feature 4a87db9] add "feature" to the readme
 1 file changed, 1 insertion(+)
```

现在，如果我们仍然使用`-d`选项，Git 将拒绝删除`feature`分支:

```java
$ git checkout master
Switched to branch 'master'
Your branch is up to date with 'origin/master'.

$ git branch -d feature
error: The branch 'feature' is not fully merged.
If you are sure you want to delete it, run 'git branch -D feature'.
```

**这是因为要删除的分支(`feature`)在默认分支(`master` )** 之前:

```java
$ git log --graph --abbrev-commit 
* commit 4a87db9 (HEAD -> feature)
| Author: ...
| Date:   ...| 
|     add "feature" to the readme
| 
* commit 3aac499 (origin/master, origin/HEAD, master)
| Author: ...
| Date:   ...| 
|     the first commit
| 
* commit e1ccb56
  Author: ...
  Date:   ...  
      Initial commit 
```

有两种方法可以解决这个问题。首先，我们可以将`feature`分支合并成`master`，然后再次执行`git branch -d feature`。

然而，如果我们想要丢弃未合并的提交，正如错误消息所建议的，我们可以**运行`git branch -D feature`来执行强制删除:**

```java
$ git branch -D feature
Deleted branch feature (was 4a87db9)

$ git branch -a
* master
remotes/origin/HEAD -> origin/master
remotes/origin/master.
```

### 3.3.`git branch -d/-D` 不会删除远程分支

到目前为止，我们已经使用带有`-d`和`-D`选项的`git branch`删除了一个本地分支。值得一提的是，无论我们用`-d`还是`-D`删除，这个命令只会删除本地分支。**不会删除远程分支，即使被删除的本地分支正在跟踪远程分支**。

接下来，我们通过一个例子来理解这一点。同样，让我们创建一个`feature`分支，进行一些更改，并将提交推送到远程存储库:

```java
$ git checkout -b feature
Switched to a new branch 'feature'

# add a new file
$ echo "a wonderful new file" > wonderful.txt

$ git add . && git ci -am'add wonderful.txt'
[feature 2dd012d] add wonderful.txt
 1 file changed, 1 insertion(+)
 create mode 100644 wonderful.txt
$ git push
...
To github.com:sk1418/myRepo.git
 * [new branch]      feature -> feature
```

如上面的输出所示，我们在`feature`分支上创建了一个新文件`wonderful.txt,`，并将提交推送到远程存储库。

因此，本地`feature`分支正在跟踪远程`feature`分支:

```java
$ git remote show origin | grep feature
    feature tracked
    feature pushes to feature (up to date)
```

由于我们没有将`feature `合并到`master`中，让我们用`-D`选项删除局部特征分支:

```java
$ git checkout master
Switched to branch 'master'
Your branch is up to date with 'origin/master'.

$ git branch -D feature
Deleted branch feature (was 2dd012d).

$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/feature
  remotes/origin/master
```

正如我们在命令`git branch -a`的输出中看到的，本地`feature`分支消失了。但是`/remotes/origin/feature`分支并没有被移除。

现在，如果我们再次检查`feature`分支，我们所做的更改仍然存在:

```java
$ git checkout feature
Switched to branch 'feature'
Your branch is up to date with 'origin/feature'.
$ cat wonderful.txt 
a wonderful new file 
```

接下来，让我们看看如何删除远程分支。

## 4.删除远程分支

如果我们的 Git 版本低于 1.7.0，我们可以使用命令`git push origin :<branchName>`删除远程分支。但是，这个命令看起来不像删除操作。因此，**从版本 1.7.0 开始，Git 引入了`git push origin -d <branchName>`命令来删除远程分支**。显然，这个命令比旧版本更容易理解和记忆。

刚才，我们删除了本地分支 `feature`，我们看到远程分支`feature`仍然存在。所以现在，让我们使用提到的命令删除远程`feature`分支。

在我们删除远程`feature`之前，让我们首先创建一个本地`feature`分支来跟踪远程分支。这是因为我们希望检查删除远程分支是否会影响本地分支的跟踪:

```java
$ git checkout feature 
branch 'feature' set up to track 'origin/feature'.
Switched to a new branch 'feature'
$ git branch -a
* feature
  master
  remotes/origin/HEAD -> origin/master
  remotes/origin/feature
  remotes/origin/master
```

所以，现在我们有了本地和远程分支机构。此外，我们目前在当地的`feature`分公司。

接下来，让我们删除远程`feature`分支:

```java
$ git push origin -d feature
To github.com:sk1418/myRepo.git
 - [deleted]         feature
$ git branch -a
* feature
  master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master 
```

正如我们所看到的，在我们执行了`git push -d feature`命令之后，远程`feature`分支被删除了。但是，当地的`feature`分公司还在。也就是说，**删除远程分支不会影响本地跟踪分支**。因此，如果我们现在推出`git push`，本地的`feature`分支将再次被推到远程。

此外，与本地分支删除不同，**我们可以删除一个远程分支，不管我们当前正在哪个本地分支上工作**。在上面的例子中，我们在本地`feature`分支上，但是我们仍然可以删除远程`feature`分支，没有任何问题。

## 5.结论

在本文中，我们探索了如何使用命令删除 Git 的本地和远程分支。

我们来快速总结一下:

*   删除本地分支:`git branch -d/-D <branchName>`(`-D`选项用于强制删除)
*   删除远程分支:`git push origin -d <branchName>`或 `git push origin :<branchName>`

此外，我们知道删除本地或远程的分支不会影响另一端的分支。