# 将 Java 属性转换为 HashMap

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-properties-to-hashmap>

## 1.介绍

许多开发人员决定将应用程序参数存储在源代码之外。在 Java 中这样做的方法之一是使用外部配置文件，并通过 [`java.util.Properties`](/web/20220628063116/https://www.baeldung.com/java-properties) 类读取它们。

在本教程中，我们将重点介绍**将`java.util.Properties`转换为 [`HashMap<String, String>`](/web/20220628063116/https://www.baeldung.com/java-hashmap)** 的各种方法。我们将使用普通 Java、 [lambdas](/web/20220628063116/https://www.baeldung.com/java-8-lambda-expressions-tips) 或外部库实现不同的方法来实现我们的目标。通过示例，我们将讨论每个解决方案的优缺点。

## 2.`HashMap`构造器

在我们实现第一个代码之前，让我们检查一下`java.util.Properties` 的 [Javadoc。正如我们看到的，这个实用程序类继承了](https://web.archive.org/web/20220628063116/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Properties.html) [`Hashtable<Object, Object>`](/web/20220628063116/https://www.baeldung.com/java-hash-table) ，它还实现了 [`Map`](https://web.archive.org/web/20220628063116/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html) 接口。此外，Java 包装了它的`Reader`和`Writer`类来直接处理`String`值。

根据这些信息，我们可以使用类型转换和构造函数调用将`Properties`转换成`HashMap<String, String>`。

假设我们已经正确加载了我们的`Properties`,我们可以实现:

```java
public static HashMap<String, String> typeCastConvert(Properties prop) {
    Map step1 = prop;
    Map<String, String> step2 = (Map<String, String>) step1;
    return new HashMap<>(step2);
}
```

在这里，我们用三个简单的步骤来实现我们的转换。

首先，根据继承图，我们需要将我们的`Properties`转换成一个原始的`Map`。这个动作将强制第一个编译器警告，可以通过使用 [`@SuppressWarnings(“rawtypes”)`注释](/web/20220628063116/https://www.baeldung.com/java-suppresswarnings)来禁用。

之后，我们将原始的`Map`转换成`Map<String, String>`，导致另一个编译器警告，可以通过使用`@SupressWarnings(“unchecked”)`来省略。

最后，我们使用[复制构造器](https://web.archive.org/web/20220628063116/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/HashMap.html#%3Cinit%3E(java.util.Map))构建我们的`HashMap`。这是转换我们的`Properties`最快的方式**，但是这个解决方案也有一个与类型安全相关的很大的缺点**:我们的`Properties`可能会在转换前被破坏和修改。

根据文档，`Properties`类具有强制使用`String`值的 [`setProperty()`](https://web.archive.org/web/20220628063116/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Properties.html#setProperty(java.lang.String,java.lang.String)) 和 [`getProperty()`](https://web.archive.org/web/20220628063116/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Properties.html#getProperty(java.lang.String)) 方法。但是从`Hashtable`继承而来的 [`put()`](https://web.archive.org/web/20220628063116/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Hashtable.html#put(K,V)) 和 [`putAll()`](https://web.archive.org/web/20220628063116/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Hashtable.html#putAll(java.util.Map)) 方法允许在我们的`Properties`中使用任何类型作为键或值:

```java
properties.put("property4", 456);
properties.put(5, 10.11);

HashMap<String, String> hMap = typeCastConvert(properties);
assertThrows(ClassCastException.class, () -> {
    String s = hMap.get("property4");
});
assertEquals(Integer.class, ((Object) hMap.get("property4")).getClass());

assertThrows(ClassCastException.class, () -> {
    String s = hMap.get(5);
});
assertEquals(Double.class, ((Object) hMap.get(5)).getClass());
```

正如我们所看到的，我们的转换执行时没有任何错误，但是并不是所有的`HashMap`中的元素都是字符串`.` 所以，即使**这个方法看起来最简单，我们也必须记住将来一些安全相关的检查**。

## 3.番石榴 API

如果我们可以使用第三方库，[谷歌番石榴 API](/web/20220628063116/https://www.baeldung.com/guava-guide) 就派上用场了。这个库提供了一个静态的`[Maps.fromProperties()](https://web.archive.org/web/20220628063116/https://guava.dev/releases/snapshot-jre/api/docs/com/google/common/collect/Maps.html#fromProperties-java.util.Properties-)`方法，它几乎为我们做了所有的事情。根据文档，这个调用返回一个 [`ImmutableMap`](/web/20220628063116/https://www.baeldung.com/java-immutable-maps#guava-immutable-map) ，所以如果我们想要有`HashMap,`我们可以使用:

```java
public HashMap<String, String> guavaConvert(Properties prop) {
    return Maps.newHashMap(Maps.fromProperties(prop));
}
```

如前所述，当我们完全确定`Properties`只包含`String` 值时，**这种方法** **工作正常。**拥有一些不一致的价值观会导致意想不到的行为:

```java
properties.put("property4", 456);
assertThrows(NullPointerException.class, 
    () -> PropertiesToHashMapConverter.guavaConvert(properties));

properties.put(5, 10.11);
assertThrows(ClassCastException.class, 
    () -> PropertiesToHashMapConverter.guavaConvert(properties));
```

Guava API 不执行任何额外的映射。结果，它不允许我们转换那些`Properties`，抛出异常。

在第一种情况下，`NullPointerException`由于`Integer`值而被抛出，该值不能被`Properties.` `getProperty()`方法检索，因此被解释为`null`。第二个示例抛出与输入属性映射上出现的*非字符串*键相关的`ClassCastException`。

这个解决方案给了我们更好的类型控制，也通知了我们在转换过程中发生的违反 **的情况。**

## 4.自定义类型安全实现

现在是时候解决前面例子中的安全问题了。正如我们所知，`Properties`类实现了从`Map`接口继承的方法。我们将使用[迭代`Map`](/web/20220628063116/https://www.baeldung.com/java-iterate-map) 的一种可能方式来实现一个合适的解决方案，并用类型检查来丰富它。

### 4.1.使用`for` 循环进行迭代

让我们实现一个简单的`for`-循环:

```java
public HashMap<String, String> loopConvert(Properties prop) {
    HashMap<String, String> retMap = new HashMap<>();
    for (Map.Entry<Object, Object> entry : prop.entrySet()) {
        retMap.put(String.valueOf(entry.getKey()), String.valueOf(entry.getValue()));
    }
    return retMap;
}
```

在这个方法中，我们以与典型的`Map`相同的方式迭代`Properties`。因此，我们可以逐个访问由 [`Map.Entry`](https://web.archive.org/web/20220628063116/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.Entry.html) 类表示的每个密钥对值。

在将值放入返回的`HashMap`之前，我们可以执行额外的检查，所以我们决定使用 [`String.valueOf()`](https://web.archive.org/web/20220628063116/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#valueOf(java.lang.Object)) 方法。

### 4.2.`Stream`和`Collectors` API

我们甚至可以使用现代的 Java 8 方式重构我们的方法:

```java
public HashMap<String, String> streamConvert(Properties prop) {
    return prop.entrySet().stream().collect(
      Collectors.toMap(
        e -> String.valueOf(e.getKey()),
        e -> String.valueOf(e.getValue()),
        (prev, next) -> next, HashMap::new
    ));
}
```

在这种情况下，我们使用的是没有显式构造的 [Java 8 流收集器](/web/20220628063116/https://www.baeldung.com/java-collectors-tomap)。该方法实现了与上一个示例中介绍的完全相同的逻辑。

**这两种解决方案都稍微复杂一些，因为它们需要一些定制实现**，而类型转换和番石榴示例则不需要。

**然而，在将值**放到结果`HashMap`、**之前，我们可以访问它们，因此我们可以实现额外的检查或映射**:

```java
properties.put("property4", 456);
properties.put(5, 10.11);

HashMap<String, String> hMap1 = loopConvert(properties);
HashMap<String, String> hMap2 = streamConvert(properties);

assertDoesNotThrow(() -> {
    String s1 = hMap1.get("property4");
    String s2 = hMap2.get("property4");
});
assertEquals("456", hMap1.get("property4"));
assertEquals("456", hMap2.get("property4"));

assertDoesNotThrow(() -> {
    String s1 = hMap1.get("property4");
    String s2 = hMap2.get("property4");
});
assertEquals("10.11", hMap1.get("5"));
assertEquals("10.11", hMap2.get("5"));

assertEquals(hMap2, hMap1);
```

正如我们所看到的，我们解决了与非字符串值相关的问题。使用这种方法，我们可以手动调整映射逻辑来实现正确的实现。

## 5.结论

在本教程中，我们检查了将`java.util.Properties`转换成`HashMap<String, String>`的不同方法。

我们从类型转换解决方案开始，这可能是最快的转换，但也会带来编译器警告和**潜在的类型安全错误**。

然后我们看了一个使用 Guava API 的解决方案，它解决了编译器警告，并带来了一些处理错误的改进。

最后，我们实现了我们的自定义方法，这些方法处理类型安全错误并给予我们最大的控制权。

本教程的所有代码片段都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220628063116/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-3)