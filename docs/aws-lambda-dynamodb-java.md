# AWS Lambda 使用 DynamoDB 和 Java

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/aws-lambda-dynamodb-java>

## 1。简介

AWS Lambda 是由亚马逊网络服务提供的无服务器计算服务，而 [WS DynamoDB](https://web.archive.org/web/20220626210605/https://aws.amazon.com/dynamodb/) 是同样由亚马逊提供的 NoSQL 数据库服务。

有趣的是，DynamoDB 支持文档存储和键值存储，并且完全由 AWS 管理。

在我们开始之前，请注意本教程需要一个有效的 AWS 帐户(您可以在这里创建一个)。另外，最好先阅读一下 [AWS Lambda with Java](/web/20220626210605/https://www.baeldung.com/java-aws-lambda) 的文章。

## 2。Maven 依赖关系

要启用 lambda，我们需要以下依赖关系，这些依赖关系可以在 [Maven Central](https://web.archive.org/web/20220626210605/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.amazonaws%22%20AND%20a%3A%22aws-lambda-java-core%22) 上找到:

```java
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-lambda-java-core</artifactId>
    <version>1.1.0</version>
</dependency> 
```

为了使用不同的 AWS 资源，我们需要以下依赖关系，这也可以在 [Maven Central](https://web.archive.org/web/20220626210605/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.amazonaws%22%20AND%20a%3A%22aws-lambda-java-events%22) 上找到:

```java
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-lambda-java-events</artifactId>
    <version>1.3.0</version>
</dependency> 
```

为了构建应用程序，我们将使用 [Maven Shade 插件](https://web.archive.org/web/20220626210605/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.maven.plugins%22%20AND%20a%3A%22maven-shade-plugin%22):

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.0.0</version>
    <configuration>
        <createDependencyReducedPom>false</createDependencyReducedPom>
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## 3。λ代码

在 lambda 应用程序中创建处理程序有不同的方式:

*   `MethodHandler`
*   `RequestHandler`
*   `RequestStreamHandler`

我们将在应用程序中使用`RequestHandler`接口。我们将接受 JSON 格式的`PersonRequest`，响应将是`PersonResponse` 也是`JSON`格式的 *:*

```java
public class PersonRequest {
    private String firstName;
    private String lastName;

    // standard getters and setters
} 
```

```java
public class PersonResponse {
    private String message;

    // standard getters and setters
}
```

接下来是我们的入口点类，它将实现`RequestHandler`接口，如下所示:

```java
public class SavePersonHandler 
  implements RequestHandler<PersonRequest, PersonResponse> {

    private DynamoDB dynamoDb;
    private String DYNAMODB_TABLE_NAME = "Person";
    private Regions REGION = Regions.US_WEST_2;

    public PersonResponse handleRequest(
      PersonRequest personRequest, Context context) {

        this.initDynamoDbClient();

        persistData(personRequest);

        PersonResponse personResponse = new PersonResponse();
        personResponse.setMessage("Saved Successfully!!!");
        return personResponse;
    }

    private PutItemOutcome persistData(PersonRequest personRequest) 
      throws ConditionalCheckFailedException {
        return this.dynamoDb.getTable(DYNAMODB_TABLE_NAME)
          .putItem(
            new PutItemSpec().withItem(new Item()
              .withString("firstName", personRequest.getFirstName())
              .withString("lastName", personRequest.getLastName());
    }

    private void initDynamoDbClient() {
        AmazonDynamoDBClient client = new AmazonDynamoDBClient();
        client.setRegion(Region.getRegion(REGION));
        this.dynamoDb = new DynamoDB(client);
    }
} 
```

这里，当我们实现`RequestHandler`接口时，我们需要实现`handleRequest()`来实际处理请求。至于代码的其余部分，我们有:

*   `PersonRequest`对象——包含以 JSON 格式传递的请求值
*   `Context`对象–用于从 lambda 执行环境中获取信息
*   哪个是 lambda 请求的响应对象

当创建 DynamoDB 对象时，我们将首先创建`AmazonDynamoDBClient`对象，并使用它来创建`DynamoDB`对象。注意 `region`是强制的。

为了在 DynamoDB 表中添加条目，我们将利用一个`PutItemSpec`对象——通过指定列数及其值。

在 DynamoDB 表中我们不需要任何预定义的模式，我们只需要定义主键列名，在我们的例子中是`**“id”**`。

## 4。构建部署文件

为了构建 lambda 应用程序，我们需要执行以下 Maven 命令:

```java
mvn clean package shade:shade
```

Lambda 应用程序将被编译并打包成目标文件夹下的一个`jar`文件。

## 5。创建 DynamoDB 表

按照以下步骤创建 DynamoDB 表:

*   登录 [AWS 账户](https://web.archive.org/web/20220626210605/https://aws.amazon.com/)
*   点击**“DynamoDB”**，可以在**“所有服务”**下找到
*   该页面将显示已经创建的 DynamoDB 表(如果有)
*   点击**【创建表格】**按钮
*   提供**【表名】****【主键】**，其数据类型为**【数字】**
*   点击**“创建”**按钮
*   将创建表格

## 6。创建 Lambda 函数

按照以下步骤创建 Lambda 函数:

*   登录 [AWS 账户](https://web.archive.org/web/20220626210605/https://aws.amazon.com/)
*   点击**【所有服务】**下的**【λ】**
*   该页面将显示已创建的 lambda 函数(如果有)或未创建 Lambda 函数。点击**“立即开始”**
*   **【选择蓝图】** - >选择**空白功能**
*   **【配置触发器】** - >点击**“下一步”**按钮
*   **“配置功能”**
    *   **【姓名】**:救人
    *   **【描述】**:救人到 DDB
    *   **“运行时”**:选择**“Java 8”**
    *   **【上传】**:点击**【上传】**按钮，选择 lambda 应用的 jar 文件
*   **【处理程序】**:com . BAE message . lambda . dynamodb . save personhandler
*   **“角色”**:选择**“创建自定义角色”**
*   将弹出一个新窗口，允许为 lambda 执行配置 IAM 角色，我们需要在其中添加 DynamoDB 授权。完成后，点击**“允许”**按钮
*   点击**【下一步】**按钮
*   **【查看】**:查看配置
*   点击**“创建功能”**按钮

## 7。测试λ函数

下一步是测试 lambda 函数:

*   点击**“测试”**按钮
*   将显示**“输入测试事件”**窗口。这里，我们将为我们的请求提供 JSON 输入:

```java
{
  "id": 1,
  "firstName": "John",
  "lastName": "Doe",
  "age": 30,
  "address": "United States"
}
```

*   点击**“保存并测试”**或**“保存”**按钮
*   在**“执行结果”**部分可以看到输出:

```java
{
  "message": "Saved Successfully!!!"
}
```

*   我们还需要在 DynamoDB 中检查记录是否被持久化:
    *   进入**“dynamo db”管理控制台**
    *   选择表格**“人”**
    *   选择**【物品】**标签
    *   在这里你可以看到这个人的详细信息，这些信息被请求传递给 lambda 应用程序
*   所以我们的 lambda 应用程序成功地处理了请求

## 8。结论

在这篇简短的文章中，我们学习了如何用 DynamoDB 和 Java 8 创建 Lambda 应用程序。详细的说明会让你在设置一切时有一个良好的开端。

和往常一样，示例应用程序的完整源代码可以在 Github 的[中找到。](https://web.archive.org/web/20220626210605/https://github.com/eugenp/tutorials/tree/master/aws-modules/aws-lambda)