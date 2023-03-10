# Java For 循环

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-for-loop>

## 1。概述

在本文中，我们将了解 Java 语言的一个核心方面——使用`for`循环重复执行一条或一组语句。

## 2。简单的`for`循环

**`for` 循环是一种控制结构，它允许我们通过递增和评估循环计数器来重复某些操作。**

在第一次迭代之前，循环计数器被初始化，然后执行条件评估，接着是步骤定义(通常是简单的递增)。

`for` 循环的语法是:

```java
for (initialization; Boolean-expression; step) 
  statement;
```

让我们看一个简单的例子:

```java
for (int i = 0; i < 5; i++) {
    System.out.println("Simple for loop: i = " + i);
}
```

**在`for`语句中使用的`initialization`、`Boolean-expression,` 和`step`是可选的。**下面是一个**无限** for 循环的例子:

```java
for ( ; ; ) {
    // Infinite for loop
}
```

### 2.1。标记为`for`的循环

我们也可以标记`for` 循环。如果我们有嵌套的 for 循环，这是很有用的，这样我们就可以从一个特定的`for`循环中`break` / `continue`:

```java
aa: for (int i = 1; i <= 3; i++) {
    if (i == 1)
      continue;
    bb: for (int j = 1; j <= 3; j++) {
        if (i == 2 && j == 2) {
            break aa;
        }
        System.out.println(i + " " + j);
    }
}
```

## 3。增强型`for`循环

从 Java 5 开始，我们有了第二种叫做`enhanced for` *的`for`循环，这种循环*使得迭代数组或集合中的所有元素变得更加容易。

`enhanced for` 循环的语法是:

```java
for(Type item : items)
  statement;
```

因为与标准 for 循环相比，这个循环被简化了，所以在初始化循环时，我们只需要声明两件事:

1.  我们当前正在迭代的元素的句柄
2.  我们正在迭代的源数组/集合

因此，我们可以这样说:**对于`items,`中的每个元素，将该元素赋给`item` 变量并运行循环体**。

让我们来看一个简单的例子:

```java
int[] intArr = { 0,1,2,3,4 }; 
for (int num : intArr) {
    System.out.println("Enhanced for-each loop: i = " + num);
}
```

我们可以用它来迭代各种 Java 数据结构:

给定一个`List<String> list` 对象，我们可以迭代它:

```java
for (String item : list) {
    System.out.println(item);
}
```

我们可以类似地迭代一个`Set<String> set`:

```java
for (String item : set) {
    System.out.println(item);
}
```

并且，给定一个`Map<String,Integer> map` ,我们也可以迭代它:

```java
for (Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(
      "Key: " + entry.getKey() + 
      " - " + 
      "Value: " + entry.getValue());
}
```

### 3.1。`Iterable.forEach()`

从 Java 8 开始，我们可以用稍微不同的方式利用 for-each 循环。我们现在在`Iterable`接口中有了**一个专用的`forEach()`方法，它接受一个 lambda 表达式来表示我们想要执行的动作**。

在内部，它只是将作业委托给标准循环:

```java
default void forEach(Consumer<? super T> action) {
    Objects.requireNonNull(action);
    for (T t : this) {
        action.accept(t);
    }
}
```

让我们来看看这个例子:

```java
List<String> names = new ArrayList<>();
names.add("Larry");
names.add("Steve");
names.add("James");
names.add("Conan");
names.add("Ellen");

names.forEach(name -> System.out.println(name));
```

## 4。结论

在这个快速教程中，我们探索了 Java 的`for`循环。

像往常一样，可以在 GitHub 上找到例子[。](https://web.archive.org/web/20221127213054/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-syntax)