# Java 散列表指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hashmap>

## 1.概观

**在这篇文章中，我们将看到如何在 Java 中使用`HashMap`，并且我们将看看它在内部是如何工作的。**

和`HashMap`非常相似的一个类是`Hashtable`。请参考我们的其他几篇文章来了解更多关于[`java.util.Hashtable`](/web/20221205110945/https://www.baeldung.com/java-hash-table)类本身以及`HashMap`和`Hashtable` 之间的[差异。](/web/20221205110945/https://www.baeldung.com/hashmap-hashtable-differences)

## 2.基本用法

**我们先来看看`HashMap`是一张地图是什么意思。映射是键-值映射，这意味着每个键都映射到一个值，我们可以使用该键从映射中检索相应的值。**

有人可能会问，为什么不直接将值添加到列表中。为什么我们需要一个`HashMap`？原因很简单，就是性能。如果我们想在一个列表中找到一个特定的元素，时间复杂度是`O(n)`,如果列表是排序的，它将是`O(log n)`,例如使用一个二分搜索法。

`HashMap`的优点是插入和检索一个值的时间复杂度平均为`O(1)`。我们将在后面研究如何实现这一点。让我们先来看看如何使用`HashMap`。

### 2.1.设置

让我们创建一个将在整篇文章中使用的简单类:

```java
public class Product {

    private String name;
    private String description;
    private List<String> tags;

    // standard getters/setters/constructors

    public Product addTagsOfOtherProduct(Product product) {
        this.tags.addAll(product.getTags());
        return this;
    }
}
```

### 2.2.放

我们现在可以用类型为`String`的键和类型为`Product`的元素创建一个`HashMap`:

```java
Map<String, Product> productsByName = new HashMap<>(); 
```

并将产品添加到我们的`HashMap`:

```java
Product eBike = new Product("E-Bike", "A bike with a battery");
Product roadBike = new Product("Road bike", "A bike for competition");
productsByName.put(eBike.getName(), eBike);
productsByName.put(roadBike.getName(), roadBike); 
```

### 2.3.得到

我们可以通过它的键从映射中检索一个值:

```java
Product nextPurchase = productsByName.get("E-Bike");
assertEquals("A bike with a battery", nextPurchase.getDescription());
```

如果我们试图为一个 map 中不存在的键寻找一个值，我们将得到一个`null`值:

```java
Product nextPurchase = productsByName.get("Car");
assertNull(nextPurchase);
```

如果我们用同一个键插入第二个值，我们将只得到最后插入的那个键的值:

```java
Product newEBike = new Product("E-Bike", "A bike with a better battery");
productsByName.put(newEBike.getName(), newEBike);
assertEquals("A bike with a better battery", productsByName.get("E-Bike").getDescription());
```

### 2.4.Null 作为键

`HashMap`也允许我们将`null`作为一个键:

```java
Product defaultProduct = new Product("Chocolate", "At least buy chocolate");
productsByName.put(null, defaultProduct);

Product nextPurchase = productsByName.get(null);
assertEquals("At least buy chocolate", nextPurchase.getDescription());
```

### 2.5.具有相同键的值

此外，我们可以用不同的键将同一个对象插入两次:

```java
productsByName.put(defaultProduct.getName(), defaultProduct);
assertSame(productsByName.get(null), productsByName.get("Chocolate"));
```

### 2.6.删除一个值

我们可以从`HashMap`中删除一个键值映射:

```java
productsByName.remove("E-Bike");
assertNull(productsByName.get("E-Bike"));
```

### 2.7.检查映射中是否存在键或值

要检查一个键是否出现在 map 中，我们可以使用`containsKey()`方法:

```java
productsByName.containsKey("E-Bike");
```

或者，为了检查一个值是否出现在地图中，我们可以使用`containsValue()`方法:

```java
productsByName.containsValue(eBike);
```

在我们的例子中，两个方法调用都将返回`true`。尽管它们看起来非常相似，但这两个方法调用在性能上有很大的不同。**检查一个键是否存在的复杂度是`O(1)`，而检查一个元素的复杂度是`O(n),`，因为它需要遍历地图中的所有元素。**

### 2.8.迭代一个`HashMap`

有三种基本方法可以迭代一个`HashMap`中的所有键值对。

我们可以迭代所有键的集合:

```java
for(String key : productsByName.keySet()) {
    Product product = productsByName.get(key);
}
```

或者我们可以迭代所有条目的集合:

```java
for(Map.Entry<String, Product> entry : productsByName.entrySet()) {
    Product product =  entry.getValue();
    String key = entry.getKey();
    //do something with the key and value
}
```

最后，我们可以迭代所有的值:

```java
List<Product> products = new ArrayList<>(productsByName.values());
```

## 3.钥匙

在我们的`HashMap`中，我们可以使用任何类作为键。然而，为了让地图正常工作，我们需要为`equals()`和 `**hashCode().**` 提供一个实现，假设我们想要一个地图，其中产品作为键，价格作为值:

```java
HashMap<Product, Integer> priceByProduct = new HashMap<>();
priceByProduct.put(eBike, 900);
```

让我们实现`equals()`和`hashCode()`方法:

```java
@Override
public boolean equals(Object o) {
    if (this == o) {
        return true;
    }
    if (o == null || getClass() != o.getClass()) {
        return false;
    }

    Product product = (Product) o;
    return Objects.equals(name, product.name) &&
      Objects.equals(description, product.description);
}

@Override
public int hashCode() {
    return Objects.hash(name, description);
}
```

**注意，`hashCode()`和`equals()`只需要为我们想用作映射键的类而被覆盖，而不需要为在映射中只用作值的类而被覆盖。**我们将在本文的第 5 节中了解这一点的必要性。

## 4.从 Java 8 开始的其他方法

Java 8 给`HashMap`增加了几个函数式的方法。在这一节中，我们将看看其中的一些方法。

对于每种方法，我们来看两个例子。第一个例子展示了如何使用新方法，第二个例子展示了如何在 Java 的早期版本中实现相同的功能。

由于这些方法非常简单，我们不会看更详细的例子。

### 4.1.`forEach()`

`forEach`方法是遍历地图中所有元素的函数式方法:

```java
productsByName.forEach( (key, product) -> {
    System.out.println("Key: " + key + " Product:" + product.getDescription());
    //do something with the key and value
}); 
```

Java 8 之前的版本:

```java
for(Map.Entry<String, Product> entry : productsByName.entrySet()) {
    Product product =  entry.getValue();
    String key = entry.getKey();
    //do something with the key and value
}
```

我们的文章《Java 8`forEach`T2 指南》更详细地介绍了`forEach`循环。

### 4.2.`getOrDefault()`

使用`getOrDefault()`方法，我们可以从映射中获得一个值，或者在给定键没有映射的情况下返回一个默认元素:

```java
Product chocolate = new Product("chocolate", "something sweet");
Product defaultProduct = productsByName.getOrDefault("horse carriage", chocolate); 
Product bike = productsByName.getOrDefault("E-Bike", chocolate);
```

Java 8 之前的版本:

```java
Product bike2 = productsByName.containsKey("E-Bike") 
    ? productsByName.get("E-Bike") 
    : chocolate;
Product defaultProduct2 = productsByName.containsKey("horse carriage") 
    ? productsByName.get("horse carriage") 
    : chocolate; 
```

### 4.3.`putIfAbsent()`

使用这种方法，我们可以添加一个新的映射，但前提是对于给定的键还没有映射:

```java
productsByName.putIfAbsent("E-Bike", chocolate); 
```

Java 8 之前的版本:

```java
if(productsByName.containsKey("E-Bike")) {
    productsByName.put("E-Bike", chocolate);
}
```

我们的文章[用 Java 8](/web/20221205110945/https://www.baeldung.com/java-merge-maps) 合并两个地图仔细研究了这种方法。

### 4.4.`merge()`

使用`[merge()](/web/20221205110945/https://www.baeldung.com/java-merge-maps),`,如果存在映射，我们可以修改给定键的值，否则添加新值:

```java
Product eBike2 = new Product("E-Bike", "A bike with a battery");
eBike2.getTags().add("sport");
productsByName.merge("E-Bike", eBike2, Product::addTagsOfOtherProduct);
```

Java 8 之前的版本:

```java
if(productsByName.containsKey("E-Bike")) {
    productsByName.get("E-Bike").addTagsOfOtherProduct(eBike2);
} else {
    productsByName.put("E-Bike", eBike2);
} 
```

### 4.5.`compute()`

使用`compute()`方法，我们可以计算给定键的值:

```java
productsByName.compute("E-Bike", (k,v) -> {
    if(v != null) {
        return v.addTagsOfOtherProduct(eBike2);
    } else {
        return eBike2;
    }
});
```

Java 8 之前的版本:

```java
if(productsByName.containsKey("E-Bike")) {    
    productsByName.get("E-Bike").addTagsOfOtherProduct(eBike2); 
} else {
    productsByName.put("E-Bike", eBike2); 
} 
```

值得注意的是，**的方法`merge()`和`compute()`非常相似。**`compute() method` 接受两个参数:用于重新映射的`key`和`BiFunction`。并且`merge()`接受三个参数:一个是`key`，一个是`default value`，如果键还不存在就添加到映射中，还有一个是`BiFunction`，用于重新映射。

## 5.`HashMap`内部构件

在这一节中，我们将看看`HashMap`如何在内部工作，以及使用`HashMap`而不是简单的列表有什么好处。

正如我们所见，我们可以通过关键字从`HashMap`中检索元素。一种方法是使用一个列表，遍历所有元素，当我们找到一个与键匹配的元素时返回。这种方法的时间和空间复杂度都是`O(n)`。

**有了`HashMap`，我们可以实现`put`和`get`操作的平均时间复杂度`O(1)`和`O(n)`的空间复杂度。让我们看看它是如何工作的。**

### 5.1.哈希代码和等于

**不是遍历所有的元素，`HashMap`试图根据一个值的键来计算它的位置。**

最简单的方法是创建一个包含尽可能多的键的列表。举个例子，假设我们的键是一个小写字符。那么拥有一个大小为 26 的列表就足够了，如果我们想访问键为' c '的元素，我们就知道它是位置 3 的元素，我们可以直接检索它。

然而，如果我们有一个大得多的密钥空间，这种方法就不会非常有效。例如，假设我们的密钥是一个整数。在这种情况下，列表的大小必须是 2，147，483，647。在大多数情况下，我们的元素也会少得多，所以大部分分配的内存都没有使用。

**`HashMap`将元素存储在所谓的桶中，桶的数量称为`capacity`。**

当我们在 map 中放入一个值时，键的`hashCode()`方法用于确定存储该值的桶。

为了检索该值，`HashMap`以同样的方式计算 bucket 使用`hashCode()`。然后，它遍历在该桶中找到的对象，并使用 key 的`equals()`方法找到精确匹配。

### 5.2.密钥的不变性

在大多数情况下，我们应该使用不可变的键。或者至少，我们必须意识到使用可变键的后果。

让我们来看看，当我们使用键在 map 中存储一个值之后，键发生了什么变化。

对于这个例子，我们将创建`MutableKey`:

```java
public class MutableKey {
    private String name;

    // standard constructor, getter and setter

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        MutableKey that = (MutableKey) o;
        return Objects.equals(name, that.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name);
    }
}
```

测试开始了:

```java
MutableKey key = new MutableKey("initial");

Map<MutableKey, String> items = new HashMap<>();
items.put(key, "success");

key.setName("changed");

assertNull(items.get(key));
```

正如我们所看到的，一旦键发生变化，我们就不能再获得相应的值，而是返回`null`。**这是因为`HashMap`在错误的桶中搜索。**

如果我们没有很好地理解`HashMap`内部是如何工作的，上面的测试案例可能会令人惊讶。

### 5.3.碰撞

**为了正确工作，相同的键必须有相同的散列，然而，不同的键可以有相同的散列**。如果两个不同的键具有相同的散列，则属于它们的两个值将被存储在同一个桶中。在一个 bucket 中，值存储在一个列表中，并通过遍历所有元素来检索。这个的成本是`O(n)`。

从 Java 8 开始(参见 [JEP 180](https://web.archive.org/web/20221205110945/https://openjdk.java.net/jeps/180) )，如果一个桶包含 8 个或更多的值，那么存储一个桶中的值的数据结构将从列表改变为平衡树，如果在某个时刻桶中只剩下 6 个值，它将变回列表。这就把性能提高到了`O(log n)`。

### 5.4.容量和负载系数

为了避免许多存储桶具有多个值，如果 75%(负载系数)的存储桶变为非空，则容量会翻倍。负载系数的默认值为 75%，默认初始容量为 16。两者都可以在构造函数中设置。

### 5.5.`put`和`get`操作总结

让我们总结一下`put`和`get`操作是如何工作的。

**当我们在地图上添加一个元素时，** `HashMap`会计算这个桶。如果存储桶已经包含一个值，那么该值将被添加到属于该存储桶的列表(或树)中。如果负载系数变得大于 map 的最大负载系数，则容量加倍。

**当我们想从 map 中获取一个值的时候，** `HashMap`计算 bucket，从 list(或 tree)中用相同的键获取值。

## 6.结论

在本文中，我们看到了如何使用一个`HashMap`以及它如何在内部工作。和`ArrayList`，`HashMap`是 Java 中最常用的数据结构之一，所以很好地了解如何使用它以及它是如何工作的非常方便。我们的文章[罩下的 Java 散列表](/web/20221205110945/https://www.baeldung.com/java-hashmap-advanced)更详细地介绍了*散列表*的内部。

和往常一样，完整的源代码可以在 Github 的[上找到。](https://web.archive.org/web/20221205110945/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-2)