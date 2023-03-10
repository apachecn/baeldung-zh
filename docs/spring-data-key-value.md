# Spring 数据键值指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-key-value>

## 1.介绍

Spring 数据键值框架使得编写使用键值存储的 Spring 应用程序变得容易。

它最大限度地减少了与商店交互所需的冗余任务和样板代码。该框架非常适合 Redis 和 Riak 这样的键值存储。

在本教程中，**我们将介绍如何在基于默认`java.util.Map`的实现中使用 Spring 数据键值。**

## 2.要求

Spring 数据键值 1.x 二进制文件需要 6.0 或更高版本，以及 Spring Framework 3.0.x 或更高版本。

## 3.Maven 依赖性

为了使用 Spring 数据键值，我们需要添加以下依赖项:

```java
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-keyvalue</artifactId>
    <version>2.0.6.RELEASE</version>
</dependency> 
```

最新版本可以在这里找到[。](https://web.archive.org/web/20220524015904/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.data%22%20AND%20a%3A%22spring-data-keyvalue%22)

## 4.创建实体

让我们创建一个`Employee`实体:

```java
@KeySpace("employees")
public class Employee {

    @Id
    private Integer id;

    private String name;

    private String department;

    private String salary;

    // constructors/ standard getters and setters

}
```

**`Keyspaces`定义实体应该保存在数据结构的哪个部分。**这个概念非常类似于 MongoDB 和 Elasticsearch 中的集合，Solr 中的核心和 JPA 中的表。

默认情况下，实体的键空间是从其类型中提取的。

## 5.贮藏室ˌ仓库

类似于其他 Spring 数据框架，我们将需要使用`@EnableMapRepositories`注释来激活 Spring 数据仓库。

默认情况下，存储库将使用基于`ConcurrentHashMap-`的实现:

```java
@SpringBootApplication
@EnableMapRepositories
public class SpringDataKeyValueApplication {
}
```

**可以改变默认的`ConcurrentHashMap`实现，使用一些其他的`java.util.Map`实现:**

```java
@EnableMapRepositories(mapType = WeakHashMap.class) 
```

使用 Spring 数据键值创建存储库的工作方式与其他 Spring 数据框架相同:

```java
@Repository
public interface EmployeeRepository
  extends CrudRepository<Employee, Integer> {
}
```

为了学习更多关于 Spring 数据仓库的知识，我们可以看看这篇文章。

## 6.使用存储库

通过在`EmployeeRepository`中扩展`CrudRepository`，我们得到了一整套执行 CRUD 功能的持久性方法。

现在，我们将看看如何使用一些可用的持久性方法。

### 6.1.保存对象

让我们使用存储库将新的`Employee`对象保存到数据存储中:

```java
Employee employee = new Employee(1, "Mike", "IT", "5000");
employeeRepository.save(employee);
```

### 6.2.检索现有对象

我们可以通过获取雇员来验证上一节中雇员的正确保存:

```java
Optional<Employee> savedEmployee = employeeRepository.findById(1); 
```

### 6.3.更新现有对象

`CrudRepository`没有提供更新对象的专用方法。

相反，我们可以使用`save()`方法:

```java
employee.setName("Jack");
employeeRepository.save(employee);
```

### 6.4.删除现有对象

我们可以使用存储库删除插入的对象:

```java
employeeRepository.deleteById(1); 
```

### 6.5.获取所有对象

我们可以获取所有保存的对象:

```java
Iterable<Employee> employees = employeeRepository.findAll();
```

## 7.`KeyValueTemplate`

对数据结构执行操作的另一种方式是使用`KeyValueTemplate`。

简单来说，`KeyValueTemplate`使用一个`MapAdapter`包装一个`java.util.Map`实现来执行查询和排序:

```java
@Bean
public KeyValueOperations keyValueTemplate() {
    return new KeyValueTemplate(keyValueAdapter());
}

@Bean
public KeyValueAdapter keyValueAdapter() {
    return new MapKeyValueAdapter(WeakHashMap.class);
} 
```

**注意，如果我们使用了`@EnableMapRepositories`，我们不需要指定一个`KeyValueTemplate.`，它将由框架自己创建。**

## 8.使用`KeyValueTemplate`

使用`KeyValueTemplate`，我们可以执行与存储库相同的操作。

### 8.1.保存对象

让我们看看如何使用模板将新的`Employee`对象保存到数据存储中:

```java
Employee employee = new Employee(1, "Mile", "IT", "5000");
keyValueTemplate.insert(employee); 
```

### 8.2.检索现有对象

我们可以通过使用模板从结构中获取对象来验证对象的插入:

```java
Optional<Employee> savedEmployee = keyValueTemplate
  .findById(id, Employee.class); 
```

### 8.3.更新现有对象

与`CrudRepository`不同，模板提供了更新对象的专用方法:

```java
employee.setName("Jacek");
keyValueTemplate.update(employee);
```

### 8.4.删除现有对象

我们可以用模板删除对象:

```java
keyValueTemplate.delete(id, Employee.class);
```

### 8.5.获取所有对象

我们可以使用模板获取所有保存的对象:

```java
Iterable<Employee> employees = keyValueTemplate
  .findAll(Employee.class);
```

### 8.6.分类对象

除了基本功能之外，**模板还支持`KeyValueQuery`来编写定制查询。**

例如，我们可以使用一个查询来获得一个基于薪水的排序列表:

```java
KeyValueQuery<Employee> query = new KeyValueQuery<Employee>();
query.setSort(new Sort(Sort.Direction.DESC, "salary"));
Iterable<Employee> employees 
  = keyValueTemplate.find(query, Employee.class);
```

## 9.结论

本文展示了如何通过使用`Repository`或`KeyValueTemplate`将 Spring 数据键值框架与默认的 Map 实现结合使用。

还有更多 Spring 数据框架，比如 Spring Data Redis，它们是在 Spring 数据键值之上编写的。关于 Spring Data Redis 的介绍，请参考[这篇文章](/web/20220524015904/https://www.baeldung.com/spring-data-redis-tutorial)。

和往常一样，这里展示的所有代码示例都可以在 GitHub 上找到[。](https://web.archive.org/web/20220524015904/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-keyvalue)