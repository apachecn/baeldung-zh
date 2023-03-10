# Spring Data Couchbase 中的实体验证、乐观锁定和查询一致性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/entity-validation-locking-and-query-consistency-in-spring-data-couchbase>

## 1。简介

在我们对 Spring Data Couchbase 的[介绍之后，在第二篇教程中，我们将重点关注对实体验证(JSR-303)、乐观锁定以及 Couchbase 文档数据库的不同级别的查询一致性的支持。](/web/20220628151411/https://www.baeldung.com/spring-data-couchbase)

## 2。实体验证

Spring Data Couchbase 支持 JSR-303 实体验证注释。为了利用这个特性，首先我们将 JSR-303 库添加到 Maven 项目的依赖部分:

```java
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>1.1.0.Final</version>
</dependency>
```

然后我们添加一个 JSR-303 的实现。我们将使用 Hibernate 实现:

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>5.2.4.Final</version>
</dependency>
```

最后，我们向 Couchbase 配置添加一个验证器工厂 bean 和相应的 Couchbase 事件监听器:

```java
@Bean
public LocalValidatorFactoryBean localValidatorFactoryBean() {
    return new LocalValidatorFactoryBean();
}

@Bean
public ValidatingCouchbaseEventListener validatingCouchbaseEventListener() {
    return new ValidatingCouchbaseEventListener(localValidatorFactoryBean());
}
```

等效的 XML 配置如下所示:

```java
<bean id="validator"
  class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean"/>

<bean id="validatingEventListener" 
  class="org.springframework.data.couchbase.core.mapping.event.ValidatingCouchbaseEventListener"/>
```

现在我们将 JSR-303 注释添加到我们的实体类中。当在持久化操作中遇到约束冲突时，操作将失败，抛出一个`ConstraintViolationException`。

以下是我们可以强制实施的涉及`Student`实体的约束示例:

```java
@Field
@NotNull
@Size(min=1, max=20)
@Pattern(regexp="^[a-zA-Z .'-]+$")
private String firstName;

...
@Field
@Past
private DateTime dateOfBirth;
```

## 3。乐观锁定

Spring Data Couchbase 不支持多文档事务，类似于您可以在 Spring Data JPA 等其他 Spring 数据模块中实现的事务(通过`@Transactional`注释)，也不提供回滚功能。

然而，通过使用`@Version`注释，它确实像其他 Spring 数据模块一样支持乐观锁定:

```java
@Version
private long version;
```

在幕后，Couchbase 使用所谓的“比较和交换”(CAS)机制来实现数据存储级别的乐观锁定。

Couchbase 中的每个文档都有一个关联的 CAS 值，该值会在文档的元数据或内容发生变化时自动修改。在一个字段上使用`@Version`注释会导致每当从 Couchbase 中检索到一个文档时，该字段就会被当前的 CAS 值填充。

当您试图将文档保存回 Couchbase 时，将根据 Couchbase 中的当前 CAS 值检查该字段。如果值不匹配，持久化操作将失败，并显示`OptimisticLockingException`。

非常重要的一点是,千万不要试图在代码中访问或修改这个字段。

## 4。查询一致性

在 Couchbase 上实现持久层时，您必须考虑陈旧读写的可能性。这是因为当插入、更新或删除文档时，可能需要一段时间才能更新后台视图和索引来反映这些更改。

如果您有一个由 Couchbase 节点集群支持的大型数据集，这可能会成为一个严重的问题，尤其是对于 OLTP 系统。

Spring Data 为一些存储库和模板操作提供了强大的一致性级别，还提供了一些选项，让您可以确定应用程序可以接受的读写一致性级别。

### 4.1。一致性级别

Spring Data 允许您通过`org.springframework.data.couchbase.core.query`包中的`Consistency` 枚举为应用程序指定不同级别的查询一致性和陈旧性。

该枚举定义了以下级别的查询一致性和陈旧性，从最不严格到最严格:

*   `EVENTUALLY_CONSISTENT`
    *   允许陈旧读取
    *   根据 Couchbase 标准算法更新索引
*   `UPDATE_AFTER`
    *   允许陈旧读取
    *   每次请求后都会更新索引
*   `DEFAULT_CONSISTENCY`(同`READ_YOUR_OWN_WRITES`)
*   `READ_YOUR_OWN_WRITES`
    *   不允许过时读取
    *   每次请求后都会更新索引
*   `STRONGLY_CONSISTENT`
    *   不允许过时读取
    *   每条语句后都会更新索引

### 4.2。默认行为

考虑这样一种情况，您的文档已经从 Couchbase 中删除，而后台视图和索引还没有完全更新。

`CouchbaseRepository`内置方法`deleteAll()`安全地忽略了由后台视图找到的文档，但是视图还没有反映出这些文档的删除。

同样地，`CouchbaseTemplate`内置方法`findByView`和`findBySpatialView`通过不返回最初被后台视图发现但后来被删除的文档，提供了类似的一致性。

对于所有其他模板方法、内置存储库方法和派生存储库查询方法，根据撰写本文时的官方 Spring Data Couchbase 2.1.x 文档，Spring Data 使用默认的一致性级别`Consistency.READ_YOUR_OWN_WRITES.`

值得注意的是，这个库的早期版本使用了默认的`Consistency.UPDATE_AFTER`。

无论您使用哪个版本，如果您对盲目接受所提供的默认一致性级别有所保留，Spring 提供了两种方法，您可以通过这两种方法声明性地控制所使用的一致性级别，下面的小节将对此进行描述。

### 4.3。全局一致性设置

如果您正在使用 Couchbase 存储库，并且您的应用程序要求更高的一致性级别，或者如果它能够容忍较低的一致性级别，那么您可以通过在 Couchbase 配置中覆盖`getDefaultConsistency()`方法来覆盖所有存储库的默认一致性设置。

下面是如何在 Couchbase 配置类中覆盖全局一致性级别:

```java
@Override
public Consistency getDefaultConsistency() {
    return Consistency.STRONGLY_CONSISTENT;
}
```

下面是等效的 XML 配置:

```java
<couchbase:template consistency="STRONGLY_CONSISTENT"/>
```

请注意，更严格的一致性级别的代价是增加查询时的延迟，因此请确保根据您的应用程序的需求来定制此设置。

例如，一个数据仓库或报告应用程序，其中的数据经常是成批追加或更新的，这将是一个很好的选择`EVENTUALLY_CONSISTENT`，而 OLTP 应用程序可能倾向于更严格的级别，如`READ_YOUR_OWN_WRITES`或`STRONGLY_CONSISTENT`。

### 4.4。自定义一致性实现

如果您需要更好的一致性设置，您可以在逐个查询的基础上覆盖默认的一致性级别，方法是为您想要独立控制其一致性级别的任何查询提供您自己的存储库实现，并利用由`CouchbaseTemplate`提供的`queryView`和/或`queryN1QL`方法。

让我们为我们不希望允许陈旧读取的`Student`实体实现一个名为`findByFirstNameStartsWith`的定制存储库方法。

首先，创建一个包含自定义方法声明的接口:

```java
public interface CustomStudentRepository {
    List<Student> findByFirstNameStartsWith(String s);
}
```

接下来，实现接口，将底层 Couchbase Java SDK 中的`Stale`设置设置为所需的级别:

```java
public class CustomStudentRepositoryImpl implements CustomStudentRepository {

    @Autowired
    private CouchbaseTemplate template;

    public List<Student> findByFirstNameStartsWith(String s) {
        return template.findByView(ViewQuery.from("student", "byFirstName")
          .startKey(s)
          .stale(Stale.FALSE),
          Student.class);
    }
}
```

最后，通过让您的标准存储库接口扩展通用`CrudRepository`接口和您的定制存储库接口，客户端将可以访问您的标准存储库接口的所有内置和派生方法，以及您在定制存储库类中实现的任何定制方法:

```java
public interface StudentRepository extends CrudRepository<Student, String>,
  CustomStudentRepository {
    ...
}
```

## 5。结论

在本教程中，我们展示了如何在使用 Spring Data Couchbase 社区项目时实现 JSR-303 实体验证并实现乐观锁定功能。

我们还讨论了理解 Couchbase 中查询一致性的必要性，并介绍了 Spring Data Couchbase 提供的不同级别的一致性。

最后，我们解释了 Spring Data Couchbase 全局使用的默认一致性级别和一些特定的方法，并演示了覆盖全局默认一致性设置的方法，以及如何通过提供您自己的定制存储库实现来逐个查询地覆盖一致性设置。

你可以在 GitHub 项目中查看本教程的完整源代码。

要了解更多关于 Spring Data Couchbase 的信息，请访问官方的 [Spring Data Couchbase](https://web.archive.org/web/20220628151411/https://projects.spring.io/spring-data-couchbase/) 项目网站。