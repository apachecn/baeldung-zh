# 将 JDBI 与 Spring Boot 一起使用

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-jdbi>

## 1.介绍

在之前的教程中，我们介绍了 [JDBI](https://web.archive.org/web/20221205142150/http://jdbi.org/) 、**的基础知识，这是一个用于关系数据库访问的开源库**，它删除了许多与直接使用 JDBC 相关的样板代码。

这次，我们将看看如何在 Spring Boot 应用程序中使用 JDBI。我们还将讨论这个库的一些方面，这些方面使它在某些场景中成为 Spring Data JPA 的一个很好的替代品。

## 2.项目设置

首先，让我们将适当的 JDBI 依赖项添加到项目中。**这一次，我们将使用 JDBI 的 Spring 集成插件，它带来了所有需要的核心依赖项**。我们还将引入 SqlObject 插件，它为我们将在示例中使用的基本 JDBI 添加了一些额外的特性:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
    <version>2.1.8.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.jdbi</groupId>
    <artifactId>jdbi3-spring4</artifactId>
    <version>3.9.1</version>
</dependency>
<dependency>
    <groupId>org.jdbi</groupId>
    <artifactId>jdbi3-sqlobject</artifactId>
    <version>3.9.1</version> 
</dependency>
```

这些工件的最新版本可以在 Maven Central 中找到:

*   [Spring Boot 开始 JDBC](https://web.archive.org/web/20221205142150/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-jdbc%22)
*   [JDBI Spring 集成](https://web.archive.org/web/20221205142150/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.jdbi%22%20AND%20a%3A%22jdbi3-spring4%22)
*   [JDBI SqlObject 插件](https://web.archive.org/web/20221205142150/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.jdbi%22%20AND%20a%3A%22jdbi3-sqlobject%22)

我们还需要一个合适的 JDBC 驱动程序来访问我们的数据库。在本文中，我们将使用 [H2](https://web.archive.org/web/20221205142150/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.h2database%22%20AND%20a%3A%22h2%22) ，因此我们也必须将它的驱动程序添加到依赖列表中:

```java
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.199</version>
    <scope>runtime</scope>
</dependency>
```

## 3.JDBI 实例化和配置

我们在上一篇文章中已经看到，我们需要一个`Jdbi`实例作为访问 JDBI 的 API 的入口点。因为我们在 Spring 世界中，所以让这个类的实例作为 bean 可用是有意义的。

我们将利用 Spring Boot 的自动配置能力来初始化一个`DataSource`，并将其传递给一个带 `@Bean`注释的方法，该方法将创建我们的全局`Jdbi`实例。

我们还将把任何发现的插件和`RowMapper`实例传递给这个方法，以便它们被预先注册:

```java
@Configuration
public class JdbiConfiguration {
    @Bean
    public Jdbi jdbi(DataSource ds, List<JdbiPlugin> jdbiPlugins, List<RowMapper<?>> rowMappers) {        
        TransactionAwareDataSourceProxy proxy = new TransactionAwareDataSourceProxy(ds);        
        Jdbi jdbi = Jdbi.create(proxy);
        jdbiPlugins.forEach(plugin -> jdbi.installPlugin(plugin));
        rowMappers.forEach(mapper -> jdbi.registerRowMapper(mapper));       
        return jdbi;
    }
}
```

这里，我们使用一个可用的`DataSource`并将它包装在一个`TransactionAwareDataSourceProxy`中。**我们需要这个包装器，以便将 Spring 管理的事务与 JDBI** 集成，我们将在后面看到。

注册插件和 RowMapper 实例非常简单。我们所要做的就是分别为每个可用的`JdbiPlugin`和`RowMapper, `调用`installPlugin`和`installRowMapper`。之后，我们有了一个完全配置好的`Jdbi`实例，可以在我们的应用程序中使用。

## 4.样本域

我们的例子使用了一个非常简单的领域模型，只包含两个类:`CarMaker`和`CarModel`。由于 JDBI 不需要对我们的域类进行任何注释，我们可以使用简单的 POJOs:

```java
public class CarMaker {
    private Long id;
    private String name;
    private List<CarModel> models;
    // getters and setters ...
}

public class CarModel {
    private Long id;
    private String name;
    private Integer year;
    private String sku;
    private Long makerId;
    // getters and setters ...
} 
```

## 5.创建 Dao

现在，让我们为领域类创建数据访问对象(Dao)。JDBI SqlObject 插件提供了一种实现这些类的简单方法，类似于 Spring Data 处理这个问题的方式。

我们只需要定义一个带有一些注释的接口，自动地， **JDBI 将处理所有底层的东西，比如处理 JDBC 连接，创建/处理语句和`ResultSet` s** :

```java
@UseClasspathSqlLocator
public interface CarMakerDao {
    @SqlUpdate
    @GetGeneratedKeys
    Long insert(@BindBean CarMaker carMaker);

    @SqlBatch("insert")
    @GetGeneratedKeys
    List<Long> bulkInsert(@BindBean List<CarMaker> carMakers);

    @SqlQuery
    CarMaker findById(Long id);
}

@UseClasspathSqlLocator
public interface CarModelDao {    
    @SqlUpdate
    @GetGeneratedKeys
    Long insert(@BindBean CarModel carModel);

    @SqlBatch("insert")
    @GetGeneratedKeys
    List<Long> bulkInsert(@BindBean List<CarModel> models);

    @SqlQuery
    CarModel findByMakerIdAndSku(@Bind("makerId") Long makerId, @Bind("sku") String sku );
}
```

这些接口都有大量注释，所以让我们快速浏览一下它们。

### 5.1.`@UseClasspathSqlLocator`

**@`UseClasspathSqlLocator` 注释告诉 JDBI，与每个方法相关联的实际 SQL 语句位于外部资源文件**。默认情况下，JDBI 将使用接口的完全限定名和方法来查找资源。例如，给定一个带有`findById()`方法的`a.b.c.Foo` 接口的 FQN，JDBI 将寻找一个名为`a/b/c/Foo/findById.sql.`的资源

对于任何给定的方法，可以通过将资源名称作为值传递给`@SqlXXX`注释来覆盖这个默认行为。

### `5.2\. @SqlUpdate/@SqlBatch/@SqlQuery`

**我们使用`@SqlUpdate`、`@SqlBatch`和`@SqlQuery`注释来标记数据访问方法，这些方法将使用给定的参数**来执行。这些注释可以接受一个可选的字符串值，该值将是要执行的 SQL 语句(包括任何命名的参数),或者当与`@UseClasspathSqlLocator`一起使用时，是包含它的资源名。

`@SqlBatch`-带注释的方法可以有类似集合的参数，并在单个批处理语句中为每个可用项执行相同的 SQL 语句。在上面的每个 DAO 类中，我们都有一个`bulkInsert `方法来说明它的用法。使用批处理语句的主要优点是在处理大型数据集时可以获得更高的性能。

### `5.3\. @GetGeneratedKeys`

顾名思义，**`@GetGeneratedKeys`注释允许我们恢复成功执行**后生成的任何密钥。它主要用在`insert`语句中，我们的数据库会自动生成新的标识符，我们需要在代码中恢复它们。

### `5.4\. @BindBean/@Bind`

**我们使用`@BindBean`和`@Bind`注释将 SQL 语句中的命名参数与方法参数**绑定。`@BindBean`使用标准的 bean 约定从 POJO 中提取属性——包括嵌套属性。`@Bind`使用参数名或提供的值将其值映射到命名参数。

## 6.使用 DAOs

为了在我们的应用程序中使用这些 Dao，我们必须使用 JDBI 中可用的工厂方法之一来实例化它们。

在 Spring 上下文中，最简单的方法是使用`onDemand`方法为每个 DAO 创建一个 bean:

```java
@Bean
public CarMakerDao carMakerDao(Jdbi jdbi) {        
    return jdbi.onDemand(CarMakerDao.class);       
}

@Bean
public CarModelDao carModelDao(Jdbi jdbi) {
    return jdbi.onDemand(CarModelDao.class);
} 
```

**`onDemand`创建的实例是线程安全的，只在方法调用**期间使用数据库连接。由于 JDBI 我们将使用提供的`TransactionAwareDataSourceProxy,` **，这意味着我们可以将它无缝地用于 Spring 管理的事务**。

虽然简单，但当我们必须处理多个表时，我们在这里使用的方法远非理想。避免编写这种样板代码的一种方法是创建一个自定义的`BeanFactory. `来描述如何实现这样一个组件，但是这超出了本教程的范围。

## 7.交易服务

让我们在一个简单的服务类中使用我们的 DAO 类，该服务类创建几个`CarModel`实例，给定一个用模型填充的`CarMaker`。首先，我们将检查给定的`CarMaker`以前是否被保存过，如果需要，将它保存到数据库中。然后，我们将逐一插入每一个`CarModel`。

**如果在任何时候出现唯一键违规(或其他错误),整个操作肯定会失败，应该执行完全回滚**。

JDBI 提供了一个`@Transaction`注释**，但是我们不能在这里使用它**，因为它不知道可能参与同一业务事务的其他资源。相反，我们将在我们的服务方法中使用 Spring 的`@Transactional`注释:

```java
@Service
public class CarMakerService {

    private CarMakerDao carMakerDao;
    private CarModelDao carModelDao;

    public CarMakerService(CarMakerDao carMakerDao,CarModelDao carModelDao) {        
        this.carMakerDao = carMakerDao;
        this.carModelDao = carModelDao;
    }    

    @Transactional
    public int bulkInsert(CarMaker carMaker) {
        Long carMakerId;
        if (carMaker.getId() == null ) {
            carMakerId = carMakerDao.insert(carMaker);
            carMaker.setId(carMakerId);
        }
        carMaker.getModels().forEach(m -> {
            m.setMakerId(carMaker.getId());
            carModelDao.insert(m);
        });                
        return carMaker.getModels().size();
    }
} 
```

该操作的实现本身非常简单:我们使用标准约定，即`id`字段中的`null`值意味着该实体尚未被持久化到数据库中。如果是这种情况，我们使用构造函数中注入的`CarMakerDao`实例在数据库中插入一条新记录，并获得生成的`id.`

一旦我们有了`CarMaker`的 id，我们迭代模型，在保存到数据库之前为每个模型设置`makerId `字段。

**所有这些数据库操作将使用相同的底层连接进行，并将成为相同事务**的一部分。这里的技巧在于我们使用`TransactionAwareDataSourceProxy`和创建`onDemand`Dao 将 JDBI 绑定到 Spring 的方式。当 JDBI 请求一个新的`Connection`时，它将获得一个与当前事务相关联的现有的【】，从而将其生命周期集成到其他可能注册的资源中。

## 8.结论

在本文中，我们展示了如何快速将 JDBI 集成到 Spring Boot 应用程序中。在我们由于某种原因不能使用 Spring Data JPA，但仍然希望使用所有其他功能(如事务管理、集成等)的场景中，这是一个强大的组合。

像往常一样，所有代码都可以在 GitHub 上获得[。](https://web.archive.org/web/20221205142150/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-2)