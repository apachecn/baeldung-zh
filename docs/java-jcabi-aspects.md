# jcabi-aspects AOP 注释库简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-jcabi-aspects>

## 1.概观

在这个快速教程中，我们将探索`jcabi-aspects` Java 库，这是一个使用面向方面编程(AOP)修改 Java 应用程序行为的便利注释集合。

`jcabi-aspects`库提供了类似于`@Async`、`@Loggable`和`@RetryOnFailure`的注释，这些注释对于使用 AOP 高效地执行某些操作非常有用。同时，它们有助于减少应用程序中样板代码的数量。该库要求 [AspectJ](/web/20221126224052/https://www.baeldung.com/aspectj) 将方面编织到编译后的类中。

## 2.设置

首先，我们将最新的 [`jcabi-aspects`](https://web.archive.org/web/20221126224052/https://search.maven.org/search?q=g:com.jcabi%20a:jcabi-aspects) Maven 依赖项添加到`pom.xml`:

```java
<dependency>
    <groupId>com.jcabi</groupId>
    <artifactId>jcabi-aspects</artifactId>
    <version>0.22.6</version>
</dependency> 
```

`jcabi-aspects`库需要 AspectJ 运行时支持才能运行。因此，让我们添加 [`aspectjrt`](https://web.archive.org/web/20221126224052/https://search.maven.org/search?q=g:org.aspectj%20a:aspectjrt) 美芬依赖:

```java
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjrt</artifactId>
    <version>1.9.2</version>
    <scope>runtime</scope>
</dependency>
```

接下来，让我们添加 **[`jcabi-maven-plugin`](https://web.archive.org/web/20221126224052/https://search.maven.org/search?q=g:com.jcabi%20a:jcabi-maven-plugin) 插件，它在编译时用 AspectJ 方面编织二进制文件**。插件提供了进行自动编织的`ajc`目标:

```java
<plugin>
    <groupId>com.jcabi</groupId>
    <artifactId>jcabi-maven-plugin</artifactId>
    <version>0.14.1</version>
    <executions>
        <execution>
            <goals>
                <goal>ajc</goal>
            </goals>
        </execution>
    </executions>
    <dependencies>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjtools</artifactId>
            <version>1.9.2</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.2</version>
        </dependency>
    </dependencies>
</plugin>
```

最后，让我们使用 Maven 命令编译这些类:

```java
mvn clean package
```

由`jcabi-maven-plugin`在编译时生成的日志看起来像这样:

```java
[INFO] --- jcabi-maven-plugin:0.14.1:ajc (default) @ jcabi ---
[INFO] jcabi-aspects 0.18/55a5c13 started new daemon thread jcabi-loggable for watching of 
  @Loggable annotated methods
[INFO] Unwoven classes will be copied to /jcabi/target/unwoven
[INFO] Created temp dir /jcabi/target/jcabi-ajc
[INFO] jcabi-aspects 0.18/55a5c13 started new daemon thread jcabi-cacheable for automated
  cleaning of expired @Cacheable values
[INFO] ajc result: 11 file(s) processed, 0 pointcut(s) woven, 0 error(s), 0 warning(s)
```

现在我们知道了如何将库添加到我们的项目中，让我们看看它的注释是如何工作的。

## 3.`@Async`

[`@Async`](https://web.archive.org/web/20221126224052/https://aspects.jcabi.com/apidocs-0.22.6/com/jcabi/aspects/Async.html) 注释允许异步执行方法。但是，它只与返回`void`或 [`Future`](/web/20221126224052/https://www.baeldung.com/java-future) 类型的方法兼容。

让我们编写一个`displayFactorial`方法来异步显示一个数字的阶乘:

```java
@Async
public static void displayFactorial(int number) {
    long result = factorial(number);
    System.out.println(result);
} 
```

然后，我们将重新编译该类，让 Maven 为`@Async`注释编织方面。最后，我们可以运行我们的示例:

```java
[main] INFO com.jcabi.aspects.aj.NamedThreads - 
jcabi-aspects 0.22.6/3f0a1f7 started new daemon thread jcabi-async for Asynchronous method execution
```

从日志中我们可以看到，**库创建了一个单独的守护线程`jcabi-async` 来执行所有的异步操作**。

现在，让我们使用`@Async`注释来返回一个`Future`实例:

```java
@Async
public static Future<Long> getFactorial(int number) {
    Future<Long> factorialFuture = CompletableFuture.supplyAsync(() -> factorial(number));
    return factorialFuture;
}
```

如果我们在一个不返回`void`或`Future`的方法上使用`@Async`，当我们调用它时，将在运行时抛出一个异常。

## 4.`@Cacheable`

[`@Cacheable`](https://web.archive.org/web/20221126224052/https://aspects.jcabi.com/apidocs-0.22.6/com/jcabi/aspects/Cacheable.html) 注释允许缓存方法的结果以避免重复计算。

例如，让我们编写一个返回最新汇率的`cacheExchangeRates`方法:

```java
@Cacheable(lifetime = 2, unit = TimeUnit.SECONDS)
public static String cacheExchangeRates() {
    String result = null;
    try {
        URL exchangeRateUrl = new URL("https://api.exchangeratesapi.io/latest");
        URLConnection con = exchangeRateUrl.openConnection();
        BufferedReader in = new BufferedReader(new InputStreamReader(con.getInputStream()));
        result = in.readLine();
    } catch (IOException e) {
        e.printStackTrace();
    }
    return result;
}
```

在这里，缓存结果的生存期为 2 秒。类似地，我们可以使用以下方法使结果永远可缓存:

```java
@Cacheable(forever = true)
```

一旦我们重新编译并再次执行该类，该库将记录处理缓存机制的两个守护线程的详细信息:

```java
[main] INFO com.jcabi.aspects.aj.NamedThreads - 
jcabi-aspects 0.22.6/3f0a1f7 started new daemon thread jcabi-cacheable-clean for automated 
  cleaning of expired @Cacheable values
[main] INFO com.jcabi.aspects.aj.NamedThreads - 
jcabi-aspects 0.22.6/3f0a1f7 started new daemon thread jcabi-cacheable-update for async 
  update of expired @Cacheable values
```

当我们调用我们的`cacheExchangeRates`方法时，库将缓存结果并记录执行的细节:

```java
[main] INFO com.baeldung.jcabi.JcabiAspectJ - #cacheExchangeRates(): 
'{"rates":{"CAD":1.458,"HKD":8.5039,"ISK":137.9,"P..364..:4.5425},"base":"EUR","date":"2020-02-10"}'
  cached in 560ms, valid for 2s
```

所以，如果再次调用(2 秒内)，`cacheExchangeRates`将从缓存中返回结果:

```java
[main] INFO com.baeldung.jcabi.JcabiAspectJ - #cacheExchangeRates(): 
'{"rates":{"CAD":1.458,"HKD":8.5039,"ISK":137.9,"P..364..:4.5425},"base":"EUR","date":"2020-02-10"}'
  from cache (hit #1, 563ms old)
```

如果方法抛出异常，结果不会被缓存。

## 5.`@Loggable`

该库为使用 SLF4J 日志记录工具的简单日志记录提供了 [`@Loggable`](https://web.archive.org/web/20221126224052/https://aspects.jcabi.com/apidocs-0.22.6/com/jcabi/aspects/Loggable.html) 注释。

让我们给我们的`displayFactorial`和`cacheExchangeRates`方法添加`@Loggable`注释:

```java
@Loggable
@Async
public static void displayFactorial(int number) {
    ...
}

@Loggable
@Cacheable(lifetime = 2, unit = TimeUnit.SECONDS)
public static String cacheExchangeRates() {
    ...
}
```

然后，在重新编译之后，注释将记录方法名、返回值和执行时间:

```java
[main] INFO com.baeldung.jcabi.JcabiAspectJ - #displayFactorial(): in 1.16ms
[main] INFO com.baeldung.jcabi.JcabiAspectJ - #cacheExchangeRates(): 
'{"rates":{"CAD":1.458,"HKD":8.5039,"ISK":137.9,"P..364..:4.5425},"base":"EUR","date":"2020-02-10"}'
  in 556.92ms
```

## 6.`@LogExceptions`

与`@Loggable`类似，我们可以使用 [`@LogExceptions`](https://web.archive.org/web/20221126224052/https://aspects.jcabi.com/apidocs-0.22.6/com/jcabi/aspects/LogExceptions.html) 注释来只记录方法抛出的异常。

让我们将`@LogExceptions`用在将抛出`ArithmeticException`的方法`divideByZero`上:

```java
@LogExceptions
public static void divideByZero() {
    int x = 1/0;
}
```

方法的执行将记录异常并引发异常:

```java
[main] WARN com.baeldung.jcabi.JcabiAspectJ - java.lang.ArithmeticException: / by zero
    at com.baeldung.jcabi.JcabiAspectJ.divideByZero_aroundBody12(JcabiAspectJ.java:77)

java.lang.ArithmeticException: / by zero
    at com.baeldung.jcabi.JcabiAspectJ.divideByZero_aroundBody12(JcabiAspectJ.java:77)
    ...
```

## 7.`@Quietly`

[`@Quietly`](https://web.archive.org/web/20221126224052/https://aspects.jcabi.com/apidocs-0.22.6/com/jcabi/aspects/Quietly.html) 注释类似于`@LogExceptions`，除了它**不传播任何由方法**抛出的异常。相反，它只是记录它们。

让我们将`@Quietly`注释添加到`divideByZero`方法中:

```java
@Quietly
public static void divideByZero() {
    int x = 1/0;
}
```

因此，注释将吞下异常，并且只记录异常的详细信息，否则将会被抛出:

```java
[main] WARN com.baeldung.jcabi.JcabiAspectJ - java.lang.ArithmeticException: / by zero
    at com.baeldung.jcabi.JcabiAspectJ.divideByZero_aroundBody12(JcabiAspectJ.java:77)
```

`@Quietly`注释只与返回类型为`void`的方法兼容。

## 8.`@RetryOnFailure`

[`@RetryOnFailure`](https://web.archive.org/web/20221126224052/https://aspects.jcabi.com/apidocs-0.22.6/com/jcabi/aspects/RetryOnFailure.html) 注释允许我们在出现异常或失败时重复执行一个方法。

例如，让我们将`@RetryOnFailure`注释添加到我们的`divideByZero`方法中:

```java
@RetryOnFailure(attempts = 2)
@Quietly
public static void divideByZero() {
    int x = 1/0;
}
```

因此，如果方法抛出异常，AOP 通知将尝试执行两次:

```java
[main] WARN com.baeldung.jcabi.JcabiAspectJ - 
#divideByZero(): attempt #1 of 2 failed in 147µs with java.lang.ArithmeticException: / by zero
[main] WARN com.baeldung.jcabi.JcabiAspectJ - 
#divideByZero(): attempt #2 of 2 failed in 110µs with java.lang.ArithmeticException: / by zero
```

此外，我们可以定义其他参数，如`delay`、`unit`和`types`，同时声明`@RetryOnFailure`注释:

```java
@RetryOnFailure(attempts = 3, delay = 5, unit = TimeUnit.SECONDS, 
  types = {java.lang.NumberFormatException.class})
```

在这种情况下，只有当方法抛出一个`NumberFormatException`时，AOP 通知才会尝试该方法三次，每次尝试之间有 5 秒的延迟。

## 9.`@UnitedThrow`

[`@UnitedThrow`](https://web.archive.org/web/20221126224052/https://aspects.jcabi.com/apidocs-0.22.6/com/jcabi/aspects/UnitedThrow.html) 注释允许我们捕捉方法抛出的所有异常，并将其包装在我们指定的异常中。因此，它统一了方法引发的异常。

例如，让我们创建一个抛出`IOException`和`InterruptedException`的方法`processFile`:

```java
@UnitedThrow(IllegalStateException.class)
public static void processFile() throws IOException, InterruptedException {
    BufferedReader reader = new BufferedReader(new FileReader("baeldung.txt"));
    reader.readLine();
    // additional file processing
}
```

在这里，我们添加了注释，将所有异常包装到`IllegalStateException`中。因此，当调用该方法时，异常的堆栈跟踪将如下所示:

```java
java.lang.IllegalStateException: java.io.FileNotFoundException: baeldung.txt (No such file or directory)
    at com.baeldung.jcabi.JcabiAspectJ.processFile(JcabiAspectJ.java:92)
    at com.baeldung.jcabi.JcabiAspectJ.main(JcabiAspectJ.java:39)
Caused by: java.io.FileNotFoundException: baeldung.txt (No such file or directory)
    at java.io.FileInputStream.open0(Native Method)
    ...
```

## 10.结论

在本文中，我们探索了`jcabi-aspects` Java 库。

首先，我们看到了一种使用`jcabi-maven-plugin`在 Maven 项目中建立库的快速方法。

然后，我们研究了一些方便的注释，如`@Async`、`@Loggable`和`@RetryOnFailure`，它们使用 AOP 修改了 Java 应用程序的行为。

像往常一样，所有的代码实现都可以在 GitHub 上获得[。](https://web.archive.org/web/20221126224052/https://github.com/eugenp/tutorials/tree/master/libraries-3)