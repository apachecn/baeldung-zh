# 数组.深度等于

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-arrays-deepequals>

## 1.概观

在本教程中，我们将深入研究来自`Arrays`类的 **`deepEquals`方法的细节。我们会看到什么时候应该使用这个方法，我们会通过一些简单的例子。**

要了解更多关于`java.util.Arrays`类中不同方法的信息，请查看我们的[快速指南](/web/20221206163346/https://www.baeldung.com/java-util-arrays)。

## 2.目的

当我们想要检查两个嵌套或者多维数组之间的相等性时，我们应该使用`deepEquals`方法**。同样，当我们想要比较由用户定义的对象组成的两个数组时，正如我们将在后面看到的，我们必须覆盖`equals`方法。**

现在，让我们找出关于`deepEquals`方法的更多细节。

### 2.1.句法

我们将从查看**方法签名**开始:

```java
public static boolean deepEquals(Object[] a1, Object[] a2)
```

从方法签名中，我们注意到**我们不能使用`deepEquals`来比较原始数据类型**的两个一维数组。为此，我们必须将原始数组封装到其对应的包装器中，或者使用`Arrays.equals`方法，该方法重载了原始数组的方法。

### 2.2.履行

通过分析该方法的内部实现，我们可以看到**该方法不仅检查数组的顶层元素，还递归地检查它的每个子元素**。

因此，我们应该**避免对具有自引用**的数组使用`deepEquals`方法，因为这将导致`java.lang.StackOverflowError`。

接下来，让我们看看这个方法能得到什么样的输出。

## 3.输出

`Arrays.deepEquals`方法返回:

*   `true` 如果两个参数是同一个对象(有相同的引用)
*   `true` 如果两个参数都是`null`
*   `false `如果两个参数中只有一个是`null`
*   `false` 如果数组长度不同
*   如果两个数组都是空的
*   如果数组包含相同数量的元素，并且每一对子元素都完全相等
*   `false` 在其他情况下

在下一节中，我们将看一些代码示例。

## 4.例子

现在是时候开始研究实际使用的`deepEquals`方法了。此外，我们将比较来自同一个`Arrays`类的`deepEquals`方法和`equals` 方法。

### 4.1.一维数组

首先，让我们从一个简单的例子开始，比较两个类型为`Object`的一维数组:

```java
 Object[] anArray = new Object[] { "string1", "string2", "string3" };
    Object[] anotherArray = new Object[] { "string1", "string2", "string3" };

    assertTrue(Arrays.equals(anArray, anotherArray));
    assertTrue(Arrays.deepEquals(anArray, anotherArray));
```

我们看到`equals`和`deepEquals`方法都返回了`true`。让我们看看如果数组中的一个元素是`null`会发生什么:

```java
 Object[] anArray = new Object[] { "string1", null, "string3" };
    Object[] anotherArray = new Object[] { "string1", null, "string3" };

    assertTrue(Arrays.equals(anArray, anotherArray));
    assertTrue(Arrays.deepEquals(anArray, anotherArray));
```

我们看到这两种说法都是过去式。因此，我们可以得出结论，当使用`deepEquals`方法时， **`null`值在输入数组**的任何深度都是可接受的。

但是，让我们再尝试一件事，让我们检查嵌套数组的行为:

```java
 Object[] anArray = new Object[] { "string1", null, new String[] {"nestedString1", "nestedString2" }};
    Object[] anotherArray = new Object[] { "string1", null, new String[] {"nestedString1", "nestedString2" } };

    assertFalse(Arrays.equals(anArray, anotherArray));
    assertTrue(Arrays.deepEquals(anArray, anotherArray));
```

这里我们发现`deepEquals`返回`true`而`equals`返回`false`。这是因为 **`deepEquals`在遇到数组**时会递归调用自己，而 equals 只是比较子数组的引用。

### 4.2.原始类型的多维数组

接下来，让我们使用多维数组来检查行为。在下一个例子中，两个方法有不同的输出，强调了这样一个事实，当我们比较多维数组时，我们应该使用`deepEquals`而不是`equals`方法:

```java
 int[][] anArray = { { 1, 2, 3 }, { 4, 5, 6, 9 }, { 7 } };
    int[][] anotherArray = { { 1, 2, 3 }, { 4, 5, 6, 9 }, { 7 } };

    assertFalse(Arrays.equals(anArray, anotherArray));
    assertTrue(Arrays.deepEquals(anArray, anotherArray));
```

### 4.3.用户定义对象的多维数组

最后，让我们检查一下在测试一个用户定义对象的两个多维数组的相等性时`deepEquals `和`equals `方法的行为:

让我们从创建一个简单的`Person`类开始:

```java
 class Person {
        private int id;
        private String name;
        private int age;

        // constructor & getters & setters

        @Override
        public boolean equals(Object obj) {
            if (this == obj) {
                return true;
            }
            if (obj == null) {
                return false;
            }
            if (!(obj instanceof Person))
                return false;
            Person person = (Person) obj;
            return id == person.id && name.equals(person.name) && age == person.age;
        }
    }
```

**有必要为我们的`Person`类重写`equals`方法**。否则，默认的`equals`方法将只比较对象的引用。

同样，让我们考虑一下，尽管这与我们的例子无关，但是当我们覆盖`equals`方法时，我们应该总是覆盖`hashCode`，这样我们就不会违反它们的[契约](/web/20221206163346/https://www.baeldung.com/java-equals-hashcode-contracts)。

接下来，我们可以比较`Person`类的两个多维数组:

```java
 Person personArray1[][] = { { new Person(1, "John", 22), new Person(2, "Mike", 23) },
      { new Person(3, "Steve", 27), new Person(4, "Gary", 28) } };
    Person personArray2[][] = { { new Person(1, "John", 22), new Person(2, "Mike", 23) }, 
      { new Person(3, "Steve", 27), new Person(4, "Gary", 28) } };

    assertFalse(Arrays.equals(personArray1, personArray2));
    assertTrue(Arrays.deepEquals(personArray1, personArray2));
```

作为递归比较子元素的结果，这两种方法也有不同的结果。

最后，值得一提的是， **`Objects.deepEquals`方法在用两个`Object`数组调用时，在**内部执行`Arrays.deepEquals`方法:

```java
 assertTrue(Objects.deepEquals(personArray1, personArray2));
```

## 5.结论

在这个快速教程中，我们了解到，当我们想要比较两个嵌套或多维对象数组或原始类型之间的**相等性时，我们应该使用`Arrays.deepEquals`方法。**

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221206163346/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-operations-advanced)