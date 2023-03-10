# 获取所有正在运行的 JVM 线程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-get-all-threads>

## 1.概观

在这个简短的教程中，我们将学习如何**获取当前 JVM** 中所有正在运行的线程，包括不是由我们的类启动的线程。

## 2.使用`Thread`类

`Thread`类的`getAllStackTrace()`方法给出了所有正在运行的线程的堆栈跟踪。它返回一个`Map`，它的键是`Thread`对象，所以我们可以获得键集，并简单地循环它的元素来获得关于线程的信息。

让我们使用 [`printf()`](/web/20220529032210/https://www.baeldung.com/java-printstream-printf) 方法使输出更具可读性:

```java
Set<Thread> threads = Thread.getAllStackTraces().keySet();
System.out.printf("%-15s \t %-15s \t %-15s \t %s\n", "Name", "State", "Priority", "isDaemon");
for (Thread t : threads) {
    System.out.printf("%-15s \t %-15s \t %-15d \t %s\n", t.getName(), t.getState(), t.getPriority(), t.isDaemon());
}
```

输出将如下所示:

```java
Name            	 State           	 Priority        	 isDaemon
main            	 RUNNABLE        	 5               	 false
Signal Dispatcher 	 RUNNABLE        	 9               	 true
Finalizer       	 WAITING         	 8               	 true
Reference Handler 	 WAITING         	 10              	 true
```

正如我们所见，除了运行主程序的线程`main`，我们还有另外三个线程。这个结果可能因 Java 版本的不同而不同。

让我们进一步了解这些其他线程:

*   这个线程处理操作系统发送给 JVM 的信号。
*   `Finalizer`:这个线程为不再需要释放系统资源的对象执行终结。
*   `Reference Handler`:该线程将不再需要的对象放入队列，由`Finalizer`线程处理。

如果主程序退出，所有这些线程都将被终止。

## 3.使用 Apache Commons 中的`ThreadUtils`类

我们也可以使用来自 [Apache Commons Lang](https://web.archive.org/web/20220529032210/https://search.maven.org/search?q=g:org.apache.commons%20a:commons-lang3) 库的`ThreadUtils`类来实现相同的目标:

让我们给我们的`pom.xml`文件添加一个依赖项:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.10</version>
</dependency>
```

只需使用`getAllThreads()`方法来获取所有正在运行的线程:

```java
System.out.printf("%-15s \t %-15s \t %-15s \t %s\n", "Name", "State", "Priority", "isDaemon");
for (Thread t : ThreadUtils.getAllThreads()) {
    System.out.printf("%-15s \t %-15s \t %-15d \t %s\n", t.getName(), t.getState(), t.getPriority(), t.isDaemon());
}
```

输出和上面一样。

## 4.结论

总之，我们已经学习了两种方法来**获取当前 JVM** 中所有正在运行的线程。