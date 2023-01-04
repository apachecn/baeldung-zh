# 自动赋值简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/introduction-to-autovalue>

## 1。概述

[AutoValue](https://web.archive.org/web/20221006083408/https://github.com/google/auto) 是一个 Java 源代码生成器，更确切地说，它是一个库，用于**为值对象或值类型对象**生成源代码。

为了生成一个值类型对象，你所要做的就是**用`@AutoValue`注释**注释一个抽象类，然后编译你的类。生成的是一个带有访问器方法、参数化构造函数、正确覆盖的`toString(), equals(Object)`和`hashCode()`方法的值对象。

下面的代码片段是**一个抽象类的快速示例**，当编译时会产生一个名为`AutoValue_Person`的值对象。

```
@AutoValue
abstract class Person {
    static Person create(String name, int age) {
        return new AutoValue_Person(name, age);
    }

    abstract String name();
    abstract int age();
} 
```

让我们继续深入了解值对象，我们为什么需要它们，以及 AutoValue 如何帮助减少生成和重构代码的时间。

## 2。Maven 设置

要在 Maven 项目中使用 AutoValue，您需要在`pom.xml`中包含以下依赖项:

```
<dependency>
    <groupId>com.google.auto.value</groupId>
    <artifactId>auto-value</artifactId>
    <version>1.2</version>
</dependency>
```

最新版本可以通过关注[这个链接](https://web.archive.org/web/20221006083408/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22auto-value%22)找到。

## 3。值类型对象

值类型是库的最终产品，所以为了理解它在我们的开发任务中的位置，我们必须彻底理解值类型，它们是什么，它们不是什么，以及我们为什么需要它们。

### 3.1。什么是值类型？

值类型对象是这样的对象，它们之间的相等性不是由标识决定的，而是由它们的内部状态决定的。这意味着只要值类型对象的两个实例具有相同的字段值，它们就被认为是相同的。

**通常，值类型是不可变的**。它们的字段必须被设为`final`，并且它们不能有`setter`方法，因为这将使它们在实例化后可以改变。

它们必须通过构造函数或工厂方法使用所有字段值。

值类型不是 JavaBeans，因为它们没有默认或零参数构造函数，也没有 setter 方法，类似地，**它们不是数据传输对象，也不是普通的 Java 对象**。

此外，值类型类必须是 final，这样它们就不可扩展，至少不会有人覆盖这些方法。JavaBeans、dto 和 POJOs 不一定是最终版本。

### 3.2。创建值类型

假设我们想创建一个名为`Foo`的值类型，它的字段名为`text`和`number.`，我们该怎么做呢？

我们将创建一个 final 类，并将它的所有字段都标记为 final。然后我们将使用 IDE 生成构造函数、`hashCode()`方法、`equals(Object)`方法、`getters` 作为强制方法和一个`toString()`方法，我们将有一个类似这样的类:

```
public final class Foo {
    private final String text;
    private final int number;

    public Foo(String text, int number) {
        this.text = text;
        this.number = number;
    }

    // standard getters

    @Override
    public int hashCode() {
        return Objects.hash(text, number);
    }
    @Override
    public String toString() {
        return "Foo [text=" + text + ", number=" + number + "]";
    }
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null) return false;
        if (getClass() != obj.getClass()) return false;
        Foo other = (Foo) obj;
        if (number != other.number) return false;
        if (text == null) {
            if (other.text != null) return false;
        } else if (!text.equals(other.text)) {
            return false;
        }
        return true;
    }
}
```

在创建了`Foo`的实例之后，我们希望它的内部状态在整个生命周期中保持不变。

正如我们将在下面的**小节中看到的，一个对象的`hashCode`必须随着实例的不同而变化**，但是对于值类型，我们必须将它绑定到定义值对象内部状态的字段。

因此，即使改变同一对象的一个字段也会改变`hashCode`值。

### 3.3。值类型如何工作

值类型必须是不可变的原因是为了防止应用程序在实例化后对它们的内部状态进行任何更改。

因此，每当我们想要比较任何两个值类型的对象时，**我们必须使用`Object`类**的`equals(Object)`方法。

这意味着我们必须总是在我们自己的值类型中覆盖这个方法，并且只有当我们比较的值对象的字段具有相等的值时才返回 true。

此外，为了让我们在基于散列的集合中使用我们的值对象，比如`HashSet` s 和`HashMap` s 而不中断，**我们必须正确地实现`hashCode()`方法**。

### 3.4。为什么我们需要值类型

对值类型的需求经常出现。在这些情况下，我们想要覆盖原始`Object`类的默认行为。

正如我们已经知道的，当两个对象具有相同的身份时，`Object`类的默认实现认为它们是相等的，然而对于我们的目的来说，当两个对象具有相同的内部状态时，我们认为它们是相等的。

假设我们想要创建一个货币对象，如下所示:

```
public class MutableMoney {
    private long amount;
    private String currency;

    public MutableMoney(long amount, String currency) {
        this.amount = amount;
        this.currency = currency;
    }

    // standard getters and setters

}
```

我们可以对它运行下面的测试来测试它的相等性:

```
@Test
public void givenTwoSameValueMoneyObjects_whenEqualityTestFails_thenCorrect() {
    MutableMoney m1 = new MutableMoney(10000, "USD");
    MutableMoney m2 = new MutableMoney(10000, "USD");
    assertFalse(m1.equals(m2));
}
```

注意测试的语义。

当两个货币对象不相等时，我们认为它已经过去了。这是因为**我们没有覆盖`equals`方法**,所以相等性是通过比较对象的内存引用来衡量的，这些对象当然不会不同，因为它们是占用不同内存位置的不同对象。

每个对象代表 10，000 美元，但是 **Java 告诉我们，我们的货币对象不等于**。我们希望这两个对象只在货币数量不同或者货币类型不同时才进行不相等的测试。

现在让我们创建一个等价的值对象，这次我们将让 IDE 生成大部分代码:

```
public final class ImmutableMoney {
    private final long amount;
    private final String currency;

    public ImmutableMoney(long amount, String currency) {
        this.amount = amount;
        this.currency = currency;
    }

    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        result = prime * result + (int) (amount ^ (amount >>> 32));
        result = prime * result + ((currency == null) ? 0 : currency.hashCode());
        return result;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null) return false;
        if (getClass() != obj.getClass()) return false;
        ImmutableMoney other = (ImmutableMoney) obj;
        if (amount != other.amount) return false;
        if (currency == null) {
            if (other.currency != null) return false;
        } else if (!currency.equals(other.currency))
            return false;
        return true;
    }
}
```

唯一的区别是我们覆盖了 `equals(Object)`和`hashCode()`方法，现在我们可以控制我们希望 Java 如何比较我们的货币对象。让我们运行它的等价测试:

```
@Test
public void givenTwoSameValueMoneyValueObjects_whenEqualityTestPasses_thenCorrect() {
    ImmutableMoney m1 = new ImmutableMoney(10000, "USD");
    ImmutableMoney m2 = new ImmutableMoney(10000, "USD");
    assertTrue(m1.equals(m2));
}
```

注意这个测试的语义，当两个 money 对象通过`equals`方法测试相等时，我们期望它通过。

## 4。为什么选择 AutoValue？

既然我们彻底理解了值类型以及我们为什么需要它们，我们可以看看 AutoValue 以及它是如何进入等式的。

### 4.1。手工编码的问题

当我们创建值类型时，就像我们在上一节所做的那样，我们会遇到许多与糟糕的设计和大量样板代码相关的问题。

一个两个字段的类将有 9 行代码:一行用于包声明，两行用于类签名及其右括号，两行用于字段声明，两行用于构造函数及其右括号，两行用于初始化字段，但是我们需要字段的 getterss，每个 getter 需要多三行代码，这样就多了六行。

覆盖`hashCode()`和`equalTo(Object)`方法分别需要大约 9 行和 18 行，覆盖`toString()`方法增加了另外 5 行。

这意味着我们的两个字段类的格式良好的代码库将需要大约 50 行代码。

### 4.2。想去营救吗？

这对于像 Eclipse 或 IntilliJ 这样的 IDE 来说很容易，并且只需要创建一两个值类型类。想一想创建大量这样的类，即使 IDE 帮助了我们，它还会那么容易吗？

很快，几个月后，假设我们必须重新审视我们的代码，修改我们的`Money`类，并且可能将`currency`字段从`String` 类型转换为另一种叫做`Currency.` 的值类型

### 4.3。IDEs 并不真的很有帮助

像 Eclipse 这样的 IDE 不能简单地为我们编辑访问器方法，也不能编辑`toString()`、`hashCode()`或`equals(Object)`方法。

这个重构必须手工完成。编辑代码增加了潜在的错误，并且随着我们向`Money`类添加每个新字段，行数呈指数增长。

认识到这种情况会发生，而且经常大量发生，会让我们真正体会到自动增值的作用。

## 5。自动赋值示例

AutoValue 解决的问题是将我们在上一节中谈到的所有样板代码从我们的方式中去掉，这样我们就不必编写、编辑甚至阅读它们。

我们将查看完全相同的`Money` 示例，但这次使用的是 AutoValue。为了一致起见，我们将这个类称为`AutoValueMoney`:

```
@AutoValue
public abstract class AutoValueMoney {
    public abstract String getCurrency();
    public abstract long getAmount();

    public static AutoValueMoney create(String currency, long amount) {
        return new AutoValue_AutoValueMoney(currency, amount);
    }
}
```

我们编写了一个抽象类，为它定义了抽象访问器，但没有字段，我们用`@AutoValue` 注释了这个类，总共只有 8 行代码，`javac`为我们生成了一个具体的子类，如下所示:

```
public final class AutoValue_AutoValueMoney extends AutoValueMoney {
    private final String currency;
    private final long amount;

    AutoValue_AutoValueMoney(String currency, long amount) {
        if (currency == null) throw new NullPointerException(currency);
        this.currency = currency;
        this.amount = amount;
    }

    // standard getters

    @Override
    public int hashCode() {
        int h = 1;
        h *= 1000003;
        h ^= currency.hashCode();
        h *= 1000003;
        h ^= amount;
        return h;
    }

    @Override
    public boolean equals(Object o) {
        if (o == this) {
            return true;
        }
        if (o instanceof AutoValueMoney) {
            AutoValueMoney that = (AutoValueMoney) o;
            return (this.currency.equals(that.getCurrency()))
              && (this.amount == that.getAmount());
        }
        return false;
    }
}
```

我们根本不需要直接处理这个类，当我们需要添加更多的字段或对我们的字段进行修改时，也不需要编辑它，就像上一节中的`currency`场景一样。

**`Javac`总是会为我们**重新生成更新的代码。

当使用这个新的值类型时，所有调用方看到的只是父类型，我们将在下面的单元测试中看到。

下面是一个测试，验证我们的字段设置是否正确:

```
@Test
public void givenValueTypeWithAutoValue_whenFieldsCorrectlySet_thenCorrect() {
    AutoValueMoney m = AutoValueMoney.create("USD", 10000);
    assertEquals(m.getAmount(), 10000);
    assertEquals(m.getCurrency(), "USD");
}
```

验证具有相同货币和相同金额的两个`AutoValueMoney`对象相等的测试如下:

```
@Test
public void given2EqualValueTypesWithAutoValue_whenEqual_thenCorrect() {
    AutoValueMoney m1 = AutoValueMoney.create("USD", 5000);
    AutoValueMoney m2 = AutoValueMoney.create("USD", 5000);
    assertTrue(m1.equals(m2));
}
```

当我们将一个货币对象的货币类型更改为 GBP 时，测试: `5000 GBP == 5000 USD`不再成立:

```
@Test
public void given2DifferentValueTypesWithAutoValue_whenNotEqual_thenCorrect() {
    AutoValueMoney m1 = AutoValueMoney.create("GBP", 5000);
    AutoValueMoney m2 = AutoValueMoney.create("USD", 5000);
    assertFalse(m1.equals(m2));
}
```

## 6。使用构建器自动赋值

我们看到的第一个例子涵盖了使用静态工厂方法作为公共创建 API 的 AutoValue 的基本用法。

**注意，如果我们所有的字段都是** `**Strings**,`，那么当我们将它们传递给静态工厂方法时，交换它们就很容易了，比如将`amount`放在`currency`的位置，反之亦然。

如果我们有很多字段并且都是`String`类型，这种情况尤其容易发生。使用 AutoValue，**所有的字段都是通过构造函数**初始化的，这使得问题变得更加严重。

为了解决这个问题，我们应该使用`builder`模式。幸好。这可以由 AutoValue 生成。

我们的 AutoValue 类实际上没有太大的变化，除了静态工厂方法被一个生成器所取代:

```
@AutoValue
public abstract class AutoValueMoneyWithBuilder {
    public abstract String getCurrency();
    public abstract long getAmount();
    static Builder builder() {
        return new AutoValue_AutoValueMoneyWithBuilder.Builder();
    }

    @AutoValue.Builder
    abstract static class Builder {
        abstract Builder setCurrency(String currency);
        abstract Builder setAmount(long amount);
        abstract AutoValueMoneyWithBuilder build();
    }
}
```

生成的类与第一个完全相同，但是生成了一个具体的内部类，并实现了构建器中的抽象方法:

```
static final class Builder extends AutoValueMoneyWithBuilder.Builder {
    private String currency;
    private long amount;
    Builder() {
    }
    Builder(AutoValueMoneyWithBuilder source) {
        this.currency = source.getCurrency();
        this.amount = source.getAmount();
    }

    @Override
    public AutoValueMoneyWithBuilder.Builder setCurrency(String currency) {
        this.currency = currency;
        return this;
    }

    @Override
    public AutoValueMoneyWithBuilder.Builder setAmount(long amount) {
        this.amount = amount;
        return this;
    }

    @Override
    public AutoValueMoneyWithBuilder build() {
        String missing = "";
        if (currency == null) {
            missing += " currency";
        }
        if (amount == 0) {
            missing += " amount";
        }
        if (!missing.isEmpty()) {
            throw new IllegalStateException("Missing required properties:" + missing);
        }
        return new AutoValue_AutoValueMoneyWithBuilder(this.currency,this.amount);
    }
}
```

还要注意测试结果没有变化。

如果我们想知道字段值实际上是通过构建器正确设置的，我们可以执行这个测试:

```
@Test
public void givenValueTypeWithBuilder_whenFieldsCorrectlySet_thenCorrect() {
    AutoValueMoneyWithBuilder m = AutoValueMoneyWithBuilder.builder().
      setAmount(5000).setCurrency("USD").build();
    assertEquals(m.getAmount(), 5000);
    assertEquals(m.getCurrency(), "USD");
}
```

要测试等式是否依赖于内部状态:

```
@Test
public void given2EqualValueTypesWithBuilder_whenEqual_thenCorrect() {
    AutoValueMoneyWithBuilder m1 = AutoValueMoneyWithBuilder.builder()
      .setAmount(5000).setCurrency("USD").build();
    AutoValueMoneyWithBuilder m2 = AutoValueMoneyWithBuilder.builder()
      .setAmount(5000).setCurrency("USD").build();
    assertTrue(m1.equals(m2));
}
```

当字段值不同时:

```
@Test
public void given2DifferentValueTypesBuilder_whenNotEqual_thenCorrect() {
    AutoValueMoneyWithBuilder m1 = AutoValueMoneyWithBuilder.builder()
      .setAmount(5000).setCurrency("USD").build();
    AutoValueMoneyWithBuilder m2 = AutoValueMoneyWithBuilder.builder()
      .setAmount(5000).setCurrency("GBP").build();
    assertFalse(m1.equals(m2));
}
```

## 7。结论

在本教程中，我们介绍了 Google 的 AutoValue 库的大部分基础知识，以及如何用很少的代码创建值类型。

谷歌 AutoValue 的一个替代方案是 [Lombok 项目](https://web.archive.org/web/20221006083408/https://projectlombok.org/)——你可以在这里看看关于使用 [Lombok 的介绍文章。](/web/20221006083408/https://www.baeldung.com/intro-to-project-lombok)

所有这些例子和代码片段的完整实现可以在 AutoValue [GitHub 项目](https://web.archive.org/web/20221006083408/https://github.com/eugenp/tutorials/tree/master/code-generation)中找到。