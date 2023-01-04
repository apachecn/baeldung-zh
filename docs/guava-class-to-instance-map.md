# 番石榴分类指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-class-to-instance-map>

## 1。概述

`ClassToInstanceMap<B>`是一种特殊的映射，它将类与相应的实例关联起来。它确保所有的键和值都是上限类型`B.`的子类型

`ClassToInstanceMap`扩展了 Java 的`Map`接口，提供了两个额外的方法:`T getInstance(Class<T>)`和`T putInstance(Class<T>, T).`这个映射的好处是，这两个方法可以用来执行类型安全操作，避免强制转换。

在本教程中，我们将展示如何使用谷歌番石榴的`ClassToInstanceMap`界面及其实现。

## 2。谷歌番石榴的`ClassToInstanceMap`

让我们看看如何使用实现。

我们将从在`pom.xml`中添加 Google Guava 库依赖项开始:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

依赖关系的最新版本可以在[这里](https://web.archive.org/web/20220523235249/https://search.maven.org/classic/#search|gav|1|g%3A%22com.google.guava%22%20AND%20a%3A%22guava%22)查看。

接口有两个实现:可变的和不可变的。让我们分别来看看它们。

## 3。创造一个`ImmutableClassToInstanceMap`

我们可以用多种方式创建一个`ImmutableClassToInstanceMap`的实例:

*   使用`of()`方法创建空地图:

    ```java
    ImmutableClassToInstanceMap.of()
    ```

*   使用`of(Class<T> type, T value)`方法创建单入口映射:

    ```java
    ImmutableClassToInstanceMap.of(Save.class, new Save());
    ```

*   使用接受另一个地图作为参数的`copyOf()`方法。它将创建一个与参数:

    ```java
    ImmutableClassToInstanceMap.copyOf(someMap)
    ```

    中提供的 map 条目相同的 map
*   使用构建器:

    ```java
    ImmutableClassToInstanceMap.<Action>builder()
      .put(Save.class, new Save())
      .put(Open.class, new Open())
      .put(Delete.class, new Delete())
      .build();
    ```

## 4。创造一个`MutableClassToInstanceMap`

我们还可以创建一个`MutableClassToInstanceMap`的实例:

*   使用`create()`方法制作一个由`HashMap` :

    ```java
    MutableClassToInstanceMap.create();
    ```

    支持的实例
*   使用`create(Map<Class<? extends B>, B> backingMap)`生成一个由提供的空地图支持的实例:

    ```java
    MutableClassToInstanceMap.create(new HashMap());
    ```

## 5。用途

让我们看看如何使用添加到常规`Map`接口的两个新方法:

*   第一种方法是`<T extends B> T getInstance(Class<T> type)` :

    ```java
    Action openAction = map.get(Open.class);
    Delete deleteAction = map.getInstance(Delete.class);
    ```

*   第二种方法是`<T extends B> T putInstance(Class<T> type, @Nullable T value)` :

    ```java
    Action newOpen = map.put(Open.class, new Open());
    Delete newDelete = map.putInstance(Delete.class, new Delete());
    ```

## 6。结论

在这个快速教程中，我们展示了如何使用番石榴库中的`ClassToInstanceMap`的例子。

这些例子的实现可以在[GitHub 项目](https://web.archive.org/web/20220523235249/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-collections-map)中找到。