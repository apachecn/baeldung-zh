# Mockito 和流畅的 API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mockito-fluent-apis>

## 1。简介

Fluent APIs 是一种基于方法链的软件工程设计技术，用于构建简洁、可读和有说服力的界面。

它们通常用于建筑商、工厂和其他创造性的设计模式。最近，随着 [Java](/web/20220715043119/https://www.baeldung.com/java-8-new-features) 的发展，它们变得越来越流行，并且可以在流行的 API 中找到，例如 [Java 流 API](/web/20220715043119/https://www.baeldung.com/java-8-streams-introduction) 和 [Mockito](/web/20220715043119/https://www.baeldung.com/tag/mockito/) 测试框架。

尽管如此，模仿流畅的 API 可能会很痛苦，因为我们经常需要建立一个复杂的模仿对象层次结构。

在本教程中，我们将看看如何使用 Mockito 的一个强大功能来避免这种情况。

## 2。一个简单流畅的 API

在本教程中，我们将使用构建器设计模式来演示一个简单的 fluent API 来构建一个 pizza 对象:

```java
Pizza pizza = new Pizza
  .PizzaBuilder("Margherita")
  .size(PizzaSize.LARGE)
  .withExtaTopping("Mushroom")
  .withStuffedCrust(false)
  .willCollect(true)
  .applyDiscount(20)
  .build();
```

正如我们所见，我们已经创建了一个易于理解的 API，它读起来像一个 [DSL](/web/20220715043119/https://www.baeldung.com/spring-integration-java-dsl) ，并允许我们创建一个具有各种特性的`Pizza`对象。

现在，我们将定义一个使用我们的构建器的简单服务类。这将是我们稍后要测试的类:

```java
public class PizzaService {

    private Pizza.PizzaBuilder builder;

    public PizzaService(Pizza.PizzaBuilder builder) {
        this.builder = builder;
    }

    public Pizza orderHouseSpecial() {
        return builder.name("Special")
          .size(PizzaSize.LARGE)
          .withExtraTopping("Mushrooms")
          .withStuffedCrust(true)
          .withExtraTopping("Chilli")
          .willCollect(true)
          .applyDiscount(20)
          .build();
    }
}
```

我们的服务非常简单，包含一个名为`orderHouseSpecial`的方法。顾名思义，我们可以使用这种方法构建一个具有一些预定义属性的特殊披萨。

## 3。传统嘲讽

以传统方式使用 mock**需要我们创建八个 mock `PizzaBuilder`对象**。我们需要对由`name` 方法返回的`PizzaBuilder`进行模拟，然后对由`size`方法返回的`PizzaBuilder`进行模拟，等等。我们将以这种方式继续下去，直到我们满足了我们的 fluent API 链中的所有方法调用。

现在让我们看看如何使用传统的模拟[模拟](/web/20220715043119/https://www.baeldung.com/mockito-mock-methods)编写单元测试来测试我们的服务方法:

```java
@Test
public void givenTraditonalMocking_whenServiceInvoked_thenPizzaIsBuilt() {
    PizzaBuilder nameBuilder = Mockito.mock(Pizza.PizzaBuilder.class);
    PizzaBuilder sizeBuilder = Mockito.mock(Pizza.PizzaBuilder.class);
    PizzaBuilder firstToppingBuilder = Mockito.mock(Pizza.PizzaBuilder.class);
    PizzaBuilder secondToppingBuilder = Mockito.mock(Pizza.PizzaBuilder.class);
    PizzaBuilder stuffedBuilder = Mockito.mock(Pizza.PizzaBuilder.class);
    PizzaBuilder willCollectBuilder = Mockito.mock(Pizza.PizzaBuilder.class);
    PizzaBuilder discountBuilder = Mockito.mock(Pizza.PizzaBuilder.class);

    PizzaBuilder builder = Mockito.mock(Pizza.PizzaBuilder.class);
    when(builder.name(anyString())).thenReturn(nameBuilder);
    when(nameBuilder.size(any(Pizza.PizzaSize.class))).thenReturn(sizeBuilder);        
    when(sizeBuilder.withExtraTopping(anyString())).thenReturn(firstToppingBuilder);
    when(firstToppingBuilder.withStuffedCrust(anyBoolean())).thenReturn(stuffedBuilder);
    when(stuffedBuilder.withExtraTopping(anyString())).thenReturn(secondToppingBuilder);
    when(secondToppingBuilder.willCollect(anyBoolean())).thenReturn(willCollectBuilder);
    when(willCollectBuilder.applyDiscount(anyInt())).thenReturn(discountBuilder);
    when(discountBuilder.build()).thenReturn(expectedPizza);

    PizzaService service = new PizzaService(builder);
    Pizza pizza = service.orderHouseSpecial();
    assertEquals("Expected Pizza", expectedPizza, pizza);

    verify(builder).name(stringCaptor.capture());
    assertEquals("Pizza name: ", "Special", stringCaptor.getValue());

    // rest of test verification
} 
```

在这个例子中，我们需要模拟提供给`PizzaService`的`PizzaBuilder`。正如我们所看到的，这不是一个简单的任务，因为我们需要返回一个 mock，它将为我们的 fluent API 中的每个调用返回一个 mock。

这导致了一个复杂的模仿对象层次结构，很难理解，也很难维护。

## 4。深度拔救

**谢天谢地，Mockito 提供了一个名为`deep stubbing`的非常棒的特性，它允许我们在创建模拟**时指定一个`[Answer](/web/20220715043119/https://www.baeldung.com/mockito-behavior)`模式。

要创建一个深存根，我们只需在创建 mock 时添加`Mockito.RETURNS_DEEP_STUBS`常量作为附加参数:

```java
@Test
public void givenDeepMocks_whenServiceInvoked_thenPizzaIsBuilt() {
    PizzaBuilder builder = Mockito.mock(Pizza.PizzaBuilder.class, Mockito.RETURNS_DEEP_STUBS);

    Mockito.when(builder.name(anyString())
      .size(any(Pizza.PizzaSize.class))
      .withExtraTopping(anyString())
      .withStuffedCrust(anyBoolean())
      .withExtraTopping(anyString())
      .willCollect(anyBoolean())
      .applyDiscount(anyInt())
      .build())
      .thenReturn(expectedPizza);

    PizzaService service = new PizzaService(builder);
    Pizza pizza = service.orderHouseSpecial();
    assertEquals("Expected Pizza", expectedPizza, pizza);
} 
```

通过使用`Mockito.RETURNS_DEEP_STUBS`参数，我们告诉 Mockito 做一种深度嘲弄。这使得模拟完整方法链的结果成为可能，或者在我们的例子中，一次性模拟 fluent API 的结果。

这导致了一个更优雅的解决方案和一个比我们在前一节看到的更容易理解的测试。本质上，我们避免了创建复杂的模拟对象层次结构的需要。

我们也可以通过`@Mock`注释直接使用这种回答模式:

```java
@Mock(answer = Answers.RETURNS_DEEP_STUBS)
private PizzaBuilder anotherBuilder;
```

需要注意的一点是，验证只适用于链中的最后一个 mock。

## 5。结论

在这个快速教程中，我们已经看到了如何使用 Mockito 来模拟一个简单流畅的 API。首先，我们看了传统的模仿方法，并理解了这种方法的困难。

然后我们看了一个例子，它使用了 Mockito 的一个鲜为人知的特性，叫做 deep stubs，它允许以一种更优雅的方式来模仿我们流畅的 API。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220715043119/https://github.com/eugenp/tutorials/tree/master/testing-modules/mockito-2)