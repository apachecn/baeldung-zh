# AtomicMarkableReference 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-atomicmarkablereference>

## 1.概观

在本教程中，我们将深入研究`java.util.concurrent.atomic`包中的 **`AtomicMarkableReference`类的细节。**

接下来，我们将遍历该类的 API 方法，并看看如何在实践中使用`AtomicMarkableReference`类。

## 2.目的

`AtomicMarkableReference`是一个通用类，它封装了对`Object`和`boolean`标志的引用。这两个字段耦合在一起，并且**可以一起或单独更新。**

`AtomicMarkableReference`也可能是**针对 [ABA 问题](/web/20221107200852/https://www.baeldung.com/cs/aba-concurrency)的一种可能的补救措施。**

## 3.履行

让我们更深入地看看`AtomicMarkableReference`类的实现:

```java
public class AtomicMarkableReference<V> {

    private static class Pair<T> {
        final T reference;
        final boolean mark;
        private Pair(T reference, boolean mark) {
            this.reference = reference;
            this.mark = mark;
        }
        static <T> Pair<T> of(T reference, boolean mark) {
            return new Pair<T>(reference, mark);
        }
    }

    private volatile Pair<V> pair;

    // ...
}
```

注意`AtomicMarkableReference`有一个**静态嵌套类`Pair`，它保存了引用和标志。**

同样，我们看到**两个变量都是`final`** 。因此，**每当我们想要修改这些变量时，就会创建一个新的`Pair`类实例，而旧的实例会被替换为**。

## 4.方法

首先，为了发现`AtomicMarkableReference`的有用性，让我们从创建一个`Employee` POJO 开始:

```java
class Employee {
    private int id;
    private String name;

    // constructor & getters & setters
}
```

现在，我们可以创建一个`AtomicMarkableReference`类的实例:

```java
AtomicMarkableReference<Employee> employeeNode 
  = new AtomicMarkableReference<>(new Employee(123, "Mike"), true);
```

对于我们的例子，让我们假设我们的`AtomicMarkableReference `实例代表组织图中的一个节点。它保存两个变量:指向`Employee`类的一个实例的`reference`和指示雇员是否在职或已经离开公司的`mark` 。

`AtomicMarkableReference`提供了几种方法来更新或检索一个或两个字段。让我们逐一看看这些方法:

### 4.1.`getReference()`

我们使用`getReference`方法返回`reference`变量的当前值:

```java
Employee employee = new Employee(123, "Mike");
AtomicMarkableReference<Employee> employeeNode = new AtomicMarkableReference<>(employee, true);

Assertions.assertEquals(employee, employeeNode.getReference());
```

### 4.2.`isMarked()`

为了获得`mark`变量的值，我们应该调用`isMarked`方法:

```java
Employee employee = new Employee(123, "Mike");
AtomicMarkableReference<Employee> employeeNode = new AtomicMarkableReference<>(employee, true);

Assertions.assertTrue(employeeNode.isMarked());
```

### 4.3.`get()`

接下来，当我们想要检索当前的`reference`和当前的`mark`时，我们使用`get`方法。为了得到`mark`，**，我们应该发送一个大小至少为 1 的`boolean`数组作为参数，该数组将在索引 0 处存储`boolean`变量**的当前值。同时，该方法将返回`reference`的当前值:

```java
Employee employee = new Employee(123, "Mike");
AtomicMarkableReference<Employee> employeeNode = new AtomicMarkableReference<>(employee, true);

boolean[] markHolder = new boolean[1];
Employee currentEmployee = employeeNode.get(markHolder);

Assertions.assertEquals(employee, currentEmployee);
Assertions.assertTrue(markHolder[0]);
```

这种获取`reference`和`mark`字段的方式有点奇怪，因为内部的`Pair`类没有向调用者公开。

Java 的公共 API 中没有通用的`Pair<T, U>`类。这样做的主要原因是我们可能倾向于过度使用它，而不是创建不同的类型。

### 4.4.`set()`

如果我们想无条件地更新`reference`和`mark`字段，我们应该使用`set`方法。如果作为参数发送的值中至少有一个不同，则`reference`和`mark`将被更新:

```java
Employee employee = new Employee(123, "Mike");
AtomicMarkableReference<Employee> employeeNode = new AtomicMarkableReference<>(employee, true);

Employee newEmployee = new Employee(124, "John");
employeeNode.set(newEmployee, false);

Assertions.assertEquals(newEmployee, employeeNode.getReference());
Assertions.assertFalse(employeeNode.isMarked());
```

### 4.5.`compareAndSet()`

接下来，如果当前`reference`等于预期的`reference`，并且当前`mark`等于预期的`mark`，则`compareAndSet`方法将`reference`和`mark`更新为给定的更新值**。**

现在，让我们看看如何使用`compareAndSet`来更新`reference`和`mark`字段:

```java
Employee employee = new Employee(123, "Mike");
AtomicMarkableReference<Employee> employeeNode = new AtomicMarkableReference<>(employee, true);
Employee newEmployee = new Employee(124, "John");

Assertions.assertTrue(employeeNode.compareAndSet(employee, newEmployee, true, false));
Assertions.assertEquals(newEmployee, employeeNode.getReference());
Assertions.assertFalse(employeeNode.isMarked());
```

同样，当调用`compareAndSet`方法时，如果字段被更新，我们得到`true`,如果更新失败，我们得到`false`。

### 4.6.`weakCompareAndSet()`

**`weakCompareAndSet`方法应该是`compareAndSet`方法的较弱版本。**也就是说，它不像`compareAndSet`那样提供强大的内存排序保证。此外，在硬件级别获得独占访问可能会错误地失败。

这是`weakCompareAndSet`方法的规范。然而，**目前，`weakCompareAndSet`简单地被[称为`compareAndSet`方法的幕后](https://web.archive.org/web/20221107200852/https://github.com/openjdk/jdk/blob/927a7287b70e8526435ff018d8f0504ebe9bbf94/src/java.base/share/classes/java/util/concurrent/atomic/AtomicMarkableReference.java#L126)。**所以，它们有着同样强大的实现。

即使这两种方法现在有相同的实现，我们也应该根据它们的规范来使用它们。因此，**我们应该把`weakCompareAndSet`看成一个弱原子**。

在某些平台和环境下，弱原子可能更便宜。例如，如果我们要在一个循环中执行一个`compareAndSet`，使用弱版本可能是一个更好的主意。在这种情况下，我们最终会在循环中更新状态，所以虚假的失败不会影响程序的正确性。

底线是，弱原子在某些特定的用例中是有用的，因此，并不适用于所有可能的场景。所以，当有疑问时，选择更强的

### 4.7.`attemptMark()`

最后，我们有`attemptMark`方法。它检查当前的`reference`是否等于作为参数发送的预期的`reference`。如果它们匹配，它自动将标记的值设置为给定的更新值:

```java
Employee employee = new Employee(123, "Mike");
AtomicMarkableReference<Employee> employeeNode = new AtomicMarkableReference<>(employee, true);

Assertions.assertTrue(employeeNode.attemptMark(employee, false));
Assertions.assertFalse(employeeNode.isMarked());
```

**需要注意的是，即使预期值和当前值`reference`相等，这种方法也可能错误地失败。因此，我们要注意方法执行**返回的`boolean`。

如果`mark`更新成功，结果为`true`，否则为`false`。然而，当当前的`reference`等于预期的`reference`时，重复调用将修改`mark`值。因此，建议**在`while`循环结构**中使用该方法。

此故障可能是由`attemptMark`方法更新字段所使用的底层比较和交换(CAS)算法导致的。如果我们有多个线程试图使用 CAS 更新同一个值，其中一个会设法更改该值，而其他线程会收到更新失败的通知。

## 5.结论

在这个快速指南中，我们学习了`AtomicMarkableReference`类是如何实现的。此外，我们发现了如何通过遍历该类的公共 API 方法来自动更新其属性。

和往常一样，GitHub 上有更多的例子和文章的完整源代码。