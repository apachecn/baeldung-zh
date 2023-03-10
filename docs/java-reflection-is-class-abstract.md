# 使用反射检查 Java 类是否是“抽象的”

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-reflection-is-class-abstract>

## 1.概观

在这个快速教程中，我们将讨论如何通过使用[反射](/web/20221208143837/https://www.baeldung.com/java-reflection) API 在 Java 中检查一个类是否是*抽象的*。

## 2.示例类和接口

为了演示这一点，我们将创建一个`AbstractExample`类和一个`InterfaceExample`接口:

```java
public abstract class AbstractExample {

    public abstract LocalDate getLocalDate();

    public abstract LocalTime getLocalTime();
}

public interface InterfaceExample {
}
```

## 3.`Modifier#isAbstract`法

我们可以通过使用反射 API 中的 [`Modifier#isAbstract`](https://web.archive.org/web/20221208143837/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/Modifier.html#isAbstract(int)) 方法来检查一个类是否为`abstract`:

```java
@Test
void givenAbstractClass_whenCheckModifierIsAbstract_thenTrue() throws Exception {
    Class<AbstractExample> clazz = AbstractExample.class;

    Assertions.assertTrue(Modifier.isAbstract(clazz.getModifiers()));
}
```

在上面的例子中，我们首先获得我们想要测试的类的实例。**一旦我们有了类引用，我们就可以调用` Modifier#isAbstract`方法**。如我们所料，如果类是`abstract`，它返回`true`，否则，它返回`false`。

值得一提的是，**和`interface`同时也是`abstract`和**。我们可以通过一种测试方法来验证它:

```java
@Test
void givenInterface_whenCheckModifierIsAbstract_thenTrue() {
    Class<InterfaceExample> clazz = InterfaceExample.class;

    Assertions.assertTrue(Modifier.isAbstract(clazz.getModifiers()));
} 
```

如果我们执行上面的测试方法，它就会通过。

反射 API 也提供了一个 [`isInterface()`](https://web.archive.org/web/20221208143837/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/Modifier.html#isInterface(int)) 方法。如果我们想检查一个给定的类是否是`abstract`而不是`interface`，我们可以结合这两种方法:

```java
@Test
void givenAbstractClass_whenCheckIsAbstractClass_thenTrue() {
    Class<AbstractExample> clazz = AbstractExample.class;
    int mod = clazz.getModifiers();

    Assertions.assertTrue(Modifier.isAbstract(mod) && !Modifier.isInterface(mod));
}
```

让我们也验证一个具体的类返回适当的结果:

```java
@Test
void givenConcreteClass_whenCheckIsAbstractClass_thenFalse() {
    Class<Date> clazz = Date.class;
    int mod = clazz.getModifiers();

    Assertions.assertFalse(Modifier.isAbstract(mod) && !Modifier.isInterface(mod));
} 
```

## 4.结论

在本教程中，我们已经看到了如何检查一个类是否是`abstract`。

此外，我们已经通过一个例子说明了如何检查一个类是否是一个`abstract`类而不是一个`interface`类。

和往常一样，这个例子的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143837/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-reflection-2)