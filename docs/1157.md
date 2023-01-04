# 迭代器和 Iterable 的区别以及如何使用？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-iterator-vs-iterable>

## 1.概观

在本教程中，我们将看看 Java 中的`Iterable`和`Iterator`接口的用法以及它们之间的区别。

## 2.`Iterable`界面

[`Iterable`](https://web.archive.org/web/20220606083223/https://docs.oracle.com/javase/8/docs/api/java/lang/Iterable.html "Iterable") 界面属于`java.lang`包。**它代表了一种可以迭代的数据结构。**

`Iterable`接口提供了一个产生`Iterator`的方法。当使用`Iterable`时，我们不能通过索引获取元素。同样，我们也不能从数据结构中获得第一个或最后一个元素。

**Java 中所有的集合都实现了`Iterable`接口。**

### 2.1.迭代一个`Iterable`

我们可以使用增强的`for`循环迭代集合中的元素，也称为 [`for` -each](/web/20220606083223/https://www.baeldung.com/foreach-java "for-each") 循环。然而，只有实现了`Iterable`接口的对象才能在这样的语句中使用。还可以结合使用`while`语句和`Iterator`来迭代元素。

让我们看一个使用`for` -each 语句迭代`List`中元素的例子:

```
List<Integer> numbers = getNumbers();
for (Integer number : numbers) {
    System.out.println(number);
}
```

类似地，我们可以将`forEach()`方法与 lambda 表达式结合使用:

```
List<Integer> numbers = getNumbers();
numbers.forEach(System.out::println);
```

### 2.2.实现`Iterable`接口

当我们有想要迭代的定制数据结构时，`Iterable`接口的定制实现会很方便。

让我们首先创建一个表示购物车的类，它将在一个数组中保存元素。我们不会直接在数组上调用`for` -each 循环。相反，我们将实现`Iterable`接口。**我们不希望我们的客户依赖于选择的数据结构。如果我们为客户提供迭代的能力，我们可以很容易地使用不同的数据结构，而客户不必改变代码。**

`ShoppingCart`类实现了`Iterable`接口并覆盖了它的`iterate()`方法:

```
public class ShoppingCart<E> implements Iterable<E> {

    private E[] elementData;
    private int size;

    public void add(E element) {
        ensureCapacity(size + 1);
        elementData[size++] = element;
    }

    @Override
    public Iterator<E> iterator() {
        return new ShoppingCartIterator();
    }
}
```

方法在一个数组中存储元素。由于数组的大小和容量是固定的，我们使用`ensureCapacity()`方法来扩展元素的最大数量。

定制数据结构上的`iterator()`方法的每次调用都会产生一个`Iterator`的新实例。我们创建一个新的实例，因为迭代器负责维护当前的迭代状态。

**通过提供`iterator()`方法的具体实现，我们可以使用增强的`for`语句来迭代实现类的对象。**

现在，让我们在`ShoppingCart`类中创建一个内部类来表示我们的自定义迭代器:

```
public class ShoppingCartIterator implements Iterator<E> {
    int cursor;
    int lastReturned = -1;

    public boolean hasNext() {
        return cursor != size;
    }

    public E next() {
        return getNextElement();
    }

    private E getNextElement() {
        int current = cursor;
        exist(current);

        E[] elements = ShoppingCart.this.elementData;
        validate(elements, current);

        cursor = current + 1;
        lastReturned = current;
        return elements[lastReturned];
    }
}
```

最后，让我们创建一个 iterable 类的实例，并在增强的`for`循环中使用它:

```
ShoppingCart<Product> shoppingCart  = new ShoppingCart<>();

shoppingCart.add(new Product("Tuna", 42));
shoppingCart.add(new Product("Eggplant", 65));
shoppingCart.add(new Product("Salad", 45));
shoppingCart.add(new Product("Banana", 29));

for (Product product : shoppingCart) {
   System.out.println(product.getName());
}
```

## 3.`Iterator`界面

[`Iterator`](/web/20220606083223/https://www.baeldung.com/java-iterator "Iterator") 是 Java 集合框架的成员。属于`java.util`套餐。**这个接口允许我们在迭代过程中从集合中检索或移除元素。**

此外，它有两个方法帮助迭代数据结构并检索其元素—`next()`和`hasNext()`。

此外，它还有一个`remove()`方法，该方法删除由`Iterator`指向的当前元素。

最后，`forEachRemaining(Consumer<? super E> action)`方法为数据结构中的每个剩余元素执行给定的动作。

### 3.1.迭代`Collection`

让我们看看如何迭代一个包含`Integer`个元素的`List`。在这个例子中，我们将结合`while`循环和`hasNext()`和`next()`方法。

`List`接口是`Collection`的一部分，因此，它扩展了`Iterable`接口。要从集合中获取迭代器，我们只需调用`iterator()`方法:

```
List<Integer> numbers = new ArrayList<>();
numbers.add(10);
numbers.add(20);
numbers.add(30);
numbers.add(40);

Iterator<Integer> iterator = numbers.iterator();
```

此外，我们可以通过调用`hasNext()`方法来检查迭代器是否还有剩余的元素。之后，我们可以通过调用`next()`方法获得一个元素:

```
while (iterator.hasNext()) {
   System.out.println(iterator.next());
}
```

`next()`方法返回迭代中的下一个元素。另一方面，如果没有这样的元素，它抛出`NoSuchElementException`。

### 3.2.实现`Iterator`接口

现在，我们将实现`Iterator`接口。当我们需要使用条件元素检索来迭代集合时，自定义实现会很有用。例如，我们可以使用自定义迭代器来迭代奇数或偶数。

为了说明，我们将从给定的集合中迭代[个素数](/web/20220606083223/https://www.baeldung.com/java-prime-numbers "prime numbers")。众所周知，如果一个数只能被 1 和它自己整除，那么它就被认为是质数。

首先，让我们创建一个包含数值元素集合的类:

```
class Numbers {

    private static final List<Integer> NUMBER_LIST =
      Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
}
```

此外，让我们定义一个`Iterator`接口的具体实现:

```
private static class PrimeIterator implements Iterator<Integer> {

    private int cursor;

    @Override
    public Integer next() {
        exist(cursor);
        return NUMBER_LIST.get(cursor++);
    }

    @Override
    public boolean hasNext() {
        if (cursor > NUMBER_LIST.size()) {
            return false;
        }

        for (int i = cursor; i < NUMBER_LIST.size(); i++) {
            if (isPrime(NUMBER_LIST.get(i))) {
                cursor = i;
                return true;
            }
        }

        return false;
    }
}
```

具体的实现通常被创建为内部类。此外，他们还负责维护当前的迭代状态。在上面的例子中，我们在实例变量中存储了下一个质数的当前位置。每次我们调用`next()`方法，变量都会包含一个即将到来的质数的索引。

**`next()`方法的任何实现都应该在不再有元素时抛出`NoSuchElementException`异常。**否则，迭代会导致意外的行为

让我们在`Number`类中定义一个方法，该方法返回`PrimeIterator`类的新实例:

```
public static Iterator<Integer> iterator() {
    return new PrimeIterator();
}
```

最后，我们可以在`while`语句中使用自定义迭代器:

```
Iterator<Integer> iterator = Numbers.iterator();

while (iterator.hasNext()) {
   System.out.println(iterator.next());
}
```

## 4.`Iterable`和`Iterator`的区别

综上所述，下表显示了`Iterable`和`Iterator`接口的主要区别:

| 可迭代的 | 迭代程序 |
| 表示一个集合，可以使用一个`for` -each 循环进行迭代 | 表示可用于循环访问集合的接口 |
| 当实现一个`Iterable`时，我们需要覆盖`iterator()`方法 | 当实现一个`Iterator`时，我们需要覆盖`hasNext()`和`next()`方法 |
| 不存储迭代状态 | 存储迭代状态 |
| 不允许在迭代过程中移除元素 | 允许在迭代过程中删除元素 |

## 5.结论

在本文中，我们研究了 Java 中的`Iterable`和`Iterator`接口之间的区别以及它们的用法。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20220606083223/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-2 "over on GitHub")