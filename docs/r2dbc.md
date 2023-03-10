# R2DBC–反应式关系数据库连接

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/r2dbc>

## 1.概观

在本教程中，我们将展示如何使用 R2DBC 以一种被动的方式执行数据库操作。

为了探索 R2DBC，我们将创建一个简单的 Spring WebFlux REST 应用程序，它为单个实体实现 CRUD 操作，只使用异步操作来实现这个目标。

## 2.什么是`R2DBC`？

反应式开发正在兴起，每天都有新的框架出现，现有的框架也越来越多地被采用。然而，反应式开发的一个主要问题是，Java/JVM 世界中的**数据库访问基本上保持同步**。这是 JDBC 设计方式的直接后果，并导致一些丑陋的黑客来适应这两种根本不同的方法。

为了满足 Java 领域对异步数据库访问的需求，出现了两个标准。第一个是 ADBC(Asynchronous Database Access API ),由 Oracle 支持，但是在撰写本文时，它似乎有些停滞，没有明确的时间表。

第二个是 R2DBC(反应式关系数据库连接),由 Pivotal 和其他公司的团队领导的一个社区项目。这个项目仍处于测试阶段，已经显示出更多的活力，并且已经为 Postgres、H2 和 MSSQL 数据库提供了驱动程序。

## 3.项目设置

在项目中使用 R2DBC 要求我们向核心 API 和合适的驱动程序添加依赖项。在我们的示例中，我们将使用 H2，因此这意味着只有两个依赖项:

```java
<dependency>
    <groupId>io.r2dbc</groupId>
    <artifactId>r2dbc-spi</artifactId>
    <version>0.8.0.M7</version>
</dependency>
<dependency>
    <groupId>io.r2dbc</groupId>
    <artifactId>r2dbc-h2</artifactId>
    <version>0.8.0.M7</version>
</dependency>
```

Maven Central 现在仍然没有 R2DBC 工件，所以我们还需要向我们的项目添加几个 Spring 的存储库:

```java
<repositories>
    <repository>
        <id>spring-milestones</id>
        <name>Spring Milestones</name>
        <url>https://repo.spring.io/milestone</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
   </repository>
   <repository>
       <id>spring-snapshots</id>
       <name>Spring Snapshots</name>
       <url>https://repo.spring.io/snapshot</url>
       <snapshots>
           <enabled>true</enabled>
       </snapshots>
    </repository>
</repositories>
```

## 4.连接工厂设置

使用 R2DBC 访问数据库，我们需要做的第一件事是**创建一个 *ConnectionFactory 对象*** ，它的作用类似于 JDBC 的`DataSource.`。创建`ConnectionFactory`最直接的方法是通过`ConnectionFactories`类。

这个类有静态方法，接受一个`ConnectionFactoryOptions`对象并返回一个`ConnectionFactory. `,因为我们只需要我们的`ConnectionFactory`的一个实例，让我们创建一个`@Bean`,我们可以在以后需要时通过注入使用它:

```java
@Bean
public ConnectionFactory connectionFactory(R2DBCConfigurationProperties properties) {
    ConnectionFactoryOptions baseOptions = ConnectionFactoryOptions.parse(properties.getUrl());
    Builder ob = ConnectionFactoryOptions.builder().from(baseOptions);
    if (!StringUtil.isNullOrEmpty(properties.getUser())) {
        ob = ob.option(USER, properties.getUser());
    }
    if (!StringUtil.isNullOrEmpty(properties.getPassword())) {
        ob = ob.option(PASSWORD, properties.getPassword());
    }        
    return ConnectionFactories.get(ob.build());    
} 
```

在这里，我们从用`@ConfigurationProperties`注释修饰的助手类接收选项，并填充我们的`ConnectionFactoryOptions`实例。为了填充它，R2DBC 实现了一个带有一个`Option`和一个值的`option `方法的构建器模式。

R2DBC 定义了许多众所周知的选项，比如我们前面用过的`USERNAME`和`PASSWORD `。设置这些选项的另一种方法是将一个连接字符串传递给`ConnectionFactoryOptions`类的`parse()`方法。

以下是典型 R2DBC 连接 URL 的示例:

```java
r2dbc:h2:mem://./testdb
```

让我们把这个字符串分解成几个部分:

*   `r2dbc`:R2DBC URL 的固定模式标识符——另一个有效的模式是`rd2bcs`，用于 SSL 安全连接
*   `h2`:用于定位适当连接工厂的驱动程序标识符
*   特定于驱动程序的协议——在我们的例子中，这对应于内存数据库
*   `//./testdb`:特定于驱动程序的字符串，通常包含主机、数据库和任何附加选项。

一旦我们准备好了选项集，我们就把它传递给静态工厂方法来创建我们的 bean。

## 5.执行语句

与 JDBC 类似，使用 R2DBC 主要是为了向数据库发送 SQL 语句并处理结果集。**然而，由于 R2DBC 是一个反应式 API，它严重依赖于反应式流类型，比如`Publisher `和`Subscriber`** 。

