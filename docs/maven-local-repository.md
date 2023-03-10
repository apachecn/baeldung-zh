# Maven 本地存储库在哪里？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-local-repository>

## 1。概述

在本教程中，我们将关注 Maven 在本地存储所有本地依赖项的位置，这是 Maven 本地存储库中的**。**

简单地说，当我们运行 Maven 构建时，我们项目的依赖项(jar、插件 jar、其他工件)都存储在本地以备后用。

还要记住，除了这种类型的本地存储库，Maven 还支持三种类型的存储库:

*   `Local` –本地开发机器上的文件夹位置
*   `Central`–由 Maven 社区提供的存储库
*   `Remote`–组织拥有的自定义存储库

现在让我们关注本地存储库。

## 2。本地存储库

Maven 的本地存储库是本地机器上存储所有项目工件的目录。

当我们执行 Maven 构建时，Maven 会自动将所有的依赖 jar 下载到本地存储库中。通常，这个目录被命名为`.m2`。

以下是基于操作系统的默认本地存储库的位置:

```java
Windows: C:\Users\<User_Name>\.m2
```

```java
Linux: /home/<User_Name>/.m2
```

```java
Mac: /Users/<user_name>/.m2
```

对于 Linux 和 Mac，我们可以用简短的形式写:

```java
~/.m2
```

## 3。在`settings.xml` 中自定义本地库

如果回购不在这个默认位置，可能是因为一些预先存在的配置。

这个配置文件位于 Maven 安装目录下的一个名为`settings.xml`的文件夹中。

下面是确定我们丢失的本地回购位置的相关配置:

```java
<settings>
    <localRepository>C:/maven_repository</localRepository>
    ...
```

这实质上是我们如何改变本地回购的位置。当然，如果我们改变位置，我们将不再在默认位置找到回购协议。

**储存在先前位置的文件不会自动移动。**

## 4.通过命令行传递本地存储库位置

除了在 Maven 的`settings.xml`中设置定制的本地存储库之外，`mvn`命令还支持`maven.repo.local`属性，这允许我们将本地存储库位置作为命令行参数传递:

```java
mvn -Dmaven.repo.local=/my/local/repository/path clean install
```

这样我们就不用换 Maven 的`settings.xml`。

## 5。结论

在这篇简短的文章中，我们研究了 Maven 本地存储库的默认设置。

我们还讨论了如何告诉 Maven 使用定制的本地存储库位置。