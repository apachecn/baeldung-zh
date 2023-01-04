# Java 中的泛型构造函数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-generic-constructors>

## 1.概观

我们之前讨论过 [Java 泛型](/web/20221206163153/https://www.baeldung.com/java-generics)的基础。在本教程中，我们将看看 Java 中的泛型构造函数。

泛型构造函数是至少有一个泛型类型参数的构造函数。

我们会看到泛型构造函数不一定要在泛型类中，也不是泛型类中的所有构造函数都必须是泛型的。

## 2.非泛型类

首先，我们有一个简单的类`Entry`，它不是一个泛型类:

```java
public class Entry {
    private String data;
    private int rank;
}
```

在这个类中，我们将添加两个构造函数:一个带两个参数的基本构造函数和一个泛型构造函数。

### 2.1.基本构造函数

第一个`Entry`构造函数是一个简单的构造函数，有两个参数:

```java
public Entry(String data, int rank) {
    this.data = data;
    this.rank = rank;
}
```

现在，让我们使用这个基本的构造函数来创建一个`Entry`对象:

```java
@Test
public void givenNonGenericConstructor_whenCreateNonGenericEntry_thenOK() {
    Entry entry = new Entry("sample", 1);

    assertEquals("sample", entry.getData());
    assertEquals(1, entry.getRank());
}
```

### 2.2.通用构造函数

接下来，我们的第二个构造函数是一个泛型构造函数:

```java
public <E extends Rankable & Serializable> Entry(E element) {
    this.data = element.toString();
    this.rank = element.getRank();
}
```

**尽管`Entry`类不是泛型的，但它有一个泛型构造函数，因为它有一个类型为`E`的参数`element`。**

泛型类型`E`是有界的，应该同时实现`Rankable`和`Serializable`接口。

现在，让我们看看`Rankable`接口，它有一个方法:

```java
public interface Rankable {
    public int getRank();
}
```

假设我们有一个实现了`Rankable`接口的类`Product`:

```java
public class Product implements Rankable, Serializable {
    private String name;
    private double price;
    private int sales;

    public Product(String name, double price) {
        this.name = name;
        this.price = price;
    }

    @Override
    public int getRank() {
        return sales;
    }
}
```

然后，我们可以使用通用构造函数通过一个`Product`来创建`Entry`对象:

```java
@Test
public void givenGenericConstructor_whenCreateNonGenericEntry_thenOK() {
    Product product = new Product("milk", 2.5);
    product.setSales(30);

    Entry entry = new Entry(product);

    assertEquals(product.toString(), entry.getData());
    assertEquals(30, entry.getRank());
}
```

## 3.通用类

接下来，我们将看看一个名为`GenericEntry`的泛型类:

```java
public class GenericEntry<T> {
    private T data;
    private int rank;
}
```

我们还将在这个类中添加与上一节相同的两种类型的构造函数。

### 3.1.基本构造函数

首先，让我们为我们的`GenericEntry`类编写一个简单的非泛型构造函数:

```java
public GenericEntry(int rank) {
    this.rank = rank;
}
```

**尽管`GenericEntry`是一个泛型类，但这是一个简单的构造函数，没有泛型类型的参数。**

现在，我们可以使用这个构造函数来创建一个`GenericEntry<String>`:

```java
@Test
public void givenNonGenericConstructor_whenCreateGenericEntry_thenOK() {
    GenericEntry<String> entry = new GenericEntry<String>(1);

    assertNull(entry.getData());
    assertEquals(1, entry.getRank());
}
```

### 3.2.通用构造函数

接下来，让我们向我们的类添加第二个构造函数:

```java
public GenericEntry(T data, int rank) {
    this.data = data;
    this.rank = rank;
}
```

**这是一个泛型构造函数，因为它有一个泛型类型`T`的`data`参数。**注意，我们不需要在构造函数声明中添加`<T>`，因为它隐含在那里。

现在，让我们测试我们的通用构造函数:

```java
@Test
public void givenGenericConstructor_whenCreateGenericEntry_thenOK() {
    GenericEntry<String> entry = new GenericEntry<String>("sample", 1);

    assertEquals("sample", entry.getData());
    assertEquals(1, entry.getRank());        
}
```

## 4.不同类型的泛型构造函数

在我们的泛型类中，我们也可以有一个泛型类型不同于类的泛型类型的构造函数:

```java
public <E extends Rankable & Serializable> GenericEntry(E element) {
    this.data = (T) element;
    this.rank = element.getRank();
}
```

这个`GenericEntry`构造函数有一个类型为`E`的参数`element`，它不同于`T`类型。让我们来看看它的实际应用:

```java
@Test
public void givenGenericConstructorWithDifferentType_whenCreateGenericEntry_thenOK() {
    Product product = new Product("milk", 2.5);
    product.setSales(30);

    GenericEntry<Serializable> entry = new GenericEntry<Serializable>(product);

    assertEquals(product, entry.getData());
    assertEquals(30, entry.getRank());
}
```

请注意:

*   在我们的例子中，我们使用`Product` ( `E`)创建一个`Serializable` ( `T`)类型的`GenericEntry`
*   只有当类型为`E`的参数可以转换为`T`时，我们才能使用这个构造函数

## 5.多个泛型类型

接下来，我们有一个包含两个泛型类型的泛型类`MapEntry`:

```java
public class MapEntry<K, V> {
    private K key;
    private V value;

    public MapEntry(K key, V value) {
        this.key = key;
        this.value = value;
    }
}
```

**`MapEntry`有一个带两个参数的泛型构造函数，每个参数都是不同的类型。**让我们在一个简单的单元测试中使用它:

```java
@Test
public void givenGenericConstructor_whenCreateGenericEntryWithTwoTypes_thenOK() {
    MapEntry<String,Integer> entry = new MapEntry<String,Integer>("sample", 1);

    assertEquals("sample", entry.getKey());
    assertEquals(1, entry.getValue().intValue());        
}
```

## 6.通配符

最后，我们可以在通用构造函数中使用通配符:

```java
public GenericEntry(Optional<? extends Rankable> optional) {
    if (optional.isPresent()) {
        this.data = (T) optional.get();
        this.rank = optional.get().getRank();
    }
}
```

这里，我们在这个`GenericEntry`构造函数中使用通配符来绑定`Optional`类型:

```java
@Test
public void givenGenericConstructorWithWildCard_whenCreateGenericEntry_thenOK() {
    Product product = new Product("milk", 2.5);
    product.setSales(30);
    Optional<Product> optional = Optional.of(product);

    GenericEntry<Serializable> entry = new GenericEntry<Serializable>(optional);

    assertEquals(product, entry.getData());
    assertEquals(30, entry.getRank());
}
```

注意，我们应该能够将可选的参数类型(在我们的例子中为`Product`)转换为`GenericEntry`类型(在我们的例子中为`Serializable`)。

## 7.结论

在本文中，我们学习了如何在泛型和非泛型类中定义和使用泛型构造函数。

完整的源代码可以在 GitHub 上找到。