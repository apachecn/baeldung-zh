# 在 Git 中撤销和恢复提交

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/git-revert-commit>

## 1.介绍

我们经常发现自己在使用 Git 时需要撤销或恢复提交，无论是回滚到特定的时间点还是恢复一个特别麻烦的提交。在本教程中，我们将介绍 Git 中撤销和恢复提交的最常用命令。我们还将展示这些命令在运行方式上的细微差别。

## 2.使用`git checkout`查看旧提交

首先，我们可以通过使用`git checkout`命令来查看项目在特定提交时的状态。我们可以通过使用`git log`命令来查看 Git 存储库的历史。每个提交都有一个唯一的 SHA-1 标识散列，我们可以使用`git checkout`来重新访问时间线中的任何提交。

在这个例子中，我们将重新访问一个具有标识哈希`e0390cd8d75dc0f1115ca9f350ac1a27fddba67d` :的提交

```java
git checkout e0390cd8d75dc0f1115ca9f350ac1a27fddba67d
```

我们的`working directory`现在将匹配我们指定的提交的确切状态。因此，我们**能够查看项目的历史状态并编辑文件，而不用担心丢失当前的项目状态。**我们在这里所做的一切都不会保存到存储库中。这就是所谓的`detached HEAD`状态。

我们可以在本地修改的文件上使用`git checkout`来将它们恢复到它们的`working copy`版本。

## 3.使用`git revert`恢复提交

我们通过使用`git revert` 命令来恢复 Git 中的提交。重要的是要记住这个命令不是传统的撤销操作。相反，它反转由提交引入的更改，并生成具有相反内容的新提交。

这意味着`git revert` **应该只在我们想要应用特定提交**的逆操作时使用。它不会通过删除所有后续提交来恢复到项目的先前状态，而是撤消一次提交。

`git revert`不会将引用指针移动到我们正在恢复的提交，这与其他“撤销”命令如`git checkout` 和`git reset`形成对比。相反，这些命令将`HEAD` ref 指针移动到指定的提交。

让我们看一个恢复提交的例子:

```java
mkdir git_revert_example
cd git_revert_example/
git init .
touch test_file
echo "Test content" >> test_file 
git add test_file
git commit -m "Adding content to test file"
echo "More test content" >> test_file 
git add test_file
git commit -m "Adding more test content"
git log
git revert e0390cd8d75dc0f1115ca9f350ac1a27fddba67d
cat test_file
```

在这个例子中，我们已经创建了一个 test_file，添加了一些内容，并提交了它。然后，在运行一个`git log`来识别我们想要恢复的提交的`commit hash`之前，我们已经向文件添加并提交了更多的内容。

在这种情况下，我们正在恢复最近的提交。最后，我们运行了`git revert` 并验证了提交中的更改通过输出文件的内容被恢复。

## 4.使用`git reset`恢复到之前的项目状态

通过使用`git reset`命令，可以使用 Git 恢复到项目中的先前状态。此工具可撤销更复杂的更改。它有三种与 Git 的`internal state management system`相关的主要调用形式:`–hard`、`–soft`和`–mixed`。理解使用哪个调用是执行`git revert`最复杂的部分。

`git reset`在行为上与`git checkout`相似。但是，`git reset` **会移动`HEAD`参考指针，而`git checkout`操作`HEAD`参考指针，不移动**。

为了理解不同的调用，我们来看看 Git 的`internal state management system`——也称为 Git 的`three trees`。

第一棵树是`the working directory`。此树与本地文件系统同步，表示对文件和目录中的内容所做的即时更改。

接下来，我们有了`staging index tree`。这个树跟踪`the working directory`中的变更——换句话说，用`git add`选择的要在下一次提交中存储的变更。

最后一棵树是`commit history`。`git commit`命令将更改添加到存储在`commit history`中的永久快照中。

### 4.1.`–hard`

对于这种调用，最危险和最常用的选项是提交历史引用指针，以更新指定的提交。在此之后，`staging index`和`working index`复位以匹配指定的提交。对`staging index`和`working directory`的任何先前待定的改变被重置以匹配`commit tree`的状态。**我们将在*暂存索引*和`working index`中丢失一个** **ny 未完成或未提交的工作。**

根据上面的例子，让我们向文件提交更多的内容，并向存储库提交一个全新的文件:

```java
echo "Text to be committed" >> test_file
git add test_file
touch new_test_file
git add new_test_file git commit -m "More text added to test_file, added new_test_file"
```

假设我们决定恢复到存储库中的第一次提交。我们将通过运行以下命令来实现这一点:

```java
git reset --hard 9d6bedfd771f73373348f8337cf60915372d7954
```

Git 会告诉我们`HEAD`现在位于指定的提交散列中。查看`test_file`的内容可以看到，我们的**最新添加的文本不存在，我们的`new_test_file`也不再存在**。这种数据丢失是不可逆的，所以我们理解`–hard`如何与 Git 的三棵树一起工作是至关重要的。

### 4.2.`–soft`

当使用`–soft`进行调用时，ref 指针被更新，复位停止。因此，`staging index`和`working directory`保持相同的状态。

**在我们之前的例子中，如果我们使用`–soft`参数**，我们提交给`staging index`的更改就不会被删除。我们仍然能够在`staging index`中提交我们的变更。

### 4.3.`–mixed`

默认的操作模式，如果没有参数被传递，`–mixed`在`–soft`和`–hard`调用之间提供了一个中间地带。`staging index`重置为指定提交和引用指针更新的状态。从`staging index`到`working directory`的任何未完成的更改。

在上面的例子中使用`–mixed` 意味着我们对文件的本地更改不会被删除。然而，与`–soft`不同的是，变化**从`staging index`中被撤销，并等待进一步的动作**。

## 5.结论

比较两种方法的一个简单方法就是 **`git revert`安全，`git reset`危险**。正如我们在例子中看到的，有可能会丢失与`git reset`的工作。有了`git revert`，我们可以安全地撤销公共提交，而`git reset`是为撤销`working directory`和`staging index.`中的本地更改而定制的

`git reset`将移动`HEAD` ref 指针，而`git revert` 将简单地恢复一个提交，并通过对`HEAD`的新提交来应用撤销。同样重要的是要注意到，当任何后续的快照被推送到共享库时，我们永远不应该使用`git reset`。我们必须假设其他开发人员依赖于发布的提交。