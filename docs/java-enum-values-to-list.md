# 用 Java 中的所有枚举值填充一个列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-enum-values-to-list>

## 1.概观

Java 在 1.5 版本中引入了 [`enum`](/web/20221115043639/https://www.baeldung.com/a-guide-to-java-enums) 。将常量定义为`enum`使得代码可读性更好。此外，它允许编译时检查。

在这个快速教程中，让我们探索如何用一个`enum`类型的所有实例获得一个`List`。

## 2.问题简介

像往常一样，我们通过一个例子来理解这个问题。

首先，让我们创建一个`enum`类型`MagicNumber`:

```java
enum MagicNumber {
    ONE, TWO, THREE, FOUR, FIVE
}
```

然后，我们的目标是获得一个填充了所有`MagicNumber enum`实例的`List` :

```java
List<MagicNumber> EXPECTED_LIST = Arrays.asList(ONE, TWO, THREE, FOUR, FIVE);
```

这里，我们使用了`Arrays.asList()`方法来[初始化来自数组](/web/20221115043639/https://www.baeldung.com/java-init-list-one-line#create-from-an-array)的列表。

稍后，我们将探索几种不同的方法来获得预期的结果。最后，为了简单起见，我们将使用单元测试断言来验证每个方法是否给出了期望的结果。

接下来，让我们看看他们的行动。

## 3.使用`EnumType.values()`方法

当我们准备`EXPECTED_LIST, `时，我们从一个数组中初始化它。因此，如果我们可以从一个数组中的一个`enum`获得所有实例，我们就可以构建这个列表并解决这个问题。

**每个`enum`类型都提供了标准的`values()`方法来返回数组**中的所有实例。接下来，让我们从`MagicNumber.values()`开始建立一个列表:

```java
List<MagicNumber> result = Arrays.asList(MagicNumber.values());
assertEquals(EXPECTED_LIST, result);
```

如果我们进行测试，就会通过。我们已经得到了预期的列表。

## 4.使用`EnumType.class.getEnumConstants()`方法

我们已经看到使用`enum`类型的`values()`来获取数组中的所有`enum`实例。这是一个标准且简单的方法。但是，我们需要确切地知道`enum`类型的名称，并在代码中硬编码，例如`MagicNumber.values()`。换句话说，通过这种方式，我们无法构建一个适用于所有`enum`类型的实用方法。

从 Java 1.5 开始，**`Class`对象提供了从`enum Class`对象**获取所有`enum`实例的`getEnumConstants()`方法。因此，我们可以让`getEnumConstants()` 提供`enum`实例:

```java
List<MagicNumber> result = Arrays.asList(MagicNumber.class.getEnumConstants());
assertEquals(EXPECTED_LIST, result);
```

正如上面的测试所示，我们已经使用了`MagicNumber.class.getEnumConstants()`来提供`enum`实例数组。此外，很容易构建一个适用于所有`enum`类型的实用方法:

```java
static <T> List<T> enumValuesInList(Class<T> enumCls) {
    T[] arr = enumCls.getEnumConstants();
    return arr == null ? Collections.emptyList() : Arrays.asList(arr);
}
```

值得一提的是**如果`Class`对象不是`enum`类型，`getEnumConstants()`方法返回`null`** 。正如我们所见，在这种情况下，我们返回一个空的`List` 。

接下来，让我们创建一个测试来验证`enumValuesInList()`:

```java
List<MagicNumber> result1 = enumValuesInList(MagicNumber.class);
assertEquals(EXPECTED_LIST, result1);

List<Integer> result2 = enumValuesInList(Integer.class);
assertTrue(result2.isEmpty());
```

如果我们试一试，测试就会通过。正如我们看到的，如果类对象不在`enum`类型中，我们有一个空列表*。*

## 5.使用`EnumSet.allOf()`方法

从 1.5 版本开始，Java 引入了一个特殊的`Set`来处理`enum`类: [`EnumSet`](/web/20221115043639/https://www.baeldung.com/java-enumset) 。此外， **`EnumSet`有`allOf()`方法来加载给定`enum`类型**的所有实例。

因此，我们可以使用`ArrayList()`构造函数和填充的`EnumSet`构造一个`List`对象。接下来，让我们通过测试来看看它是如何工作的:

```java
List<MagicNumber> result = new ArrayList<>(EnumSet.allOf(MagicNumber.class));
assertEquals(EXPECTED_LIST, result);
```

值得一提的是，**调用`allOf()`方法以自然顺序存储`enum`的实例。**

## 6.结论

在本文中，我们学习了三种方法来获得包含所有`enum.`实例的`List`对象

像往常一样，这里展示的所有代码片段都可以在 GitHub 上获得。