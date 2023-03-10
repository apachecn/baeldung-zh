# 休眠二级缓存

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-second-level-cache>

## 1。概述

数据库抽象层的优势之一，比如 ORM(对象关系映射)框架，是它们透明地缓存从底层存储中检索的数据的能力。这有助于消除频繁访问数据的数据库访问成本。

如果缓存内容的读/写比率很高，性能提升会非常显著。对于由大型对象图组成的实体来说尤其如此。

在本教程中，我们将探索 Hibernate 二级缓存。我们将解释一些基本概念，并用简单的例子来说明一切。我们将使用 JPA，对于那些 JPA 中没有标准化的特性，我们将退回到 Hibernate native API。

## 2。什么是二级缓存？

和大多数其他功能齐全的 ORM 框架一样，Hibernate 有一级缓存的概念。它是一个会话范围的缓存，确保每个实体实例在持久上下文中只加载一次。

一旦会话关闭，一级缓存也会终止。这实际上是可取的，因为它允许并发会话与彼此隔离的实体实例一起工作。

相反，二级缓存是`SessionFactory`范围的，这意味着它由使用相同会话工厂创建的所有会话共享。当一个实体实例通过它的 id 被查找时(无论是通过应用程序逻辑还是通过 Hibernate 内部，`e.g.`当它从其他实体加载到那个实体的关联时)，并且为那个实体启用了二级缓存时，会发生以下情况:

*   如果一个实例已经存在于一级缓存中，它将从那里返回。
*   如果在一级缓存中找不到实例，而相应的实例状态缓存在二级缓存中，则从二级缓存中提取数据，并组装和返回一个实例。
*   否则，将从数据库中加载必要的数据，并组装和返回一个实例。

一旦实例存储在持久性上下文(一级缓存)中，它就会在同一会话中的所有后续调用中返回，直到会话关闭，或者实例被手动从持久性上下文中逐出。如果加载的实例状态不在 L2 缓存中，它也会存储在那里。

## 3。地区工厂

Hibernate 二级缓存被设计成不知道实际使用的缓存提供者。Hibernate 只需要提供一个`org.hibernate.cache.spi.RegionFactory`接口的实现，它封装了所有特定于实际缓存提供者的细节。基本上，它充当 Hibernate 和缓存提供者之间的桥梁。

在本文中，**我们将使用 Ehcache，一个成熟且广泛使用的缓存，作为我们的缓存提供者。**我们可以选择任何其他的提供者，只要它有一个`RegionFactory`的实现。

我们将使用以下 Maven 依赖项将 Ehcache 区域工厂实现添加到类路径中:

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-ehcache</artifactId>
    <version>5.2.2.Final</version>
</dependency>
```

我们可以[在这里](https://web.archive.org/web/20221215162710/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.hibernate%22%20AND%20a%3A%22hibernate-ehcache%22)看一下`hibernate-ehcache`的最新版本。然而，我们需要确保`hibernate-ehcache`版本等于我们在项目中使用的 Hibernate 版本(`e.g.`)如果我们像这个例子中一样使用`hibernate-ehcache 5.2.2.Final,`，那么 Hibernate 的版本也应该是`5.2.2.Final).`

`hibernate-ehcache`工件依赖于 Ehcache 实现本身，它也包含在类路径中。

## 4.启用二级缓存

通过下面的两个属性，我们将告诉 Hibernate 缓存已启用，并给它取区域工厂类的名称:

```java
hibernate.cache.use_second_level_cache=true
hibernate.cache.region.factory_class=org.hibernate.cache.ehcache.EhCacheRegionFactory 
```

例如，在`persistence.xml,` 中，它看起来像是:

```java
<properties>
    ...
    <property name="hibernate.cache.use_second_level_cache" value="true"/>
    <property name="hibernate.cache.region.factory_class" 
      value="org.hibernate.cache.ehcache.EhCacheRegionFactory"/>
    ...
