# 找出两个 Git 分支之间的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/git-branches-differences>

## 1.概观

在本教程中，我们将发现寻找两个`[git](/web/20220911180309/https://www.baeldung.com/git-guide#what-is-git)`分支之间差异的方法。我们将探索 [`git diff`](https://web.archive.org/web/20220911180309/https://git-scm.com/docs/git-diff) 命令，并在分支比较中使用它。

## 2.在单个命令中比较分支

`git diff`是一个有用的命令，它允许我们比较不同类型的 git 对象，比如文件、提交、分支等等。**当我们需要比较两个分支之间的差异时，这使得`git diff`成为一个很好的选择。**

为了比较分支，我们在`git diff`命令后指定了两个分支的名称:

```java
$ git diff branch1 branch2 
diff --git a/file1.txt b/file1.txt
index 3b18e51..c28f4fa 100644
--- a/file1.txt
+++ b/file1.txt
@@ -1 +1 @@
-hello world
+hello from branch2
```

让我们检查输出。第一行`diff ‐‐git a/file1.txt b/file1.txt` 显示了分支中的不同文件。在我们的例子中，文件`file1.txt,`被表示为 `branch1,` 的`a/file1.txt` ，而`b/file1.txt` 是`branch2`的同一个文件。

下一行显示了两个分支中`file1.txt`的索引缓存号。然后，有几行显示了添加了内容的文件名，以及删除了内容的文件名。

最后，最重要的行在输出的末尾，我们可以看到修改后的行。我们可以看到来自`branch1`的文件 `file1.txt` 中的行`“hello world”` 被替换为`branch2`中的行`“hello from branch2”` 。

## 3.仅显示文件的名称

虽然我们在上面的例子中发现了两个分支之间的差异，但是显示每一行的变化是不方便的。通常，我们只想看到被更改文件的名称。

**为了只显示两个分支之间不同的文件名，我们使用了`git diff`命令**中的`‐‐name-only` 选项:

```java
$ git diff branch1 branch2 --name-only
file1.txt
```

现在，输出只显示两个分支中不同的文件名。在我们的例子中，它只是一个文件`file1.txt`。

## 4.比较分支的另一种方法

在上面的例子中，我们使用了一个`git diff`命令来查找分支之间的差异。然而，我们必须在这个命令中指定两个分支的名称。有时，如果我们从其中一个分支内部进行比较，感觉会更直观。

为了检查一个分支，我们使用`[git checkout](https://web.archive.org/web/20220911180309/https://git-scm.com/docs/git-checkout)`命令。在这之后，我们只需要`diff `另一个分支，并看到相对于我们当前分支的所有变化。

例如，让我们从`branch2`内部比较一下`branch1`和`branch2`:

```java
$ git checkout branch2 
$ git diff branch1
```

我们现在应该能够看到与之前相同的输出。

## 5.从一个共同的祖先身上寻找差异

到目前为止，我们比较了两个分支的最新状态。但是，有时我们需要将一个分支与另一个分支的共同祖先进行比较。

换句话说，我们可能需要找出分支中与另一个分支相比的所有差异，因为它们是分支出来的。为了便于说明，让我们看看下面的树:

```java
---A---B---C---D  <== branch1
        \
         E---F    <== branch2
```

我们想找出`branch2`(当前提交位置 F)和`branch1`在其提交位置 B 的差异。正如我们所看到的，在位置 B，两个分支分叉，然后它们彼此分开发展。在这种情况下，位置 B 是两个分支的共同祖先。

为了找出`branch2`和`branch1`共同祖先之间的区别，我们需要在分支名称之间使用三个点:

```java
git diff branch1...branch2
```

输出将采用与前面类似的格式。然而，现在比较发生在`branch2` 和`branch1` `.`的共同祖先之间

## 6.找出提交哈希之间的差异

不仅可以显示分支之间的差异，还可以显示提交之间的差异。

其语法类似于分支比较。但是，我们需要指定要比较的提交散列，而不是分支:

```java
git diff b94a88bac17318fb3c3cc881d657c04de9fd7901 73ea8956375c10fe41c669ba8c6f6f9e01490452
```

输出的格式类似于上面的例子。

## 7.结论

在本文中，我们学习了如何使用`git diff`命令找到两个 git 分支之间的差异。

我们首先使用一个命令来比较分支。然后，我们发现了如何使用`git checkout`命令比较一个分支中的分支。

最后，我们看到了如何将分支与共同的祖先进行比较，以及找出提交之间的差异。