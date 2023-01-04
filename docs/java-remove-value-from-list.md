# 从列表中删除特定值的所有匹配项

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-remove-value-from-list>

## 1.介绍

在 Java 中，使用`List.remove()`从`List`中删除一个特定的值是很简单的。然而，**有效地移除值**的所有出现要困难得多。

在本教程中，我们将看到这个问题的多种解决方案，描述其利弊。

为了可读性，我们在测试中使用了一个定制的`list(int…)`方法，它返回一个包含我们传递的元素的`ArrayList`。

## 2.使用`while`循环

既然我们知道如何**移除单个元素，那么在一个循环中重复执行**看起来就足够简单了:

```java
void removeAll(List<Integer> list, int element) {
    while (list.contains(element)) {
        list.remove(element);
    }
}
```

然而，它并不像预期的那样工作:

```java
// given
List<Integer> list = list(1, 2, 3);
int valueToRemove = 1;

// when
assertThatThrownBy(() -> removeAll(list, valueToRemove))
  .isInstanceOf(IndexOutOfBoundsException.class);
```

问题出在第三行:我们调用 **`List.remove(int),`，它把它的参数当作索引，而不是我们想要删除的值。**

在上面的测试中，我们总是调用`list.remove(1)`，但是我们想要移除的元素的索引是`0.`，调用`List.remove()`会将移除的元素之后的所有元素转移到更小的索引。

在这个场景中，这意味着我们删除所有元素，除了第一个。

当只剩下第一个时，索引`1`将是非法的。因此我们得到了一个`Exception`。

请注意，只有当我们用原始的`byte`、`short, char`或`int`参数调用`List.remove()`时，我们才会面临这个问题，因为编译器在试图找到匹配的重载方法时做的第一件事就是加宽。

我们可以通过将该值作为`Integer:`传递来纠正它

```java
void removeAll(List<Integer> list, Integer element) {
    while (list.contains(element)) {
        list.remove(element);
    }
}
```

现在代码如预期的那样工作了:

```java
// given
List<Integer> list = list(1, 2, 3);
int valueToRemove = 1;

// when
removeAll(list, valueToRemove);

// then
assertThat(list).isEqualTo(list(2, 3));
```

由于`List.contains()`和`List.remove()`都必须找到元素的第一次出现，这段代码导致了不必要的元素遍历。

如果我们存储第一次出现的索引，我们可以做得更好:

```java
void removeAll(List<Integer> list, Integer element) {
    int index;
    while ((index = list.indexOf(element)) >= 0) {
        list.remove(index);
    }
}
```

我们可以验证它是否有效:

```java
// given
List<Integer> list = list(1, 2, 3);
int valueToRemove = 1;

// when
removeAll(list, valueToRemove);

// then
assertThat(list).isEqualTo(list(2, 3));
```

虽然这些解决方案产生了简短而干净的代码，**它们仍然有很差的性能**:因为我们没有跟踪进度，`List.remove()`必须找到所提供值的第一次出现来删除它。

此外，当我们使用一个`ArrayList`时，元素移位会导致许多引用复制，甚至多次重新分配后备数组。

## 3.移除直到`List`改变

**`List.remove(E element)`** 有一个我们还没有提到的特性:它**返回一个`boolean`值，如果`List`因为操作而改变，那么这个值就是`true`，因此它包含了元素**。

注意，`List.remove(int index)`返回 void，因为如果提供的索引有效，`List`总是删除它。否则，它抛出`IndexOutOfBoundsException`。

这样，我们可以执行移除，直到`List`改变:

```java
void removeAll(List<Integer> list, int element) {
    while (list.remove(element));
}
```

它像预期的那样工作:

```java
// given
List<Integer> list = list(1, 1, 2, 3);
int valueToRemove = 1;

// when
removeAll(list, valueToRemove);

// then
assertThat(list).isEqualTo(list(2, 3));
```

尽管这个实现很短，但它也存在我们在上一节中描述的问题。

## 3.使用`for`循环

我们可以通过使用一个`for`循环遍历元素来跟踪我们的进度，如果匹配就删除当前的元素:

```java
void removeAll(List<Integer> list, int element) {
    for (int i = 0; i < list.size(); i++) {
        if (Objects.equals(element, list.get(i))) {
            list.remove(i);
        }
    }
}
```

