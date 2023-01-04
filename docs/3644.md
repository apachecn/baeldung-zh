# Java 中 ClassCastException 的解释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-classcastexception>

## 1.概观

在这个简短的教程中，我们将关注 [`ClassCastException`](https://web.archive.org/web/20221208143830/https://javadoc.scijava.org/Java8/java/lang/ClassCastException.html) ，一个[常见的 Java 异常](/web/20221208143830/https://www.baeldung.com/java-common-exceptions)。

**`ClassCastException` 是一个[未检查的异常](/web/20221208143830/https://www.baeldung.com/java-checked-unchecked-exceptions)，表示代码试图对一个不是子类型**的类型进行强制引用。

让我们来看看导致这个异常被抛出的一些场景，以及我们如何避免它们。

## 2.显式造型

对于我们的下一个实验，让我们考虑下面的类:

```
public interface Animal {
    String getName();
}
```

```
public class Mammal implements Animal {
    @Override
    public String getName() {
        return "Mammal";
    }
}
```

```
public class Amphibian implements Animal {
    @Override
    public String getName() {
        return "Amphibian";
    }
}
```

```
public class Frog extends Amphibian {
    @Override
    public String getName() {
        return super.getName() + ": Frog";
    }
}
```

### 2.1.铸造类

**到目前为止，遇到`ClassCastException`最常见的情况是显式强制转换为不兼容的类型。**

例如，让我们尝试将一个`Frog`转换为一个`Mammal`:

```
Frog frog = new Frog();
Mammal mammal = (Mammal) frog;
```

**我们可能期望这里有一个`ClassCastException`，但实际上，我们得到了一个编译错误:“不兼容的类型:青蛙不能被转换成哺乳动物”。**然而，当我们使用普通的超类型时，情况发生了变化:

```
Animal animal = new Frog();
Mammal mammal = (Mammal) animal;
```

现在，我们从第二行得到一个`ClassCastException` :

```
Exception in thread "main" java.lang.ClassCastException: class Frog cannot be cast to class Mammal (Frog and Mammal are in unnamed module of loader 'app') 
at Main.main(Main.java:9)
```

检查到`Mammal`的向下转换与`Frog`引用不兼容，因为`Frog`不是`Mammal`的子类型。在这种情况下，编译器帮不了我们，因为`Animal`变量可能包含一个兼容类型的引用。

有趣的是，只有当我们试图强制转换为明确不兼容的类时，才会出现编译错误。对于接口来说就不一样了，因为 Java 支持多接口继承，但是对于类来说只支持单一继承。因此，编译器不能确定引用类型是否实现了特定的接口。让我们举例说明:

```
Animal animal = new Frog();
Serializable serial = (Serializable) animal;
```

我们在第二行得到一个`ClassCastException`,而不是一个编译错误:

```
Exception in thread "main" java.lang.ClassCastException: class Frog cannot be cast to class java.io.Serializable (Frog is in unnamed module of loader 'app'; java.io.Serializable is in module java.base of loader 'bootstrap') 
at Main.main(Main.java:11)
```

### 2.2.铸造阵列

我们已经看到了类如何处理类型转换，现在让我们看看数组。数组转换的工作原理与类转换相同。然而，我们可能会被自动装箱和类型提升，或者缺乏自动装箱和类型提升弄糊涂。

因此，让我们看看当我们尝试以下转换时，原始数组会发生什么:

```
Object primitives = new int[1];
Integer[] integers = (Integer[]) primitives;
```

第二行抛出一个`ClassCastException`，因为[自动装箱](/web/20221208143830/https://www.baeldung.com/java-wrapper-classes)对数组不起作用。

类型提升怎么样？让我们试试下面的方法:

```
Object primitives = new int[1];
long[] longs = (long[]) primitives;
```

我们还会得到一个`ClassCastException`，因为[类型的升级](/web/20221208143830/https://www.baeldung.com/java-primitive-conversions)并不适用于整个阵列。

### 2.3.安全铸造

在显式强制转换的情况下，强烈建议**在尝试使用 [`instanceof`](/web/20221208143830/https://www.baeldung.com/java-instanceof) **强制转换**之前检查类型的兼容性。**

让我们看一个安全强制转换的例子:

```
Mammal mammal;
if (animal instanceof Mammal) {
    mammal = (Mammal) animal;
} else {
    // handle exceptional case
}
```

## 3.堆积污染

根据 [Java 规范](https://web.archive.org/web/20221208143830/https://docs.oracle.com/javase/specs/jls/se8/html/jls-4.html#jls-4.12.2):[堆污染](https://web.archive.org/web/20221208143830/https://en.wikipedia.org/wiki/Heap_pollution)只有当程序执行了一些涉及原始类型的操作，会产生编译时未检查的警告时才会发生。

对于我们的实验，让我们考虑下面的泛型类:

```
public static class Box<T> {
    private T content;

    public T getContent() {
        return content;
    }

    public void setContent(T content) {
        this.content = content;
    }
}
```

我们现在将尝试如下污染堆:

```
Box<Long> originalBox = new Box<>();
Box raw = originalBox;
raw.setContent(2.5);
Box<Long> bound = (Box<Long>) raw;
Long content = bound.getContent();
```

最后一行将抛出一个`ClassCastException`，因为它不能将 D `ouble`引用转换为`Long`。

## 4.泛型类型

当在 Java 中使用泛型时，我们必须警惕[类型擦除](/web/20221208143830/https://www.baeldung.com/java-type-erasure)，这在某些情况下也会导致`ClassCastException`。

让我们考虑下面的通用方法:

```
public static <T> T convertInstanceOfObject(Object o) {
    try {
        return (T) o;
    } catch (ClassCastException e) {
        return null;
    }
}
```

现在让我们称之为:

```
String shouldBeNull = convertInstanceOfObject(123);
```

乍一看，我们有理由期待从 catch 块返回一个空引用。但是，在运行时，由于类型擦除，参数被强制转换为`Object`而不是`String`。因此编译器面临的任务是将一个`Integer`赋值给`String`，后者抛出`ClassCastException.`

## 5.结论

在本文中，我们研究了一系列不适当的造型的常见场景。

无论是隐式的还是显式的，**将 Java 引用转换为另一种类型会导致`ClassCastException`，除非目标类型与实际类型**相同或者是其后代。

本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-3)