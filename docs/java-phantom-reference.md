# Java 中的幻影引用

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-phantom-reference>

## 1。概述

在本文中，我们将了解 Java 语言中幻像引用的概念。

## 2。虚拟参考

幻像引用与[软](/web/20221104133502/https://www.baeldung.com/java-soft-references)和[弱](/web/20221104133502/https://www.baeldung.com/java-weak-reference)引用有两个主要区别。

我们不能得到一个虚指的参照物。引用对象永远不能通过 API 直接访问，这就是为什么我们需要一个引用队列来处理这种类型的引用。

垃圾收集器在其 referent 的 finalize 方法被执行之后，将幻影引用添加到引用队列**。这意味着该实例仍在内存中。**

## 3。用例

它们通常用于两种情况。

第一项技术是**确定对象何时从内存**中移除，这有助于调度内存敏感的任务。例如，我们可以在加载另一个对象之前等待一个大对象被移除。

第二种做法是**避免使用`finalize`方法，改进** **定案流程**。

### 3.1。示例

现在，让我们实现第二个用例，实际上弄清楚这种引用是如何工作的。

首先，我们需要一个 [`PhantomReference`](https://web.archive.org/web/20221104133502/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/PhantomReference.html) 类的子类来定义一个清除资源的方法:

```java
public class LargeObjectFinalizer extends PhantomReference<Object> {

    public LargeObjectFinalizer(
      Object referent, ReferenceQueue<? super Object> q) {
        super(referent, q);
    }

    public void finalizeResources() {
        // free resources
        System.out.println("clearing ...");
    }
}
```

现在我们要编写一个增强的细粒度终结:

```java
ReferenceQueue<Object> referenceQueue = new ReferenceQueue<>();
List<LargeObjectFinalizer> references = new ArrayList<>();
List<Object> largeObjects = new ArrayList<>();

for (int i = 0; i < 10; ++i) {
    Object largeObject = new Object();
    largeObjects.add(largeObject);
    references.add(new LargeObjectFinalizer(largeObject, referenceQueue));
}

largeObjects = null;
System.gc();

Reference<?> referenceFromQueue;
for (PhantomReference<Object> reference : references) {
    System.out.println(reference.isEnqueued());
}

while ((referenceFromQueue = referenceQueue.poll()) != null) {
    ((LargeObjectFinalizer)referenceFromQueue).finalizeResources();
    referenceFromQueue.clear();
} 
```

首先，我们初始化所有必要的对象:`referenceQueue`–跟踪排队的引用，`references`–随后执行清理工作，`largeObjects`–模拟大型数据结构。

接下来，我们使用`Object`和`LargeObjectFinalizer`类创建这些对象。

在调用垃圾收集器之前，我们通过解引用`largeObjects`列表来手动释放一大块数据。注意，我们为`Runtime.getRuntime().gc()` 语句使用了一个快捷方式来调用垃圾收集器。

重要的是要知道`System.gc()`并没有立即触发垃圾收集——它只是 JVM 触发进程的一个提示。

`for`循环演示了如何确保所有引用都已入队——它将为每个引用打印出`true`。

最后，我们使用了一个 *while* 循环来轮询出排队的引用，并为每个引用做清理工作。

## 4。结论

在这个快速教程中，我们介绍了 Java 的幻影引用。

在一些简单扼要的例子中，我们了解了这些是什么，以及它们如何有用。