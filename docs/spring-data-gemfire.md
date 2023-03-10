# 带 Spring 数据的 GemFire 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-gemfire>

## 1。概述

[GemFire](https://web.archive.org/web/20220626203414/https://pivotal.io/pivotal-gemfire) 是一种高性能分布式数据管理基础设施，位于应用集群和后端数据源之间。

使用 GemFire，可以在内存中管理数据，这使得访问速度更快。 **Spring Data 提供了一个简单的配置和从 Spring 应用程序访问 GemFire 的方法。**

在本文中，我们将了解如何使用 GemFire 来满足应用程序的缓存需求。

## 2。Maven 依赖关系

为了利用 Spring 数据 GemFire 支持，我们首先需要在我们的`pom.xml:`中添加以下依赖项

```java
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-gemfire</artifactId>
    <version>1.9.1.RELEASE</version>
</dependency>
```

这个依赖关系的最新版本可以在[这里](https://web.archive.org/web/20220626203414/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.data%22%20AND%20a%3A%22spring-data-gemfire%22)找到。

## 3。GemFire 基本特性

### 3.1。缓存

GemFire 中的缓存提供基本的数据管理服务，并管理与其他对等设备的连接。

缓存配置(`cache.xml`)描述了数据将如何在不同的节点之间分布:

```java
<cache>
    <region name="region">
        <region-attributes>
            <cache-listener>
                <class-name>
                ...
                </class-name>
            </cache-listener>
        </region-attributes>
    </region>
    ...
</cache>
```

### 3.2。区域

数据区域是高速缓存中单个数据集的逻辑分组。

简而言之，**区域让我们可以将数据存储在系统中的多个虚拟机中**，而无需考虑数据存储在集群中的哪个节点。

区域分为三大类:

*   **复制区域**保存每个节点上的完整数据集。它提供了很高的读取性能。写操作较慢，因为数据更新需要传播到每个节点:

    ```java
    <region name="myRegion" refid="REPLICATE"/>
    ```

*   **分区**分发数据，使得每个节点只存储一部分区域内容。数据的副本存储在其他节点之一上。它提供了良好的写入性能。

    ```java
    <region name="myRegion" refid="PARTITION"/>
    ```

*   **本地区域**驻留在定义成员节点上。与群集中的其他节点没有连接。

    ```java
    <region name="myRegion" refid="LOCAL"/>
    ```

### 3.3。查询缓存

GemFire 提供了一种称为 OQL(对象查询语言)的查询语言，允许我们引用存储在 GemFire 数据区域中的对象。这在语法上非常类似于 SQL。让我们看看一个非常基本的查询是什么样子的:

SELECT DISTINCT * FROM example region

GemFire 的`QueryService`提供了创建查询对象的方法。

### 3.4。数据序列化

为了管理数据序列化-反序列化，GemFire 提供了除 Java 序列化之外的选项，这些选项提供了更高的性能，为数据存储和数据传输提供了更大的灵活性，还支持不同的语言。

考虑到这一点，GemFire 定义了可移植数据交换(PDX)数据格式。PDX 是一种跨语言的数据格式，它通过将数据存储在命名字段中来提供更快的序列化和反序列化，无需完全反序列化对象即可直接访问该字段。

### 3.5。功能执行

在 GemFire 中，函数可以驻留在服务器上，并且可以从客户端应用程序或另一个服务器调用，而无需发送函数代码本身。

调用者可以指示依赖于数据的函数对特定数据集进行操作，也可以引导独立的数据函数对特定的服务器、成员或成员组进行操作。

### 3.6。连续查询

对于连续查询，客户端通过使用 SQL 类型的查询过滤来订阅服务器端事件。服务器发送修改查询结果的所有事件。连续查询事件交付使用客户端/服务器订阅框架。

连续查询的语法类似于用 OQL 编写的基本查询。例如，提供来自`Stock`地区的最新股票数据的查询可以写成:

```java
SELECT * from StockRegion s where s.stockStatus='active';
```

为了从这个查询中获得状态更新，需要将`CQListener` 的实现与`StockRegion:`联系起来

```java
<cache>
    <region name="StockRegion>
        <region-attributes refid="REPLICATE">
            ...
            <cache-listener>
                <class-name>...</class-name>
            </cache-listener>
        ...
        </region-attributes>
    </region>
</cache>
```

## 4。Spring Data GemFire 支持

### 4.1。Java 配置

为了简化配置，Spring Data GemFire 为配置核心 GemFire 组件提供了各种注释:

```java
@Configuration
public class GemfireConfiguration {

    @Bean
    Properties gemfireProperties() {
        Properties gemfireProperties = new Properties();
        gemfireProperties.setProperty("name","SpringDataGemFireApplication");
        gemfireProperties.setProperty("mcast-port", "0");
        gemfireProperties.setProperty("log-level", "config");
        return gemfireProperties;
    }

    @Bean
    CacheFactoryBean gemfireCache() {
        CacheFactoryBean gemfireCache = new CacheFactoryBean();
        gemfireCache.setClose(true);
        gemfireCache.setProperties(gemfireProperties());
        return gemfireCache;
    }

    @Bean(name="employee")
    LocalRegionFactoryBean<String, Employee> getEmployee(final GemFireCache cache) {
        LocalRegionFactoryBean<String, Employee> employeeRegion = new LocalRegionFactoryBean();
        employeeRegion.setCache(cache);
        employeeRegion.setName("employee");
        // ...
        return employeeRegion;
    }
}
```

要设置 GemFire 缓存和区域，我们必须首先设置一些特定的属性。这里`mcast-port`被设置为零，这表示该 GemFire 节点被禁止用于多播发现和分发。这些属性然后被传递给`CacheFactoryBean`以创建一个`GemFireCache` 实例。

使用`GemFireCache` bean，创建了一个`LocalRegionFatcoryBean`实例，它代表了缓存中用于`Employee` 实例的区域。

### 4.2。实体映射

该库提供了对要存储在 GemFire grid 中的地图对象的支持。映射元数据通过在域类中使用注释来定义:

```java
@Region("employee")
public class Employee {

    @Id
    public String name;
    public double salary;

    @PersistenceConstructor
    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    // standard getters/setters
}
```

在上面的例子中，我们使用了以下注释:

*   **@** `**Region**,`指定`Employee` 类的区域实例
*   **`@Id,`** 标注要作为缓存键使用的属性
*   **`@PersistenceConstructor`** `,`这有助于在多个构造函数可用的情况下，标记将用于创建实体的一个构造函数

### 4.3。GemFire 仓库

接下来，让我们看看 Spring Data 的一个核心组件——存储库:

```java
@Configuration
@EnableGemfireRepositories(basePackages
  = "com.baeldung.spring.data.gemfire.repository")
public class GemfireConfiguration {

    @Autowired
    EmployeeRepository employeeRepository;

    // ...
}
```

### 4.4。Oql 查询支持

储存库允许定义查询方法，以针对受管实体映射到的区域高效地运行 OQL 查询:

```java
@Repository
public interface EmployeeRepository extends   
  CrudRepository<Employee, String> {

    Employee findByName(String name);

    Iterable<Employee> findBySalaryGreaterThan(double salary);

    Iterable<Employee> findBySalaryLessThan(double salary);

    Iterable<Employee> 
      findBySalaryGreaterThanAndSalaryLessThan(double salary1, double salary2);
}
```

### 4.5。功能执行支持

我们还提供注释支持，以简化 GemFire 函数的执行。

当我们使用函数时，有两个问题需要解决，实现和执行。

让我们看看如何使用 Spring 数据注释将 POJO 公开为 GemFire 函数:

```java
@Component
public class FunctionImpl {

    @GemfireFunction
    public void greeting(String message){
        // some logic
    }

    // ...
}
```

我们需要显式激活注释处理以使`@GemfireFunction`工作:

```java
@Configuration
@EnableGemfireFunctions
public class GemfireConfiguration {
    // ...
}
```

对于函数执行，调用远程函数的进程需要提供调用参数、函数`id`、执行目标(`onServer`、`onRegion`、`onMember`等。):

```java
@OnRegion(region="employee")
public interface FunctionExecution {

    @FunctionId("greeting")
    public void execute(String message);

    // ...
}
```

为了启用函数执行注释处理，我们需要添加使用 Spring 的组件扫描功能来激活它:

```java
@Configuration
@EnableGemfireFunctionExecutions(
  basePackages = "com.baeldung.spring.data.gemfire.function")
public class GemfireConfiguration {
    // ...
}
```

## 5。结论

在本文中，我们探索了 GemFire 的基本特性，并研究了 Spring Data 提供的 API 是如何使使用它变得容易的。

这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220626203414/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-gemfire)