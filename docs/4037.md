# 具有多个 Spring 数据模块的存储库

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-multiple-data-modules>

## 1.介绍

有时，我们需要在同一个应用程序中连接到多种数据库技术。

在本教程中，我们将探索在同一个应用程序中使用多个 Spring 数据模块时的各种配置选项。

让我们用一个玩具 Spring Boot 书店来探讨这个话题。

## 2.必需的依赖关系

首先，我们需要在 `pom.xml` 文件中添加依赖项，这样我们就可以使用`[spring-boot-starter-data-mongodb](https://web.archive.org/web/20221206033711/https://spring.io/projects/spring-data-mongodb)`和`[spring-boot-starter-data-cassandra](https://web.archive.org/web/20221206033711/https://spring.io/projects/spring-data-cassandra)` Spring Boot 数据绑定:

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-cassandra</artifactId>
  <version>2.2.2.RELEASE</version>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-mongodb</artifactId>
  <version>2.2.2.RELEASE</version>
</dependency>
```

## 3.数据库设置

接下来，**我们需要通过使用预先构建的 [Docker](https://web.archive.org/web/20221206033711/https://docs.docker.com/install/) 图像 [Cassandra](https://web.archive.org/web/20221206033711/https://hub.docker.com/_/cassandra) 和 [Mongo](https://web.archive.org/web/20221206033711/https://hub.docker.com/_/mongo) **:** 来建立实际的数据库**

```
$ docker run --name mongo-db -d -p 27017:27017 mongo:latest
$ docker run --name cassandra-db -d -p 9042:9042 cassandra:latest 
```

这两个命令将自动下载最新的 Cassandra 和 MongoDB Docker 映像，并运行实际的容器。

我们还需要在实际的 OS 环境中从容器内部转发端口(使用`-p`选项),以便我们的应用程序可以访问数据库。

**我们必须使用`cqlsh`实用程序为 Cassandra** 创建数据库结构。 **Keyspace 的创建不能由`CassandraDataAutoConfiguration`** 自动完成，所以我们需要使用 [CQL](https://web.archive.org/web/20221206033711/https://cassandra.apache.org/doc/latest/cql/) 语法来声明它。

首先，我们附加到 Cassandra 容器的`bash` 外壳:

```
$ docker exec -it cassandra-db /bin/bash
[[email protected]](/web/20221206033711/https://www.baeldung.com/cdn-cgi/l/email-protection):/# cqlsh
Connected to Test Cluster at 127.0.0.1:9042.
[cqlsh 5.0.1 | Cassandra 3.11.4 | CQL spec 3.4.4 | Native protocol v4]
Use HELP for help.
cqlsh> CREATE KEYSPACE IF NOT exists baeldung 
WITH replication = {'class':'SimpleStrategy', 'replication_factor':1};
cqlsh> USE baeldung;
cqlsh> CREATE TABLE bookaudit(
   bookid VARCHAR,
   rentalrecno VARCHAR,
   loandate VARCHAR,
   loaner VARCHAR,
   primary key(bookid, rentalrecno)
); 
```

在第 6 行和第 9 行，我们创建了相关的密钥空间和表`.`

我们可以跳过创建表，简单地依靠`spring-boot-starter-data-cassandra`为我们初始化模式，然而，由于我们想要单独探索框架配置，这是一个必要的步骤。

默认情况下， **Mongo 不强制任何类型的模式验证，** **因此不需要为它配置任何东西**。

最后，我们在`application.properties`中配置数据库的相关信息:

```
spring.data.cassandra.username=cassandra
spring.data.cassandra.password=cassandra
spring.data.cassandra.keyspaceName=baeldung
spring.data.cassandra.contactPoints=localhost
spring.data.cassandra.port=9042
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.database=baeldung
```

## 4.数据存储检测机制

当在类路径上检测到多个 Spring 数据模块时，Spring 框架进入严格的存储库配置模式。这意味着它在底层使用不同的检测机制，来识别哪个存储库属于哪个持久性技术。

### 4.1.扩展特定于模块的存储库接口

第一种机制试图确定存储库是否扩展了特定于 Spring 数据模块的存储库类型:

```
public interface BookAuditRepository extends CassandraRepository<BookAudit, String> {

}
```

在我们的例子中，`BookAudit.java` 包含一些基本的存储结构，用于跟踪借书的用户:

```
public class BookAudit {
  private String bookId;
  private String rentalRecNo;
  private String loaner;
  private String loanDate;

  // standard getters and setters
} 
```

这同样适用于 MongoDB 相关的存储库定义:

```
public interface BookDocumentRepository extends MongoRepository<BookDocument, String> {

}
```

这个存储了书的内容和一些相关的元数据:

```
public class BookDocument {
  private String bookId;
  private String bookName;
  private String bookAuthor;
  private String content;

  // standard getters and setters
}
```

当应用程序上下文被加载时，**框架将使用从其派生的基类来匹配每个存储库类型**:

```
@Test
public void givenBookAudit_whenPersistWithBookAuditRepository_thenSuccess() {

  // given
  BookAudit bookAudit = 
    new BookAudit("lorem", "ipsum", "Baeldung", "19:30/20.08.2017");

  // when
  bookAuditRepository.save(bookAudit);

  // then
  List<BookAudit> result = bookAuditRepository.findAll();
  assertThat(result.isEmpty(), is(false));
  assertThat(result.contains(bookAudit), is(true));
}
```

您可以注意到我们的域类是简单的 Java 对象。在这种特殊情况下，Cassandra 数据库模式必须用`CQL`在外部创建，正如我们在第 3 节中所做的那样。

### 4.2.在域对象上使用特定于模块的注释

第二个策略**基于领域对象上特定于模块的注释来确定持久性技术。**

让我们扩展一个通用的`CrudRepostitory`,现在依靠托管对象注释进行检测:

```
public interface BookAuditCrudRepository extends CrudRepository<BookAudit, String> {

}
```

```
public interface BookDocumentCrudRepository extends CrudRepository<BookDocument, String> {

} 
```

`BookAudit.java`现在用 Cassandra 特有的`@Table`进行注释，并且需要一个复合主键:

```
@Table
public class BookAudit {

  @PrimaryKeyColumn(type = PrimaryKeyType.PARTITIONED)
  private String bookId;
  @PrimaryKeyColumn
  private String rentalRecNo;
  private String loaner;
  private String loanDate;

  // standard getters and setters
} 
```

我们选择组合`bookId`和`rentalRecNo`作为我们的唯一标准，因为我们的应用程序只是在每次有人借书时记录一个新的租赁记录。

对于`BookDocument.java`,我们使用特定于 MongoDB 的`@Document` 注释:

```
@Document
public class BookDocument {

  private String bookId;
  private String bookName;
  private String bookAuthor;
  private String content;

  // standard getters and setters
} 
```

用`CrudRepository` 触发`BookDocument` 保存仍然成功，但是第 11 行返回的类型现在是`Iterable` 而不是`List`:

```
@Test
public void givenBookAudit_whenPersistWithBookDocumentCrudRepository_thenSuccess() {

  // given
  BookDocument bookDocument = 
    new BookDocument("lorem", "Foundation", "Isaac Asimov", "Once upon a time ...");

  // when
  bookDocumentCrudRepository.save(bookDocument);

  // then
  Iterable<BookDocument> resultIterable = bookDocumentCrudRepository.findAll();
  List<BookDocument> result = StreamSupport.stream(resultIterable.spliterator(), false)
                                           .collect(Collectors.toList());
  assertThat(result.isEmpty(), is(false));
  assertThat(result.contains(bookDocument), is(true));
}
```

### 4.3.使用基于包的作用域

最后，**我们可以通过使用`@EnableCassandraRepositories`和`@EnableMongoRepositories`注释来指定定义库的基础包**:

```
@EnableCassandraRepositories(basePackages="com.baeldung.multipledatamodules.cassandra")
@EnableMongoRepositories(basePackages="com.baeldung.multipledatamodules.mongo")
public class SpringDataMultipleModules {

  public static void main(String[] args) {
    SpringApplication.run(SpringDataMultipleModules.class, args);
  }
} 
```

正如我们在第 1 行和第 2 行看到的，**这个配置模式假设我们为 Cassandra 和 MongoDB 库使用不同的包**。

## 5.结论

在本教程中，我们配置了一个简单的 Spring Boot 应用程序，以三种方式使用两个不同的 Spring 数据模块。

作为第一个例子，我们扩展了`CassandraRepository` 和`MongoRepository`，并为域对象使用了简单的类。

在我们的第二种方法中，我们扩展了通用的`CrudRepository`接口，并依赖于特定于模块的注释，比如托管对象上的`@Table` 和`@Document` 。

最后，当我们在`@EnableCassandraRepositories` 和`@EnableMongoRepositories`的帮助下配置应用程序时，我们使用了基于包的检测。

和往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221206033711/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-data)