# ArrayStoreException 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-arraystoreexception>

## 1。概述

当试图在对象 的[数组中存储不正确的对象类型时，在 Java **的运行时抛出`ArrayStoreException`。由于`ArrayStoreException`是一个**](/web/20220523232733/https://www.baeldung.com/java-arrays-guide)**[未检查的异常](/web/20220523232733/https://www.baeldung.com/java-checked-unchecked-exceptions)，处理或声明它并不常见。**

在本教程中，我们将演示`ArrayStoreException`的原因，如何处理它，以及避免它的最佳实践。

## 2。`ArrayStoreException`起因

当我们试图在数组中存储不同类型的对象而不是声明的类型时，Java 抛出了一个`ArrayStoreException`。

假设我们实例化了一个类型为`String`的数组，然后试图在其中存储`Integer`。在这种情况下，在运行时，`ArrayStoreException`被抛出:

```java
Object array[] = new String[5];
array[0] = 2;
```

当我们试图在数组中存储不正确的值类型时，将在第二行代码中引发异常:

```java
Exception in thread "main" java.lang.ArrayStoreException: java.lang.Integer
    at com.baeldung.array.arraystoreexception.ArrayStoreExceptionExample.main(ArrayStoreExceptionExample.java:9)
```

因为我们将`array`声明为 `Object`，所以**编译是没有错误的**。

## 3。`ArrayStoreException` 搬运

这个异常的处理非常简单。如任何其他异常，也需要用[试抓块](/web/20220523232733/https://www.baeldung.com/java-exceptions) 包围**进行处理:**

```java
try{
    Object array[] = new String[5];
    array[0] = 2;
}
catch (ArrayStoreException e) {
    // handle the exception
}
```

## 4。避免这种例外的最佳实践

建议**将数组类型声明为特定的类，如`String`或`Integer`，而不是`Object`** 。当我们将数组类型声明为`Object,`时，编译器不会抛出任何错误。

但是**用基类声明数组，然后存储不同类的对象，会导致编译错误**。让我们看一个简单的例子:

```java
String array[] = new String[5];
array[0] = 2;
```

在上面的例子中，我们将数组类型声明为`String `，并尝试在其中存储一个`Integer `。这将导致编译错误:

```java
Exception in thread "main" java.lang.Error: Unresolved compilation problem: 
  Type mismatch: cannot convert from int to String
    at com.baeldung.arraystoreexception.ArrayStoreExampleCE.main(ArrayStoreExampleCE.java:8)
```

如果我们在编译时而不是运行时捕获错误会更好，因为我们对前者有更多的控制。

## 5。结论

在本教程中，我们学习了 Java 中`ArrayStoreException`的原因、处理和预防。

GitHub 上的[提供了完整的示例。](https://web.archive.org/web/20220523232733/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-guides)