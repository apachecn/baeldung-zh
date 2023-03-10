# Java 10 的新特性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-10-overview>

[This article is part of a series:](javascript:void(0);)[• Java 10 LocalVariable Type-Inference](/web/20220824084102/https://www.baeldung.com/java-10-local-variable-type-inference)
[• Java 10 Performance Improvements](/web/20220824084102/https://www.baeldung.com/java-10-performance-improvements)
• New Features in Java 10 (current article)[This article is part of a series:](javascript:void(0);)[• New Features in Java 8](/web/20220824084102/https://www.baeldung.com/java-8-new-features)
[• New Features in Java 9](/web/20220824084102/https://www.baeldung.com/new-java-9)
• New Features in Java 10 (current article)[• New Features in Java 11](/web/20220824084102/https://www.baeldung.com/java-11-new-features)
[• New Features in Java 12](/web/20220824084102/https://www.baeldung.com/java-12-new-features)
[• New Features in Java 13](/web/20220824084102/https://www.baeldung.com/java-13-new-features)
[• New Features in Java 14](/web/20220824084102/https://www.baeldung.com/java-14-new-features)
[• What's New in Java 15](/web/20220824084102/https://www.baeldung.com/java-15-new)
[• New Features in Java 16](/web/20220824084102/https://www.baeldung.com/java-16-new-features)
[• New Features in Java 17](/web/20220824084102/https://www.baeldung.com/java-17-new-features)

## 1。简介

[JDK 10](https://web.archive.org/web/20220824084102/https://openjdk.java.net/projects/jdk/10) ，是 Java SE 10 的实现，于 2018 年 3 月 20 日发布。

在本文中，我们将介绍和探索 JDK 10 中引入的新功能和变化。

## 2。局部变量类型推理

请点击链接阅读关于该特性的深入文章:

[Java 10 局部变量类型推断](/web/20220824084102/https://www.baeldung.com/java-10-local-variable-type-inference)

## 3。不可修改的集合

Java 10 中有一些与不可修改集合相关的变化。

### 3.1.`copyOf()`

`java.util.List`、`java.util.Map `和`java.util.Set `各得到一个新的静态方法`copyOf(Collection)`。

它返回给定`Collection:`的不可修改副本

```java
@Test(expected = UnsupportedOperationException.class)
public void whenModifyCopyOfList_thenThrowsException() {
    List<Integer> copyList = List.copyOf(someIntList);
    copyList.add(4);
}
```

任何修改此类集合的尝试都会导致`java.lang.UnsupportedOperationException`运行时异常。

### 3.2.`toUnmodifiable*()`

`java.util.stream.Collectors `获得额外的方法将一个`Stream `收集成不可修改的`List`、`Map `或`Set`:

```java
@Test(expected = UnsupportedOperationException.class)
public void whenModifyToUnmodifiableList_thenThrowsException() {
    List<Integer> evenList = someIntList.stream()
      .filter(i -> i % 2 == 0)
      .collect(Collectors.toUnmodifiableList());
    evenList.add(4);
}
```

任何修改此类集合的尝试都会导致`java.lang.UnsupportedOperationException`运行时异常。

## 4。`Optional*.orElseThrow()`

`java.util.Optional`、`java.util.OptionalDouble`、`java.util.OptionalInt`和`java.util.OptionalLong`各自得到了一个新方法`orElseThrow()`，该方法不接受任何参数，如果没有值，则抛出`NoSuchElementException`:

```java
@Test
public void whenListContainsInteger_OrElseThrowReturnsInteger() {
    Integer firstEven = someIntList.stream()
      .filter(i -> i % 2 == 0)
      .findFirst()
      .orElseThrow();
    is(firstEven).equals(Integer.valueOf(2));
}
```

**它与现有的`get()`方法同义，现在是首选的替代方法。**

## 5。性能改进

请点击链接阅读关于该特性的深入文章:

[Java 10 的性能提升](/web/20220824084102/https://www.baeldung.com/java-10-performance-improvements)

## 6。集装箱意识

JVM 现在意识到正在 Docker 容器中运行，并将提取特定于容器的配置，而不是查询操作系统本身——它适用于 CPU 数量和分配给容器的总内存等数据。

但是，这种支持仅适用于基于 Linux 的平台。这种新的支持是默认启用的，可以在命令行中使用 JVM 选项禁用:

```java
-XX:-UseContainerSupport
```

此外，此更改还添加了一个 JVM 选项，该选项提供了指定 JVM 将使用的 CPU 数量的能力:

```java
-XX:ActiveProcessorCount=count
```

此外，还添加了三个新的 JVM 选项，允许 Docker 容器用户对将用于 Java 堆的系统内存量进行更细粒度的控制:

```java
-XX:InitialRAMPercentage
-XX:MaxRAMPercentage
-XX:MinRAMPercentage
```

## 7.根证书

cacerts 密钥库最初是空的，它旨在包含一组根证书，这些证书可用于在各种安全协议使用的证书链中建立信任。

因此，在 OpenJDK 构建下，TLS 等关键安全组件在默认情况下无法工作。

**在 Java 10 中，Oracle 开源了 Oracle Java SE Root CA 程序中的根证书**,以使 OpenJDK 版本对开发人员更具吸引力，并减少这些版本与 Oracle JDK 版本之间的差异。

## 8。折旧和移除

### 8.1。命令行选项和工具

工具`javah `已经从 Java 10 中移除，该工具生成实现本地方法所需的 C 头文件和源文件，现在可以使用`javac -h `来代替。

`policytool `是基于 UI 的工具，用于创建和管理策略文件。现已移除。用户可以使用简单的文本编辑器来执行此操作。

删除了`java -Xprof`选项。该选项用于分析正在运行的程序，并将分析数据发送到标准输出。用户现在应该使用`jmap `工具。

### 8.2。API

不推荐使用的 java.security.acl 包已标记为 forRemoval=true，在 Java SE 的未来版本中可能会被删除。它已经被`java.security.Policy `和相关类所取代。

类似地，java.security.{Certificate，Identity，IdentityScope，Signer } APIs 被标记为`forRemoval=true`。

## 9。基于时间的版本控制

从 Java 10 开始，Oracle 已经转向基于时间的 Java 版本。这具有以下含义:

1.  每六个月发布一个新的 Java 版本。2018 年 3 月发布的是 JDK 10，2018 年 9 月发布的是 JDK 11，依此类推。这些被称为特性版本，预计至少包含一个或两个重要特性
2.  **对功能发布的支持仅持续六个月**，即直到下一个功能发布
3.  **长期支持版本将被标记为 LTS。对该版本的支持将持续三年**
4.  Java 11 将是一个 LTS 版本

**`java -version `现在将包含正式发布日期**，从而更容易识别该版本的发布时间:

```java
$ java -version
openjdk version "10" 2018-03-20
OpenJDK Runtime Environment 18.3 (build 10+46)
OpenJDK 64-Bit Server VM 18.3 (build 10+46, mixed mode)
```

## 10。结论

在本文中，我们看到了 Java 10 带来的新特性和变化。

像往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220824084102/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-10)

Next **»**[New Features in Java 11](/web/20220824084102/https://www.baeldung.com/java-11-new-features)**«** Previous[New Features in Java 9](/web/20220824084102/https://www.baeldung.com/new-java-9)