它像预期的那样工作:

```java
// given
List<Integer> list = list(1, 2, 3);
int valueToRemove = 1;

// when
removeAll(list, valueToRemove);

// then
assertThat(list).isEqualTo(list(2, 3));
```

但是，如果我们尝试使用不同的输入，它会提供不正确的输出:

```java
// given
List<Integer> list = list(1, 1, 2, 3);
int valueToRemove = 1;

// when
removeAll(list, valueToRemove);

// then
assertThat(list).isEqualTo(list(1, 2, 3));
```

让我们一步一步地分析代码是如何工作的:

*   `i = 0`
    *   第 3 行的`element`和`list.get(i)`都等于`1`，所以 Java 进入了`if`语句的主体，
    *   我们移除索引`0`处的元素，
    *   所以`list`现在包含了`1`、`2`和`3`
*   `i = 1`
    *   `list.get(i)`返回`2`,因为**当我们从`List`中移除一个元素时，它将所有继续的元素转移到更小的索引**

所以当**有两个相邻的值时，我们就面临这个问题，我们想去掉**。为了解决这个问题，我们应该维护循环变量。

当我们移除元素时减少它:

```java
void removeAll(List<Integer> list, int element) {
    for (int i = 0; i < list.size(); i++) {
        if (Objects.equals(element, list.get(i))) {
            list.remove(i);
            i--;
        }
    }
}
```

只有当我们不移除元素时才增加它:

```java
void removeAll(List<Integer> list, int element) {
    for (int i = 0; i < list.size();) {
        if (Objects.equals(element, list.get(i))) {
            list.remove(i);
        } else {
            i++;
        }
    }
}
```

注意，在后者中，我们删除了第 2 行的语句`i++`。

这两种解决方案都按预期工作:

```java
// given
List<Integer> list = list(1, 1, 2, 3);
int valueToRemove = 1;

// when
removeAll(list, valueToRemove);

// then
assertThat(list).isEqualTo(list(2, 3));
```

乍一看，这个实现似乎是正确的。然而，它仍然有**严重的性能问题**:

*   从`ArrayList`中删除一个元素，会移动它后面的所有项目
*   在`LinkedList`中通过索引访问元素意味着逐个遍历元素，直到找到索引

## 4.使用`for-each`循环

从 Java 5 开始，我们可以使用`for-each`循环来遍历`List`。让我们用它来删除元素:

```java
void removeAll(List<Integer> list, int element) {
    for (Integer number : list) {
        if (Objects.equals(number, element)) {
            list.remove(number);
        }
    }
}
```

注意，我们使用`Integer`作为循环变量的类型。因此我们不会得到一个`NullPointerException`。

同样，这种方式我们调用`List.remove(E element)`，它期望我们想要移除的值，而不是索引。

虽然看起来很干净，但不幸的是，它不起作用:

```java
// given
List<Integer> list = list(1, 1, 2, 3);
int valueToRemove = 1;

// when
assertThatThrownBy(() -> removeWithForEachLoop(list, valueToRemove))
  .isInstanceOf(ConcurrentModificationException.class);
```

`for-each`循环使用`Iterator`遍历元素。然而，**当我们修改`List`时，`Iterator`进入不一致状态。于是就抛出了`ConcurrentModificationException`T6。**

教训是:当我们在一个`for-each`循环中访问它的元素时，我们不应该修改一个`List`。

## 5.使用`Iterator`

我们可以直接使用`Iterator`来遍历和修改`List`:

```java
void removeAll(List<Integer> list, int element) {
    for (Iterator<Integer> i = list.iterator(); i.hasNext();) {
        Integer number = i.next();
        if (Objects.equals(number, element)) {
            i.remove();
        }
    }
}
```

这样，**`Iterator`就可以跟踪`List`** 的状态(因为它做出了修改)。因此，上面的代码按预期工作:

```java
// given
List<Integer> list = list(1, 1, 2, 3);
int valueToRemove = 1;

// when
removeAll(list, valueToRemove);

// then
assertThat(list).isEqualTo(list(2, 3));
```

由于每个`List`类都可以提供它们自己的`Iterator`实现，我们可以有把握地认为**以最有效的方式实现了元素遍历和移除。**

