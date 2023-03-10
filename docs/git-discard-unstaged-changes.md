# 如何丢弃 Git 中未暂存的更改

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/git-discard-unstaged-changes>

## 1.概观

一个 [Git](/web/20220524054946/https://www.baeldung.com/git-guide) 工作目录可以包含不同类型的文件，包括暂存文件、未暂存文件和未跟踪文件。

在本教程中，我们将看到如何丢弃工作目录中不在索引中的更改。

## 2.分析工作目录的状态

对于我们的例子，假设我们已经分叉并克隆了一个 [Git 库](https://web.archive.org/web/20220524054946/https://github.com/eugenp/tutorials)，然后对我们的工作目录做了一些更改。

让我们检查工作目录的状态:

```java
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README.md
        modified:   gradle/maven-to-gradle/src/main/java/com/sample/javacode/DisplayTime.java

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        gradle/maven-to-gradle/src/main/java/com/sample/javacode/TimeZones.java

no changes added to commit (use "git add" and/or "git commit -a") 
```

在这里，我们可以看到一些已修改和未转移的文件，以及我们添加的一个新文件。

现在，让我们使用`git add `转移现有的 Java 文件，并再次检查状态:

```java
$ git add gradle/maven-to-gradle/src/main/java/com/sample/javacode/DisplayTime.java
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   gradle/maven-to-gradle/src/main/java/com/sample/javacode/DisplayTime.java

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        gradle/maven-to-gradle/src/main/java/com/sample/javacode/TimeZones.java
```

在这里，我们可以看到我们的工作目录中有三类文件:

*   阶段文件-`DisplayTime.java`
*   未暂存文件-`README.md`
*   未被跟踪的文件-`TimeZones.java`

## 3.删除未跟踪的文件

未跟踪的文件是存储库中的新文件，还没有被添加到版本控制中。**我们可以用`clean`命令**删除这些:

```java
$ git clean -df
```

`-df`选项确保强制删除，并且未跟踪的目录也包括在删除范围内。运行此命令将输出删除了哪些文件:

```java
Removing gradle/maven-to-gradle/src/main/java/com/sample/javacode/TimeZones.java
```

现在，让我们再次检查状态:

```java
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   gradle/maven-to-gradle/src/main/java/com/sample/javacode/DisplayTime.java

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README.md
```

我们可以看到`clean`命令是如何从我们的工作目录中删除未被跟踪的文件的。

## 4.删除未准备提交的更改

现在我们已经删除了未跟踪的文件，剩下的是处理工作目录中的暂存和未暂存文件。我们可以**使用带有“–”选项**的`checkout`命令来删除所有未提交的更改:

```java
$ git checkout -- .
```

让我们在运行命令后再次检查状态:

```java
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   gradle/maven-to-gradle/src/main/java/com/sample/javacode/DisplayTime.java
```

我们可以看到我们的工作目录现在只包含了阶段性的变更。

## 5.结论

在本教程中，我们看到了工作目录如何在当前不受 Git 版本控制的文件中包含暂存、未暂存和未跟踪的文件。

我们还看到了`git clean -df`和`git checkout — .`如何从我们的工作目录中删除所有未暂存的变更。