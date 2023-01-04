# Java 中. getClass()和. Class 的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-getclass-vs-class>

## 1.概观

在 Java 中，**类 [`java.lang.Class`](https://web.archive.org/web/20221128034827/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Class.html) 是所有反射操作**的入口点。一旦我们有了一个对象`java.lang.Class`，我们就可以调用相应的方法来获得反射类的对象。

在本教程中，我们将讨论获取一个对象`java.lang.Class`的两种不同方法的区别:

*   调用`[Object.getClass()](https://web.archive.org/web/20221128034827/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html#getClass())`方法
*   使用`.class`语法

## 2.对这两种方法的简短介绍

`Object.getClass()`方法是`Object`类的一个实例方法。如果我们有一个对象，我们可以调用`object.getClass()`来获取该类型的`Class`对象。

类似地，我们可以使用`ClassName.class`语法来获取类型的`Class`对象。一个例子可以清楚地说明这一点:

```java
@Test
public void givenObjectAndType_whenGettingClassObject_thenTwoMethodsHaveTheSameResult() {
    String str = "I am an object of the String class";

    Class fromStrObject = str.getClass();
    Class clazz = String.class;

    assertSame(fromStrObject, clazz);
} 
```

在上面的测试方法中，我们尝试使用我们提到的两种方法来获取`String`类的`Class `对象。最后，断言方法告诉我们两个`Class`对象是同一个实例。

但是，这两种方法之间存在差异。让我们仔细看看它们。

## 3.运行时类型与静态类型

让我们快速回顾一下前面的例子。当我们调用`str.getClass()`方法时，我们得到了`str`对象的运行时类型。另一方面，`String.class` 静态地评估`String`类。在这个例子中，`str`和`String.class`的运行时类型是相同的。

但是，如果阶级继承入党，他们就可以不同了。让我们看两个简单的类:

```java
public class Animal {
    protected int numberOfEyes;
}

public class Monkey extends Animal {
    // monkey stuff
}
```

现在让我们实例化一个`Animal`类的对象并做另一个测试:

```java
@Test
public void givenClassInheritance_whenGettingRuntimeTypeAndStaticType_thenGetDifferentResult() {
    Animal animal = new Monkey();

    Class runtimeType = animal.getClass();
    Class staticType = Animal.class;

    assertSame(staticType, runtimeType);
} 
```

如果我们运行上面的测试，我们将得到一个测试失败:

```java
java.lang.AssertionError: ....
Expected :class com.baeldung.getclassobject.Animal
Actual   :class com.baeldung.getclassobject.Monkey
```

在测试方法中，即使我们通过`Animal animal = new Monkey();`而不是`Monkey animal = new Monkey();`实例化了`animal`对象，但是`animal`对象的运行时类型仍然是`Monkey. `，这是因为`animal`对象在运行时是`Monkey`的一个实例。

然而，当我们得到`Animal`类的静态类型时，类型总是`Animal`。

## 4.处理原始类型

当我们编写 Java 代码时，我们经常使用原始类型。让我们尝试使用`object.getClass()`方法获得一个原始类型的`Class`对象:

```java
int number = 7;
Class numberClass = number.getClass();
```

如果我们试图编译上面的代码，我们会得到一个编译错误:

```java
Error: java: int cannot be dereferenced
```

编译器不能取消引用`number`变量，因为它是一个原始变量。因此，**`object.getClass()`方法不能帮助我们得到一个原始类型的`Class`对象。**

让我们看看是否可以使用`.class`语法获得原始类型:

```java
@Test
public void givenPrimitiveType_whenGettingClassObject_thenOnlyStaticTypeWorks() {
    Class intType = int.class;
    assertNotNull(intType);
    assertEquals("int", intType.getName());
    assertTrue(intType.isPrimitive());
} 
```

所以，我们可以通过`int.class`获得`int` 原语类型的`Class`对象。在 Java 版本 9 和更高版本中，原始类型的`Class`对象属于`java.base` 模块。

正如测试所显示的，**`.class`语法是一种简单的方法来获得基本类型的`Class`对象。**

## 5.获取没有实例的类

我们已经知道,`object.getClass()`方法可以给我们其运行时类型的`Class`对象。

现在，让我们考虑这样一种情况，我们想要获得一个类型的`Class`对象，但是我们不能获得目标类型的实例，因为它是一个`abstract`类，一个`interface, `或一些类不允许实例化:

```java
public abstract class SomeAbstractClass {
    // ...
}

interface SomeInterface {
   // some methods ...
}

public class SomeUtils {
    private SomeUtils() {
        throw new RuntimeException("This Util class is not allowed to be instantiated!");
    }
    // some public static methods...
} 
```

在这些情况下，我们不能使用`object.getClass()`方法获得那些类型的`Class`对象，但是**我们仍然可以使用`.class`语法获得它们的`Class`对象**:

```java
@Test
public void givenTypeCannotInstantiate_whenGetTypeStatically_thenGetTypesSuccefully() {
    Class interfaceType = SomeInterface.class;
    Class abstractClassType = SomeAbstractClass.class;
    Class utilClassType = SomeUtils.class;

    assertNotNull(interfaceType);
    assertTrue(interfaceType.isInterface());
    assertEquals("SomeInterface", interfaceType.getSimpleName());

    assertNotNull(abstractClassType);
    assertEquals("SomeAbstractClass", abstractClassType.getSimpleName());

    assertNotNull(utilClassType);
    assertEquals("SomeUtils", utilClassType.getSimpleName());
} 
```

如上面的测试所示，`.class`语法可以获得这些类型的`Class`对象。

因此，**当我们想要有`Class`对象，但是我们不能得到类型的实例时，`.class`语法是可行的方法。**

## 6.结论

在本文中，我们学习了两种不同的方法来获取类型的`Class`对象:`object.getClass()`方法和`.class`语法。

稍后，我们讨论了这两种方法之间的区别。下表可以给我们一个清晰的概述:

|  | `object.getClass()` | `SomeClass.class` |
| **类对象** | `object`的运行时类型 | `SomeClass`的静态类型 |
| **原始类型** | — | 直接工作 |
| **接口、抽象类或不能实例化的类** | — | 直接工作 |

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221128034827/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-3)