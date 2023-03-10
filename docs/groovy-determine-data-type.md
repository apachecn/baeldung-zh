# 如何确定 Groovy 中的数据类型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-determine-data-type>

## 1.介绍

在这个快速教程中，我们将探索在 Groovy 中查找数据类型的不同方法。

实际上，这取决于我们在做什么:

*   首先，我们将看看如何处理原语
*   然后，我们将看到集合如何带来一些独特的挑战
*   最后，我们将看看对象和类变量

## 2.原始类型

Groovy 支持与 Java 相同数量的原语类型。**我们可以通过三种方式找到原语的数据类型**。

首先，让我们想象我们对一个人的年龄有多种表述。

首先，我们从`instanceof` 运算符开始:

```java
@Test
public void givenWhenParameterTypeIsInteger_thenReturnTrue() {
    Person personObj = new Person(10)
    Assert.assertTrue(personObj.ageAsInt instanceof Integer);
}
```

`[instanceof](/web/20220625165604/https://www.baeldung.com/java-instanceof)` 是一个二元操作符，我们可以用它来检查一个对象是否是给定类型的实例。如果对象是特定类型的实例，它返回`true` ，否则返回`false` 。

另外， **Groovy 3 增加了新的`!instanceof`操作符**。如果对象不是一个类型的实例，它返回`true` ，否则返回`false` 。

然后，我们也可以从 Object 类中使用`getClass()`方法。它返回实例的运行时类:

```java
@Test
public void givenWhenParameterTypeIsDouble_thenReturnTrue() {
    Person personObj = new Person(10.0)
    Assert.assertTrue((personObj.ageAsDouble).getClass() == Double)
}
```

最后，让我们应用。`class` 运算符查找数据类型:

```java
@Test
public void givenWhenParameterTypeIsString_thenReturnTrue() {
    Person personObj = new Person("10 years")
    Assert.assertTrue(personObj.ageAsString.class == String)
}
```

同样，我们可以找到任何原始类型的数据类型。

## 3.收集

Groovy 支持各种集合类型。

让我们在 Groovy 中定义一个简单的列表:

```java
@Test
public void givenGroovyList_WhenFindClassName_thenReturnTrue() {
    def ageList = ['ageAsString','ageAsDouble', 10]
    Assert.assertTrue(ageList.class == ArrayList)
    Assert.assertTrue(ageList.getClass() == ArrayList)
}
```

但是在地图上，不能应用`.class`运算符:

```java
@Test
public void givenGrooyMap_WhenFindClassName_thenReturnTrue() {
    def ageMap = [ageAsString: '10 years', ageAsDouble: 10.0]
    Assert.assertFalse(ageMap.class == LinkedHashMap)
}
```

在上面的代码片段中， **`ageMap.class`将尝试从给定的映射**中检索 key 类的值。**对于地图，建议应用`getClass()`** **而不是`.class`** 。

## 4.对象和类变量

在上面的部分中，我们使用了各种策略来查找原语和集合的数据类型。

为了了解类变量是如何工作的，让我们假设我们有一个类`Person`:

```java
@Test
public void givenClassName_WhenParameterIsInteger_thenReturnTrue() {
    Assert.assertTrue(Person.class.getDeclaredField('ageAsInt').type == int.class)
}
```

记住 [`getDeclaredField()`](/web/20220625165604/https://www.baeldung.com/java-reflection-class-fields) 返回某个类的所有字段。

我们可以使用`instanceof, getClass()` 和 `.class` 运算符找到任何对象的类型:

```java
@Test
public void givenWhenObjectIsInstanceOfType_thenReturnTrue() {
    Person personObj = new Person()
    Assert.assertTrue(personObj instanceof Person)
}
```

此外，我们还可以使用 Groovy 成员操作符`in`:

```java
@Test
public void givenWhenInstanceIsOfSubtype_thenReturnTrue() {
    Student studentObj = new Student()
    Assert.assertTrue(studentObj in Person)
}
```

## 5.结论

在这篇短文中，我们看到了如何在 Groovy 中找到数据类型。相比之下， `getClass()`方法比`.class`操作符更安全。我们还讨论了`in` 操作员和`instanceof` 操作员的工作。此外，我们还学习了如何获取一个类的所有字段并应用`.type` 操作符。

和往常一样，所有的代码片段都可以在 GitHub 上找到[。](https://web.archive.org/web/20220625165604/https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy-2)