# Java 中的 Void 类型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-void-type>

## 1.概观

作为 Java 开发人员，我们可能在某些场合遇到过`Void`类型，并想知道它的用途是什么。

在这个快速教程中，我们将学习这个特殊的类，并了解何时以及如何使用它，以及如何尽可能避免使用它。

## 2.`Void`是什么类型

从 JDK 1.1 开始，Java 为我们提供了`Void`类型。**它的目的只是将`void`返回类型表示为一个类，并包含一个`Class<Void> `公共值。**它不是可实例化的，因为它的唯一构造函数是私有的。

因此，我们可以分配给`Void`变量的唯一值是`null`。这似乎有点没用，但是我们现在将看到何时以及如何使用这种类型。

## 3.习惯

有些情况下使用`Void`类型会很有趣。

### 3.1.反射

首先，我们可以在做反射时使用它。实际上，**任何`void`方法的返回类型都将匹配保存前面提到的**的`Class<Void>`值的`Void.TYPE`变量。

让我们想象一个简单的`Calculator`类:

```java
public class Calculator {
    private int result = 0;

    public int add(int number) {
        return result += number;
    }

    public int sub(int number) {
        return result -= number;
    }

    public void clear() {
        result = 0;
    }

    public void print() {
        System.out.println(result);
    }
}
```

有些方法返回一个整数，有些不返回任何东西。现在，假设我们必须通过反射**检索所有不返回任何结果的方法**。我们将通过使用`Void.TYPE`变量来实现这一点:

```java
@Test
void givenCalculator_whenGettingVoidMethodsByReflection_thenOnlyClearAndPrint() {
    Method[] calculatorMethods = Calculator.class.getDeclaredMethods();
    List<Method> calculatorVoidMethods = Arrays.stream(calculatorMethods)
      .filter(method -> method.getReturnType().equals(Void.TYPE))
      .collect(Collectors.toList());

    assertThat(calculatorVoidMethods)
      .allMatch(method -> Arrays.asList("clear", "print").contains(method.getName()));
}
```

正如我们所看到的，只检索了`clear()`和`print()`方法。

### 3.2.无商标消费品

类型的另一种用法是泛型类。假设我们正在调用一个需要`Callable`参数的方法:

```java
public class Defer {
    public static <V> V defer(Callable<V> callable) throws Exception {
        return callable.call();
    }
}
```

**但是，我们要传递的`Callable`不一定要返回任何东西。因此，我们可以通过一个`Callable<Void>` :**

```java
@Test
void givenVoidCallable_whenDiffer_thenReturnNull() throws Exception {
    Callable<Void> callable = new Callable<Void>() {
        @Override
        public Void call() {
            System.out.println("Hello!");
            return null;
        }
    };

    assertThat(Defer.defer(callable)).isNull();
}
```

如上所示，**为了从具有`Void`返回类型的方法返回，我们只需返回`null`** 。此外，我们可以使用随机类型(如`Callable<Integer>`)并返回`null`，或者根本不使用类型(`Callable)`)，但是**使用`Void`清楚地表明了我们的意图**。

我们也可以将这种方法应用于 lambdas。事实上，我们的`Callable`可以写成 lambda。让我们想象一个需要一个`Function`的方法，但是我们想要使用一个不返回任何东西的`Function`。然后我们只要让它返回`Void`:

```java
public static <T, R> R defer(Function<T, R> function, T arg) {
    return function.apply(arg);
}
```

```java
@Test
void givenVoidFunction_whenDiffer_thenReturnNull() {
    Function<String, Void> function = s -> {
        System.out.println("Hello " + s + "!");
        return null;
    };

    assertThat(Defer.defer(function, "World")).isNull();
}
```

## 4.如何避免使用？

现在，我们已经看到了`Void`类型的一些用法。然而，即使第一种用法完全没问题，**如果可能的话，我们可能希望避免在泛型中使用`Void`**。事实上，遇到表示没有结果并且只能包含`null`的返回类型可能会很麻烦。

我们现在来看看如何避免这些情况。首先，让我们考虑一下带有`Callable`参数的方法。**为了避免使用`Callable<Void>`，我们可以提供另一个方法，用`Runnable`参数来代替:**

```java
public static void defer(Runnable runnable) {
    runnable.run();
}
```

因此，我们可以传递给它一个不返回值的`Runnable`，从而去掉无用的`return null`:

```java
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello!");
    }
};

Defer.defer(runnable);
```

但是，如果`Defer`类不是我们可以修改的呢？然后我们要么坚持使用`Callable<Void>`选项，要么**创建另一个类，使用`Runnable`并将调用推迟到`Defer`类**:

```java
public class MyOwnDefer {
    public static void defer(Runnable runnable) throws Exception {
        Defer.defer(new Callable<Void>() {
            @Override
            public Void call() {
                runnable.run();
                return null;
            }
        });
    }
}
```

通过这样做，我们将繁琐的部分一劳永逸地封装在我们自己的方法中，允许未来的开发人员使用更简单的 API。

当然对于`Function`来说同样可以实现。在我们的例子中，`Function`不返回任何东西，因此我们可以提供另一个采用`Consumer`的方法:

```java
public static <T> void defer(Consumer<T> consumer, T arg) {
    consumer.accept(arg);
}
```

那么，如果我们的函数不带任何参数呢？我们可以使用一个`Runnable`或者创建我们自己的功能接口(如果这看起来更清楚的话):

```java
public interface Action {
    void execute();
}
```

然后，我们再次重载`defer()`方法:

```java
public static void defer(Action action) {
    action.execute();
}
```

```java
Action action = () -> System.out.println("Hello!");

Defer.defer(action);
```

## 5.结论

在这篇短文中，我们介绍了 Java `Void`类。我们看到了它的用途和使用方法。我们还学习了它的一些替代用法。

像往常一样，这篇文章的完整代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221122163841/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-reflection)