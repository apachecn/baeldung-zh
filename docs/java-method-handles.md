# Java 中的方法句柄

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-method-handles>

## 1。简介

在本文中，我们将探索 Java 7 中引入并在后续版本中得到增强的一个重要 API，即 `[java.lang.invoke.MethodHandles](https://web.archive.org/web/20220817005342/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/invoke/MethodHandles.html)`。

特别是，我们将学习什么是方法句柄，如何创建它们以及如何使用它们。

## 2。什么是方法句柄？

回到 API 文档中的定义:

> 方法句柄是对底层方法、构造函数、字段或类似的低级操作的类型化的、可直接执行的引用，具有可选的参数或返回值的转换。

更简单地说，**方法句柄是一种低级机制，用于查找、调整和调用方法**。

方法句柄是不可变的，没有可见的状态。

创建和使用`MethodHandle`，需要 4 个步骤:

*   创建查找
*   创建方法类型
*   查找方法句柄
*   调用方法句柄

### 2.1。方法句柄 vs 反射

引入方法句柄是为了与现有的`[java.lang.reflect](https://web.archive.org/web/20220817005342/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/package-summary.html)` API 一起工作，因为它们服务于不同的目的并具有不同的特性。

从性能的角度来看， **`MethodHandles` API 可以比反射 API 快得多，因为访问检查是在创建时进行的，而不是在执行时进行的**。如果存在安全管理器，这种差异会被放大，因为成员和类的查找会受到额外的检查。

然而，考虑到性能并不是任务唯一的适合性度量，我们还必须考虑到由于缺少诸如成员类枚举、可访问性标志检查等机制，`MethodHandles` API 更难使用。

即便如此，`MethodHandles` API 提供了修改方法、改变参数类型和改变它们顺序的可能性。

有了对`MethodHandles` API 的清晰定义和目标，我们现在可以开始使用它们，从查找开始。

## 3。创造`Lookup`

当我们想要创建方法句柄时，要做的第一件事是检索 lookup，它是负责为方法、构造函数和字段创建方法句柄的工厂对象，对 lookup 类是可见的。

通过`MethodHandles` API，可以用不同的访问模式创建 lookup 对象。

让我们创建一个提供对`public`方法访问的查找:

```
MethodHandles.Lookup publicLookup = MethodHandles.publicLookup();
```

然而，如果我们想访问`private`和`protected`方法，我们可以使用`lookup()`方法:

```
MethodHandles.Lookup lookup = MethodHandles.lookup();
```

## 4。创造一个`MethodType`

为了能够创建`MethodHandle`，lookup 对象需要其类型的定义，这是通过`MethodType`类实现的。

特别地， **a `MethodType`表示由方法句柄接受并返回的参数和返回类型，或者由方法句柄调用方**传递并期望的参数和返回类型。

`MethodType`的结构很简单，它由一个返回类型和适当数量的参数类型组成，这些参数类型必须在方法句柄和它的所有调用者之间正确匹配。

与`MethodHandle`一样，即使是`MethodType`的实例也是不可变的。

让我们看看如何定义一个指定一个`java.util.List`类作为返回类型和一个`Object`数组作为输入类型的`MethodType`:

```
MethodType mt = MethodType.methodType(List.class, Object[].class);
```

如果方法返回一个原始类型或者`void`作为它的返回类型，我们将使用代表这些类型的类(void.class，int.class …)。

让我们定义一个`MethodType`，它返回一个 int 值并接受一个`Object`:

```
MethodType mt = MethodType.methodType(int.class, Object.class);
```

我们现在可以开始创建`MethodHandle`。

## 5。`MethodHandle`寻找一个

一旦我们定义了我们的方法类型，为了创建一个`MethodHandle,` ,我们必须通过`lookup`或`publicLookup`对象找到它，同时提供基类和方法名。

