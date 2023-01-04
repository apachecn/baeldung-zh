# 什么是线程安全，如何实现它？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-thread-safety>

## 1。概述

Java 支持现成的多线程。这意味着通过在独立的工作线程中并发运行字节码， [JVM](/web/20220929100635/https://www.baeldung.com/jvm-vs-jre-vs-jdk) 能够提高应用程序的性能。

虽然多线程是一个强大的特性，但它是有代价的。在多线程环境中，我们需要以线程安全的方式编写实现。这意味着不同的线程可以访问相同的资源，而不会暴露错误的行为或产生不可预知的结果。这种编程方法被称为“线程安全”

在本教程中，我们将看看实现它的不同方法。

## 2。无状态实现

在大多数情况下，多线程应用程序中的错误是几个线程之间不正确地共享状态的结果。

所以，我们要看的第一种方法是使用无状态实现来实现线程安全**。**

为了更好地理解这种方法，让我们考虑一个简单的实用程序类，它有一个计算数字阶乘的静态方法:

```java
public class MathUtils {

    public static BigInteger factorial(int number) {
        BigInteger f = new BigInteger("1");
        for (int i = 2; i <= number; i++) {
            f = f.multiply(BigInteger.valueOf(i));
        }
        return f;
    }
} 
```

**`factorial()`方法是一个无状态的确定性函数。**给定特定的输入，它总是产生相同的输出。

方法**既不依赖于外部状态，也不维护状态。**所以，它被认为是线程安全的，可以被多个线程同时安全调用。

所有线程都可以安全地调用`factorial()`方法，并且将得到预期的结果，而不会互相干扰，也不会改变该方法为其他线程生成的输出。

因此，无状态实现是实现线程安全的最简单方式。

## 3。不可变实现

如果我们需要在不同的线程之间共享状态，我们可以通过使它们不可变来创建线程安全的类。

不变性是一个强大的、与语言无关的概念，在 Java 中很容易实现。

简单来说，**一个类实例，当它的内部状态被构造后不能被修改时，它就是不可变的。**

在 Java 中创建不可变类最简单的方法是声明所有字段`private`和`final`，并且不提供 setters:

```java
public class MessageService {

    private final String message;

    public MessageService(String message) {
        this.message = message;
    }

    // standard getter

}
```

一个`MessageService`对象实际上是不可变的，因为它的状态在构造后不能改变。所以，它是线程安全的。

此外，如果`MessageService`实际上是可变的，但是多个线程只能对它进行只读访问，那么它也是线程安全的。

正如我们所见，**不变性只是实现线程安全的另一种方式。**

## 4。线程本地字段

在面向对象编程(OOP)中，对象实际上需要通过字段维护状态，并通过一个或多个方法实现行为。

如果我们真的需要维护状态，我们可以创建线程安全的类，这些类不会在线程间共享状态，方法是将它们的字段设为线程本地的。

通过简单地在`[Thread](https://web.archive.org/web/20220929100635/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Thread.html)`类中定义私有字段，我们可以很容易地创建其字段是线程本地的类。

例如，我们可以定义一个存储`integers`的`array`的`Thread`类:

```java
public class ThreadA extends Thread {

    private final List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

    @Override
    public void run() {
        numbers.forEach(System.out::println);
    }
}
```

同时，另一个可能持有`strings`的`array`:

```java
public class ThreadB extends Thread {

    private final List<String> letters = Arrays.asList("a", "b", "c", "d", "e", "f");

    @Override
    public void run() {
        letters.forEach(System.out::println);
    }
}
```

在这两个实现中，类都有自己的状态，但不与其他线程共享。所以，这些类是线程安全的。

类似地，我们可以通过将`[ThreadLocal](/web/20220929100635/https://www.baeldung.com/java-threadlocal)`实例分配给一个字段来创建线程本地字段。

让我们考虑下面的`StateHolder`类:

```java
public class StateHolder {

    private final String state;

    // standard constructors / getter
}
```

我们可以很容易地使它成为一个线程局部变量:

```java
public class ThreadState {

    public static final ThreadLocal<StateHolder> statePerThread = new ThreadLocal<StateHolder>() {

        @Override
        protected StateHolder initialValue() {
            return new StateHolder("active");  
        }
    };

    public static StateHolder getState() {
        return statePerThread.get();
    }
}
```

线程本地字段非常像普通的类字段，除了每个通过 setter/getter 访问它们的线程都得到一个独立初始化的字段副本，这样每个线程都有自己的状态。

## 5。同步收藏

通过使用包含在[集合框架](https://web.archive.org/web/20220929100635/https://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html)中的一组同步包装器，我们可以很容易地创建线程安全的集合。

例如，我们可以使用这些[同步包装器](/web/20220929100635/https://www.baeldung.com/java-synchronized-collections)中的一个来创建线程安全集合:

```java
Collection<Integer> syncCollection = Collections.synchronizedCollection(new ArrayList<>());
Thread thread1 = new Thread(() -> syncCollection.addAll(Arrays.asList(1, 2, 3, 4, 5, 6)));
Thread thread2 = new Thread(() -> syncCollection.addAll(Arrays.asList(7, 8, 9, 10, 11, 12)));
thread1.start();
thread2.start(); 
```

让我们记住，同步的集合在每个方法中都使用内部锁定(我们稍后将研究内部锁定)。

这意味着一次只有一个线程可以访问这些方法，而其他线程将被阻塞，直到该方法被第一个线程解锁。

因此，由于同步访问的底层逻辑，同步会有性能损失。

## 6。并发收款

除了同步集合，我们还可以使用并发集合来创建线程安全的集合。

Java 提供了`[java.util.concurrent](https://web.archive.org/web/20220929100635/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/package-summary.html)`包，其中包含几个并发集合，比如`[ConcurrentHashMap](https://web.archive.org/web/20220929100635/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ConcurrentHashMap.html)`:

```java
Map<String,String> concurrentMap = new ConcurrentHashMap<>();
concurrentMap.put("1", "one");
concurrentMap.put("2", "two");
concurrentMap.put("3", "three"); 
```

与它们的同步对应物不同，**并发收集通过将数据分成段来实现线程安全。**比如在一个`ConcurrentHashMap`中，几个线程可以获得不同 map 段上的锁，所以多个线程可以同时访问`Map`。

**由于并发线程访问的固有优势，并发集合的** **比同步集合的**性能高得多。

值得一提的是,**同步和并发集合只使集合本身线程安全，而不是内容。**

## 7.原子物体

还可以使用 Java 提供的一组[原子类](/web/20220929100635/https://www.baeldung.com/java-atomic-variables)来实现线程安全，包括`[AtomicInteger](https://web.archive.org/web/20220929100635/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicInteger.html)`、`[AtomicLong](https://web.archive.org/web/20220929100635/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicLong.html)`、`[AtomicBoolean](https://web.archive.org/web/20220929100635/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicBoolean.html)`和`[AtomicReference](https://web.archive.org/web/20220929100635/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicReference.html)`。

原子类允许我们在不使用同步的情况下执行线程安全的原子操作。原子操作在单个机器级操作中执行。

为了理解这解决的问题，让我们看下面的`Counter`类:

```java
public class Counter {

    private int counter = 0;

    public void incrementCounter() {
        counter += 1;
    }

    public int getCounter() {
        return counter;
    }
}
```

**让我们假设在[竞争条件](/web/20220929100635/https://www.baeldung.com/cs/race-conditions)下，两个线程同时访问`incrementCounter()`方法。**

理论上，`counter`字段的最终值将是 2。但是我们不能确定结果，因为线程同时执行相同的代码块，并且增量不是原子的。

让我们通过使用一个`AtomicInteger`对象来创建一个`Counter`类的线程安全实现:

```java
public class AtomicCounter {

    private final AtomicInteger counter = new AtomicInteger();

    public void incrementCounter() {
        counter.incrementAndGet();
    }

    public int getCounter() {
        return counter.get();
    }
}
```

**这是线程安全的，因为增量++需要不止一次操作，而`incrementAndGet`是原子性的。**

## 8。同步方法

早期的方法对于集合和原语非常好，但是我们有时需要更好的控制。

因此，我们可以用来实现线程安全的另一种常见方法是实现同步方法。

简单地说，**一次只有一个线程可以访问一个同步的方法，同时阻止其他线程访问这个方法。其他线程将保持阻塞状态，直到第一个线程完成或者方法抛出异常。**

我们可以用另一种方式创建一个线程安全版本的`incrementCounter()` ,使它成为一个同步方法:

```java
public synchronized void incrementCounter() {
    counter += 1;
}
```

我们已经创建了一个同步方法，在方法签名前添加了关键字 [`synchronized`](/web/20220929100635/https://www.baeldung.com/java-synchronized) 。

由于一次只有一个线程可以访问一个同步的方法，因此一个线程将执行`incrementCounter()`方法，依次地，其他线程也将执行同样的操作。无论如何都不会发生重叠执行。

同步方法依赖于“内在锁”或“监控锁”的使用内在锁是与特定类实例相关联的隐式内部实体。

在多线程上下文中，术语`monitor`只是指锁在相关对象上执行的角色，因为它强制对一组指定的方法或语句进行独占访问。

当一个线程调用一个同步的方法时，它获得固有锁。线程执行完方法后，释放锁，允许其他线程获取锁并访问方法。

我们可以在实例方法、静态方法和语句(synchronized statements)中实现同步。

## 9。同步语句

有时，如果我们只需要使方法的一部分线程安全，那么同步整个方法可能会矫枉过正。

为了举例说明这个用例，让我们重构一下`incrementCounter()`方法:

```java
public void incrementCounter() {
    // additional unsynced operations
    synchronized(this) {
        counter += 1; 
    }
}
```

这个例子很简单，但是它展示了如何创建一个同步语句。假设该方法现在执行一些不需要同步的额外操作，我们只通过将相关的状态修改部分包装在一个`synchronized`块中来同步它。

与同步方法不同，同步语句必须指定提供内在锁的对象，通常是 [`this`](/web/20220929100635/https://www.baeldung.com/java-this) 引用。

同步是昂贵的，所以使用这个选项，我们只能同步一个方法的相关部分。

### 9.1.其他对象作为锁

我们可以通过利用另一个对象代替`this`作为监控锁来稍微改进`Counter`类的线程安全实现。

这不仅提供了对多线程环境中的共享资源**的协调访问，而且还使用外部实体来强制对资源**的独占访问:

```java
public class ObjectLockCounter {

    private int counter = 0;
    private final Object lock = new Object();

    public void incrementCounter() {
        synchronized(lock) {
            counter += 1;
        }
    }

    // standard getter
}
```

我们用一个普通的 [`Object`](https://web.archive.org/web/20220929100635/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html) 实例来强制互斥。这个实现稍微好一点，因为它提高了锁级别的安全性。

当使用`this `进行内部锁定时，**攻击者可以通过获取内部锁定并触发拒绝服务(DoS)条件来导致[死锁](/web/20220929100635/https://www.baeldung.com/cs/deadlock-livelock-starvation)。**

相反，当使用其他对象时，**那个私有实体是不能从外部访问的。**这使得攻击者更难获得锁并导致死锁。

### 9.2.警告

即使我们可以使用任何 Java 对象作为内部锁，我们也应该避免使用`Strings`进行锁定:

```java
public class Class1 {
    private static final String LOCK  = "Lock";

    // uses the LOCK as the intrinsic lock
}

public class Class2 {
    private static final String LOCK  = "Lock";

    // uses the LOCK as the intrinsic lock
}
```

乍一看，这两个类似乎在使用两个不同的对象作为它们的锁。然而，**因为[字符串实习](/web/20220929100635/https://www.baeldung.com/string/intern)，这两个“锁”值实际上可能指的是[字符串池](/web/20220929100635/https://www.baeldung.com/java-string-pool)上的同一个对象。**即`Class1 `和`Class2 `共享同一个锁！

反过来，这可能会在并发上下文中导致一些意外的行为。

除了`Strings`、**之外，我们应该避免使用任何[可缓存或可重用的](https://web.archive.org/web/20220929100635/https://mail.openjdk.java.net/pipermail/valhalla-spec-observers/2020-February/001199.html)对象作为固有锁。**比如 [`Integer.valueOf() `](https://web.archive.org/web/20220929100635/https://github.com/openjdk/jdk/blob/8c647801fce4d6efcb3780570192973d16e4e6dc/src/java.base/share/classes/java/lang/Integer.java#L1062) 方法缓存小数字。因此，调用`Integer.valueOf(1) `即使在不同的类中也会返回相同的对象。

## 10。易变字段

同步的方法和块对于解决线程间可变的可见性问题很方便。尽管如此，常规类字段的值可能会被 CPU 缓存。因此，对特定字段的后续更新，即使它们是同步的，也可能对其他线程不可见。

为了防止这种情况，我们可以使用 [`volatile`](/web/20220929100635/https://www.baeldung.com/java-volatile) 类字段:

```java
public class Counter {

    private volatile int counter;

    // standard constructors / getter

}
```

**使用`volatile`关键字，我们指示 JVM 和编译器将`counter`变量存储在主内存中。这样，我们确保每次 JVM 读取`counter`变量的值时，它实际上是从主内存中读取的，而不是从 CPU 缓存中读取的。同样，每次 JVM 写入`counter`变量时，该值将被写入主内存。**

此外，**`volatile`变量的使用确保了给定线程可见的所有变量也将从主存储器中读取。**

让我们考虑下面的例子:

```java
public class User {

    private String name;
    private volatile int age;

    // standard constructors / getters

}
```

在这种情况下，每次 JVM 将`age` `volatile`变量写入主内存时，它也会将非易失性`name`变量写入主内存。这确保了两个变量的最新值都存储在主内存中，因此对变量的后续更新将自动对其他线程可见。

类似地，如果一个线程读取一个`volatile`变量的值，那么该线程可见的所有变量也将从主存储器中读取。

**这个由`volatile`变量提供的扩展保证被称为[全可变可见性保证](https://web.archive.org/web/20220929100635/http://tutorials.jenkov.com/java-concurrency/volatile.html)。**

## 11。重入锁

Java 提供了一组改进的`[Lock](/web/20220929100635/https://www.baeldung.com/java-concurrent-locks)`实现，其行为比上面讨论的固有锁稍微复杂一些。

**对于内部锁，锁获取模型相当严格**:一个线程获取锁，然后执行一个方法或代码块，最后释放锁，这样其他线程就可以获取它并访问该方法。

没有底层机制来检查排队的线程，并给等待时间最长的线程优先访问权。

`[ReentrantLock](https://web.archive.org/web/20220929100635/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/locks/ReentrantLock.html)`实例允许我们这样做，**防止排队线程遭受某种类型的[资源匮乏](https://web.archive.org/web/20220929100635/https://en.wikipedia.org/wiki/Starvation_(computer_science))** :

```java
public class ReentrantLockCounter {

    private int counter;
    private final ReentrantLock reLock = new ReentrantLock(true);

    public void incrementCounter() {
        reLock.lock();
        try {
            counter += 1;
        } finally {
            reLock.unlock();
        }
    }

    // standard constructors / getter

}
```

`ReentrantLock`构造函数接受一个可选的`fairness` `boolean`参数。当设置为`true`，并且多个线程试图获取一个锁，**JVM 将优先考虑等待时间最长的线程，并授予对锁的访问权。**

## 12。读/写锁

我们可以用来实现线程安全的另一个强大机制是使用`[ReadWriteLock](https://web.archive.org/web/20220929100635/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/locks/ReadWriteLock.html)`实现。

一个`ReadWriteLock`锁实际上使用了一对关联锁，一个用于只读操作，另一个用于写操作。

因此，只要没有线程写入资源，就有可能有多个线程读取资源。此外，线程写入资源将阻止其他线程读取它。

下面是我们如何使用`ReadWriteLock`锁:

```java
public class ReentrantReadWriteLockCounter {

    private int counter;
    private final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
    private final Lock readLock = rwLock.readLock();
    private final Lock writeLock = rwLock.writeLock();

    public void incrementCounter() {
        writeLock.lock();
        try {
            counter += 1;
        } finally {
            writeLock.unlock();
        }
    }

    public int getCounter() {
        readLock.lock();
        try {
            return counter;
        } finally {
            readLock.unlock();
        }
    }

   // standard constructors

} 
```

## 13。结论

在本文中，**我们学习了什么是 Java 中的线程安全，并深入研究了实现它的不同方法。**

像往常一样，本文中显示的所有代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220929100635/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-basic)