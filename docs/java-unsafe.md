# sun.misc.Unsafe 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-unsafe>

## 1。概述

在本文中，我们将看看 JRE 提供的一个有趣的类—`sun.misc` 包中的`Unsafe`。这个类为我们提供了一些低级机制，这些机制被设计成只能由核心 Java 库使用，而不能由标准用户使用。

这为我们提供了主要为核心库内部使用而设计的底层机制。

## 2。`Unsafe`获取的实例

首先，为了能够使用`Unsafe` 类，我们需要获得一个实例——这并不简单，因为该类只是为内部使用而设计的。

**获取实例的方法是通过静态方法`getUnsafe().`** 注意，默认情况下——这将抛出一个 `SecurityException.`

幸运的是，**我们可以使用反射获得实例:**

```java
Field f = Unsafe.class.getDeclaredField("theUnsafe");
f.setAccessible(true);
unsafe = (Unsafe) f.get(null);
```

## 3。使用`Unsafe` 实例化一个类

假设我们有一个简单的类，它的构造函数在对象创建时设置一个变量值:

```java
class InitializationOrdering {
    private long a;

    public InitializationOrdering() {
        this.a = 1;
    }

    public long getA() {
        return this.a;
    }
}
```

当我们使用构造函数初始化该对象时，`getA()` 方法将返回值 1:

```java
InitializationOrdering o1 = new InitializationOrdering();
assertEquals(o1.getA(), 1);
```

但是我们可以使用`Unsafe.`来使用`allocateInstance()` 方法，它只会为我们的类分配内存，而不会调用构造函数:

```java
InitializationOrdering o3 
  = (InitializationOrdering) unsafe.allocateInstance(InitializationOrdering.class);

assertEquals(o3.getA(), 0);
```

注意，构造函数没有被调用，因此，`getA()` 方法返回了`long`类型的默认值——0。

## 4。改变私有字段

假设我们有一个保存私有值的类:

```java
class SecretHolder {
    private int SECRET_VALUE = 0;

    public boolean secretIsDisclosed() {
        return SECRET_VALUE == 1;
    }
}
```

使用来自`Unsafe,`的`putInt()` 方法，我们可以改变私有`SECRET_VALUE` 字段的值，改变/破坏该实例的状态:

```java
SecretHolder secretHolder = new SecretHolder();

Field f = secretHolder.getClass().getDeclaredField("SECRET_VALUE");
unsafe.putInt(secretHolder, unsafe.objectFieldOffset(f), 1);

assertTrue(secretHolder.secretIsDisclosed());
```

一旦我们通过反射调用获得了一个字段，我们就可以使用`Unsafe`将它的值更改为任何其他的`int`值。

## 5。抛出异常

编译器不会像检查普通 Java 代码一样检查通过`Unsafe` 调用的代码。我们可以使用`throwException()` 方法抛出任何异常，而不限制调用者处理该异常，即使它是一个检查过的异常:

```java
@Test(expected = IOException.class)
public void givenUnsafeThrowException_whenThrowCheckedException_thenNotNeedToCatchIt() {
    unsafe.throwException(new IOException());
}
```

在抛出一个被检查的`IOException,` 后，我们不需要捕捉它，也不需要在方法声明中指定它。

## 6。堆外内存

如果应用程序耗尽了 JVM 上的可用内存，我们可能会迫使 GC 进程过于频繁地运行。理想情况下，我们需要一个特殊的内存区域，在堆外，不受 GC 进程的控制。

来自`Unsafe`类的`allocateMemory()` 方法给了我们从堆中分配巨大对象的能力，这意味着**这个内存将不会被 GC 和 JVM** 看到和考虑。

这可能非常有用，但我们需要记住，这些内存需要手动管理，并在不再需要时使用`freeMemory()`正确回收。

假设我们想要创建大型堆外内存字节数组。我们可以使用`allocateMemory()`方法来实现:

```java
class OffHeapArray {
    private final static int BYTE = 1;
    private long size;
    private long address;

    public OffHeapArray(long size) throws NoSuchFieldException, IllegalAccessException {
        this.size = size;
        address = getUnsafe().allocateMemory(size * BYTE);
    }

    private Unsafe getUnsafe() throws IllegalAccessException, NoSuchFieldException {
        Field f = Unsafe.class.getDeclaredField("theUnsafe");
        f.setAccessible(true);
        return (Unsafe) f.get(null);
    }

    public void set(long i, byte value) throws NoSuchFieldException, IllegalAccessException {
        getUnsafe().putByte(address + i * BYTE, value);
    }

    public int get(long idx) throws NoSuchFieldException, IllegalAccessException {
        return getUnsafe().getByte(address + idx * BYTE);
    }

    public long size() {
        return size;
    }

    public void freeMemory() throws NoSuchFieldException, IllegalAccessException {
        getUnsafe().freeMemory(address);
    }
```

