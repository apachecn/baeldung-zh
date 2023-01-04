# Java 泛型中的类型参数与通配符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-generics-type-parameter-vs-wildcard>

## 1.概观

在本教程中，我们将讨论 Java [泛型](/web/20221212194241/https://www.baeldung.com/java-generics)中的形式类型参数和通配符之间的区别，以及如何正确使用它们。

认为它们是一样的是一个常见的错误。在编写泛型方法时，我们经常会考虑应该使用类型参数还是通配符。在本教程中，我们将尝试找到这个问题的答案。

## 2.通用类

在许多编程语言中，一种类型可以被另一种类型参数化。这是一种特性，在 Java 中称为泛型，在其他语言中称为参数多态性。

泛型是在 Java 5 中引入的。使用它们，我们可以创建仅在类型上不同的类和方法。此外，我们可以创建可重用的代码。

当在我们的类和接口中引入泛型时，我们应该使用类型参数。

举个例子，我们来看看 [`java.lang.Comparable`](/web/20221212194241/https://www.baeldung.com/java-comparator-comparable#comparable) 界面:

```
public interface Comparable<T> {
    public int compareTo(T o);
}
```

type 参数定义了正式类型，通常用一个大写字母命名(如`T`、`E`)。这里，参数`T`在整个`Comparable`界面中可用。

在实例化时，我们需要提供一个具体的类型(类型参数)。我们不能使用通配符来定义一个通用类或接口。

## 3.通用方法

使用泛型最常见的方式是用于公共静态方法，因为它们不是可以声明类型的实例的一部分。我们经常使用它们来创建库或向客户提供 API。

### 3.1.方法参数

要编写具有泛型类型参数的方法，我们应该使用类型参数。所以让我们创建一个打印给定的`item`的方法:

```
public static <T> void print(T item){
    System.out.println(item);
}
```

在这个例子中，我们不能用通配符替换类型参数`T`。**我们不能直接使用通配符来指定方法中参数的类型。**我们唯一可以使用它们的地方是作为泛型代码的一部分，例如，作为尖括号中定义的泛型类型参数。

通常情况下，我们可以使用通配符或类型参数来声明泛型方法。例如，下面是一个`swap()`方法的两个可能的声明:

```
public static <E> void swap(List<E> list, int src, int des);
public static void swap(List<?> list, int src, int des);
```

第一种方法使用一个无界类型参数，而第二种方法使用一个无界通配符。每当我们有一个无界的泛型类型时，我们应该选择第二个声明。

通配符使代码更简单、更灵活。我们可以传递任何列表，此外，我们不必担心类型参数。如果一个类型参数在方法声明中只出现一次，我们应该考虑用通配符代替它。同样的规则也适用于有界类型参数。

没有上限或下限，通配符代表“任何类型”，或者未知的类型。其目的是允许在不同的方法调用中使用各种实际的参数类型。此外，通配符旨在支持灵活的子类型。

### 3.2.返回类型

现在，让我们看一个返回通配符类型的`merge()`方法:

```
public static <E> List<? extends E> mergeWildcard(List<? extends E> listOne, List<? extends E> listTwo) {
    return Stream.concat(listOne.stream(), listTwo.stream())
            .collect(Collectors.toList());
}
```

假设我们有两个想要合并的列表:

```
List<Number> numbers1 = new ArrayList<>();
numbers1.add(5);
numbers1.add(10L);

List<Number> numbers2 = new ArrayList<>();
numbers2.add(15f);
numbers2.add(20.0);
```

因为我们传递了两个`Number`列表，所以我们期望收到相同类型的`List`返回。如果我们使用通配符类型作为返回类型，就不会出现这种情况。以下代码无法编译:

```
List<Number> numbersMerged = CollectionUtils.mergeWildcard(numbers1, numbers2);
```

使用通配符不但不能提供额外的灵活性，反而会迫使客户端自己处理它们。正确使用时，客户端不应该知道通配符的用法。否则，这可能表明我们的代码存在设计问题。

**当泛型方法返回泛型类型时，我们应该使用类型参数而不是通配符:**

```
public static <E> List<E> mergeTypeParameter(List<? extends E> listOne, List<? extends E> listTwo) {
    return Stream.concat(listOne.stream(), listTwo.stream())
            .collect(Collectors.toList());
}
```

使用新的实现，我们可以将结果存储在`Number`元素的列表中。

## 4.界限

泛型类型绑定允许我们限制使用什么类型来代替泛型类型。这个 Java 特性使得多态处理泛型成为可能。

例如，对数字进行操作的方法可能只想接受`Number`类或其子类的实例。这就是有界类型参数的用途。

我们可以通过三种方式使用带界限的通配符:

*   无界通配符:`List<?>`–表示任何类型的列表
*   上限通配符:`List<? extends Number>`–表示`Number`或其子类型的列表(例如，`Double`或`Integer`)。
*   下界通配符:`List<? super Integer>`–表示一列`Integer`或其超类型、`Number,`和`Object`

另一方面，有界参数类型是为泛型指定界限的泛型类型。我们可以用两种方式绑定类型参数:

*   无界类型参数:`List<T>`表示类型`T`的列表
*   有界类型参数:`List<T extends Number & Comparable>`表示实现`Comparable`接口的`Number`或其子类型如`Integer`和`Double`的列表

我们不能使用带下限的类型参数。此外，**类型参数可以有多个界限，而通配符不能。**

### 4.1.上限类型

在泛型中，参数化类型是不变的。换句话说，我们知道，例如，`Long`类是`Number`类的子类型。然而，理解`List<Long>`不是`List<Number>`的子类型是很重要的。为了更好地理解后者，让我们创建一个方法来总结集合中元素的值。

假设我们想要总结一个`Number`类的任何子类型。如果没有泛型，我们的实现可能如下所示:

```
public static long sum(List<Number> numbers) {
    return numbers.stream().mapToLong(Number::longValue).sum();
}
```

现在，让我们创建一个`Number`元素的列表并调用方法:

```
List<Number> numbers = new ArrayList<>();
numbers.add(5);
numbers.add(10L);
numbers.add(15f);
numbers.add(20.0);
CollectionUtils.sum(numbers);
```

在这里，一切都按预期运行。然而，如果我们想要传递一个只包含`Integer`元素的列表，我们会得到一个编译器错误。尽管`Integer`是`Number`的子类型，但是`List<Integer>`不是`List<Number>`的子类型。

澄清一下，我们不能传递一个`Integer`类型的列表，因为我们违反了[里斯科夫替换原则](/web/20221212194241/https://www.baeldung.com/java-liskov-substitution-principle)。类型`List<Integer>`的值不提供类型`List<Number>`的值的所有功能。

为了解决这个问题，我们可以使用带有上限的通配符。这种类型的绑定使用关键字`extends`，它指定泛型类型必须是类本身的给定类的子类型。

首先，让我们使用通配符来修改这个方法:

```
public static long sumWildcard(List<? extends Number> numbers) {
    return numbers.stream().mapToLong(Number::longValue).sum();
}
```

接下来，我们可以使用包含`Number`类的子类型的列表来调用该方法:

```
List<Integer> integers = new ArrayList<>();
integers.add(5);
integers.add(10);
CollectionUtils.sumWildcard(integers);
```

在这个例子中，`List<Integer>`是`List<? extends Number>`的一种类型，它使得方法的调用有效并且是类型安全的。

同样，我们可以使用类型参数实现相同的功能:

```
public static <T extends Number> long sumTypeParameter(List<T> numbers) {
    return numbers.stream().mapToLong(Number::longValue).sum();
} 
```

这里，列表中元素的类型成为方法的类型参数。它的名字是`T`，但是它仍然是不确定的，并且有一个上限(`<T extends Number>`)。

### 4.2.下界类型

**这种类型的界限只能与通配符一起使用，因为类型参数不支持它们。**它是用`super`关键字表达的，它指定了层次结构中可以用作泛型类型的较低的类。

假设我们想编写一个泛型方法，将数字添加到列表中。说我们不想支持十进制数而只支持`Integer`。为了最大化灵活性，我们希望允许用户使用列表`Integer`及其所有超类型(`Number`或`Object`)来调用我们的方法。换句话说，任何可以持有`Integer`价值观的东西。我们应该使用下界通配符:

```
public static void addNumber(List<? super Integer> list, Integer number) {
    list.add(number);
}
```

在这里，任何子类型都可以插入到用父类型定义的集合中。

如果我们不确定应该使用上限还是下限，我们可以考虑一下[PECS](/web/20221212194241/https://www.baeldung.com/java-generics-interview-questions#q13-when-would-you-choose-to-use-a-lower-bounded-type-vs-an-upper-bounded-type)——生产者延伸，消费者超。

每当我们的方法消耗一个集合时，例如添加元素，我们应该使用一个下限。另一方面，如果我们的方法只读取元素，我们应该使用上限。此外，如果我们的方法两者都做，也就是说，它产生和消费一个集合，我们不能应用 PECS 规则，而应该使用无界类型。

为了澄清，让我们将 PECS 规则应用到我们的示例中。我们的泛型方法通过向列表中添加元素来修改列表。从列表的角度来看，它允许添加元素，但是读取元素是不安全的。如果我们试图用一个上界代替下界，我们会得到一个编译错误。

### 4.3.无限制类型

有时，我们希望创建一个通用方法来修改集合。

我们已经提到我们应该考虑使用通配符代替类型参数来增加灵活性。然而，在支持集合修改的方法中使用通配符可能会很棘手。

让我们考虑一下前面提到的`swap()`方法的实现。该方法的简单实现不会编译:

```
public static void swap(List<?> list, int srcIndex, int destIndex) {
    list.set(srcIndex, list.set(destIndex, list.get(srcIndex)));
}
```

代码不会编译，因为我们将列表声明为任何类型，所以 Java 希望通过禁止对包含任何元素的列表进行任何修改来拯救我们。

此外，我们不能使用边界，因为我们的方法产生(读取)和消耗(更新)列表。换句话说，PECS 规则不能在这里应用，我们应该使用一个未绑定的类型。

我们可以通过编写一个通用的助手方法来捕获通配符类型来解决这个问题:

```
private static <E> void swapHelper(List<E> list, int src, int des) {
    list.set(src, list.set(des, list.get(src)));
}
```

`swapHelper()`方法知道列表是一个`List<E>`。因此，它知道从这个列表中得到的任何值都是类型`E`的，并且可以安全地将类型`E`的任何值放回到列表中。

### 4.4.多重界限

当我们想要用多个界限来限制类型时，类型参数是很有用的。

首先，让我们创建一个简单的层次结构。让我们定义一个`Animal`类:

```
abstract class Animal {

    protected final String type;
    protected final String name;

    protected Animal(String type, String name) {
        this.type = type;
        this.name = name;
    }

    abstract String makeSound();
} 
```

其次，让我们创建两个具体的类:

```
class Dog extends Animal {

    public Dog(String type, String name) {
        super(type, name);
    }

    @Override
    public String makeSound() {
        return "Wuf";
    }

}
```

此外，第二个具体类也实现了一个`Comparable`接口:

```
class Cat extends Animal implements Comparable<Cat> {
    public Cat(String type, String name) {
        super(type, name);
    }

    @Override
    public String makeSound() {
        return "Meow";
    }

    @Override
    public int compareTo(@NotNull Cat cat) {
        return this.getName().length() - cat.getName().length();
    }
}
```

最后，让我们定义一个方法，对给定列表中的元素进行排序，并要求值具有可比性:

```
public static <T extends Animal & Comparable<T>> void order(List<T> list) {
    list.sort(Comparable::compareTo);
}
```

这样，我们的列表不能包含`Dog`类型的元素，因为这个类没有实现`Comparable`接口。

## 5.结论

在本文中，我们讨论了 Java 泛型中类型参数和通配符之间的区别。总而言之，在编写广泛使用的库时，我们应该更喜欢通配符而不是类型参数。在决定使用什么绑定类型时，我们应该记住基本的 PECS 规则。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20221212194241/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-5)