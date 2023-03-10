# 在 Java 中使用反射检查方法是否是静态的

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-check-method-is-static>

## 1.概观

在这个快速教程中，我们将讨论如何通过使用[反射](/web/20220525124441/https://www.baeldung.com/java-reflection) API 来检查一个方法是否在 Java 中。

## 2.例子

为了演示这一点，我们将使用一些静态方法创建`StaticUtility` 类:

```java
public class StaticUtility {

    public static String getAuthorName() {
        return "Umang Budhwar";
    }

    public static LocalDate getLocalDate() {
        return LocalDate.now();
    }

    public static LocalTime getLocalTime() {
        return LocalTime.now();
    }
}
```

## 3.检查方法是否为`static`

**我们可以通过使用`[Modifier](https://web.archive.org/web/20220525124441/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/Modifier.html).isStatic`方法**来检查一个方法是否为`static`:

```java
@Test
void whenCheckStaticMethod_ThenSuccess() throws Exception {
    Method method = StaticUtility.class.getMethod("getAuthorName", null);
    Assertions.assertTrue(Modifier.isStatic(method.getModifiers()));
}
```

在上面的例子中，我们首先通过使用 `Class.getMethod`方法获得了我们想要测试的方法的实例。一旦我们有了方法引用，我们需要做的就是调用 `Modifier.isStatic`方法。

## 4.获取一个类的所有方法

既然我们已经知道了如何检查一个方法是否是`static`，我们可以很容易地列出一个类的所有`static`方法:

```java
@Test
void whenCheckAllStaticMethods_thenSuccess() {
    List<Method> methodList = Arrays.asList(StaticUtility.class.getMethods())
      .stream()
      .filter(method -> Modifier.isStatic(method.getModifiers()))
      .collect(Collectors.toList());
    Assertions.assertEquals(3, methodList.size());
} 
```

在上面的代码中，我们刚刚验证了我们的类`StaticUtility`中的`static`方法的总数。

## 5.结论

在本教程中，我们已经看到了如何检查一个方法是否是`static`。我们还看到了如何获取一个类的所有`static`方法。

和往常一样，这个例子的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220525124441/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-reflection-2)