# 清除 Maven 缓存

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-clear-cache>

## 1.概观

在这个简短的教程中，我们将探索清除本地 [Maven](/web/20221128045513/https://www.baeldung.com/maven) 缓存的方法。我们可能希望这样做，以便节省磁盘空间或清理我们不再引用的工件。

我们将首先手动清除缓存，在这里我们物理地删除目录。然后，我们将使用 Maven Dependency 插件清除缓存，使用一些不同的插件选项。

## 2.删除本地缓存目录

我们的[本地 Maven 库](/web/20221128045513/https://www.baeldung.com/maven-local-repository#Repository)基于操作系统存储在不同的位置。此外，作为。目录可能是隐藏的，我们需要更改目录属性才能显示它。

在 Windows 中，默认位置是:

```java
C:\Users\<user_name>\.m2
```

在 Mac 上:

```java
/Users/<user_name>/.m2 
```

在基于 Linux 的系统上:

```java
/home/<user_name>/.m2
```

找到目录后，我们可以简单地删除文件夹`.m2/repository.`对于基于 Unix 的系统，如 MacOS 或 Linux，我们可以用一个命令删除目录:

```java
rm -rf ~/.m2/repository
```

如果我们的缓存目录不在默认位置，我们可以使用 [Maven 帮助插件](https://web.archive.org/web/20221128045513/https://maven.apache.org/plugins/maven-help-plugin/evaluate-mojo.html)来定位它:

```java
mvn help:evaluate -Dexpression=settings.localRepository -q -DforceStdout 
```

## 3.使用 Maven 依赖插件

我们可以使用目标为`purge-local-repository`的 [Maven 依赖插件](https://web.archive.org/web/20221128045513/https://mvnrepository.com/artifact/org.apache.maven.plugins/maven-dependency-plugin)，而不是直接删除缓存目录。

首先，我们需要导航到 Maven 项目的根目录。然后，我们可以运行:

```java
mvn dependency:purge-local-repository
```

当我们在没有任何附加标志的情况下运行这个插件时，它可能会下载缓存中不存在的工件来解析依赖树。这就是所谓的传递依赖解决方案。接下来，**它删除我们的本地缓存，最后，重新下载工件。**

或者，为了删除我们的缓存并避免预下载缺失依赖项的第一步，我们可以传入标志`actTransitively=false`:

```java
mvn dependency:purge-local-repository -DactTransitively=false
```

最后，**如果我们只是想清空我们的缓存，而不是预下载或重新解析工件**:

```java
mvn dependency:purge-local-repository -DactTransitively=false -DreResolve=false
```

这里，我们传入一个额外的标志`reResolve=false`，它告诉插件避免重新下载依赖项。

## 4.结论

在这篇短文中，我们研究了两种清除本地 Maven 缓存的方法。

首先，我们研究了手动清空本地缓存目录。然后，我们使用 Maven 依赖插件，探索不同的选项来实现我们想要的结果。