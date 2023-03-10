# JUnit 5 临时目录支持

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-5-temporary-directory>

## 1。概述

测试时，我们经常需要访问一个临时文件。然而，自己管理这些文件的创建和删除可能会很麻烦。

在这个快速教程中，**我们将看看 JUnit 5 如何通过提供 TempDirectory 扩展**来缓解这个问题。

要深入了解 JUnit 测试指南，请查看我们优秀的 JUnit 5 指南[。](/web/20221224002422/https://www.baeldung.com/junit-5)

## 2。临时目录扩展

从版本 5.4.2 开始，JUnit 5 提供了 TempDirectory 扩展。然而，重要的是要注意，这仍然是一个官方的[实验性的](https://web.archive.org/web/20221224002422/https://apiguardian-team.github.io/apiguardian/docs/1.0.0/api/org/apiguardian/api/API.Status.html?is-external=true#EXPERIMENTAL)特性，我们鼓励向 JUnit 团队提供反馈。

正如我们将在后面看到的，**我们可以使用这个扩展来为一个测试类**中的单个测试或者所有测试创建和清理一个临时目录。

通常当[使用扩展](/web/20221224002422/https://www.baeldung.com/junit-5-extensions)时，我们需要使用`@ExtendWith`注释在 JUnit 5 测试中注册它。但是对于默认情况下内置和注册的 TempDirectory 扩展，这是不必要的。

## 3。Maven 依赖关系

首先，让我们添加例子中需要的项目依赖项。

除了主要的 JUnit 5 库`junit-jupiter-engine`之外，我们还需要`junit-jupiter-api`库:

```java
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.8.1</version>
    <scope>test</scope>
</dependency>
```

一如既往，我们可以从 [Maven Central](https://web.archive.org/web/20221224002422/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22junit-jupiter-api%22) 获得最新版本。

除此之外，我们还需要添加`junit-jupiter-params`依赖项:

```java
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-params</artifactId>
    <version>5.8.1</version>
    <scope>test</scope>
</dependency>
```

同样，我们可以在 [Maven Central](https://web.archive.org/web/20221224002422/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22junit-jupiter-params%22) 中找到最新版本。

## 4。使用`@TempDir`标注

为了使用 TempDirectory 扩展，**我们需要使用`@TempDir`注释**。我们只能对以下两种类型使用此注释:

*   `java.nio.file.Path`
*   `java.io.File`

事实上，如果我们试图用不同的类型来使用它，那么就会抛出一个`org.junit.jupiter.api.extension.ParameterResolutionException`。

接下来，让我们探索使用该注释的几种不同方式。

### 4.1。`@TempDir`作为方法参数

让我们先来看看如何**将一个用`@TempDir`标注的参数注入到一个测试方法**中:

```java
@Test
void givenTestMethodWithTempDirectory_whenWriteToFile_thenContentIsCorrect(@TempDir Path tempDir) 
  throws IOException {
    Path numbers = tempDir.resolve("numbers.txt");

    List<String> lines = Arrays.asList("1", "2", "3");
    Files.write(numbers, lines);

    assertAll(
      () -> assertTrue("File should exist", Files.exists(numbers)),
      () -> assertLinesMatch(lines, Files.readAllLines(numbers)));
}
```

正如我们所看到的，我们的测试方法在临时目录`tempDir`中创建和写入了一个名为`numbers.txt`的文件。

然后，我们检查文件是否存在，内容是否与最初编写的内容相匹配。真的好简单！

### 4.2。`@TempDir`上一个实例字段

在下一个例子中，我们将使用`@TempDir`注释来注释测试类中的一个字段:

```java
@TempDir
File anotherTempDir;

@Test
void givenFieldWithTempDirectoryFile_whenWriteToFile_thenContentIsCorrect() throws IOException {
    assertTrue("Should be a directory ", this.anotherTempDir.isDirectory());

    File letters = new File(anotherTempDir, "letters.txt");
    List<String> lines = Arrays.asList("x", "y", "z");

    Files.write(letters.toPath(), lines);

    assertAll(
      () -> assertTrue("File should exist", Files.exists(letters.toPath())),
      () -> assertLinesMatch(lines, Files.readAllLines(letters.toPath())));
}
```

这一次，我们使用一个`java.io.File`作为临时目录。同样，我们写一些行并检查它们是否写成功。

如果我们在其他测试方法中再次使用这个引用，每个测试将使用它自己的临时目录。

### 4.3。共享临时目录

有时，我们可能想要在测试方法之间共享一个临时目录。

我们可以通过声明我们的字段`static`来做到这一点:

```java
@TempDir
static Path sharedTempDir;

@Test
@Order(1)
void givenFieldWithSharedTempDirectoryPath_whenWriteToFile_thenContentIsCorrect() throws IOException {
    Path numbers = sharedTempDir.resolve("numbers.txt");

    List<String> lines = Arrays.asList("1", "2", "3");
    Files.write(numbers, lines);

    assertAll(
        () -> assertTrue("File should exist", Files.exists(numbers)),
        () -> assertLinesMatch(lines, Files.readAllLines(numbers)));
}

@Test
@Order(2)
void givenAlreadyWrittenToSharedFile_whenCheckContents_thenContentIsCorrect() throws IOException {
    Path numbers = sharedTempDir.resolve("numbers.txt");

    assertLinesMatch(Arrays.asList("1", "2", "3"), Files.readAllLines(numbers));
  } 
```

**这里的关键点是我们使用了一个静态字段`sharedTempDir`，它是我们在两个测试方法**之间共享的。

在第一个测试中，我们再次向名为`numbers.txt`的文件中写入一些行。然后我们在下一个测试中检查文件和内容是否已经存在。

我们还通过`@Order`注释强制[测试](/web/20221224002422/https://www.baeldung.com/junit-5-test-order)的顺序，以确保行为总是一致的。

## 5。抓到你了

现在让我们回顾一下在使用 TempDirectory 扩展时应该注意的一些微妙之处。

### 5.1。创作

好奇的读者很可能想知道这些临时文件实际上是在哪里创建的？

嗯，在内部，JUnit `TemporaryDirectory`类使用了`Files.createTempDirectory(String prefix)`方法。**同样，这个方法使用默认的系统临时文件目录**。

这通常在环境变量`TMPDIR`中指定:

```java
TMPDIR=/var/folders/3b/rp7016xn6fz9g0yf5_nj71m00000gn/T/ 
```

例如，产生一个临时文件位置:

```java
/var/folders/3b/rp7016xn6fz9g0yf5_nj71m00000gn/T/junit5416670701666180307/numbers.txt
```

同时，如果不能创建临时目录，将会适当地抛出一个`ExtensionConfigurationException`。或者如前所述，一个`ParameterResolutionException`。

### 5.2。删除

当测试方法或类已经完成执行并且临时目录超出范围时，JUnit 框架将尝试递归地删除该目录中的所有文件和目录，最后是临时目录本身。

如果在这个删除阶段出现问题，将抛出一个`IOException`，测试或测试类将失败。

## 6。结论

总之，在本教程中，我们已经探索了 JUnit 5 提供的 TempDirectory 扩展。

首先，我们从介绍扩展开始，了解了使用它需要哪些 Maven 依赖项。接下来，我们看了几个如何在我们的单元测试中使用扩展的例子。

最后，我们看了几个问题，包括临时文件的创建位置和删除过程中发生的事情。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221224002422/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5-basics)