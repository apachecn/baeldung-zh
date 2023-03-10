# 在 Java 中对散列表进行排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hashmap-sort>

## 1。简介

在这个快速教程中，我们将学习如何在 Java 中**排序`HashMap`。**

更具体地说，我们将通过关键字或值来查看对`HashMap`条目的排序，使用:

*   `TreeMap`
*   `ArrayList`和`Collections.sort()`
*   `TreeSet`
*   使用`Stream` API
*   使用`Guava `库

## 2。使用`TreeMap`

我们知道，`TreeMap`中的**键是按照它们的自然顺序**来排序的。当我们想按键对键值对进行排序时，这是一个很好的解决方案。因此，我们的想法是将所有数据从我们的`HashMap`推送到 [`TreeMap`](/web/20221201083836/https://www.baeldung.com/java-treemap) 。

首先，让我们定义一个`HashMap`并用一些数据初始化它:

```java
Map<String, Employee> map = new HashMap<>();

Employee employee1 = new Employee(1L, "Mher");
map.put(employee1.getName(), employee1);
Employee employee2 = new Employee(22L, "Annie");
map.put(employee2.getName(), employee2);
Employee employee3 = new Employee(8L, "John");
map.put(employee3.getName(), employee3);
Employee employee4 = new Employee(2L, "George");
map.put(employee4.getName(), employee4);
```

对于`Employee`类，**注意，我们实现了`Comparable` :**

```java
public class Employee implements Comparable<Employee> {

    private Long id;
    private String name;

    // constructor, getters, setters

    // override equals and hashCode
    @Override
    public int compareTo(Employee employee) {
        return (int)(this.id - employee.getId());
    }
}
```

接下来，我们通过使用其构造函数将条目存储在`TreeMap `中:

```java
TreeMap<String, Employee> sorted = new TreeMap<>(map);
```

我们也可以使用`putAll`方法来复制数据:

```java
TreeMap<String, Employee> sorted = new TreeMap<>();
sorted.putAll(map);
```

就是这样！为了确保我们的地图条目按关键字排序，让我们将它们打印出来:

```java
Annie=Employee{id=22, name='Annie'}
George=Employee{id=2, name='George'}
John=Employee{id=8, name='John'}
Mher=Employee{id=1, name='Mher'}
```

正如我们所看到的，这些键是按照自然顺序排序的。

## 3。使用 **`ArrayList`**

当然，我们可以借助`ArrayList`对地图的条目进行排序。与前一种方法的关键区别在于，**我们没有在这里**维护`Map`接口。

### 3.1。按键排序

让我们将密钥集加载到一个`ArrayList`中:

```java
List<String> employeeByKey = new ArrayList<>(map.keySet());
Collections.sort(employeeByKey);
```

输出是:

```java
[Annie, George, John, Mher]
```

### 3.2。按值排序

现在，如果我们想通过`Employee`对象的`id`字段对我们的地图值进行排序呢？我们也可以为此使用`ArrayList`。

首先，让我们将值复制到列表中:

```java
List<Employee> employeeById = new ArrayList<>(map.values());
```

然后我们对其进行分类:

```java
Collections.sort(employeeById);
```

记住这是因为 **`Employee`实现了`Comparable`接口**。否则，我们需要为对`Collections.sort`的调用定义一个手动比较器。

为了检查结果，我们打印了`employeeById`:

```java
[Employee{id=1, name='Mher'}, 
Employee{id=2, name='George'}, 
Employee{id=8, name='John'}, 
Employee{id=22, name='Annie'}]
```

正如我们看到的，对象是按照它们的`id`字段排序的。

## 4。使用`TreeSet `

如果我们**不想在排序后的集合中接受重复值，那么`TreeSet.`** 有一个很好的解决方案

首先，让我们在初始地图中添加一些重复的条目:

```java
Employee employee5 = new Employee(1L, "Mher");
map.put(employee5.getName(), employee5);
Employee employee6 = new Employee(22L, "Annie");
map.put(employee6.getName(), employee6);
```

### 4.1。按键排序

要按关键字条目对地图进行排序，请执行以下操作:

```java
SortedSet<String> keySet = new TreeSet<>(map.keySet());
```

让我们打印`keySet`并查看输出:

```java
[Annie, George, John, Mher]
```

现在，我们已经对没有重复的映射键进行了排序。

### 4.2。按值排序

同样，对于映射值，转换代码如下所示:

```java
SortedSet<Employee> values = new TreeSet<>(map.values());
```

结果是:

```java
[Employee{id=1, name='Mher'}, 
Employee{id=2, name='George'}, 
Employee{id=8, name='John'}, 
Employee{id=22, name='Annie'}]
```

正如我们所看到的，输出中没有重复项。**当我们覆盖`equals`和`hashCode.`** 时，这适用于自定义对象

## 5。使用 Lambdas 和流

**从 Java 8 开始，我们可以使用 Stream API 和 lambda 表达式对 map 进行排序**。我们所需要的就是通过地图的`stream `管道调用`sorted`方法。

### 5.1。按键排序

为了按键排序，我们使用了`comparingByKey `比较器:

```java
map.entrySet()
  .stream()
  .sorted(Map.Entry.<String, Employee>comparingByKey())
  .forEach(System.out::println);
```

最后的`forEach`阶段打印出结果:

```java
Annie=Employee{id=22, name='Annie'}
George=Employee{id=2, name='George'}
John=Employee{id=8, name='John'}
Mher=Employee{id=1, name='Mher'}
```

默认情况下，排序模式是升序。

### 5.2。按值排序

当然，我们也可以按`Employee`对象排序:

```java
map.entrySet()
  .stream()
  .sorted(Map.Entry.comparingByValue())
  .forEach(System.out::println);
```

正如我们所看到的，上面的代码打印出了一个按照`Employee`对象的`id`字段排序的地图:

```java
Mher=Employee{id=1, name='Mher'}
George=Employee{id=2, name='George'}
John=Employee{id=8, name='John'}
Annie=Employee{id=22, name='Annie'}
```

此外，我们可以将结果收集到新的地图中:

```java
Map<String, Employee> result = map.entrySet()
  .stream()
  .sorted(Map.Entry.comparingByValue())
  .collect(Collectors.toMap(
    Map.Entry::getKey, 
    Map.Entry::getValue, 
    (oldValue, newValue) -> oldValue, LinkedHashMap::new));
```

**请注意，我们将结果收集到了一个`LinkedHashMap`中。**默认情况下，`Collectors.toMap`返回一个新的 HashMap，但是我们知道， **`HashMap`不保证迭代** **顺序** `,`，而`LinkedHashMap`保证。

## 6。使用番石榴

最后，一个允许我们对`HashMap`进行排序的库是 Guava。在我们开始之前，检查一下我们写的关于瓜哇的[地图会很有用。](/web/20221201083836/https://www.baeldung.com/guava-maps)

首先，让我们声明一个 [`Ordering`](/web/20221201083836/https://www.baeldung.com/guava-ordering) ，因为我们想通过`Employee's` `Id`字段对我们的地图进行排序:

```java
Ordering naturalOrdering = Ordering.natural()
  .onResultOf(Functions.forMap(map, null));
```

现在我们只需要用`ImmutableSortedMap `来说明结果:

```java
ImmutableSortedMap.copyOf(map, naturalOrdering);
```

同样，输出是按`id`字段排序的地图:

```java
Mher=Employee{id=1, name='Mher'}
George=Employee{id=2, name='George'}
John=Employee{id=8, name='John'}
Annie=Employee{id=22, name='Annie'}
```

## 7。总结

在本文中，我们回顾了通过键或值对`HashMap `进行排序的许多方法。

我们还学习了当属性是一个定制类时，如何通过实现`Comparable `来实现这一点。

最后，和往常一样，本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221201083836/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-2)