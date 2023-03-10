# Collection.stream()的区别。forEach()和 Collection.forEach()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-collection-stream-foreach>

## 1.概观

在 Java 中有几个选项可以迭代一个集合。**在这个简短的教程中，我们将看看两种相似的方法——`Collection.stream().forEach()`和`Collection.forEach()`。**

在大多数情况下，两者会产生相同的结果，但我们将看看一些微妙的差异。

## 2.一份简单的清单

首先，让我们创建一个要迭代的列表:

```java
List<String> list = Arrays.asList("A", "B", "C", "D");
```

最直接的方法是使用增强的 for 循环:

```java
for(String s : list) {
    //do something with s
} 
```

如果我们想用函数式的 Java，也可以用`forEach()`。

我们可以直接在产品系列上这样做:

```java
Consumer<String> consumer = s -> { System.out::println }; 
list.forEach(consumer); 
```

或者我们可以在集合的流上调用`forEach()`:

```java
list.stream().forEach(consumer); 
```

两个版本都将遍历列表并打印所有元素:

```java
ABCD ABCD
```

在这个简单的例子中，我们使用哪个`forEach()`并没有什么区别。

## 3.执行次序

**`Collection.forEach()`使用集合的迭代器(如果指定了的话)，因此定义了项目的处理顺序。相比之下，`Collection.stream().forEach()`的处理顺序是未定义的。**

在大多数情况下，我们选择两者中的哪一个都没什么区别。

### 3.1.平行流

并行流允许我们在多个线程中执行流，在这种情况下，执行顺序是不确定的。Java 只要求所有线程在任何终端操作(比如`Collectors.toList()`)被调用之前完成。

让我们看一个例子，首先在集合上直接调用`forEach()`，然后在并行流上调用:

```java
list.forEach(System.out::print);
System.out.print(" ");
list.parallelStream().forEach(System.out::print); 
```

**如果我们多次运行代码，我们会看到`list.forEach()`按照插入顺序处理项目，而`list.parallelStream().forEach()`每次运行都会产生不同的结果。**

下面是一个可能的输出:

```java
ABCD CDBA
```

这是另一个:

```java
ABCD DBCA
```

### 3.2.自定义迭代器

让我们用自定义迭代器定义一个列表，以逆序遍历集合:

```java
class ReverseList extends ArrayList<String> {

    @Override
    public Iterator<String> iterator() {

        int startIndex = this.size() - 1;
        List<String> list = this;

        Iterator<String> it = new Iterator<String>() {

            private int currentIndex = startIndex;

            @Override
            public boolean hasNext() {
                return currentIndex >= 0;
            }

            @Override
            public String next() {
                String next = list.get(currentIndex);
                currentIndex--;
                return next;
             }

             @Override
             public void remove() {
                 throw new UnsupportedOperationException();
             }
         };
         return it;
    }
} 
```

然后我们将直接在集合上使用`forEach()`再次迭代列表，然后在流上迭代:

```java
List<String> myList = new ReverseList();
myList.addAll(list);

myList.forEach(System.out::print); 
System.out.print(" "); 
myList.stream().forEach(System.out::print); 
```

我们得到了不同的结果:

```java
DCBA ABCD 
```

产生不同结果的原因是，直接在列表上使用的`forEach()`使用自定义迭代器**，而`stream().forEach()`只是从列表中一个接一个地取出元素，忽略迭代器。**

## 4.集合的修改

许多集合(例如 *ArrayList* 或 *HashSet* )在迭代时不应该被修改结构。如果在迭代过程中删除或添加了一个元素，我们将得到一个`ConcurrentModification`异常。

此外，集合被设计为快速失败，这意味着一旦有修改就会抛出异常。

类似地，当我们在流管道的执行过程中添加或删除元素时，我们会得到一个*并发修改*异常。但是，该异常将在以后引发。

两个`forEach()`方法之间的另一个微妙区别是 Java 明确允许使用迭代器修改元素。相比之下，流应该是无干扰的。

让我们更详细地看看删除和修改元素。

### 4.1.移除元素

让我们定义一个删除列表中最后一个元素(“D”)的操作:

```java
Consumer<String> removeElement = s -> {
    System.out.println(s + " " + list.size());
    if (s != null && s.equals("A")) {
        list.remove("D");
    }
};
```

当我们遍历列表时，在打印第一个元素(“A”)后，最后一个元素被删除:

```java
list.forEach(removeElement);
```

**由于`forEach()`是快速失效的，我们停止迭代，在处理下一个元素之前看到一个异常**:

```java
A 4
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.ArrayList.forEach(ArrayList.java:1252)
	at ReverseList.main(ReverseList.java:1)
```

让我们看看如果使用 *stream()会发生什么。forEach()* 改为:

```java
list.stream().forEach(removeElement);
```

在我们看到一个异常之前，我们继续遍历整个列表。

```java
A 4
B 3
C 3
null 3
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.ArrayList$ArrayListSpliterator.forEachRemaining(ArrayList.java:1380)
	at java.util.stream.ReferencePipeline$Head.forEach(ReferencePipeline.java:580)
	at ReverseList.main(ReverseList.java:1)
```

但是，Java 根本不保证抛出一个 [`ConcurrentModificationException`](https://web.archive.org/web/20220711151814/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/ConcurrentModificationException.html) 。这意味着我们不应该编写依赖于这个异常的程序。

### 4.2.改变元素

我们可以在遍历列表时改变元素:

```java
list.forEach(e -> {
    list.set(3, "E");
});
```

但是尽管使用`Collection.forEach()`或`stream().forEach()`这样做没有问题，Java 要求流上的操作是非干扰性的。这意味着在流管道的执行过程中不应该修改元素。

这背后的原因是流应该便于并行执行。在这里，修改流的元素可能会导致意想不到的行为。

## 5.结论

在本文中，我们看到了一些展示`Collection.forEach()`和`Collection.stream().forEach()`之间细微差别的例子。

需要注意的是，上面显示的所有例子都是琐碎的，只是为了比较遍历集合的两种方式。我们不应该编写其正确性依赖于所显示的行为的代码。

**如果我们不需要一个流，而只想迭代一个集合，那么第一选择应该是直接在集合上使用`forEach()`。**

本文[中示例的源代码可以在 GitHub](https://web.archive.org/web/20220711151814/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-streams-3) 上获得。