# 番石榴反射工具指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-reflection>

## 1。概述

在本文中，我们将关注`Guava``reflection`API——与标准的 Java 反射 API 相比，它无疑更加通用。

我们将使用`Guava`在运行时捕获泛型类型，我们也将很好地利用`[Invokable](https://web.archive.org/web/20220815041353/https://google.github.io/guava/releases/21.0/api/docs/com/google/common/reflect/Invokable.html)`。

## 2。运行时捕获泛型类型

在 Java 中，泛型是通过类型擦除来实现的。这意味着泛型类型信息只在编译时可用，在运行时不再可用。

例如，`List<String>,`关于泛型类型的信息在运行时被[删除。由于这个事实，在运行时传递通用的`Class`对象是不安全的。](https://web.archive.org/web/20220815041353/https://docs.oracle.com/javase/tutorial/java/generics/erasure.html)

我们最终可能会将两个具有不同泛型类型的列表分配给同一个引用，这显然不是一个好主意:

```java
List<String> stringList = Lists.newArrayList();
List<Integer> intList = Lists.newArrayList();

boolean result = stringList.getClass()
  .isAssignableFrom(intList.getClass());

assertTrue(result);
```

由于类型删除，方法`isAssignableFrom()` 无法知道列表的实际泛型类型。它基本上比较了两种类型，这两种类型只是一个`List` ，没有任何关于实际类型的信息。

通过使用标准的 Java 反射 API，我们可以检测方法和类的一般类型。如果我们有一个返回一个`List<String>`的方法，我们可以使用反射来获得该方法的返回类型——一个代表`List<String>`的 [`ParameterizedType`](https://web.archive.org/web/20220815041353/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/ParameterizedType.html) 。

`TypeToken`类使用这种变通方法来允许泛型类型的操作。我们可以使用`TypeToken` 类来捕获泛型列表的实际类型，并检查它们是否真的可以被同一个引用引用:

```java
TypeToken<List<String>> stringListToken
  = new TypeToken<List<String>>() {};
TypeToken<List<Integer>> integerListToken
  = new TypeToken<List<Integer>>() {};
TypeToken<List<? extends Number>> numberTypeToken
  = new TypeToken<List<? extends Number>>() {};

assertFalse(stringListToken.isSubtypeOf(integerListToken));
assertFalse(numberTypeToken.isSubtypeOf(integerListToken));
assertTrue(integerListToken.isSubtypeOf(numberTypeToken));
```

只有`integerListToken`可以被分配给类型`nubmerTypeToken` 的引用，因为一个`Integer` 类扩展了一个`Number` 类`.`

## 3。使用`TypeToken` 捕捉复杂类型

假设我们想要创建一个泛型参数化类，并且我们想要在运行时获得关于泛型类型的信息。我们可以创建一个以`TypeToken` 为字段的类来捕获信息:

```java
abstract class ParametrizedClass<T> {
    TypeToken<T> type = new TypeToken<T>(getClass()) {};
}
```

然后，当创建该类的实例时，泛型类型将在运行时可用:

```java
ParametrizedClass<String> parametrizedClass = new ParametrizedClass<String>() {};

assertEquals(parametrizedClass.type, TypeToken.of(String.class));
```

我们还可以创建一个具有多个泛型类型的复杂类型的`TypeToken` ，并在运行时检索每个泛型类型的信息:

```java
TypeToken<Function<Integer, String>> funToken
  = new TypeToken<Function<Integer, String>>() {};

TypeToken<?> funResultToken = funToken
  .resolveType(Function.class.getTypeParameters()[1]);

assertEquals(funResultToken, TypeToken.of(String.class));
```

我们得到了`Function`的实际返回类型，也就是一个`String.` 我们甚至可以得到 map 中条目的类型:

```java
TypeToken<Map<String, Integer>> mapToken
  = new TypeToken<Map<String, Integer>>() {};

TypeToken<?> entrySetToken = mapToken
  .resolveType(Map.class.getMethod("entrySet")
  .getGenericReturnType());

assertEquals(
  entrySetToken,
  new TypeToken<Set<Map.Entry<String, Integer>>>() {}); 
```

这里我们使用 Java 标准库中的反射方法`getMethod()` 来捕获方法的返回类型。

## 4。`Invokable`

`Invokable`是 [`java.lang.reflect.Method`](https://web.archive.org/web/20220815041353/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/Method.html) 和 [`java.lang.reflect.Constructor`](https://web.archive.org/web/20220815041353/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/Constructor.html) 的流畅包装器。它在标准 Java `reflection` API 的基础上提供了一个更简单的 API。假设我们有一个类，它有两个公共方法，其中一个是 final 方法:

```java
class CustomClass {
    public void somePublicMethod() {}

    public final void notOverridablePublicMethod() {}
}
```

现在让我们使用 Guava API 和 Java 标准的`reflection` API 来检查`somePublicMethod()` :

```java
Method method = CustomClass.class.getMethod("somePublicMethod");
Invokable<CustomClass, ?> invokable 
  = new TypeToken<CustomClass>() {}
  .method(method);

boolean isPublicStandradJava = Modifier.isPublic(method.getModifiers());
boolean isPublicGuava = invokable.isPublic();

assertTrue(isPublicStandradJava);
assertTrue(isPublicGuava);
```

这两种变体之间没有太大的区别，但是在 Java 中，检查一个方法是否是可重写的是一项非常重要的任务。幸运的是，`Invokable` 类中的`isOverridable()` 方法使这变得更容易:

```java
Method method = CustomClass.class.getMethod("notOverridablePublicMethod");
Invokable<CustomClass, ?> invokable
 = new TypeToken<CustomClass>() {}.method(method);

boolean isOverridableStandardJava = (!(Modifier.isFinal(method.getModifiers()) 
  || Modifier.isPrivate(method.getModifiers())
  || Modifier.isStatic(method.getModifiers())
  || Modifier.isFinal(method.getDeclaringClass().getModifiers())));
boolean isOverridableFinalGauava = invokable.isOverridable();

assertFalse(isOverridableStandardJava);
assertFalse(isOverridableFinalGauava);
```

我们看到，即使如此简单的操作也需要使用标准的`reflection` API 进行大量检查。`Invokable` 类将这一点隐藏在简单易用且非常简洁的 API 后面。

## 5。结论

在本文中，我们研究了 Guava 反射 API，并将其与标准 Java 进行了比较。我们看到了如何在运行时捕获泛型类型，以及`Invokable` 类如何为使用反射的代码提供优雅且易于使用的 API。

所有这些示例和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220815041353/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-utilities)中找到——这是一个 Maven 项目，因此它应该很容易导入和运行。