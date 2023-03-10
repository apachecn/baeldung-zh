# 自动执行 JAR 中主清单属性的重要性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jar-executable-manifest-main-class>

## 1。概述

每个可执行的 Java 类都必须包含一个`main`方法。简单地说，这个方法是一个应用程序的起点。

要从自执行 JAR 文件运行我们的 main 方法，我们必须创建一个适当的清单文件，并将其与我们的代码打包在一起。这个 manifest 文件必须有一个 main manifest 属性，它定义了包含我们的 main 方法的类的路径。

在本教程中，我们将展示如何**将一个简单的 Java 类打包成一个自执行的 JAR，并展示主清单属性**对于成功执行的重要性。

## 2。执行没有主清单属性的 JAR

为了更加实际，我们将展示一个没有适当的 manifest 属性的不成功执行的例子。

让我们用 main 方法编写一个简单的 Java 类:

```java
public class AppExample {
    public static void main(String[] args){
        System.out.println("AppExample executed!");
    }
} 
```

要将我们的示例类打包到 JAR 归档文件中，我们必须进入操作系统的外壳并编译它:

```java
javac -d . AppExample.java 
```

然后我们可以把它装进一个罐子里:

```java
jar cvf example.jar com/baeldung/manifest/AppExample.class 
```

我们的`example.jar`将包含一个默认的清单文件。我们现在可以尝试执行 JAR:

```java
java -jar example.jar 
```

执行将失败，并出现错误:

```java
no main manifest attribute, in example.jar 
```

## 3。使用主清单属性执行 JAR

正如我们所看到的，JVM 找不到我们的主 manifest 属性。因此，它找不到包含我们的主方法的主类。

让我们在代码中包含一个适当的清单属性。我们需要创建一个包含单行的`MANIFEST.MF`文件:

```java
Main-Class: com.baeldung.manifest.AppExample 
```

我们的清单现在包含了编译后的`AppExample.class`的类路径。因为我们已经编译了我们的示例类，所以没有必要再做一遍。

我们将把它和我们的清单文件打包在一起:

```java
jar cvmf MANIFEST.MF example.jar com/baeldung/manifest/AppExample.class 
```

这一次 JAR 按预期执行并输出:

```java
AppExample executed!
```

## 4。结论

在这篇简短的文章中，我们展示了如何将一个简单的 Java 类打包成一个自执行的 JAR，并通过两个简单的例子展示了 main manifest 属性的重要性。

该示例的完整源代码可以在 GitHub 上的[处获得。这是一个基于 Maven 的项目，因此可以导入并按原样使用。](https://web.archive.org/web/20220525202517/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jar)