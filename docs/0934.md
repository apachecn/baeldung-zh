# Java 中的三函数接口

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-trifunction>

## 1.概观

在本文中，我们将定义一个`TriFunction` [`FunctionalInterface`](/web/20221116161839/https://www.baeldung.com/java-8-functional-interfaces) ，它表示一个接受三个参数并计算结果的函数。稍后，我们还将看到一个使用 [Vavr](/web/20221116161839/https://www.baeldung.com/vavr) 库的内置`Function3`的例子。

## 2.创建我们自己的`TriFunction`界面

从版本 8 开始，Java 定义了`[BiFunction](/web/20221116161839/https://www.baeldung.com/java-bifunction-interface) FunctionalInterface`。它表示一个接受两个参数并计算其结果的函数。为了允许函数组合，它还提供了一个`andThen()`方法，将另一个`Function`应用于`BiFunction`的结果。

**同样，我们将定义我们的`TriFunction`接口，并赋予它`andThen()`方法:**

```
@FunctionalInterface
public interface TriFunction<T, U, V, R> {

    R apply(T t, U u, V v);

    default <K> TriFunction<T, U, V, K> andThen(Function<? super R, ? extends K> after) {
        Objects.requireNonNull(after);
        return (T t, U u, V v) -> after.apply(apply(t, u, v));
    }
}
```

让我们看看如何使用这个接口。我们将定义一个采用三个`Integers`的函数，将前两个操作数相乘，然后将最后一个操作数相加:

```
static TriFunction<Integer, Integer, Integer, Integer> multiplyThenAdd = (x, y, z) -> x * y + z;
```

让我们注意，只有当两个第一操作数的乘积低于 [`Integer`最大值](/web/20221116161839/https://www.baeldung.com/cs/max-int-java-c-python)时，这个方法的结果才是准确的。

例如，我们可以使用`andThen()`方法来定义一个`TriFunction`,它:

*   首先，将`multiplyThenAdd()`应用于参数
*   然后，将一个`Function`应用于上一步的结果，该值计算一个`Integer`除以 10 的欧几里德除法的商

```
static TriFunction<Integer, Integer, Integer, Integer> multiplyThenAddThenDivideByTen = multiplyThenAdd.andThen(x -> x / 10);
```

我们现在可以编写一些快速测试来检查我们的`TriFunction`的行为是否符合预期:

```
@Test
void whenMultiplyThenAdd_ThenReturnsCorrectResult() {
    assertEquals(25, multiplyThenAdd.apply(2, 10, 5));
}

@Test
void whenMultiplyThenAddThenDivideByTen_ThenReturnsCorrectResult() {
    assertEquals(2, multiplyThenAddThenDivideByTen.apply(2, 10, 5));
}
```

最后要注意的是，`TriFunction`的操作数可以是各种类型。例如，我们可以定义一个`TriFunction`，它将一个`Integer`转换成一个`String`，或者根据一个`Boolean`条件返回另一个给定的`String`:

```
static TriFunction<Integer, String, Boolean, String> convertIntegerOrReturnStringDependingOnCondition = (myInt, myStr, myBool) -> {
    if (Boolean.TRUE.equals(myBool)) {
        return myInt != null ? myInt.toString() : "";
    } else {
        return myStr;
    }
};
```

## 3.使用 Vavr 的`Function3`

**Vavr 库已经定义了一个`Function3`接口，它具有我们想要的行为。**首先，让我们将 Vavr [依赖项](https://web.archive.org/web/20221116161839/https://search.maven.org/search?q=io.vavr)添加到我们的项目中:

```
<dependency>
    <groupId>io.vavr</groupId>
    <artifactId>vavr</artifactId>
    <version>0.10.4</version>
</dependency>
```

我们现在可以用它重新定义`multiplyThenAdd()`和`multiplyThenAddThenDivideByTen()` 方法:

```
static Function3<Integer, Integer, Integer, Integer> multiplyThenAdd = (x, y, z) -> x * y + z;

static Function3<Integer, Integer, Integer, Integer> multiplyThenAddThenDivideByTen = multiplyThenAdd.andThen(x -> x / 10);
```

如果我们需要定义多达 8 个参数的函数，使用 Vavr 可能是一个不错的选择。 `Function4`、`Function5,` ……`Function8`确实已经在库中定义好了。

## 4.结论

在本教程中，我们为一个接受 3 个参数的函数实现了自己的`FunctionalInterface`。我们还强调了 Vavr 库包含这种功能的实现。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221116161839/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-function)