特别是，查找工厂提供了一组[方法](https://web.archive.org/web/20220817005342/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/invoke/MethodHandles.Lookup.html#lookups)，允许我们根据方法的范围以适当的方式找到方法句柄。从最简单的场景开始，让我们探索主要的场景。

### 5.1。方法的方法句柄

使用`findVirtual()`方法允许我们为一个对象方法创建一个方法句柄。让我们基于`String`类的`concat()`方法创建一个:

```
MethodType mt = MethodType.methodType(String.class, String.class);
MethodHandle concatMH = publicLookup.findVirtual(String.class, "concat", mt);
```

### 5.2。静态方法的方法句柄

当我们想要访问一个静态方法时，我们可以使用`findStatic()`方法:

```
MethodType mt = MethodType.methodType(List.class, Object[].class);

MethodHandle asListMH = publicLookup.findStatic(Arrays.class, "asList", mt);
```

在这种情况下，我们创建了一个方法句柄，它将一个数组`Objects` 转换成一个数组`List`。

### 5.3。构造函数的方法句柄

使用`findConstructor()`方法可以访问构造函数。

让我们创建一个方法句柄，其行为类似于`Integer`类的构造函数，接受一个`String`属性:

```
MethodType mt = MethodType.methodType(void.class, String.class);

MethodHandle newIntegerMH = publicLookup.findConstructor(Integer.class, mt);
```

### 5.4。字段的方法句柄

使用方法句柄也可以访问字段。

让我们开始定义`Book` 类:

```
public class Book {

    String id;
    String title;

    // constructor

}
```

以方法句柄和声明的属性之间的直接访问可见性为前提，我们可以创建一个行为类似于 getter 的方法句柄:

```
MethodHandle getTitleMH = lookup.findGetter(Book.class, "title", String.class);
```

关于处理变量/字段的更多信息，请看一下 [Java 9 变量句柄揭秘](/web/20220817005342/https://www.baeldung.com/java-variable-handles)，其中我们讨论了 Java 9 中添加的[Java . lang . invoke . var handle](https://web.archive.org/web/20220817005342/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/invoke/VarHandle.html)API。

### 5.5。私有方法的方法句柄

在`[java.lang.reflect](https://web.archive.org/web/20220817005342/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/package-summary.html)` API 的帮助下，可以为私有方法创建一个方法句柄。

让我们开始向`Book`类添加一个`private`方法:

```
private String formatBook() {
    return id + " > " + title;
}
```

现在我们可以创建一个行为与`formatBook()`方法完全相同的方法句柄:

```
Method formatBookMethod = Book.class.getDeclaredMethod("formatBook");
formatBookMethod.setAccessible(true);

MethodHandle formatBookMH = lookup.unreflect(formatBookMethod);
```

## 6。调用方法句柄

一旦我们创建了方法句柄，下一步就是使用它们。特别是， *MethodHandle* 类提供了 3 种不同的方法来执行方法句柄:`invoke(),` `invokeWithArugments()`和`invokeExact()`。

让我们从`invoke`选项开始。

### 6.1。调用方法句柄

当使用`invoke()`方法时，我们强制参数(arity)的数量是固定的，但是我们允许执行参数和返回类型的转换和装箱/拆箱。

让我们看看如何将`invoke()`用于一个装箱的参数:

```
MethodType mt = MethodType.methodType(String.class, char.class, char.class);
MethodHandle replaceMH = publicLookup.findVirtual(String.class, "replace", mt);

String output = (String) replaceMH.invoke("jovo", Character.valueOf('o'), 'a');

assertEquals("java", output);
```

在这种情况下， *replaceMH* 需要`char`参数，但是`invoke()`在执行之前会对`Character`参数进行拆箱。

### 6.2。使用参数调用

使用`invokeWithArguments`方法调用方法句柄是三个选项中限制最少的。

事实上，除了参数和返回类型的转换和装箱/拆箱之外，它还允许变量 arity 调用。

实际上，这允许我们从`int`值的`array`开始创建`Integer`的`List`:

```
MethodType mt = MethodType.methodType(List.class, Object[].class);
MethodHandle asList = publicLookup.findStatic(Arrays.class, "asList", mt);

List<Integer> list = (List<Integer>) asList.invokeWithArguments(1,2);

assertThat(Arrays.asList(1,2), is(list));
```

### 6.3。调用精确的

如果我们想在执行方法句柄的方式上有更多的限制(参数的数量和类型)，我们必须使用`invokeExact()`方法。

事实上，它不提供对所提供的类的任何强制转换，并且需要固定数量的参数。

让我们看看如何使用方法句柄来`sum`两个`int`值:

```
MethodType mt = MethodType.methodType(int.class, int.class, int.class);
MethodHandle sumMH = lookup.findStatic(Integer.class, "sum", mt);

int sum = (int) sumMH.invokeExact(1, 11);

assertEquals(12, sum);
```

如果在这种情况下，我们决定向`invokeExact`方法传递一个不是`int`的数字，调用将导致`WrongMethodTypeException.`

## 7。使用数组

不仅仅是为了处理字段或对象，也是为了处理数组。事实上，使用`asSpreader()` API，可以创建一个数组扩展方法句柄。

在这种情况下，方法句柄接受一个数组参数，将其元素作为位置参数展开，并可以选择数组的长度。

让我们看看如何扩展方法句柄来检查数组中的元素是否相等:

```
MethodType mt = MethodType.methodType(boolean.class, Object.class);
MethodHandle equals = publicLookup.findVirtual(String.class, "equals", mt);

MethodHandle methodHandle = equals.asSpreader(Object[].class, 2);

assertTrue((boolean) methodHandle.invoke(new Object[] { "java", "java" }));
```

## 8。增强方法句柄

一旦我们定义了一个方法句柄，就可以通过将方法句柄绑定到一个参数来增强它，而不需要实际调用它。

例如，在 Java 9 中，这种行为用于优化`String`连接。

让我们看看如何执行连接，将后缀绑定到我们的`concatMH`:

```
MethodType mt = MethodType.methodType(String.class, String.class);
MethodHandle concatMH = publicLookup.findVirtual(String.class, "concat", mt);

MethodHandle bindedConcatMH = concatMH.bindTo("Hello ");

assertEquals("Hello World!", bindedConcatMH.invoke("World!"));
```

## 9。Java 9 增强功能

在 Java 9 中，`MethodHandles` API 做了一些改进，目的是让它更容易使用。

这些改进影响了 3 个主要主题:

*   **查找函数**——允许从不同的上下文中查找类，并支持接口中的非抽象方法
*   **参数处理**–改进参数折叠、参数收集和参数传播功能
*   **额外的组合**–添加循环(`loop`、`whileLoop,` 、`doWhileLoop…`)和更好的异常处理支持`tryFinally`

这些变化并没有带来额外的好处:

*   增加了 JVM 编译器优化
*   实例化简化
*   使用`MethodHandles` API 时启用精度

增强的细节可以在 [`MethodHandles` API Javadoc](https://web.archive.org/web/20220817005342/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/invoke/MethodHandles.html) 获得。

## 10。结论

在本文中，我们介绍了`MethodHandles` API，它们是什么以及我们如何使用它们。

我们还讨论了它与反射 API 的关系，因为方法句柄允许低级操作，所以最好避免使用它们，除非它们完全符合工作的范围。

和往常一样，本文的完整源代码可以在 Github 的[上找到。](https://web.archive.org/web/20220817005342/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9)