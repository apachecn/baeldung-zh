# Java 类型系统面试问题

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-type-system-interview-questions>

[This article is part of a series:](javascript:void(0);)[• Java Collections Interview Questions](/web/20220812051601/https://www.baeldung.com/java-collections-interview-questions)
• Java Type System Interview Questions (current article)[• Java Concurrency Interview Questions (+ Answers)](/web/20220812051601/https://www.baeldung.com/java-concurrency-interview-questions)
[• Java Class Structure and Initialization Interview Questions](/web/20220812051601/https://www.baeldung.com/java-classes-initialization-questions)
[• Java 8 Interview Questions(+ Answers)](/web/20220812051601/https://www.baeldung.com/java-8-interview-questions)
[• Memory Management in Java Interview Questions (+Answers)](/web/20220812051601/https://www.baeldung.com/java-memory-management-interview-questions)
[• Java Generics Interview Questions (+Answers)](/web/20220812051601/https://www.baeldung.com/java-generics-interview-questions)
[• Java Flow Control Interview Questions (+ Answers)](/web/20220812051601/https://www.baeldung.com/java-flow-control-interview-questions)
[• Java Exceptions Interview Questions (+ Answers)](/web/20220812051601/https://www.baeldung.com/java-exceptions-interview-questions)
[• Java Annotations Interview Questions (+ Answers)](/web/20220812051601/https://www.baeldung.com/java-annotations-interview-questions)
[• Top Spring Framework Interview Questions](/web/20220812051601/https://www.baeldung.com/spring-interview-questions)

## 1。简介

Java 类型系统是 Java 开发人员技术访谈中经常提到的一个话题。本文回顾了一些最常被问到的重要问题，这些问题可能很难回答正确。

## 2。问题

### Q1。描述对象类在类型层次结构中的位置。哪些类型继承自 Object，哪些不继承？数组是从对象继承的吗？Lambda 表达式可以赋给对象变量吗？

在 Java 中， `java.lang.Object`位于类层次结构的顶端。所有的类都从它继承，要么显式继承，要么隐式继承(当类定义中省略了关键字`extends`时)，要么通过继承链传递。

但是有八个原语类型不是从`Object`继承而来的，分别是`boolean`、`byte`、`short`、`int`、`float`、`long`、`double`。

根据 Java 语言规范，数组也是对象。它们可以被赋值给一个`Object`引用，所有的`Object`方法都可以在它们上面被调用。

Lambda 表达式不能直接赋给一个`Object`变量，因为`Object`不是一个函数接口。但是你可以把一个 lambda 赋给一个函数接口变量，然后把它赋给一个`Object`变量(或者简单地通过同时把它强制转换成一个函数接口来把它赋给一个对象变量)。

### Q2。解释基本类型和引用类型的区别。

引用类型继承自顶层的`java.lang.Object`类，并且本身是可继承的(除了`final`类)。基本类型不继承，也不能被子类化。

原始类型的参数值总是通过堆栈传递，这意味着它们是通过值传递的，而不是通过引用。这意味着:对方法内部的基元参数值所做的更改不会传播到实际的参数值。

基本类型通常使用底层硬件值类型来存储。

例如，为了存储`int`值，可以使用 32 位存储单元。引用类型引入了对象头的开销，对象头存在于引用类型的每个实例中。

相对于简单的数值大小，对象头的大小非常重要。这就是最初引入基本类型的原因——节省对象开销的空间。不利的一面是，从技术上讲，并不是 Java 中的所有东西都是对象——原始值并不是从`Object`类继承的。

### Q3。描述不同的原语类型和它们占用的内存量。

Java 有 8 种基本类型:

*   `boolean` —逻辑`true` / `false`值。JVM 规范没有定义布尔值的大小，并且在不同的实现中可以有所不同。
*   `byte` —带符号的 8 位值，
*   `short` —带符号的 16 位值，
*   `char` —无符号 16 位值，
*   `int` —带符号的 32 位值，
*   `long` —带符号的 64 位值，
*   `float`—IEEE 754 标准对应的 32 位单精度浮点值，
*   `double` —对应 IEEE 754 标准的 64 位双精度浮点值。

### Q4。抽象类和接口的区别是什么？一个和另一个的用例是什么？

抽象类是定义中带有`abstract`修饰符的`class`。不能实例化，但可以子类化。接口是用`interface`关键字描述的类型。它也不能被实例化，但是可以被实现。

抽象类和接口的主要区别在于，一个类可以实现多个接口，但只能扩展一个抽象类。

一个`abstract`类通常被用作某个类层次结构中的基本类型，它标志着所有从它继承的类的主要意图。

一个`abstract`类也可以实现所有子类中需要的一些基本方法。例如，JDK 的大多数地图集合继承自`AbstractMap`类，该类实现了子类使用的许多方法(例如`equals`方法)。

接口指定了该类同意的一些约定。一个实现的接口可能不仅表示类的主要意图，还表示一些附加的契约。

例如，如果一个类实现了`Comparable`接口，这意味着这个类的实例可以被比较，不管这个类的主要目的是什么。

### Q5。接口类型的成员(字段和方法)有什么限制？

一个接口可以声明字段，但是它们被隐式声明为 `public`、`static`和`final`，即使你没有指定这些修饰符。因此，您不能显式地将接口字段定义为`private`。本质上，一个接口可能只有常量字段，而没有实例字段。

接口的所有方法也是隐式的`public`。它们也可以是(隐含的)`abstract`或`default`。

### Q6。内部类和静态嵌套类有什么区别？

简而言之——嵌套类基本上是在另一个类中定义的类。

嵌套类分为两类，具有非常不同的属性。内部类是一个不先实例化封闭类就不能实例化的类，也就是说，内部类的任何实例都隐式绑定到封闭类的某个实例。

这里有一个内部类的例子——你可以看到它可以以`OuterClass1.this`构造的形式访问外部类实例的引用:

```java
public class OuterClass1 {

    public class InnerClass {

        public OuterClass1 getOuterInstance() {
            return OuterClass1.this;
        }

    }

}
```

要实例化这样的内部类，您需要有一个外部类的实例:

```java
OuterClass1 outerClass1 = new OuterClass1();
OuterClass1.InnerClass innerClass = outerClass1.new InnerClass();
```

静态嵌套类则完全不同。从语法上来说，它只是一个定义中带有`static`修饰符的嵌套类。

实际上，这意味着该类可以作为任何其他类进行实例化，而无需绑定到封闭类的任何实例:

```java
public class OuterClass2 {

    public static class StaticNestedClass {
    }

}
```

要实例化这样的类，不需要外部类的实例:

```java
OuterClass2.StaticNestedClass staticNestedClass = new OuterClass2.StaticNestedClass();
```

### Q7。Java 有多重继承吗？

Java 不支持类的多重继承，这意味着一个类只能从一个超类继承。

但是你可以用一个类实现多个接口，这些接口的一些方法可能被定义为`default`并有一个实现。这允许您以更安全的方式在一个类中混合不同的功能。

### Q8。什么是包装类？什么是自动装箱？

对于 Java 中的八种原语类型，都有一个包装器类，可以用来包装一个原语值，并像使用对象一样使用它。这些等级相应地是`Boolean`、`Byte`、`Short`、`Character`、`Float`、`Long`和`Double`。例如，当您需要将一个原始值放入一个只接受引用对象的泛型集合时，这些包装器会很有用。

```java
List<Integer> list = new ArrayList<>();
list.add(new Integer(5));
```

为了省去手动来回转换原语的麻烦，Java 编译器提供了一种称为自动装箱/自动拆箱的自动转换。

```java
List<Integer> list = new ArrayList<>();
list.add(5);
int value = list.get(0);
```

### Q9。描述 equals()和== 的区别

==运算符允许您比较两个对象的“相同性”(即两个变量都引用内存中的同一个对象)。重要的是要记住，`new`关键字总是创建一个新对象，它不会通过与任何其他对象的`==`相等，即使它们看起来具有相同的值:

```java
String string1 = new String("Hello");
String string2 = new String("Hello");

assertFalse(string1 == string2);
```

另外，==运算符允许比较原始值:

```java
int i1 = 5;
int i2 = 5;

assertTrue(i1 == i2);
```

`equals()`方法在`java.lang.Object`类中定义，因此可用于任何引用类型。默认情况下，它只是通过==操作符检查对象是否相同。但是它通常在子类中被覆盖，以便为一个类提供特定的比较语义。

例如，对于`String`类，该方法检查字符串是否包含相同的字符:

```java
String string1 = new String("Hello");
String string2 = new String("Hello");

assertTrue(string1.equals(string2));
```

### Q10。假设您有一个引用类类型实例的变量。如何检查一个对象是这个类的实例？

在这种情况下，您不能使用`instanceof`关键字，因为它只有在您以文字形式提供实际类名时才有效。

幸运的是，`Class`类有一个方法`isInstance`,允许检查一个对象是否是这个类的实例:

```java
Class<?> integerClass = new Integer(5).getClass();
assertTrue(integerClass.isInstance(new Integer(4)));
```

### Q11。什么是匿名类？描述它的用例。

匿名类是一个一次性的类，在需要它的实例的地方定义。这个类是在同一个地方定义和实例化的，因此它不需要名字。

在 Java 8 之前，你会经常使用匿名类来定义单个方法接口的实现，比如`Runnable`。在 Java 8 中，lambdas 被用来代替单一的抽象方法接口。但是匿名类仍然有用例，例如，当你需要一个具有多个方法的接口的实例或者一个具有一些附加特性的类的实例时。

以下是创建和填充地图的方法:

```java
Map<String, Integer> ages = new HashMap<String, Integer>(){{
    put("David", 30);
    put("John", 25);
    put("Mary", 29);
    put("Sophie", 22);
}};
```

Next **»**[Java Concurrency Interview Questions (+ Answers)](/web/20220812051601/https://www.baeldung.com/java-concurrency-interview-questions)**«** Previous[Java Collections Interview Questions](/web/20220812051601/https://www.baeldung.com/java-collections-interview-questions)