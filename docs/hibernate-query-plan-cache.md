# Hibernate 查询计划缓存

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-query-plan-cache>

## 1.介绍

在这个快速教程中，我们将探索 Hibernate 提供的查询计划缓存及其对性能的影响。

## 2.查询计划缓存

每个 JPQL 查询或标准查询在执行之前都被解析成抽象语法树(AST ),以便 Hibernate 可以生成 SQL 语句。由于查询编译需要时间，Hibernate 提供了一个 [QueryPlanCache](https://web.archive.org/web/20220523231156/https://docs.jboss.org/hibernate/orm/5.0/javadocs/org/hibernate/engine/query/spi/QueryPlanCache.html) 来获得更好的性能。

对于本地查询，Hibernate 提取关于命名参数和查询返回类型的信息，并将其存储在`[ParameterMetadata](https://web.archive.org/web/20220523231156/https://docs.jboss.org/hibernate/orm/5.0/javadocs/org/hibernate/engine/query/spi/ParameterMetadata.html)`中。

对于每次执行，Hibernate 首先检查计划缓存，只有当没有可用的计划时，它才会生成一个新的计划，并将执行计划存储在缓存中以供将来参考。

## 3.配置

查询计划缓存配置由以下属性控制:

*   `hibernate.query.plan_cache_max_size`–控制计划缓存中条目的最大数量(默认为 2048)
*   `hibernate.query.plan_parameter_metadata_max_size`–管理缓存中`ParameterMetadata`实例的数量(默认为 128)

因此，如果我们的应用程序执行的查询超过了查询计划缓存的大小，Hibernate 将不得不花费额外的时间来编译查询。因此，总的查询执行时间将会增加。

## 4.设置测试用例

业内有句话说，说到业绩，千万不要相信那些说法。因此，**让我们测试一下当我们改变缓存设置**时，查询编译时间是如何变化的。

### 4.1.测试中涉及的实体类

让我们先来看看我们将在示例中使用的实体，`DeptEmployee`和`Department`:

```java
@Entity
public class DeptEmployee {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private long id;

    private String employeeNumber;

    private String title;

    private String name;

    @ManyToOne
    private Department department;

   // standard getters and setters
}
```

```java
@Entity
public class Department {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private long id;

    private String name;

    @OneToMany(mappedBy="department")
    private List<DeptEmployee> employees;

    // standard getters and setters
} 
```

### 4.2.测试中涉及的 Hibernate 查询

我们只对测量总的查询编译时间感兴趣，因此，我们可以为我们的测试选择任何有效的 HQL 查询组合。

出于本文的目的，我们将使用以下三个查询:

*   ` **findEmployeesByDepartmentName**`

```java
session.createQuery("SELECT e FROM DeptEmployee e " +
  "JOIN e.department WHERE e.department.name = :deptName")
  .setMaxResults(30)
  .setHint(QueryHints.HINT_FETCH_SIZE, 30);
```

*   `**findEmployeesByDesignation**`

```java
session.createQuery("SELECT e FROM DeptEmployee e " +
  "WHERE e.title = :designation")
  .setHint(QueryHints.SPEC_HINT_TIMEOUT, 1000);
```

*   `**findDepartmentOfAnEmployee**`

```java
session.createQuery("SELECT e.department FROM DeptEmployee e " +
  "JOIN e.department WHERE e.employeeNumber = :empId");
```

## 5.衡量绩效影响

### 5.1.基准代码设置

**我们将改变缓存大小，从 1 到 3**–之后，我们的所有三个查询都将在缓存中。因此，没有必要进一步增加它:

```java
@State(Scope.Thread)
public static class QueryPlanCacheBenchMarkState {
    @Param({"1", "2", "3"})
    public int planCacheSize;

    public Session session;

    @Setup
    public void stateSetup() throws IOException {
       session = initSession(planCacheSize);
    }

    private Session initSession(int planCacheSize) throws IOException {
        Properties properties = HibernateUtil.getProperties();
        properties.put("hibernate.query.plan_cache_max_size", planCacheSize);
        properties.put("hibernate.query.plan_parameter_metadata_max_size", planCacheSize);
        SessionFactory sessionFactory = HibernateUtil.getSessionFactoryByProperties(properties);
        return sessionFactory.openSession();
    }
    //teardown...
}
```

### 5.2.测试中的代码

接下来，让我们看一下用来测量 Hibernate 编译查询所用平均时间的基准代码:

```java
@Benchmark
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
@Fork(1)
@Warmup(iterations = 2)
@Measurement(iterations = 5)
public void givenQueryPlanCacheSize_thenCompileQueries(
  QueryPlanCacheBenchMarkState state, Blackhole blackhole) {

    Query query1 = findEmployeesByDepartmentNameQuery(state.session);
    Query query2 = findEmployeesByDesignationQuery(state.session);
    Query query3 = findDepartmentOfAnEmployeeQuery(state.session);

    blackhole.consume(query1);
    blackhole.consume(query2);
    blackhole.consume(query3);
}
```

请注意，我们使用了 [JMH](/web/20220523231156/https://www.baeldung.com/java-microbenchmark-harness) 来编写我们的基准。

### 5.3.基准测试结果

现在，让我们来看一下通过运行上述基准测试准备的编译时间与缓存大小的关系图:

[![](img/8d93905bfe2c6cf59e2155e879077071.png)](/web/20220523231156/https://www.baeldung.com/wp-content/uploads/2019/02/plan-cache.png)

从图中我们可以清楚地看到，**增加 Hibernate 允许缓存的查询数量会减少编译时间**。

对于大小为 1 的高速缓存，平均编译时间最高为 709 微秒，然后对于大小为 2 的高速缓存，平均编译时间减少到 409 微秒，对于大小为 3 的高速缓存，平均编译时间一直减少到 0.637 微秒。

## 6.使用休眠统计

为了监控查询计划缓存的有效性，Hibernate 通过`[Statistics](https://web.archive.org/web/20220523231156/http://docs.jboss.org/hibernate/orm/5.0/javadocs/org/hibernate/stat/Statistics.html)`接口公开了以下方法:

*   `getQueryPlanCacheHitCount`
*   `getQueryPlanCacheMissCount`

因此，如果命中数很高而未命中数很低，那么大多数查询都是由缓存本身提供的，而不是一遍又一遍地编译。

## 7.结论

在本文中，我们了解了 Hibernate 中的查询计划缓存是什么，以及它如何影响应用程序的整体性能。总的来说，我们应该根据应用程序中运行的查询数量来保持查询计划缓存的大小。

和往常一样，本教程的源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220523231156/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-queries)