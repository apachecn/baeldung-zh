# Java 注释面试问题(+答案)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-annotations-interview-questions>

[This article is part of a series:](javascript:void(0);)[• Java Collections Interview Questions](/web/20221208143855/https://www.baeldung.com/java-collections-interview-questions)
[• Java Type System Interview Questions](/web/20221208143855/https://www.baeldung.com/java-type-system-interview-questions)
[• Java Concurrency Interview Questions (+ Answers)](/web/20221208143855/https://www.baeldung.com/java-concurrency-interview-questions)
[• Java Class Structure and Initialization Interview Questions](/web/20221208143855/https://www.baeldung.com/java-classes-initialization-questions)
[• Java 8 Interview Questions(+ Answers)](/web/20221208143855/https://www.baeldung.com/java-8-interview-questions)
[• Memory Management in Java Interview Questions (+Answers)](/web/20221208143855/https://www.baeldung.com/java-memory-management-interview-questions)
[• Java Generics Interview Questions (+Answers)](/web/20221208143855/https://www.baeldung.com/java-generics-interview-questions)
[• Java Flow Control Interview Questions (+ Answers)](/web/20221208143855/https://www.baeldung.com/java-flow-control-interview-questions)
[• Java Exceptions Interview Questions (+ Answers)](/web/20221208143855/https://www.baeldung.com/java-exceptions-interview-questions)
• Java Annotations Interview Questions (+ Answers) (current article)[• Top Spring Framework Interview Questions](/web/20221208143855/https://www.baeldung.com/spring-interview-questions)

## 1。简介

注释从 Java 5 开始就存在了，现在，它们是无处不在的编程结构，可以丰富代码。

在本文中，我们将回顾一些关于注释的问题；在技术面试中经常被问到的问题。我们将实现一些例子来更好地理解他们的答案。

## 2.问题

### Q1。什么是注释？他们有哪些典型的用例？

注释是绑定到程序源代码元素的元数据，对它们所操作的代码的操作没有影响。

它们的典型使用案例有:

*   **编译器信息**–通过注释，编译器可以检测错误或抑制警告
*   **编译时和部署时处理**–软件工具可以处理注释并生成代码、配置文件等。
*   **运行时处理**–可以在运行时检查注释，以定制程序的行为

### Q2。描述标准库中一些有用的注释。

`java.lang`和`java.lang.annotation`包中有几个注释，比较常见的包括但不限于:

*   标记一个方法意味着覆盖一个在超类中声明的元素。如果未能正确重写该方法，编译器将发出一个错误
*   `@Deprecated` –表示元素已被弃用，不应使用。如果程序使用用此批注标记的方法、类或字段，编译器将发出警告
*   `@SuppressWarnings` –告诉编译器抑制特定的警告。在与泛型出现之前编写的遗留代码交互时最常用
*   `@FunctionalInterface`–在 Java 8 中引入，表示类型声明是一个函数接口，其实现可以使用 Lambda 表达式提供

### Q3。如何创建注释？

注释是一种接口形式，其中关键字`interface`在`@,`之前，并且其主体包含看起来非常类似于方法的`annotation type element`声明:

```java
public @interface SimpleAnnotation {
    String value();

    int[] types();
}
```

定义注释后，您可以开始在代码中使用它:

```java
@SimpleAnnotation(value = "an element", types = 1)
public class Element {
    @SimpleAnnotation(value = "an attribute", types = { 1, 2 })
    public Element nextElement;
}
```

请注意，为数组元素提供多个值时，必须用括号将它们括起来。

也可以选择提供默认值，只要它们是编译器的常量表达式:

```java
public @interface SimpleAnnotation {
    String value() default "This is an element";

    int[] types() default { 1, 2, 3 };
}
```

现在，您可以在没有这些元素的情况下使用注释:

```java
@SimpleAnnotation
public class Element {
    // ...
}
```

或者只是其中的一部分:

```java
@SimpleAnnotation(value = "an attribute")
public Element nextElement;
```

### Q4。注释方法声明可以返回哪些对象类型？

返回类型必须是基元、`String`、`Class`、`Enum`或上述类型之一的数组。否则，编译器将抛出错误。

下面是一个成功遵循这一原则的示例代码:

```java
enum Complexity {
    LOW, HIGH
}

public @interface ComplexAnnotation {
    Class<? extends Object> value();

    int[] types();

    Complexity complexity();
}
```

下一个示例将无法编译，因为`Object` 不是有效的返回类型:

```java
public @interface FailingAnnotation {
    Object complexity();
}
```

### Q5。哪些程序元素可以被注释？

注释可以应用于整个源代码的几个地方。它们可以应用于类、构造函数和字段的声明:

```java
@SimpleAnnotation
public class Apply {
    @SimpleAnnotation
    private String aField;

    @SimpleAnnotation
    public Apply() {
        // ...
    }
}
```

方法及其参数:

```java
@SimpleAnnotation
public void aMethod(@SimpleAnnotation String param) {
    // ...
}
```

局部变量，包括循环和资源变量:

```java
@SimpleAnnotation
int i = 10;

for (@SimpleAnnotation int j = 0; j < i; j++) {
    // ...
}

try (@SimpleAnnotation FileWriter writer = getWriter()) {
    // ...
} catch (Exception ex) {
    // ...
}
```

其他注释类型:

```java
@SimpleAnnotation
public @interface ComplexAnnotation {
    // ...
}
```

甚至是包，通过`package-info.java`文件:

```java
@PackageAnnotation
package com.baeldung.interview.annotations;
```

从 Java 8 开始，它们也可以应用于类型的`use`。为此，注释必须指定一个值为`ElementType.USE`的`@Target` 注释:

```java
@Target(ElementType.TYPE_USE)
public @interface SimpleAnnotation {
    // ...
}
```

现在，注释可以应用于类实例的创建:

```java
new @SimpleAnnotation Apply();
```

类型转换:

```java
aString = (@SimpleAnnotation String) something;
```

实现子句:

```java
public class SimpleList<T>
  implements @SimpleAnnotation List<@SimpleAnnotation T> {
    // ...
}
```

和`throws`条款:

```java
void aMethod() throws @SimpleAnnotation Exception {
    // ...
}
```

### Q6。有没有办法限制可以应用注释的元素？

是的，`@Target`注释可以用于这个目的。如果我们试图在不适用的上下文中使用注释，编译器将会发出一个错误。

这里有一个例子，将`@SimpleAnnotation`注释的使用仅限于字段声明:

```java
@Target(ElementType.FIELD)
public @interface SimpleAnnotation {
    // ...
}
```

如果我们想让它适用于更多的上下文，我们可以传递多个常量:

```java
@Target({ ElementType.FIELD, ElementType.METHOD, ElementType.PACKAGE })
```

我们甚至可以做一个注释，这样它就不能用来注释任何东西。当声明的类型仅用作复杂批注中的成员类型时，这可能会很方便:

```java
@Target({})
public @interface NoTargetAnnotation {
    // ...
}
```

### Q7。什么是元注释？

是应用于其他批注的批注。

所有未标有`@Target,`或标有但包含`ANNOTATION_TYPE`常量的注释也是元注释:

```java
@Target(ElementType.ANNOTATION_TYPE)
public @interface SimpleAnnotation {
    // ...
}
```

### Q8。什么是重复标注？

这些注释可以多次应用于同一个元素声明。

出于兼容性的原因，由于这个特性是在 Java 8 中引入的，所以重复的注释存储在一个由 Java 编译器自动生成的`container annotation`中。对于编译器来说，有两个步骤来声明它们。

首先，我们需要声明一个可重复的注释:

```java
@Repeatable(Schedules.class)
public @interface Schedule {
    String time() default "morning";
}
```

然后，我们用一个强制的`value`元素定义包含注释，其类型必须是可重复注释类型的数组:

```java
public @interface Schedules {
    Schedule[] value();
}
```

现在，我们可以多次使用@Schedule:

```java
@Schedule
@Schedule(time = "afternoon")
@Schedule(time = "night")
void scheduledMethod() {
    // ...
}
```

### Q9。如何检索注释？这与其保留政策有什么关系？

您可以使用反射 API 或注释处理器来检索注释。

`@Retention`注释和它的`RetentionPolicy`参数影响你如何检索它们。`RetentionPolicy`枚举中有三个常量:

*   `RetentionPolicy.SOURCE`–使注释被编译器丢弃，但注释处理器可以读取它们
*   `RetentionPolicy.CLASS`–表示注释被添加到类文件中，但不能通过反射访问
*   `RetentionPolicy.RUNTIME`–注释由编译器记录在类文件中，并由 JVM 在运行时保留，以便它们可以被反射性地读取

下面是创建可在运行时读取的注释的示例代码:

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface Description {
    String value();
}
```

现在，可以通过反射来检索注释:

```java
Description description
  = AnnotatedClass.class.getAnnotation(Description.class);
System.out.println(description.value());
```

一个注释处理器可以与`RetentionPolicy.SOURCE`一起工作，这在文章 [Java 注释处理和创建一个生成器](/web/20221208143855/https://www.baeldung.com/java-annotation-processing-builder)中有描述。

当你编写 Java 字节码解析器时,`RetentionPolicy.CLASS`是有用的。

### Q10。下面的代码会编译吗？

```java
@Target({ ElementType.FIELD, ElementType.TYPE, ElementType.FIELD })
public @interface TestAnnotation {
    int[] value() default {};
}
```

不。如果同一个枚举常量在一个`@Target`注释中出现不止一次，这是一个编译时错误。

删除重复的常量将使代码成功编译:

```java
@Target({ ElementType.FIELD, ElementType.TYPE})
```

### Q11。有可能扩展注释吗？

不。注释总是按照 Java 语言规范中的[来扩展`java.lang.annotation.Annotation,` 。](https://web.archive.org/web/20221208143855/https://docs.oracle.com/javase/specs/jls/se7/html/jls-9.html#jls-9.6)

如果我们试图在注释声明中使用`extends`子句，我们会得到一个编译错误:

```java
public @interface AnAnnotation extends OtherAnnotation {
    // Compilation error
}
```

## 3。结论

在本文中，我们讨论了 Java 开发人员技术访谈中出现的一些关于注释的常见问题。这绝不是一个详尽的列表，而应该被认为是进一步研究的开始。

我们 Baeldung 祝您在即将到来的面试中取得成功。

Next **»**[Top Spring Framework Interview Questions](/web/20221208143855/https://www.baeldung.com/spring-interview-questions)**«** Previous[Java Exceptions Interview Questions (+ Answers)](/web/20221208143855/https://www.baeldung.com/java-exceptions-interview-questions)