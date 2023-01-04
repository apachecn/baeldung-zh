# Java 中的 NoSuchFieldError

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-nosuchfielderror>

## 1.概观

在本文中，我们将展示`NoSuchFieldError`背后的原因，并探索如何解决它。

## 2.`NoSuchFieldError`

[`NoSuchFieldError`顾名思义，](https://web.archive.org/web/20221126230134/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/NoSuchFieldError.html)发生在指定字段不存在的时候。`NoSuchFieldError`扩展了 [`IncompatibleClassChangeError`](https://web.archive.org/web/20221126230134/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/IncompatibleClassChangeError.html) 类，当应用程序试图访问或修改一个对象的字段或一个类的静态字段，但该对象或类不再有那个字段时，就会抛出**。**

`IncompatibleClassChangeError`类扩展了 [`LinkageError`](https://web.archive.org/web/20221126230134/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/LinkageError.html) 类，并在我们执行不兼容的类定义更改时发生。最后，`LinkageError`扩展了`Error`,表明一个类依赖于另一个不兼容的类。

让我们借助一个例子来看看这个错误是如何发生的。作为第一步，让我们创建一个`Dependency` 类:

```
public class Dependency {
    public static String message = "Hello Baeldung!!";
}
```

然后我们将创建一个引用我们的`Dependency`类的字段的`FieldErrorExample`类:

```
public class FieldErrorExample {
    public static String getDependentMessage() {
        return Dependency.message;
    }
}
```

让我们也添加代码来检查我们是否从`Dependency`类获得了一个`message`:

```
public static void fetchAndPrint() {
    System.out.println(getDependentMessage());
} 
```

现在，我们可以使用`javac`命令编译这些文件，在使用`java` 命令执行`FieldErrorExample`类时，它将打印指定的`message`。

然而，**如果我们注释掉、移除或者改变`Dependency`类中的属性名并重新编译它，那么我们将会遇到错误**。

例如，让我们更改我们的`Dependency`类中的属性名:

```
public class Dependency {
    public static String msg = "Hello Baeldung!!";
}
```

现在，如果我们只重新编译我们的`Dependency`类`,`，然后再次执行`FieldErrorExample`，我们会遇到`NoSuchFieldError`:

```
Exception in thread "main" java.lang.NoSuchFieldError: message
```

发生上面的错误是因为`FieldErrorExample` 类仍然引用`Dependency`类的静态字段`message`，但是它不再存在——我们对`Dependency`类做了不兼容的更改。

## 3.解决错误

为了避免这个错误，我们需要**清理和编译现有的文件**。我们可以使用`javac`命令或 Maven 通过运行`mvn clean install.` 来完成这个**。通过执行这个步骤，我们将拥有所有最新编译的文件，并且我们将避免遇到错误。**

如果错误仍然存在，那么问题可能是多个 JAR 文件:一个在编译时，另一个在运行时。当应用程序依赖外部 jar 时，这种情况经常发生。这里，我们应该**验证构建路径**中 JAR 的顺序，以识别不一致的 JAR。

如果我们需要进一步调查，用`-verbose: class option` 运行应用程序来检查加载的类是很有帮助的。这可以帮助我们识别过时的类。

有时第三方 JAR 可能在内部引用另一个版本，这导致了`NoSuchFieldError`。如果发生这种情况，我们可以使用 **`mvn dependency:tree -Dverbose.`来生成 maven 依赖树**并帮助我们识别不一致的 JAR。

## 4.结论

在这个简短的教程中，我们已经展示了为什么`NoSuchFieldError`会发生，以及我们如何解决它。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221126230134/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-3)