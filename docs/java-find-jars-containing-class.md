# 查找包含给定类的所有 jar

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-find-jars-containing-class>

## 1.介绍

在本文中，我们将学习查找包含特定类的所有 jar。我们将使用两种不同的方法来演示这一点，即基于命令和基于程序。

## 2。基于命令的

在这种方法中，我们将使用 shell 命令来识别本地 maven 存储库中具有`ObjectMapper`类的所有 jar。让我们从编写一个脚本来标识 jar 中的类开始。该脚本使用 [`jar`](https://web.archive.org/web/20221208143856/https://docs.oracle.com/en/java/javase/11/tools/jar.html) 和 [`grep`](https://web.archive.org/web/20221208143856/https://man7.org/linux/man-pages/man1/grep.1.html) 命令打印合适的 jar:

```java
jar -tf $1 | grep $2 && echo "Found in : $1"
```

这里$1 是 jar 文件路径，$2 是类名。在这种情况下，类名将始终是`com.fasterxml.jackson.databind.ObjectMapper` 。让我们将上面的命令保存在一个 bash 文件`findJar.sh`中。之后，我们将在本地 maven 存储库上运行下面的 **[`find`](/web/20221208143856/https://www.baeldung.com/linux/find-command) 命令，用`findJar.sh`获得结果 jar:**

```java
$ find ~/.m2/repository -type f -name '*.jar' -exec ./findJar.sh {} com.fasterxml.jackson.databind.ObjectMapper \;
com/spotify/docker/client/shaded/com/fasterxml/jackson/databind/ObjectMapper$1.class
com/spotify/docker/client/shaded/com/fasterxml/jackson/databind/ObjectMapper$2.class
com/spotify/docker/client/shaded/com/fasterxml/jackson/databind/ObjectMapper$3.class
com/spotify/docker/client/shaded/com/fasterxml/jackson/databind/ObjectMapper$DefaultTypeResolverBuilder.class
com/spotify/docker/client/shaded/com/fasterxml/jackson/databind/ObjectMapper$DefaultTyping.class
com/spotify/docker/client/shaded/com/fasterxml/jackson/databind/ObjectMapper.class
Found in : <strong>/home/user/.m2/repository/com/spotify/docker-client/8.16.0/docker-client-8.16.0-shaded.jar</strong>
com/fasterxml/jackson/databind/ObjectMapper$1.class
com/fasterxml/jackson/databind/ObjectMapper$2.class
com/fasterxml/jackson/databind/ObjectMapper$3.class
com/fasterxml/jackson/databind/ObjectMapper$DefaultTypeResolverBuilder.class
com/fasterxml/jackson/databind/ObjectMapper$DefaultTyping.class
com/fasterxml/jackson/databind/ObjectMapper.class
Found in : <strong>/home/user/.m2/repository/com/fasterxml/jackson/core/jackson-databind/2.12.3/jackson-databind-2.12.3.jar</strong>
```

## 3.基于程序

在基于程序的方法中，**我们将编写一个 java 类来查找 Java 类路径中的`ObjectMapper`类。**我们可以显示如下所示的 jar 程序:

```java
public class App { 
    public static void main(String[] args) { 
        Class klass = ObjectMapper.class; 
        URL path = klass.getProtectionDomain().getCodeSource().getLocation(); 
        System.out.println(path); 
    } 
}
```

输出:

```java
file:/Users/home/.m2/repository/com/fasterxml/jackson/core/jackson-databind/2.12.3/jackson-databind-2.12.3.jar
```

这里我们看到每个`Class`类都有`getProtectionDomain().getCodeSource().getLocation()`。这个方法提供了 jar 文件，其中包含了所需的类。因此，我们可以使用它来获取包含该类的 jar 文件。

## 4.结论

在本文中，我们学习了从 jars 列表中查找类的命令和基于程序的方法。

首先我们从一个说明性的例子开始。之后，我们探索了一种基于命令的方法来从本地 maven 存储库中识别给定的类。然后，在第二种方法中，我们学习编写一个程序，从类路径中找到运行时使用的 jar 来实例化该类。

这两种方法都很有效，但是它们有自己的使用案例。