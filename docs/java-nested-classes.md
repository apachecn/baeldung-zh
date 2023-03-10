# Java 中的嵌套类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-nested-classes>

## 1。简介

本教程是对 Java 语言中嵌套类的简明扼要的介绍。

简单地说，Java 允许我们在其他类中定义类。嵌套类使我们能够对只在一个地方使用的类进行逻辑分组，编写更具可读性和可维护性的代码，并增加封装性。

在我们开始之前，让我们看一下语言中可用的几种嵌套类:

*   静态嵌套类
*   非静态嵌套类
*   本地课程
*   匿名类

在接下来的几节中，我们将详细讨论其中的每一项。

## 2。静态嵌套类

关于静态嵌套类，需要记住以下几点:

*   与静态成员一样，这些成员属于它们的封闭类，而不属于该类的实例
*   它们的声明中可以包含所有类型的访问修饰符
*   它们只能访问封闭类中的静态成员
*   它们可以定义静态和非静态成员

让我们看看如何声明一个静态嵌套类:

```java
public class Enclosing {

    private static int x = 1;

    public static class StaticNested {

        private void run() {
            // method implementation
        }
    }

    @Test
    public void test() {
        Enclosing.StaticNested nested = new Enclosing.StaticNested();
        nested.run();
    }
}
```

## 3。非静态嵌套类

接下来，这里有一些关于非静态嵌套类需要快速记住的要点:

*   它们也被称为内部类
*   它们的声明中可以包含所有类型的访问修饰符
*   就像实例变量和方法一样，内部类与封闭类的实例相关联
*   它们可以访问封闭类的所有成员，不管它们是静态的还是非静态的
*   它们只能定义非静态成员

下面是我们如何声明一个内部类:

```java
public class Outer {

    public class Inner {
        // ...
    }
}
```

如果我们用修饰符`static`声明一个嵌套类，那么它就是一个静态成员。否则就是内班。尽管语法上的区别只是一个关键字(即`static`)，但语义上这些嵌套类之间有着巨大的区别。内部类实例被绑定到封闭类实例，因此它们可以访问它们的成员。在选择是否将嵌套类作为内部类时，我们应该意识到这个问题。

要实例化一个内部类，我们必须首先实例化它的封闭类。

让我们看看如何做到这一点:

```java
Outer outer = new Outer();
Outer.Inner inner = outer.new Inner();
```

在接下来的小节中，我们将展示一些特殊类型的内部类。

### 3.1。本地课程

局部类是一种特殊类型的内部类——其中**类被定义在方法**或作用域块中。

让我们来看看这类课程需要记住的几点:

*   它们的声明中不能有访问修饰符
*   它们可以访问封闭上下文中的静态和非静态成员
*   他们只能定义实例成员

这里有一个简单的例子:

```java
public class NewEnclosing {

    void run() {
        class Local {

            void run() {
                // method implementation
            }
        }
        Local local = new Local();
        local.run();
    }

    @Test
    public void test() {
        NewEnclosing newEnclosing = new NewEnclosing();
        newEnclosing.run();
    }
}
```

### 3.2。匿名类

匿名类可用于定义接口或抽象类的实现，而不必创建可重用的实现。

让我们列出关于匿名类需要记住的几点:

*   它们的声明中不能有访问修饰符
*   它们可以访问封闭上下文中的静态和非静态成员
*   他们只能定义实例成员
*   它们是唯一不能定义构造函数或扩展/实现其他类或接口的嵌套类

要定义一个匿名类，让我们首先定义一个简单的抽象类:

```java
abstract class SimpleAbstractClass {
    abstract void run();
}
```

现在让我们看看如何定义一个匿名类:

```java
public class AnonymousInnerUnitTest {

    @Test
    public void whenRunAnonymousClass_thenCorrect() {
        SimpleAbstractClass simpleAbstractClass = new SimpleAbstractClass() {
            void run() {
                // method implementation
            }
        };
        simpleAbstractClass.run();
    }
}
```

要了解更多细节，我们可能会发现 Java 中关于[匿名类的教程很有用。](/web/20220724090411/https://www.baeldung.com/java-anonymous-classes)

## 4。阴影

如果内部类成员的声明与封闭类成员的声明同名，那么它们就会隐藏起来。

在这种情况下，`this`关键字引用嵌套类的实例，外部类的成员可以使用外部类的名称来引用。

让我们看一个简单的例子:

```java
public class NewOuter {

    int a = 1;
    static int b = 2;

    public class InnerClass {
        int a = 3;
        static final int b = 4;

        public void run() {
            System.out.println("a = " + a);
            System.out.println("b = " + b);
            System.out.println("NewOuterTest.this.a = " + NewOuter.this.a);
            System.out.println("NewOuterTest.b = " + NewOuter.b);
            System.out.println("NewOuterTest.this.b = " + NewOuter.this.b);
        }
    }

    @Test
    public void test() {
        NewOuter outer = new NewOuter();
        NewOuter.InnerClass inner = outer.new InnerClass();
        inner.run();

    }
}
```

## 5。序列化

为了避免在试图序列化嵌套类时出现`java.io.NotSerializableException`，我们应该:

*   将嵌套类声明为`static`
*   让嵌套类和封闭类都实现`Serializable`

## 6。结论

在本文中，我们已经看到了什么是嵌套类以及它们的不同类型。我们还研究了不同类型的字段可见性和访问修饰符的区别。

一如既往，本教程的完整实现可以在 GitHub 上找到[。](https://web.archive.org/web/20220724090411/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-types)