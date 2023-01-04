# Java 中的空类型是什么？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-null>

## 1.介绍

在 Java 的世界里，`null`类型无处不在，不遇到它很难使用这种语言。在大多数情况下，直觉上理解它代表着虚无或缺少某种东西就足以有效地编程。尽管如此，有时我们还是想深入挖掘，彻底理解主题。

在本教程中，我们将看看`null`类型是如何工作的，以及它与其他类型的关系。

## 2.什么是类型？

在我们回答关于`null`类型的具体问题之前，我们需要定义一个类型到底是什么。这不是一项容易的任务，因为有许多相互竞争的定义。对我们最有用的是值空间的定义。在这个定义中，**类型是由它可以保存的**的一组可能值定义的。

假设我们想要声明一个`boolean`变量:

```
boolean valid;
```

我们所做的是声明名为“valid”的变量将保存两个可能值之一:`true`或`false`。可能值的集合只有两个元素。如果我们想声明一个`int`变量，可能值的集合会大得多，但仍然定义明确:从-2^31 到 2^31-1.的每一个可能的数字

## 3.`null`是什么类型？

`null`是只有一个可能值的特殊类型。换句话说，可能值的集合只有一个元素。光是这个特点就让`null`型非常奇特。通常，变量的全部目的是它们可以假定不同的值。只有一个`null`引用，所以一个`null`类型的变量只能保存那个特定的引用。除了变量存在之外，它不会带来任何信息。

有一个特征使得`null`类型在我们使用它的时候是有用的。**`null`引用可以转换成任何其他引用类型。**这意味着我们可以把它当作一个特殊的文字，可以是任何非原始类型。在实践中，`null`引用扩展了这些类型的可能值的有效集合。

这解释了为什么我们可以将完全相同的`null`引用赋给完全不同引用类型的变量:

```
Integer age = null;
List<String> names = null;
```

这也解释了为什么我们不能将`null`的值赋给像`boolean`这样的原始类型的变量:

```
Boolean validReference = null // this works fine
boolean validPrimitive = null // this does not
```

这是因为`null`引用可以被转换为引用类型，但不能转换为原始类型。一个`boolean`变量的可能值集合总是有两个元素。

## 4.`null`作为方法参数

让我们来看看两个简单的方法，它们都接受一个参数，但类型不同:

```
void printMe(Integer number) {
  System.out.println(number);
}

void printMe(String string) {
  System.out.println(string);
}
```

由于 Java 中的多态性，我们可以这样调用这些方法:

```
printMe(6);
printMe("Hello");
```

编译器会理解我们引用的是什么方法。但是下面的语句会导致编译器错误:

```
printMe(null); // does not compile
```

**为什么？因为`null`可以被强制转换为`String`和`Integer –` ，编译器不知道选择哪个方法。**

## 5.`NullPointerException`

正如我们已经看到的，我们可以将`null`引用分配给一个引用类型的变量，即使`null`在技术上是一个不同的、独立的类型。**如果我们试图使用那个变量的某些属性，就好像它不是`null,`一样，我们将得到一个运行时异常—`NullPointerException`。**发生这种情况是因为`null`引用不是我们所引用的类型，也没有我们期望的属性:

```
String name = null;
name.toLowerCase(); // will cause exception at runtime
```

在 Java 14 之前，`NullPointerExceptions`很短，只是简单地指出错误发生在代码的哪一行。如果代码行很复杂，并且有一连串的调用，那么这些信息就不是有用的。然而，在 Java 14 中，我们可以依靠所谓的[有用的 NullPointerExceptions](/web/20221112101142/https://www.baeldung.com/java-14-nullpointerexception) 。

## 6.结论

在本文中，我们仔细研究了`null`类型是如何工作的。首先，我们定义了一个类型，然后我们发现`null`类型如何符合这个定义。最后，我们学习了如何将一个`null`引用转换成任何其他的引用类型，使它成为我们知道并使用的工具。