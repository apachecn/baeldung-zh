# 比较 Java 中的数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-comparing-arrays>

## 1.概观

在本教程中，我们将看看在 Java 中用**不同的方法来比较数组。我们将讨论常规方法，我们还将看到一些使用`lambda` `expressions`的例子。**

## 2.比较数组

我们将比较 Java 中的数组，正如我们所知，这些是对象。因此，让我们刷新一些基本概念:

*   对象有引用和值
*   两个相等的引用应该指向相同的值
*   两个不同的值应该有不同的引用
*   两个相等的值不一定有相同的引用
*   原始值只能按值进行比较
*   字符串文字只能按值进行比较

### 2.1.比较对象引用

**如果我们有两个指向同一个数组的引用，我们应该总是得到一个与`==`操作符**相等的结果`true `。

让我们看一个例子:

```java
String[] planes1 = new String[] { "A320", "B738", "A321", "A319", "B77W", "B737", "A333", "A332" };
String[] planes2 = planes1;
```

首先，我们创建了一个由`planes1`引用的平面模型数组。然后我们创建引用了`planes1`的`planes2`。通过这样做，我们**创建了对内存**中同一个数组的两个引用 **。因此，`“planes1 == planes2”`表达式将返回`true`。**

**对于数组，`equals()`方法与==运算符**相同。所以，`planes1.equals(planes2)`返回`true `，因为两个引用都指向同一个对象。一般来说，`array1.eqauls(array2)`会返回`true`当且仅当表达式`“` `array1 == array2″`返回`true`。

让我们断言这两个引用是否相同:

```java
assertThat(planes1).isSameAs(planes2);
```

现在让我们确定`planes1`引用的值实际上与`planes2`引用的值相同。因此，我们可以更改`planes2,`引用的数组，并检查这些更改是否对`planes1`引用的数组有任何影响:

```java
planes2[0] = "747";
```

为了最终看到这一点，让我们断言:

```java
assertThat(planes1).isSameAs(planes2);
assertThat(planes2[0]).isEqualTo("747");
assertThat(planes1[0]).isEqualTo("747");
```

通过这个单元测试，我们能够通过引用来比较两个数组。

然而，我们只证明了**一个引用一旦被赋予另一个引用的值，就会引用相同的值。**

我们现在将创建两个具有相同值的不同数组:

```java
String[] planes1 = new String[] { "A320", "B738", "A321", "A319", "B77W", "B737", "A333", "A332" };
String[] planes2 = new String[] { "A320", "B738", "A321", "A319", "B77W", "B737", "A333", "A332" };
```

既然它们是不同的物体，我们肯定知道它们是不同的。因此，我们可以比较它们:

```java
assertThat(planes1).isNotSameAs(planes2);
```

总而言之，在这种情况下，我们在内存中有两个数组，它们以完全相同的顺序包含相同的`String`值。但是，不仅被引用的数组内容不同，引用本身也不同。

### 2.2.比较数组长度

可以比较数组**的长度，而不管** **的元素类型，或者它们的值是否被填入**。

让我们创建两个数组:

```java
final String[] planes1 = new String[] { "A320", "B738", "A321", "A319", "B77W", "B737", "A333", "A332" };
final Integer[] quantities = new Integer[] { 10, 12, 34, 45, 12, 43, 5, 2 };
```

这是两个具有不同元素类型的不同数组。在这个数据集中，作为一个例子，我们记录了仓库中存储的每个型号的飞机数量。现在让我们对它们进行单元测试:

```java
assertThat(planes1).hasSize(8);
assertThat(quantities).hasSize(8);
```

这样，我们已经证明了两个数组都有八个元素，并且`length`属性返回每个数组的正确元素数。

### 2.3.用`Arrays.equals`比较数组

到目前为止，我们只是根据对象标识来比较数组。另一方面，**为了检查两个数组的内容是否相等，Java 提供了`Arrays.equals`静态方法。该方法将并行遍历数组的每个位置，并对每对元素**应用==操作符 **。**

让我们以完全相同的顺序用相同的`String`文字创建两个不同的数组:

```java
String[] planes1 = new String[] { "A320", "B738", "A321", "A319", "B77W", "B737", "A333", "A332" };
String[] planes2 = new String[] { "A320", "B738", "A321", "A319", "B77W", "B737", "A333", "A332" };
```

现在，让我们断言它们是相等的:

