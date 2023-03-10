# Spring 数据中 MongoDB 文档的唯一字段

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-mongodb-unique>

## 1.介绍

在本教程中，我们将学习如何使用 Spring 数据在 [MongoDB 中定义一个唯一的字段。唯一字段是数据库设计的重要组成部分。它们同时保证了一致性和性能，防止了不应该出现的重复值。](/web/20220813065802/https://www.baeldung.com/spring-data-mongodb-tutorial)

## 2.配置

与关系数据库不同，MongoDB 不提供创建约束的选项。**因此，我们唯一的选择是创建唯一的[索引](/web/20220813065802/https://www.baeldung.com/spring-data-mongodb-index-annotations-converter)。**但是，默认情况下，Spring 数据中的自动索引创建是关闭的。首先，让我们打开我们的`application.properties`:

```java
spring.data.mongodb.auto-index-creation=true
```

使用该配置，如果索引尚不存在，将在引导时创建索引。但是，我们必须记住，在我们已经有了重复的值之后，我们就不能创建唯一的索引了。这将导致在应用程序启动时抛出一个异常。

## 3.`@Indexed`注解

`@Indexed`注释允许我们将字段标记为有索引。因为我们配置了自动索引创建，所以我们不必自己创建它们。**默认情况下，索引不是唯一的。**因此，我们必须通过`unique`属性打开它。让我们通过创建第一个示例来看看它的实际应用:

```java
@Document
public class Company {
    @Id
    private String id;

    private String name;

    @Indexed(unique = true)
    private String email;

    // getters and setters
} 
```

注意，我们仍然可以拥有我们的`@Id`注释，它完全独立于我们的索引。**这就是我们拥有一个具有唯一字段的文档所需要的一切。**因此，如果我们插入多个具有相同`email`的文档，就会产生一个`DuplicateKeyException`:

```java
@Test
public void givenUniqueIndex_whenInsertingDupe_thenExceptionIsThrown() {
    Company a = new Company();
    a.setName("Name");
    a.setEmail("[[email protected]](/web/20220813065802/https://www.baeldung.com/cdn-cgi/l/email-protection)");

    companyRepo.insert(a);

    Company b = new Company();
    b.setName("Other");
    b.setEmail("[[email protected]](/web/20220813065802/https://www.baeldung.com/cdn-cgi/l/email-protection)");
    assertThrows(DuplicateKeyException.class, () -> {
        companyRepo.insert(b);
    });
}
```

当我们想要强制惟一性，但仍然有一个自动生成的惟一 ID 字段时，这种方法很有用。

### 3.1.注释多个字段

我们还可以向多个字段添加注释。让我们继续创建第二个示例:

```java
@Document
public class Asset {
    @Indexed(unique = true)
    private String name;

    @Indexed(unique = true)
    private Integer number;
}
```

注意，我们没有在任何字段上显式设置`@Id`。MongoDB 仍然会自动为我们设置一个“`_id`”字段，但是我们的应用程序不能访问它。**但是，我们不能将`@Id`和标注为`unique`的`@Indexed`放在同一个字段中。当应用程序试图创建索引时，它会抛出一个异常。**

此外，现在我们有两个独特的领域。**注意，这并不意味着它是一个复合指数。**因此，对任何字段多次插入相同的值都会导致重复的键。让我们来测试一下:

```java
@Test
public void givenMultipleIndexes_whenAnyFieldDupe_thenExceptionIsThrown() {
    Asset a = new Asset();
    a.setName("Name");
    a.setNumber(1);

    assetRepo.insert(a);

    assertThrows(DuplicateKeyException.class, () -> {
        Asset b = new Asset();
        b.setName("Name");
        b.setNumber(2);

        assetRepo.insert(b);
    });

    assertThrows(DuplicateKeyException.class, () -> {
        Asset b = new Asset();
        b.setName("Other");
        b.setNumber(1);

        assetRepo.insert(b);
    });
}
```

如果我们只希望组合值形成一个唯一的索引，我们必须创建一个复合索引。

### 3.2.使用自定义类型作为索引

类似地，我们可以注释一个自定义类型的字段。这将达到复合指数的效果。让我们从一个`SaleId`类开始来表示我们的复合指数:

```java
public class SaleId {
    private Long item;
    private String date;

    // getters and setters
}
```

现在让我们创建我们的`Sale`类来使用它:

```java
@Document
public class Sale {
    @Indexed(unique = true)
    private SaleId saleId;

    private Double value;

    // getters and setters
}
```

现在，每次我们试图用相同的`SaleId`添加一个新的`Sale`，我们都会得到一个重复的键。让我们来测试一下:

```java
@Test
public void givenCustomTypeIndex_whenInsertingDupe_thenExceptionIsThrown() {
    SaleId id = new SaleId();
    id.setDate("2022-06-15");
    id.setItem(1L);

    Sale a = new Sale(id);
    a.setValue(53.94);

    saleRepo.insert(a);

    assertThrows(DuplicateKeyException.class, () -> {
        Sale b = new Sale(id);
        b.setValue(100.00);

        saleRepo.insert(b);
    });
}
```

这种方法的优点是保持了索引定义的独立性。**这允许我们在`SaleId`中包含或删除新字段，而不必重新创建或更新我们的索引。**也很像一个[组合键](/web/20220813065802/https://www.baeldung.com/spring-data-mongodb-composite-key)。但是，索引不同于键，因为它们可以有一个空值。

## 4.`@CompoundIndex`注解

要拥有一个由多个字段组成的唯一索引，而不需要定制类，我们必须创建一个复合索引。**为此，我们在类中直接使用了`@CompoundIndex`注释。**这个注释包含一个`def`属性，我们将使用它来包含我们需要的字段。让我们创建我们的`Customer`类，为`storeId`和`number`字段定义唯一的索引:

```java
@Document
@CompoundIndex(def = "{'storeId': 1, 'number': 1}", unique = true)
public class Customer {
    @Id
    private String id;

    private Long storeId;
    private Long number;
    private String name;

    // getters and setters
}
```

这与多个字段上的`@Indexed` 不同。如果我们尝试插入一个具有相同`storeId`和`number`值的客户，这种方法只会产生一个`DuplicateKeyException`:

```java
@Test
public void givenCompoundIndex_whenDupeInsert_thenExceptionIsThrown() {
    Customer customerA = new Customer("Name A");
    customerA.setNumber(1l);
    customerA.setStoreId(2l);

    Customer customerB = new Customer("Name B");
    customerB.setNumber(1l);
    customerB.setStoreId(2l);

    customerRepo.insert(customerA);

    assertThrows(DuplicateKeyException.class, () -> {
        customerRepo.insert(customerB);
    });
}
```

使用这种方法，我们的优势在于不必仅为我们的索引创建另一个类。**此外，还可以从复合索引定义中为字段添加`@Id`注释。**然而，与`@Indexed`不同的是，它不会导致异常。

## 5.结论

在本文中，我们学习了如何为文档定义唯一的字段。**因此，我们知道我们唯一的选择是使用唯一的索引。**此外，使用 Spring 数据，我们可以轻松地配置应用程序来自动创建索引。而且，我们看到了使用`@Indexed`和`@CompoundIndex`注释的许多方式。

和往常一样，源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220813065802/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-mongodb-2)