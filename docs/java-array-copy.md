# 如何在 Java 中复制数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-array-copy>

## 1。概述

在这个快速教程中，我们将讨论 Java 中不同的数组复制方法。数组复制看起来似乎是一个微不足道的任务，但是如果不小心的话，它会导致意想不到的结果和程序行为。

## 2。`System`班

让我们从核心 Java 库开始，`System.arrayCopy().`这将一个数组从一个源数组复制到一个目的数组，开始从源位置到目标位置的复制动作，直到指定的长度。

复制到目标数组的元素数等于指定的长度。它提供了一种将数组的子序列复制到另一个数组的简单方法。

如果任何一个数组参数为`null,`，它将抛出一个`NullPointerException.` ，如果任何一个整数参数为负或超出范围，它将抛出一个`IndexOutOfBoundException`。

让我们看一个使用`java.util.System`类将一个完整数组复制到另一个数组的例子:

```java
int[] array = {23, 43, 55};
int[] copiedArray = new int[3];

System.arraycopy(array, 0, copiedArray, 0, 3);
```

此方法采用以下参数:源数组、要从源数组复制的起始位置、目标数组、目标数组中的起始位置以及要复制的元素数量。

让我们看一下将子序列从源阵列复制到目标阵列的另一个示例:

```java
int[] array = {23, 43, 55, 12, 65, 88, 92};
int[] copiedArray = new int[3];

System.arraycopy(array, 2, copiedArray, 0, 3); 
```

```java
assertTrue(3 == copiedArray.length);
assertTrue(copiedArray[0] == array[2]);
assertTrue(copiedArray[1] == array[3]);
assertTrue(copiedArray[2] == array[4]); 
```

## 3。`Arrays`班

`Arrays`类还提供了多个重载方法来将一个数组复制到另一个数组。在内部，它使用我们之前研究过的由`System`类提供的相同方法。它主要提供了两种方法，`copyOf(…)`和`copyRangeOf(…)`。

先来看`copyOf` `:`

```java
int[] array = {23, 43, 55, 12};
int newLength = array.length;

int[] copiedArray = Arrays.copyOf(array, newLength); 
```

值得注意的是，`Arrays`类使用 `Math.min(…)`来选择源数组长度的最小值，并使用新长度参数的值来确定结果数组的大小。

除了源数组参数之外，`Arrays.copyOfRange()`接受两个参数，`from'`和`to',` 。得到的数组包括'`from'`索引，但不包括`‘to'`索引:

```java
int[] array = {23, 43, 55, 12, 65, 88, 92};

int[] copiedArray = Arrays.copyOfRange(array, 1, 4); 
```

```java
assertTrue(3 == copiedArray.length);
assertTrue(copiedArray[0] == array[1]);
assertTrue(copiedArray[1] == array[2]);
assertTrue(copiedArray[2] == array[3]);
```

如果应用于非原始对象类型的数组，这两种方法**都会对对象进行浅层复制**:

```java
Employee[] copiedArray = Arrays.copyOf(employees, employees.length);

employees[0].setName(employees[0].getName() + "_Changed");

assertArrayEquals(copiedArray, array);
```

因为结果是浅层复制，所以原始数组元素的雇员名的变化导致了复制数组的变化。

如果我们想做非原始类型的深层复制，我们可以选择在接下来的部分中描述的其他选项之一。

## 4。`Object.clone()`用阵列复制

`Object.clone()`继承自数组中的`Object`类。

首先，我们将使用 clone 方法复制一组基本类型:

```java
int[] array = {23, 43, 55, 12};

int[] copiedArray = array.clone(); 
```

这就是它有效的证明:

```java
assertArrayEquals(copiedArray, array);
array[0] = 9;

assertTrue(copiedArray[0] != array[0]);
```

上面的例子显示了它们在克隆后具有相同的内容，但是它们拥有不同的引用，所以其中一个的任何改变都不会影响另一个。

另一方面，如果我们使用相同的方法克隆一个非基元类型的数组，那么结果将会不同。

它创建了非原始类型数组元素的浅层副本，即使被封装对象的类实现了`Cloneable`接口并覆盖了来自`Object`类的`clone()`方法。

让我们来看一个例子:

```java
public class Address implements Cloneable {
    // ...

    @Override
    protected Object clone() throws CloneNotSupportedException {
         super.clone();
         Address address = new Address();
         address.setCity(this.city);

         return address;
    }
} 
```

我们可以通过创建一个新的地址数组并调用我们的`clone()`方法来测试我们的实现:

```java
Address[] addresses = createAddressArray();
Address[] copiedArray = addresses.clone();
addresses[0].setCity(addresses[0].getCity() + "_Changed"); 
```

```java
assertArrayEquals(copiedArray, addresses);
```

这个例子表明，原始数组或复制数组中的任何变化都会导致另一个数组中的变化，即使包含的对象是`Cloneable`。

## 5。使用`Stream` API

事实证明，我们也可以使用流 API 来复制数组:

```java
String[] strArray = {"orange", "red", "green'"};
String[] copiedArray = Arrays.stream(strArray).toArray(String[]::new); 
```

对于非原始类型，它还会进行对象的浅层复制。要了解更多关于`Java 8 Streams`的信息，我们可以从[这里开始](/web/20221012100327/https://www.baeldung.com/java-8-streams)。

## 6。外部库

`Apache Commons 3`提供了一个名为`SerializationUtils,`的实用程序类，它提供了一个`clone(…)`方法。如果我们需要对非原始类型的数组进行深度复制，这将非常有用。这里可以下载[，它的 Maven 依赖关系是:](https://web.archive.org/web/20221012100327/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22)

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency> 
```

让我们来看一个测试案例:

```java
public class Employee implements Serializable {
    // fields
    // standard getters and setters
}

Employee[] employees = createEmployeesArray();
Employee[] copiedArray = SerializationUtils.clone(employees); 
```

```java
employees[0].setName(employees[0].getName() + "_Changed");
assertFalse(
  copiedArray[0].getName().equals(employees[0].getName()));
```

这个类要求每个对象都应该实现`Serializable` 接口。就性能而言，它比手动为我们的对象图中的每个对象编写的克隆方法要慢。

## 7。结论

在本文中，我们讨论了在 Java 中复制数组的各种选项。

我们选择使用的方法主要取决于具体的场景。只要我们使用原始类型数组，我们就可以使用`System`和`Arrays`类提供的任何方法。性能上应该不会有什么差别。

对于非原始类型，如果我们需要做一个数组的深度拷贝，我们可以使用`SerializationUtils`或者显式地添加克隆方法到我们的类中。

和往常一样，本文展示的例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20221012100327/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-operations-advanced)