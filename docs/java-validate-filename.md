# 在 Java 中验证字符串作为文件名

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-validate-filename>

## 1.概观

在本教程中，我们将讨论使用 Java 来验证给定的 `String`对于操作系统是否有有效的文件名的不同方法。我们希望对照受限字符或长度限制来检查该值。

通过示例，我们将只关注核心解决方案，而不使用任何外部依赖。我们将检查 SDK 的`java.io`和 NIO2 包，并最终实现我们自己的解决方案。

## 2.使用`java.io.File`

让我们从第一个例子开始，使用 [`java.io.File`类](/web/20220926192736/https://www.baeldung.com/java-io-file)。在这个解决方案中，我们需要用给定的字符串创建一个`File`实例，然后在本地磁盘上创建一个文件:

```java
public static boolean validateStringFilenameUsingIO(String filename) throws IOException {
    File file = new File(filename);
    boolean created = false;
    try {
        created = file.createNewFile();
        return created;
    } finally {
        if (created) {
            file.delete();
        }
    }
}
```

当给定的文件名不正确时，它**抛出一个`IOException`。**我们注意一下，由于文件创建在里面，这个方法要求给定的`filename` `String`不对应已经存在的文件。

我们知道不同的文件系统有各自的 [**文件名限制**](https://web.archive.org/web/20220926192736/https://en.wikipedia.org/wiki/Filename) 。因此，通过使用`java.io.File`方法，**我们不需要为每个操作系统指定规则**，因为 Java 会自动为我们处理它。

然而，我们需要创建一个虚拟文件。当我们成功的时候，我们必须**记得在最后删除它**。此外，我们必须确保我们有适当的权限来执行这些操作。任何失败也可能导致`IOException`，因此最好检查错误消息:

```java
assertThatThrownBy(() -> validateStringFilenameUsingIO("baeldung?.txt"))
  .isInstanceOf(IOException.class)
  .hasMessageContaining("Invalid file path");
```

## 3.使用 NIO2 API

我们知道`java.io`包[有很多缺点](/web/20220926192736/https://www.baeldung.com/java-path-vs-file)，因为它是在 Java 的第一个版本中创建的。作为`java.io` 包的继任者，NIO2 API 带来了许多改进，这也极大地简化了我们之前的解决方案:

```java
public static boolean validateStringFilenameUsingNIO2(String filename) {
    Paths.get(filename);
    return true;
}
```

我们的功能现在是流线型的，所以这是执行这种测试最快的方法。我们不创建任何文件，所以我们**不需要有任何磁盘权限，并在测试后执行清理**。

无效的文件名**抛出扩展了`RuntimeException`的** `**InvalidPathException**,` 。**错误信息还包含比之前更多的细节**:

```java
assertThatThrownBy(() -> validateStringFilenameUsingNIO2(filename))
  .isInstanceOf(InvalidPathException.class)
  .hasMessageContaining("character not allowed");
```

这个解决方案有一个与文件系统限制相关的**严重缺陷。`Path`类可能代表带有子目录的文件路径。与第一个例子不同，这个方法不检查文件名字符的溢出限制。让我们对照使用 Apache Commons 的 [`randomAlphabetic()`](https://web.archive.org/web/20220926192736/https://commons.apache.org/proper/commons-lang/javadocs/api-3.9/org/apache/commons/lang3/RandomStringUtils.html#randomAlphabetic-int-) 方法生成的 500 个字符的随机`String`进行检查:**

```java
String filename = RandomStringUtils.randomAlphabetic(500);
assertThatThrownBy(() -> validateStringFilenameUsingIO(filename))
  .isInstanceOf(IOException.class)
  .hasMessageContaining("File name too long");

assertThat(validateStringFilenameUsingNIO2(filename)).isTrue();
```

要解决这个问题，我们应该像以前一样，创建一个文件并检查结果。

## 4。定制实现

最后，让我们尝试实现我们自己的自定义函数来测试文件名。我们还将尽量避免任何 I/O 功能，只使用核心 Java 方法。

这类解决方案提供了更多的控制，并允许我们**实施我们自己的规则**。然而，我们**必须考虑不同系统的许多附加限制**。

### 4.1.使用`String.contains`

我们可以**使用 [`String.contains()`](https://web.archive.org/web/20220926192736/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#contains(java.lang.CharSequence)) 方法**来检查给定的`String`是否包含任何被禁止的字符。首先，我们需要手动指定一些示例值:

```java
public static final Character[] INVALID_WINDOWS_SPECIFIC_CHARS = {'"', '*', '<', '>', '?', '|'};
public static final Character[] INVALID_UNIX_SPECIFIC_CHARS = {'\000'};
```

在我们的例子中，让我们只关注这两个操作系统。众所周知，Windows 的文件名比 UNIX 的更加严格。另外，一些**空白字符可能会有问题**。

定义了受限字符集后，让我们来确定当前的操作系统:

```java
public static Character[] getInvalidCharsByOS() {
    String os = System.getProperty("os.name").toLowerCase();
    if (os.contains("win")) {
        return INVALID_WINDOWS_SPECIFIC_CHARS;
    } else if (os.contains("nix") || os.contains("nux") || os.contains("mac")) {
        return INVALID_UNIX_SPECIFIC_CHARS;
    } else {
        return new Character[]{};
    }
}
```

现在我们可以用它来测试给定值:

```java
public static boolean validateStringFilenameUsingContains(String filename) {
    if (filename == null || filename.isEmpty() || filename.length() > 255) {
        return false;
    }
    return Arrays.stream(getInvalidCharsByOS())
      .noneMatch(ch -> filename.contains(ch.toString()));
}
```

如果我们定义的任何字符不在给定的文件名中，这个`Stream`谓词返回 true。此外，我们实现了对`null`值和不正确长度的支持。

### 4.2.正则表达式模式匹配

我们也可以**在给定的`String`上直接使用[正则表达式](/web/20220926192736/https://www.baeldung.com/regular-expressions-java)** 。让我们实现一个只接受字母数字和点字符的模式，长度不超过 255:

```java
public static final String REGEX_PATTERN = "^[A-za-z0-9.]{1,255}$";

public static boolean validateStringFilenameUsingRegex(String filename) {
    if (filename == null) {
        return false;
    }
    return filename.matches(REGEX_PATTERN);
} 
```

现在，我们可以根据之前准备好的模式测试给定值。我们也可以很容易地修改模式。在本例中，我们跳过了操作系统检查功能。

## 5.结论

在本文中，我们主要关注文件名及其局限性。我们使用 Java 引入了不同的算法来检测无效的文件名。

我们从`java.io`包开始，它**为我们处理大部分系统限制，但是执行额外的 I/O 操作**，并且可能需要一些权限。然后我们检查了 NIO2 API，它是最快的解决方案，有文件名长度检查限制。

最后，我们实现了自己的方法**，没有使用任何 I/O API，但是需要文件系统规则**的自定义实现。

你可以在 GitHub 上找到所有附加测试[的例子。](https://web.archive.org/web/20220926192736/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-3)