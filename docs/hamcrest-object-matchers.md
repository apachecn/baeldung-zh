# Hamcrest 对象匹配器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hamcrest-object-matchers>

## 1。概述

Hamcrest 提供了匹配器，使单元测试断言更简单、更易读。你可以在这里开始探索一些可用的匹配器[。](/web/20221128050352/https://www.baeldung.com/java-junit-hamcrest-guide)

在这个快速教程中，我们将深入研究对象匹配器。

## 2。设置

要获得 Hamcrest，我们只需要**将下面的 Maven 依赖项添加到我们的`pom.xml`** 中:

```java
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>java-hamcrest</artifactId>
    <version>2.0.0.0</version>
    <scope>test</scope>
</dependency>
```

最新的 Hamcrest 版本可以在 [Maven Central](https://web.archive.org/web/20221128050352/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22java-hamcrest%22) 上找到。

## 3。对象匹配器

**对象匹配器用于检查对象的属性**。

在研究匹配器之前，我们将创建几个 beans 来使示例简单易懂。

我们的第一个对象叫做`Location`，没有属性:

```java
public class Location {}
```

我们将第二个 bean 命名为`City`,并向其添加以下实现:

```java
public class City extends Location {

    String name;
    String state;

    // standard constructor, getters and setters

    @Override
    public String toString() {
        if (this.name == null && this.state == null) {
            return null;
        }
        StringBuilder sb = new StringBuilder();
        sb.append("[");
        sb.append("Name: ");
        sb.append(this.name);
        sb.append(", ");
        sb.append("State: ");
        sb.append(this.state);
        sb.append("]");
        return sb.toString();
    }
}
```

注意`City` 扩展了`Location`。我们以后会用到它。现在，让我们从对象匹配器开始！

### 3.1。`hasToString`

顾名思义， **`hasToString`方法验证某个对象有一个返回特定`String`** 的`toString`方法:

```java
@Test
public void givenACity_whenHasToString_thenCorrect() {
    City city = new City("San Francisco", "CA");

    assertThat(city, hasToString("[Name: San Francisco, State: CA]"));
}
```

因此，我们正在创建一个`City` ，并验证它的`toString`方法是否返回我们想要的`String`。我们可以更进一步，不检查相等性，而是检查一些其他条件:

```java
@Test
public void givenACity_whenHasToStringEqualToIgnoringCase_thenCorrect() {
    City city = new City("San Francisco", "CA");

    assertThat(city, hasToString(
      equalToIgnoringCase("[NAME: SAN FRANCISCO, STATE: CA]")));
}
```

正如我们所看到的， **`hasToString` 是重载的，可以接收一个`String` 或者一个文本匹配器作为参数**。所以，我们也可以做这样的事情:

```java
@Test
public void givenACity_whenHasToStringEmptyOrNullString_thenCorrect() {
    City city = new City(null, null);

    assertThat(city, hasToString(emptyOrNullString()));
}
```

你可以在这里找到更多关于文本匹配器的信息。现在让我们转到下一个对象匹配器。

### 3.2。`typeCompatibleWith`

这个匹配器**代表一个`is-a` 关系**。我们的`Location` 超类开始发挥作用了:

```java
@Test
public void givenACity_whenTypeCompatibleWithLocation_thenCorrect() {
    City city = new City("San Francisco", "CA");

    assertThat(city.getClass(), is(typeCompatibleWith(Location.class)));
}
```

这是说`City`是-a `Location,` ，这是真的，这个测试应该通过。同样，如果我们想测试否定的情况:

```java
@Test
public void givenACity_whenTypeNotCompatibleWithString_thenCorrect() {
    City city = new City("San Francisco", "CA");

    assertThat(city.getClass(), is(not(typeCompatibleWith(String.class))));
}
```

当然，我们的`City` 班不是`String.`

最后，注意所有的 Java 对象都应该通过下面的测试:

```java
@Test
public void givenACity_whenTypeCompatibleWithObject_thenCorrect() {
    City city = new City("San Francisco", "CA");

    assertThat(city.getClass(), is(typeCompatibleWith(Object.class)));
}
```

请记住**匹配器`is`由另一个匹配器的包装器组成，目的是使整个断言更具可读性。**

## 4。结论

Hamcrest 提供了一种简单而干净的方式来创建断言。有各种各样的匹配器，使每个开发人员的生活更简单，每个项目更可读。

和对象匹配器绝对是检查类属性的一种简单方法。

和往常一样，你会在 GitHub 项目的[中找到完整的实现。](https://web.archive.org/web/20221128050352/https://github.com/eugenp/tutorials/tree/master/testing-modules/hamcrest)