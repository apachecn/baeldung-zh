# 去神秘化的 Java 9 变量句柄

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-variable-handles>

## 1。简介

Java 9 为开发人员带来了许多有用的新特性。

其中之一是[`java.lang.invoke.VarHandle`](https://web.archive.org/web/20220823220128/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/invoke/VarHandle.html)API——代表变量句柄——我们将在本文中探讨。

## 2。什么是可变句柄？

一般来说，**变量句柄只是对变量**的类型化引用。变量可以是数组元素、实例或类的静态字段。

**`VarHandle`类在特定条件下提供对变量的读写访问。**

`VarHandles`是不可变的，没有可见的状态。更重要的是，它们不能被细分。

每个`VarHandle`都有:

*   通用类型`T`，它是由这个`VarHandle`表示的每个变量的类型
*   坐标类型列表`CT`，是坐标表达式的类型，允许定位该`VarHandle`引用的变量

坐标类型列表可能为空。

`VarHandle`的目标是定义一个标准，用于对字段和数组元素调用 **`java` `.util.concurrent.atomic`** 和 **`sun.misc.Unsafe`** 操作的**等价物。**

这些操作大多是原子操作或有序操作，例如原子字段增量。

## 3。创建变量句柄

要使用`VarHandle`，我们首先需要有变量。

让我们声明一个简单的类，它包含我们将在示例中使用的不同类型的变量:

```java
public class VariableHandlesUnitTest {
    public int publicTestVariable = 1;
    private int privateTestVariable = 1;
    public int variableToSet = 1;
    public int variableToCompareAndSet = 1;
    public int variableToGetAndAdd = 0;
    public byte variableToBitwiseOr = 0;
}
```

### 3.1.指导方针和惯例

**按照惯例，我们应该将`VarHandle` s 声明为`[static final](https://web.archive.org/web/20220823220128/http://gee.cs.oswego.edu/dl/html/j9mm.html)`字段，并在静态块中显式初始化它们。另外，我们通常使用相应字段名的大写版本作为它们的名称。**

例如，下面是 Java 本身如何在内部使用`VarHandle`来实现 [`AtomicReference`](https://web.archive.org/web/20220823220128/https://github.com/openjdk/jdk14u/blob/d6d8d4d931b06d919e7688c6106f489a173d8608/src/java.base/share/classes/java/util/concurrent/atomic/AtomicReference.java#L51) :

```java
private volatile V value;
private static final VarHandle VALUE;
static {
    try {
        MethodHandles.Lookup l = MethodHandles.lookup();
        VALUE = l.findVarHandle(AtomicReference.class, "value", Object.class);
    } catch (ReflectiveOperationException e) {
        throw new ExceptionInInitializerError(e);
    }
}
```

很多时候，我们在使用`VarHandle` s 的时候可以使用相同的模式。

现在我们知道了这一点，让我们继续，看看如何在实践中使用它们。

### 3.2。公共变量的变量句柄

**现在我们可以使用 `findVarHandle()`方法**为我们的`publicTestVariable`获得一个`VarHandle`:

```java
VarHandle PUBLIC_TEST_VARIABLE = MethodHandles
  .lookup()
  .in(VariableHandlesUnitTest.class)
  .findVarHandle(VariableHandlesUnitTest.class, "publicTestVariable", int.class);

assertEquals(1, PUBLIC_TEST_VARIABLE.coordinateTypes().size());
assertEquals(VariableHandlesUnitTest.class, PUBLIC_TEST_VARIABLE.coordinateTypes().get(0));
```

我们可以看到这个`VarHandle`的`coordinateTypes`属性不是空的，并且有一个元素，就是我们的`VariableHandlesUnitTest`类。

### 3.3。私有变量的变量句柄

如果我们有一个私有成员，并且我们需要这样一个变量的变量句柄，**我们可以使用`privateLookupIn()`方法**来获得它:

```java
VarHandle PRIVATE_TEST_VARIABLE = MethodHandles
  .privateLookupIn(VariableHandlesUnitTest.class, MethodHandles.lookup())
  .findVarHandle(VariableHandlesUnitTest.class, "privateTestVariable", int.class);

assertEquals(1, PRIVATE_TEST_VARIABLE.coordinateTypes().size());
assertEquals(VariableHandlesUnitTest.class, PRIVATE_TEST_VARIABLE.coordinateTypes().get(0));
```

这里，我们选择了`privateLookupIn()`方法，它比普通的`lookup()`具有更广泛的访问权限。这允许我们访问`private`、`public`或`protected`变量。

在 Java 9 之前，这个操作的等价 API 是来自`Reflection` API 的`Unsafe`类和 *setAccessible()* 方法。

然而，这种方法有其缺点。例如，它只对变量的特定实例有效。

在这种情况下，`VarHandle`是更好、更快的解决方案。

### 3.4。数组的变量句柄

我们可以使用前面的语法来获取数组字段。

然而，我们也可以获得特定类型数组的`VarHandle`:

```java
VarHandle arrayVarHandle = MethodHandles.arrayElementVarHandle(int[].class);

assertEquals(2, arrayVarHandle.coordinateTypes().size());
assertEquals(int[].class, arrayVarHandle.coordinateTypes().get(0));
```

我们现在可以看到，这样的`VarHandle`有两种坐标类型`int`和 `[]`，它们代表了一组`int`图元。

## 4。调用`VarHandle`方法

**大多数`VarHandle`方法期望类型为** `**Object.**` 的可变数量的参数，使用`Object…`作为参数会禁用静态参数检查。

所有的参数检查都在运行时完成。此外，不同的方法期望有不同数量的不同类型的参数。

如果我们没有给出适当数量的适当类型的参数，方法调用将抛出一个`WrongMethodTypeException`。

例如，`get()`将期望至少一个参数，这有助于定位变量，但是`set()`期望多一个参数，这是分配给变量的值。

## 5。变量句柄访问模式

一般来说，`VarHandle`类的所有方法都属于五种不同的访问模式。

让我们在接下来的小节中逐一介绍它们。

### 5.1。读取权限

具有读取访问级别的方法允许在指定的内存排序效果下获取变量的值。这种访问方式有几种方法，如:`get()`、`getAcquire()`、`getVolatile()`和`getOpaque()`。

我们可以很容易地在我们的`VarHandle`上使用`get()`方法:

```java
assertEquals(1, (int) PUBLIC_TEST_VARIABLE.get(this));
```

`get()`方法只接受`CoordinateTypes`作为参数，所以在我们的例子中我们可以简单地使用`this`。

### 5.2。写访问

具有写访问级别的方法允许我们在特定的内存排序效果下设置变量的值。

类似于具有读访问权限的方法，我们有几个具有写访问权限的方法:`set()`、`setOpaque()`、`setVolatile()`和`setRelease()`。

我们可以在我们的`VarHandle`上使用`set()`方法:

```java
VARIABLE_TO_SET.set(this, 15);
assertEquals(15, (int) VARIABLE_TO_SET.get(this));
```

`set()`方法需要至少两个参数。第一个将帮助定位变量，而第二个是要设置给变量的值。

### 5.3。原子更新访问

具有此访问级别的方法可用于自动更新变量值。

让我们使用`compareAndSet()`方法来看看效果:

```java
VARIABLE_TO_COMPARE_AND_SET.compareAndSet(this, 1, 100);
assertEquals(100, (int) VARIABLE_TO_COMPARE_AND_SET.get(this));
```

除了`CoordinateTypes`，`compareAndSet()`方法还有两个额外的值:`oldValue`和`newValue`。如果变量值等于`oldVariable`，则该方法设置变量值，否则保持不变。

### 5.4。数字原子更新访问

这些方法允许在特定的存储器排序效果下执行诸如`getAndAdd`()的数字操作。

让我们看看如何使用`VarHandle`来执行原子操作:

```java
int before = (int) VARIABLE_TO_GET_AND_ADD.getAndAdd(this, 200);

assertEquals(0, before);
assertEquals(200, (int) VARIABLE_TO_GET_AND_ADD.get(this));
```

这里，`getAndAdd()`方法首先返回变量的值，然后加上提供的值。

### 5.5。逐位原子更新访问

具有这种访问的方法允许我们在特定的内存排序效果下自动执行按位操作。

让我们看一个使用`getAndBitwiseOr()`方法的例子:

```java
byte before = (byte) VARIABLE_TO_BITWISE_OR.getAndBitwiseOr(this, (byte) 127);

assertEquals(0, before);
assertEquals(127, (byte) VARIABLE_TO_BITWISE_OR.get(this));
```

这个方法将获得我们的变量的值，并对它执行按位 OR 运算。

**如果方法调用无法将方法要求的访问模式与变量允许的访问模式匹配，方法调用将抛出一个 `IllegalAccessException`。**

例如，如果我们试图对一个`final`变量使用一个`set()`方法，就会发生这种情况。

## 6。记忆排序效应

我们之前提到过 **`VarHandle`方法允许在特定的内存排序效果下访问变量。**

对于大多数方法，有 4 种记忆排序效应:

*   对于 32 位以下的引用和原语，读写保证了按位原子性。此外，它们没有对其他特征施加排序约束。
*   `Opaque`操作是逐位原子的，并且相对于对同一变量的访问是连贯有序的。
*   `Acquire`和`Release`操作服从`Opaque`属性。此外，`Acquire`读取将仅在匹配`Release`模式写入后排序。
*   `Volatile`操作彼此之间是完全有序的。

记住**访问模式将覆盖之前的内存排序效果**非常重要。这意味着，例如，如果我们使用`get()`，这将是一个普通的读操作，即使我们将变量声明为`volatile`。

因此，开发人员在使用`VarHandle`操作时必须非常小心。

## 7 .**。结论**

在本教程中，我们介绍了变量句柄以及如何使用它们。

这个主题非常复杂，因为变量句柄旨在允许低级操作，除非必要，否则不应该使用它们。

与往常一样，代码示例可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220823220128/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9-new-features)