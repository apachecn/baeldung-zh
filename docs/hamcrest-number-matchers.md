# 使用 Hamcrest 数字匹配器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hamcrest-number-matchers>

## 1。概述

Hamcrest 提供了静态匹配器，有助于使单元测试断言更简单、更易读。你可以在这里开始探索一些可用的匹配器[。](/web/20221208143845/https://www.baeldung.com/java-junit-hamcrest-guide)

在本文中，我们将深入研究与数字相关的匹配器。

## 2。设置

要获得 Hamcrest，我们只需将以下 Maven 依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>java-hamcrest</artifactId>
    <version>2.0.0.0</version>
</dependency>
```

最新的 Hamcrest 版本可以在 [Maven Central](https://web.archive.org/web/20221208143845/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22java-hamcrest%22) 上找到。

## 3。邻近匹配器

我们要看的第一组匹配器是那些**检查某个元素是否接近一个值+/-一个误差**的匹配器。

更正式地说:

```java
value - error <= element <= value + error
```

如果上面的比较为真，断言将通过。

让我们看看它的实际效果吧！

### 3.1。`isClose` 带有`Double`值

假设我们在一个名为`actual.` 的双变量中存储了一个数字，我们想测试一下`actual` 是否接近 1 +/- 0.5。

那就是:

```java
1 - 0.5 <= actual <= 1 + 0.5
    0.5 <= actual <= 1.5
```

现在让我们使用`isClose` 匹配器创建一个单元测试:

```java
@Test
public void givenADouble_whenCloseTo_thenCorrect() {
    double actual = 1.3;
    double operand = 1;
    double error = 0.5;

    assertThat(actual, closeTo(operand, error));
}
```

由于 1.3 介于 0.5 和 1.5 之间，测试将通过。同样，我们可以测试负面场景:

```java
@Test
public void givenADouble_whenNotCloseTo_thenCorrect() {
    double actual = 1.6;
    double operand = 1;
    double error = 0.5;

    assertThat(actual, not(closeTo(operand, error)));
}
```

现在，让我们看看一个类似的情况，不同类型的变量。

### 3.2。`isClose` 带有`BigDecimal`值

**`isClose` 是重载的，可以和 double 值一样使用，但是和`BigDecimal`对象**一起使用:

```java
@Test
public void givenABigDecimal_whenCloseTo_thenCorrect() {
    BigDecimal actual = new BigDecimal("1.0003");
    BigDecimal operand = new BigDecimal("1");
    BigDecimal error = new BigDecimal("0.0005");

    assertThat(actual, is(closeTo(operand, error)));
}

@Test
public void givenABigDecimal_whenNotCloseTo_thenCorrect() {
    BigDecimal actual = new BigDecimal("1.0006");
    BigDecimal operand = new BigDecimal("1");
    BigDecimal error = new BigDecimal("0.0005");

    assertThat(actual, is(not(closeTo(operand, error))));
}
```

请注意**`is`匹配器只修饰其他匹配器，不增加额外的逻辑**。它只是让整个断言更具可读性。

接近匹配器大概就是这样。接下来，我们将看看订单匹配器。

## 4。订单匹配器

顾名思义，这些匹配器有助于对订单进行断言。

他们有五个人:

*   `comparesEqualTo`
*   `greaterThan`
*   `greaterThanOrEqualTo`
*   `lessThan`
*   `lessThanOrEqualTo`

它们几乎是不言自明的，但是让我们看一些例子。

### 4.1。带有`Integer V`值的订单匹配器

最常见的场景是**使用这些带有数字**的匹配器。

因此，让我们继续创建一些测试:

```java
@Test
public void given5_whenComparesEqualTo5_thenCorrect() {
    Integer five = 5;

    assertThat(five, comparesEqualTo(five));
}

@Test
public void given5_whenNotComparesEqualTo7_thenCorrect() {
    Integer seven = 7;
    Integer five = 5;

    assertThat(five, not(comparesEqualTo(seven)));
}

@Test
public void given7_whenGreaterThan5_thenCorrect() {
    Integer seven = 7;
    Integer five = 5;

    assertThat(seven, is(greaterThan(five)));
}

@Test
public void given7_whenGreaterThanOrEqualTo5_thenCorrect() {
    Integer seven = 7;
    Integer five = 5;

    assertThat(seven, is(greaterThanOrEqualTo(five)));
}

@Test
public void given5_whenGreaterThanOrEqualTo5_thenCorrect() {
    Integer five = 5;

    assertThat(five, is(greaterThanOrEqualTo(five)));
}

@Test
public void given3_whenLessThan5_thenCorrect() {
   Integer three = 3;
   Integer five = 5;

   assertThat(three, is(lessThan(five)));
}

@Test
public void given3_whenLessThanOrEqualTo5_thenCorrect() {
   Integer three = 3;
   Integer five = 5;

   assertThat(three, is(lessThanOrEqualTo(five)));
}

@Test
public void given5_whenLessThanOrEqualTo5_thenCorrect() {
   Integer five = 5;

   assertThat(five, is(lessThanOrEqualTo(five)));
}
```

有道理，对吧？请注意理解谓词断言的内容是多么简单。

### 4.2。具有`String` 值的订单匹配器

尽管比较数字很有意义，但很多时候比较其他类型的元素也很有用。这就是为什么**订单匹配器可以应用于任何实现`Comparable` 接口**的类。

我们来看一些`Strings:`的例子

```java
@Test
public void givenBenjamin_whenGreaterThanAmanda_thenCorrect() {
    String amanda = "Amanda";
    String benjamin = "Benjamin";

    assertThat(benjamin, is(greaterThan(amanda)));
}

@Test
public void givenAmanda_whenLessThanBenajmin_thenCorrect() {
    String amanda = "Amanda";
    String benjamin = "Benjamin";

    assertThat(amanda, is(lessThan(benjamin)));
}
```

`String`在`Comparable` 接口的`compareTo` 方法中实现字母顺序。

所以，“阿曼达”这个词出现在“本杰明”这个词之前是有道理的。

### 4.3。具有`LocalDate` 值的订单匹配器

和`Strings`一样，我们可以比较日期。让我们看看上面创建的相同示例，但是使用了`LocalDate` 对象:

```java
@Test
public void givenToday_whenGreaterThanYesterday_thenCorrect() {
    LocalDate today = LocalDate.now();
    LocalDate yesterday = today.minusDays(1);

    assertThat(today, is(greaterThan(yesterday)));
}

@Test
public void givenToday_whenLessThanTomorrow_thenCorrect() {
    LocalDate today = LocalDate.now();
    LocalDate tomorrow = today.plusDays(1);

    assertThat(today, is(lessThan(tomorrow)));
}
```

很高兴看到语句`assertThat(today, is(lessThan(tomorrow)))` 接近常规英语。

### 4.4。订单匹配器与定制类*es*

那么，为什么不创建我们自己的类，并以这种方式实现`Comparable?`，**我们可以利用订单匹配器来使用定制订单规则**。

让我们从创建一个`Person` bean 开始:

```java
public class Person {
    String name;
    int age;

    // standard constructor, getters and setters
}
```

现在，让我们实现`Comparable`:

```java
public class Person implements Comparable<Person> {

    // ...

    @Override
    public int compareTo(Person o) {
        if (this.age == o.getAge()) return 0;
        if (this.age > o.getAge()) return 1;
        else return -1;
    }
}
```

我们的`compareTo` 实现通过年龄来比较两个人。现在让我们创建几个新的测试:

```java
@Test
public void givenAmanda_whenOlderThanBenjamin_thenCorrect() {
    Person amanda = new Person("Amanda", 20);
    Person benjamin = new Person("Benjamin", 18);

    assertThat(amanda, is(greaterThan(benjamin)));
}

@Test
public void 
givenBenjamin_whenYoungerThanAmanda_thenCorrect() {
    Person amanda = new Person("Amanda", 20);
    Person benjamin = new Person("Benjamin", 18);

    assertThat(benjamin, is(lessThan(amanda)));
}
```

匹配器现在将基于我们的`compareTo` 逻辑工作。

## 5 号。南匹配

Hamcrest 提供了一个额外的数字匹配器来定义一个数字实际上是不是一个数字:

```java
@Test
public void givenNaN_whenIsNotANumber_thenCorrect() {
    double zero = 0d;

    assertThat(zero / zero, is(notANumber()));
}
```

## 6。结论

如您所见，**数字匹配器对于简化常见断言**非常有用。

更重要的是，Hamcrest matchers 一般来说是**自明的，并且容易阅读**。

所有这些，加上将匹配器与定制比较逻辑相结合的能力，使它们成为大多数项目的强大工具。

本文示例的完整实现可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143845/https://github.com/eugenp/tutorials/tree/master/testing-modules/hamcrest)