</properties>
```

要禁用二级缓存(比如出于调试目的)，我们只需将`hibernate.cache.use_second_level_cache`属性设置为 false。

## 5。使实体可缓存

为了使**实体符合二级缓存**的条件，我们将用 Hibernate 特有的`@org.hibernate.annotations.Cache`注释对其进行注释，并指定一个[缓存并发策略](#cacheConcurrencyStrategy)。

一些开发人员认为添加标准的`@javax.persistence.Cacheable`注释也是一个好习惯(尽管 Hibernate 并不要求)，所以实体类的实现可能如下所示:

```java
@Entity
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Foo {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "ID")
    private long id;

    @Column(name = "NAME")
    private String name;

    // getters and setters
}
```

对于每个实体类，Hibernate 将使用一个单独的缓存区域来存储该类的实例状态。区域名是完全限定的类名。

例如，`Foo`实例存储在 Ehcache 中名为`com.baeldung.hibernate.cache.model.Foo`的缓存中。

为了验证缓存工作正常，我们可以编写一个快速测试:

```java
Foo foo = new Foo();
fooService.create(foo);
fooService.findOne(foo.getId());
int size = CacheManager.ALL_CACHE_MANAGERS.get(0)
  .getCache("com.baeldung.hibernate.cache.model.Foo").getSize();
assertThat(size, greaterThan(0));
```

在这里，我们直接使用 Ehcache API 来验证在加载一个`Foo`实例后`com.baeldung.hibernate.cache.model.Foo`缓存不为空。

我们还可以启用 Hibernate 生成的 SQL 日志，并在测试中多次调用`fooService.findOne(foo.getId())`,以验证用于加载`Foo`的`select`语句只打印一次(第一次),这意味着在后续调用中，实体实例从缓存中取出。

## 6。缓存并发策略

根据使用案例，我们可以自由选择以下缓存并发策略之一:

*   **`READ_ONLY`** :仅用于从不改变的实体(如果试图更新此类实体，则会引发异常)。它非常简单，具有表演性。它适用于不变的静态参考数据。
*   **`NONSTRICT_READ_WRITE`** :更改受影响数据的事务提交后，更新缓存。因此，不能保证强一致性，而且从缓存中获取陈旧数据的时间窗口很小。这种策略适合能够容忍最终一致性的用例。
*   **`READ_WRITE`** :这种策略通过使用‘软’锁来保证强一致性。当缓存的实体被更新时，一个软锁也存储在该实体的缓存中，在事务被提交后被释放。所有访问软锁定条目的并发事务都将直接从数据库中获取相应的数据。
*   **`TRANSACTIONAL`** :缓存更改在分布式 XA 事务中完成。缓存实体中的更改在同一个 XA 事务中的数据库和缓存中提交或回滚。

## 7。缓存管理

如果没有定义过期和回收策略，缓存可能会无限增长，并最终耗尽所有可用内存。在大多数情况下，Hibernate 将这样的缓存管理任务留给了缓存提供者，因为它们确实是特定于每个缓存实现的。

例如，我们可以定义以下 Ehcache 配置，将缓存的`Foo`实例的最大数量限制为 1000:

```java
<ehcache>
    <cache name="com.baeldung.persistence.model.Foo" maxElementsInMemory="1000" />
