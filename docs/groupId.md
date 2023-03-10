# Spring Data Redis 简介

> 原文:# t0]https://web . archive . org/web/202209930061024/https://www . BAE message . com/spring-data-redis-tutorial

## 1。概述

本教程是**Spring Data Redis**的介绍，它为 [Redis](https://web.archive.org/web/20220926182159/http://redis.io/) 提供了 Spring 数据平台的抽象——流行的内存数据结构存储。

Redis 由基于 keystore 的数据结构驱动来持久化数据，可以用作数据库、缓存、消息代理等。

我们将能够使用 Spring 数据的通用模式(模板等。)同时还具有所有 Spring 数据项目的传统简单性。

## 2。Maven 依赖关系

让我们从在`pom.xml`中声明 Spring 数据 Redis 依赖关系开始:

```java
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>2.3.3.RELEASE</version>
 </dependency>

<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.3.0</version>
    <type>jar</type>
</dependency>
```

最新版本的[`spring-data-redis`](https://web.archive.org/web/20220926182159/https://search.maven.org/classic/#search%7Cga%7C1%7Cspring-data-redis)`[jedis](https://web.archive.org/web/20220926182159/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22jedis%22%20AND%20g%3A%22redis.clients%22)`可以从 Maven Central 下载。

或者，我们可以使用 Redis 的 Spring Boot 启动器，这将消除对单独的`spring-data`和`jedis`依赖项的需要:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <version>2.7.2</version>
</dependency>
```

同样， [Maven Central](https://web.archive.org/web/20220926182159/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-redis%22) 提供最新的版本信息。

## 3 .The Redis Configuration

为了定义应用程序客户机和 Redis 服务器实例之间的连接设置，我们需要使用 Redis 客户机。

有许多 Redis 客户机实现可用于 Java。在本教程中，**我们将使用[Jedis](https://web.archive.org/web/20220926182159/https://github.com/xetorthio/jedis)——一个简单而强大的 Redis 客户端实现。**

该框架对 XML 和 Java 配置都有很好的支持。对于本教程，我们将使用基于 Java 的配置。

### 3.1。Java 配置

让我们从配置 bean 定义开始: 

```java
@Bean
JedisConnectionFactory jedisConnectionFactory() {
    return new JedisConnectionFactory();
}

@Bean
public RedisTemplate<String, Object> redisTemplate() {
    RedisTemplate<String, Object> template = new RedisTemplate<>();
    template.setConnectionFactory(jedisConnectionFactory());
    return template;
} 
```

配置相当简单。

首先，使用 Jedis 客户端，我们定义一个`connectionFactory`。

然后我们使用`jedisConnectionFactory`定义了一个`RedisTemplate`。这可用于通过自定义存储库查询数据。

### 3.2。自定义连接属性

请注意，在上面的配置中缺少了与连接相关的常见属性。例如，配置中缺少服务器地址和端口。原因很简单:我们正在使用默认值。

但是，如果我们需要配置连接细节，我们总是可以修改`jedisConnectionFactory`配置:

```java
@Bean
JedisConnectionFactory jedisConnectionFactory() {
    JedisConnectionFactory jedisConFactory
      = new JedisConnectionFactory();
    jedisConFactory.setHostName("localhost");
    jedisConFactory.setPort(6379);
    return jedisConFactory;
}
```

## 4 .Redis 存储库

让我们使用一个`Student`实体:

```java
@RedisHash("Student")
public class Student implements Serializable {

    public enum Gender { 
        MALE, FEMALE
    }

    private String id;
    private String name;
    private Gender gender;
    private int grade;
    // ...
}
```

### 4.1。春天数据仓库

现在让我们创建`StudentRepository`:

```java
@Repository
public interface StudentRepository extends CrudRepository<Student, String> {}
```

## 5。数据访问使用`StudentRepository`

**通过在`StudentRepository`中扩展`CrudRepository` ，我们自动获得了一整套执行 CRUD 功能的持久化方法。**

### 5.1。保存新的学生对象

让我们在数据存储中保存一个新的学生对象:

```java
Student student = new Student(
  "Eng2015001", "John Doe", Student.Gender.MALE, 1);
studentRepository.save(student);
```

### 5.2。检索现有的学生对象

我们可以通过获取学生数据来验证前面部分中学生的正确插入:

```java
Student retrievedStudent = 
  studentRepository.findById("Eng2015001").get();
```

### 5.3。更新现有的学生对象

让我们更改上面检索到的学生姓名并再次保存:

```java
retrievedStudent.setName("Richard Watson");
studentRepository.save(student);
```

最后，我们可以再次检索学生的数据，并验证数据存储中的姓名是否已更新。

### 5.4。删除现有学生数据

我们可以删除插入的学生数据:

```java
studentRepository.deleteById(student.getId());
```

现在我们可以搜索学生对象，并验证结果是`null`。

### 5.5。查找所有学生数据

我们可以插入一些学生对象:

```java
Student engStudent = new Student(
  "Eng2015001", "John Doe", Student.Gender.MALE, 1);
Student medStudent = new Student(
  "Med2015001", "Gareth Houston", Student.Gender.MALE, 2);
studentRepository.save(engStudent);
studentRepository.save(medStudent);
```

我们也可以通过插入一个集合来实现这一点。为此，有一个不同的方法——`saveAll()`——接受单个*可迭代*对象，该对象包含多个我们想要持久化的学生对象。

要查找所有插入的学生，我们可以使用`findAll()`方法:

```java
List<Student> students = new ArrayList<>();
studentRepository.findAll().forEach(students::add);
```

然后我们可以快速检查 `students`列表的大小，或者通过检查每个对象的属性进行更大粒度的验证。

## 6。结论

在本文中，我们介绍了 Spring Data Redis 的基础知识。

上面例子的源代码可以在[GitHub 项目](https://web.archive.org/web/20220926182159/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-redis)中找到。