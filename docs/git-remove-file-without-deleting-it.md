# 从 Git 储存库中移除文件，而不在本地删除它

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/git-remove-file-without-deleting-it>

## 1.概观

Git 已经成为一个广泛使用的分布式版本控制系统。在本教程中，让我们探索如何从 Git 存储库中删除文件或目录，但保留其本地副本。

## 2.问题简介

像往常一样，让我们通过一个例子来理解这个问题。假设我们正在开发一个 Git 库`myRepo`:

```java
$ ls -l
total 12
drwxr-xr-x 2 kent kent 60 May 12 23:00 logs/
-rw-r--r-- 1 kent kent 26 May 11 13:22 README.md
-rw-r--r-- 1 kent kent 21 May 11 13:22 some-file.txt
-rw-r--r-- 1 kent kent 16 May 12 22:40 user-list.txt
```

我们已经将存储库克隆到本地，正如`ls`输出所示，我们在存储库中有三个文件和一个`logs`目录。

现在，假设我们想从 Git 存储库中删除文件`user-list.txt`和`logs`目录。但是，我们不想将它们从本地工作副本中删除。

一个常见的场景是，我们已经提交了一些文件或目录，然后意识到我们应该忽略一些文件。因此，**我们将从存储库中移除相关文件，保留本地副本，并将相应的模式添加到`.gitignore`文件**中，这样 Git 就不会再跟踪这些文件了。

我们知道`git rm user-list.txt`命令会将文件从存储库中删除。但是，它也会删除本地文件。

当然，我们可以将文件和目录移动到另一个目录，提交一个 commit，然后将它们复制回本地工作目录。它解决了问题。然而，这种方法效率很低，尤其是当文件或目录很大时。

接下来，我们来看看如何更高效地解决这个问题。

## 3.使用`git rm –cached`命令

我们已经提到过 **`git rm FILE` 会默认从索引和本地工作树**中移除文件。

然而，`git rm`命令提供了`–cached`选项，只允许我们从存储库的索引中删除文件，并保持本地文件不变。

接下来，让我们用`user-list.txt`文件试试:

```java
$ git rm --cached user-list.txt
rm 'user-list.txt' 
```

如上面的输出所示，`user-list.txt`文件已经被删除。现在，让我们执行`git status`命令来验证它:

```java
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	deleted:    user-list.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	user-list.txt 
```

我们可以看到，`user-list.txt`就是`deleted`。此外，由于它的本地副本仍在那里，它已被标记为“未跟踪”。

我们可以类似地删除`logs`目录。然而，由于它是一个目录，我们需要额外将`-r (recursively)`选项传递给`git rm`命令:

```java
$ git rm --cached -r logs
rm 'logs/server.log'
```

现在，让我们提交我们的更改:

```java
$ git commit -m 'remove user-list.txt and logs'
[master ee8cfe8] remove user-list.txt and logs
 2 files changed, 4 deletions(-)
 delete mode 100644 logs/server.log
 delete mode 100644 user-list.txt 
```

然后，让我们使用`[git ls-files](https://web.archive.org/web/20220811181129/https://git-scm.com/docs/git-ls-files) `命令检查当前暂存的文件:

```java
$ git ls-files -c
.gitignore
README.md
some-file.txt 
```

如输出所示，目标文件和目录不再存在。此外，还会保留本地副本。因此，我们已经解决了这个问题。

如果我们愿意，我们可以将它们添加到`.gitignore`文件中，以防止 Git 再次跟踪它们。

## 4.删除`.gitignore`中定义的所有文件

有时，我们希望检查 Git 的索引，并删除所有在`.gitignore`中定义的文件。假设我们已经完成了我们的 `.gitignore`定义。那么，一个简单的方法就是三步走:

*   首先，从索引中删除所有文件:`git rm -r –cached`
*   然后，再次转移所有文件。在`.gitignore`中定义的文件将被自动忽略:`git add`
*   提交我们的更改:`git commit -m “a proper commit message”`

或者，**我们可以只找到并删除当前被跟踪但应该被忽略的文件**。命令可以帮助我们找到文件。

让我们恢复之前的提交，并再次删除`user-list.txt`文件和`logs`目录。这一次，让我们先把它们添加到`.gitignore`文件中:

```java
$ cat .gitignore
user-list.txt
logs/ 
```

接下来，让我们找出想要从 Git 索引中删除的文件:

```java
$ git ls-files -i -c -X .gitignore
logs/server.log
user-list.txt 
```

正如我们所看到的，上面的命令列出了我们想要删除的暂存文件。

现在，让**将`git rm –cached `和`git ls-files`命令结合起来，一次性删除它们**:

```java
$ git rm --cached $(git ls-files -i -c -X .gitignore)
rm 'logs/server.log'
rm 'user-list.txt'
```

值得一提的是，该命令将删除`logs`目录下的所有文件，所以最后，它将从索引中删除空的 logs 目录。因此，在这个例子中，我们在`logs`目录下只有一个文件。

现在，如果我们检查转移的文件，被删除的文件就不见了:

```java
$ git ls-files -c
.gitignore
README.md
some-file.txt 
```

当然，`user-list.txt`和`logs/`仍然在我们的本地工作树中:

```java
$ ls -l
total 12
drwxr-xr-x 2 kent kent 60 May 13 00:45 logs/
-rw-r--r-- 1 kent kent 26 May 11 13:22 README.md
-rw-r--r-- 1 kent kent 21 May 11 13:22 some-file.txt
-rw-r--r-- 1 kent kent 16 May 13 00:45 user-list.txt
```

## 5.移除的文件仍然在 Git 历史中

我们已经使用`git rm –cached`命令解决了我们的问题。然而，我们应该记住**我们只是从 Git 的跟踪索引**中删除了这个文件。我们仍然可以在 Git 的提交历史中看到该文件及其内容。例如，我们仍然可以通过检查之前的提交来查看`user-list.txt`的内容:

```java
$ git show 668fa2f user-list.txt
commit 668fa2f...
Author: ...
Date:   ...

    add user-list.txt and some-file.txt

diff --git a/user-list.txt b/user-list.txt
new file mode 100644
index 0000000..3da7fab
--- /dev/null
+++ b/user-list.txt
@@ -0,0 +1,3 @@
+kent
+eric
+kevin 
```

了解这一点很重要，因为有时我们会忘记将一些敏感文件添加到`.gitingore`文件中，比如凭证。但是我们已经提交了它们，并将更改推送到远程存储库。在我们意识到这一点后，我们可能希望将敏感文件从 Git 历史中彻底清除。

如果是这种情况，我们需要[从 Git 的提交历史](/web/20220811181129/https://www.baeldung.com/git-remove-file-commit-history)中删除文件。

## 6.结论

在本文中，我们通过示例展示了如何从 Git 存储库中删除文件或目录，但保留其本地副本。

此外，我们还提出了一种快速删除在`.gitignore`文件中定义的所有文件的方法。最后，我们应该记住被`git rm –cached`删除的文件仍然存在于 Git 的提交历史中。