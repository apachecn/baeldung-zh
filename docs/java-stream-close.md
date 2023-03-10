# 我们应该关闭一个 Java 流吗？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stream-close>

## 1.概观

随着 Java 8 中 lambda 表达式的引入，有可能以更简洁和功能性更强的方式编写代码。[流](/web/20221129020054/https://www.baeldung.com/java-streams)和[功能接口](/web/20221129020054/https://www.baeldung.com/java-8-functional-interfaces)是 Java 平台革命性变化的核心。

在这个快速教程中，我们将通过从资源的角度来看 Java 8 流，了解我们是否应该显式地关闭它们。

## 2.关闭流

Java 8 [流](https://web.archive.org/web/20221129020054/https://github.com/openjdk/jdk/blob/6bab0f539fba8fb441697846347597b4a0ade428/src/java.base/share/classes/java/util/stream/BaseStream.java#L64)实现`AutoCloseable`接口:

```java
public interface Stream<T> extends BaseStream<...> {
    // omitted
}
public interface BaseStream<...> extends AutoCloseable {
    // omitted
}
```

简而言之，我们应该把溪流看作是我们可以借用的资源，当我们用完后可以归还。与大多数资源相反，我们不必总是关闭流。

这一开始听起来可能有违直觉，所以让我们看看什么时候应该，什么时候不应该关闭 Java 8 流。

### 2.1.集合、数组和生成器

大多数时候，我们从 Java 集合、数组或生成器函数中创建`Stream`实例。例如，在这里，我们通过流 API 对一组`String`进行操作:

```java
List<String> colors = List.of("Red", "Blue", "Green")
  .stream()
  .filter(c -> c.length() > 4)
  .map(String::toUpperCase)
  .collect(Collectors.toList());
```

有时，我们会生成一个有限或无限的序列流:

```java
Random random = new Random();
random.ints().takeWhile(i -> i < 1000).forEach(System.out::println);
```

此外，我们还可以使用基于阵列的流:

```java
String[] colors = {"Red", "Blue", "Green"};
Arrays.stream(colors).map(String::toUpperCase).toArray()
```

当处理这种类型的流时，我们不应该显式地关闭它们。与这些流相关的唯一有价值的资源是内存，而[垃圾收集](/web/20221129020054/https://www.baeldung.com/jvm-garbage-collectors) (GC)会自动处理这些资源。

### 2.2.IO 资源

然而，有些流是由 IO 资源支持的，比如文件或套接字。例如， [`Files.lines()`](https://web.archive.org/web/20221129020054/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#lines(java.nio.file.Path)) 方法对给定文件的所有行进行流式处理:

```java
Files.lines(Paths.get("/path/to/file"))
  .flatMap(line -> Arrays.stream(line.split(",")))
  // omitted
```

在幕后，这个方法打开一个`[FileChannel](https://web.archive.org/web/20221129020054/https://github.com/openjdk/jdk/blob/6bab0f539fba8fb441697846347597b4a0ade428/src/java.base/share/classes/java/nio/file/Files.java#L4033)`实例，然后在流关闭时关闭它。因此，**如果我们忘记关闭流，底层通道将保持打开，然后我们将以[资源泄漏](https://web.archive.org/web/20221129020054/https://en.wikipedia.org/wiki/Resource_leak)** 结束。

为了防止这种资源泄漏，强烈建议使用`[try-with-resources](/web/20221129020054/https://www.baeldung.com/java-try-with-resources) `习语来关闭基于 IO 的流:

```java
try (Stream<String> lines = Files.lines(Paths.get("/path/to/file"))) {
    lines.flatMap(line -> Arrays.stream(line.split(","))) // omitted
}
```

这样，编译器将自动关闭通道。**这里的关键要点是关闭所有基于 IO 的流**。

请注意，关闭一个已经关闭的流会抛出`IllegalStateException`。

## 3.结论

在这个简短的教程中，我们看到了简单流和 IO 密集型流之间的区别。我们还了解了这些差异是如何影响我们决定是否关闭 Java 8 流的。

像往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20221129020054/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-3)