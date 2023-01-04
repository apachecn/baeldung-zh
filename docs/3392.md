# 在 Java 中调用私有方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-call-private-method>

## 1.概观

虽然在 Java 中使用了`private`方法来防止它们被所属类外部调用，但出于某种原因，我们可能仍然需要调用它们。

为了实现这一点，我们需要解决 Java 的访问控制。这可能帮助我们到达一个库的角落，或者允许我们测试一些通常应该保持私有的代码。

在这个简短的教程中，我们将看看如何验证一个方法的功能，而不管它的可见性。我们将考虑两种不同的方法: [Java 反射 API](/web/20221126233137/https://www.baeldung.com/java-reflection) 和 Spring 的 [`ReflectionTestUtils`](/web/20221126233137/https://www.baeldung.com/spring-reflection-test-utils) 。

## 2.能见度超出我们的控制

对于我们的例子，让我们使用一个在`long`数组上操作的实用程序类`LongArrayUtil`。我们班有两个`indexOf`方法:

```
public static int indexOf(long[] array, long target) {
    return indexOf(array, target, 0, array.length);
}

private static int indexOf(long[] array, long target, int start, int end) {
    for (int i = start; i < end; i++) {
        if (array[i] == target) {
            return i;
        }
    }
    return -1;
} 
```

让我们假设这些方法的可见性是不可改变的，然而我们想要调用私有的`indexOf`方法。

## 3.Java 反射 API

### 3.1.用反思寻找方法

虽然编译器阻止我们调用对我们的类不可见的函数，但是我们可以通过反射调用函数。首先，我们需要访问描述我们想要调用的函数的`Method`对象:

```
Method indexOfMethod = LongArrayUtil.class.getDeclaredMethod(
  "indexOf", long[].class, long.class, int.class, int.class);
```

我们必须使用`getDeclaredMethod`来访问非私有方法。我们在具有函数的类型上调用它，在本例中是`LongArrayUtil`，我们传递参数的类型来识别正确的方法。

如果方法不存在，函数可能会失败并引发异常。

### 3.2.允许访问该方法

现在我们需要暂时提升方法的可见性:

```
indexOfMethod.setAccessible(true);
```

这种改变将持续到 JVM 停止，或者将`accessible`属性重新设置为`false.`

### 3.3.调用带有反射的方法

最后，我们在`Method` 对象上[调用`invoke`:](/web/20221126233137/https://www.baeldung.com/java-method-reflection)

```
int value = (int) indexOfMethod.invoke(
  LongArrayUtil.class, someLongArray, 2L, 0, someLongArray.length);
```

我们现在已经成功地访问了一个私有方法。

`invoke`的第一个参数是目标对象，其余的参数需要匹配我们方法的签名。在这个例子中，我们的方法是`static`，目标对象是父类——`LongArrayUtil`。对于调用实例方法，我们将传递要调用其方法的对象。

我们还应该注意到，`invoke`返回的是`Object`，它是`void`函数的`null`，需要转换成正确的类型才能使用。

## 4.弹簧`ReflectionTestUtils`

到达类的内部是测试中的一个常见问题。Spring 的测试库提供了一些快捷方式来帮助单元测试到达类。这通常可以解决单元测试特有的问题，其中测试需要访问一个私有字段，Spring 可能会在运行时实例化这个私有字段。

首先，我们需要在 pom.xml 中添加 [`spring-test`](https://web.archive.org/web/20221126233137/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-test%22) 依赖项:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.3.4</version>
    <scope>test</scope>
</dependency>
```

现在我们可以在`ReflectionTestUtils`中使用`invokeMethod`函数，它使用与上面相同的算法，并节省我们编写的代码:

```
int value = ReflectionTestUtils.invokeMethod(
  LongArrayUtil.class, "indexOf", someLongArray, 1L, 1, someLongArray.length);
```

由于这是一个测试库，我们不会期望在测试代码之外使用它。

## 5.考虑

使用反射来绕过函数可见性会带来一些风险，甚至是不可能的。我们应该考虑:

*   Java 安全管理器是否允许在我们的运行时这样做
*   我们调用的函数，如果没有编译时检查，是否会继续存在，以便我们将来调用
*   重构我们自己的代码，让事情变得更加可见和容易理解

## 6.结论

在本文中，我们研究了如何使用 Java 反射 API 和 Spring 的`ReflectionTestUtils`来访问私有方法。

和往常一样，本文的示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221126233137/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-reflection-2)