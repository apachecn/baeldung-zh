# spring Data–crud repository save()方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-crud-repository-save>

## 1。概述

`CrudRepository`是一个 **Spring 数据接口，用于特定类型的存储库的通用 CRUD 操作。**它提供了几种与数据库交互的现成方法。

在本教程中，我们将解释如何以及何时使用`CrudRepository` `save()`方法。

要了解更多关于 Spring 数据存储库的信息，请看一下我们的文章，它将`CrudRepository`与框架的其他存储库接口进行了比较。

## 延伸阅读:

## [春季数据 JPA @Query](/web/20220707143833/https://www.baeldung.com/spring-data-jpa-query)

Learn how to use the @Query annotation in Spring Data JPA to define custom queries using JPQL and native SQL.[Read more](/web/20220707143833/https://www.baeldung.com/spring-data-jpa-query) →

## [Spring 数据 JPA 派生的删除方法](/web/20220707143833/https://www.baeldung.com/spring-data-jpa-deleteby)

Learn how to define Spring Data deleteBy and removeBy methods[Read more](/web/20220707143833/https://www.baeldung.com/spring-data-jpa-deleteby) →

## 2。依赖性

我们必须将 [Spring 数据](https://web.archive.org/web/20220707143833/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa)和 [H2](https://web.archive.org/web/20220707143833/https://mvnrepository.com/artifact/com.h2database/h2) 数据库依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

## 3。应用示例

首先，让我们创建名为`MerchandiseEntity`的 Spring 数据实体。这个类将**定义数据类型，当我们调用`save()`** 方法时，这些数据类型将被保存到数据库中:

```java
@Entity
public class MerchandiseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private double price;

    private String brand;

    public MerchandiseEntity() {
    }

    public MerchandiseEntity(String brand, double price) {
        this.brand = brand;
        this.price = price;
    }
}
```

接下来让我们创建一个`CrudRepository`接口来使用`MerchandiseEntity`:

```java
@Repository
public interface InventoryRepository 
  extends CrudRepository<MerchandiseEntity, Long> {
}
```

**这里我们指定实体的类和实体 id 的类，`MerchandiseEntity`和`Long`** 。当这个存储库的一个实例被实例化时，底层逻辑将自动就位，与我们的`MerchandiseEntity` 类一起工作。

所以用很少的代码，我们已经能够开始使用`save()` 方法。

## 4。`CrudRepository` save()添加一个新实例

让我们创建一个新的`MerchandiseEntity`实例，并使用`InventoryRepository`将其保存到数据库中:

```java
InventoryRepository repo = context
  .getBean(InventoryRepository.class);

MerchandiseEntity pants = new MerchandiseEntity(
  "Pair of Pants", BigDecimal.ONE);
pants = repo.save(pants);
```

运行它将在数据库表中为`MerchandiseEntity`创建一个新条目。请注意，我们从未指定过一个`id`。实例最初是用一个`null` 值为其`id,` 创建的，当我们调用`save()` 方法时，会自动生成一个`id` 。

`save()` 方法返回保存的实体，包括更新的`id` 字段。

## 5。`CrudRepository` save()更新一个实例

我们可以使用相同的`save()`方法**来更新数据库**中的现有条目。假设我们保存了一个具有特定标题的`MerchandiseEntity`实例:

```java
MerchandiseEntity pants = new MerchandiseEntity(
  "Pair of Pants", 34.99);
pants = repo.save(pants); 
```

后来，我们发现我们想要更新商品的价格。然后，我们可以简单地从数据库中获取实体，进行修改，并像以前一样使用`save()`方法。

假设我们知道条目的`id`(`pantsId`)，我们可以使用`CRUDRepository`方法`findById`从数据库中获取我们的实体:

```java
MerchandiseEntity pantsInDB = repo.findById(pantsId).get(); 
pantsInDB.setPrice(44.99); 
repo.save(pantsInDB); 
```

这里，我们用新价格更新了原始实体，并将更改保存回数据库。

我们需要记住，**调用`save()` 来更新[事务方法](/web/20220707143833/https://www.baeldung.com/transaction-configuration-with-jpa-and-spring)中的对象并不是强制的**。

**当我们使用`findById()`在事务方法中检索实体时，返回的实体由持久性提供者**管理。因此，对该实体的任何更改都将自动保存在数据库中，不管我们是否调用了`save()`方法。现在，让我们创建一个简单的测试用例来证实这一点:

```java
@Test
@Transactional
public void shouldUpdateExistingEntryInDBWithoutSave() {
    MerchandiseEntity pants = new MerchandiseEntity(
      ORIGINAL_TITLE, BigDecimal.ONE);
    pants = repository.save(pants);

    Long originalId = pants.getId();

    // Update using setters
    pants.setTitle(UPDATED_TITLE);
    pants.setPrice(BigDecimal.TEN);
    pants.setBrand(UPDATED_BRAND);

    Optional<MerchandiseEntity> resultOp = repository.findById(originalId);

    assertTrue(resultOp.isPresent());
    MerchandiseEntity result = resultOp.get();

    assertEquals(originalId, result.getId());
    assertEquals(UPDATED_TITLE, result.getTitle());
    assertEquals(BigDecimal.TEN, result.getPrice());
    assertEquals(UPDATED_BRAND, result.getBrand());
}
```

## 6。结论

在这篇简短的文章中，我们介绍了`CrudRepository`的 save()方法的使用。我们可以使用这种方法向数据库中添加一个新条目，以及更新一个现有条目。

和往常一样，这篇文章的代码在 GitHub 上[。](https://web.archive.org/web/20220707143833/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-repo)