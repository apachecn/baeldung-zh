# Java 中的协变返回类型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-covariant-return-type>

## 1.概观

在本教程中，我们将仔细研究 Java 中的协变返回类型。在从返回类型的角度研究协方差之前，让我们看看这意味着什么。

## 2.协方差

当只定义超类型时，协方差可以被认为是子类型如何被接受的契约。

让我们考虑几个协方差的基本例子:

```java
List<? extends Number> integerList = new ArrayList<Integer>();
List<? extends Number> doubleList = new ArrayList<Double>();
```

**所以** **协方差意味着，我们可以访问通过它们的超类型**定义的特定元素。然而，**我们不允许将元素放入协变系统**，因为编译器无法确定泛型结构的实际类型。

## 3.协变返回类型

**协变返回类型是——当我们覆盖一个方法时——允许返回类型成为被覆盖方法**的类型的子类型。

为了实践这一点，让我们用一个简单的`Producer` 类和一个`produce()` 方法`.`，默认情况下，它返回一个`String` 作为`Object` ，为子类提供灵活性:

```java
public class Producer {
    public Object produce(String input) {
        Object result = input.toLowerCase();
        return result;
    }
}
```

由于将`Object `作为返回类型，我们可以在子类中有一个更具体的返回类型。这将是协变返回类型，并将从字符序列中产生数字:

```java
public class IntegerProducer extends Producer {
    @Override
    public Integer produce(String input) {
        return Integer.parseInt(input);
    }
}
```

## 4.结构的用法

协变返回类型背后的主要思想是支持 [Liskov 替换](/web/20221206030049/https://www.baeldung.com/solid-principles#l)。

例如，让我们考虑下面的生产者场景:

```java
@Test
public void whenInputIsArbitrary_thenProducerProducesString() {
    String arbitraryInput = "just a random text";
    Producer producer = new Producer();

    Object objectOutput = producer.produce(arbitraryInput);

    assertEquals(arbitraryInput, objectOutput);
    assertEquals(String.class, objectOutput.getClass());
}
```

更改为`IntegerProducer`后，实际产生结果的业务逻辑可以保持不变:

```java
@Test
public void whenInputIsSupported_thenProducerCreatesInteger() {
    String integerAsString = "42";
    Producer producer = new IntegerProducer();

    Object result = producer.produce(integerAsString);

    assertEquals(Integer.class, result.getClass());
    assertEquals(Integer.parseInt(integerAsString), result);
}
```

然而，我们仍然通过一个`Object.`来引用结果，每当我们开始使用对`IntegerProducer, `的显式引用时，我们可以在不向下转换的情况下将结果作为`Integer`来检索:

```java
@Test
public void whenInputIsSupported_thenIntegerProducerCreatesIntegerWithoutCasting() {
    String integerAsString = "42";
    IntegerProducer producer = new IntegerProducer();

    Integer result = producer.produce(integerAsString);

    assertEquals(Integer.parseInt(integerAsString), result);
}
```

**一个众所周知的场景是`Object#` `clone` 方法，它默认返回一个`Object` 。每当我们覆盖`clone()` 方法时，协变返回类型的功能允许我们拥有比`Object `本身更具体的返回对象。**

## 5.结论

在本文中，我们看到了什么是协方差和协变返回类型，以及它们在 Java 中的行为。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221206030049/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-methods)