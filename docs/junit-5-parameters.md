# 将参数注入 JUnit Jupiter 单元测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-5-parameters>

## 1.**概述**

在 JUnit 5 之前，为了引入一个很酷的新特性，JUnit 团队必须对核心 API 进行改进。对于 JUnit 5，团队决定是时候在 JUnit 本身之外扩展核心 JUnit API 的功能了，这是 JUnit 5 的一个核心理念，叫做“[比起特性](https://web.archive.org/web/20220630013527/https://github.com/junit-team/junit5/wiki/Core-Principles)更喜欢扩展点”。

在本文中，我们将关注其中一个扩展点接口——`ParameterResolver` ——您可以使用它将参数注入到您的测试方法中。有几种不同的方法可以让 JUnit 平台知道您的扩展(这个过程称为“注册”)，在本文中，我们将重点关注*声明式*注册(即通过源代码注册)。

## 2.**T2`ParameterResolver`**

可以使用 JUnit 4 API 将参数注入到测试方法中，但是这相当有限。通过 JUnit 5，可以扩展 Jupiter API 通过实现`ParameterResolver` ——为您的测试方法提供任何类型的对象。让我们看一看。

### 2.1。`FooParameterResolver`

```java
public class FooParameterResolver implements ParameterResolver {
  @Override
  public boolean supportsParameter(ParameterContext parameterContext, 
    ExtensionContext extensionContext) throws ParameterResolutionException {
      return parameterContext.getParameter().getType() == Foo.class;
  }

  @Override
  public Object resolveParameter(ParameterContext parameterContext, 
    ExtensionContext extensionContext) throws ParameterResolutionException {
      return new Foo();
  }
} 
```

首先，我们需要实现有两个方法的`ParameterResolver –`:

*   `supportsParameter()`–如果参数的类型受支持(本例中为 Foo)，则返回 true，并且
*   `resolveParamater()`–提供一个正确类型的对象(在这个例子中是一个新的 Foo 实例)，它将被注入到您的测试方法中

### 2.2。`FooTest`

```java
@ExtendWith(FooParameterResolver.class)
public class FooTest {
    @Test
    public void testIt(Foo fooInstance) {
        // TEST CODE GOES HERE
    }  
} 
```

然后要使用这个扩展，我们需要通过`@ExtendWith`注释(第 1 行)声明它——也就是告诉 JUnit 平台。

当 JUnit 平台运行您的单元测试时，它将从`FooParameterResolver`获得一个`Foo` 实例，并将其传递给`testIt()`方法(第 4 行)。

扩展有一个`scope of influence`，它激活扩展，这取决于**在哪里声明了**。

扩展可以在以下位置激活:

*   方法级别，它仅对该方法有效，或者
*   类级别，它在整个测试类中是活动的，或者我们很快会看到的`@Nested`测试类

`Note: you should not declare a` 参数解析器 `at both scopes **for the same parameter type**, or the JUnit Platform will complain about this ambiguity`。

对于本文，我们将看到如何编写和使用两个扩展来注入`Person`对象:一个注入“好的”数据(称为`ValidPersonParameterResolver`)和一个注入“坏的”数据(`InvalidPersonParameterResolver`)。我们将使用这些数据对一个名为`PersonValidator`的类进行单元测试，它验证了一个`Person`对象的状态。

## 3.**编写扩展**

既然我们理解了什么是`ParameterResolver`扩展，我们就准备写:

*   一个提供**有效的** `Person`对象(`ValidPersonParameterResolver`，以及
*   其中提供了**无效的** `Person`对象(`InvalidPersonParameterResolver`

### 3.1。`ValidPersonParameterResolver`

```java
public class ValidPersonParameterResolver implements ParameterResolver {

  public static Person[] VALID_PERSONS = {
      new Person().setId(1L).setLastName("Adams").setFirstName("Jill"),
      new Person().setId(2L).setLastName("Baker").setFirstName("James"),
      new Person().setId(3L).setLastName("Carter").setFirstName("Samanta"),
      new Person().setId(4L).setLastName("Daniels").setFirstName("Joseph"),
      new Person().setId(5L).setLastName("English").setFirstName("Jane"),
      new Person().setId(6L).setLastName("Fontana").setFirstName("Enrique"),
  }; 
```

注意`Person`对象的`VALID_PERSONS`数组。这是有效的`Person`对象的存储库，每次 JUnit 平台调用`resolveParameter()`方法时都会从中随机选择一个对象。

这里的有效 Person 对象完成了两件事:

1.  分离单元测试和驱动它的数据之间的关注点
2.  重用，如果其他单元测试需要有效的`Person`对象来驱动它们

```java
@Override
public boolean supportsParameter(ParameterContext parameterContext, 
  ExtensionContext extensionContext) throws ParameterResolutionException {
    boolean ret = false;
    if (parameterContext.getParameter().getType() == Person.class) {
        ret = true;
    }
    return ret;
} 
```

如果参数的类型是`Person`，那么扩展告诉 JUnit 平台它支持该参数类型，否则返回 false，表示不支持。

这有什么关系？虽然本文中的例子很简单，但是在现实世界的应用程序中，单元测试类可能非常庞大和复杂，有许多采用不同参数类型的测试方法。在当前影响范围内解析参数**时，JUnit 平台必须检查所有注册的`ParameterResolver`。**

```java
@Override
public Object resolveParameter(ParameterContext parameterContext, 
  ExtensionContext extensionContext) throws ParameterResolutionException {
    Object ret = null;
    if (parameterContext.getParameter().getType() == Person.class) {
        ret = VALID_PERSONS[new Random().nextInt(VALID_PERSONS.length)];
    }
    return ret;
}
```

从`VALID_PERSONS` 数组返回一个随机的`Person`对象。请注意，只有当`supportsParameter()`返回`true`时，JUnit 平台才会调用`resolveParameter()`。

### 3.2。`InvalidPersonParameterResolver`

```java
public class InvalidPersonParameterResolver implements ParameterResolver {
  public static Person[] INVALID_PERSONS = {
      new Person().setId(1L).setLastName("Ad_ams").setFirstName("Jill,"),
      new Person().setId(2L).setLastName(",Baker").setFirstName(""),
      new Person().setId(3L).setLastName(null).setFirstName(null),
      new Person().setId(4L).setLastName("Daniel&").setFirstName("{Joseph}"),
      new Person().setId(5L).setLastName("").setFirstName("English, Jane"),
      new Person()/*.setId(6L).setLastName("Fontana").setFirstName("Enrique")*/,
  }; 
```

注意`Person` 对象的`INVALID_PERSONS`数组。就像`ValidPersonParameterResolver`一样，这个类包含一个“坏的”(即无效的)数据的存储，供单元测试使用，以确保例如`PersonValidator.ValidationExceptions`在无效数据出现时被正确抛出:

```java
@Override
public Object resolveParameter(ParameterContext parameterContext, 
  ExtensionContext extensionContext) throws ParameterResolutionException {
    Object ret = null;
    if (parameterContext.getParameter().getType() == Person.class) {
        ret = INVALID_PERSONS[new Random().nextInt(INVALID_PERSONS.length)];
    }
    return ret;
}

@Override
public boolean supportsParameter(ParameterContext parameterContext, 
  ExtensionContext extensionContext) throws ParameterResolutionException {
    boolean ret = false;
    if (parameterContext.getParameter().getType() == Person.class) {
        ret = true;
    }
    return ret;
} 
```

这个类的其余部分自然表现得和它的“好”对应部分一模一样。

## 4。声明并使用扩展

现在我们有两个`ParameterResolver`了，是时候使用它们了。让我们为`PersonValidator`创建一个名为`PersonValidatorTest`的 JUnit 测试类。

我们将使用仅在 JUnit Jupiter 中提供的几个功能:

*   @`DisplayName`–这是显示在测试报告上的名称，更易于阅读
*   @`Nested`–创建一个嵌套的测试类，完成它自己的测试生命周期，独立于它的父类
*   @`RepeatedTest`–测试重复由 value 属性指定的次数(在每个示例中为 10 次)

通过使用@ `Nested`类，我们能够在同一个测试类中测试有效和无效数据，同时使它们彼此完全隔离:

```java
@DisplayName("Testing PersonValidator")
public class PersonValidatorTest {

    @Nested
    @DisplayName("When using Valid data")
    @ExtendWith(ValidPersonParameterResolver.class)
    public class ValidData {

        @RepeatedTest(value = 10)
        @DisplayName("All first names are valid")
        public void validateFirstName(Person person) {
            try {
                assertTrue(PersonValidator.validateFirstName(person));
            } catch (PersonValidator.ValidationException e) {
                fail("Exception not expected: " + e.getLocalizedMessage());
            }
        }
    }

    @Nested
    @DisplayName("When using Invalid data")
    @ExtendWith(InvalidPersonParameterResolver.class)
    public class InvalidData {

        @RepeatedTest(value = 10)
        @DisplayName("All first names are invalid")
        public void validateFirstName(Person person) {
            assertThrows(
              PersonValidator.ValidationException.class, 
              () -> PersonValidator.validateFirstName(person));
        }
    }
} 
```

请注意我们如何能够在同一个主测试类中使用`ValidPersonParameterResolver` 和`InvalidPersonParameterResolver`扩展——只在@ `Nested`类级别声明它们。用 JUnit 4 试试吧！(剧透提醒:你做不到！)

## 5.结论

在本文中，我们探讨了如何编写两个`ParameterResolver`扩展——提供有效和无效的对象。然后我们看了看如何在单元测试中使用这两个`ParameterResolver`实现。

和往常一样，代码可以在 Github 的[上获得。](https://web.archive.org/web/20220630013527/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5)

而且，如果您想了解更多关于 JUnit Jupiter 扩展模型的信息，请查看我在 developerWorks 上的教程 [JUnit 5 用户指南](https://web.archive.org/web/20220630013527/http://junit.org/junit5/docs/current/user-guide/)或[第 2 部分](https://web.archive.org/web/20220630013527/https://developer.ibm.com/tutorials/j-introducing-junit5-part2-vintage-jupiter-extension-model/)。