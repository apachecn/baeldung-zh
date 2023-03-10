# 用 Hibernate/JPA 批量插入/更新

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-hibernate-batch-insert-update>

## 1.概观

在本教程中，我们将学习如何使用 [Hibernate/JPA](/web/20220628083432/https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa) 批量插入和更新实体。

批处理允许我们在一次网络调用中向数据库发送一组 SQL 语句。这样，我们可以优化应用程序的网络和内存使用。

## 2.设置

### 2.1.样本数据模型

让我们看看我们将在示例中使用的样本数据模型。

首先，我们将创建一个`School`实体:

```java
@Entity
public class School {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private long id;

    private String name;

    @OneToMany(mappedBy = "school")
    private List<Student> students;

    // Getters and setters...
}
```

每个`School`将有零个或多个`Student`:

```java
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private long id;

    private String name;

    @ManyToOne
    private School school;

    // Getters and setters...
}
```

### 2.2.跟踪 SQL 查询

在运行我们的示例时，我们需要验证 insert/update 语句确实是成批发送的。不幸的是，我们无法从 [Hibernate 日志语句](/web/20220628083432/https://www.baeldung.com/sql-logging-spring-boot)中判断 SQL 语句是否是批处理的。因此，我们将使用数据源代理来跟踪 Hibernate/JPA SQL 语句:

```java
private static class ProxyDataSourceInterceptor implements MethodInterceptor {
    private final DataSource dataSource;
    public ProxyDataSourceInterceptor(final DataSource dataSource) {
        this.dataSource = ProxyDataSourceBuilder.create(dataSource)
            .name("Batch-Insert-Logger")
            .asJson().countQuery().logQueryToSysOut().build();
    }

    // Other methods...
}
```

## 3.默认行为

**Hibernate 默认不启用批处理**。这意味着它将为每个插入/更新操作发送一个单独的 SQL 语句:

```java
@Transactional
@Test
public void whenNotConfigured_ThenSendsInsertsSeparately() {
    for (int i = 0; i < 10; i++) {
        School school = createSchool(i);
        entityManager.persist(school);
    }
    entityManager.flush();
}
```

这里我们持久化了 10 个`School`实体。如果我们查看查询日志，我们可以看到 Hibernate 分别发送每个 insert 语句:

```java
"querySize":1, "batchSize":0, "query":["insert into school (name, id) values (?, ?)"], 
  "params":[["School1","1"]]
"querySize":1, "batchSize":0, "query":["insert into school (name, id) values (?, ?)"], 
  "params":[["School2","2"]]
"querySize":1, "batchSize":0, "query":["insert into school (name, id) values (?, ?)"], 
  "params":[["School3","3"]]
"querySize":1, "batchSize":0, "query":["insert into school (name, id) values (?, ?)"], 
  "params":[["School4","4"]]
"querySize":1, "batchSize":0, "query":["insert into school (name, id) values (?, ?)"], 
  "params":[["School5","5"]]
"querySize":1, "batchSize":0, "query":["insert into school (name, id) values (?, ?)"], 
  "params":[["School6","6"]]
"querySize":1, "batchSize":0, "query":["insert into school (name, id) values (?, ?)"], 
  "params":[["School7","7"]]
"querySize":1, "batchSize":0, "query":["insert into school (name, id) values (?, ?)"], 
  "params":[["School8","8"]]
"querySize":1, "batchSize":0, "query":["insert into school (name, id) values (?, ?)"], 
  "params":[["School9","9"]]
"querySize":1, "batchSize":0, "query":["insert into school (name, id) values (?, ?)"], 
  "params":[["School10","10"]]
```

因此，我们应该配置 Hibernate 来启用批处理。为此，**我们应该将`hibernate.jdbc.batch_size` 属性设置为一个大于 0 的数字**。

如果我们手动创建`EntityManager` ，我们应该将`hibernate.jdbc.batch_size`添加到 Hibernate 属性中:

```java
public Properties hibernateProperties() {
    Properties properties = new Properties();
    properties.put("hibernate.jdbc.batch_size", "5");

    // Other properties...
    return properties;
}
```

如果我们使用 Spring Boot，我们可以将其定义为应用程序属性:

```java
spring.jpa.properties.hibernate.jdbc.batch_size=5
```

## 4.单个表的批量插入

### 4.1.没有显式刷新的批量插入

让我们首先看看当我们只处理一种实体类型时，如何使用批量插入。

我们将使用前面的代码示例，但这次启用了批处理:

```java
@Transactional
@Test
public void whenInsertingSingleTypeOfEntity_thenCreatesSingleBatch() {
    for (int i = 0; i < 10; i++) {
        School school = createSchool(i);
        entityManager.persist(school);
    }
}
```

这里我们持久化了 10 个`School` 实体。当我们查看日志时，我们可以验证 Hibernate 成批发送 insert 语句:

```java
"batch":true, "querySize":1, "batchSize":5, "query":["insert into school (name, id) values (?, ?)"], 
  "params":[["School1","1"],["School2","2"],["School3","3"],["School4","4"],["School5","5"]]
"batch":true, "querySize":1, "batchSize":5, "query":["insert into school (name, id) values (?, ?)"], 
  "params":[["School6","6"],["School7","7"],["School8","8"],["School9","9"],["School10","10"]]
```

这里要提到的一件重要的事情是内存消耗。当我们持久化一个实体时，Hibernate 将它存储在持久化上下文中。例如，如果我们在一个事务中持久化 100，000 个实体，我们最终会在内存中拥有 100，000 个实体实例，这可能会导致一个`OutOfMemoryException`。

### 4.2.带有显式刷新的批量插入

现在我们来看看如何在批处理操作中优化内存使用。让我们深入挖掘持久性上下文的角色。

首先，持久性上下文在内存中存储新创建和修改的实体。当事务被同步时，Hibernate 将这些更改发送到数据库。这通常发生在交易结束时。然而，**调用`EntityManager.flush()` 也会触发一个事务同步**。

其次，持久性上下文充当实体缓存，也称为一级缓存。**为了清除持久化上下文中的实体，我们可以调用** `**EntityManager.clear()**.`

因此，为了减少批处理期间的内存负载，每当达到批处理大小时，我们可以在应用程序代码上调用`EntityManager.flush()`和`EntityManager.clear()`:

```java
@Transactional
@Test
public void whenFlushingAfterBatch_ThenClearsMemory() {
    for (int i = 0; i < 10; i++) {
        if (i > 0 && i % BATCH_SIZE == 0) {
            entityManager.flush();
            entityManager.clear();
        }
        School school = createSchool(i);
        entityManager.persist(school);
    }
}
```

在这里，我们刷新持久性上下文中的实体，从而使 Hibernate 向数据库发送查询。此外，通过清除持久性上下文，我们从内存中移除了`School`实体。批处理行为将保持不变。

## 5.多个表的批量插入

现在让我们看看在一个事务中处理多个实体类型时，如何配置批量插入。

当我们想要持久化几种类型的实体时，Hibernate 为每种实体类型创建不同的批处理。这是因为**在单个批次**中只能有一种类型的实体。

此外，当 Hibernate 收集 insert 语句时，只要遇到与当前批处理中不同的实体类型，它就会创建一个新的批处理。即使该实体类型已经有一个批处理，也是如此:

```java
@Transactional
@Test
public void whenThereAreMultipleEntities_ThenCreatesNewBatch() {
    for (int i = 0; i < 10; i++) {
        if (i > 0 && i % BATCH_SIZE == 0) {
            entityManager.flush();
            entityManager.clear();
        }
        School school = createSchool(i);
        entityManager.persist(school);
        Student firstStudent = createStudent(school);
        Student secondStudent = createStudent(school);
        entityManager.persist(firstStudent);
        entityManager.persist(secondStudent);
    }
}
```

这里我们插入一个`School,` 并给它分配两个`Student`，重复这个过程 10 次。

在日志中，我们看到 Hibernate 在几个大小为 1 的批处理中发送了`School` insert 语句，而我们预期只有 2 个大小为 5 的批处理。此外，`Student` insert 语句也以大小为 2 的几个批次发送，而不是大小为 5 的 4 个批次:

```java
"batch":true, "querySize":1, "batchSize":1, "query":["insert into school (name, id) values (?, ?)"], 
  "params":[["School1","1"]]
"batch":true, "querySize":1, "batchSize":2, "query":["insert into student (name, school_id, id) 
  values (?, ?, ?)"], "params":[["Student-School1","1","2"],["Student-School1","1","3"]]
"batch":true, "querySize":1, "batchSize":1, "query":["insert into school (name, id) values (?, ?)"], 
  "params":[["School2","4"]]
"batch":true, "querySize":1, "batchSize":2, "query":["insert into student (name, school_id, id) 
  values (?, ?, ?)"], "params":[["Student-School2","4","5"],["Student-School2","4","6"]]
"batch":true, "querySize":1, "batchSize":1, "query":["insert into school (name, id) values (?, ?)"], 
  "params":[["School3","7"]]
"batch":true, "querySize":1, "batchSize":2, "query":["insert into student (name, school_id, id) 
  values (?, ?, ?)"], "params":[["Student-School3","7","8"],["Student-School3","7","9"]]
Other log lines...
```

为了批量处理相同实体类型的所有 insert 语句，**我们应该配置`hibernate.order_inserts`属性**。

我们可以使用`EntityManagerFactory`手动配置 Hibernate 属性:

```java
public Properties hibernateProperties() {
    Properties properties = new Properties();
    properties.put("hibernate.order_inserts", "true");

    // Other properties...
    return properties;
}
```

如果我们使用 Spring Boot，我们可以在应用程序中配置属性。属性:

```java
spring.jpa.properties.hibernate.order_inserts=true
```

添加该属性后，我们将有 1 批`School` 插入和 2 批`Student` 插入:

```java
"batch":true, "querySize":1, "batchSize":5, "query":["insert into school (name, id) values (?, ?)"], 
  "params":[["School6","16"],["School7","19"],["School8","22"],["School9","25"],["School10","28"]]
"batch":true, "querySize":1, "batchSize":5, "query":["insert into student (name, school_id, id) 
  values (?, ?, ?)"], "params":[["Student-School6","16","17"],["Student-School6","16","18"],
  ["Student-School7","19","20"],["Student-School7","19","21"],["Student-School8","22","23"]]
"batch":true, "querySize":1, "batchSize":5, "query":["insert into student (name, school_id, id) 
  values (?, ?, ?)"], "params":[["Student-School8","22","24"],["Student-School9","25","26"],
  ["Student-School9","25","27"],["Student-School10","28","29"],["Student-School10","28","30"]]
```

## 6.批量更新

现在让我们转到批量更新。类似于批量插入，我们可以将几个 update 语句分组，然后一次性发送到数据库。

为了启用这个，**我们将配置`hibernate.order_updates`和`hibernate.batch_versioned_data`属性**。

如果我们手动创建我们的`EntityManagerFactory` ,我们可以通过编程来设置属性:

```java
public Properties hibernateProperties() {
    Properties properties = new Properties();
    properties.put("hibernate.order_updates", "true");
    properties.put("hibernate.batch_versioned_data", "true");

    // Other properties...
    return properties;
}
```

如果我们使用 Spring Boot，我们只需将它们添加到应用程序中。属性:

```java
spring.jpa.properties.hibernate.order_updates=true
spring.jpa.properties.hibernate.batch_versioned_data=true
```

配置完这些属性后，Hibernate 应该分批对更新语句进行分组:

```java
@Transactional
@Test
public void whenUpdatingEntities_thenCreatesBatch() {
    TypedQuery<School> schoolQuery = 
      entityManager.createQuery("SELECT s from School s", School.class);
    List<School> allSchools = schoolQuery.getResultList();
    for (School school : allSchools) {
        school.setName("Updated_" + school.getName());
    }
}
```

这里我们已经更新了学校实体，Hibernate 分两批发送 SQL 语句，每批大小为 5:

```java
"batch":true, "querySize":1, "batchSize":5, "query":["update school set name=? where id=?"], 
  "params":[["Updated_School1","1"],["Updated_School2","2"],["Updated_School3","3"],
  ["Updated_School4","4"],["Updated_School5","5"]]
"batch":true, "querySize":1, "batchSize":5, "query":["update school set name=? where id=?"], 
  "params":[["Updated_School6","6"],["Updated_School7","7"],["Updated_School8","8"],
  ["Updated_School9","9"],["Updated_School10","10"]]
```

## 7.`@Id`生成策略

当我们想要对插入使用批处理时，我们应该知道主键生成策略。**如果我们的实体使用了`GenerationType.IDENTITY`标识符生成器， [Hibernate 将静默地禁用批量插入](https://web.archive.org/web/20220628083432/http://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#batch-session-batch)** 。

由于我们示例中的实体使用了`GenerationType.SEQUENCE`标识符生成器，Hibernate 支持批处理操作:

```java
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE)
private long id;
```

## 8.摘要

在本文中，我们研究了使用 Hibernate/JPA 进行批量插入和更新。

在 Github 上查看本文[的代码示例。](https://web.archive.org/web/20220628083432/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-crud)