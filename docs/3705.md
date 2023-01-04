# 检查 Java 中是否存在一个类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-check-class-exists>

## 1.概观

当确定使用哪个接口实现时，检查类的存在可能是有用的。这种技术通常用于旧的 JDBC 设置。

在本教程中，**我们将探索使用`Class.forName()`检查 Java 类路径**中类的存在的细微差别。

## 2.使用`Class.forName()`

我们可以使用 [Java 反射](/web/20220625173731/https://www.baeldung.com/java-reflection)，特别是`Class.forName()`，来检查一个类的存在。文档显示，如果找不到类，就会抛出一个`ClassNotFoundException`。

### 2.1.什么时候可以期待`ClassNotFoundException`

首先，让我们编写一个测试，它肯定会抛出一个 [`ClassNotFoundException`](/web/20220625173731/https://www.baeldung.com/java-classnotfoundexception-and-noclassdeffounderror) ，这样我们就可以知道我们的阳性测试是安全的:

```
@Test(expected = ClassNotFoundException.class)
public void givenNonExistingClass_whenUsingForName_thenClassNotFound() throws ClassNotFoundException {
    Class.forName("class.that.does.not.exist");
}
```

所以，我们证明了一个不存在的类会抛出一个`ClassNotFoundException`。让我们为一个确实存在的类编写一个测试:

```
@Test
public void givenExistingClass_whenUsingForName_thenNoException() throws ClassNotFoundException {
    Class.forName("java.lang.String");
}
```

这些测试证明，**运行`Class.forName()`而没有捕获到`ClassNotFoundException`相当于指定的类存在于类路径**中。然而，由于[的副作用](https://web.archive.org/web/20220625173731/https://medium.com/@bishonbopanna/java-side-effect-methods-good-bad-and-ugly-8ffa697323ec)，这并不是一个完美的解决方案。

### 2.2.副作用:类初始化

需要指出的是，**在没有指定类加载器的情况下，`Class.forName()`必须在被请求的类**上运行静态初始化器。这可能会导致意外的行为。

为了举例说明这种行为，让我们创建一个类，当它的静态初始化器块被执行时抛出一个`RuntimeException`,这样我们可以立即知道它何时被执行:

```
public static class InitializingClass {
    static {
        if (true) { //enable throwing of an exception in a static initialization block
            throw new RuntimeException();
        }
    }
}
```

我们可以从`forName()`文档中看到，如果这个方法引发的初始化失败，它会抛出一个`ExceptionInInitializerError`。

让我们编写一个测试，当试图在不指定类加载器的情况下找到我们的`InitializingClass` 时，它将期待一个`ExceptionInInitializerError`:

```
@Test(expected = ExceptionInInitializerError.class)
public void givenInitializingClass_whenUsingForName_thenInitializationError() throws ClassNotFoundException {
    Class.forName("path.to.InitializingClass");
}
```

由于类的静态初始化块的执行是一个不可见的副作用，我们现在可以看到它是如何导致性能问题甚至错误的。让我们看看如何跳过类初始化。

## 3.告诉`Class.forName()`跳过初始化

幸运的是，有一个重载的方法`forName(),`,它接受一个类加载器以及是否应该执行类初始化。

根据文档，以下调用是等效的:

```
Class.forName("Foo")
Class.forName("Foo", true, this.getClass().getClassLoader())
```

通过将`true`改为`false`，我们现在可以编写一个测试来检查我们的`InitializingClass`T3 的存在，而不触发它的静态初始化块:

```
@Test
public void givenInitializingClass_whenUsingForNameWithoutInitialization_thenNoException() throws ClassNotFoundException {
    Class.forName("path.to.InitializingClass", false, getClass().getClassLoader());
}
```

## 4.Java 9 模块

对于 Java 9+项目，还有第三个重载`Class.forName()`，它接受一个`Module`和一个`String`类名。默认情况下，这个重载不运行类初始化器。另外，值得注意的是，当请求的类不存在时，它返回`null`，而不是抛出`ClassNotFoundException`。

## 5.结论

在这个简短的教程中，我们揭示了使用`Class.forName()`时类初始化的副作用，并且发现可以使用`forName()`重载来防止这种情况发生。

本教程中所有例子的源代码可以在 GitHub 的[中找到。](https://web.archive.org/web/20220625173731/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-3)