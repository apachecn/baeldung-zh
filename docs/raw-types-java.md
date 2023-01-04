# Java 中的原始类型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/raw-types-java>

## 1.介绍

在这个快速教程中，我们将看看原始类型，它们是什么，以及为什么我们应该避免它们。

## 2.原始类型

原始类型是没有类型参数的通用接口或类的名称

```
List list = new ArrayList(); // raw type
```

而不是:

```
List<Integer> listIntgrs = new ArrayList<>(); // parameterized type
```

`List<Integer>`是接口`List<E>`的一个`parameterized type` ，而`List`是接口`List<E>`的一个`raw type`。

当与非泛型遗留代码交互时，原始类型会很有用。

否则，尽管如此，这是令人气馁的。这是因为:

1.  他们不善于表达
2.  它们缺乏类型安全，并且
3.  问题是在运行时观察到的，而不是在编译时

## 3.无意义的

原始类型不像参数化类型那样记录和解释自己。

我们可以很容易地推断出参数化类型`List<String>`是一个包含`String`的列表。然而，原始类型缺乏这种清晰度，使得很难使用它和它的 API 方法。

让我们看看`List`接口中方法`get(int index)` 的签名，以便更好地理解这一点:

```
/**
 * Returns the element at the specified position in this list.
 *
 * @param index index of the element to return
 * @return the element at the specified position in this list
 * @throws IndexOutOfBoundsException if the index is out of range
 *         (<tt>index < 0 || index >= size()</tt>)
 */
E get(int index);
```

方法`get(int index)`在参数化类型`List<String>`中的位置`index`返回一个`String`。

然而，对于原始类型`List`，它返回一个`Object`。因此，我们需要付出额外的努力来检查和识别原始类型`List`和**中的元素类型，并添加适当的类型铸件。**这可能会在运行时引入错误，因为原始类型**不是类型安全的**。

## 4.不是类型安全的

我们得到了原始类型的前泛型行为。因此，原始类型`List`接受`Object`，**可以保存任何数据类型**的元素。当我们混合参数化类型和原始类型时，这会导致类型安全问题。

让我们通过创建一些代码来了解这一点，这些代码在将一个`List<String>`传递给一个接受原始类型`List`并向其添加一个`Integer`的方法之前实例化它:

```
public void methodA() {
    List<String> parameterizedList = new ArrayList<>();
    parameterizedList.add("Hello Folks");
    methodB(parameterizedList);
}

public void methodB(List rawList) { // raw type!
    rawList.add(1);
}
```

代码被编译(带有警告)，并且在执行时`Integer`被添加到原始类型`List` 中。作为参数**传递的`List<String>`现在包含一个`String`和一个`Integer`。**

由于使用了原始类型，编译器会打印出一条警告:

```
Note: RawTypeDemo.java uses unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.
```

## 5.运行时的问题

原始类型缺乏类型安全会导致运行时出现异常。

让我们修改前面的例子，这样在调用`methodB`之后，`methodA`可以获得`List<String>`的索引位置 1 的元素:

```
public void methodA() {
    List<String> parameterizedList = new ArrayList<>();
    parameterizedList.add("Hello Folks");
    methodB(parameterizedList);
    String s = parameterizedList.get(1);
}

public void methodB(List rawList) {
    rawList.add(1);
}
```

代码被编译(带有相同的警告)并在执行时抛出一个`ClassCastException`。当方法`get(int index)`返回一个`Integer`时就会发生这种情况，该值不能赋给类型为`String`的变量:

```
Exception in thread "main" java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String
```

## 6.结论

原始类型很难处理，会在我们的代码中引入错误。

使用它们会导致灾难性的后果，不幸的是，这些灾难大多发生在运行时。

在 GitHub 上查看本教程[中的所有片段。](https://web.archive.org/web/20220908212421/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-generics)