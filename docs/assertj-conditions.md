# 对 AssertJ 断言使用条件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/assertj-conditions>

## 1。概述

在本教程中，我们将看一看 AssertJ 库，特别是定义和使用条件来创建可读和可维护的测试。

AssertJ 基础知识可以在[这里](/web/20220524034754/https://www.baeldung.com/introduction-to-assertj)找到。

## 2。测试中的类别

让我们来看看我们将要编写测试用例的目标类:

```java
public class Member {
    private String name;
    private int age;

    // constructors and getters
}
```

## 3。创造条件

我们可以通过简单地用适当的参数实例化`Condition`类来定义断言条件。

**创建`Condition`最方便的方法是使用将`Predicate`作为参数**的构造函数。其他构造函数要求我们创建一个子类并覆盖`matches`方法，这不太方便。

当构造一个`Condition`对象时，我们必须指定一个类型参数，它是用来评估条件的值的类型。

让我们为我们的`Member`类的`age`字段声明一个条件:

```java
Condition<Member> senior = new Condition<>(
  m -> m.getAge() >= 60, "senior");
```

`senior`变量现在引用一个`Condition`实例，该实例根据`age`测试一个`Person`是否是高级的。

**构造函数的第二个参数`String` `“senior”`是一个简短的描述，如果条件失败，AssertJ 将使用它来构建一个用户友好的错误消息。**

另一个条件，检查一个`Person`是否有`name`“John”，看起来像这样:

```java
Condition<Member> nameJohn = new Condition<>(
  m -> m.getName().equalsIgnoreCase("John"), 
  "name John"
);
```

## 4。测试用例

现在，让我们看看如何在我们的测试类中使用`Condition`对象。假设条件`senior`和`nameJohn`在我们的测试类中作为字段可用。

### 4.1。断言标量值

当`age`值高于资历阈值时，应通过以下测试:

```java
Member member = new Member("John", 65);
assertThat(member).is(senior);
```

由于使用`is`方法的断言通过了，使用具有相同参数的`isNot`的断言将失败:

```java
// assertion fails with an error message containing "not to be <senior>"
assertThat(member).isNot(senior);
```

使用`nameJohn`变量，我们可以编写两个类似的测试:

```java
Member member = new Member("Jane", 60);
assertThat(member).doesNotHave(nameJohn);

// assertion fails with an error message containing "to have:\n <name John>"
assertThat(member).has(nameJohn);
```

**`is`和`has`方法，以及`isNot`和`doesNotHave`方法具有相同的语义**。我们使用哪一种只是选择的问题。尽管如此，我们还是建议选择一个能让我们的测试代码可读性更好的方法。

### 4.2。断言集合

条件不仅仅对标量值起作用，它们还可以验证集合中元素的存在或不存在。让我们来看一个测试案例:

```java
List<Member> members = new ArrayList<>();
members.add(new Member("Alice", 50));
members.add(new Member("Bob", 60));

assertThat(members).haveExactly(1, senior);
assertThat(members).doNotHave(nameJohn);
```

`haveExactly`方法断言满足给定`Condition`的元素的确切数量，而`doNotHave`方法检查元素的缺失。

方法`haveExactly`和`doNotHave`不是唯一使用收集条件的方法。有关这些方法的完整列表，请参见 API 文档中的[AbstractIterableAssert 类](https://web.archive.org/web/20220524034754/https://joel-costigliola.github.io/assertj/core-8/api/org/assertj/core/api/AbstractIterableAssert.html)。

### 4.3。组合条件

**我们可以使用`Assertions`类的三个静态方法组合各种条件:**

*   `**not –**`如果不满足指定的条件，则创建一个满足的条件
*   **`allOf –`** 创建一个仅当所有指定条件都满足时才满足的条件
*   `**anyOf –**`创建一个条件，如果至少满足一个指定条件，则满足该条件

下面是如何使用`not`和`allOf`方法来组合条件:

```java
Member john = new Member("John", 60);
Member jane = new Member("Jane", 50);

assertThat(john).is(allOf(senior, nameJohn));
assertThat(jane).is(allOf(not(nameJohn), not(senior)));
```

同样，我们可以利用`anyOf`:

```java
Member john = new Member("John", 50);
Member jane = new Member("Jane", 60);

assertThat(john).is(anyOf(senior, nameJohn));
assertThat(jane).is(anyOf(nameJohn, senior));
```

## 5。结论

本教程给出了 AssertJ 条件的指南，以及如何使用它们在测试代码中创建可读性很强的断言。

所有示例和代码片段的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20220524034754/https://github.com/eugenp/tutorials/tree/master/testing-modules/assertion-libraries)