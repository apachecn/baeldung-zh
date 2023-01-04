# 在 Java 中比较长值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-compare-long-values>

## 1.概观

在这个简短的教程中，我们将讨论比较两个`Long`实例的不同方法。我们强调使用引用比较操作符(`==`)时出现的问题。

## 2.使用引用比较时出现问题

`Long`是原始类型`long`的[包装类](/web/20220628155747/https://www.baeldung.com/java-wrapper-classes)。由于它们是对象而不是原始值，**我们需要使用`.equals()`而不是引用比较操作符(==)来比较`Long`实例的内容。**

在某些情况下，我们可能会认为==没问题，但外表是具有欺骗性的。考虑到我们可以将==用于较小的数字:

```java
Long l1 = 127L;
Long l2 = 127L;

assertThat(l1 == l2).isTrue();
```

但数量不多。如果值超出了-128 到 127 的范围，就会出现完全不同的意外结果:

```java
Long l1 = 128L;
Long l2 = 128L;

assertThat(l1 == l2).isFalse();
```

这是因为 **[Java 为-128 和 127](https://web.archive.org/web/20220628155747/https://docs.oracle.com/javase/specs/jls/se7/html/jls-5.html#jls-5.1.7) 之间的`Long` 实例维护了一个常量池。**

然而，这种优化并没有给我们使用==的许可。在一般情况下，具有相同原始值的两个装箱实例不会产生相同的对象引用。

## 3.使用`.equals()`

解决方法之一是使用 [`.equals()`](https://web.archive.org/web/20220628155747/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Long.html#equals(java.lang.Object)) 。**这将评估两个对象的内容**(而非参考):

```java
Long l1 = 128L;
Long l2 = 128L;

assertThat(l1.equals(l2)).isTrue();
```

## 4.`Objects.equals()`

使用`equals()`的问题是我们需要小心不要在`null`引用中调用它。

幸运的是，有一个**`null`——安全的实用方法我们可以使用——`[Objects.equals()](https://web.archive.org/web/20220628155747/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Objects.html#equals(java.lang.Object,java.lang.Object)).`**

让我们看看它在实践中是如何工作的:

```java
Long l1 = null;
Long l2 = 128L;

assertThatCode(() -> Objects.equals(l1, l2)).doesNotThrowAnyException();
```

正如我们所看到的，我们不需要担心我们想要比较的`Long`是否是`null.`

在幕后，`Objects.equals()`首先使用==操作符进行比较，如果失败，它就使用标准的`equals().`

## 5.取消装箱长整型值

### 5.1.使用`.longValue()`方法

接下来，让我们使用“==”比较运算符，但是要安全。类`Number`有一个方法 [`.longValue()`](https://web.archive.org/web/20220628155747/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Number.html#longValue()) ，该方法解开原始的`long`值:

```java
Long l1 = 128L;
Long l2 = 128L;

assertThat(l1.longValue() == l2.longValue()).isTrue();
```

### 5.2.转换为原始值

对`Long`取消装箱的另一种方式是通过[将对象转换为原始类型的](/web/20220628155747/https://www.baeldung.com/java-type-casting)。因此，我们将提取原始值，然后我们可以继续使用比较运算符:

```java
Long l1 = 128L;
Long l2 = 128L;

assertThat((long) l1 == (long) l2).isTrue();
```

注意，**对于`.longValue()`方法或者使用铸造，我们应该检查对象是否是`null`。如果对象是`null`，我们可以有一个`NullPointerException`。**

## 6.结论

在这个简短的教程中，**我们探讨了如何比较`Long`对象的不同选项。**我们已经分析了在比较对对象或内容的引用时的差异。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220628155747/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-2)