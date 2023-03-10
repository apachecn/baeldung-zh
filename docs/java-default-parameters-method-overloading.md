# 使用方法重载的 Java 默认参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-default-parameters-method-overloading>

## 1.概观

在这个简短的教程中，我们将演示如何使用方法重载来模拟 Java 中的默认参数。

这里，我们说 simulate 是因为与某些其他 OOP 语言(如 C++和 Scala)不同，**Java 规范不支持为方法参数**分配默认值。

## 2.例子

举个例子，我们来泡点茶吧！首先，我们需要一个`Tea` POJO:

```java
public class Tea {

    static final int DEFAULT_TEA_POWDER = 1;

    private String name; 
    private int milk;
    private boolean herbs;
    private int sugar;
    private int teaPowder;

    // standard getters 
} 
```

这里，`name`是必填字段，因为我们的`Tea`至少要有一个名称。

那么，没有茶粉就没有茶。因此，我们假设用户想要一杯标准的 1 tbsp `teaPowder`茶，如果调用时没有提供的话。这是我们的第一个默认参数。

其他可选参数有`milk`(单位 ml)`herbs`(添加或不添加)`sugar`(单位 tbsp)。如果没有提供它们的任何值，我们假设用户不想要它们。

**让我们看看如何使用方法重载在 Java 中实现这一点**:

```java
public Tea(String name, int milk, boolean herbs, int sugar, int teaPowder) {
    this.name = name;
    this.milk = milk;
    this.herbs = herbs;
    this.sugar = sugar;
    this.teaPowder = teaPowder;
}

public Tea(String name, int milk, boolean herbs, int sugar) {
    this(name, milk, herbs, sugar, DEFAULT_TEA_POWDER);
}

public Tea(String name, int milk, boolean herbs) {
    this(name, milk, herbs, 0);
}

public Tea(String name, int milk) {
    this(name, milk, false);
}

public Tea(String name) {
    this(name, 0);
}
```

很明显，这里我们使用了构造函数链接，这是一种重载的形式，为方法参数提供一些默认值。

现在，让我们添加一个简单的测试来看看这一点:

```java
@Test
public void whenTeaWithOnlyName_thenCreateDefaultTea() {
    Tea blackTea = new Tea("Black Tea");

    assertThat(blackTea.getName()).isEqualTo("Black Tea");
    assertThat(blackTea.getMilk()).isEqualTo(0);
    assertThat(blackTea.isHerbs()).isFalse();
    assertThat(blackTea.getSugar()).isEqualTo(0);
    assertThat(blackTea.getTeaPowder()).isEqualTo(Tea.DEFAULT_TEA_POWDER);
}
```

## 3.可供选择的事物

在 Java 中实现默认参数模拟还有其他方法。其中一些是:

*   使用[构建器模式](/web/20221126214719/https://www.baeldung.com/java-builder-pattern-freebuilder#Builder%20java)
*   使用[可选](/web/20221126214719/https://www.baeldung.com/java-optional)
*   允许空值作为方法参数

下面是我们如何在示例中利用允许空参数的第三种方式:

```java
public Tea(String name, Integer milk, Boolean herbs, Integer sugar, Integer teaPowder) {
    this.name = name;
    this.milk = milk == null ? 0 : milk.intValue();
    this.herbs = herbs == null ? false : herbs.booleanValue();
    this.sugar = sugar == null ? 0 : sugar.intValue();
    this.teaPowder = teaPowder == null ? DEFAULT_TEA_POWDER : teaPowder.intValue();
}
```

## 4.结论

在本文中，我们研究了如何使用方法重载来模拟 Java 中的默认参数。

虽然有其他方法可以达到同样的目的，但是重载是最干净和简单的。和往常一样，GitHub 上的代码[可用。](https://web.archive.org/web/20221126214719/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-2)