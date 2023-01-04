# Java 8 中的双冒号操作符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-8-double-colon-operator>

## 1。概述

在这篇简短的文章中，我们将讨论 Java 8 中的**双冒号运算符** ( `**::**`),并回顾可以使用该运算符的场景。

## 延伸阅读:

## [Java 8 面试问题(+答案)](/web/20220812064002/https://www.baeldung.com/java-8-interview-questions)

A set of popular Java8-related interview questions and of course answers.[Read more](/web/20220812064002/https://www.baeldung.com/java-8-interview-questions) →

## [Java 8 可选指南](/web/20220812064002/https://www.baeldung.com/java-optional)

Quick and practical guide to Optional in Java 8[Read more](/web/20220812064002/https://www.baeldung.com/java-optional) →

## [Java 8 中的新特性](/web/20220812064002/https://www.baeldung.com/java-8-new-features)

A short intro into the new features of Java 8; the focus is on default and static interface methods, static method references, and Optional.[Read more](/web/20220812064002/https://www.baeldung.com/java-8-new-features) →

## 2。从 Lambdas 到双冒号运算符

通过 Lambdas 表达式，我们看到代码可以变得非常简洁。

例如，要**创建一个比较器**，以下语法就足够了:

```java
Comparator c = (Computer c1, Computer c2) -> c1.getAge().compareTo(c2.getAge()); 
```

然后，用类型推断:

```java
Comparator c = (c1, c2) -> c1.getAge().compareTo(c2.getAge());
```

但是我们能让上面的代码更有表现力和可读性吗？让我们来看看:

```java
Comparator c = Comparator.comparing(Computer::getAge); 
```

我们使用`::`操作符作为 lambdas 调用特定方法的简写——通过名称。最终，结果当然是更加可读的语法。

## 3。它是如何工作的？

简单地说，当我们使用方法引用时——目标引用放在分隔符`::`之前，方法的名称放在它之后。

例如:

```java
Computer::getAge;
```

我们正在查看对在`Computer`类中定义的方法`getAge`的方法引用。

然后，我们可以使用该函数进行操作:

```java
Function<Computer, Integer> getAge = Computer::getAge;
Integer computerAge = getAge.apply(c1); 
```

请注意，我们引用了函数，然后将它应用于正确的论点。

## 4。方法引用

在很多情况下，我们可以很好地利用这个操作符。

### 4.1。静态方法

首先，我们将利用**一个静态实用方法**:

```java
List inventory = Arrays.asList(
  new Computer( 2015, "white", 35), new Computer(2009, "black", 65));
inventory.forEach(ComputerUtils::repair); 
```

### 4.2。现有对象的实例方法

接下来，让我们看看一个有趣的场景——**引用一个现有对象实例**的方法。

我们将使用变量`System`。`out`–支持`print`方法的`PrintStream` 类型的对象:

```java
Computer c1 = new Computer(2015, "white");
Computer c2 = new Computer(2009, "black");
Computer c3 = new Computer(2014, "black");
Arrays.asList(c1, c2, c3).forEach(System.out::print); 
```

### 4.3。特定类型的任意对象的实例方法

```java
Computer c1 = new Computer(2015, "white", 100);
Computer c2 = new MacbookPro(2009, "black", 100);
List inventory = Arrays.asList(c1, c2);
inventory.forEach(Computer::turnOnPc); 
```

如您所见，我们引用的`turnOnPc`方法不是针对特定的实例，而是针对类型本身。

在第 4 行，将为`inventory`的每个对象调用实例方法`turnOnPc`。

这自然意味着——对于`c1`,将在`Computer`实例上调用方法`turnOnPc`,对于`c2`,将在`MacbookPro`实例上调用方法`turnOnPc`。

### 4.4。特定对象的超级方法

假设在`Computer`超类中有以下方法:

```java
public Double calculateValue(Double initialValue) {
    return initialValue/1.50;
} 
```

这个在`MacbookPro`子类中:

```java
@Override
public Double calculateValue(Double initialValue){
    Function<Double, Double> function = super::calculateValue;
    Double pcValue = function.apply(initialValue);
    return pcValue + (initialValue/10) ;
} 
```

对`MacbookPro`实例上的`calculateValue`方法的调用:

```java
macbookPro.calculateValue(999.99); 
```

也将产生对`Computer`超类上的`calculateValue`的调用。

## 5。构造函数引用

### 5.1。创建一个新实例

引用构造函数来实例化对象可能非常简单:

```java
@FunctionalInterface
public interface InterfaceComputer {
    Computer create();
}

InterfaceComputer c = Computer::new;
Computer computer = c.create(); 
```

如果一个构造函数中有两个参数呢？

```java
BiFunction<Integer, String, Computer> c4Function = Computer::new; 
Computer c4 = c4Function.apply(2013, "white"); 
```

如果参数为三个或更多，您必须定义一个新的功能接口:

```java
@FunctionalInterface 
interface TriFunction<A, B, C, R> { 
    R apply(A a, B b, C c); 
    default <V> TriFunction<A, B, C, V> andThen( Function<? super R, ? extends V> after) { 
        Objects.requireNonNull(after); 
        return (A a, B b, C c) -> after.apply(apply(a, b, c)); 
    } 
} 
```

然后，初始化您的对象:

```java
TriFunction <Integer, String, Integer, Computer> c6Function = Computer::new;
Computer c3 = c6Function.apply(2008, "black", 90); 
```

### 5.2。创建一个数组

最后，让我们看看如何创建一个包含五个元素的`Computer`对象的数组:

```java
Function <Integer, Computer[]> computerCreator = Computer[]::new;
Computer[] computerArray = computerCreator.apply(5); 
```

## 6。结论

正如我们开始看到的，双冒号操作符——在 Java 8 中引入——在某些场景中非常有用，尤其是与流结合使用时。

为了更好地理解幕后发生的事情，查看功能接口也是非常重要的。

该示例的完整的**源代码**可以在[的 GitHub 项目](https://web.archive.org/web/20220812064002/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lambdas)中找到——这是一个 Maven 和 Eclipse 项目，因此可以按原样导入和使用。