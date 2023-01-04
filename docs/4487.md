# Apache 用 Spring 数据点燃

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-ignite-spring-data>

## 1。概述

在这个快速指南中，我们将重点关注如何将 Spring 数据 API 与 Apache Ignite 平台集成。

要了解 Apache Ignite，请查看我们之前的指南。

## 2。Maven 设置

除了现有的依赖项，我们还必须启用 Spring 数据支持:

```
<dependency>
    <groupId>org.apache.ignite</groupId>
    <artifactId>ignite-spring-data</artifactId>
    <version>${ignite.version}</version>
</dependency>
```

这个`ignite-spring-data`神器可以从 [Maven Central](https://web.archive.org/web/20220526043830/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.ignite%22%20AND%20a%3A%22ignite-spring-data%22) 下载。

## 3。模型和知识库

为了演示集成，我们将构建一个应用程序，通过使用 Spring 数据 API 将`employees`存储到 Ignite 的缓存中。

`EmployeeDTO`的 POJO 将如下所示:

```
public class EmployeeDTO implements Serializable {

    @QuerySqlField(index = true)
    private Integer id;

    @QuerySqlField(index = true)
    private String name;

    @QuerySqlField(index = true)
    private boolean isEmployed;

    // getters, setters
}
```

这里，`@QuerySqlField`注释支持使用 SQL 查询字段。

接下来，我们将创建存储库来保存`Employee`对象:

```
@RepositoryConfig(cacheName = "baeldungCache")
public interface EmployeeRepository 
  extends IgniteRepository<EmployeeDTO, Integer> {
    EmployeeDTO getEmployeeDTOById(Integer id);
}
```

**Apache Ignite 使用自己的`IgniteRepository`，它从 Spring Data 的`CrudRepository`扩展而来。**它还支持从 Spring 数据访问 SQL 网格。

这支持标准的 CRUD 方法，除了一些不需要 id 的方法。我们将在测试部分更详细地了解原因。

**`The @RepositoryConfig`标注将`EmployeeRepository`映射到点燃的`baeldungCache`。**

## 4。弹簧配置

现在让我们创建我们的 Spring 配置类。

**我们将使用`@EnableIgniteRepositories`注释来添加对 Ignite 存储库的支持:**

```
@Configuration
@EnableIgniteRepositories
public class SpringDataConfig {

    @Bean
    public Ignite igniteInstance() {
        IgniteConfiguration config = new IgniteConfiguration();

        CacheConfiguration cache = new CacheConfiguration("baeldungCache");
        cache.setIndexedTypes(Integer.class, EmployeeDTO.class);

        config.setCacheConfiguration(cache);
        return Ignition.start(config);
    }
}
```

这里，`igniteInstance()`方法创建了`Ignite`实例并将其传递给`IgniteRepositoryFactoryBean`,以便访问 Apache Ignite 集群。

我们还定义并设置了`baeldungCache`配置。`setIndexedTypes()`方法为缓存设置 SQL 模式。

## 5。测试存储库

为了测试应用程序，让我们在应用程序上下文中注册`SpringDataConfiguration`并从中获取`EmployeeRepository`:

```
AnnotationConfigApplicationContext context
 = new AnnotationConfigApplicationContext();
context.register(SpringDataConfig.class);
context.refresh();

EmployeeRepository repository = context.getBean(EmployeeRepository.class);
```

然后，我们想要创建`EmployeeDTO`实例并将其保存在缓存中:

```
EmployeeDTO employeeDTO = new EmployeeDTO();
employeeDTO.setId(1);
employeeDTO.setName("John");
employeeDTO.setEmployed(true);

repository.save(employeeDTO.getId(), employeeDTO);
```

这里我们使用了`IgniteRepository`的`save` `(key, value)`方法。原因是**标准的`CrudRepository save(entity), save(entities), delete(entity)` 操作还不被支持`.`**

这背后的问题是由`CrudRepository.save()`方法生成的 id 在集群中不是唯一的。

相反，我们必须使用`save` `(key, value), save(Map<ID, Entity> values), deleteAll(Iterable<ID> ids)` 方法。

之后，我们可以使用 Spring Data 的`getEmployeeDTOById()`方法从缓存中获取 employee 对象:

```
EmployeeDTO employee = repository.getEmployeeDTOById(employeeDTO.getId());
System.out.println(employee);
```

输出显示我们成功获取了初始对象:

```
EmployeeDTO{id=1, name='John', isEmployed=true}
```

或者，我们可以使用`IgniteCache` API 检索相同的对象:

```
IgniteCache<Integer, EmployeeDTO> cache = ignite.cache("baeldungCache");
EmployeeDTO employeeDTO = cache.get(employeeId);
```

或者使用标准的 SQL:

```
SqlFieldsQuery sql = new SqlFieldsQuery(
  "select * from EmployeeDTO where isEmployed = 'true'");
```

## 6。总结

这个简短的教程展示了如何将 Spring 数据框架与 Apache Ignite 项目集成在一起。在实际例子的帮助下，**我们通过使用 Spring 数据 API 学习了如何使用 Apache Ignite 缓存。**

和往常一样，本文的完整代码可以在 [GitHub 项目](https://web.archive.org/web/20220526043830/https://github.com/eugenp/tutorials/tree/master/libraries-data/)中找到。