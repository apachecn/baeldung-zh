# 如何用 Java 检查一个整数是否在一个范围内

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-interval-contains-integer>

## 1.概观

在本教程中，我们将研究一些方法来检查一个整数是否存在于给定的范围内。我们将使用操作符和几个实用程序类来实现这一点。

## 2.范围类型

在我们使用这些方法之前，我们必须清楚我们正在谈论的是什么样的范围。在本教程中，我们将重点介绍这四种有界值域类型:

*   **封闭范围**–**包括其下限和上限**
*   **开放范围**–**不包括其下限和上限**
*   **左开右闭范围**–**包含上界，不包含下界**
*   **左闭右开范围**–**包含其下限，不包含其上限**

例如，假设我们想知道整数 20 是否出现在这两个范围内:`R1 = [10, 2o)`，左闭右开范围，和`R2 = (10, 20]`，左开右闭范围。由于`R1`不包含它的上界，所以整数 20 只存在于`R2`中。

## 3.使用`<`和`<=`运算符

我们的目标是确定一个数是否在给定的下限和上限之间。我们将首先使用基本的 Java 操作符来检查这一点。

让我们定义一个对所有四种范围进行检查的类:

```
public class IntRangeOperators {

    public static boolean isInClosedRange(Integer number, Integer lowerBound, Integer upperBound) {
        return (lowerBound <= number && number <= upperBound);
    }

    public static boolean isInOpenRange(Integer number, Integer lowerBound, Integer upperBound) {
        return (lowerBound < number && number < upperBound);
    }

    public static boolean isInOpenClosedRange(Integer number, Integer lowerBound, Integer upperBound) {
        return (lowerBound < number && number <= upperBound);
    }

    public static boolean isInClosedOpenRange(Integer number, Integer lowerBound, Integer upperBound) {
        return (lowerBound <= number && number < upperBound);
    }
}
```

这里，**通过改变操作符来包含或排除边界，我们可以将区间调整为开放、封闭或半开。**

让我们来测试一下我们的`static` `isInOpenClosedRange()`方法。我们将通过为下限传入 10 和为上限传入 20 来指定左开右闭范围`(10,20]`:

```
assertTrue(IntRangeClassic.isInOpenClosedRange(20, 10, 20));

assertFalse(IntRangeClassic.isInOpenClosedRange(10, 10, 20));
```

在我们的第一个测试中，我们成功地验证了整数 20 存在于`(10,20]`范围内，这个范围包括了它的上界。然后我们确认整数 10 不存在于同一个范围内，排除了它的下界。

## 4.使用范围类

作为使用 Java 操作符的替代方法，我们也可以使用表示范围的实用程序类。使用预定义类的主要好处是 **range 类为上面描述的一些或所有 range 类型提供了现成的实现。**

此外，**我们可以用我们定义的边界配置一个 range 对象，并在其他方法或类中重用该对象**。通过定义一次范围，如果我们需要在整个代码库中对同一范围进行多次检查，我们的代码就不容易出错。

另一方面，我们下面要看的两个 range 类位于外部库中，在我们使用它们之前，必须将它们导入到我们的项目中。

### 4.1.**使用`java.time.temporal.ValueRange`**

不需要导入外部库的 range 类是在 JDK 1.8 中引入的`java.time.temporal.ValueRange`:

```
public class IntRangeValueRange {

    public boolean isInClosedRange(Integer number, Integer lowerBound, Integer upperBound) {
        final ValueRange range = ValueRange.of(lowerBound, upperBound);
        return range.isValidIntValue(number);
    }

    public boolean isInOpenRange(Integer number, Integer lowerBound, Integer upperBound) {
        final ValueRange range = ValueRange.of(lowerBound + 1, upperBound - 1);
        return range.isValidIntValue(number);
    }

    public boolean isInOpenClosedRange(Integer number, Integer lowerBound, Integer upperBound) {
        final ValueRange range = ValueRange.of(lowerBound + 1, upperBound);
        return range.isValidIntValue(number);
    }

    public boolean isInClosedOpenRange(Integer number, Integer lowerBound, Integer upperBound) {
        final ValueRange range = ValueRange.of(lowerBound, upperBound - 1);
        return range.isValidIntValue(number);
    }
} 
```

