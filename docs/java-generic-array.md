# 在 Java 中创建泛型数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-generic-array>

## 1.介绍

我们可能希望使用[数组](/web/20220929040025/https://www.baeldung.com/java-arrays-guide)作为支持[泛型](/web/20220929040025/https://www.baeldung.com/java-generics)的类或函数的一部分，但是由于 Java 处理泛型的方式，这可能很困难。

在本教程中，我们将讨论在数组中使用泛型的挑战。然后我们将创建一个泛型数组的例子。

最后，我们将看到 Java API 是如何解决类似问题的。

## 2.使用泛型数组时的注意事项

数组和泛型的一个重要区别是它们如何执行类型检查。具体来说，数组在运行时存储并检查类型信息。然而，泛型在编译时检查类型错误，而[在运行时没有类型信息。](/web/20220929040025/https://www.baeldung.com/java-generics#type-erasure)

Java 的语法表明我们也许能够创建一个新的通用数组:

```java
T[] elements = new T[size];
```

但是如果我们尝试这样做，我们会得到一个编译错误。

要了解原因，让我们考虑以下情况:

```java
public <T> T[] getArray(int size) {
    T[] genericArray = new T[size]; // suppose this is allowed
    return genericArray;
}
```

当一个未绑定的泛型类型`T`解析为`Object,`时，我们在运行时的方法将是:

```java
public Object[] getArray(int size) {
    Object[] genericArray = new Object[size];
    return genericArray;
}
```

如果我们调用我们的方法并将结果存储在一个`String`数组中:

```java
String[] myArray = getArray(5);
```

代码可以很好地编译，但是在运行时会以`ClassCastException`失败。这是因为我们刚刚给一个`String[]`引用分配了一个`Object[]`。具体来说，编译器的隐式强制转换无法将`Object[]`转换为我们需要的类型` String[]`。

尽管我们不能直接初始化泛型数组，但是如果调用代码提供了精确的信息类型，还是有可能实现相同的操作。

## 3.创建通用数组

对于我们的例子，让我们考虑一个有界堆栈数据结构，`MyStack`，其中容量固定为某个大小。因为我们希望栈可以与任何类型一起工作，所以一个合理的实现选择是泛型数组。

首先，我们将创建一个字段来存储堆栈的元素，这是一个类型为`E`的通用数组:

```java
private E[] elements;
```

然后我们将添加一个构造函数:

```java
public MyStack(Class<E> clazz, int capacity) {
    elements = (E[]) Array.newInstance(clazz, capacity);
}
```

注意**如何使用 `java.lang.reflect.Array#newInstance`来初始化我们的通用数组**，它需要两个参数。第一个参数指定新数组中对象的类型。第二个参数指定为数组创建多少空间。由于`Array#newInstance`的结果属于`Object`类型，我们需要将其转换为`E[]`来创建我们的泛型数组。

我们还应该注意命名类型参数`clazz,`而不是`class,`的约定，T1 是 Java 中的保留字。

## 4.考虑`ArrayList`

### 4.1.使用`ArrayList`代替数组

使用泛型`ArrayList`代替泛型数组通常更容易。让我们看看如何改变`MyStack`来使用`ArrayList`。

首先，我们将创建一个字段来存储我们的元素:

```java
private List<E> elements;
```

然后，在我们的堆栈构造函数中，我们可以用初始容量初始化`ArrayList`:

```java
elements = new ArrayList<>(capacity);
```

这使得我们的类更简单，因为我们不必使用反射。此外，在创建堆栈时，我们不需要传入一个类文本。由于我们可以设置一个`ArrayList`的初始容量，我们可以获得与数组相同的好处。

因此，我们只需要在极少数情况下或者当我们与一些需要数组的外部库接口时构造泛型数组。

### 4.2.`ArrayList`实施

有趣的是， **`ArrayList`本身就是用泛型数组实现的。**让我们来窥探一下`ArrayList`内部的情况。

首先，让我们看看列表元素字段:

```java
transient Object[] elementData;
```

注意`ArrayList`使用`Object`作为元素类型。由于我们的泛型直到运行时才知道，`Object`被用作任何类型的超类。

值得注意的是，`ArrayList`中几乎所有的操作都可以使用这个通用数组，因为它们不需要向外界提供强类型数组(除了一个方法，`toArray).`

## 5.从集合构建数组

### 5.1.链接列表示例

让我们看看在 Java Collections API 中使用泛型数组，我们将从一个集合中构建一个新数组。

首先，我们将创建一个带有类型参数`String,`的新`LinkedList`，并向其中添加项目:

```java
List<String> items = new LinkedList();
items.add("first item");
items.add("second item"); 
```

然后，我们将为刚刚添加的项目构建一个数组:

```java
String[] itemsAsArray = items.toArray(new String[0]);
```

建我们的阵， **`List`。`toArray`方法需要一个输入数组。**它纯粹使用这个数组来获取类型信息，以创建正确类型的返回数组。

在上面的例子中，我们使用`new String[0]`作为输入数组来构建结果`String`数组。

### 5.2.`LinkedList.toArray`实施

让我们看一看`LinkedList.toArray`内部，看看它是如何在 Java JDK 中实现的。

首先，我们来看看方法签名:

```java
public <T> T[] toArray(T[] a)
```

然后，我们将了解如何在需要时创建新阵列:

```java
a = (T[])java.lang.reflect.Array.newInstance(a.getClass().getComponentType(), size);
```

注意它是如何利用`Array#newInstance`来构建一个新数组的，就像我们之前的堆栈例子一样。我们还可以看到，参数`a`用于向`Array#newInstance. `提供一个类型。最后，来自`Array#newInstance`的结果被转换为`T[]`以创建一个通用数组。

## 6.从流创建数组

Java Streams API 允许我们从流中的项目创建数组。有几个陷阱需要注意，以确保我们生成正确类型的数组。

### 6.1.使用`toArray`

我们可以很容易地将项目从 Java 8 `Stream`转换成一个数组:

```java
Object[] strings = Stream.of("A", "AAA", "B", "AAB", "C")
  .filter(string -> string.startsWith("A"))
  .toArray();

assertThat(strings).containsExactly("A", "AAA", "AAB"); 
```

然而，我们应该注意到，基本的`toArray`函数为我们提供了一个`Object`数组，而不是一个`String`数组:

```java
assertThat(strings).isNotInstanceOf(String[].class);
```

正如我们前面看到的，每个数组的精确类型是不同的。由于`Stream`中的类型是泛型，库无法在运行时[推断类型](/web/20220929040025/https://www.baeldung.com/java-type-erasure)。

### 6.2.使用`toArray`重载获取类型化数组

常见的集合类方法使用反射来构造特定类型的数组，而 Java Streams 库使用函数式方法。我们可以传入一个 lambda 或方法引用，当`Stream`准备好填充数组时，它会创建一个正确大小和类型的数组:

```java
String[] strings = Stream.of("A", "AAA", "B", "AAB", "C")
  .filter(string -> string.startsWith("A"))
  .toArray(String[]::new);

assertThat(strings).containsExactly("A", "AAA", "AAB");
assertThat(strings).isInstanceOf(String[].class);
```

我们传递的方法是一个`IntFunction,`，它接受一个整数作为输入，并返回该大小的新数组。这正是`String[]`的构造函数所做的，所以我们可以用[方法引用](/web/20220929040025/https://www.baeldung.com/java-method-references) `String[]::new`。

### 6.3.具有自己的类型参数的泛型

现在让我们假设我们想要将流中的值转换成一个对象，这个对象本身有一个类型参数，比如说`List`或`Optional`。也许我们有一个我们想要调用的 API，它将`Optional<String>[]`作为它的输入。

声明这种数组是有效的:

```java
Optional<String>[] strings = null;
```

我们也可以通过使用`map`方法轻松地将我们的`Stream<String>`转换成`Stream<Optional<String>>`:

```java
Stream<Optional<String>> stream = Stream.of("A", "AAA", "B", "AAB", "C")
  .filter(string -> string.startsWith("A"))
  .map(Optional::of);
```

然而，如果我们试图构造我们的数组，我们会再次得到一个编译器错误:

```java
// compiler error
Optional<String>[] strings = new Optional<String>[1];
```

幸运的是，这个例子和我们之前的例子有所不同。其中`String[]`不是`Object[]`的子类，`Optional[]`实际上是与`Optional<String>[]`相同的运行时类型。换句话说，这是一个我们可以通过类型转换来解决的问题:

```java
Stream<Optional<String>> stream = Stream.of("A", "AAA", "B", "AAB", "C")
  .filter(string -> string.startsWith("A"))
  .map(Optional::of);
Optional<String>[] strings = stream
  .toArray(Optional[]::new);
```

这段代码可以编译并运行，但是会给我们一个`unchecked assignment` 警告。我们需要在我们的方法中添加一个`SuppressWarnings`来解决这个问题:

```java
@SuppressWarnings("unchecked")
```

### 6.4.使用助手功能

如果我们想避免将`SuppressWarnings`添加到代码中的多个位置，并希望记录从原始类型创建泛型数组的方式，我们可以编写一个助手函数:

```java
@SuppressWarnings("unchecked")
static <T, R extends T> IntFunction<R[]> genericArray(IntFunction<T[]> arrayCreator) {
    return size -> (R[]) arrayCreator.apply(size);
}
```

此函数将生成原始类型数组的函数转换为承诺生成我们需要的特定类型数组的函数:

```java
Optional<String>[] strings = Stream.of("A", "AAA", "B", "AAB", "C")
  .filter(string -> string.startsWith("A"))
  .map(Optional::of)
  .toArray(genericArray(Optional[]::new));
```

这里不需要取消未检查的赋值警告。

但是，我们应该注意，可以调用这个函数来执行到更高类型的类型转换。例如，如果我们的流包含类型为`List<String>`的对象，我们可能会错误地调用`genericArray`来产生一个`ArrayList<String>`数组:

```java
ArrayList<String>[] lists = Stream.of(singletonList("A"))
  .toArray(genericArray(List[]::new));
```

这可以编译，但是会抛出一个`ClassCastException,` ,因为`ArrayList[]`不是`List[].`的子类。但是编译器会产生一个未检查的赋值警告，所以很容易发现。

## 7.结论

在本文中，我们研究了数组和泛型之间的区别。然后我们看了一个创建泛型数组的例子，展示了使用`ArrayList`可能比使用泛型数组更容易。我们还讨论了集合 API 中泛型数组的使用。

最后，我们学习了如何从 Streams API 生成数组，以及如何创建使用类型参数的类型数组。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20220929040025/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-guides)