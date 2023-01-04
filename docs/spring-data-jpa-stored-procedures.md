# 从 Spring Data JPA 存储库中调用存储过程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jpa-stored-procedures>

## 1.概观

[存储过程](/web/20220617075734/https://www.baeldung.com/jpa-stored-procedures)是存储在数据库中的一组预定义的 SQL 语句。在 Java 中，有几种方法可以访问存储过程。在本教程中，我们将学习如何从 Spring Data JPA 存储库中调用存储过程。

## 2.项目设置

**我们将使用 [Spring Boot 启动器数据 JPA](/web/20220617075734/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) 模块作为数据访问层**。我们还将使用 MySQL 作为我们的后端数据库。因此，我们将需要项目`pom.xml`文件中的 [Spring 数据 JPA](https://web.archive.org/web/20220617075734/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-data-jpa) 、 [Spring 数据 JDBC](https://web.archive.org/web/20220617075734/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-data-jdbc) 和 [MySQL 连接器](https://web.archive.org/web/20220617075734/https://search.maven.org/search?q=g:mysql%20AND%20a:mysql-connector-java)依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency> 
```

一旦我们有了 MySQL 依赖项定义，我们就可以在`application.properties`文件中配置数据库连接:

```java
spring.datasource.url=jdbc:mysql://localhost:3306/baeldung
spring.datasource.username=baeldung
spring.datasource.password=baeldung
```

## 3.实体类

**在 Spring Data JPA 中，[实体](/web/20220617075734/https://www.baeldung.com/jpa-entities)表示存储在数据库中的一个表。**因此，我们可以构造一个实体类来映射`car`数据库表:

```java
@Entity
public class Car {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column
    private long id;

    @Column
    private String model;

    @Column
    private Integer year;

   // standard getters and setters
}
```

## 4.存储过程创建

一个存储过程可以有参数，这样我们可以根据输入得到不同的结果。例如，我们可以创建一个存储过程，它接受整数类型的输入参数并返回汽车列表:

```java
CREATE PROCEDURE FIND_CARS_AFTER_YEAR(IN year_in INT)
BEGIN 
    SELECT * FROM car WHERE year >= year_in ORDER BY year;
END
```

存储过程**也可以使用输出参数将数据**返回给调用应用程序。例如，我们可以创建一个存储过程，它接受字符串类型的输入参数，并将查询结果存储到输出参数中:

```java
CREATE PROCEDURE GET_TOTAL_CARS_BY_MODEL(IN model_in VARCHAR(50), OUT count_out INT)
BEGIN
    SELECT COUNT(*) into count_out from car WHERE model = model_in;
END
```

## 5.引用存储库中的存储过程

在 Spring Data JPA 中，存储库是我们提供数据库操作的地方。我们可以为`Car`实体上的数据库操作构建一个存储库，并引用这个存储库中的存储过程:

```java
@Repository
public interface CarRepository extends JpaRepository<Car, Integer> {
    // ...
} 
```

接下来，让我们将一些调用存储过程的方法添加到我们的存储库中。

### 5.1.直接映射存储过程名

**我们可以使用`@Procedure` 注释定义一个存储过程方法，并直接映射存储过程名称。**

有四种等效的方法可以做到这一点。例如，我们可以直接使用存储过程名作为方法名:

```java
@Procedure
int GET_TOTAL_CARS_BY_MODEL(String model); 
```

如果我们想定义一个不同的方法名，我们可以把存储过程名作为元素的`@Procedure` 注释:

```java
@Procedure("GET_TOTAL_CARS_BY_MODEL")
int getTotalCarsByModel(String model); 
```

**我们还可以使用`procedureName`属性来映射存储过程名:**

```java
@Procedure(procedureName = "GET_TOTAL_CARS_BY_MODEL")
int getTotalCarsByModelProcedureName(String model); 
```

最后，我们可以使用`value`属性来映射存储过程名:

```java
@Procedure(value = "GET_TOTAL_CARS_BY_MODEL")
int getTotalCarsByModelValue(String model); 
```

### 5.2.引用实体中定义的存储过程

**我们还可以使用`@NamedStoredProcedureQuery`注释在实体类中定义一个存储过程:**

```java
@Entity
@NamedStoredProcedureQuery(name = "Car.getTotalCardsbyModelEntity", 
  procedureName = "GET_TOTAL_CARS_BY_MODEL", parameters = {
    @StoredProcedureParameter(mode = ParameterMode.IN, name = "model_in", type = String.class),
    @StoredProcedureParameter(mode = ParameterMode.OUT, name = "count_out", type = Integer.class)})
public class Car {
    // class definition
}
```

然后我们可以在存储库中引用这个定义:

```java
@Procedure(name = "Car.getTotalCardsbyModelEntity")
int getTotalCarsByModelEntiy(@Param("model_in") String model); 
```

**我们使用`name`属性来引用实体类中定义的存储过程。**对于存储库方法，我们使用`@Param `来匹配存储过程的输入参数。我们还将存储过程的输出参数与存储库方法的返回值进行匹配。

### 5.3.用`@Query`注释引用存储过程

我们也可以用`@Query`注释直接调用存储过程:

```java
@Query(value = "CALL FIND_CARS_AFTER_YEAR(:year_in);", nativeQuery = true)
List<Car> findCarsAfterYear(@Param("year_in") Integer year_in);
```

在这个方法中，我们使用一个本地查询来调用存储过程。我们将查询存储在注释的`value`属性中。

类似地，我们使用`@Param `来匹配存储过程的输入参数。我们还将存储过程输出映射到实体`Car`对象列表。

## 6.摘要

在本文中，我们探讨了如何通过 JPA 存储库访问存储过程。我们还讨论了引用 JPA 存储库中存储过程的两种简单方法。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220617075734/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-repo)