```java
assertThat(Arrays.equals(planes1, planes2)).isTrue();
```

如果我们改变第二个数组的值的顺序:

```java
String[] planes1 = new String[] { "A320", "B738", "A321", "A319", "B77W", "B737", "A333", "A332" };
String[] planes2 = new String[] { "B738", "A320", "A321", "A319", "B77W", "B737", "A333", "A332" }; 
```

我们会得到不同的结果:

```java
assertThat(Arrays.equals(planes1, planes2)).isFalse();
```

### 2.4.用`Arrays.deepEquals`比较数组

**如果我们在 Java** 中使用简单类型，使用`==`操作符就很容易。这些可能是原始类型或`String`文字。T2 数组之间的比较可能更复杂。这背后的原因在我们的 [`Arrays.deepEquals`](/web/20221208143830/https://www.baeldung.com/java-arrays-deepequals) 文章中有充分的解释。让我们看一个例子。

首先，让我们从一个`Plane `类开始:

```java
public class Plane {
    private final String name;
    private final String model;

    // getters and setters
}
```

让我们实现`hashCode `和`equals`方法:

```java
@Override
public boolean equals(Object o) {
    if (this == o)
        return true;
    if (o == null || getClass() != o.getClass())
        return false;
    Plane plane = (Plane) o;
    return Objects.equals(name, plane.name) && Objects.equals(model, plane.model);
}

@Override
public int hashCode() {
    return Objects.hash(name, model);
}
```

其次，让我们创建以下两元素数组:

```java
Plane[][] planes1 
  = new Plane[][] { new Plane[]{new Plane("Plane 1", "A320")}, new Plane[]{new Plane("Plane 2", "B738") }};
Plane[][] planes2 
  = new Plane[][] { new Plane[]{new Plane("Plane 1", "A320")}, new Plane[]{new Plane("Plane 2", "B738") }}; 
```

现在让我们看看它们是否是真正的深度相等的数组:

```java
assertThat(Arrays.deepEquals(planes1, planes2)).isTrue();
```

为了确保我们的比较按预期进行，现在让我们更改最后一个数组的顺序:

```java
Plane[][] planes1 
  = new Plane[][] { new Plane[]{new Plane("Plane 1", "A320")}, new Plane[]{new Plane("Plane 2", "B738") }};
Plane[][] planes2 
  = new Plane[][] { new Plane[]{new Plane("Plane 2", "B738")}, new Plane[]{new Plane("Plane 1", "A320") }};
```

最后，让我们测试一下它们是否真的不再相等:

```java
assertThat(Arrays.deepEquals(planes1, planes2)).isFalse();
```

### 2.5.比较元素顺序不同的数组

为了检查数组是否相等，不管元素的顺序如何，我们需要定义**是什么使得我们的`Plane`的一个实例是唯一的**。对于我们的情况，不同的名称或型号足以确定一个平面不同于另一个平面。我们已经通过实现`hashCode` 和`equals `方法建立了这一点。这意味着在我们比较数组之前，我们应该对它们进行排序。为此，我们需要一个 [`Comparator`](/web/20221208143830/https://www.baeldung.com/java-comparator-comparable) :

```java
Comparator<Plane> planeComparator = (o1, o2) -> {
    if (o1.getName().equals(o2.getName())) {
        return o2.getModel().compareTo(o1.getModel());
    }
    return o2.getName().compareTo(o1.getName());
};
```

在这个`Comparator`中，我们优先考虑名字。如果名称相等，我们通过查看模型来解决歧义。我们通过使用类型`String`的`compareTo `方法来比较字符串。

我们希望能够发现数组是否相等，而不考虑排序顺序。为此，我们现在对数组进行排序:

```java
Arrays.sort(planes1[0], planeComparator);
Arrays.sort(planes2[0], planeComparator);
```

最后，我们来测试一下:

```java
assertThat(Arrays.deepEquals(planes1, planes2)).isTrue();
```

首先按照相同的顺序对数组进行排序，然后我们让`deepEquals`方法判断这两个数组是否相等。

## 3.结论

在本教程中，我们看到了比较数组的不同方法。其次，我们看到了比较引用和值之间的区别。此外，我们还研究了如何深入比较数组**。**最后，我们分别使用`equals `和`deepEquals`看到了普通比较和深度比较之间的区别。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-arrays-operations-advanced)