正如我们在上面看到的，我们通过将`lowerBound`和`upperBound`传递给`static` `of()`方法来创建`ValueRange`对象。然后，我们通过使用每个对象的`isValidIntValue()` 方法来检查`number`是否存在于每个范围内。

需要注意的是 **`ValueRange`只支持开箱即用**的封闭范围检查。因此，**我们必须通过增加`lowerBound`来验证左开范围，通过减少`upperBound`** 来验证右开范围，如上所述。

### 4.2.使用 Apache Commons

让我们继续讨论一些我们可以从第三方库中使用的范围类。首先，我们将把 [Apache Commons](https://web.archive.org/web/20220921090117/https://search.maven.org/search?q=g:org.apache.commons%20a:commons-lang3) 依赖项添加到我们的项目中:

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

这里，我们实现了和以前一样的行为，但是使用了 Apache Commons `Range`类:

```
public class IntRangeApacheCommons {

    public boolean isInClosedRange(Integer number, Integer lowerBound, Integer upperBound) {
        final Range<Integer> range = Range.between(lowerBound, upperBound);
        return range.contains(number);
    }

    public boolean isInOpenRange(Integer number, Integer lowerBound, Integer upperBound) {
        final Range<Integer> range = Range.between(lowerBound + 1, upperBound - 1);
        return range.contains(number);
    }

    public boolean isInOpenClosedRange(Integer number, Integer lowerBound, Integer upperBound) {
        final Range<Integer> range = Range.between(lowerBound + 1, upperBound);
        return range.contains(number);
    }

    public boolean isInClosedOpenRange(Integer number, Integer lowerBound, Integer upperBound) {
        final Range<Integer> range = Range.between(lowerBound, upperBound - 1);
        return range.contains(number);
    }
}
```

与`ValueRange`的`of()`方法一样，我们将`lowerBound` 和`upperBound`传递给`Range`的`static between()`方法来创建`Range`对象。然后我们使用`contains()`方法来检查`number`是否存在于每个对象的范围内。

**Apache Commons`Range`类也只支持封闭区间**，但是我们简单地调整了`lowerBound` 和`upperBound`，就像我们调整`ValueRange`一样。

此外，作为一个泛型类，`Range`不仅可以用于 `Integer`，还可以用于任何其他实现了`Comparable.`的类型

### 4.3.使用谷歌番石榴

最后，让我们将 [Google Guava](https://web.archive.org/web/20220921090117/https://search.maven.org/search?q=g:com.google.guava%20a:guava) 依赖项添加到我们的项目中:

```
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.1-jre</version>
</dependency>
```

我们可以使用 Guava 的`Range`类重新实现与之前相同的行为:

```
public class IntRangeGoogleGuava {

    public boolean isInClosedRange(Integer number, Integer lowerBound, Integer upperBound) {
        final Range<Integer> range = Range.closed(lowerBound, upperBound);
        return range.contains(number);
    }

    public boolean isInOpenRange(Integer number, Integer lowerBound, Integer upperBound) {
        final Range<Integer> range = Range.open(lowerBound, upperBound);
        return range.contains(number);
    }

    public boolean isInOpenClosedRange(Integer number, Integer lowerBound, Integer upperBound) {
        final Range<Integer> range = Range.openClosed(lowerBound, upperBound);
        return range.contains(number);
    }

    public boolean isInClosedOpenRange(Integer number, Integer lowerBound, Integer upperBound) {
        final Range<Integer> range = Range.closedOpen(lowerBound, upperBound);
        return range.contains(number);
    }
}
```

我们可以在上面看到，Guava 的`Range`类有四个单独的方法来创建我们之前讨论过的每个范围类型。也就是说，与我们迄今为止看到的其他 range 类不同， **Guava 的`Range`类本身支持开放和半开范围**。例如，为了指定一个不包括其上限的半开区间，我们将`lowerBound` 和`upperBound`传递给了`static` `closedOpen()`方法。对于排除其下界的半开区间，我们用`openClosed()`。然后我们使用`contains()` 方法检查`number`是否存在于每个范围中。

## 5.结论

在本文中，我们学习了如何使用基本操作符和 range 类来检查一个整数是否在给定的范围内。我们还探讨了各种方法的优缺点。

和往常一样，这些例子的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220921090117/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-5)