# Spring 数据可组合仓库

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-composable-repositories>

## 1.介绍

当对现实世界的系统或过程建模时，领域驱动设计(DDD)风格的存储库是一个很好的选择。为此，我们可以使用 Spring Data JPA 作为我们的数据访问抽象层。

如果你是这个概念的新手，请查看[这个介绍性教程](/web/20220705092756/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)来帮助你快速上手。

在本教程中，我们将重点介绍创建自定义以及可组合存储库的概念，这些存储库是使用称为片段的较小存储库创建的。

## 2.Maven 依赖性

从 Spring 5 开始，可以选择创建可组合的存储库。

让我们为 Spring 数据 JPA 添加所需的依赖项:

```java
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-jpa</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency>
```

为了让我们的数据访问层工作，我们还需要建立一个数据源。为开发和快速测试建立一个像 H2 一样的内存数据库是个好主意。

## 3.背景

### 3.1.Hibernate 作为 JPA 实现

默认情况下，Spring Data JPA 使用 Hibernate 作为 JPA 实现。我们很容易将两者混淆或比较，但它们有不同的用途。

Spring Data JPA 是数据访问抽象层，在它下面我们可以使用任何实现。例如，我们可以用 EclipseLink 取代 Hibernate。

### 3.2.默认存储库

在很多情况下，我们不需要自己编写任何查询。

相反，我们只需要创建接口来扩展通用的 Spring 数据存储库接口:

```java
public interface LocationRepository extends JpaRepository<Location, Long> {
}
```

这本身将允许我们对拥有类型为`Long`的主键的`Location`对象进行普通操作——CRUD、分页和排序。

此外，Spring Data JPA 配备了一个查询构建器机制，该机制提供了使用方法名称约定代表我们生成查询的能力:

```java
public interface StoreRepository extends JpaRepository<Store, Long> {
    List<Store> findStoreByLocationId(Long locationId);
}
```

### 3.3.自定义存储库

如果需要，我们可以**通过编写一个片段接口并实现所需的功能来丰富我们的模型库**。然后可以将它注入到我们自己的 JPA 存储库中。

例如，这里我们通过扩展一个片段库来丰富我们的`ItemTypeRepository`:

```java
public interface ItemTypeRepository 
  extends JpaRepository<ItemType, Long>, CustomItemTypeRepository {
}
```

这里`CustomItemTypeRepository`是另一个界面:

```java
public interface CustomItemTypeRepository {
    void deleteCustomById(ItemType entity);
}
```

它的实现可以是任何类型的存储库，而不仅仅是 JPA:

```java
public class CustomItemTypeRepositoryImpl implements CustomItemTypeRepository {

    @Autowired
    private EntityManager entityManager;

    @Override
    public void deleteCustomById(ItemType itemType) {
        entityManager.remove(itemType);
    }
}
```

我们只需要确保它有后缀`Impl`。但是，我们可以使用以下 XML 配置来设置自定义后缀:

```java
<repositories base-package="com.baeldung.repository" repository-impl-postfix="CustomImpl" />
```

或者使用以下注释:

```java
@EnableJpaRepositories(
  basePackages = "com.baeldung.repository", repositoryImplementationPostfix = "CustomImpl")
```

## 4.使用多个片段组成存储库

直到几个版本之前，我们只能使用单个定制实现来扩展我们的存储库接口。这是一个限制，因为我们必须将所有相关的功能放在一个对象中。

不用说，对于具有复杂领域模型的大型项目，这会导致臃肿的类。

现在有了 Spring 5，我们可以选择用多个片段存储库来丰富我们的 JPA 存储库。同样，我们仍然需要将这些片段作为接口-实现对。

为了演示这一点，让我们创建两个片段:

```java
public interface CustomItemTypeRepository {
    void deleteCustom(ItemType entity);
    void findThenDelete(Long id);
}

public interface CustomItemRepository {
    Item findItemById(Long id);
    void deleteCustom(Item entity);
    void findThenDelete(Long id);
}
```

当然，我们需要编写它们的实现。但是，我们可以扩展单个 JPA 存储库的功能，而不是将这些定制存储库(具有相关功能)插入到它们自己的 JPA 存储库中:

```java
public interface ItemTypeRepository 
  extends JpaRepository<ItemType, Long>, CustomItemTypeRepository, CustomItemRepository {
}
```

现在，我们将所有链接的功能放在一个存储库中。

## 5.处理歧义

由于我们从多个存储库继承，我们可能很难确定在冲突的情况下使用我们的哪个实现。例如，在我们的例子中，两个片段存储库都有一个方法`findThenDelete()`，具有相同的签名。

在这个场景中，**接口声明的顺序用于解决模糊性**。因此，在我们的例子中，将使用`CustomItemTypeRepository`中的方法，因为它是首先声明的。

我们可以通过使用这个测试用例来测试这一点:

```java
@Test
public void givenItemAndItemTypeWhenDeleteThenItemTypeDeleted() {
    Optional<ItemType> itemType = composedRepository.findById(1L);
    assertTrue(itemType.isPresent());

    Item item = composedRepository.findItemById(2L);
    assertNotNull(item);

    composedRepository.findThenDelete(1L);
    Optional<ItemType> sameItemType = composedRepository.findById(1L);
    assertFalse(sameItemType.isPresent());

    Item sameItem = composedRepository.findItemById(2L);
    assertNotNull(sameItem);
}
```

## 6.结论

在本文中，我们了解了使用 Spring Data JPA 存储库的不同方式。我们看到，Spring 使得在我们的域对象上执行数据库操作变得简单，无需编写太多代码，甚至无需 SQL 查询。

通过使用可组合的存储库，这种支持是可定制的。

这篇文章的代码片段可以在 GitHub 上的 [Maven 项目中找到。](https://web.archive.org/web/20220705092756/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-repo)