直接使用这些类型有点麻烦，所以我们将使用 project reactor 的类型，如`Mono `和`Flux`，它们可以帮助我们编写更干净、更简洁的代码。

在接下来的小节中，我们将看到如何通过为一个简单的`Account`类创建一个反应性的 DAO 类来实现与数据库相关的任务。这个类只包含三个属性，并且在我们的数据库中有一个对应的表:

```java
public class Account {
    private Long id;
    private String iban;
    private BigDecimal balance;
    // ... getters and setters omitted
}
```

### 5.1.建立联系

在向数据库发送任何语句之前，**我们需要一个`Connection`实例**。我们已经看到了如何创建一个`ConnectionFactory`，所以毫不奇怪我们将使用它来获得一个`Connection`。我们必须记住的是，现在，我们得到的不是常规的`Connection`，而是单个`Connection.`的`Publisher `

我们的`ReactiveAccountDao,` 是一个普通的弹簧`@Component`，通过构造函数注入获得它的`ConnectionFactory`，所以它在处理程序方法中很容易得到。

让我们看一下`findById()`方法的前几行，看看如何检索并开始使用`Connection`:

```java
public Mono<Account>> findById(Long id) {         
    return Mono.from(connectionFactory.create())
      .flatMap(c ->
          // use the connection
      )
      // ... downstream processing omitted
}
```

这里，我们将从我们的`ConnectionFactory`返回的`Publisher`改编成一个`Mono` ，它是我们事件流的初始源。

### 5.1.准备和提交报表

现在我们有了一个`Connection`，让我们用它来创建一个`Statement`，并为它绑定一个参数:

```java
.flatMap( c -> 
    Mono.from(c.createStatement("select id,iban,balance from Account where id = $1")
      .bind("$1", id)
      .execute())
      .doFinally((st) -> close(c))
 ) 
```

