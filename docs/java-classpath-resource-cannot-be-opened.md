# 加载资源时如何避免 Java FileNotFoundException

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-classpath-resource-cannot-be-opened>

## 1.概观

在本教程中，我们将探讨在 Java 应用程序中读取资源文件时可能出现的一个问题:在运行时，资源文件夹在磁盘上的位置很少与在源代码中的位置相同。

让我们看看在我们的代码被打包后，Java 是如何允许我们访问资源文件的。

## 2.读取文件

假设我们的应用程序在启动时读取一个文件:

```java
try (FileReader fileReader = new FileReader("src/main/resources/input.txt"); 
     BufferedReader reader = new BufferedReader(fileReader)) {
    String contents = reader.lines()
      .collect(Collectors.joining(System.lineSeparator()));
}
```

如果我们在 IDE 中运行上面的代码，文件加载时不会出错。这是因为**我们的 IDE 使用我们的项目目录作为它当前的工作目录**，而`src/main/resources`目录就在那里供应用程序读取。

现在假设我们使用 [Maven JAR 插件](/web/20221013193922/https://www.baeldung.com/executable-jar-with-maven)将我们的代码打包成一个 JAR。

当我们在命令行运行它时:

```java
java -jar core-java-io2.jar
```

我们将看到以下错误:

```java
Exception in thread "main" java.io.FileNotFoundException: 
    src/main/resources/input.txt (No such file or directory)
	at java.io.FileInputStream.open0(Native Method)
	at java.io.FileInputStream.open(FileInputStream.java:195)
	at java.io.FileInputStream.<init>(FileInputStream.java:138)
	at java.io.FileInputStream.<init>(FileInputStream.java:93)
	at java.io.FileReader.<init>(FileReader.java:58)
	at com.baeldung.resource.MyResourceLoader.loadResourceWithReader(MyResourceLoader.java:14)
	at com.baeldung.resource.MyResourceLoader.main(MyResourceLoader.java:37)
```

## 3.源代码与编译代码

当我们构建一个 JAR 时，资源被放入打包工件的根目录中。

在我们的例子中，我们看到源代码目录的`src/main/resources` 中有`input.txt`。

然而，在相应的 JAR 结构中，我们看到:

```java
META-INF/MANIFEST.MF
META-INF/
com/
com/baeldung/
com/baeldung/resource/
META-INF/maven/
META-INF/maven/com.baeldung/
META-INF/maven/com.baeldung/core-java-io-files/
input.txt
com/baeldung/resource/MyResourceLoader.class
META-INF/maven/com.baeldung/core-java-io-files/pom.xml
META-INF/maven/com.baeldung/core-java-io-files/pom.properties
```

这里，`input.txt`位于 JAR 的根目录。所以当代码执行时，我们会看到`FileNotFoundException`。

即使我们将路径更改为`/input.txt`，原始代码也无法加载该文件，因为**资源通常不能作为文件在磁盘上寻址。**资源文件打包在 JAR 中，所以我们需要一种不同的方式来访问它们。

## 4.资源

让我们使用资源加载来从类路径而不是特定的文件位置加载资源。不管代码是如何打包的，这都是可行的:

```java
try (InputStream inputStream = getClass().getResourceAsStream("/input.txt");
    BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream))) {
    String contents = reader.lines()
      .collect(Collectors.joining(System.lineSeparator()));
}
```

`ClassLoader.getResourceAsStream()`查看给定资源的类路径。`getResourceAsStream()`输入上的前导斜杠告诉加载器从类路径的基底读取。**我们的 JAR 文件的内容在类路径**上，所以这个方法是有效的。

**IDE 通常在它的类路径**中包含`src/main/resources`，从而找到文件。

## 5.结论

在这篇简短的文章中，我们实现了将文件作为类路径资源加载，以允许我们的代码一致地工作，而不管它是如何打包的。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20221013193922/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io)