# 春天的金秋简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-jinq>

## 1。简介

Jinq 提供了一种用 Java 查询数据库的直观便捷的方法。在本教程中，我们将探索**如何配置一个 Spring 项目来使用 Jinq** 以及它的一些简单的例子。

## 2。Maven 依赖关系

我们需要在`pom.xml`文件中添加[Jinq 依赖关系](https://web.archive.org/web/20221126231416/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.jinq%22%20AND%20a%3A%22jinq%22):

```java
<dependency>
    <groupId>org.jinq</groupId>
    <artifactId>jinq-jpa</artifactId>
    <version>1.8.22</version>
</dependency>
```

对于 Spring，我们将在`pom.xml`文件中添加[Spring ORM 依赖关系](https://web.archive.org/web/20221126231416/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-orm%22):

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-orm</artifactId>
    <version>5.3.3</version>
</dependency>
```

最后，为了测试，我们将使用一个 H2 内存数据库，所以我们也将这个[依赖项](https://web.archive.org/web/20221126231416/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.h2database%22%20AND%20a%3A%22h2%22)和`spring-boot-starter-data-jpa`添加到 pom.xml 文件中:

```java
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.200</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>2.7.2</version>
</dependency>
```

## 3。了解 Jinq

Jinq 通过公开一个基于 Java Stream API 的 fluent API，帮助我们编写更简单、更可读的数据库查询。

让我们看一个按车型过滤汽车的例子:

```java
jinqDataProvider.streamAll(entityManager, Car.class)
  .where(c -> c.getModel().equals(model))
  .toList();
```

**Jinq 以一种高效的方式将上述代码片段转换成一个 SQL 查询**，因此本例中的最终查询将是:

```java
select c.* from car c where c.model=?
```

由于我们不使用纯文本来编写查询，而是使用类型安全的 API，这种方法不容易出错。

此外，Jinq 旨在通过使用通用、易读的表达式来加快开发速度。

然而，它在我们可以使用的类型和操作的数量上有一些限制，我们将在下面看到。

### 3.1。局限性

Jinq 只支持 JPA 中的基本类型和 SQL 函数的具体列表。它通过将所有对象和方法映射到 JPA 数据类型和 SQL 函数，将 lambda 操作转换为原生 SQL 查询。

因此，我们不能指望该工具翻译每一个自定义类型或一个类型的所有方法。

### 3.2。支持的数据类型

让我们看看支持的数据类型和方法:

*   `String`–`equals()`，`compareTo()`仅方法
*   原始数据类型–算术运算
*   `Enums`和自定义类–supports = = and！=仅操作
*   `java.util.Collection –` `contains()`
*   `Date`API–仅`equals()`、`before()`、`after()`方法

**注意:如果我们想要定制从 Java 对象到数据库对象的转换，我们需要在 Jinq 中注册一个`AttributeConverter`的具体实现。**

## 4。集成 Jinq 和 Spring

**Jinq 需要一个 [`EntityManager`](https://web.archive.org/web/20221126231416/https://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html) 实例来获取持久上下文。**在本教程中，我们将介绍一个简单的 Spring 方法，让 Jinq 与 [Hibernate](https://web.archive.org/web/20221126231416/http://hibernate.org/) 提供的`EntityManager`一起工作。

### 4.1。储存库接口

**Spring 使用存储库的概念来管理实体。**让我们来看看我们的`CarRepository`接口，其中我们有一个方法来检索给定模型的`Car`:

```java
public interface CarRepository {
    Optional<Car> findByModel(String model);
}
```

### 4.2。抽象基础知识库

接下来，**我们需要一个基础库**来提供所有的 Jinq 功能:

```java
public abstract class BaseJinqRepositoryImpl<T> {
    @Autowired
    private JinqJPAStreamProvider jinqDataProvider;

    @PersistenceContext
    private EntityManager entityManager;

    protected abstract Class<T> entityType();

    public JPAJinqStream<T> stream() {
        return streamOf(entityType());
    }

    protected <U> JPAJinqStream<U> streamOf(Class<U> clazz) {
        return jinqDataProvider.streamAll(entityManager, clazz);
    }
}
```

### 4.3。实现存储库

**现在，Jinq 需要的只是一个`EntityManager`实例和实体类型类。**

让我们看一下使用我们刚刚定义的 Jinq 基本存储库的`Car`存储库实现:

```java
@Repository
public class CarRepositoryImpl 
  extends BaseJinqRepositoryImpl<Car> implements CarRepository {

    @Override
    public Optional<Car> findByModel(String model) {
        return stream()
          .where(c -> c.getModel().equals(model))
          .findFirst();
    }

    @Override
    protected Class<Car> entityType() {
        return Car.class;
    }
}
```

### 4.4。布线`JinqJPAStreamProvider`

为了连接`JinqJPAStreamProvider`实例，我们将**添加 Jinq 提供者配置:**

```java
@Configuration
public class JinqProviderConfiguration {

    @Bean
    @Autowired
    JinqJPAStreamProvider jinqProvider(EntityManagerFactory emf) {
        return new JinqJPAStreamProvider(emf);
    }
}
```

### 4.5。配置 Spring 应用程序

最后一步是使用 Hibernate 和我们的 Jinq 配置来配置我们的 Spring 应用程序。作为参考，参见我们的`application.properties`文件，其中我们使用内存中的 H2 实例作为数据库:

```java
spring.datasource.url=jdbc:h2:~/jinq
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.hibernate.ddl-auto=create-drop
```

## 5。查询指南

**Jinq 提供了许多直观的选项，可以用`select, where,` `joins and more.`** 定制最终的 SQL 查询。注意，这些选项也有我们前面已经介绍过的限制。

### 5.1。其中

`where`子句允许对一个数据集合应用多个过滤器。

在下一个示例中，我们希望按车型和描述过滤汽车:

```java
stream()
  .where(c -> c.getModel().equals(model)
    && c.getDescription().contains(desc))
  .toList();
```

这是 Jinq 翻译的 SQL:

```java
select c.model, c.description from car c where c.model=? and locate(?, c.description)>0
```

### 5.2。选择

如果我们只想从数据库中检索一些列/字段，我们需要使用`select`子句。

为了映射多个值，Jinq 提供了许多具有多达八个值的`Tuple`类:

```java
stream()
  .select(c -> new Tuple3<>(c.getModel(), c.getYear(), c.getEngine()))
  .toList()
```

以及翻译后的 SQL:

```java
select c.model, c.year, c.engine from car c
```

### 5.3。加入

**如果实体正确链接，Jinq 能够解决一对一和多对一关系**。

例如，如果我们在`Car`中添加制造商实体:

```java
@Entity(name = "CAR")
public class Car {
    //...
    @OneToOne
    @JoinColumn(name = "name")
    public Manufacturer getManufacturer() {
        return manufacturer;
    }
}
```

以及具有列表`Car`的`Manufacturer`实体:

```java
@Entity(name = "MANUFACTURER")
public class Manufacturer {
    // ...
    @OneToMany(mappedBy = "model")
    public List<Car> getCars() {
        return cars;
    }
}
```

我们现在能够获得给定模型的`Manufacturer`:

```java
Optional<Manufacturer> manufacturer = stream()
  .where(c -> c.getModel().equals(model))
  .select(c -> c.getManufacturer())
  .findFirst();
```

正如所料， **Jinq 将在这个场景中使用一个内部连接 SQL 子句**:

```java
select m.name, m.city from car c inner join manufacturer m on c.name=m.name where c.model=?
```

**如果我们需要对`join`子句有更多的控制，以便对实体实现更复杂的关系，比如多对多关系，我们可以使用`join`方法:**

```java
List<Pair<Manufacturer, Car>> list = streamOf(Manufacturer.class)
  .join(m -> JinqStream.from(m.getCars()))
  .toList()
```

最后，我们可以通过使用`leftOuterJoin`方法而不是`join`方法来使用左外连接 SQL 子句。

### 5.4。聚合

到目前为止，我们介绍的所有例子都使用了`toList`或`findFirst`方法——返回我们在 Jinq 中查询的最终结果。

除了这些方法，**我们还可以使用其他方法来汇总结果**。

例如，让我们使用`count`方法来获得数据库中具体车型的汽车总数:

```java
long total = stream()
  .where(c -> c.getModel().equals(model))
  .count()
```

最终的 SQL 使用了预期的`count` SQL 方法:

```java
select count(c.model) from car c where c.model=?
```

Jinq 还提供了`sum`、`average`、`min`、`max,`等聚合方法，以及组合不同聚合的**可能性。**

### 5.5。分页

如果我们想批量读取数据，我们可以使用`limit`和`skip`方法。

让我们看一个例子，我们想跳过前 10 辆车，只得到 20 件商品:

```java
stream()
  .skip(10)
  .limit(20)
  .toList()
```

生成的 SQL 是:

```java
select c.* from car c limit ? offset ?
```

## 6。结论

我们走吧。在本文中，我们看到了一种使用 Hibernate(最低限度地)通过 Jinq 设置 Spring 应用程序的方法。

我们还简要介绍了 Jinq 的优势和一些主要特性。

和往常一样，这些资源可以在 GitHub 上找到[。](https://web.archive.org/web/20221126231416/https://github.com/eugenp/tutorials/tree/master/spring-jinq)