# CDI 和 EJB·辛格尔顿的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jee-cdi-vs-ejb-singleton>

## 1。概述

在本教程中，我们将仔细研究 Jakarta EE 中可用的两种类型的[singleton](/web/20221206042903/https://www.baeldung.com/java-singleton)。我们将解释和演示不同之处，并查看适合每种用法的用法。

首先，在进入细节之前，让我们看看什么是单身。

## 2。单一设计模式

回想一下，实现[单例模式](/web/20221206042903/https://www.baeldung.com/java-singleton)的一种常见方式是使用静态实例和私有构造函数:

```java
public final class Singleton {
    private static final Singleton instance = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return instance;
    }
} 
```

但是，唉，这不是真正的面向对象。并且它有一些[多线程问题](/web/20221206042903/https://www.baeldung.com/java-singleton#enum)。

然而，CDI 和 EJB 容器给了我们一个面向对象的选择。

## 3。CDI 单例

有了 [CDI(上下文和依赖注入)](/web/20221206042903/https://www.baeldung.com/java-ee-cdi)，我们可以使用`@Singleton`注释轻松创建单例。这个注释是`javax.inject`包的一部分。它指示容器**实例化单例一次**，并在注入过程中将其引用传递给其他对象。

正如我们所见，用 CDI 实现单例非常简单:

```java
@Singleton
public class CarServiceSingleton {
    // ...
} 
```

我们班模拟一个汽车维修店。我们有很多各种`Car`的例子，但是他们都使用同一个维修店。因此，Singleton 是一个很好的选择。

我们可以用一个简单的 JUnit 测试来验证它是同一个实例，这个测试两次询问类的上下文。注意，为了可读性，我们在这里得到了一个 [`getBean`](https://web.archive.org/web/20221206042903/https://github.com/eugenp/tutorials/blob/master/web-modules/jee-7/src/test/java/com/baeldung/singleton/CarServiceIntegrationTest.java#L69) 帮助器方法:

```java
@Test
public void givenASingleton_whenGetBeanIsCalledTwice_thenTheSameInstanceIsReturned() {       
    CarServiceSingleton one = getBean(CarServiceSingleton.class);
    CarServiceSingleton two = getBean(CarServiceSingleton.class);
    assertTrue(one == two);
} 
```

**因为有了`@Singleton`注释，容器两次都将返回相同的引用。**但是，如果我们用一个普通的托管 bean 来尝试，容器每次都会提供不同的实例。

虽然这对于`javax.inject.Singleton`或`javax.ejb.Singleton, ` **来说是一样的，但这两者之间有一个关键的区别。**

## 4。EJB·辛格尔顿

为了创建 EJB 单例，我们使用了来自`javax.ejb`包的`@Singleton`注释。这样我们就创建了一个[单例会话 Bean](/web/20221206042903/https://www.baeldung.com/java-ee-singleton-session-bean) 。

我们可以用与前一个例子中测试 CDI 实现相同的方式来测试这个实现，结果将是相同的。正如所料，EJB 单例提供了该类的单个实例。

然而， **EJB 单例还以容器管理的并发控制的形式提供了额外的功能。**

当我们使用这种类型的实现时，EJB 容器确保该类的每个公共方法每次都被一个线程访问。如果多个线程试图访问同一个方法，那么只有一个线程可以使用它，而其他线程则等待轮到它们。

我们可以通过一个简单的测试来验证这种行为。我们将为我们的单例类引入一个服务队列模拟:

```java
private static int serviceQueue;

public int service(Car car) {
    serviceQueue++;
    Thread.sleep(100);
    car.setServiced(true); 
    serviceQueue--;
    return serviceQueue;
} 
```

`serviceQueue`是一个普通的静态整数，当汽车“进入”服务时增加，当汽车“离开”服务时减少。如果容器提供了适当的锁定，这个变量在服务前后应该等于零，在服务期间应该等于一。

我们可以通过一个简单的测试来检查这种行为:

```java
@Test
public void whenEjb_thenLockingIsProvided() {
    for (int i = 0; i < 10; i++) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                int serviceQueue = carServiceEjbSingleton.service(new Car("Speedster xyz"));
                assertEquals(0, serviceQueue);
            }
        }).start();
    }
    return;
} 
```

该测试启动 10 个并行线程。每个线程实例化一辆汽车，并试图服务它。在服务之后，它断言`serviceQueue`的值回到零。

例如，如果我们在 CDI singleton 上执行类似的测试，我们的测试将会失败。

## 5。结论

在本文中，我们介绍了 Jakarta EE 中可用的两种类型的单体实现。我们看到了它们的优点和缺点，并且演示了如何以及何时使用它们。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221206042903/https://github.com/eugenp/tutorials/tree/master/web-modules/jee-7)