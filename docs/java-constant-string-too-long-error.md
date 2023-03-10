# 修复“常量字符串太长”构建错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-constant-string-too-long-error>

## 1。概述

当我们试图使用一个对 Java 编译器来说太长的变量(大于 64 KB)时，我们会从编译器收到一个**“常量字符串太长”的错误。**

在本教程中，我们将展示如何解决这一错误。

## 2。描述问题

让我们通过编写一个小测试来重现这个问题，其中我们声明了一个太长的`String`:

```java
@Test
public void whenDeclaringTooLongString_thenCompilationError() {
    String stringTooLong = "stringstringstring ... 100,000 characters ... string";  
    assertThat(stringTooLong).isNotEmpty();
}
```

包含在我们的`stringTooLong`变量中的`String`包含超过 100，000 个字符的文本。通过最后的 GitHub 链接，可以在一个文件中找到具有这些特征的字符串。要引发错误，复制其内容并替换`stringTooLong`的值。

注意，**如果我们从一些 ide 运行这个测试，我们不会收到任何错误**。

原因是 ide 通常更宽松。然而，当试图从命令行编译项目(`mvn package`)或者只是试图执行测试(`mvn test`)时，我们将收到以下输出:

```java
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  5.058 s
[INFO] Finished at: 2020-03-14T17:56:34+01:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.7.0:testCompile (default-testCompile) on project core-java-strings: Compilation failure
[ERROR] <path to the test class>:[10,32] constant string too long
```

这是因为在 UTF-8 编码中，类文件中字符串常量的长度限制为 2^16 字节。

## 3。解决问题

一旦我们重现了问题，让我们想办法解决它。最好的方法是**将我们的字符串存储在一个单独的文件**中，而不是存储在一个声明的变量或常量中。

让我们创建一个文本文件来存储变量的内容，并将我们的测试修改为[从文件](/web/20221208143837/https://www.baeldung.com/reading-file-in-java)中获取值:

```java
@Test
public void whenStoringInFileTooLongString_thenNoCompilationError() throws IOException {
    FileInputStream fis = new FileInputStream("src/test/resources/stringtoolong.txt");
    String stringTooLong = IOUtils.toString(fis, "UTF-8");
    assertThat(stringTooLong).isNotEmpty();
}
```

解决这个问题的另一种方法是将变量的内容存储在一个属性文件中，然后从我们的测试方法中访问它:

```java
@Test
public void whenStoringInPropertiesString_thenNoCompilationError() throws IOException {
    try (InputStream input = new FileInputStream("src/main/resources/config.properties")) {         
        Properties prop = new Properties();
        prop.load(input);
        String sValue = prop.getProperty("stringtoolong");
        assertThat(sValue).isNotEmpty();
    }  
}
```

现在，如果我们试图编译我们的项目或执行测试，一切都将正常工作:

```java
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  6.433 s
[INFO] Finished at: 2020-03-14T18:23:54+01:00
[INFO] ------------------------------------------------------------------------
```

当然，我们也可以在字符串中引入串联，但是不推荐这样做。如果我们有这么长的字符串，我们的 Java 文件可能不是它的最佳归宿。

## 4。结论

在本文中，我们研究了“常量字符串太长”编译错误。我们发现可以通过将字符串的值存储在单独的文件或配置属性中来解决这个问题。

和往常一样，你可以在 GitHub 上找到代码[。](https://web.archive.org/web/20221208143837/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-strings)