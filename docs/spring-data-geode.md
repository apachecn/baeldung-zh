# Spring 数据 Geode 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-geode>

## 1.概观

[Apache Geode](/web/20220627173722/https://www.baeldung.com/apache-geode) 通过分布式云架构提供数据管理解决方案。利用[Spring Data](/web/20220627173722/https://www.baeldung.com/spring-data)API 通过 Apache Geode 服务器进行数据访问是最理想的。

在本教程中，我们将探索用于配置和开发 Apache Geode Java 客户端应用程序的 [Spring Data Geode](https://web.archive.org/web/20220627173722/https://spring.io/projects/spring-data-geode) 。

## 2.Spring 数据 Geode

Spring Data Geode 库使 Java 应用程序能够通过 XML 和注释来配置 Apache Geode 服务器。同时，该库也便于创建 Apache Geode 缓存客户端-服务器应用程序。

Spring Data Geode 库类似于 [Spring Data Gemfire](/web/20220627173722/https://www.baeldung.com/spring-data-gemfire) 。除了[的细微差异](https://web.archive.org/web/20220627173722/https://spring.io/blog/2017/10/26/diff-q-spring-data-gemfire-spring-data-geode)，后者提供了与 [Pivotal Gemfire](https://web.archive.org/web/20220627173722/https://pivotal.io/pivotal-gemfire) 的集成，后者是阿帕奇 Geode 的商业版本。

在此过程中，我们将探索一些 Spring 数据 Geode 注释，以将 Java 应用程序配置到 Apache Geode 的缓存客户端中。

## 3.Maven 依赖性

让我们将最新的 [`spring-geode-starter`](https://web.archive.org/web/20220627173722/https://search.maven.org/search?q=g:org.springframework.geode%20a:spring-geode-starter) 依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.springframework.geode</groupId>
    <artifactId>spring-geode-starter</artifactId>
    <version>1.1.1.RELEASE</version>
</dependency>
```

## 4.阿帕奇 Geode 与 Spring Boot 的`@ClientCacheApplication`

首先，让我们使用`@SpringBootApplication`创建一个 Spring Boot `ClientCacheApp`:

```java
@SpringBootApplication 
public class ClientCacheApp {
    public static void main(String[] args) {
        SpringApplication.run(ClientCacheApp.class, args); 
    } 
}
```

然后，为了将`ClientCacheApp`类转换成 Apache Geode 缓存客户端，我们将添加 Spring 数据 Geode 提供的 [`@ClientCacheApplication`](https://web.archive.org/web/20220627173722/https://docs.spring.io/spring-data/geode/docs/current/api/org/springframework/data/gemfire/config/annotation/ClientCacheApplication.html) :

```java
@ClientCacheApplication
// existing annotations
public class ClientCacheApp {
    // ...
}
```

就是这样！缓存客户端应用程序已准备好运行。

然而，在启动我们的应用程序之前，我们需要启动 Apache Geode 服务器。

## 5.启动 Apache Geode 服务器

假设 Apache Geode 和 [`gfsh`](https://web.archive.org/web/20220627173722/https://geode.apache.org/docs/guide/19/tools_modules/gfsh/chapter_overview.html) 命令行界面已经[设置好](/web/20220627173722/https://www.baeldung.com/apache-geode#installation-and-setup)，我们可以启动一个名为`basicLocator`的定位器，然后启动一个名为`basicServer.`的服务器

为此，让我们在`gfsh` CLI 中运行以下命令:

```java
gfsh>start locator --name="basicLocator"
```

```java
gfsh>start server --name="basicServer"
```

一旦服务器开始运行，我们可以列出所有成员:

```java
gfsh>list members
```

CLI 输出应该列出定位器和服务器:

```java
 Name     | Id
------------ | ------------------------------------------------------------------
basicLocator | 10.25.3.192(basicLocator:25461:locator)<ec><v0>:1024 [Coordinator]
basicServer  | 10.25.3.192(basicServer:25546)<v1>:1025
```

瞧啊。我们已经准备好使用 Maven 命令运行我们的缓存客户端应用程序:

```java
mvn spring-boot:run
```

## 6.配置

让我们配置缓存客户端应用程序，通过 Apache Geode 服务器访问数据。

### 6.1.地区

首先，我们将创建一个名为`Author` 的实体，然后将其定义为 Apache Geode`Region.``Region`类似于 RDBMS 中的一个表:

```java
@Region("Authors")
public class Author {
    @Id
    private Long id;

    private String firstName;
    private String lastName;
    private int age;
}
```

让我们回顾一下在`Author`实体中声明的 Spring 数据 Geode 注释。

首先， [`@Region`](https://web.archive.org/web/20220627173722/https://docs.spring.io/spring-data/geode/docs/current/api/org/springframework/data/gemfire/mapping/annotation/Region.html) 将在 Apache Geode 服务器中创建`Authors`区域来保存`Author`对象。

然后，`@Id`会将该属性标记为主键。

### 6.2.实体

我们可以通过添加 [`@EnableEntityDefinedRegions`](https://web.archive.org/web/20220627173722/https://docs.spring.io/spring-data/geode/docs/current/api/org/springframework/data/gemfire/config/annotation/EnableEntityDefinedRegions.html) 来启用`Author`实体。

此外，我们将添加 [`@EnableClusterConfiguration`](https://web.archive.org/web/20220627173722/https://docs.spring.io/spring-data/geode/docs/current/api/org/springframework/data/gemfire/config/annotation/EnableClusterConfiguration.html) ，让应用程序在 Apache Geode 服务器中创建区域:

```java
@EnableEntityDefinedRegions(basePackageClasses = Author.class)
@EnableClusterConfiguration
// existing annotations
public class ClientCacheApp {
    // ...
}
```

因此，重新启动应用程序将自动创建区域:

```java
gfsh>list regions

List of regions
---------------
Authors
```

### 6.3.贮藏室ˌ仓库

接下来，我们将在`Author`实体上添加 CRUD 操作。

为此，让我们创建一个名为`AuthorRepository,`的存储库，它扩展了 Spring Data 的 [`CrudRepository`](/web/20220627173722/https://www.baeldung.com/spring-data-repositories#crudrepository) :

```java
public interface AuthorRepository extends CrudRepository<Author, Long> {
}
```

然后，我们将通过添加 [`@EnableGemfireRepositories`](https://web.archive.org/web/20220627173722/https://docs.spring.io/spring-data/geode/docs/current/api/org/springframework/data/gemfire/repository/config/EnableGemfireRepositories.html) 来启用`AuthorRepository`:

```java
@EnableGemfireRepositories(basePackageClasses = AuthorRepository.class)
// existing annotations
public class ClientCacheApp {
    // ...
}
```

现在，我们已经准备好使用`CrudRepository`提供的`save`和`findById`等方法对`Author`实体执行 CRUD 操作。

### 6.4.指数

Spring Data Geode 提供了一种在 Apache Geode 服务器中创建和启用索引的简单方法。

首先，我们将把 [`@EnableIndexing`](https://web.archive.org/web/20220627173722/https://docs.spring.io/spring-data/geode/docs/current/api/org/springframework/data/gemfire/config/annotation/EnableIndexing.html) 添加到`ClientCacheApp`类中:

```java
@EnableIndexing
// existing annotations
public class ClientCacheApp {
    // ...
}
```

然后，让我们将`@Indexed`添加到`Author`类的一个属性中:

```java
public class Author {
    @Id
    private Long id;

    @Indexed
    private int age;

    // existing data members
}
```

在这里，Spring Data Geode 将根据在`Author`实体中定义的注释自动实现索引。

因此，`@Id`将为`id`实现主键索引。类似地，`@Indexed`将为`age`实现散列索引。

现在，让我们重新启动应用程序，并确认在 Apache Geode 服务器中创建的索引:

```java
gfsh> list indexes

Member Name | Region Path |       Name        | Type  | Indexed Expression | From Clause | Valid Index
----------- | ----------- | ----------------- | ----- | ------------------ | ----------- | -----------
basicServer | /Authors    | AuthorsAgeKeyIdx  | RANGE | age                | /Authors    | true
basicServer | /Authors    | AuthorsIdHashIdx  | RANGE | id                 | /Authors    | true
```

同样，我们可以使用`@LuceneIndexed` 为`String`类型的属性创建一个 Apache Geode Lucene 索引。

### 6.5.连续查询

连续查询使应用程序能够在服务器中的数据发生变化时接收自动通知。它匹配查询并依赖于订阅模型。

为了添加功能，我们将创建`AuthorService`并添加带有匹配查询的 [`@ContinuousQuery`](https://web.archive.org/web/20220627173722/https://docs.spring.io/spring-data/geode/docs/current/api/org/springframework/data/gemfire/listener/annotation/ContinuousQuery.html) :

```java
@Service
public class AuthorService {
    @ContinuousQuery(query = "SELECT * FROM /Authors a WHERE a.id = 1")
    public void process(CqEvent event) {
        System.out.println("Author #" + event.getKey() + " updated to " + event.getNewValue());
    }
}
```

为了使用连续查询，我们将启用服务器到客户端的订阅:

```java
@ClientCacheApplication(subscriptionEnabled = true)
// existing annotations
public class ClientCacheApp {
    // ...
}
```

因此，每当我们用等于 1 的`id`修改一个`Author`对象时，我们的应用程序将在`process`方法中收到自动通知。

## 7.附加注释

让我们探索一下 Spring 数据地理数据库中额外提供的一些方便的注释。

### 7.1.`@PeerCacheApplication`

到目前为止，我们已经检查了作为 Apache Geode 缓存客户端的 Spring Boot 应用程序。有时，我们可能要求我们的应用程序是 Apache Geode 对等缓存应用程序。

然后，我们应该用 [`@PeerCacheApplication`](https://web.archive.org/web/20220627173722/https://docs.spring.io/spring-data/geode/docs/current/api/org/springframework/data/gemfire/config/annotation/PeerCacheApplication.html) 代替`@CacheClientApplication`来注释主类。

另外，`@PeerCacheApplication` 将自动创建一个嵌入式对等缓存实例进行连接。

### 7.2.`@CacheServerApplication`

类似地，为了让我们的 Spring Boot 应用程序既作为对等成员又作为服务器，我们可以用`[@CacheServerApplication](https://web.archive.org/web/20220627173722/https://docs.spring.io/spring-data/geode/docs/current/api/org/springframework/data/gemfire/config/annotation/CacheServerApplication.html).`注释主类

### 7.3.`@EnableHttpService`

我们可以为`@PeerCacheApplication`和`@CacheServerApplication.`启用 Apache Geode 的嵌入式 HTTP 服务器

为此，我们需要用`[@EnableHttpService](https://web.archive.org/web/20220627173722/https://docs.spring.io/spring-data/geode/docs/current/api/org/springframework/data/gemfire/config/annotation/EnableHttpService.html).`注释主类。默认情况下，HTTP 服务从端口 7070 开始。

### 7.4.`@EnableLogging`

我们可以通过简单地将 [`@EnableLogging`](https://web.archive.org/web/20220627173722/https://docs.spring.io/spring-data/geode/docs/current/api/org/springframework/data/gemfire/config/annotation/EnableLogging.html) 添加到主类来启用日志记录。同时，我们可以使用`logLevel`和`logFile`属性来设置相应的属性。

### 7.5.`@EnablePdx`

此外，我们可以为我们所有的域启用 Apache Geode 的 PDX 序列化技术，只需将 [`@EnablePdx`](https://web.archive.org/web/20220627173722/https://docs.spring.io/spring-data/geode/docs/current/api/org/springframework/data/gemfire/config/annotation/EnablePdx.html) 添加到主类中。

### 7.6.`@EnableSsl`和`@EnableSecurity`

我们可以使用 [`@EnableSsl`](https://web.archive.org/web/20220627173722/https://docs.spring.io/spring-data/geode/docs/current/api/org/springframework/data/gemfire/config/annotation/EnableSsl.html) 来打开 Apache Geode 的 TCP/IP 套接字 SSL。类似地， [`@EnableSecurity`](https://web.archive.org/web/20220627173722/https://docs.spring.io/spring-data/geode/docs/current/api/org/springframework/data/gemfire/config/annotation/EnableSecurity.html) 可用于启用 Apache Geode 的安全性以进行身份验证和授权。

## 8.结论

在本教程中，我们探索了 Apache Geode 的 Spring 数据。

首先，我们创建了一个 Spring Boot 应用程序作为 Apache Geode 缓存客户端应用程序。

同时，我们研究了 Spring Data Geode 为配置和启用 Apache Geode 特性而提供的一些方便的注释。

最后，我们研究了一些附加注释，如@ `PeerCacheApplication` 和 `@` `CacheServerApplication` ,以将应用程序更改为集群配置中的对等点或服务器。

像往常一样，所有的代码实现都可以在 GitHub 上获得[。](https://web.archive.org/web/20220627173722/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-geode)