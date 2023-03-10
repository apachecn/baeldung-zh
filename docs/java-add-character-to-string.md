# 在字符串的给定位置添加一个字符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-add-character-to-string>

## 1.介绍

在这个快速教程中，我们将**演示如何在`Java`的`String`中的任意给定位置添加一个角色。**

我们将展示一个简单函数的三个实现，该函数接受原始的`String,` a 字符和我们需要添加它的位置。

由于字符串类是最终的和不可变的，函数应该返回一个新的添加了字符的`String`。

## 2.使用字符`Array`

这里的想法是创建一个新的字符数组，并复制给定位置之前的原始`String`中的字符。

之后，我们把新字符放在这个位置，并把原来的`String`中的剩余字符复制到新数组的后续位置。

最后，我们从该数组中构造所需的`String`。

```java
public String addChar(String str, char ch, int position) {
    int len = str.length();
    char[] updatedArr = new char[len + 1];
    str.getChars(0, position, updatedArr, 0);
    updatedArr[position] = ch;
    str.getChars(position, len, updatedArr, position + 1);
    return new String(updatedArr);
}
```

与其他两种方法相比，这是一种低层次的设计方法，为我们提供了最大的灵活性。

## 3.使用`substring`方法

一种更简单、更高级的方法是使用`String`类的 `substring()`方法。它通过连接以下内容来准备`String`:

1.  原`String`位置前的子串
2.  新角色
3.  原`String`位置后的子串

```java
public String addChar(String str, char ch, int position) {
    return str.substring(0, position) + ch + str.substring(position);
}
```

虽然上面的代码可读性更好，但是它有一个缺点，那就是它创建了许多临时对象来确定结果。因为`String`是一个不可变的类，每次调用它的`substring()`方法都会创建一个新的`String`实例。

最后，当我们连接这些部分时，编译器会创建一个`StringBuilder`对象来一个接一个地追加它们。每个`String`和`StringBuilder`对象为其内部字符数组分配独立的内存位置。

这个实现还需要将所有字符从一个数组复制到另一个数组三次。

如果我们需要多次调用该方法，临时对象可能会填满堆内存，这将会非常频繁地触发 GC。这也会在一定程度上影响性能。

## 4.使用`StringBuilder`

`StringBuilder`是由`Java`库提供的一个实用程序类，用于以多种方式构造和操作`String`对象。

我们可以使用`StringBuilder`类的`insert()`方法实现相同的功能:

```java
public String addChar(String str, char ch, int position) {
    StringBuilder sb = new StringBuilder(str);
    sb.insert(position, ch);
    return sb.toString();
}
```

上面的代码只需要创建一个单独的`StringBuilder`对象来在该位置插入字符。它分配与原来的`String`相同的内存量，但是为了给新字符创建一个位置，底层数组将下一个字符移动 1 个位置。

虽然使用一个`StringBuilder`可能会慢一些，但是它没有初始化临时对象的内存负担。**我们也最终得到了简单易读的代码。**

## 5.结论

在本文中，我们关注了几种在`Java`中的`String`对象中添加字符的方法。我们已经看到，使用字符数组的实现提供了最好的性能，而使用`substring`方法提供了一种可读性更好的方法。

实现该解决方案的首选方式是使用**的`StringBuilder`类——因为它简单，不容易出错，并且提供良好和稳定的性能**。

像往常一样，上述教程的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221206170156/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms-2)