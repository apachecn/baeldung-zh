# 确定一个对象是否是原始类型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-object-primitive-type>

## 1.概观

有时我们需要确定一个对象是否是原始类型，特别是对于包装器原始类型。但是，标准 JDK 中没有内置的方法来实现这一点。

在这个快速教程中，我们将看到如何使用核心 Java 实现一个解决方案。然后，我们将看看如何使用几个常用的库来实现这一点。

## 2.原语和包装类

Java 中有九个预定义对象来表示八个原语和一个`void` 类型。每个原始类型都有一个对应的[包装类](/web/20220526060526/https://www.baeldung.com/java-wrapper-classes)。

要了解更多关于`Primitives`和`Object`的信息，请参见这篇[文章](/web/20220526060526/https://www.baeldung.com/java-primitives-vs-objects)。

**`java.lang.Class`。`isPrimitive()`方法可以确定指定的对象是否表示一个原语类型。然而，它不能在原语的包装器上工作。**

例如，以下语句返回`false`:

```java
Integer.class.isPrimitive(); 
```

现在，让我们来看看实现这一目标的不同方法。

## 3.使用核心 Java

首先，让我们定义一个 [`HashMap`](/web/20220526060526/https://www.baeldung.com/java-hashmap) 变量，它存储包装器和原始类型类:

```java
private static final Map<Class<?>, Class<?>> WRAPPER_TYPE_MAP;
static {
    WRAPPER_TYPE_MAP = new HashMap<Class<?>, Class<?>>(16);
    WRAPPER_TYPE_MAP.put(Integer.class, int.class);
    WRAPPER_TYPE_MAP.put(Byte.class, byte.class);
    WRAPPER_TYPE_MAP.put(Character.class, char.class);
    WRAPPER_TYPE_MAP.put(Boolean.class, boolean.class);
    WRAPPER_TYPE_MAP.put(Double.class, double.class);
    WRAPPER_TYPE_MAP.put(Float.class, float.class);
    WRAPPER_TYPE_MAP.put(Long.class, long.class);
    WRAPPER_TYPE_MAP.put(Short.class, short.class);
    WRAPPER_TYPE_MAP.put(Void.class, void.class);
}
```

**如果对象是一个原始包装类，我们可以用`java.utils.Map.ContainsKey()`方法从预定义的`HashMap`变量中查找。**

现在，我们可以创建一个简单的实用方法来确定对象源是否属于原始类型:

```java
public static boolean isPrimitiveType(Object source) {
    return WRAPPER_TYPE_MAP.containsKey(source.getClass());
}
```

让我们验证这是否如预期的那样工作:

```java
assertTrue(PrimitiveTypeUtil.isPrimitiveType(false));
assertTrue(PrimitiveTypeUtil.isPrimitiveType(1L));
assertFalse(PrimitiveTypeUtil.isPrimitiveType(StringUtils.EMPTY));
```

## 4.使用 Apache Commons-`ClassUtils.`*isPrimitiveOrWrapper()*

[阿帕奇公地朗](/web/20220526060526/https://www.baeldung.com/java-commons-lang-3)有 **`ClassUtils`。`isPrimitiveOrWrapper`用于确定一个类是原语还是原语的包装的方法**。

首先，让我们将来自 [Maven Central](https://web.archive.org/web/20220526060526/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22) 的`commons-lang3`依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.apache.commons<groupId>
    <artifactId>commons-lang3<artifactId>
    <version>3.12.0<version>
<dependency>
```

那我们来测试一下:

```java
assertTrue(ClassUtils.isPrimitiveOrWrapper(Boolean.False.getClass()));
assertTrue(ClassUtils.isPrimitiveOrWrapper(boolean.class));
assertFalse(ClassUtils.isPrimitiveOrWrapper(StringUtils.EMPTY.getClass()));
```

## 5.使用番石榴-`Primitives.`*isWrapperType()*

[Guava](/web/20220526060526/https://www.baeldung.com/whats-new-in-guava-19) 通过`Primitives.isWrapperType`方法提供了类似的实现。

同样，让我们先添加来自 [Maven Central](https://web.archive.org/web/20220526060526/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22) 的依赖项:

```java
<dependency>
    <groupId>com.google.guava<groupId>
    <artifactId>guava<artifactId>
    <version>31.0.1-jre<version>
<dependency>
```

同样，我们可以使用以下方法测试它:

```java
assertTrue(Primitives.isWrapperType(Boolean.FALSE.getClass()));
assertFalse(Primitives.isWrapperType(StringUtils.EMPTY.getClass()));
```

然而，`Primitives.isWrapperType`方法不能在原语类上工作，下面的代码将返回 false:

```java
assertFalse(Primitives.isWrapperType(boolean.class));
```

## 6.结论

在本教程中，我们用自己的 Java 实现说明了如何确定一个对象是否可以表示一个基本数据类型。然后我们看了几个流行的库，它们提供了实现这一点的实用方法。

完整的代码可以在 Github 上找到[。](https://web.archive.org/web/20220526060526/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-types)