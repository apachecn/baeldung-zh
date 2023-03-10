# 将 AWS Lambda 与 API 网关一起使用

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/aws-lambda-api-gateway>

## 1。概述

[AWS Lambda](https://web.archive.org/web/20220526045901/https://aws.amazon.com/lambda/) 是亚马逊网络服务提供的无服务器计算服务。

在之前的两篇文章中，我们讨论了[如何使用 Java](/web/20220526045901/https://www.baeldung.com/java-aws-lambda) 创建 AWS Lambda 函数，以及[如何从 Lambda 函数](/web/20220526045901/https://www.baeldung.com/aws-lambda-dynamodb-java)访问 DynamoDB。

在本教程中，我们将讨论**如何使用 [AWS 网关](https://web.archive.org/web/20220526045901/https://aws.amazon.com/api-gateway/)** 发布一个 Lambda 函数作为 REST 端点。

我们将详细了解以下主题:

*   API 网关的基本概念和术语
*   使用 Lambda 代理集成将 Lambda 函数与 API 网关集成
*   API 的创建，它的结构，以及如何将 API 资源映射到 Lambda 函数上
*   API 的部署和测试

## 2。基础和术语

API Gateway 是一个**完全托管的服务，支持开发人员创建、发布、维护、监控和保护任何规模的 API**。

我们可以实现一个一致且可扩展的基于 HTTP 的编程接口(也称为 RESTful 服务)**来访问后端服务，如 Lambda 函数、更多 AWS 服务(如 EC2、S3、DynamoDB)和任何 HTTP 端点**。

功能包括但不限于:

*   交通管理
*   授权和访问控制
*   监视
*   API 版本管理
*   限制请求以防止攻击

像 AWS Lambda 一样，API Gateway 是自动扩展的，并按 API 调用计费。

详细信息可以在[官方文档](https://web.archive.org/web/20220526045901/https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html)中找到。

### 2.1。条款

`**API Gateway**`是一种 AWS 服务，支持创建、部署和管理 RESTful 应用程序编程接口，以公开后端 HTTP 端点、AWS Lambda 函数和其他 AWS 服务。

一个`**API Gateway API**`是资源和方法的集合，可以与 Lambda 函数、其他 AWS 服务或后端的 HTTP 端点集成。API 由构成 API 结构的资源组成。每个 API 资源可以公开一个或多个 API 方法，这些方法必须具有唯一的 HTTP 谓词。

要发布一个 API，我们必须创建一个`**API deployment**`并将它与一个所谓的`stage`相关联。阶段就像 API 的时间快照。如果我们重新部署一个 API，我们可以更新一个现有的阶段或者创建一个新的阶段。这样，一个 API 可能同时有不同的版本，例如一个`dev`阶段，一个`test`阶段，甚至多个生产版本，如`v1`、`v2`等。

`Lambda Proxy integration`是 Lambda 函数和 API 网关之间集成的简化配置。

API 网关将整个请求作为输入发送给后端 Lambda 函数。在响应方面，API Gateway 将 Lambda 函数输出转换回前端 HTTP 响应。

## 3。依赖性

我们将需要与文章 [AWS Lambda 使用 DynamoDB With Java](/web/20220526045901/https://www.baeldung.com/aws-lambda-dynamodb-java) 中相同的依赖关系。

除此之外，我们还需要简单的 JSON 库:

```java
<dependency>
    <groupId>com.googlecode.json-simple</groupId>
    <artifactId>json-simple</artifactId>
    <version>1.1.1</version>
</dependency>
```

## 4。开发和部署 Lambda 功能

在这一节中，我们将使用 Java 开发和构建我们的 Lambda 函数，我们将使用 AWS 控制台部署它，并且我们将运行一个快速测试。

因为我们想要演示将 API Gateway 与 Lambda 集成的基本功能，所以我们将创建两个函数:

*   **函数 1:** 使用 PUT 方法从 API 接收有效载荷
*   **函数 2:** 演示如何使用来自 API 的 HTTP 路径参数或 HTTP 查询参数

在实现方面，我们将创建一个`RequestHandler`类，它有两个方法——每个函数一个方法。

### 4.1。型号

在我们实现实际的请求处理程序之前，让我们快速看一下我们的数据模型:

```java
public class Person {

    private int id;
    private String name;

    public Person(String json) {
        Gson gson = new Gson();
        Person request = gson.fromJson(json, Person.class);
        this.id = request.getId();
        this.name = request.getName();
    }

    public String toString() {
        Gson gson = new GsonBuilder().setPrettyPrinting().create();
        return gson.toJson(this);
    }

    // getters and setters
}
```

我们的模型由一个简单的`Person`类组成，它有两个属性。唯一值得注意的部分是`Person(String)`构造函数，它接受一个 JSON 字符串。

### 4.2。RequestHandler 类的实现

就像在文章 [AWS Lambda With Java](/web/20220526045901/https://www.baeldung.com/java-aws-lambda#handler) 中一样，我们将创建一个`RequestStreamHandler`接口的实现:

```java
public class APIDemoHandler implements RequestStreamHandler {

    private static final String DYNAMODB_TABLE_NAME = System.getenv("TABLE_NAME"); 

    @Override
    public void handleRequest(
      InputStream inputStream, OutputStream outputStream, Context context)
      throws IOException {

        // implementation
    }

    public void handleGetByParam(
      InputStream inputStream, OutputStream outputStream, Context context)
      throws IOException {

        // implementation
    }
}
```

正如我们所见，`RequestStreamHander`接口只定义了一个方法，`handeRequest()`。无论如何，我们可以在同一个类中定义更多的函数，就像我们在这里做的那样。另一种选择是为每个函数创建一个`RequestStreamHander`的实现。

在我们的具体案例中，为了简单起见，我们选择了前者。但是，必须根据具体情况进行选择，考虑性能和代码可维护性等因素。

我们还从`TABLE_NAME `环境变量中读取了 DynamoDB 表的名称。我们将在稍后的部署过程中定义该变量。

### 4.3。功能 1 的实现

在我们的第一个函数中，我们想要演示**如何从 API 网关**获取有效载荷(比如从 PUT 或 POST 请求中获取):

```java
public void handleRequest(
  InputStream inputStream, 
  OutputStream outputStream, 
  Context context)
  throws IOException {

    JSONParser parser = new JSONParser();
    BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
    JSONObject responseJson = new JSONObject();

    AmazonDynamoDB client = AmazonDynamoDBClientBuilder.defaultClient();
    DynamoDB dynamoDb = new DynamoDB(client);

    try {
        JSONObject event = (JSONObject) parser.parse(reader);

        if (event.get("body") != null) {
            Person person = new Person((String) event.get("body"));

            dynamoDb.getTable(DYNAMODB_TABLE_NAME)
              .putItem(new PutItemSpec().withItem(new Item().withNumber("id", person.getId())
                .withString("name", person.getName())));
        }

        JSONObject responseBody = new JSONObject();
        responseBody.put("message", "New item created");

        JSONObject headerJson = new JSONObject();
        headerJson.put("x-custom-header", "my custom header value");

        responseJson.put("statusCode", 200);
        responseJson.put("headers", headerJson);
        responseJson.put("body", responseBody.toString());

    } catch (ParseException pex) {
        responseJson.put("statusCode", 400);
        responseJson.put("exception", pex);
    }

    OutputStreamWriter writer = new OutputStreamWriter(outputStream, "UTF-8");
    writer.write(responseJson.toString());
    writer.close();
}
```

**如前所述，我们稍后将配置 API 以使用 Lambda 代理集成。我们期望 API 网关将完整的请求传递给参数`InputStream`中的 Lambda 函数。**

我们所要做的就是从包含的 JSON 结构中挑选相关的属性。

正如我们所看到的，该方法基本上由三个步骤组成:

1.  从我们的输入流中获取`body`对象，并从中创建一个`Person`对象
2.  将那个`Person`对象存储在 DynamoDB 表中
3.  构建一个 JSON 对象，它可以保存几个属性，比如响应的`body`、定制头以及 HTTP 状态代码

这里值得一提的一点是:API Gateway 期望`body`是一个`String`(对于请求和响应)。

由于我们期望从 API 网关获得一个`String`作为`body`，我们将`body`转换为`String`并初始化我们的`Person`对象:

```java
Person person = new Person((String) event.get("body"));
```

API Gateway 还期望响应`body`是一个`String`:

```java
responseJson.put("body", responseBody.toString());
```

官方文档中没有明确提到这个主题。然而，如果我们仔细观察，我们会发现请求的片段[和响应](https://web.archive.org/web/20220526045901/https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-input-format)的片段[中的 body 属性都是`String`。](https://web.archive.org/web/20220526045901/https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-output-format)

优点应该是显而易见的:即使 JSON 是 API Gateway 和 Lambda 函数之间的格式，实际的主体也可以包含纯文本、JSON、XML 或其他任何东西。然后，正确处理格式是 Lambda 函数的责任。

稍后当我们在 AWS 控制台中测试我们的功能时，我们将看到请求和响应主体是什么样子的。

这同样适用于以下两个函数。

### 4.4。功能 2 的实现

在第二步中，我们想要演示**如何使用路径参数或查询字符串参数**通过 ID 从数据库中检索`Person`项:

```java
public void handleGetByParam(
  InputStream inputStream, OutputStream outputStream, Context context)
  throws IOException {

    JSONParser parser = new JSONParser();
    BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
    JSONObject responseJson = new JSONObject();

    AmazonDynamoDB client = AmazonDynamoDBClientBuilder.defaultClient();
    DynamoDB dynamoDb = new DynamoDB(client);

    Item result = null;
    try {
        JSONObject event = (JSONObject) parser.parse(reader);
        JSONObject responseBody = new JSONObject();

        if (event.get("pathParameters") != null) {
            JSONObject pps = (JSONObject) event.get("pathParameters");
            if (pps.get("id") != null) {
                int id = Integer.parseInt((String) pps.get("id"));
                result = dynamoDb.getTable(DYNAMODB_TABLE_NAME).getItem("id", id);
            }
        } else if (event.get("queryStringParameters") != null) {
            JSONObject qps = (JSONObject) event.get("queryStringParameters");
            if (qps.get("id") != null) {

                int id = Integer.parseInt((String) qps.get("id"));
                result = dynamoDb.getTable(DYNAMODB_TABLE_NAME)
                  .getItem("id", id);
            }
        }
        if (result != null) {
            Person person = new Person(result.toJSON());
            responseBody.put("Person", person);
            responseJson.put("statusCode", 200);
        } else {
            responseBody.put("message", "No item found");
            responseJson.put("statusCode", 404);
        }

        JSONObject headerJson = new JSONObject();
        headerJson.put("x-custom-header", "my custom header value");

        responseJson.put("headers", headerJson);
        responseJson.put("body", responseBody.toString());

    } catch (ParseException pex) {
        responseJson.put("statusCode", 400);
        responseJson.put("exception", pex);
    }

    OutputStreamWriter writer = new OutputStreamWriter(outputStream, "UTF-8");
    writer.write(responseJson.toString());
    writer.close();
}
```

同样，三个步骤是相关的:

1.  我们检查具有`id`属性的`pathParameters`或`queryStringParameters`数组是否存在。
2.  如果是`true`，我们使用所属值从数据库中请求一个具有该 ID 的`Person`条目。
3.  我们将接收到的项目的 JSON 表示添加到响应中。

官方文档对代理集成的[输入格式](https://web.archive.org/web/20220526045901/https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-input-format)和[输出格式](https://web.archive.org/web/20220526045901/https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-output-format)提供了更详细的解释。

### 4.5。建筑规范

同样，我们可以使用 Maven 简单地构建我们的代码:

```java
mvn clean package shade:shade
```

JAR 文件将被创建在`target`文件夹下。

### 4.6。创建 DynamoDB 表

我们可以使用 DynamoDB 和 Java 创建表，如 [AWS Lambda 中所述。](/web/20220526045901/https://www.baeldung.com/aws-lambda-dynamodb-java#dynamodb-table)

让我们选择`Person`作为表名，`id`作为主键名，`Number`作为主键的类型。

### 4.7。通过 AWS 控制台部署代码

在构建代码和创建表之后，我们现在可以创建函数并上传代码。

这可以通过重复文章 [AWS Lambda with Java](/web/20220526045901/https://www.baeldung.com/java-aws-lambda#lambda-console) 中的步骤 1-5 来完成，对我们的两种方法各重复一次。

让我们使用以下函数名:

*   `StorePersonFunction`为`handleRequest`方法(功能 1)
*   `GetPersonByHTTPParamFunction `为`handleGetByParam`方法(功能 2)

我们还必须定义一个值为`“Person”`的环境变量`TABLE_NAME`。

### 4.8。测试功能

在继续实际的 API 网关部分之前，我们可以在 AWS 控制台中运行一个快速测试，只是为了检查我们的 Lambda 函数是否正确运行，以及是否可以处理代理集成格式。

从 AWS 控制台测试 Lambda 函数的工作方式如 [AWS Lambda with Java](/web/20220526045901/https://www.baeldung.com/java-aws-lambda#invoke) 文章所述。

然而，**当我们创建一个测试事件时，我们必须考虑特殊的代理集成格式**，这是我们的函数所期望的。我们可以使用`API Gateway AWS Proxy`模板并根据我们的需要进行定制，或者我们可以复制并粘贴以下事件:

对于`StorePersonFunction`，我们应该用这个:

```java
{
    "body": "{\"id\": 1, \"name\": \"John Doe\"}"
}
```

如前所述，`body`必须具有类型`String`，即使包含 JSON 结构。原因是 API 网关将以相同的格式发送请求。

应该返回以下响应:

```java
{
    "isBase64Encoded": false,
    "headers": {
        "x-custom-header": "my custom header value"
    },
    "body": "{\"message\":\"New item created\"}",
    "statusCode": 200
}
```

这里，我们可以看到我们的响应的`body`是一个`String`，尽管它包含一个 JSON 结构。

让我们看看`GetPersonByHTTPParamFunction.`的输入

为了测试路径参数功能，输入如下所示:

```java
{
    "pathParameters": {
        "id": "1"
    }
}
```

并且用于发送查询字符串参数的输入将是:

```java
{
    "queryStringParameters": {
        "id": "1"
    }
}
```

作为响应，对于这两种情况，我们应该得到以下方法:

```java
{
  "headers": {
    "x-custom-header": "my custom header value"
  },
  "body": "{\"Person\":{\n  \"id\": 88,\n  \"name\": \"John Doe\"\n}}",
  "statusCode": 200
}
```

同样，`body`是一个`String`。

## 5。创建和测试 API

在前面的章节**中创建和部署了 Lambda 函数之后，我们现在可以使用 AWS 控制台**创建实际的 API。

让我们看看基本的工作流程:

1.  在我们的 AWS 帐户中创建一个 API。
2.  将资源添加到 API 的资源层次结构中。
3.  为资源创建一个或多个方法。
4.  设置方法和所属 Lambda 函数之间的集成。

在接下来的章节中，我们将对我们的两个功能分别重复步骤 2-4。

### 5.1。创建 API

为了创建 API，我们必须:

1.  在[https://console.aws.amazon.com/apigateway](https://web.archive.org/web/20220526045901/https://console.aws.amazon.com/apigateway)登录 API 网关控制台
2.  点击“开始”，然后选择“新 API”
3.  键入我们的 API 的名称(`TestAPI`)并点击“创建 API”进行确认

创建了 API 之后，我们现在可以创建 API 结构并将其链接到我们的 Lambda 函数。

### 5.2。函数 1 的 API 结构

以下步骤对于我们的`StorePersonFunction`是必要的:

1.  选择“资源”树下的父资源项目，然后从“操作”下拉菜单中选择“创建资源”。然后，我们必须在“新建子资源”窗格中执行以下操作:
    *   在“资源名称”输入文本字段中键入“人员”作为名称
    *   在“资源路径”输入文本字段中保留默认值
    *   选择“创建资源”
2.  选择刚刚创建的资源，从“Actions”下拉菜单中选择“Create Method ”,并执行以下步骤:
    *   从 HTTP 方法下拉列表中选择上传，然后选择复选标记图标保存选择
    *   保留“Lambda 函数”作为集成类型，并选择“使用 Lambda 代理集成”选项
    *   从“Lambda Region”中选择区域，之前我们在这里部署了 Lambda 函数
    *   在“Lambda 函数”中键入`“StorePersonFunction”`
3.  选择“保存”,并在提示“添加 Lambda 函数权限”时确认“确定”

### 5.3。 **函数 2 的 API 结构——路径参数**

检索路径参数的步骤是相似的:

1.  选择“资源”树下的`/` `persons`资源项，然后从“操作”下拉菜单中选择“创建资源”。然后，我们必须在新的子资源窗格中执行以下操作:
    *   在“资源名称”输入文本字段中键入`“Person”`作为名称
    *   将“资源路径”输入文本字段更改为`“{id}”`
    *   选择“创建资源”
2.  选择刚刚创建的资源，从“Actions”下拉菜单中选择“Create Method ”,并执行以下步骤:
    *   从 HTTP 方法下拉列表中选择获取，然后选择复选标记图标保存选择
    *   保留“Lambda 函数”作为集成类型，并选择“使用 Lambda 代理集成”选项
    *   从“Lambda Region”中选择区域，之前我们在这里部署了 Lambda 函数
    *   在“Lambda 函数”中键入`“GetPersonByHTTPParamFunction”`
3.  选择“保存”,并在提示“添加 Lambda 函数权限”时确认“确定”

注意:这里将“资源路径”参数设置为`“{id}”`是很重要的，因为我们的`GetPersonByPathParamFunction `希望该参数的命名完全像这样。

### 5.4。 **函数 2 的 API 结构——查询字符串参数**

接收查询字符串参数的步骤略有不同，因为**我们不必创建资源，而是必须为`id`参数**创建一个查询参数:

1.  选择“Resources”树下的`/persons`资源项，从“Actions”下拉菜单中选择“Create Method ”,执行以下步骤:
    *   从 HTTP 方法下拉列表中选择获取，然后选择复选标记图标保存选择
    *   保留“Lambda 函数”作为集成类型，并选择“使用 Lambda 代理集成”选项
    *   从“Lambda Region”中选择区域，之前我们在这里部署了 Lambda 函数
    *   在“Lambda Function”中键入`“GetPersonByHTTPParamFunction”`。
2.  选择“保存”,并在提示“添加 Lambda 函数权限”时确认“确定”
3.  选择右侧的“方法请求”并执行以下步骤:
    *   展开 URL 查询字符串参数列表
    *   点击“添加查询字符串”
    *   在名称字段中键入`“id”`，并选择复选标记图标进行保存
    *   选择“必需”复选框
    *   单击面板顶部“请求验证器”旁边的钢笔符号，选择“验证查询字符串参数和标题”，然后选择复选标记图标

注意:将“查询字符串”参数设置为`“id”`是很重要的，因为我们的`GetPersonByHTTPParamFunction `希望该参数的命名与此完全相同。

### 5.5。测试 API

我们的 API 现在已经准备好了，但是还没有公开。在我们发布它之前，我们想先从控制台运行一个快速测试。

为此，我们可以在“Resources”树中选择要测试的相应方法，然后单击“Test”按钮。在下面的屏幕上，我们可以键入我们的输入，就像我们通过 HTTP 用客户机发送它一样。

对于`StorePersonFunction`，我们必须在“请求正文”字段中键入以下结构:

```java
{
    "id": 2,
    "name": "Jane Doe"
}
```

对于带有路径参数的`GetPersonByHTTPParamFunction` ,我们必须将`2`作为一个值输入到“path”下的“{id}”字段中。

对于带有查询字符串参数的`GetPersonByHTTPParamFunction `,我们必须在“查询字符串”下的“{persons}”字段中输入值`id=2`。

### 5.6。部署 API

到目前为止，我们的 API 还没有公开，因此只能从 AWS 控制台获得。

如前所述，**当我们部署一个 API 时，我们必须将它与一个阶段相关联，这就像 API 的时间快照。如果我们重新部署一个 API，我们可以更新一个现有的 stage 或者创建一个新的 stage**。

让我们看看我们的 API 的 URL 方案是什么样子的:

```java
https://{restapi-id}.execute-api.{region}.amazonaws.com/{stageName}
```

部署需要以下步骤:

1.  在“API”导航窗格中选择特定的 API
2.  在资源导航窗格中选择“操作”,并从“操作”下拉菜单中选择“部署 API”
3.  从“部署阶段”下拉列表中选择“[新阶段]”，在“阶段名称”中键入`“test”`，并可选地提供阶段和部署的描述
4.  通过选择“部署”来触发部署

最后一步之后，控制台会提供 API 的根 URL，例如`https://0skaqfgdw4.execute-api.eu-central-1.amazonaws.com/test`。

### 5.7。调用端点

由于 API 现在是公共的，**我们可以使用任何我们想要的 HTTP 客户端调用它**。

使用`cURL`，调用将如下所示。

`StorePersonFunction`:

```java
curl -X PUT 'https://0skaqfgdw4.execute-api.eu-central-1.amazonaws.com/test/persons' \
  -H 'content-type: application/json' \
  -d '{"id": 3, "name": "Richard Roe"}'
```

`GetPersonByHTTPParamFunction `对于路径参数:

```java
curl -X GET 'https://0skaqfgdw4.execute-api.eu-central-1.amazonaws.com/test/persons/3' \
  -H 'content-type: application/json'
```

`GetPersonByHTTPParamFunction` 对于查询字符串参数:

```java
curl -X GET 'https://0skaqfgdw4.execute-api.eu-central-1.amazonaws.com/test/persons?id=3' \
  -H 'content-type: application/json'
```

## 6。结论

在本文中，我们了解了如何使用 AWS API Gateway 使 AWS Lambda 函数作为 REST 端点可用。

我们探讨了 API Gateway 的基本概念和术语，并学习了如何使用 Lambda 代理集成来集成 Lambda 函数。

最后，我们看到了如何创建、部署和测试 API。

像往常一样，本文的所有代码都可以在 GitHub 的[上获得。](https://web.archive.org/web/20220526045901/https://github.com/eugenp/tutorials/tree/master/aws-modules/aws-lambda)