```java
}
```

在`OffHeapArray,` 的构造函数中，我们正在初始化给定`size.` 的数组，我们正在将数组的起始地址存储在`address`字段中。`set()`方法获取索引和给定的`value`，它们将存储在数组中。`get()` 方法使用其索引来检索字节值，该索引是从数组起始地址的偏移量。

接下来，我们可以使用其构造函数来分配堆外数组:

```java
long SUPER_SIZE = (long) Integer.MAX_VALUE * 2;
OffHeapArray array = new OffHeapArray(SUPER_SIZE);
```

我们可以将 N 个字节值放入该数组，然后检索这些值，将它们相加以测试我们的寻址是否正确:

```java
int sum = 0;
for (int i = 0; i < 100; i++) {
    array.set((long) Integer.MAX_VALUE + i, (byte) 3);
    sum += array.get((long) Integer.MAX_VALUE + i);
}

assertEquals(array.size(), SUPER_SIZE);
assertEquals(sum, 300);
```

最后，我们需要通过调用`freeMemory().`将内存释放回操作系统

## 7。`CompareAndSwap`操作

来自`java.concurrent`包的非常高效的构造，像`AtomicInteger,`正在使用下面`Unsafe`的`compareAndSwap()`方法，以提供可能的最佳性能。这种结构广泛用于无锁算法中，与 Java 中的标准悲观同步机制相比，这种算法可以利用 CAS 处理器指令来提供极大的加速。

我们可以使用`Unsafe`中的`compareAndSwapLong()` 方法构建基于 CAS 的计数器:

```java
class CASCounter {
    private Unsafe unsafe;
    private volatile long counter = 0;
    private long offset;

    private Unsafe getUnsafe() throws IllegalAccessException, NoSuchFieldException {
        Field f = Unsafe.class.getDeclaredField("theUnsafe");
        f.setAccessible(true);
        return (Unsafe) f.get(null);
    }

    public CASCounter() throws Exception {
        unsafe = getUnsafe();
        offset = unsafe.objectFieldOffset(CASCounter.class.getDeclaredField("counter"));
    }

    public void increment() {
        long before = counter;
        while (!unsafe.compareAndSwapLong(this, offset, before, before + 1)) {
            before = counter;
        }
    }

    public long getCounter() {
        return counter;
    }
}
```

在`CASCounter` 构造函数中，我们正在获取计数器字段的地址，以便稍后在`increment()`方法中使用。该字段需要声明为 volatile，以便对所有读写该值的线程可见。我们使用`objectFieldOffset()` 方法来获取`offset`字段的内存地址。

这个类最重要的部分是`increment()` 方法。我们在`while`循环中使用`compareAndSwapLong()` 来增加之前获取的值，检查之前的值在我们获取之后是否发生了变化。

如果是，那么我们将重试该操作，直到成功。这里没有阻塞，这就是为什么它被称为无锁算法。

我们可以通过从多个线程递增共享计数器来测试我们的代码:

```java
int NUM_OF_THREADS = 1_000;
int NUM_OF_INCREMENTS = 10_000;
ExecutorService service = Executors.newFixedThreadPool(NUM_OF_THREADS);
CASCounter casCounter = new CASCounter();

IntStream.rangeClosed(0, NUM_OF_THREADS - 1)
  .forEach(i -> service.submit(() -> IntStream
    .rangeClosed(0, NUM_OF_INCREMENTS - 1)
    .forEach(j -> casCounter.increment())));
```

接下来，为了断言计数器的状态是正确的，我们可以从中获得计数器值:

```java
assertEquals(NUM_OF_INCREMENTS * NUM_OF_THREADS, casCounter.getCounter());
```

## 8。停车/不停车

在`Unsafe` API 中有两个有趣的方法被 JVM 用来上下文切换线程。当线程等待某个动作时，JVM 可以通过使用来自`Unsafe` 类的`park()` 方法来阻塞这个线程。

它非常类似于`Object.wait()` 方法，但是它调用本机 OS 代码，从而利用一些架构细节来获得最佳性能。

**当线程被阻塞，需要再次运行时，JVM 使用`unpark()` 方法。**我们经常会在线程转储中看到这些方法调用，尤其是在使用线程池的应用程序中。

## 9。结论

在本文中，我们正在研究`Unsafe`类及其最有用的构造。

我们看到了如何访问私有字段，如何分配堆外内存，以及如何使用比较和交换结构来实现无锁算法。

所有这些例子和代码片段的实现都可以在 GitHub 上找到[——这是一个 Maven 项目，所以它应该很容易导入和运行。](https://web.archive.org/web/20221113134426/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-sun)