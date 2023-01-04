# Java 中的 IllegalAccessError

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-illegalaccesserror>

## 1.概观

在这个快速教程中，我们将讨论`java.lang.IllegalAccessError`。

我们将研究一些关于何时抛出以及如何避免抛出的例子。

## 2.`IllegalAccessError`简介

当一个应用程序试图访问一个字段或者调用一个不可访问的方法时，就会抛出一个`IllegalAccessError`。

编译器会捕捉这种非法调用，但我们仍可能在运行时遇到`IllegalAccessError`。

首先，让我们观察一下`IllegalAccessError:`的类层次

```
java.lang.Object
  |_java.lang.Throwable
    |_java.lang.Error
      |_java.lang.LinkageError
        |_java.lang.IncompatibleClassChangeError
          |_java.lang.IllegalAccessError
```

它的父类是`IncompatibleClassChangeError.` 因此，该错误的原因是应用程序中一个或多个类定义的不兼容更改。

简单地说，运行时类的版本不同于编译时的版本。

## 3.这种错误是如何发生的？

让我们用一个简单的程序来理解这一点:

```
public class Class1 {
    public void bar() {
        System.out.println("SUCCESS");
    }
}

public class Class2 {
    public void foo() {
        Class1 c1 = new Class1();
        c1.bar();
    }
}
```

运行时，上面的代码调用了`Class1\.` 中的方法`bar()`，到目前为止，一切顺利。

现在，我们把`bar()`的访问修饰符更新为`private`，独立编译。

接下来，用新编译的版本替换先前定义的`Class1 (the .`类文件`)` ，并重新运行程序:

```
java.lang.IllegalAccessError: 
  class Class2 tried to access private method Class1.bar()
```

上述例外是不言自明的。方法`bar()`现在是`private`在`Class1. `显然，访问是非法的。

## 4.`IllegalAccessError`在行动

### 4.1.库更新

考虑一个在编译时使用库的应用程序，在运行时类路径中也有同样的库。

库所有者将一个公共可用的方法更新为私有的，重新构建它，但是忘记了将这一变化更新给其他方。

此外，在执行过程中，当应用程序调用这个方法(假设是公共访问)时，它会遇到一个`IllegalAccessError.`

### 4.2.接口默认方法

在接口中误用[默认方法](/web/20220625231247/https://www.baeldung.com/java-static-default-methods)是这个错误的另一个原因。

考虑以下接口和类定义:

```
interface Baeldung {
    public default void foobar() {
        System.out.println("This is a default method.");
    }
}

class Super {
    private void foobar() {
        System.out.println("Super class method foobar");
    }
}
```

同样，让我们扩展`Super `并实现`Baeldung:`

```
class MySubClass extends Super implements Baeldung {}
```

最后，让我们通过实例化`MySubClass:`来调用`foobar()`

```
new MySubClass().foobar();
```

方法 `foobar()`在`Super` 中是私有的，而`default`在`Baeldung.` 中是私有的，因此`,`可以在`MySubClass.` 的层次结构中访问

因此，编译器不会抱怨，但在运行时，我们会得到一个错误:

```
java.lang.IllegalAccessError:
  class IllegalAccessErrorExample tried to access private method 'void Super.foobar()'
```

在执行过程中，超类方法声明总是优先于接口默认方法。

从技术上讲，应该调用来自`Super`的`foobar`，但它是私有的。毫无疑问，一个`IllegalAccessError`将被抛出。

## 5.如何避免？

准确地说，如果我们遇到一个`IllegalAccessError`，我们应该主要寻找关于访问修饰符的类定义的变化。

其次，我们应该验证用私有访问修饰符覆盖的接口默认方法。

将类级别的方法公开将会达到这个目的。

## 6.结论

总之，编译器将解决大多数非法的方法调用。如果我们仍然遇到`IllegalAccesError`，我们需要研究类定义的变化。

GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20220625231247/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-3)