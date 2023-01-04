# Java 注释属性值限制

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-annotation-attribute-value-restrictions>

## 1.概观

如今，很难想象没有注释的 Java，注释是 Java 语言中一个强大的工具。

Java 提供了一组[内置注释](/web/20220926191050/https://www.baeldung.com/java-default-annotations)。此外，还有大量来自不同库的注释。我们甚至可以定义和处理自己的注释。我们可以用属性值来调整这些注释，但是，这些属性值有局限性。特别是，**注释属性值必须是常量表达式**。

在本教程中，我们将了解这种限制的一些原因，并从 JVM 的角度更好地解释它。我们还将看一些涉及注释属性值的问题和解决方案的例子。

## 2.幕后的 Java 注释属性

让我们考虑 Java 类文件如何存储注释属性。Java 有一个特殊的结构叫做`[element_value](https://web.archive.org/web/20220926191050/https://docs.oracle.com/javase/specs/jvms/se15/html/jvms-4.html#jvms-4.7.16.1)`。这个结构存储一个特定的注释属性。

结构`element_value`可以存储四种不同类型的值:

*   常数池中的常数
*   类文字
*   嵌套注释
*   一组值

因此，来自注释属性的常量是一个编译时常量。否则，编译器不知道应该将什么值放入常量池并用作注释属性。

Java 规范定义了产生[常量表达式](https://web.archive.org/web/20220926191050/https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.28)的操作。如果我们将这些操作应用于编译时常数，我们将得到编译时常数。

假设我们有一个注释`@Marker`，它有一个属性`value`:

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Marker {
    String value();
}
```

例如，以下代码编译时没有错误:

```java
@Marker(Example.ATTRIBUTE_FOO + Example.ATTRIBUTE_BAR)
public class Example {
    static final String ATTRIBUTE_FOO = "foo";
    static final String ATTRIBUTE_BAR = "bar";

    // ...
}
```

这里，我们将注释属性定义为两个字符串的串联。串联运算符产生一个常量表达式。

## 3.使用静态初始化器

让我们考虑一个在`static`块中初始化的常数:

```java
@Marker(Example.ATTRIBUTE_FOO)
public class Example {
    static final String[] ATTRIBUTES = {"foo", "Bar"};
    static final String ATTRIBUTE_FOO;

    static {
        ATTRIBUTE_FOO = ATTRIBUTES[0];
    }

    // ...
} 
```

它初始化`static`块中的字段，并尝试使用该字段作为注释属性。**这种方法会导致编译错误。**

首先，变量`ATTRIBUTE_FOO`有`static`和`final`修饰符，但是编译器不能计算那个字段。应用程序在运行时计算它。

其次，**注释属性在 JVM 加载类**之前必须有一个确切的值。然而，当`static`初始化器运行时，类已经被加载了。所以，这个限制是有意义的。

在字段初始化时出现相同的错误。出于同样的原因，此代码不正确:

```java
@Marker(Example.ATTRIBUTE_FOO)
public class Example {
    static final String[] ATTRIBUTES = {"foo", "Bar"};
    static final String ATTRIBUTE_FOO = ATTRIBUTES[0];

    // ...
} 
```

JVM 如何初始化`ATTRIBUTE_FOO`？数组访问操作符`ATTRIBUTES[0]`在类初始化器中运行。所以，`ATTRIBUTE_FOO`是一个运行时常数。它不是在编译时定义的。

## 4.作为注释属性的数组常量

让我们考虑一个数组注释属性:

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Marker {
    String[] value();
} 
```

此代码将不会编译:

```java
@Marker(value = Example.ATTRIBUTES)
public class Example {
    static final String[] ATTRIBUTES = {"foo", "bar"};

    // ...
}
```

首先，尽管`final`修饰符保护引用不被改变，但是**我们仍然可以修改数组元素**。

其次，**数组文字不能是运行时常量。JVM 在静态初始化器**中设置每个元素——这是我们之前描述过的一个限制。

最后，一个类文件存储数组中每个元素的值。因此，编译器计算属性数组的每个元素，这发生在编译时。

因此，我们每次只能指定一个数组属性:

```java
@Marker(value = {"foo", "bar"})
public class Example {
    // ...
}
```

我们仍然可以使用常量作为数组属性的基本元素。

## 5。标记界面中的注释:为什么不起作用？

所以，如果一个注释属性是一个数组，我们必须每次都重复它。但是我们希望避免这种复制粘贴。我们为什么不做注解呢？我们可以将我们的注释添加到一个[标记接口](/web/20220926191050/https://www.baeldung.com/java-marker-interfaces#:~:text=A%20marker%20interface%20is%20an,also%20called%20a%20tagging%20interface.):

```java
@Marker(value = {"foo", "bar"})
public interface MarkerInterface {
} 
```

然后，我们可以让需要这个注释的类实现它:

```java
public class Example implements MarkerInterface {
    // ...
}
```

**这种方法行不通**。代码将编译无误。然而， **Java 不支持从接口**继承注释，即使注释本身有`@Inherited`注释。因此，实现标记接口的类不会继承注释。

**之所以这样，是多重继承的问题**。的确，如果多个接口有相同的注释，Java 无法选择一个。

所以，我们无法避免这种带有标记接口的复制粘贴。

## 6.作为注释属性的数组元素

假设我们有一个数组常量，我们将这个常量用作注释属性:

```java
@Marker(Example.ATTRIBUTES[0])
public class Example {
    static final String[] ATTRIBUTES = {"Foo", "Bar"};
    // ...
}
```

这段代码无法编译。批注参数必须是编译时常数。但是，正如我们之前所考虑的，**数组不是编译时常数**。

此外，**数组访问表达式不是常量表达式**。

如果我们有一个`List`而不是一个数组会怎么样？方法调用不属于常量表达式。因此，使用`List`类的`get`方法会导致同样的错误。

相反，我们应该明确地引用一个常数:

```java
@Marker(Example.ATTRIBUTE_FOO)
public class Example {
    static final String ATTRIBUTE_FOO = "Foo";
    static final String[] ATTRIBUTES = {ATTRIBUTE_FOO, "Bar"};
    // ...
} 
```

这样，我们在字符串常量中指定注释属性值，Java 编译器可以明确地找到属性值。

## 7.结论

在本文中，我们研究了注释参数的局限性。我们考虑了一些注释属性问题的例子。我们还在这些限制的背景下讨论了 JVM 的内部机制。

在所有的例子中，我们对常量和注释使用了相同的类。然而，所有这些限制都适用于常数来自另一个类的情况。