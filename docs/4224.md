# Hamcrest 配豆器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hamcrest-bean-matchers>

## 1。概述

Hamcrest 是一个库，它提供了称为匹配器的方法，帮助开发人员编写更简单的单元测试。有很多匹配器，你可以通过阅读其中的一些来开始[这里](/web/20220628155347/https://www.baeldung.com/java-junit-hamcrest-guide)。

在本文中，我们将探索 beans 匹配器。

## 2。设置

要获得 Hamcrest，我们只需要**将下面的 Maven 依赖项添加到我们的`pom.xml`** 中:

```
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>java-hamcrest</artifactId>
    <version>2.0.0.0</version>
    <scope>test</scope>
</dependency>
```

最新的 Hamcrest 版本可以在 [Maven Central](https://web.archive.org/web/20220628155347/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22java-hamcrest%22) 上找到。

## 3。豆子匹配器

Bean 匹配器**对于检查 POJO**上的条件非常有用，这是编写大多数单元测试时经常需要的。

在开始之前，我们将创建一个类来帮助我们完成示例:

```
public class City {
    String name;
    String state;

    // standard constructor, getters and setters

}
```

现在我们都准备好了，让我们看看豆子匹配器的运行情况！

### 3.1。`hasProperty`

这个匹配器基本上是用来**检查某个 bean 是否包含由属性名**标识的特定属性

```
@Test
public void givenACity_whenHasProperty_thenCorrect() {
    City city = new City("San Francisco", "CA");

    assertThat(city, hasProperty("state"));
}
```

所以，这个测试会通过，因为我们的`City` bean 有一个名为`state.`的属性

按照这种想法，我们也可以测试一个 bean 是否具有某种属性，并且该属性具有某种值:

```
@Test
public void givenACity_whenHasPropertyWithValueEqualTo_thenCorrect() {
    City city = new City("San Francisco", "CA");

    assertThat(city, hasProperty("name", equalTo("San Francisco")));
}
```

正如我们看到的， **`hasProperty`是重载的，可以和第二个匹配器一起使用来检查属性的特定条件。**

所以，我们也可以这样做:

```
@Test
public void givenACity_whenHasPropertyWithValueEqualToIgnoringCase_thenCorrect() {
    City city = new City("San Francisco", "CA");

    assertThat(city, hasProperty("state", equalToIgnoringCase("ca")));
}
```

有用吧？我们可以用接下来要探讨的 matcher 将这个想法更进一步。

### 3.2。`samePropertyValuesAs`

有时**当我们必须检查一个 bean 的许多属性时，用期望的值**创建一个新的 bean 可能更简单。然后，我们可以检查测试的 bean 和新的 bean 之间是否相等。当然，Hamcrest 为这种情况提供了一个匹配器:

```
@Test
public void givenACity_whenSamePropertyValuesAs_thenCorrect() {
    City city = new City("San Francisco", "CA");
    City city2 = new City("San Francisco", "CA");

    assertThat(city, samePropertyValuesAs(city2));
}
```

这导致更少的断言和更简单的代码。同样，我们可以测试否定的情况:

```
@Test
public void givenACity_whenNotSamePropertyValuesAs_thenCorrect() {
    City city = new City("San Francisco", "CA");
    City city2 = new City("Los Angeles", "CA");

    assertThat(city, not(samePropertyValuesAs(city2)));
}
```

接下来，我们将看到几个检查类属性的 util 方法。

### 3.3。`getPropertyDescriptor`

在某些情况下，探索一个职业结构可能会派上用场。 Hamcrest 为此提供了一些实用方法:

```
@Test
public void givenACity_whenGetPropertyDescriptor_thenCorrect() {
    City city = new City("San Francisco", "CA");
    PropertyDescriptor descriptor = getPropertyDescriptor("state", city);

    assertThat(descriptor
      .getReadMethod()
      .getName(), is(equalTo("getState")));
}
```

**对象`descriptor` 检索关于属性`state`的大量信息。**在这种情况下，我们提取了 getter 的名称，并断言它等于某个期望值。注意，我们也可以应用其他文本匹配器。

接下来，我们将探讨的最后一种方法是这种想法的更一般的情况。

### 3.4。`propertyDescriptorsFor`

除了 bean 的所有属性之外，这个方法的**基本上与上一节中的方法相同。我们还需要指定我们希望在类层次结构中达到的高度:**

```
@Test
public void givenACity_whenGetPropertyDescriptorsFor_thenCorrect() {
    City city = new City("San Francisco", "CA");
    PropertyDescriptor[] descriptors = propertyDescriptorsFor(
      city, Object.class);

    List<String> getters = Arrays.stream(descriptors)
      .map(x -> x.getReadMethod().getName())
      .collect(toList());

    assertThat(getters, containsInAnyOrder("getName", "getState"));
}
```

因此，我们在这里做的是:从 bean `city` 中获取所有属性描述符，并在`Object`级别停止。

然后，我们只是使用 Java 8 的特性来过滤 getter 方法。

最后，我们使用集合匹配器来检查`getters` 列表中的一些内容。你可以在这里找到更多关于收藏匹配器[的信息。](/web/20220628155347/https://www.baeldung.com/hamcrest-collections-arrays)

## 4。结论

Hamcrest matchers 包含了一套适用于每个项目的工具。它们很容易学习，并且在短时间内变得非常有用。

尤其是 Beans 匹配器，它提供了一种有效的方式来对 POJOs 进行断言，这是编写单元测试时经常需要的。

要获得这个例子的完整实现，请参考[GitHub 项目](https://web.archive.org/web/20220628155347/https://github.com/eugenp/tutorials/tree/master/testing-modules/hamcrest)。