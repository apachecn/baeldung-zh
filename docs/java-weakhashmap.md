# Java 中的 WeakHashMap 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weakhashmap>

## 1。概述

在本文中，我们将看到来自`java.util` 包的`[WeakHashMap](https://web.archive.org/web/20221015012634/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/WeakHashMap.html)`。

为了理解数据结构，我们将在这里使用它来推出一个简单的缓存实现。但是，请记住，这是为了理解地图的工作方式，创建自己的缓存实现几乎总是一个坏主意。

简单地说，`WeakHashMap` 是`Map`接口的基于哈希表的实现，键是 [`WeakReference`](https://web.archive.org/web/20221015012634/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/WeakReference.html) 类型的。

当键不再被正常使用时，`WeakHashMap`中的条目将被自动删除，这意味着没有单独的`[Reference](https://web.archive.org/web/20221015012634/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/Reference.html)` 指向该键。当垃圾收集(GC)进程丢弃一个键时，它的条目被有效地从映射中删除，所以这个类的行为与其他`Map`实现有些不同。

## 2。强引用、软引用和弱引用

为了理解`WeakHashMap`如何工作，**我们需要看一下`WeakReference` 类**——它是`WeakHashMap` 实现中键的基本构造。在 Java 中，我们有三种主要类型的引用，我们将在下面的部分中解释。

### 2.1。强引用

强引用是我们在日常编程中最常用的`Reference`类型:

```java
Integer prime = 1;
```

变量`prime`有一个对值为 1 的`Integer`对象的*强引用*。任何被强引用指向的对象都没有资格进行 GC。

### 2.2。软引用

简单地说，一个有`[SoftReference](https://web.archive.org/web/20221015012634/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/SoftReference.html)` 指向的对象不会被垃圾收集，直到 JVM 绝对需要内存。

让我们看看如何用 Java 创建一个`SoftReference`:

```java
Integer prime = 1;  
SoftReference<Integer> soft = new SoftReference<Integer>(prime); 
prime = null;
```

`prime`对象有一个指向它的强引用。

接下来，我们将把`prime` 强引用包装成软引用。在进行了那个强引用`null`之后，`prime`对象有资格进行 GC，但是只有当 JVM 绝对需要内存时才会被收集。

### 2.3。弱引用

仅由弱引用引用的对象被急切地垃圾收集；在这种情况下，GC 不会等到它需要内存的时候。

我们可以用下面的方法在 Java 中创建一个`WeakReference`:

```java
Integer prime = 1;  
WeakReference<Integer> soft = new WeakReference<Integer>(prime); 
prime = null;
```

当我们创建一个`prime`引用`null`时，`prime`对象将在下一个 GC 周期被垃圾收集，因为没有其他强引用指向它。

**类型的引用被用作`WeakHashMap`中的关键字。**

## 3。`WeakHashMap`作为高效的内存缓存

假设我们想要构建一个缓存，将大图像对象作为值，将图像名称作为键。我们想选择一个合适的 map 实现来解决这个问题。

使用简单的`HashMap` 并不是一个好的选择，因为值对象可能会占用大量内存。此外，它们永远不会被 GC 进程从缓存中回收，即使在我们的应用程序中不再使用它们。

理想情况下，我们想要一个允许 GC 自动删除未使用对象的`Map` 实现。当一个大图像对象的一个键在我们的应用程序中的任何地方都没有被使用时，那个条目将从内存中删除。

幸运的是，`WeakHashMap` 恰恰具有这些特征。让我们测试一下我们的`WeakHashMap` ,看看它的表现如何:

```java
WeakHashMap<UniqueImageName, BigImage> map = new WeakHashMap<>();
BigImage bigImage = new BigImage("image_id");
UniqueImageName imageName = new UniqueImageName("name_of_big_image");

map.put(imageName, bigImage);
assertTrue(map.containsKey(imageName));

imageName = null;
System.gc();

await().atMost(10, TimeUnit.SECONDS).until(map::isEmpty);
```

我们正在创建一个`WeakHashMap` 实例，它将存储我们的`BigImage` 对象。我们将一个`BigImage` 对象作为值，一个`imageName`对象引用作为键。`imageName` 将作为`WeakReference`类型存储在地图中。

接下来，我们将`imageName`引用设置为`null`，因此不再有指向`bigImage`对象的引用。`WeakHashMap` 的默认行为是在下一次 GC 时回收一个对它没有引用的条目，所以这个条目将被下一次 GC 进程从内存中删除。

我们调用一个`System.gc()` 来强制 JVM 触发一个 GC 进程。在 GC 循环之后，我们的`WeakHashMap` 将是空的:

```java
WeakHashMap<UniqueImageName, BigImage> map = new WeakHashMap<>();
BigImage bigImageFirst = new BigImage("foo");
UniqueImageName imageNameFirst = new UniqueImageName("name_of_big_image");

BigImage bigImageSecond = new BigImage("foo_2");
UniqueImageName imageNameSecond = new UniqueImageName("name_of_big_image_2");

map.put(imageNameFirst, bigImageFirst);
map.put(imageNameSecond, bigImageSecond);

assertTrue(map.containsKey(imageNameFirst));
assertTrue(map.containsKey(imageNameSecond));

imageNameFirst = null;
System.gc();

await().atMost(10, TimeUnit.SECONDS)
  .until(() -> map.size() == 1);
await().atMost(10, TimeUnit.SECONDS)
  .until(() -> map.containsKey(imageNameSecond));
```

请注意，只有`imageNameFirst` 参考被设置为`null`。`imageNameSecond` 参考保持不变。触发 GC 后，映射将只包含一个条目—`imageNameSecond`。

## 4.结论

在本文中，我们研究了 Java 中的引用类型，以充分理解`java.util.` `WeakHashMap`是如何工作的。我们创建了一个简单的缓存，它利用了一个`WeakHashMap` 的行为，并测试它是否如我们预期的那样工作。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20221015012634/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-2)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。