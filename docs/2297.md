# 同步的不良实践

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-synchronization-bad-practices>

## 1.概观

[Java 中的同步](/web/20221108113415/https://www.baeldung.com/java-synchronized#why-synchronization)对于摆脱多线程问题很有帮助。然而，当同步的原则没有被深思熟虑地使用时，它们会给我们带来很多麻烦。

在本教程中，我们将讨论一些与同步相关的不良实践，以及每个用例的更好方法。

## 2.同步原理

一般来说，**我们应该只同步那些我们确信没有外部代码会锁定**的对象。

换句话说，**使用池化或可重用的对象进行同步**是一种不好的做法。原因是 JVM 中的其他进程可以访问池化/可重用对象，外部/不可信代码对此类对象的任何修改都会导致死锁和不确定行为。

现在，让我们讨论基于某些类型如`String`、`Boolean`、`Integer`和`Object`的同步原理。

## 3.`String`字面意思

### 3.1.不良做法

[字符串文字被汇集](/web/20221108113415/https://www.baeldung.com/java-string-pool)并经常在 Java 中重用。因此，不建议使用带有 [`synchronized`关键字的`String`类型进行同步](/web/20221108113415/https://www.baeldung.com/java-synchronized#the-synchronized-keyword):

```
public void stringBadPractice1() {
    String stringLock = "LOCK_STRING";
    synchronized (stringLock) {
        // ...
    }
}
```

类似地，如果我们使用`private final String`文字，它仍然是从常量池中引用的:

```
private final String stringLock = "LOCK_STRING";
public void stringBadPractice2() {
    synchronized (stringLock) {
        // ...
    }
}
```

此外，将`intern`与`String`同步也被认为是一种不好的做法:

```
private final String internedStringLock = new String("LOCK_STRING").intern();
public void stringBadPractice3() {
  synchronized (internedStringLock) {
      // ...
  }
}
```

[根据 Javadocs](https://web.archive.org/web/20221108113415/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#intern()),`intern`方法为我们获取了`String`对象的规范表示。换句话说，`intern`方法从池中返回一个`String`——如果它不在池中，就显式地将它添加到池中——它与这个`String`具有相同的内容。

因此，可重用对象上的同步问题对于被拘留的`String`对象也是如此。

注意:**所有的`String`文字和字符串值常量表达式都被自动定义为**。

### 3.2.解决办法

避免对`String`文字同步的坏习惯的建议是**使用`new`关键字**创建一个`String`的新实例。

让我们修复我们已经讨论过的代码中的问题。首先，我们将创建一个新的`String`对象，它有一个惟一的引用(以避免任何重用)和自己的内在锁，这有助于同步。

然后，我们保留对象`private`和`final`以防止任何外部/不可信代码访问它:

```
private final String stringLock = new String("LOCK_STRING");
public void stringSolution() {
    synchronized (stringLock) {
        // ...
    }
}
```

## 4.`Boolean`字面意思

具有两个值`true`和`false`的`Boolean`类型不适用于锁定目的。与 JVM 中的`String`文字相似，`boolean`文字值也共享`Boolean`类的唯一实例。

让我们来看一个在`Boolean`锁对象上同步的糟糕代码示例:

```
private final Boolean booleanLock = Boolean.FALSE;
public void booleanBadPractice() {
    synchronized (booleanLock) {
        // ...
    }
}
```

在这里，如果任何外部代码也对具有相同值的`Boolean`文字进行同步，系统可能会变得无响应或导致死锁情况。

因此，我们不建议使用 `Boolean` 对象作为同步锁。

## 5.装箱原语

### 5.1.不良做法

类似于`boolean`文字，装箱的类型可以为一些值重用实例。原因是 JVM 缓存并共享可以用字节表示的值。

例如，让我们编写一个在装箱类型`Integer`上同步的糟糕代码示例:

```
private int count = 0;
private final Integer intLock = count; 
public void boxedPrimitiveBadPractice() { 
    synchronized (intLock) {
        count++;
        // ... 
    } 
}
```

### 5.2.解决办法

然而，与`boolean`不同，boxed 原语上同步的解决方案是创建一个新的实例。

类似于`String`对象，我们应该使用`new`关键字创建一个`Integer`对象的唯一实例，它有自己的内在锁，并保持它的`private`和`final`:

```
private int count = 0;
private final Integer intLock = new Integer(count);
public void boxedPrimitiveSolution() {
    synchronized (intLock) {
        count++;
        // ...
    }
}
```

## 6.类同步

当一个类用`this`关键字实现方法同步或块同步时，JVM 使用对象本身作为监视器(它的内部锁)。

不受信任的代码可以获取并无限期持有可访问类的固有锁。因此，这可能会导致死锁情况。

### 6.1.不良做法

例如，让我们用`synchronized`方法`setName` 和`setOwner`方法`synchronized`块创建`Animal`类:

```
public class Animal {
    private String name;
    private String owner;

    // getters and constructors

    public synchronized void setName(String name) {
        this.name = name;
    }

    public void setOwner(String owner) {
        synchronized (this) {
            this.owner = owner;
        }
    }
}
```

现在，让我们编写一些糟糕的代码，创建一个`Animal`类的实例并在其上同步:

```
Animal animalObj = new Animal("Tommy", "John");
synchronized (animalObj) {
    while(true) {
        Thread.sleep(Integer.MAX_VALUE);
    }
}
```

这里，不受信任的代码示例引入了一个不确定的延迟，阻止了`setName`和`setOwner`方法实现获得同一个锁。

### 6.2.解决办法

防止这个漏洞的解决方案是私有锁对象。

这个想法是使用与我们的类中定义的`Object`类的 **`private final`实例相关联的固有锁来代替对象**本身的固有锁。

此外，我们应该使用块同步来代替方法同步，以增加灵活性，将非同步代码排除在块之外。

因此，让我们对我们的`Animal`类进行必要的修改:

```
public class Animal {
    // ...

    private final Object objLock1 = new Object();
    private final Object objLock2 = new Object();

    public void setName(String name) {
        synchronized (objLock1) {
            this.name = name;
        }
    }

    public void setOwner(String owner) {
        synchronized (objLock2) {
            this.owner = owner;
        }
    }
}
```

在这里，为了更好的并发性，我们通过定义多个`private final`锁对象来细化锁定方案，以分离我们对两种方法——`setName`和`setOwner`的同步关注。

此外，如果实现`synchronized`块的方法修改了`static`变量，我们必须通过锁定`static`对象来同步:

```
private static int staticCount = 0;
private static final Object staticObjLock = new Object();
public void staticVariableSolution() {
    synchronized (staticObjLock) {
        count++;
        // ...
    }
}
```

## 7.结论

在本文中，我们讨论了与某些类型的同步相关的一些不好的实践，比如`String`、`Boolean`、`Integer`和`Object`。

本文最重要的一点是，不建议使用池化或可重用的对象进行同步。

另外，建议**在`Object`类**的`private final`实例上同步。这样的对象对于外部/不受信任的代码来说是不可访问的，否则这些代码可能会与我们的`public`类进行交互，从而降低这种交互导致死锁的可能性。

和往常一样，源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221108113415/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-advanced-4)