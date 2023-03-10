# 捕捉可投掷物体是一种不好的做法吗？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-catch-throwable-bad-practice>

## 1.概观

在本教程中，我们将看看捕捉`Throwable` 的**含义** **。**

## 2.`Throwable`类

在 Java 文档中，`Throwable` 类被定义为“**Java 语言**中所有错误和异常的超类”。

让我们看看`Throwable`类的层次结构:

[![](img/6f1adaf3c28e29a860313736ad7dfab9.png)](/web/20220720104416/https://www.baeldung.com/wp-content/uploads/2019/11/Throwable-3.png)

`Throwable`类有两个直接子类——即`Error`和`Exception`类。

`Error`及其子类为未检查异常，而`Exception`的子类可以是[已检查或未检查异常](/web/20220720104416/https://www.baeldung.com/java-exceptions)。

让我们看看程序失败时可能经历的情况类型。

## 3.可恢复的情况

有些情况下，恢复通常是可能的，并且可以用`Exception`类的已检查或未检查子类来处理。

例如，一个程序可能想要使用一个在指定位置不存在的文件，导致被检查的`FileNotFoundException`被抛出。

另一个例子是程序试图在没有权限的情况下访问系统资源，导致抛出未检查的`AccessControl` `Exception`。

根据 Java 文档，**`Exception` 类“表示一个合理的应用程序可能想要捕获**的条件”。

## 4.无法挽回的局面

在某些情况下，程序可能会在出现故障时处于无法恢复的状态。常见的例子是发生堆栈溢出或 JVM 内存不足时。

在这些情况下，JVM 分别抛出`StackOverflowError` 和`OutOfMemoryError`。顾名思义，这些是`Error`类的子类。

根据 Java 文档，**`Error` 类“表示一个合理的应用程序不应该试图捕捉**的严重问题”。

## 5.可恢复和不可恢复情况示例

假设我们有一个 API，它允许调用者使用`addIDsToStorage`方法将惟一的 id 添加到一些存储设备中:

```java
class StorageAPI {

    public void addIDsToStorage(int capacity, Set<String> storage) throws CapacityException {
        if (capacity < 1) {
            throw new CapacityException("Capacity of less than 1 is not allowed");
        }
        int count = 0;
        while (count < capacity) {
            storage.add(UUID.randomUUID().toString());
            count++;
        }
    }

    // other methods go here ...
}
```

调用`addIDsToStorage`时可能会出现几个潜在的故障点:

*   `CapacityException –`当传递小于 1 的`capacity`值时，`Exception` 的检查子类
*   `NullPointerException –`如果提供了`null storage` 值而不是`Set<String>`的实例，则为`Exception`的未检查子类
*   如果 JVM 在退出`while`循环之前耗尽了内存，则为`Error`的一个未检查的子类

`CapacityException`和`NullPointerException`情况是程序可以恢复的失败，但是`OutOfMemoryError`是不可恢复的。

## 6.捕捉`Throwable`

让我们假设 API 的用户在调用`addIDsToStorage`时只捕获`try-catch`中的`Throwable` :

```java
public void add(StorageAPI api, int capacity, Set<String> storage) {
    try {
        api.addIDsToStorage(capacity, storage);
    } catch (Throwable throwable) {
        // do something here
    }
}
```

这意味着调用代码以同样的方式对可恢复和不可恢复的情况做出反应。

处理异常的一般规则是`try-catch`块在捕捉异常时必须尽可能具体。也就是说，**一个包罗万象的场景必须避免**。

**在我们的例子中，捕捉`Throwable`违反了这条一般规则。**为了分别对可恢复和不可恢复的情况做出反应，调用代码必须检查`catch`块中`Throwable`对象的实例。

更好的方法是使用特定的方法处理异常，避免试图处理不可恢复的情况。

## 7.结论

在本文中，我们研究了在`try-catch`块中捕捉`Throwable`的含义。

和往常一样，这个例子的完整源代码可以从 Github 上的[处获得。](https://web.archive.org/web/20220720104416/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-2)