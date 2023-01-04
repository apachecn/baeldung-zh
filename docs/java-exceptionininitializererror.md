# Java 什么时候抛出 ExceptionInInitializerError？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-exceptionininitializererror>

## 1.概观

在这个快速教程中，我们将看到是什么导致 Java 抛出了一个`[ExceptionInInitializerError](https://web.archive.org/web/20220627185046/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ExceptionInInitializerError.html) `异常的实例。

我们将从一点理论开始。然后，我们将在实践中看到这种异常的几个例子。

## 2.`ExceptionInInitializerError`

**`ExceptionInInitializerError `表示[静态初始化器](/web/20220627185046/https://www.baeldung.com/java-static)发生了意外异常。**基本上，当我们看到这个异常时，我们应该知道 Java 未能评估一个静态初始化器块或实例化一个静态变量。

事实上，每当静态初始化器中发生异常时，Java 会自动将该异常包装在`ExceptionInInitializerError`类的实例中。这样，它还维护了对实际异常的引用作为根本原因。

现在我们知道了这个异常背后的基本原理，让我们看看它的实际应用。

## 3.静态初始值设定项块

为了有一个失败的静态块初始化器，我们打算故意把一个整数除以零:

```java
public class StaticBlock {

    private static int state;

    static {
        state = 42 / 0;
    }
}
```

现在，如果我们用类似下面的语句触发类初始化:

```java
new StaticBlock();
```

然后，我们会看到以下异常:

```java
java.lang.ExceptionInInitializerError
    at com.baeldung...(ExceptionInInitializerErrorUnitTest.java:18)
Caused by: java.lang.ArithmeticException: / by zero
    at com.baeldung.StaticBlock.<clinit>(ExceptionInInitializerErrorUnitTest.java:35)
    ... 23 more
```

如前所述，Java 抛出`ExceptionInInitializerError `异常，同时维护对根本原因的引用:

```java
assertThatThrownBy(StaticBlock::new)
  .isInstanceOf(ExceptionInInitializerError.class)
  .hasCauseInstanceOf(ArithmeticException.class);
```

另外值得一提的是，`<clinit> `方法是 JVM 中的一个[类初始化方法](/web/20220627185046/https://www.baeldung.com/jvm-init-clinit-methods#clinit)。

## 4.静态变量初始化

如果 Java 无法初始化静态变量，也会发生同样的情况:

```java
public class StaticVar {

    private static int state = initializeState();

    private static int initializeState() {
        throw new RuntimeException();
    }
}
```

同样，如果我们触发类初始化过程:

```java
new StaticVar();
```

然后出现相同的异常:

```java
java.lang.ExceptionInInitializerError
    at com.baeldung...(ExceptionInInitializerErrorUnitTest.java:11)
Caused by: java.lang.RuntimeException
    at com.baeldung.StaticVar.initializeState(ExceptionInInitializerErrorUnitTest.java:26)
    at com.baeldung.StaticVar.<clinit>(ExceptionInInitializerErrorUnitTest.java:23)
    ... 23 more
```

类似于静态初始化程序块，异常的根本原因也被保留:

```java
assertThatThrownBy(StaticVar::new)
  .isInstanceOf(ExceptionInInitializerError.class)
  .hasCauseInstanceOf(RuntimeException.class);
```

## 5.检查异常

作为 Java 语言规范(JLS-11.2.3) 的一部分，我们不能在静态初始化器块或静态变量初始化器中抛出[检查异常](/web/20220627185046/https://www.baeldung.com/java-exceptions#1checked-exceptions)。例如，如果我们试图这样做:

```java
public class NoChecked {
    static {
        throw new Exception();
    }
}
```

编译器将失败，并出现以下编译错误:

```java
java: initializer must be able to complete normally
```

**按照惯例，当我们的静态初始化逻辑抛出一个检查过的异常:**时，我们应该将可能的检查过的异常封装在`ExceptionInInitializerError `的一个实例中

```java
public class CheckedConvention {

    private static Constructor<?> constructor;

    static {
        try {
            constructor = CheckedConvention.class.getDeclaredConstructor();
        } catch (NoSuchMethodException e) {
            throw new ExceptionInInitializerError(e);
        }
    }
}
```

如上所示，`getDeclaredConstructor() `方法抛出一个检查过的异常。因此，我们捕获了被检查的异常，并按照惯例对其进行了包装。

**因为我们已经显式返回了一个`ExceptionInInitializerError`异常的实例，Java 不会将这个异常包装在另一个`ExceptionInInitializerError `实例中。**

然而，如果我们抛出任何其他未检查的异常，Java 将抛出另一个`ExceptionInInitializerError`:

```java
static {
    try {
        constructor = CheckedConvention.class.getConstructor();
    } catch (NoSuchMethodException e) {
        throw new RuntimeException(e);
    }
}
```

这里，我们将选中的异常包装在未选中的异常中。因为这个未检查的异常不是`ExceptionInInitializerError, `的实例，Java 将再次包装它，导致这个意外的堆栈跟踪:

```java
java.lang.ExceptionInInitializerError
	at com.baeldung.exceptionininitializererror...
Caused by: java.lang.RuntimeException: java.lang.NoSuchMethodException: ...
Caused by: java.lang.NoSuchMethodException: com.baeldung.CheckedConvention.<init>()
	at java.base/java.lang.Class.getConstructor0(Class.java:3427)
	at java.base/java.lang.Class.getConstructor(Class.java:2165)
```

如上所示，如果我们遵循约定，那么堆栈跟踪会比这干净得多。

### 5.1.OpenJDK

最近，这种约定甚至被用于 OpenJDK 源代码本身。例如，下面是`[AtomicReference](https://web.archive.org/web/20220627185046/https://github.com/openjdk/jdk/blob/b87302ca99ff30a03e311ab1c0f524684ed37596/src/java.base/share/classes/java/util/concurrent/atomic/AtomicReference.java#L51) `如何使用这种方法:

```java
public class AtomicReference<V> implements java.io.Serializable {
    private static final VarHandle VALUE;
    static {
        try {
            MethodHandles.Lookup l = MethodHandles.lookup();
            VALUE = l.findVarHandle(AtomicReference.class, "value", Object.class);
        } catch (ReflectiveOperationException e) {
            throw new ExceptionInInitializerError(e);
        }
    }

    private volatile V value;

   // omitted
}
```

## 6.结论

在本教程中，我们看到了是什么导致 Java 抛出了一个`ExceptionInInitializerError `异常的实例。

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220627185046/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-2)