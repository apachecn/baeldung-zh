# 使用本地 DynamoDB 实例进行集成测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/dynamodb-local-integration-tests>

## 1。概述

如果我们开发一个使用 Amazon[dynamo db](https://web.archive.org/web/20221129013623/https://aws.amazon.com/dynamodb/)的应用程序，在没有本地实例的情况下开发集成测试可能会很棘手。

在本教程中，**我们将探索为我们的集成测试**配置、启动和停止本地 DynamoDB 的多种方式。

本教程还补充了我们现有的 [DynamoDB 文章](/web/20221129013623/https://www.baeldung.com/spring-data-dynamodb)。

## 2。配置

### 2.1。Maven 设置

[DynamoDB Local](https://web.archive.org/web/20221129013623/https://aws.amazon.com/blogs/aws/dynamodb-local-for-desktop-development/) 是亚马逊开发的工具，支持所有 DynamoDB APIs。它不直接操作生产中的实际 DynamoDB 表，而是在本地执行。

首先，我们将 DynamoDB 本地依赖项添加到 Maven 配置的依赖项列表中:

```
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>DynamoDBLocal</artifactId>
    <version>1.11.86</version>
    <scope>test</scope>
</dependency> 
```

接下来，我们还需要添加 Amazon DynamoDB 存储库，因为 Maven Central 存储库中不存在依赖关系。

我们可以选择离我们当前 IP 地址地理位置最近的亚马逊服务器:

```
<repository>
    <id>dynamodb-local</id>
    <name>DynamoDB Local Release Repository</name>
    <url>https://s3-us-west-2.amazonaws.com/dynamodb-local/release</url>
</repository>
```

### 2.2。添加 SQLite4Java 依赖关系

DynamoDB Local 内部使用 [SQLite4Java](https://web.archive.org/web/20221129013623/https://bitbucket.org/almworks/sqlite4java) 库；因此，当我们运行测试时，我们还需要包含库文件。SQLite4Java 库文件依赖于测试运行的环境，但是一旦我们声明了 DynamoDBLocal 依赖项，Maven 就可以以过渡方式提取它们。

接下来，我们需要添加一个新的构建步骤，将本地库复制到一个特定的文件夹中，稍后我们将在 JVM 系统属性中定义该文件夹。

让我们将传递提取的 SQLite4Java 库文件复制到一个名为`native-libs`的文件夹中:

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>2.10</version>
    <executions>
        <execution>
            <id>copy</id>
            <phase>test-compile</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <includeScope>test</includeScope>
                <includeTypes>so,dll,dylib</includeTypes>
                <outputDirectory>${project.basedir}/native-libs</outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin> 
```

### 2.3。设置 SQLite4Java 系统属性

现在，我们将使用名为`sqlite4java.library.path`的 JVM 系统属性引用先前创建的文件夹(SQLite4Java 库所在的位置):

```
System.setProperty("sqlite4java.library.path", "native-libs");
```

为了稍后成功运行测试，必须将所有的 SQLite4Java 库放在由`sqlite4java.library.path`系统属性定义的文件夹中。**我们必须运行 Maven test-compile ( `mvn test-compile`)至少一次**来满足先决条件。

## 3。设置测试数据库的生命周期

我们可以定义代码，在用`@BeforeClass;`标注的 setup 方法中创建和启动本地 DynamoDB 服务器，并且对称地，在用`@AfterClass`标注的 teardown 方法中停止服务器。

在以下示例中，我们将在端口 8000 上启动本地 DynamoDB 服务器，并确保它在运行我们的测试后再次停止:

```
public class ProductInfoDAOIntegrationTest {
    private static DynamoDBProxyServer server;

    @BeforeClass
    public static void setupClass() throws Exception {
        System.setProperty("sqlite4java.library.path", "native-libs");
        String port = "8000";
        server = ServerRunner.createServerFromCommandLineArgs(
          new String[]{"-inMemory", "-port", port});
        server.start();
        //...
    }

    @AfterClass
    public static void teardownClass() throws Exception {
        server.stop();
    }

    //...
}
```

我们还可以使用`java.net.ServerSocket`在任何可用端口上运行本地 DynamoDB 服务器，而不是在固定端口上运行。在这种情况下，**我们还必须配置测试，将端点设置到正确的 DynamoDB 端口**:

```
public String getAvailablePort() throws IOException {
    ServerSocket serverSocket = new ServerSocket(0);
    return String.valueOf(serverSocket.getLocalPort());
}
```

## 4。替代方法:使用`@ClassRule`

我们可以将前面的逻辑包装在一个执行相同操作的 JUnit 规则中:

```
public class LocalDbCreationRule extends ExternalResource {
    private DynamoDBProxyServer server;

    public LocalDbCreationRule() {
        System.setProperty("sqlite4java.library.path", "native-libs");
    }

    @Override
    protected void before() throws Exception {
        String port = "8000";
        server = ServerRunner.createServerFromCommandLineArgs(
          new String[]{"-inMemory", "-port", port});
        server.start();
    }

    @Override
    protected void after() {
        this.stopUnchecked(server);
    }

    protected void stopUnchecked(DynamoDBProxyServer dynamoDbServer) {
        try {
            dynamoDbServer.stop();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }    
    }
}
```

要使用我们的自定义规则，我们必须创建一个实例并用`@ClassRule`进行注释，如下所示。同样，测试将在测试类初始化之前创建并启动本地 DynamoDB 服务器。

注意，为了运行测试，测试规则的访问修饰符必须是`public` :

```
public class ProductInfoRepositoryIntegrationTest {
    @ClassRule
    public static LocalDbCreationRule dynamoDB = new LocalDbCreationRule();

    //...
}
```

在结束之前，有一个简短的注意事项——由于 DynamoDB Local 在内部使用 SQLite 数据库，其性能并不能反映生产中的真实性能。

## 5。结论

在本文中，我们看到了如何设置和配置 DynamoDB Local 来运行集成测试。

和往常一样，源代码和配置示例可以在 Github 上找到[。](https://web.archive.org/web/20221129013623/https://github.com/eugenp/tutorials/tree/master/aws-modules/aws-miscellaneous)