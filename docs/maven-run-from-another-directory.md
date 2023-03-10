# 从另一个目录运行 mvn 命令

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-run-from-another-directory>

## 1.概观

在这个快速教程中，我们将看到如何从`pom.xml`之外的任何目录运行`[mvn](/web/20221221114239/https://www.baeldung.com/maven) `命令。

## 2.从另一个目录

如果我们从不包含`pom.xml `文件的目录中运行任何`mvn `子命令，该命令将失败:

```java
$ mvn clean compile
The goal you specified requires a project to execute but there is no POM in this directory.
Please verify you invoked Maven from the correct directory
```

如上图所示，Maven 抱怨当前目录中缺少一个`pom.xml `文件。

**要解决这个问题并从另一个目录中调用一个专家[阶段或目标](/web/20221221114239/https://www.baeldung.com/maven-goals-phases)，我们可以使用`-f `或`–file option`T5:**

```java
$ mvn -f tutorials/ clean compile
```

因为在指定的目录中有一个`pom.xml `文件，所以这个命令将实际编译代码。

基本上，这个选项强制使用带有`pom.xml`的备用 POM 文件或目录。所以我们也可以使用完整的文件路径:

```java
$ mvn -f tutorials/pom.xml clean compile
```

## 3.结论

在这个简短的教程中，我们看到了如何从另一个目录运行`mvn `命令。