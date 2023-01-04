# 在 Git 中修改指定的提交

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/git-modify-commit>

## 1.概观

在本文中，我们将探索修改 [Git](/web/20220810181114/https://www.baeldung.com/git-guide) 提交的不同方法。

## 2.使用`amend`

我们可以通过简单地使用*修改*选项来修改最新的 Git 提交。它会替换最近的提交。我们可以修改提交消息并更新提交中包含的文件。Git 将修改后的提交视为新的提交。

让我们用一个例子来试试*修正*选项。为了简单起见，让我们更新一个文件并用消息“Commit 1”进行提交。现在，让我们尝试使用*修改*选项来更新提交:

```java
git commit --amend
```

执行上面的命令会打开一个编辑器来包含更改。让我们更新提交消息并保存更改。关闭编辑器后，我们可以看到更新后的提交，如下所示:

```java
[master c0bc5d3] Amended Commit 1
 Date: Wed Jun 29 22:41:08 2022 +0530
 1 file changed, 1 insertion(+), 1 deletion(-) 
```

在修改提交时，我们还可以包含分阶段的更改。让我们创建额外的更改，并使用 *amend* 选项将它们包含在最新的提交中，再次更改提交消息:

```java
[master 0a1d571] Amended Commit 1 - Added new file
 Date: Wed Jun 29 22:41:08 2022 +0530
 2 files changed, 2 insertions(+), 1 deletion(-)
 create mode 100644 README2 
```

如果我们只想添加阶段化的更改而不更新提交消息，我们可以使用 *no-edit* 选项:

```java
git commit --amend --no-edit
```

因此，我们可以看到, *amend* 选项是向最近提交添加更改的一种便捷方式。现在，让我们探索更新 Git 历史中旧提交的不同方法。

## 3.使用 [`rebase`](/web/20220810181114/https://www.baeldung.com/git-merge-vs-rebase#git-rebase)

我们可以使用 *rebase* 命令将一系列提交转移到一个新的基础上。Git 在内部为每个旧的提交创建一个新的提交，并移动到指定的新基础。

使用 *-i* 选项和 *rebase* 命令启动一个交互式会话。在此会话期间，如果需要，我们可以使用以下命令修改每个提交:

*   ***【p】*****->包括具体的提交**
***   ***挤压***->将提交与之前的提交合并*   ***删除** (d)* - >删除具体提交*   ***重新措辞** (r)* - >包含提交并更新提交消息*   ***编辑** (e)* - >包含提交，并带有更新所包含文件的选项**

 **让我们尝试使用上面的命令更新我们示例中的提交历史。在这一步， *git 日志*显示了以下提交:

```java
commit 5742fcbe1cb14a9c4f1425eea9032ffb4c6191e5 (HEAD -> master)
Author: #####
Date:   Fri Jul 1 08:11:52 2022 +0530
    commit 5
commit e9ed266b84dd29095577ddd8f6dc7fcf5cf9db0d
Author: #####
Date:   Fri Jul 1 08:11:37 2022 +0530
    commit 4
commit 080e3ecc041b7be1757af67bf03db982135b9093
Author: #####
Date:   Fri Jul 1 08:11:18 2022 +0530
    commit 3
commit d5923e0ced1caff5874d8d41f39d197b5e1e2468
Author: #####
Date:   Fri Jul 1 08:10:58 2022 +0530
    commit 2
commit 1376dc1182a798b16dc85239ec7382e8340d5267
Author: #####
Date:   Wed Jun 29 22:41:08 2022 +0530
    Amended Commit 1 - Added new file 
```

假设我们想要更改在提交“`Amended Commit 1 – Added new file`”之后提交的修改。

让我们将上面的提交设置为新的基础:

```java
git rebase -i 1376dc1182a798b16dc85239ec7382e8340d5267
```

这将打开一个编辑器，我们可以在其中根据需要进行更改:

```java
pick d5923e0 commit 2
pick 080e3ec commit 3
pick e9ed266 commit 4
pick 5742fcb commit 5
# Rebase #####..### onto #### (4 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

现在，让我们尝试几个命令来更新 Git 历史:

```java
reword d5923e0 commit 2-updated
edit 080e3ec commit 3
squash e9ed266 commit 4
drop 5742fcb commit 5
```

在第一个命令之后，我们将看到输出:

```java
[detached HEAD 178e8eb] commit 2-alter
 Date: Fri Jul 1 08:10:58 2022 +0530
 1 file changed, 1 insertion(+)
Stopped at ######...  commit 3
You can amend the commit now, with

  git commit --amend 

Once you are satisfied with your changes, run

  git rebase --continue
```

现在，正如 Git 输出中所指定的，我们执行*Git commit–amend*命令并做一些更改:

```java
commit 3          
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Fri Jul 1 08:11:18 2022 +0530
#
# interactive rebase in progress; onto 1376dc1
# Last commands done (2 commands done):
#    reword d5923e0 commit 2-updated
#    edit 080e3ec commit 3
# Next commands to do (2 remaining commands):
#    squash e9ed266 commit 4
#    drop 5742fcb commit 5
# You are currently splitting a commit while rebasing branch 'master' on '1376dc1'.
#
# Changes to be committed:
#       modified:   README
#       modified:   README2
```

关闭编辑器后，我们得到以下输出:

```java
[detached HEAD 9433120] commit 3 - updated
 Date: Fri Jul 1 08:11:18 2022 +0530
 2 files changed, 3 insertions(+), 1 deletion(-) 
```

我们在这里可以看到，我们不仅更新了提交消息，还在提交过程中添加了一个文件。

接下来，我们需要执行*git rebase–continue*命令来进行下一次更新。

我们的下一步涉及到用*提交 3* 挤压*提交 4* 的*和*，它打开了下面的编辑器:

```java
# This is a combination of 2 commits.
# This is the 1st commit message:
commit 3 - updated
# This is the commit message #2:
commit 4
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# Date:      Fri Jul 1 08:11:18 2022 +0530
# interactive rebase in progress; onto 1376dc1
# Last commands done (3 commands done):
#    edit 080e3ec commit 3
#    squash e9ed266 commit 4
# Next command to do (1 remaining command):
#    drop 5742fcb commit 5
# You are currently rebasing branch 'master' on '1376dc1'.
#
# Changes to be committed:
#       modified:   README
#       modified:   README2 
```

在上面的编辑器中添加修改后，我们得到下面的输出:

```java
[detached HEAD 917c583] commit 3 - squashed with commit 4
Date: Fri Jul 1 08:11:18 2022 +0530
2 files changed, 3 insertions(+), 1 deletion(-)
Successfully rebased and updated refs/heads/master. 
```

最后，我们已经请求*删除**提交 5* ，这不需要我们做任何进一步的修改。

### 4.分析日志

执行上述步骤后，我们可以看到`git log`的输出如下:

```java
commit 917c583d5bb02803ee43cf87a2143f201c97bbe8 (HEAD -> master)
Author: #######
Date:   Fri Jul 1 08:11:18 2022 +0530
commit 3 - squashed with commit 4
commit 4
commit 178e8ebec178c166d1c9def2d680f41933eba29b
Author: #######
Date:   Fri Jul 1 08:10:58 2022 +0530
commit 2-alter
commit 1376dc1182a798b16dc85239ec7382e8340d5267
Author: #######
Date:   Wed Jun 29 22:41:08 2022 +0530
Amended Commit 1 - Added new file 
```

在这里，我们可以提出几点意见:

*   *提交 5* 被移除
*   *提交 4* 与*提交 3* 合并
*   *提交 2* 消息被更新

在最终的日志中，我们可以看到提交的 id 现在已经更改。这是因为 Git 已经替换了以前的提交，并创建了修改后的新提交。

我们可以使用`reflog` 命令查看与当前`HEAD`相关的参考日志。它包括与 Git 提交相关的所有更新的历史。

让我们使用 *reflog* 命令来观察我们示例的 git 历史:

```java
917c583 (HEAD -> master) [[email protected]](/web/20220810181114/https://www.baeldung.com/cdn-cgi/l/email-protection){0}: rebase -i (finish): returning to refs/heads/master
917c583 (HEAD -> master) [[email protected]](/web/20220810181114/https://www.baeldung.com/cdn-cgi/l/email-protection){1}: rebase -i (squash): commit 3 - squashed with commit 4
9433120 [[email protected]](/web/20220810181114/https://www.baeldung.com/cdn-cgi/l/email-protection){2}: commit (amend): commit 3 - updated
f4e8340 [[email protected]](/web/20220810181114/https://www.baeldung.com/cdn-cgi/l/email-protection){3}: commit (amend): commit 3 - updated
fd048e1 [[email protected]](/web/20220810181114/https://www.baeldung.com/cdn-cgi/l/email-protection){4}: commit (amend): commit 3 - updated
39b2f1b [[email protected]](/web/20220810181114/https://www.baeldung.com/cdn-cgi/l/email-protection){5}: commit (amend): commit 3 - updated
f79cbfb [[email protected]](/web/20220810181114/https://www.baeldung.com/cdn-cgi/l/email-protection){6}: rebase -i (edit): commit 3
178e8eb [[email protected]](/web/20220810181114/https://www.baeldung.com/cdn-cgi/l/email-protection){7}: rebase -i (reword): commit 2-alter
d5923e0 [[email protected]](/web/20220810181114/https://www.baeldung.com/cdn-cgi/l/email-protection){8}: rebase -i: fast-forward
1376dc1 [[email protected]](/web/20220810181114/https://www.baeldung.com/cdn-cgi/l/email-protection){9}: rebase -i (start): checkout 1376dc1182a798b16dc85239ec7382e8340d5267
5742fcb [[email protected]](/web/20220810181114/https://www.baeldung.com/cdn-cgi/l/email-protection){10}: commit: commit 5
e9ed266 [[email protected]](/web/20220810181114/https://www.baeldung.com/cdn-cgi/l/email-protection){11}: commit: commit 4
080e3ec [[email protected]](/web/20220810181114/https://www.baeldung.com/cdn-cgi/l/email-protection){12}: commit: commit 3
d5923e0 [[email protected]](/web/20220810181114/https://www.baeldung.com/cdn-cgi/l/email-protection){13}: commit: commit 2
1376dc1 [[email protected]](/web/20220810181114/https://www.baeldung.com/cdn-cgi/l/email-protection){14}: commit (amend): Amended Commit 1 - Added new file
```

## 5.结论

在本文中，我们探索了改变 Git 历史的不同方法。但是，我们在使用这些选项时应该小心，因为这也可能导致内容丢失。**