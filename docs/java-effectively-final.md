# Java 中的 Final 与有效 Final

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-effectively-final>

## 1.介绍

Java 8 中引入的最有趣的特性之一实际上是 final。它允许我们不必为变量、字段和参数编写 [`final`](/web/20221206102211/https://www.baeldung.com/java-final) 修饰符，这些变量、字段和参数可以像最终变量一样被有效地处理和使用。

在本教程中，我们将探索这个特性的**来源，以及与`final` 关键字**相比，编译器是如何处理它的。此外，我们将探索一个关于有效最终变量的有问题用例的解决方案。

## 2.有效的最终来源

简单来说，**对象或原始值实际上是最终的，如果我们在初始化**后不改变它们的值。在对象的情况下，如果我们不改变对象的引用，那么它实际上是最终的——即使被引用对象的状态发生了变化。

在引入它之前，**我们不能在匿名类**中使用非最终局部变量。我们仍然不能在匿名类、内部类和 lambda 表达式中使用被赋予多个值的变量。这个特性的引入允许我们不必在有效的 final 变量上使用`final`修饰符，节省了我们几次击键的时间。

[匿名类](/web/20221206102211/https://www.baeldung.com/java-anonymous-classes)是内部类，按照 [JLS 8.1.3 的规定，它们不能访问非 final 或非 effective-final 变量，也不能在它们的封闭范围内改变它们。](https://web.archive.org/web/20221206102211/https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#jls-8.1.3)同样的限制也适用于 lambda 表达式，因为拥有访问权限可能会产生并发问题。

## 3.最终与有效最终

理解一个 final 变量是否是有效 final 的最简单的方法是考虑移除关键字`final`是否允许代码编译和运行:

```java
@FunctionalInterface
public interface FunctionalInterface {
    void testEffectivelyFinal();
    default void test() {
        int effectivelyFinalInt = 10;
        FunctionalInterface functionalInterface 
            = () -> System.out.println("Value of effectively variable is : " + effectivelyFinalInt);
    }
} 
```

重新赋值或改变上述有效的最终变量将使代码无效，不管它出现在哪里。

### 3.1.编译器处理

[JLS 4.12.4](https://web.archive.org/web/20221206102211/https://docs.oracle.com/javase/specs/jls/se8/html/jls-4.html#jls-4.12) 声明如果我们从方法参数或局部变量中移除`final`修饰符而不引入编译时错误，那么它实际上是最终的。此外，如果我们在一个有效程序的变量声明中添加了`final`关键字，那么它实际上就是 final。

Java 编译器不会对有效的最终变量进行额外的优化，不像它对`final`变量那样。

让我们考虑一个简单的例子，它声明了两个`final String`变量，但只使用它们进行连接:

```java
public static void main(String[] args) {
    final String hello = "hello";
    final String world = "world";
    String test = hello + " " + world;
    System.out.println(test);
} 
```

**编译器会将上面的`main`方法中执行的代码改为:**

```java
public static void main(String[] var0) {
    String var1 = "hello world";
    System.out.println(var1);
}
```

另一方面，**如果我们删除了`final`修饰符**，这些变量将被认为是最终变量，但是**编译器不会删除它们**，因为它们只用于连接。

## 4.原子修饰

一般来说，**修改 lambda 表达式和匿名类**中使用的变量不是一个好的做法。我们不知道这些变量将如何在方法块中使用。在多线程环境中，改变它们可能会导致意想不到的结果。

我们已经有一个教程解释了使用 [lambda 表达式](/web/20221206102211/https://www.baeldung.com/java-8-lambda-expressions-tips)的最佳实践，还有一个教程解释了当我们修改它们时[常见的反模式。但是有一种替代方法，允许我们在这种情况下修改变量，通过原子性实现线程安全。](/web/20221206102211/https://www.baeldung.com/java-lambda-effectively-final-local-variables)

套装`java.util.concurrent.atomic`提供了`AtomicReference`和`AtomicInteger`这样的职业。我们可以用它们来自动修改 lambda 表达式中的变量:

```java
public static void main(String[] args) {
    AtomicInteger effectivelyFinalInt = new AtomicInteger(10);
    FunctionalInterface functionalInterface = effectivelyFinalInt::incrementAndGet;
}
```

## 5.结论

在本教程中，我们了解了`final`和有效最终变量之间最显著的差异。此外，我们提供了一个安全的替代方法，允许我们修改 lambda 函数中的变量。