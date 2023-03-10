# 使用番石榴的地图制作工具

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-mapmaker>

## 1.介绍

`MapMaker`是 Guava 中的一个生成器类，它使得创建线程安全映射变得容易。

Java 已经支持 *WeakHashMap* 对键使用[弱引用](/web/20220627075118/https://www.baeldung.com/java-weak-reference)。但是，没有现成的解决方案可以对这些值使用相同的值。**幸运的是，`MapMaker`提供了简单的构建器方法来为键和值**使用`WeakReference`。

在本教程中，让我们看看`MapMaker`如何使创建多个地图和使用弱引用变得容易。

## 2.Maven 依赖性

首先，我们来添加[谷歌番石榴](/web/20220627075118/https://www.baeldung.com/whats-new-in-guava-19)依赖，在 [Maven Central](https://web.archive.org/web/20220627075118/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22) 上有:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

## 3.缓存示例

让我们考虑一个服务器为用户维护两个缓存的简单场景:一个会话缓存和一个配置文件缓存。

会话缓存是短暂的，在用户不再活动后，其条目将变得无效。因此，在用户对象被垃圾收集后，缓存可以删除用户的条目。

然而，配置文件缓存可以具有更高的生存时间(TTL)。只有当用户更新其配置文件时，配置文件缓存中的条目才变得无效。

在这种情况下，只有当配置文件对象被垃圾收集时，缓存才可以删除该项。

### 3.1.数据结构

让我们创建类来表示这些实体。

我们将首先从用户开始:

```java
public class User {
    private long id;
    private String name;

    public User(long id, String name) {
        this.id = id;
        this.name = name;
    }

    public long getId() {
        return id;
    }

    public String getName() {
        return name;
    }
}
```

然后会话:

```java
public class Session {
    private long id;

    public Session(long id) {
        this.id = id;
    }

    public long getId() {
        return id;
    }
} 
```

最后是简介:

```java
public class Profile {
    private long id;
    private String type;

    public Profile(long id, String type) {
        this.id = id;
        this.type = type;
    }

    public long getId() {
        return id;
    }

    public String getName() {
        return type;
    }

}
```

### 3.2.创建缓存

让我们使用`makeMap`方法为会话缓存创建一个`ConcurrentMap` 实例:

```java
ConcurrentMap<User, Session> sessionCache = new MapMaker().makeMap();
```

**返回的映射不允许键和值都为空值。**

现在，让我们为概要文件缓存创建另一个`ConcurrentMap` 实例:

```java
ConcurrentMap<User, Profile> profileCache = new MapMaker().makeMap();
```

请注意，我们没有指定缓存的初始容量。所以， **`MapMaker`默认创建一个容量为 16 的地图。**

如果需要，我们可以使用`initialCapacity `方法修改容量:

```java
ConcurrentMap<User, Profile> profileCache = new MapMaker().initialCapacity(100).makeMap();
```

### 3.3.更改并发级别

`MapMaker`将并发级别的默认值**设置为 4** 。然而，`sessionCache`需要在没有任何线程争用的情况下支持更多的并发更新。

在这里，`concurrencyLevel` 构建器方法可以解决这个问题:

```java
ConcurrentMap<User, Session> sessionCache = new MapMaker().concurrencyLevel(10).makeMap();
```

### 3.4.使用弱引用

我们上面创建的映射对键和值都使用了强引用。因此，即使键和值被垃圾收集，条目也会保留在映射中。我们应该使用弱引用。

关键字(用户对象)被垃圾收集后，`sessionCache`条目无效。因此，让我们对键使用弱引用:

```java
ConcurrentMap<User, Session> sessionCache = new MapMaker().weakKeys().makeMap();
```

对于`profileCache`，我们可以对值使用弱引用:

```java
ConcurrentMap<User, Profile> profileCache = new MapMaker().weakValues().makeMap();
```

**当这些引用被垃圾收集时，Guava 保证这些`entries `不会被包含在对地图**的任何后续读或写操作中。然而，`size()`方法有时可能不一致，并且可能包含这些`entries`。

## 4.`MapMaker`内部构件

`**MapMaker**` **默认创建一个`[ConcurrentHashMap](/web/20220627075118/https://www.baeldung.com/java-concurrent-map#concurrenthashmap)`如果弱引用未启用** `**.**` 等式检查通过通常的 equals 方法进行。

**如果我们启用弱引用，那么`MapMaker`在内部创建一个由一组哈希表表示的自定义映射**。它还具有与`ConcurrentHashMap`相似的性能特征。

**然而，与`WeakHashMap`的一个重要区别是等式检查是通过身份(==和`identityHashCode`)比较来进行的。**

## 5.结论

在这篇短文中，我们学习了如何使用`MapMaker`类来创建线程安全映射。我们还了解了如何定制地图以使用弱引用。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220627075118/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-collections-map)