</ehcache>
```

## 8。集合缓存

默认情况下，集合不会被缓存，我们需要显式地将它们标记为可缓存的:

```java
@Entity
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Foo {

    ...

    @Cacheable
    @org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
    @OneToMany
    private Collection<Bar> bars;

    // getters and setters
}
```

## 9。缓存状态的内部表示

实体不是作为 Java 实例存储在二级缓存中，而是以其分解(合并)状态存储:

*   不存储 Id(主键)(它作为缓存键的一部分存储)
*   瞬态属性不会被存储
*   收藏不会被储存(请参阅下文了解更多详细信息)
*   非关联属性值以其原始形式存储
*   对于`ToOne`关联，仅存储 id(外键)

这描述了一般的 Hibernate 二级缓存设计，其中缓存模型反映了底层的关系模型，这是一种空间高效的模型，可以很容易地保持两者的同步。

### 9.1。缓存集合的内部表示

我们已经提到过，我们必须明确指出一个集合(`OneToMany` 或`ManyToMany`关联)是可缓存的，否则它不会被缓存。

Hibernate 实际上将集合存储在单独的缓存区域中，每个集合一个缓存区域。区域名是完全限定的类名，加上集合属性的名称(例如，`com.baeldung.hibernate.cache.model.Foo.bars`)。这使我们能够灵活地为集合定义单独的缓存参数，`e.g.`驱逐/过期策略。

同样重要的是，对于每个集合条目，只有集合中包含的实体的 id 被缓存。这意味着，在大多数情况下，让包含的实体可缓存也是一个好主意。

## 10。HQL DML 风格查询和本地查询的缓存失效

当涉及到 DML 风格的 HQL ( `insert`、`update`和`delete` HQL 语句)时，Hibernate 能够确定哪些实体会受到这些操作的影响:

```java
entityManager.createQuery("update Foo set … where …").executeUpdate();
```

在这种情况下，所有 Foo 实例都从 L2 缓存中被逐出，而其他缓存的内容保持不变。

然而，当涉及到原生 SQL DML 语句时，Hibernate 无法猜测正在更新什么，因此它使整个二级缓存无效:

```java
session.createNativeQuery("update FOO set … where …").executeUpdate();
```

这可能不是我们想要的。解决方案是告诉 Hibernate 哪些实体受到原生 DML 语句的影响，这样它就可以只驱逐与`Foo`实体相关的条目:

```java
Query nativeQuery = entityManager.createNativeQuery("update FOO set ... where ...");
nativeQuery.unwrap(org.hibernate.SQLQuery.class).addSynchronizedEntityClass(Foo.class);
nativeQuery.executeUpdate();
```

我们必须退回到 Hibernate native `SQLQuery` API，因为这个特性在 JPA 中还没有定义。

注意，以上仅适用于 DML 语句(`insert`、`update`、`delete,`和本机函数/过程调用)。本机`select`查询不会使缓存失效。

## 11。查询缓存

我们还可以缓存 HQL 查询的结果。如果我们经常对很少改变的实体执行查询，这是很有用的。

为了启用查询缓存，我们将把`hibernate.cache.use_query_cache`属性的值设置为`true`:

```java
hibernate.cache.use_query_cache=true
```

对于每个查询，我们必须明确指出该查询是可缓存的(通过一个`org.hibernate.cacheable`查询提示):

```java
entityManager.createQuery("select f from Foo f")
  .setHint("org.hibernate.cacheable", true)
  .getResultList();
```

### 11.1。查询缓存最佳实践

这里有一些与查询缓存相关的指南和最佳实践:

*   与集合的情况一样，只缓存作为可缓存查询结果返回的实体的 id。因此，我们强烈建议为此类实体启用二级缓存。
*   对于每个查询，每个查询参数值(绑定变量)的组合都有一个缓存条目，所以我们期望有大量不同参数值组合的查询不适合缓存。
*   涉及数据库中频繁更改的实体类的查询也不适合缓存，因为只要有与参与查询的任何实体类相关的更改，这些查询就会失效，无论更改的实例是否作为查询结果的一部分被缓存。
*   默认情况下，所有查询缓存结果都存储在`org.hibernate.cache.internal.StandardQueryCache`区域。与实体/集合缓存一样，我们可以为这个区域定制缓存参数，以根据我们的需要定义驱逐和过期策略。对于每个查询，我们还可以指定一个定制的区域名称，以便为不同的查询提供不同的设置。
*   对于作为可缓存查询的一部分而被查询的所有表，Hibernate 在一个名为`org.hibernate.cache.spi.UpdateTimestampsCache`的单独区域中保存最后一次更新的时间戳。如果我们使用查询缓存，了解这个区域是非常重要的，因为 Hibernate 用它来验证缓存的查询结果没有过时。只要在查询结果区域中存在对应表的缓存查询结果，该缓存中的条目就不能被逐出/过期。最好关闭这个缓存区域的自动驱逐和过期，因为它不会消耗太多内存。

## 12。结论

在本文中，我们学习了如何设置 Hibernate 二级缓存。Hibernate 很容易配置和使用，使得二级缓存的使用对应用程序业务逻辑透明。

这篇文章的实现可以在 Github 的[上找到。这是一个基于 Maven 的项目，因此应该很容易导入和运行。](https://web.archive.org/web/20221215162710/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-hibernate-5)