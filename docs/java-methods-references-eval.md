# Java 中方法引用的求值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-methods-references-eval>

## 1.概观

Java 8 引入了方法引用的概念。我们经常看到它们类似于 lambda 表达式。

然而，方法引用和 lambda 表达式并不完全一样。在本文中，我们将展示为什么它们是不同的，以及以错误的方式使用它们会有什么风险。

## 2.Lambdas 和方法引用语法

首先，让我们看几个 lambda 表达式的例子:

```java
Runnable r1 = () -> "some string".toUpperCase();
Consumer<String> c1 = x -> x.toUpperCase(); 
```

和一些方法引用的例子:

```java
Function<String, String> f1 = String::toUpperCase;
Runnable r2 = "some string"::toUpperCase;
Runnable r3 = String::new;
```

这些例子可以让我们把方法引用看作是 lambdas 的一种简化符号。

但是让我们来看看官方的[甲骨文文档](https://web.archive.org/web/20220926193746/https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.13)。我们可以在那里找到一个有趣的例子:

```java
(test ? list.replaceAll(String::trim) : list) :: iterator
```

正如我们所看到的，Java 语言规范允许我们在双冒号操作符之前有不同种类的表达式。 **::** 之前的部分称为****目标参考**。**

 **接下来，我们将讨论方法引用评估的过程。

## 3.方法参考评估

当我们运行下面的代码时会发生什么？

```java
public static void main(String[] args) {
    Runnable runnable = (f("some") + f("string"))::toUpperCase;
}

private static String f(String string) {
    System.out.println(string);
    return string;
}
```

我们刚刚创建了一个`Runnable` 对象。不多不少。但是，输出是:

```java
some
string 
```

发生这种情况是因为第一次发现声明时会评估**目标引用。因此，我们失去了渴望的懒惰。**目标引用也只评估一次。**因此，如果我们将这一行添加到上面的示例中:**

```java
runnable.run()
```

我们将看不到任何输出。下一个案子呢？

```java
SomeWorker worker = null;
Runnable workLambda = () -> worker.work() // ok
Runnable workMethodReference = worker::work; // boom! NullPointerException
```

由之前提到的文档提供的解释:

`“A method invocation expression (§15.12) that invokes an instance method throws a NullPointerException if the target reference is null.”`

防止意外情况的最好方法可能是**永远不要使用变量访问和复杂表达式作为目标引用**。

一个好主意可能是只使用方法引用作为其 lambda 等价物的简洁符号。在 **::** 操作符之前只有一个类名可以保证安全。

## 4.结论

在本文中，我们已经了解了方法引用的评估过程。

我们知道风险和我们应该遵循的规则，以免突然对我们的应用程序的行为感到惊讶。**