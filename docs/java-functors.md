# Java 中的函子

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-functors>

## 1.概观

在本教程中，我们将演示如何在 Java 中创建函子。首先，让我们来了解一些关于术语“函子”本质的细节然后我们将看一些如何在 Java 中使用它的代码示例。

## 2.什么是函子？

术语“函子”来自数学领域，特别是来自一个被称为“范畴论”的子领域。在计算机编程中，函子可以被认为是一个实用程序类，它允许我们映射包含在特定上下文中的值。此外，它表示两个类别之间的结构保持映射。

两个定律支配着函子:

*   Identity:当一个函子被一个 identity 函数映射时，这个函数返回的值与它所传递的参数相同，我们需要得到初始的函子(容器及其内容保持不变)。
*   组合/结合性:当使用`Functor`映射两个部分的组合时，它应该与映射一个接一个的函数相同。

## 3.函数式编程中的函子

**函子是一种在函数式编程中使用的设计模式，它受到范畴理论**中使用的定义的启发。**它使一个泛型类型能够在其内部应用一个函数，而不影响泛型类型的结构**。在像 [Scala](/web/20221210082018/https://www.baeldung.com/scala/functors-functional-programming) 这样的编程语言中，我们可以找到函子的很多用途。

## 4.Java 中的函子

Java 和大多数其他当代编程语言不包含任何被认为是合适的内置函子等价物的东西。然而，从 Java 8 开始，函数式编程元素被引入到语言中。在 Java 编程语言中，[函数式编程](/web/20221210082018/https://www.baeldung.com/java-functional-programming)的概念仍然相对新颖。

可以使用来自`java.util.function`包的`Function`接口在 Java 中实现`Functor`。下面是一个 Java 中的`Functor`类的例子，它接受一个`Function`对象并将其应用于一个值:

```java
public class Functor<T> {
    private final T value;
    public Functor(T value) {
        this.value = value;
    }
    public <R> Functor<R> map(Function<T, R> mapper) {
        return new Functor<>(mapper.apply(value));
    }
    // getter
}
```

我们可以注意到，`map()`方法负责执行操作。对于新的类，我们定义了一个最终值属性。该属性是函数将被应用的地方。此外，我们需要一种比较值的方法。让我们将这个函数添加到`Functor`类中:

```java
public class Functor<T> {
    // Definitions
    boolean eq(T other) {
        return value.equals(other);
    }
    // Getter
}
```

在这个例子中，`Functor`类是泛型的，因为它接受一个类型参数`T`,该参数指定存储在类中的值的类型。`map`方法接受一个`Function`对象，该对象接受一个`T`类型的值并返回一个`R`类型的值。然后，`map`方法通过将函数应用于原始值来创建一个新的`Functor`对象，并返回它。

下面是如何使用该仿函数类的示例:

```java
@Test
public void whenProvideAValue_ShouldMapTheValue() {
    Functor<Integer> functor = new Functor<>(5);
    Function<Integer, Integer> addThree = (num) -> num + 3;
    Functor<Integer> mappedFunctor = functor.map(addThree);
    assertEquals(8, mappedFunctor.getValue());
}
```

## 5.法律验证程序

所以，我们需要测试一下。在我们的第一个方法之后，让我们使用我们的`Functor`类来演示函子法则。首先是同一律。在这种情况下，我们的代码片段是:

```java
@Test
public void whenApplyAnIdentityToAFunctor_thenResultIsEqualsToInitialValue() {
    String value = "baeldung";
    //Identity
    Functor<String> identity = new Functor<>(value).map(Function.identity());
    assertTrue(identity.eq(value));
}
```

在刚刚给出的例子中，我们使用了在`Function`类中可用的`identity`方法。结果`Functor`返回的值不受影响，仍然与作为参数传递的值相同。这种行为证明了身份法则得到了遵守。

下一步是应用第二定律。在开始我们的实现之前，我们需要定义一些假设。

*   `f`是一个将类型`T`和`R`相互映射的函数。
*   `g`是一个将类型`R`和`U`相互映射的函数。

之后，我们准备实施我们的测试来演示复合/结合法则。下面是我们实现的一段代码:

```java
@Test
public void whenApplyAFunctionToOtherFunction_thenResultIsEqualsBetweenBoth() {
    int value = 100;
    Function<Integer, String> f = Object::toString;
    Function<String, Long> g = Long::valueOf;
    Functor<Long> left = new Functor<>(value).map(f).map(g);
    Functor<Long> right = new Functor<>(value).map(f.andThen(g));
    assertTrue(left.eq(100L));
    assertTrue(right.eq(100L));
}
```

在我们的代码片段中，我们定义了两个标记为`f`和`g`的函数。之后，我们使用两种不同的映射策略构建了两个`Functors`，一个名为`left`，另一个名为`right`。两个`Functors`最终产生相同的输出。因此，我们成功地实施了第二项法律。

## 6.Java 8 之前的函子

到目前为止，我们已经看到了使用 Java 8 中引入的`java.util.function.Function`接口的代码示例。假设我们使用的是 Java 的早期版本。在这种情况下，我们可以使用类似的接口或创建我们自己的函数接口来表示一个接受单个参数并返回结果的函数。

另一方面，我们可以通过使用 Enum 的功能来设计一个`Functor`。虽然这不是最优答案，但它确实符合函子法则，也许最重要的是，它完成了工作。让我们定义一下我们的`EnumFunctor`类:

```java
public enum EnumFunctor {
    PLUS {
        public int apply(int a, int b) {
            return a + b;
        }
    }, MINUS {
        public int apply(int a, int b) {
            return a - b;
        }
    }, MULTIPLY {
        public int apply(int a, int b) {
            return a * b;
        }
    }, DIVIDE {
        public int apply(int a, int b) {
            return a / b;
        }
    };
    public abstract int apply(int a, int b);
}
```

在这个例子中，对每个常量值调用`apply`方法，使用两个整数作为参数。该方法执行必要的数学运算并返回结果。此外，`abstract`关键字在这个例子中被用来表示`apply`过程不是在`Enum`本身中实现的，而是必须由每个常数值实现的。现在，让我们测试我们的实现:

```java
@Test
public void whenApplyOperationsToEnumFunctors_thenGetTheProperResult() {
    assertEquals(15, EnumFunctor.PLUS.apply(10, 5));
    assertEquals(5, EnumFunctor.MINUS.apply(10, 5));
    assertEquals(50, EnumFunctor.MULTIPLY.apply(10, 5));
    assertEquals(2, EnumFunctor.DIVIDE.apply(10, 5));
}
```

## 7.结论

在本文中，我们首先描述了什么是`Functor`。然后，我们进入它的法律定义。之后，我们在 Java 8 中实现了一些代码示例来演示`Functor`的使用。此外，我们通过例子证明了这两个函子定律。最后，我们简要说明了如何在 Java 8 之前的 Java 版本中使用函子，并提供了一个带有`Enum`的例子。

像往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221210082018/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-functional)