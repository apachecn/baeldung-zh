# 番石榴地图指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-bimap>

## 1。概述

在本教程中，我们将展示如何使用 Google Guava 的`BiMap`界面及其多种实现。

`BiMap`(或“双向映射”)是一种特殊的映射，它维护映射的反向视图，同时确保不存在重复值，并且总是可以安全地使用某个值来取回密钥。

`BiMap`的基本实现是`HashBiMap`，它在内部使用两个`Map`，一个用于键到值的映射，另一个用于值到键的映射。

## 2。谷歌番石榴的`BiMap`

让我们来看看如何使用`BiMap`类。

我们将从在`pom.xml`中添加 Google Guava 库依赖项开始:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>21.0</version>
</dependency>
```

依赖关系的最新版本可以在[这里](https://web.archive.org/web/20220117212928/https://search.maven.org/classic/#search|gav|1|g%3A%22com.google.guava%22%20AND%20a%3A%22guava%22)查看。

## 3。创建双地图

您可以通过多种方式创建`BiMap`的实例，如下所示:

*   如果要处理一个定制的 Java 对象，使用 HashBiMap 类中的`create`方法:

```java
BiMap<String, String> capitalCountryBiMap = HashBiMap.create();
```

*   如果我们已经有一个现有的映射，您可以使用来自类`HashBiMap`的`create`方法的重载版本创建一个`BiMap`的实例:

```java
Map<String, String> capitalCountryBiMap = new HashMap<>();
//...
HashBiMap.create(capitalCountryBiMap); 
```

*   如果你要处理一个类型为`Enum,` 的键，使用`EnumHashBiMap`类中的`create`方法:

```java
BiMap<MyEnum, String> operationStringBiMap = EnumHashBiMap.create(MyEnum.class); 
```

*   如果您打算创建一个不可变的映射，使用`ImmutableBiMap`类(它遵循一个构建器模式):

```java
BiMap<String, String> capitalCountryBiMap
  = new ImmutableBiMap.Builder<>()
    .put("New Delhi", "India")
    .build(); 
```

## 4。使用双地图

让我们从一个简单的例子开始，展示一下`BiMap,`的用法，我们可以得到一个基于值的键和一个基于键的值:

```java
@Test
public void givenBiMap_whenQueryByValue_shouldReturnKey() {
    BiMap<String, String> capitalCountryBiMap = HashBiMap.create();
    capitalCountryBiMap.put("New Delhi", "India");
    capitalCountryBiMap.put("Washington, D.C.", "USA");
    capitalCountryBiMap.put("Moscow", "Russia");

    String keyFromBiMap = capitalCountryBiMap.inverse().get("Russia");
    String valueFromBiMap = capitalCountryBiMap.get("Washington, D.C.");

    assertEquals("Moscow", keyFromBiMap);
    assertEquals("USA", valueFromBiMap);
}
```

注意:上面的`inverse`方法返回了`BiMap`的逆视图，它将每个 BiMap 的值映射到其相关的键。

当我们试图存储一个重复值两次时,`BiMap`抛出一个`IllegalArgumentException`。

让我们看一个同样的例子:

```java
@Test(expected = IllegalArgumentException.class)
public void givenBiMap_whenSameValueIsPresent_shouldThrowException() {
    BiMap<String, String> capitalCountryBiMap = HashBiMap.create();
    capitalCountryBiMap.put("Mumbai", "India");
    capitalCountryBiMap.put("Washington, D.C.", "USA");
    capitalCountryBiMap.put("Moscow", "Russia");
    capitalCountryBiMap.put("New Delhi", "India");
} 
```

如果我们希望覆盖已经存在于`BiMap`中的值，我们可以使用`forcePut`方法:

```java
@Test
public void givenSameValueIsPresent_whenForcePut_completesSuccessfully() {
    BiMap<String, String> capitalCountryBiMap = HashBiMap.create();
    capitalCountryBiMap.put("Mumbai", "India");
    capitalCountryBiMap.put("Washington, D.C.", "USA");
    capitalCountryBiMap.put("Moscow", "Russia");
    capitalCountryBiMap.forcePut("New Delhi", "India");

    assertEquals("USA", capitalCountryBiMap.get("Washington, D.C."));
    assertEquals("Washington, D.C.", capitalCountryBiMap.inverse().get("USA"));
}
```

## 5。结论

在这篇简明教程中，我们举例说明了在番石榴库中使用`BiMap`的例子。它主要用于根据 map 中的值获取一个键。

这些例子的实现可以在 GitHub 项目中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。