# Java 15 中的隐藏类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hidden-classes>

## 1.概观

Java 15 引入了大量的[特性](/web/20220815034151/https://www.baeldung.com/java-15-new)。在本文中，我们将讨论在 [JEP-371](https://web.archive.org/web/20220815034151/https://openjdk.java.net/jeps/371) 下被称为隐藏类的新特性。这个特性是作为[不安全 API](/web/20220815034151/https://www.baeldung.com/java-unsafe) 的替代而引入的，在 JDK 之外不推荐使用。

隐藏类特性对任何使用动态字节码或 JVM 语言的人都很有帮助。

## 2.什么是隐藏类？

动态生成的类为低延迟应用程序提供了效率和灵活性。它们只在有限的时间内需要。在静态生成的类的生存期内保留它们会增加内存占用。针对这种情况的现有解决方案，如按类加载器，既麻烦又低效。

从 Java 15 开始，隐藏类已经成为生成动态类的标准方式。

隐藏类是不能被字节码或其他类直接使用的类。尽管它被称为一个类，但它应该被理解为一个隐藏的类或接口。它也可以被定义为[访问控制嵌套](https://web.archive.org/web/20220815034151/https://openjdk.java.net/jeps/181)的成员，并且可以独立于其他类卸载。

## 3.隐藏类的属性

让我们来看看这些动态生成的类的属性:

*   不可发现——隐藏类在字节码链接期间不可被 JVM 发现，也不可被显式使用类装入器的程序发现。反射方法`Class::forName`、`ClassLoader::findLoadedClass`、`Lookup::findClass`都找不到。
*   我们不能将隐藏类用作超类、字段类型、返回类型或参数类型。
*   隐藏类中的代码可以直接使用它，而不依赖于类对象。
*   隐藏类中声明的字段是不可修改的，不管它们的可访问标志是什么。
*   它用不可发现的类扩展了访问控制嵌套。
*   即使其概念上的定义类装入器仍然可达，它也可能被卸载。
*   默认情况下，堆栈跟踪不会显示隐藏类的方法或名称，但是，调整 JVM 选项可以显示它们。

## 4.创建隐藏类

隐藏类不是由任何类加载器创建的。它具有与查找类相同的定义类加载器、运行时包和保护域。

首先，让我们创建一个`Lookup`对象:

```java
MethodHandles.Lookup lookup = MethodHandles.lookup();
```

`Lookup::defineHiddenClass`方法创建隐藏类。此方法接受字节数组。

为了简单起见，我们将定义一个简单的名为`HiddenClass`的类，它有一个将给定的字符串转换成大写的方法:

```java
public class HiddenClass {
    public String convertToUpperCase(String s) {
        return s.toUpperCase();
    }
}
```

让我们获取类的路径，并将其加载到输入流中。之后，我们将使用`IOUtils.toByteArray()`把这个类转换成字节:

```java
Class<?> clazz = HiddenClass.class;
String className = clazz.getName();
String classAsPath = className.replace('.', '/') + ".class";
InputStream stream = clazz.getClassLoader()
    .getResourceAsStream(classAsPath);
byte[] bytes = IOUtils.toByteArray();
```

最后，我们将这些构造的字节传递给`Lookup::defineHiddenClass`:

```java
Class<?> hiddenClass = lookup.defineHiddenClass(IOUtils.toByteArray(stream),
  true, ClassOption.NESTMATE).lookupClass();
```

第二个`boolean`参数`true`初始化该类。第三个参数`ClassOption.NESTMATE`指定创建的隐藏类将作为嵌套添加到查找类，这样它可以访问同一个嵌套中所有类和接口的`private`成员。

假设我们想要将隐藏类与其类加载器`ClassOption.STRONG`强绑定。这意味着只有当隐藏类的定义加载器不可达时，隐藏类才能被卸载。

## 5.使用隐藏类

隐藏类由在运行时生成类并通过反射间接使用它们的框架使用。

在上一节中，我们看了如何创建一个隐藏类。在这一节中，我们将看到如何使用它并创建一个实例。

因为对从`Lookup.defineHiddenClass`获得的类进行造型不可能使用任何其他类对象，所以我们使用`Object`来存储隐藏的类实例。如果我们希望转换隐藏类，我们可以定义一个接口并创建一个实现该接口的隐藏类:

```java
Object hiddenClassObject = hiddenClass.getConstructor().newInstance();
```

现在，让我们从隐藏类中获取方法。获得该方法后，我们将像调用任何其他标准方法一样调用它:

```java
Method method = hiddenClassObject.getClass()
    .getDeclaredMethod("convertToUpperCase", String.class); Assertions.assertEquals("HELLO", method.invoke(hiddenClassObject, "Hello"));
```

现在，我们可以通过调用隐藏类的一些方法来验证它的一些属性:

方法`isHidden()`将为此类返回`true`:

```java
Assertions.assertEquals(true, hiddenClass.isHidden());
```

此外，由于隐藏类没有实际的名称，它的规范名称将是`null`:

```java
Assertions.assertEquals(null, hiddenClass.getCanonicalName());
```

隐藏类将拥有与执行查找的类相同的定义加载器。由于查找发生在同一个类中，下面的断言将会成功:

```java
Assertions.assertEquals(this.getClass()
    .getClassLoader(), hiddenClass.getClassLoader());
```

如果我们试图通过任何方法访问隐藏类，他们将抛出`ClassNotFoundException`。这是显而易见的，因为隐藏的类名非常不常见，并且没有限定，因此对其他类来说是可见的。让我们检查几个断言来证明隐藏类是不可发现的:

```java
Assertions.assertThrows(ClassNotFoundException.class, () -> Class.forName(hiddenClass.getName())); Assertions.assertThrows(ClassNotFoundException.class, () -> lookup.findClass(hiddenClass.getName()));
```

注意，其他类使用隐藏类的唯一方式是通过它的`Class`对象。

## 6.匿名类与隐藏类

我们在前几节中创建了一个隐藏类，并使用了它的一些属性。现在，让我们详细说明匿名类(没有显式名称的内部类)和隐藏类之间的区别:

*   匿名类有一个动态生成的名称，中间有一个$号，而从`com.baeldung.reflection.hiddenclass.HiddenClass`派生的隐藏类是`com.baeldung.reflection.hiddenclass.HiddenClass/1234.`
*   匿名类是用`Unsafe::defineAnonymousClass`实例化的，但不推荐使用，而`Lookup::defineHiddenClass`实例化了一个隐藏类`.`
*   隐藏类不支持常量池修补。它有助于定义匿名类，它们的常量池条目已经解析为具体值。
*   与隐藏类不同，匿名类可以访问宿主类的`protected`成员，即使它在不同的包而不是子类中。
*   匿名类可以包含其他类来访问其成员，但隐藏类不能包含其他类。

虽然**隐藏类不是匿名类**的替代品，但是它们正在取代 JDK 中匿名类的一些用法。**从 Java 15 开始，lambda 表达式使用隐藏类**。

## 7.结论

在本文中，我们详细讨论了一个叫做隐藏类的新语言特性。和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220815034151/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-15)