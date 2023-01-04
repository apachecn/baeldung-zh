# 在 Java 中从两个绝对路径构造一个相对路径

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-relative-path-absolute>

## 1.概观

在本教程中，我们将学习如何在 Java 中从两个绝对路径构造一个相对路径。我们将关注两个内置的 Java APIs 新的 I/O (NIO2)路径 API 和 URI 类。

## 2.绝对路径与相对路径

在我们开始之前，让我们快速回顾一下。对于文本中的所有示例，我们将在用户的主目录中使用相同的文件结构:

```
/ (root)
|-- baeldung
    \-- bar
    |   |-- one.txt
    |   |-- two.txt
    \-- foo
        |-- three.txt
```

绝对路径描述了一个位置，从根节点开始，不考虑当前的工作目录。以下是我们文件的绝对路径:

```
one.txt -> /baeldung/bar/one.txt
two.txt -> /baeldung/bar/two.txt
three.txt -> /baeldung/foo/three.txt
```

即使我们改变了工作目录，绝对路径也总是保持不变。

另一方面，**相对路径描述了目标节点相对于其源节点**的位置。如果我们在`baeldung`目录中，让我们看看文件的相对路径:

```
one.txt -> ./bar/one.txt
two.txt -> ./bar/two.txt
three.txt -> ./foo/three.txt
```

现在，让我们移到`bar`子目录，再次检查相对路径:

```
one.txt -> ./one.txt
two.txt -> ./two.txt
three.txt -> ../foo/three.txt
```

正如我们所见，结果略有不同。我们必须记住，如果我们修改源上下文，**相对值会改变，而绝对路径是不变的**。绝对路径是相对路径的特例，其中源节点是系统的根。

## 3.NIO2 API

现在我们知道了相对路径和绝对路径是如何工作的，是时候[看看 NIO2 API](/web/20221109203447/https://www.baeldung.com/java-nio-2-file-api) 了。众所周知，**nio 2 API 是随着 Java 7 的发布而引入的，它改进了旧的 I/O API，后者有许多缺陷**。使用这个 API，我们将尝试确定由绝对路径描述的两个文件之间的相对路径。

让我们从为我们的文件构造`Path`对象开始:

```
Path pathOne = Paths.get("/baeldung/bar/one.txt");
Path pathTwo = Paths.get("/baeldung/bar/two.txt");
Path pathThree = Paths.get("/baeldung/foo/three.txt");
```

要构建源节点和给定节点之间的相对路径，我们可以使用由`Path`类提供的 [`relativize(Path)`](https://web.archive.org/web/20221109203447/https://docs.oracle.com/javase/8/docs/api/java/nio/file/Path.html#relativize-java.nio.file.Path-) 方法:

```
Path result = pathOne.relativize(pathTwo);

assertThat(result)
  .isRelative()
  .isEqualTo(Paths.get("../two.txt"));
```

如我们所见，结果肯定是相对路径。对吗？尤其是开头带父运算符(`../`)？

我们必须记住，可以从任何类型的节点开始指定相对路径，可以是目录或文件。特别是当我们使用 CLI 或浏览器时，我们使用目录。然后基于当前工作目录计算所有相对路径。

在我们的例子中，我们创建了指向特定文件的`Path`。因此，我们首先需要找到文件的父文件及其目录，然后再找到第二个文件。总的来说，结果是正确的。

如果我们想使结果相对于源目录，我们可以使用 [`getParent()`](https://web.archive.org/web/20221109203447/https://docs.oracle.com/javase/8/docs/api/java/nio/file/Path.html#getParent--) 的方法:

```
Path result = pathOne.getParent().relativize(pathTwo);

assertThat(result)
  .isRelative()
  .isEqualTo(Paths.get("two.txt"));
```

我们应该注意到,`Path`对象可能指向任何文件或目录。如果我们正在构建一个更复杂的逻辑，我们需要提供额外的检查。

最后，让我们检查一下`one.txt`和`three.txt`文件之间的相对路径:

```
Path resultOneToThree = pathOne.relativize(pathThree);
Path resultThreeToOne = pathThree.relativize(pathOne);

assertThat(resultOneToThree)
  .isRelative()
  .isEqualTo(Paths.get("..\..\foo\three.txt"));
assertThat(result)
  .isRelative()
  .isEqualTo(Paths.get("..\..\bar\one.txt")); 
```

这个快速测试证实了**相对路径是依赖于上下文的**。虽然绝对路径仍然相同，但是当我们一起交换源节点和目标节点时，相对路径将会不同。

## 4.  `java.net.URI` API

检查完 NIO2 API 后，让我们进入`java.net.URI`类。我们知道**(统一资源标识符)是一个字符串，它允许我们标识任何资源**，这些资源也可以在处理文件时使用。

让我们为我们的文件构造`URI`对象:

```
URI uriOne = pathOne.toURI();
// URI uriOne = URI.create("file:///baeldung/bar/one.txt")
URI uriTwo = pathTwo.toURI();
URI uriThree = pathThree.toURI();
```

我们既可以使用`String`构建一个`URI`对象，也可以转换之前创建的`Path`。

和以前一样，`URI`类也提供了一个 [`relativize(URI)`](https://web.archive.org/web/20221109203447/https://docs.oracle.com/javase/8/docs/api/java/net/URI.html#relativize-java.net.URI-) 方法。让我们用它来构造相对路径:

```
URI result = uriOne.relativize(uriTwo);

assertThat(result)
  .asString()
  .contains("file:///baeldung/bar/two.txt");
```

结果不是我们所期望的，相对路径没有被正确构造。要回答为什么会这样的问题，我们需要查看类的官方文档。

**如果源 URI 是目标 URI 的前缀，此方法仅返回相对值。**否则，返回目标值。因此，我们无法在文件节点之间建立相对路径。在这种情况下，一个 URI 永远不会作为另一个的前缀。

为了返回一个相对路径，我们可以将我们的 source `URI`设置为第一个文件的目录:

```
URI uriOneParent = pathOne.getParent().toUri(); // file:///baeldung/bar/
URI result = uriOneParent.relativize(uriTwo);

assertThat(result)
  .asString()
  .contains("two.txt");
```

现在源节点是目标前缀，所以结果计算正确。由于方法的限制，我们无法使用`URI`方法确定`one.txt/two.txt`和`three.txt`文件之间的相对路径。它们的目录不会有共同的前缀。

## 5.摘要

在本文中，我们从查看绝对路径和相对路径之间的主要区别开始。

接下来，我们在由绝对路径描述的两个文件之间构建了一个相对路径。我们从检查 NIO2 API 开始，详细描述了相关的路径构建过程。

最后，我们试图用`java.net.URI`类达到同样的结果。我们发现由于它的限制，我们不能使用这个 API 完成所有的转换。

和往常一样，所有附加测试的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20221109203447/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-apis-2)