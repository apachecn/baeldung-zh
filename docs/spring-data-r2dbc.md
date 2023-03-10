# 使用 Spring 数据快速查看 R2DBC

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-r2dbc>

## 1.介绍

[R2DBC](https://web.archive.org/web/20220628092444/https://r2dbc.io/) (反应式关系数据库连接)是 Pivotal 在 Spring One Platform 2018 期间展示的一项成果。它打算为 SQL 数据库创建一个反应式 API。

换句话说，这项工作使用完全非阻塞的驱动程序创建了一个数据库连接。

在本教程中，我们将看一个使用 Spring Data R2BDC 的应用程序的例子。关于更低级的 R2DBC API 的指南，请看我们之前的文章。

## 2.我们的第一个 Spring Data R2DBC 项目

首先，R2DBC 项目是最近才开始的。目前只有 **PostGres、MSSQL 和 H2 有 R2DBC 驱动。**此外，**我们不能使用它的所有 Spring Boot 功能**。因此，我们需要手动添加一些步骤。但是，**我们可以利用像 [Spring Data](/web/20220628092444/https://www.baeldung.com/spring-data) 这样的项目来帮助我们。**

我们将首先创建一个 Maven 项目。在这一点上，R2DBC 存在一些依赖问题，所以我们的`pom.xml`将比正常情况下更大。

对于本文的范围，我们将使用 [H2](https://web.archive.org/web/20220628092444/https://search.maven.org/search?q=g:com.h2database) 作为我们的数据库，我们将为我们的应用程序创建反应式 CRUD 函数。

让我们打开生成项目的`pom.xml`,添加适当的依赖项以及一些早期发布的 Spring 存储库:

```java
<dependencies>
     <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-r2dbc</artifactId>
        <version>1.0.0.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>io.r2dbc</groupId>
        <artifactId>r2dbc-h2</artifactId>
        <version>0.8.1.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <version>1.4.199</version>
    </dependency>
</dependencies>
```

其他需要的工件包括 [Lombok、](https://web.archive.org/web/20220628092444/https://search.maven.org/search?q=g:org.projectlombok%20AND%20a:lombok&core=gav) [Spring WebFlux](https://web.archive.org/web/20220628092444/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-webflux&core=gav) 和其他一些完成我们项目[依赖项](https://web.archive.org/web/20220628092444/https://github.com/rodrigolgraciano/tutorials/blob/master/spring-5-data-reactive/pom.xml)的工件。

## 3.连接工厂

当使用数据库时，我们需要一个连接工厂。当然，R2DBC 也需要同样的东西。

因此，我们现在将添加连接到实例的详细信息:

```java
@Configuration
@EnableR2dbcRepositories
class R2DBCConfiguration extends AbstractR2dbcConfiguration {
    @Bean
    public H2ConnectionFactory connectionFactory() {
        return new H2ConnectionFactory(
            H2ConnectionConfiguration.builder()
              .url("mem:testdb;DB_CLOSE_DELAY=-1;")
              .username("sa")
              .build()
        );
    }
}
```

我们在上面的代码中注意到的第一件事是`@EnableR2dbcRepositories`。我们需要这个注释来使用 Spring 数据功能。此外，我们正在**扩展`AbstractR2dbcConfiguration` ，因为它将提供我们以后需要的大量 beans。**

## 4.我们的第一个 R2DBC 应用

我们的下一步是创建存储库:

```java
interface PlayerRepository extends ReactiveCrudRepository<Player, Integer> {}
```

**`ReactiveCrudRepository`界面非常有用。例如，它提供了基本的 CRUD 功能。**

最后，我们将定义我们的模型类。我们将使用 Lombok 来避免样板代码:

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
class Player {
    @Id
    Integer id;
    String name;
    Integer age;
}
```

## 5.测试

是时候测试我们的代码了。因此，让我们从创建几个测试用例开始:

```java
@Test
public void whenDeleteAll_then0IsExpected() {
    playerRepository.deleteAll()
      .as(StepVerifier::create)
      .expectNextCount(0)
      .verifyComplete();
}

@Test
public void whenInsert6_then6AreExpected() {
    insertPlayers();
    playerRepository.findAll()
      .as(StepVerifier::create)
      .expectNextCount(6)
      .verifyComplete();
}
```

## 6.自定义查询

**我们还可以生成自定义查询**。为了添加它，我们需要更改我们的`PlayerRepository`:

```java
@Query("select id, name, age from player where name = $1")
Flux<Player> findAllByName(String name);

@Query("select * from player where age = $1")
Flux<Player> findByAge(int age);
```

除了现有的测试，我们将向最近更新的存储库中添加测试:

```java
@Test
public void whenSearchForCR7_then1IsExpected() {
    insertPlayers();
    playerRepository.findAllByName("CR7")
      .as(StepVerifier::create)
      .expectNextCount(1)
      .verifyComplete();
}

@Test
public void whenSearchFor32YearsOld_then2AreExpected() {
    insertPlayers();
    playerRepository.findByAge(32)
      .as(StepVerifier::create)
      .expectNextCount(2)
      .verifyComplete();
}

private void insertPlayers() {
    List<Player> players = Arrays.asList(
        new Player(1, "Kaka", 37),
        new Player(2, "Messi", 32),
        new Player(3, "Mbappé", 20),
        new Player(4, "CR7", 34),
        new Player(5, "Lewandowski", 30),
        new Player(6, "Cavani", 32)
    );
    playerRepository.saveAll(players).subscribe();
} 
```

## 7.一批

R2DBC 的另一个特性是创建批处理。批处理在执行多个 SQL 语句时非常有用，因为它们比单个操作执行得更好。

为了创建一个`Batch`，我们需要一个`Connection`对象:

```java
Batch batch = connection.createBatch();
```

在我们的应用程序创建了`Batch` 实例之后，我们可以添加任意多的 SQL 语句。为了执行它，我们将调用`execute() `方法。批处理的结果是一个`Publisher`，它将为每个语句返回一个结果对象。

所以让我们直接进入代码，看看如何创建一个`Batch`:

```java
@Test
public void whenBatchHas2Operations_then2AreExpected() {
    Mono.from(factory.create())
      .flatMapMany(connection -> Flux.from(connection
        .createBatch()
        .add("select * from player")
        .add("select * from player")
        .execute()))
      .as(StepVerifier::create)
      .expectNextCount(2)
      .verifyComplete();
}
```

## 8.结论

综上所述，R2DBC 还处于早期阶段。它试图创建一个 SPI，为 SQL 数据库定义一个反应式 API。当与 Spring WebFlux 一起使用时，R2DBC 允许我们编写一个应用程序，从顶层异步处理数据，一直到数据库。

与往常一样，该代码可在 GitHub 获得。