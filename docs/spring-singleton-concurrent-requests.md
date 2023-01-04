# Spring Singleton Bean 如何服务于并发请求？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-singleton-concurrent-requests>

## 1.概观

在本教程中，我们将了解使用 [`singleton`](/web/20220524051348/https://www.baeldung.com/spring-bean-scopes#singleton) 范围创建的 Spring beans 如何在后台工作以服务多个并发请求。此外，我们将了解 Java 如何在内存中存储 bean 实例，以及如何处理对它们的并发访问。

## 2.Spring Beans 和 Java 堆内存

众所周知， [Java 堆](/web/20220524051348/https://www.baeldung.com/java-stack-heap#heap-space-in-java)是一个全局共享内存，应用程序中所有正在运行的线程都可以访问它。**当 Spring 容器创建一个具有 singleton 作用域的 bean 时，该 bean 被存储在堆中。**这样，所有并发线程都能够指向同一个 bean 实例。

接下来，让我们了解线程的堆栈内存是什么，以及它如何帮助服务于并发请求。

## 3.如何处理并发请求？

举个例子，让我们看一个 Spring 应用程序，它有一个名为`ProductService`的单例 bean:

```java
@Service
public class ProductService {
    private final static List<Product> productRepository = asList(
      new Product(1, "Product 1", new Stock(100)),
      new Product(2, "Product 2", new Stock(50))
    );

    public Optional<Product> getProductById(int id) {
        Optional<Product> product = productRepository.stream()
          .filter(p -> p.getId() == id)
          .findFirst();
        String productName = product.map(Product::getName)
          .orElse(null);

        System.out.printf("Thread: %s; bean instance: %s; product id: %s has the name: %s%n", currentThread().getName(), this, id, productName);

        return product;
    }
} 
```

这个 bean 有一个方法`getProductById()` ，它将产品数据返回给它的调用者。此外，这个 bean 返回的数据向端点` /product/{id}`上的客户机公开。

接下来，让我们探索当同时调用到达端点 `/product/{id}`时，运行时会发生什么。具体来说，第一个线程将调用端点`/product/1`，第二个线程将调用`/product/2`。

Spring 为每个请求创建一个不同的线程。正如我们在下面的控制台输出中看到的，两个线程使用同一个`ProductService` 实例来返回产品数据:

```java
Thread: pool-2-thread-1; bean instance: [[email protected]](/web/20220524051348/https://www.baeldung.com/cdn-cgi/l/email-protection); product id: 1 has the name: Product 1
Thread: pool-2-thread-2; bean instance: [[email protected]](/web/20220524051348/https://www.baeldung.com/cdn-cgi/l/email-protection); product id: 2 has the name: Product 2
```

Spring 可以在多个线程中使用同一个 bean 实例，首先是因为 Java 为每个线程创建了一个私有的[堆栈内存](/web/20220524051348/https://www.baeldung.com/java-stack-heap#stack-memory-in-java)。

堆栈内存负责存储线程执行期间方法内部使用的局部变量的状态。通过这种方式，Java 确保并行执行的线程不会覆盖彼此的变量。

其次，因为`ProductService` bean 没有在堆级别设置限制或锁定，**每个线程的[程序计数器](/web/20220524051348/https://www.baeldung.com/cs/process-control-block#2-program-counter)能够在堆内存中指向 bean 实例的相同引用。**因此，两个线程可以同时执行`getProdcutById()` 方法。

接下来，我们将理解为什么独立 beans 是无状态的至关重要。

## 4.无状态单例 bean 与有状态单例 bean

为了理解无状态单例 bean 的重要性，让我们看看使用有状态单例 bean 的副作用。

假设我们将`productName`变量移到了类级别:

```java
@Service
public class ProductService {
    private String productName = null;

    // ...

    public Optional getProductById(int id) {
        // ...

        productName = product.map(Product::getName).orElse(null);

       // ...
    }
}
```

现在，让我们再次运行服务并查看输出:

```java
Thread: pool-2-thread-2; bean instance: [[email protected]](/web/20220524051348/https://www.baeldung.com/cdn-cgi/l/email-protection); product id: 2 has the name: Product 2
Thread: pool-2-thread-1; bean instance: [[email protected]](/web/20220524051348/https://www.baeldung.com/cdn-cgi/l/email-protection); product id: 1 has the name: Product 2
```

我们可以看到，对`productId` 1 的调用显示的是`productName`“产品 2”而不是“产品 1”。这是因为`ProductService`是有状态的，并且与所有正在运行的线程共享同一个`productName`变量。

为了避免这种不希望的副作用，保持我们的单例 beans 无状态是至关重要的。

## 5.结论

在本文中，我们看到了对 singleton beans 的并发访问在 Spring 中是如何工作的。首先，我们看了 Java 如何在堆内存中存储单体 beans。接下来，我们学习了不同的线程如何从堆中访问同一个单体实例。最后，我们讨论了为什么拥有无状态 bean 是重要的，并查看了一个如果 bean 不是无状态的会发生什么的例子。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220524051348/https://github.com/eugenp/tutorials/tree/master/spring-core-5)