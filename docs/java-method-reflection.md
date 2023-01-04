# 使用 Java 反射在运行时调用方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-method-reflection>

## 1。概述

在这篇短文中，我们将快速了解如何使用 Java 反射 API 在运行时**调用方法。**

## 2。准备就绪

让我们创建一个简单的类，我们将在下面的例子中使用它:

```
public class Operations {
    public double publicSum(int a, double b) {
        return a + b;
    }

    public static double publicStaticMultiply(float a, long b) {
        return a * b;
    }

    private boolean privateAnd(boolean a, boolean b) {
        return a && b;
    }

    protected int protectedMax(int a, int b) {
        return a > b ? a : b;
    }
}
```

## 3。获得一个`Method`对象

首先，我们需要得到一个反映我们想要调用的方法的`Method`对象。 `Class`对象代表定义该方法的类型，它提供了两种实现方式。

### 3.1。`getMethod()`

我们可以使用`getMethod()`来查找该类或其任何超类的任何公共方法。

基本上，它接收方法名作为第一个参数，后面是方法参数的类型:

```
Method sumInstanceMethod
  = Operations.class.getMethod("publicSum", int.class, double.class);

Method multiplyStaticMethod
  = Operations.class.getMethod(
    "publicStaticMultiply", float.class, long.class);
```

### 3.2。`getDeclaredMethod()`

我们可以使用`getDeclaredMethod()`来获得任何一种方法。这包括公共的、受保护的、默认的访问，甚至私有的方法，但不包括继承的方法。

它接收与`getMethod()`相同的参数:

```
Method andPrivateMethod
  = Operations.class.getDeclaredMethod(
    "privateAnd", boolean.class, boolean.class);
```

```
Method maxProtectedMethod
  = Operations.class.getDeclaredMethod("protectedMax", int.class, int.class);
```

## 4。调用方法

有了`Method`实例，我们现在可以调用`invoke()`来执行底层方法并获得返回的对象。

### 4.1。实例方法

要调用实例方法，`invoke()`的第一个参数必须是反映被调用方法的`Method`的实例:

```
@Test
public void givenObject_whenInvokePublicMethod_thenCorrect() {
    Method sumInstanceMethod
      = Operations.class.getMethod("publicSum", int.class, double.class);

    Operations operationsInstance = new Operations();
    Double result
      = (Double) sumInstanceMethod.invoke(operationsInstance, 1, 3);

    assertThat(result, equalTo(4.0));
}
```

### 4.2。静态方法

由于这些方法不需要调用实例，我们可以将`null`作为第一个参数传递:

```
@Test
public void givenObject_whenInvokeStaticMethod_thenCorrect() {
    Method multiplyStaticMethod
      = Operations.class.getDeclaredMethod(
        "publicStaticMultiply", float.class, long.class);

    Double result
      = (Double) multiplyStaticMethod.invoke(null, 3.5f, 2);

    assertThat(result, equalTo(7.0));
}
```

## 5。方法可访问性

**默认情况下，并不是所有反射的方法都是`accessible`。**这意味着 JVM 在调用访问控制检查时会强制执行这些检查。

例如，如果我们试图调用其定义类之外的私有方法，或者从子类或其类的包之外调用受保护的方法，我们将得到一个`IllegalAccessException`:

```
@Test(expected = IllegalAccessException.class)
public void givenObject_whenInvokePrivateMethod_thenFail() {
    Method andPrivateMethod
      = Operations.class.getDeclaredMethod(
        "privateAnd", boolean.class, boolean.class);

    Operations operationsInstance = new Operations();
    Boolean result
      = (Boolean) andPrivateMethod.invoke(operationsInstance, true, false);

    assertFalse(result);
}

@Test(expected = IllegalAccessException.class)
public void givenObject_whenInvokeProtectedMethod_thenFail() {
    Method maxProtectedMethod
      = Operations.class.getDeclaredMethod(
        "protectedMax", int.class, int.class);

    Operations operationsInstance = new Operations();
    Integer result
      = (Integer) maxProtectedMethod.invoke(operationsInstance, 2, 4);

    assertThat(result, equalTo(4));
}
```

### 5.1。`AccessibleObject` # `setAccesible`

**通过在反射的方法对象上调用`setAccesible(true)`，JVM 取消了访问控制检查**，并允许我们调用方法而不抛出异常:

```
@Test
public void givenObject_whenInvokePrivateMethod_thenCorrect() throws Exception {
    Method andPrivatedMethod = Operations.class.getDeclaredMethod("privateAnd", boolean.class, boolean.class);
    andPrivatedMethod.setAccessible(true);

    Operations operationsInstance = new Operations();
    Boolean result = (Boolean) andPrivatedMethod.invoke(operationsInstance, true, false);

    assertFalse(result);
}
```

### 5.2。`AccessibleObject#canAccess`

Java 9 提供了一种全新的方式来**检查调用者是否可以访问反射的方法对象**。

为此，它提供了`canAccess` 作为不推荐使用的方法`[isAccessible​](https://web.archive.org/web/20220628061807/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/AccessibleObject.html#isAccessible()).`的替代

让我们来看看它的实际应用:

```
@Test
public void givenObject_whenInvokePrivateMethod_thenCheckAccess() throws Exception {
    Operations operationsInstance = new Operations();
    Method andPrivatedMethod = Operations.class.getDeclaredMethod("privateAnd", boolean.class, boolean.class);
    boolean isAccessEnabled = andPrivatedMethod.canAccess(operationsInstance);

    assertFalse(isAccessEnabled);
 }
```

在用`setAccessible(true)`将 a `ccessible`标志设置为`true`之前，我们可以使用`canAccess`来检查调用者是否已经可以访问反射的方法。

### 5.3。`AccessibleObject#trySetAccessible`

是另一种简便的方法，我们可以用它来使一个反射的物体变得可访问。

这个新方法的好处是，如果访问不能被启用，那么**将返回`false`。但是，老方法`setAccessible(true)`一失败就抛出`InaccessibleObjectException`。**

让我们举例说明`trySetAccessible`方法的使用:

```
@Test
public void givenObject_whenInvokePublicMethod_thenEnableAccess() throws Exception {
    Operations operationsInstance = new Operations();
    Method andPrivatedMethod = Operations.class.getDeclaredMethod("privateAnd", boolean.class, boolean.class);
    andPrivatedMethod.trySetAccessible();
    boolean isAccessEnabled = andPrivatedMethod.canAccess(operationsInstance);

    assertTrue(isAccessEnabled);
}
```

## 6。结论

在这篇简短的文章中，我们看到了如何通过反射在运行时调用类的实例和静态方法。我们还展示了如何更改反射方法对象上的 accessible 标志，以便在调用私有和受保护的方法时抑制 Java 访问控制检查。

和往常一样，示例代码可以在 Github 上找到[。](https://web.archive.org/web/20220628061807/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-11-2)