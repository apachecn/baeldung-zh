# Java 中的广义目标类型推理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-generalized-target-type-inference>

## 1.介绍

类型推断是在 Java 5 中引入的，以补充泛型的引入，并在随后的 Java 版本中得到了实质性的扩展，这也被称为广义目标类型推断。

在本教程中，我们将通过代码示例来探索这个概念。

## 2.无商标消费品

泛型为我们提供了许多好处，比如增加类型安全性、避免类型转换错误和泛型算法。你可以在这篇[文章](/web/20221126230104/https://www.baeldung.com/java-generics)中读到更多关于泛型的内容。

然而，**泛型的引入导致了编写样板代码的必要性，因为需要传递类型参数**。一些例子是:

```java
Map<String, Map<String, String>> mapOfMaps = new HashMap<String, Map<String, String>>();
List<String> strList = Collections.<String>emptyList();
List<Integer> intList = Collections.<Integer>emptyList();
```

## 3.Java 8 之前的类型推断

为了减少不必要的代码冗长，Java 引入了类型推断，这是一个基于上下文信息自动推断表达式的非特定数据类型的过程。

现在，我们可以调用相同的泛型类型和方法，而无需指定参数类型。编译器会在需要时自动推断参数类型。

我们可以看到使用新概念的相同代码:

```java
List<String> strListInferred = Collections.emptyList();
List<Integer> intListInferred = Collections.emptyList(); 
```

在上面的例子中，基于预期的返回类型`List<String>`和`List<Integer>`，编译器能够将类型参数推断为下面的泛型方法:

```java
public static final <T> List<T> emptyList() 
```

正如我们所看到的，结果代码是简洁的。现在，如果可以推断出类型参数，我们可以像普通方法一样调用泛型方法。

在 Java 5 中，我们可以在如上所示的特定上下文中进行类型推断。

Java 7 扩展了它的执行环境。它介绍了钻石运算符 **< >** 。你可以在这篇[文章](/web/20221126230104/https://www.baeldung.com/java-diamond-operator)中读到更多关于钻石操作者的信息。

现在，**我们可以在赋值上下文中为泛型类构造函数执行这个操作。**一个这样的例子是:

```java
Map<String, Map<String, String>> mapOfMapsInferred = new HashMap<>();
```

这里，Java 编译器使用预期的赋值类型来推断`HashMap`构造函数的类型参数。

## 4.广义目标类型推理–Java 8

Java 8 进一步扩展了类型推断的范围。我们将这种扩展的推理能力称为广义目标类型推理。你可以在这里阅读技术细节。

Java 8 也引入了 Lambda 表达式。 **Lambda 表达式没有显式类型。他们的类型是通过观察上下文或情境的目标类型来推断的。**根据表达式出现的位置，表达式的目标类型是 Java 编译器期望的数据类型。

Java 8 支持在方法上下文中使用 Target-Type 进行推理。当我们在没有显式类型参数的情况下调用泛型方法时，编译器可以查看方法调用和相应的方法声明，以确定使调用适用的类型参数。

让我们看一个示例代码:

```java
static <T> List<T> add(List<T> list, T a, T b) {
    list.add(a);
    list.add(b);
    return list;
}

List<String> strListGeneralized = add(new ArrayList<>(), "abc", "def");
List<Integer> intListGeneralized = add(new ArrayList<>(), 1, 2);
List<Number> numListGeneralized = add(new ArrayList<>(), 1, 2.0);
```

在代码中，`ArrayList<>`没有显式提供类型参数。所以，编译器需要推断它。首先，编译器检查 add 方法的参数。然后，它查看在不同调用中传递的参数。

**它执行`invocation applicability inference`分析以确定该方法是否适用于这些调用**。如果由于重载而导致多个方法都适用，编译器会选择最具体的方法。

然后，**编译器执行`invocation type inference`分析来确定类型参数。** **预期的目标类型也用于此分析**。它推导出三种情况下的论点为`ArrayList<String>`、`ArrayList<Integer>`和`ArrayList<Number>`。

目标类型推理允许我们不为 lambda 表达式参数指定类型:

```java
List<Integer> intList = Arrays.asList(5, 2, 4, 2, 1);
Collections.sort(intList, (a, b) -> a.compareTo(b));

List<String> strList = Arrays.asList("Red", "Blue", "Green");
Collections.sort(strList, (a, b) -> a.compareTo(b));
```

这里，参数`a`和`b`没有明确定义的类型。它们的类型在第一个 Lambda 表达式中被推断为`Integer`，在第二个表达式中被推断为`String`。

## 5.结论

在这篇简短的文章中，我们回顾了类型推理，它与泛型和 Lambda 表达式一起使我们能够编写简洁的 Java 代码。

像往常一样，完整的源代码可以在 Github 上找到[。](https://web.archive.org/web/20221126230104/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8)