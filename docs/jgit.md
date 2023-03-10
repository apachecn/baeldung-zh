# JGit 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jgit>

## 1。简介

JGit 是 Git 版本控制系统的轻量级纯 Java 库实现——包括存储库访问例程、网络协议和核心版本控制算法。

JGit 是用 Java 编写的功能相对完整的 Git 实现，在 Java 社区中广泛使用。JGit 项目在 Eclipse 的保护伞下，它的家可以在 [JGit](https://web.archive.org/web/20221129011837/https://eclipse.org/jgit/) 找到。

在本教程中，我们将解释如何使用它。

## 2。入门

有许多方法可以将您的项目与 JGit 连接起来并开始编写代码。最简单的方法可能是使用 Maven——通过向我们的 `pom.xml`文件中的`<dependencies>`标签添加以下代码片段来完成集成:

```java
<dependency>
    <groupId>org.eclipse.jgit</groupId>
    <artifactId>org.eclipse.jgit</artifactId>
    <version>4.6.0.201612231935-r</version>
</dependency>
```

请访问 [Maven 中央存储库](https://web.archive.org/web/20221129011837/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22org.eclipse.jgit%22)获取最新版本的 JGit。一旦这一步完成，Maven 将自动获取并使用我们需要的 JGit 库。

如果您喜欢 OSGi 包，也有一个 p2 资源库。请访问 [Eclipse JGit](https://web.archive.org/web/20221129011837/https://www.eclipse.org/jgit/download/) 以获得如何集成这个库的必要信息。

## 3。创建存储库

JGit 有两个基本级别的 API: `plumbing`和`porcelain`。这些术语来自 Git 本身。JGit 分为相同的区域:

*   `porcelain`API——常见用户级操作的前端(类似于 Git 命令行工具)
*   APIs 直接与底层存储库对象交互

大多数 JGit 会话的起点都在`Repository`类中。我们要做的第一件事是创建一个新的`Repository`实例。

`init` 命令将让我们创建一个空的存储库:

```java
Git git = Git.init().setDirectory("/path/to/repo").call();
```

这将在指定给`setDirectory()`的位置创建一个包含工作目录的存储库。

可以使用`cloneRepository`命令克隆一个现有的存储库:

```java
Git git = Git.cloneRepository()
  .setURI("https://github.com/eclipse/jgit.git")
  .setDirectory("/path/to/repo")
  .call();
```

上面的代码将把 JGit 存储库克隆到名为`path/to/repo`的本地目录中。

## 4。Git 对象

在 Git 对象模型中，所有对象都由一个 SHA-1 id 表示。在 JGit 中，这由`AnyObjectId`和`ObjectId`类表示。

Git 对象模型中有四种类型的对象:

*   `blob`–用于存储文件数据
*   `tree`–一个目录；它引用了其他的`trees`和`blobs`
*   `commit`–指向一棵树
*   `tag`–将提交标记为特殊；通常用于标记特定版本

要从存储库中解析对象，只需传递正确的修订版，如以下函数所示:

```java
ObjectId head = repository.resolve("HEAD");
```

### 4.1。`Ref`

`Ref`是一个保存单个对象标识符的变量。对象标识符可以是任何有效的 Git 对象(`blob`、`tree`、`commit`、`tag`)。

例如，要查询对 head 的引用，只需调用:

```java
Ref HEAD = repository.getRef("refs/heads/master");
```

### 4.2。`RevWalk`

`RevWalk`遍历提交图并按顺序产生匹配的提交:

```java
RevWalk walk = new RevWalk(repository);
```

### 4.3。`RevCommit`

`RevCommit`表示 Git 对象模型中的提交。要解析提交，使用一个`RevWalk`实例:

```java
RevWalk walk = new RevWalk(repository);
RevCommit commit = walk.parseCommit(objectIdOfCommit);
```

### 4.4。`RevTag`

`RevTag`代表 Git 对象模型中的一个标签。您可以使用一个`RevWalk`实例来解析标签:

```java
RevWalk walk = new RevWalk(repository);
RevTag tag = walk.parseTag(objectIdOfTag);
```

### 4.5。`RevTree`

`RevTree`表示 Git 对象模型中的一棵树。一个`RevWalk`实例也用于解析一棵树:

```java
RevWalk walk = new RevWalk(repository);
RevTree tree = walk.parseTree(objectIdOfTree);
```

## 5。瓷器 API

虽然 JGit 包含了许多低级代码来处理 Git 存储库，但它也包含了一个高级 API，模拟了`org.eclipse.jgit.api`包中的一些 Git `porcelain`命令。

### 5.1。`AddCommand` ( `git-add` )

`AddCommand`允许您通过以下方式将文件添加到索引中:

*   `addFilepattern`()

下面是一个如何使用`porcelain` API 将一组文件添加到索引中的快速示例:

```java
Git git = new Git(db);
AddCommand add = git.add();
add.addFilepattern("someDirectory").call();
```

### 5.2。`CommitCommand` ( `git-commit` )

`CommitCommand`允许您执行提交，并提供以下选项:

*   `setAuthor`()
*   `setCommitter`()
*   `setAll`()

这里有一个如何使用`porcelain` API 提交的快速示例:

```java
Git git = new Git(db);
CommitCommand commit = git.commit();
commit.setMessage("initial commit").call();
```

### 5.3。`TagCommand` ( `git-tag` )

`TagCommand`支持多种标记选项:

*   `setName`()
*   `setMessage`()
*   `setTagger`()
*   `setObjectId`()
*   `setForceUpdate`()
*   `setSigned`()

这里有一个使用`porcelain` API 标记提交的简单例子:

```java
Git git = new Git(db);
RevCommit commit = git.commit().setMessage("initial commit").call();
RevTag tag = git.tag().setName("tag").call();
```

### 5.4。`LogCommand` ( `git-log` )

`LogCommand`允许您轻松地遍历提交图。

*   `add(AnyObjectId start)`
*   `addRange(AnyObjectId since, AnyObjectId until)`

下面是如何获取一些日志消息的快速示例:

```java
Git git = new Git(db);
Iterable<RevCommit> log = git.log().call();
```

## 6。蚂蚁任务

JGit 也有一些常见的 Ant 任务包含在`org.eclipse.jgit.ant`包中。

要使用这些任务:

```java
<taskdef resource="org/eclipse/jgit/ant/ant-tasks.properties">
    <classpath>
        <pathelement location="path/to/org.eclipse.jgit.ant-VERSION.jar"/>
        <pathelement location="path/to/org.eclipse.jgit-VERSION.jar"/>
        <pathelement location="path/to/jsch-0.1.44-1.jar"/>
    </classpath>
</taskdef>
```

这将提供 `git-clone, git-init` 和 `git-checkout`任务。

### 6.1。`git-clone`

```java
<git-clone uri="http://egit.eclipse.org/jgit.git" />
```

下列属性是必需的:

*   `uri`:要克隆的 URI

以下属性是可选的:

*   `dest`:要克隆到的目的地(默认使用基于`URI`的最后一个路径组件的人类可读目录名)
*   `bare` : `true` / `false` / `yes` / `no`表示克隆的仓库是否应该是空的(默认为`false`
*   `branch`:克隆存储库时要检出的初始分支(默认为`HEAD`)

### 6.2。`git-init`

```java
<git-init />
```

运行`git-init`任务不需要任何属性。

以下属性是可选的:

*   `dest`:初始化 git 仓库的路径(默认为`$GIT_DIR`或当前目录)
*   `bare` : `true` / `false` / `yes` / `no`表示仓库是否应该是空的(默认为`false`

### 6.3。`git-checkout`

```java
<git-checkout src="path/to/repo" branch="origin/newbranch" />
```

下列属性是必需的:

*   `src`:git 存储库的路径
*   `branch`:要结账的初始分支

以下属性是可选的:

*   `createbranch` : `true` / `false` / `yes` / `no`表示如果分支不存在，是否需要创建(默认为`false`
*   `force` : `true` / `false` / `yes` / `no`:如果`true` / `yes`和给定名称的分支已经存在，现有分支的起点将被设置为新的起点；如果`false`，现有分支不会改变(默认为`false`)

## 7。结论

高级的 JGit API 不难理解。如果您知道使用什么 git 命令，您可以很容易地猜到在 JGit 中使用哪些类和方法。

这里有一个现成的 JGit 代码片段集合。

如果您仍然有困难或问题，请在这里留下评论或向 [JGit 社区](https://web.archive.org/web/20221129011837/https://www.eclipse.org/jgit/support/)寻求帮助。