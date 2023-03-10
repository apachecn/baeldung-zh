# Java 14 中 instanceof 的模式匹配

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-pattern-matching-instanceof>

## 1。概述

在这个快速教程中，我们将通过查看`instanceof`的模式匹配来继续我们关于 Java 14 的[系列，这是这个版本的 JDK 包含的另一个新的预览功能。](/web/20221227013536/https://www.baeldung.com/tag/java-14/)

**总之， [JEP 305](https://web.archive.org/web/20221227013536/https://openjdk.java.net/jeps/305) 旨在使从对象中有条件地提取组件更加简单、简洁、可读和安全。**

## 2。传统的`instanceOf`操作员

在某种程度上，我们可能都写过或见过包含某种条件逻辑的代码来测试一个对象是否具有特定的类型。**通常情况下，我们可以用后面跟有[转换](/web/20221227013536/https://www.baeldung.com/java-type-casting)的`[instanceof](/web/20221227013536/https://www.baeldung.com/java-instanceof)`操作符来实现。**这允许我们在应用特定于该类型的进一步处理之前提取变量。

假设我们想在一个简单的动物对象层次结构中检查类型:

```java
if (animal instanceof Cat) {
    Cat cat = (Cat) animal;
    cat.meow();
   // other cat operations
} else if (animal instanceof Dog) {
    Dog dog = (Dog) animal;
    dog.woof();
    // other dog operations
}

// More conditional statements for different animals
```

在这个例子中，对于每个条件块，我们测试 animal 参数来确定它的类型，通过强制转换来转换它，并声明一个局部变量。然后，我们可以针对特定的动物进行操作。

尽管这种方法可行，但它有几个缺点:

*   编写这种类型的代码很乏味，我们需要测试类型并对每个条件块进行强制转换
*   我们对每个`if`块重复三次类型名
*   可读性很差，因为强制转换和变量提取在代码中占主导地位
*   重复声明类型名意味着更有可能引入错误。这可能会导致意外的运行时错误
*   每当我们增加一只新动物时，这个问题就会放大

在下一节中，我们将看看 Java 14 提供了哪些增强来解决这些缺点。

## 3。Java 14 中增强的`instanceOf`

Java 14 通过 JEP 305 带来了一个改进版本的`instanceof`操作符，该操作符既测试参数，又将它赋给适当类型的绑定变量。

**这意味着我们可以用一种更简洁的方式来写我们之前的动物例子**:

```java
if (animal instanceof Cat cat) {
    cat.meow();
} else if(animal instanceof Dog dog) {
    dog.woof();
}
```

让我们了解一下这里发生了什么。在第一个`if`块中，我们将`animal`与类型模式`Cat cat`进行匹配。首先，我们测试`animal`变量，看看它是否是`Cat`的一个实例。如果是这样，它将被转换为我们的`Cat` 类型，最后，我们将结果赋给`cat`。

**值得注意的是，变量名`cat`不是一个现有的变量，而是一个模式变量的声明。**

我们还应该提到，变量`cat`和`dog`仅在各自的模式匹配表达式返回`true`时才在范围内赋值。**因此，如果我们试图在另一个位置使用任何一个变量，代码都会产生编译错误。**

正如我们所看到的，这个版本的代码更容易理解。我们简化了代码，显著减少了显式强制转换的总数，可读性也大大提高了。

此外，这种类型的测试模式在编写[等式](/web/20221227013536/https://www.baeldung.com/java-equals-hashcode-contracts)方法时特别有用。

## 4。结论

在这个简短的教程中，我们看了 Java 14 中使用`instanceof`的模式匹配。使用这种新的内置语言增强功能有助于我们编写更好、更可读的代码，这通常是一件好事。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221227013536/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-14)