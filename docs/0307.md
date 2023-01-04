# CassandraUnit 测试教程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-cassandra-unit>

## 1。概述

Apache Cassandra 是一个强大的开源 NoSQL 分布式数据库。在之前的教程中，我们学习了如何使用 [Cassandra 和 Java](/web/20220525141857/https://www.baeldung.com/cassandra-with-java) 的一些基础知识。

在本教程中，**我们将在前一个的基础上，学习如何使用 [CassandraUnit](https://web.archive.org/web/20220525141857/https://github.com/jsevellec/cassandra-unit) 编写可靠的、自包含的单元测试。**

首先，我们将从如何设置和配置 CassandraUnit 的最新版本开始。然后，我们将探索几个例子，说明如何编写不依赖于外部数据库服务器运行的单元测试。

而且，如果你在生产中运行 Cassandra，你肯定可以省去运行和维护你自己的服务器的复杂性，而使用 [Astra 数据库](/web/20220525141857/https://www.baeldung.com/datastax-post)，这是 **，一个基于 Apache Cassandra 的云数据库。**

## 2。依赖性

当然，我们需要为 Apache Cassandra 添加标准的 [Datastax Java 驱动程序](https://web.archive.org/web/20220525141857/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.datastax.oss%22%20AND%20a%3A%22java-driver-core%22)到我们的`pom.xml`:

```
<dependency>
    <groupId>com.datastax.oss</groupId>
    <artifactId>java-driver-core</artifactId>
    <version>4.13.0</version>
</dependency>
```

为了用嵌入式数据库服务器测试我们的代码，我们还应该将 [`cassandra-unit`](https://web.archive.org/web/20220525141857/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.cassandraunit%22%20AND%20a%3A%22cassandra-unit%22) 依赖项添加到我们的`pom.xml`中:

```
<dependency>
    <groupId>org.cassandraunit</groupId>
    <artifactId>cassandra-unit</artifactId>
    <version>4.3.1.0</version>
    <scope>test</scope>
</dependency>
```

现在我们已经配置了所有必要的依赖项，我们可以开始编写单元测试了。

## 3。入门

在本教程中，我们测试的重点将是一个简单的人员表，我们通过一个简单的 [CQL](/web/20220525141857/https://www.baeldung.com/cassandra-data-types) 脚本来控制它:

```
CREATE TABLE person(
    id varchar,
    name varchar,
    PRIMARY KEY(id));

INSERT INTO person(id, name) values('1234','Eugen');
INSERT INTO person(id, name) values('5678','Michael');
```

正如我们将要看到的，CassandraUnit 提供了几种变体来帮助我们编写测试，但它们的核心是几个我们将不断重复的简单概念:

*   首先，我们将启动一个嵌入式 Cassandra 服务器，它运行在 JVM 的内存中。
*   然后，我们将把 person 数据集加载到正在运行的嵌入式实例中。
*   最后，我们将启动一个简单的查询来验证我们的数据已经被正确加载。

**在结束这一节之前，简单介绍一下测试。一般来说，当编写干净的单元或集成测试时，我们不应该依赖我们可能无法控制或可能突然停止工作的外部服务**。这可能会对我们的测试结果产生不利影响。

类似地，如果我们依赖外部服务，在这种情况下是一个正在运行的 Cassandra 数据库，我们很可能无法在测试中按照我们想要的方式设置、控制和拆除它。

## 4。使用本地方法进行测试

让我们先来看看如何使用 CassandraUnit 自带的原生 API。首先，我们将继续定义我们的单元测试和测试设置:

```
public class NativeEmbeddedCassandraUnitTest {

    private CqlSession session;

    @Before
    public void setUp() throws Exception {
        EmbeddedCassandraServerHelper.startEmbeddedCassandra();
        session = EmbeddedCassandraServerHelper.getSession();
        new CQLDataLoader(session).load(new ClassPathCQLDataSet("people.cql", "people"));
    }
}
```

让我们浏览一下测试设置的关键部分。**首先，我们从启动嵌入式 Cassandra 服务器开始。为此，我们要做的就是调用`startEmbeddedCassandra()`方法。**

这将使用固定端口 9142 启动我们的数据库服务器:

```
11:13:36.754 [pool-2-thread-1] INFO  o.apache.cassandra.transport.Server
  - Starting listening for CQL clients on localhost/127.0.0.1:9142 (unencrypted)...
```

如果我们喜欢使用随机可用的端口，我们可以使用提供的卡珊德拉 YAML 配置文件:

```
EmbeddedCassandraServerHelper
  .startEmbeddedCassandra(EmbeddedCassandraServerHelper.CASSANDRA_RNDPORT_YML_FILE);
```

同样，我们也可以在启动服务器时传递我们自己的 YAML 配置文件。当然，这个文件需要在我们的类路径中。

接下来，我们可以将我们的`people.cql`数据集加载到我们的数据库中。**为此，我们使用了`ClassPathCQLDataSet` 类，它接受数据集位置和一个可选的键空间名称。**

现在我们已经加载了一些数据，我们的嵌入式服务器已经启动并运行，我们可以继续编写一个简单的单元测试:

```
@Test
public void givenEmbeddedCassandraInstance_whenStarted_thenQuerySuccess() throws Exception {
    ResultSet result = session.execute("select * from person WHERE id=1234");
    assertThat(result.iterator().next().getString("name"), is("Eugen"));
}
```

正如我们所看到的，执行一个简单的查询确认了我们的测试工作正常。厉害！我们现在有了一种使用内存中的 Cassandra 数据库来编写自包含、独立的单元测试的方法。

最后，当我们拆除测试时，我们将清理我们的嵌入式实例:

```
@After
public void tearDown() throws Exception {
    EmbeddedCassandraServerHelper.cleanEmbeddedCassandra();
}
```

执行此操作将删除除了`system` 键空间之外的所有现有键空间。

## 5。使用 CassandraUnit 抽象 JUnit 测试用例进行测试

为了帮助简化我们在上一节看到的例子，CassandraUnit 提供了一个抽象测试用例类`AbstractCassandraUnit4CQLTestCase,`,它负责我们之前看到的设置和拆除:

```
public class AbstractTestCaseWithEmbeddedCassandraUnitTest
  extends AbstractCassandraUnit4CQLTestCase {

    @Override
    public CQLDataSet getDataSet() {
        return new ClassPathCQLDataSet("people.cql", "people");
    }

    @Test
    public void givenEmbeddedCassandraInstance_whenStarted_thenQuerySuccess()
      throws Exception {
        ResultSet result = this.getSession().execute("select * from person WHERE id=1234");
        assertThat(result.iterator().next().getString("name"), is("Eugen"));
    }
}
```

这一次，通过扩展`AbstractCassandraUnit4CQLTestCase` 类，我们需要做的就是覆盖`getDataSet()`方法，该方法返回我们希望加载的`CQLDataSet`。

另一个细微的区别是，在我们的测试中，我们需要调用`getSession() `来访问 Cassandra Java 驱动程序。

## 6。使用`CassandraCQLUnit` JUnit 规则进行测试

如果我们不想强迫我们的测试扩展`AbstractCassandraUnit4CQLTestCase,` ，那么幸运的是 CassandraUnit 也提供了一个标准的 [JUnit 规则](/web/20220525141857/https://www.baeldung.com/junit-4-rules):

```
public class JUnitRuleWithEmbeddedCassandraUnitTest {

    @Rule
    public CassandraCQLUnit cassandra = new CassandraCQLUnit(new ClassPathCQLDataSet("people.cql", "people"));

    @Test
    public void givenEmbeddedCassandraInstance_whenStarted_thenQuerySuccess() throws Exception {
        ResultSet result = cassandra.session.execute("select * from person WHERE id=5678");
        assertThat(result.iterator().next().getString("name"), is("Michael"));
    }
}
```

**我们所要做的就是在测试中声明一个`CassandraCQLUnit`字段，这是一个标准的 JUnit `@Rule`。这条规则将准备和管理我们的 Cassandra 服务器的生命周期。**

## 7。使用弹簧

通常我们会在项目中集成 [Cassandra 和 Spring](/web/20220525141857/https://www.baeldung.com/spring-data-cassandra-tutorial) 。幸运的是，CassandraUnit 还支持使用 Spring TestContext 框架。

为了利用这种支持，我们需要向我们的项目添加 [`cassandra-unit-spring`](https://web.archive.org/web/20220525141857/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.cassandraunit%22%20AND%20a%3A%22cassandra-unit-spring%22) Maven 依赖项:

```
<dependency>
    <groupId>org.cassandraunit</groupId>
    <artifactId>cassandra-unit-spring</artifactId>
    <version>4.3.1.0</version>
    <scope>test</scope>
</dependency>
```

现在我们可以访问大量的注释和类，我们可以在测试中使用它们。让我们继续编写一个使用最基本的弹簧配置的测试:

```
@RunWith(SpringJUnit4ClassRunner.class)
@TestExecutionListeners({ CassandraUnitTestExecutionListener.class })
@CassandraDataSet(value = "people.cql", keyspace = "people")
@EmbeddedCassandra
public class SpringWithEmbeddedCassandraUnitTest {

    @Test
    public void givenEmbeddedCassandraInstance_whenStarted_thenQuerySuccess() throws Exception {
        CqlSession session = EmbeddedCassandraServerHelper.getSession();

        ResultSet result = session.execute("select * from person WHERE id=1234");
        assertThat(result.iterator().next().getString("name"), is("Eugen"));
    }
}
```

让我们看一下测试的关键部分。首先，我们开始用两个非常标准的 Spring 相关注释来装饰我们的测试类:

*   `@RunWith(SpringJUnit4ClassRunner.class)`注释将确保我们的测试将 Spring 的`TestContextManager`嵌入到我们的测试中，让我们能够访问 Spring 的`ApplicationContext.`
*   我们还指定了一个名为`CassandraUnitTestExecutionListener,`的自定义`TestExecutionListener,`，它负责启动和停止我们的服务器，并查找其他 CassandraUnit 注释。

**关键的部分来了；我们使用`@EmbeddedCassandra`注释将一个嵌入式 Cassandra 服务器的实例注入到我们的测试**中。此外，我们可以使用几个属性来进一步配置嵌入式数据库服务器:

*   `configuration`–不同的 Cassandra 配置文件
*   `clusterName`–集群的名称
*   我们集群的主机
*   `port`–我们集群使用的端口

这里我们保持简单，通过在声明中省略这些属性来选择默认值。

对于拼图的最后一块，我们使用`@CassandraDataSet`注释来加载我们之前看到的相同的 CQL 数据集。和以前一样，我们可以发送一个查询来验证数据库的内容是否正确。

## 8。结论

在本文中，我们学习了几种使用 Apache Cassandra 的嵌入式实例与 CassandraUnit 一起编写独立单元测试的方法。我们还讨论了如何在单元测试中使用 Spring。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220525141857/https://github.com/Baeldung/datastax-cassandra/tree/main/cassandra-unit)