**然而，使用`ArrayList`仍然意味着大量的元素移位**(也许还有数组重新分配)。此外，上面的代码有点难读，因为它不同于大多数开发人员熟悉的标准`for`循环。

## 6.收集

在此之前，我们通过删除不需要的项目来修改原始的`List`对象。相反，我们可以**创建一个新的`List`并收集我们想要保留的物品**:

```java
List<Integer> removeAll(List<Integer> list, int element) {
    List<Integer> remainingElements = new ArrayList<>();
    for (Integer number : list) {
        if (!Objects.equals(number, element)) {
            remainingElements.add(number);
        }
    }
    return remainingElements;
}
```

因为我们在一个新的`List`对象中提供了结果，所以我们必须从方法中返回它。因此，我们需要以另一种方式使用该方法:

```java
// given
List<Integer> list = list(1, 1, 2, 3);
int valueToRemove = 1;

// when
List<Integer> result = removeAll(list, valueToRemove);

// then
assertThat(result).isEqualTo(list(2, 3));
```

注意，现在我们可以使用`for-each`循环，因为我们没有修改当前正在迭代的`List`。

因为没有任何移除，所以没有必要移动元素。因此，当我们使用`ArrayList.`时，这个实现执行得很好

该实现在某些方面与以前的实现有所不同:

*   它不会修改原来的`List`，但是**会返回一个新的**
*   **该方法决定返回的`List`的实现是什么**，它可能与原来的不同

同样，我们可以修改我们的实现，使**获得旧的行为**；我们清除原始的`List`,并将收集的元素添加到其中:

```java
void removeAll(List<Integer> list, int element) {
    List<Integer> remainingElements = new ArrayList<>();
    for (Integer number : list) {
        if (!Objects.equals(number, element)) {
            remainingElements.add(number);
        }
    }

    list.clear();
    list.addAll(remainingElements);
}
```

它的工作方式和之前的一样:

```java
// given
List<Integer> list = list(1, 1, 2, 3);
int valueToRemove = 1;

// when
removeAll(list, valueToRemove);

// then
assertThat(list).isEqualTo(list(2, 3));
```

因为我们不需要不断地修改`List`，所以我们不需要通过位置或者移动来访问元素。同样，只有两种可能的数组重新分配:当我们调用`List.clear()`和`List.addAll()`时。

## 7.使用流 API

Java 8 引入了 lambda 表达式和流 API。有了这些强大的功能，我们可以用非常简洁的代码来解决我们的问题:

```java
List<Integer> removeAll(List<Integer> list, int element) {
    return list.stream()
      .filter(e -> !Objects.equals(e, element))
      .collect(Collectors.toList());
}
```

这个解决方案**的工作方式是一样的，就像我们收集剩余的元素一样。**

**结果，它有相同的特征**，我们要用它来返回结果:

```java
// given
List<Integer> list = list(1, 1, 2, 3);
int valueToRemove = 1;

// when
List<Integer> result = removeAll(list, valueToRemove);

// then
assertThat(result).isEqualTo(list(2, 3));
```

请注意，我们可以使用与最初的“收集”实现相同的方法将其转换为像其他解决方案一样的工作方式。

## 8.使用`removeIf`

有了 lambdas 和函数接口，Java 8 也引入了一些 API 扩展。例如， **`List.removeIf()`方法，它实现了我们在上一节**中看到的内容。

它期待一个`Predicate`，当我们想要删除元素时，它应该返回 **`true`，与前面的例子相反，在前面的例子中，当我们想要保留元素时，我们必须返回`true`:**

```java
void removeAll(List<Integer> list, int element) {
    list.removeIf(n -> Objects.equals(n, element));
}
```

它的工作原理与上述其他解决方案类似:

```java
// given
List<Integer> list = list(1, 1, 2, 3);
int valueToRemove = 1;

// when
removeAll(list, valueToRemove);

// then
assertThat(list).isEqualTo(list(2, 3));
```

由于这个事实，即`List`本身实现了这个方法，我们可以有把握地假设，它具有可用的最佳性能。最重要的是，这个解决方案提供了最干净的代码。

## 9.结论

在本文中，我们看到了解决一个简单问题的许多方法，包括不正确的方法。我们对它们进行了分析，以找到每种情况下的最佳解决方案。

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143941/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-list)