`Connection`的方法`createStatement`接受一个 SQL 查询字符串，该字符串可以有可选的绑定占位符——在规范中称为“标记”[。](https://web.archive.org/web/20221127043958/https://r2dbc.io/spec/0.8.0.M8/spec/html/#statements.parameterized)

这里有几个值得注意的地方:首先， **`createStatement`是一个同步操作**，它允许我们使用流畅的风格将值绑定到返回的`Statement; `第二，也是非常重要的一点，**占位符/标记语法是特定于供应商的！**

在这个例子中，我们使用 H2 的特定语法，它使用`$n`来标记参数。其他厂商可能会使用不同的语法，比如`:param`、`@Pn`，或者其他一些约定。在将遗留代码移植到这个新的 API 时，这是我们必须注意的一个重要方面。

由于流畅的 API 模式和简化的类型，绑定过程本身非常简单:**只有一个重载的`bind()`方法负责所有的类型转换**——当然，要遵守数据库规则。

传递给`bind() `的第一个参数可以是一个从零开始的序号，对应于标记在语句中的位置，也可以是一个带有实际标记的字符串。

一旦我们为所有参数设置了值，我们就调用`execute()`，它返回一个`Result `对象的`Publisher `，我们再次将它包装成一个`Mono `用于进一步处理。我们将一个`doFinally()`处理程序附加到这个`Mono `上，这样我们可以确保无论流处理是否正常完成，我们都会关闭我们的连接。

### 5.2.处理结果

我们管道中的下一步是负责**处理`Result`对象并生成`ResponseEntity<` `Account>` 实例**的流。

因为我们知道给定的`id`只能有一个实例，所以我们实际上会返回一个`Mono`流。实际的转换发生在传递给接收到的`Result`的`map()`方法的函数内部:

```java
.map(result -> result.map((row, meta) -> 
    new Account(row.get("id", Long.class),
      row.get("iban", String.class),
      row.get("balance", BigDecimal.class))))
.flatMap(p -> Mono.from(p)); 
```

结果的`map()`方法需要一个带两个参数的函数。第一个是一个`Row`对象，我们用它来收集每一列的值并填充一个`Account `实例。第二个是`meta`，是一个`RowMetadata `对象，包含当前行的信息，比如列名和类型。

我们管道中的前一个`map()`调用解析为一个`Mono<Producer<Account>>`，但是我们需要从这个方法返回一个`Mono<Account>`。为了解决这个问题，我们添加了最后的`flatMap()` 步骤，将`Producer`调整为`Mono.`

### 5.3.批处理语句

R2DBC 还支持语句批处理的创建和执行，这允许在单个`execute() `调用中执行多个 SQL 语句。与常规语句相比，**批处理语句不支持绑定**，主要用于 ETL 作业等场景中的性能原因。

我们的示例项目使用一批语句来创建`Account`表，并将一些测试数据插入其中:

```java
@Bean
public CommandLineRunner initDatabase(ConnectionFactory cf) {
    return (args) ->
      Flux.from(cf.create())
        .flatMap(c -> 
            Flux.from(c.createBatch()
              .add("drop table if exists Account")
              .add("create table Account(" +
                "id IDENTITY(1,1)," +
                "iban varchar(80) not null," +
                "balance DECIMAL(18,2) not null)")
              .add("insert into Account(iban,balance)" +
                "values('BR430120980198201982',100.00)")
              .add("insert into Account(iban,balance)" +
                "values('BR430120998729871000',250.00)")
              .execute())
            .doFinally((st) -> c.close())
          )
        .log()
        .blockLast();
}
```

这里，我们使用从`createBatch()`返回的`Batch `并添加一些 SQL 语句。然后，我们使用在`Statement` 接口中可用的相同的`execute()`方法发送这些语句来执行。

在这个特殊的例子中，我们对任何结果都不感兴趣——只是所有语句都执行得很好。如果我们需要任何生成的结果，我们所要做的就是在这个流中添加一个下游步骤来处理发出的`Result`对象。

## 6.处理

我们将在本教程中讨论的最后一个主题是事务。正如我们现在应该预料到的，我们像在 JDBC 一样管理事务，也就是说，通过使用在`Connection `对象中可用的方法。

和以前一样，主要的区别是现在**所有与事务相关的方法都是异步的**，返回一个`Publisher`，我们必须在适当的时候将它添加到我们的流中。

我们的示例项目在实现`createAccount() `方法时使用了一个事务:

```java
public Mono<Account> createAccount(Account account) {    
    return Mono.from(connectionFactory.create())
      .flatMap(c -> Mono.from(c.beginTransaction())
        .then(Mono.from(c.createStatement("insert into Account(iban,balance) values($1,$2)")
          .bind("$1", account.getIban())
          .bind("$2", account.getBalance())
          .returnGeneratedValues("id")
          .execute()))
        .map(result -> result.map((row, meta) -> 
            new Account(row.get("id", Long.class),
              account.getIban(),
              account.getBalance())))
        .flatMap(pub -> Mono.from(pub))
        .delayUntil(r -> c.commitTransaction())
        .doFinally((st) -> c.close()));   
}
```

这里，我们在两点中添加了与事务相关的调用。首先，在从数据库获得一个新连接之后，我们调用`beginTransactionMethod()`。一旦我们知道事务已经成功启动，我们就准备并执行`insert`语句。

这次我们还使用了`returnGeneratedValues()`方法来指示数据库返回为这个新的`Account`生成的标识值。R2DBC 在一个`Result `中返回这些值，其中包含一行所有生成的值，我们用它来创建`Account`实例。

同样，我们需要将传入的`Mono<Publisher<Account>>`调整为`Mono<Account>`，所以我们添加了一个`flatMap()` 来解决这个`. `接下来，我们在一个`delayUntil()`步骤中提交事务。我们需要这样做，因为我们希望确保返回的`Account `已经提交到数据库中。

最后，我们在这条管道上附加了一个`doFinally`步骤，当来自返回的`Mono`的所有事件都被消费时，这个步骤关闭`Connection`。

## 7.DAO 用法示例

现在我们有了一个反应式 DAO，让我们用它来创建一个简单的 [Spring WebFlux](/web/20221127043958/https://www.baeldung.com/spring-webflux) 应用程序，展示如何在一个典型的应用程序中使用它。由于这个框架已经支持反应式构造，这就变成了一个琐碎的任务。例如，让我们看看`GET`方法的实现:

```java
@RestController
public class AccountResource {
    private final ReactiveAccountDao accountDao;

    public AccountResource(ReactiveAccountDao accountDao) {
        this.accountDao = accountDao;
    }

    @GetMapping("/accounts/{id}")
    public Mono<ResponseEntity<Account>> getAccount(@PathVariable("id") Long id) {
        return accountDao.findById(id)
          .map(acc -> new ResponseEntity<>(acc, HttpStatus.OK))
          .switchIfEmpty(Mono.just(new ResponseEntity<>(null, HttpStatus.NOT_FOUND)));
    }
    // ... other methods omitted
}
```

这里，我们使用 DAO 返回的`Mono`来构造一个带有适当状态代码的`ResponseEntity`。我们这样做只是因为我们想要一个`NOT_FOUND` (404) 状态代码，而不存在具有给定 id 的`Account` 。

## 8.结论

在本文中，我们已经介绍了使用 R2DBC 进行反应式数据库访问的基础知识。虽然还处于起步阶段，但这个项目正在快速发展，目标是在 2020 年初的某个时候发布。

与肯定不会成为 Java 12 的一部分的 ADBA 相比，R2DBC 似乎更有前途，并且已经为一些流行的数据库提供了驱动程序——Oracle 在这里是一个明显的缺席。

像往常一样，本教程中使用的完整源代码可以从 Github 上的[处获得。](https://web.archive.org/web/20221127043958/https://github.com/eugenp/tutorials/tree/master/persistence-modules/r2dbc)