# AWS 无服务器应用程序模型简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/aws-serverless>

## 1。概述

在我们上一篇文章的[中，我们已经在 AWS 上实现了一个全栈无服务器应用，使用 API Gateway 作为 REST 端点，AWS Lambda 作为业务逻辑，以及 DynamoDB 作为数据库。](/web/20220625163453/https://www.baeldung.com/aws-lambda-api-gateway)

然而，部署包括许多手动步骤，随着复杂性的增加和环境数量的增加，这些步骤可能会变得不方便。

在本教程中，我们将讨论如何使用 **AWS 无服务器应用程序模型(SAM ),它支持基于模板的描述和自动部署 AWS** 上的无服务器应用程序。

具体来说，我们将了解以下主题:

*   无服务器应用程序模型(SAM)以及底层云结构的基础知识
*   使用 SAM 模板语法定义无服务器应用程序
*   使用 CloudFormation CLI 自动部署应用程序

## 2。基础知识

如前所述，通过使用 API 网关、Lambda 函数和 DynamoDB，AWS 使我们能够实现完全无服务器的应用程序。毫无疑问，这已经为性能、成本和可伸缩性提供了许多优势。

然而，缺点是，目前我们需要在 AWS 控制台中进行许多手动步骤，如创建每个函数、上传代码、创建 DynamoDB 表、创建 IAM 角色、创建 API 和 API 结构等。

对于复杂的应用程序和多种环境，如测试、试运行和生产，这种努力会迅速增加。

这就是 AWS 上应用程序的云形成和无服务器应用程序专用的无服务器应用程序模型(SAM)发挥作用的地方。

### 2.1。AWS 云组

**CloudFormation 是自动配置 AWS 基础设施资源的 AWS 服务。用户在蓝图(称为模板)中定义所有需要的资源，AWS 负责供应和配置。**

以下术语和概念对于理解 CloudFormation 和 SAM 至关重要:

**模板是对应用程序**的描述，它应该在运行时如何构造。我们可以定义一组所需的资源，以及如何配置这些资源。CloudFormation 提供了一种定义模板的通用语言，支持 JSON 和 YAML 作为一种格式。

资源是云形成的基石。资源可以是任何东西，比如 RestApi、RestApi 的一个阶段、批处理作业、DynamoDB 表、EC2 实例、网络接口、IAM 角色等等。[官方文件](https://web.archive.org/web/20220625163453/https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)目前列出了大约 300 种云形成资源类型。

堆栈是模板的实例化。 CloudFormation 负责配置和配置堆栈。

### 2.2。无服务器应用模型(SAM)

正如经常发生的那样，使用强大的工具会变得非常复杂和不方便，对于云的形成也是如此。

这就是亚马逊推出无服务器应用模型(SAM)的原因。SAM 开始声称提供了一个清晰和直接的语法来定义无服务器应用程序。目前，它只有三种资源类型，分别是 Lambda 函数、DynamoDB 表和 API。

SAM 基于 CloudFormation 模板语法，因此我们可以使用简单的 SAM 语法定义我们的模板，CloudFormation 将进一步处理该模板。

更多细节可从官方 GitHub 库以及 [AWS 文档](https://web.archive.org/web/20220625163453/https://docs.aws.amazon.com/lambda/latest/dg/serverless_app.html)中获得。

## 3。先决条件

在下面的教程中，我们需要一个 AWS 帐户。[自由级账户](https://web.archive.org/web/20220625163453/https://aws.amazon.com/free/)应该足够了。

除此之外，我们需要安装 AWS CLI。

最后，我们的地区需要一个 S3 存储桶，可以通过 AWS CLI 使用以下命令创建:

```
$>aws s3 mb s3://baeldung-sam-bucket
```

虽然本教程在下面使用了`baeldung-sam-bucket`,但是请注意，bucket 名称必须是唯一的，所以您必须选择自己的名称。

作为一个演示应用程序，我们将通过 API 网关使用 AWS Lambda 来使用[中的代码。](/web/20220625163453/https://www.baeldung.com/using-aws-lambda-with-api-gateway/)

## 4。创建模板

在本节中，我们将创建我们的 SAM 模板。

在定义单个资源之前，我们先来看看整体结构。

### 4.1。模板的结构

首先，让我们看看模板的整体结构:

```
AWSTemplateFormatVersion: '2010-09-09'
Transform: 'AWS::Serverless-2016-10-31'
Description: Baeldung Serverless Application Model example

Resources:
  PersonTable:
    Type: AWS::Serverless::SimpleTable
    Properties:
      # Define table properties here
  StorePersonFunction:
    Type: AWS::Serverless::Function
    Properties:
      # Define function properties here
  GetPersonByHTTPParamFunction:
    Type: AWS::Serverless::Function
    Properties:
      # Define function properties here
  MyApi:
    Type: AWS::Serverless::Api
    Properties:
      # Define API properties here
```

正如我们所看到的，模板由标题和主体组成:

头部指定了 CloudFormation 模板的版本(`AWSTemplateFormatVersion`)以及我们的 SAM 模板的版本(`Transform`)。我们也可以指定一个`Description`。

主体由一组资源组成:每个资源都有一个名称、一个资源`Type`和一组资源`Properties`。

SAM 规范目前支持三种类型:`AWS::Serverless::Api`、`AWS::Serverless::Function`以及`AWS::Serverless::SimpleTable`。

由于我们想要部署我们的[示例应用程序](/web/20220625163453/https://www.baeldung.com/using-aws-lambda-with-api-gateway/)，我们必须在模板体中定义一个`SimpleTable`，两个`Functions`，以及一个`Api`。

### 4.2。DynamoDB 表定义

现在让我们定义我们的 DynamoDB 表:

```
AWSTemplateFormatVersion: '2010-09-09'
Transform: 'AWS::Serverless-2016-10-31'
Description: Baeldung Serverless Application Model example

Resources:
  PersonTable:
    Type: AWS::Serverless::SimpleTable
    Properties:
      PrimaryKey:
          Name: id
          Type: Number
      TableName: Person
```

我们只需要为我们的`SimpleTable`定义两个属性:表名，以及一个主键，在我们的例子中称为`id`，类型为`Number `。

支持的`SimpleTable`属性的完整列表可以在官方规范的[中找到。](https://web.archive.org/web/20220625163453/https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlesssimpletable)

注意:因为我们只想使用主键访问表，所以`AWS::Serverless::SimpleTable`对我们来说已经足够了。对于更复杂的需求，可以使用本机 CloudFormation 类型`AWS::DynamoDB::Table`。

### 4.3。λ函数的定义

接下来，让我们定义我们的两个函数:

```
AWSTemplateFormatVersion: '2010-09-09'
Transform: 'AWS::Serverless-2016-10-31'
Description: Baeldung Serverless Application Model example

Resources:
  StorePersonFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: com.baeldung.lambda.apigateway.APIDemoHandler::handleRequest
      Runtime: java8
      Timeout: 15
      MemorySize: 512
      CodeUri: ../target/aws-lambda-0.1.0-SNAPSHOT.jar
      Policies: DynamoDBCrudPolicy
      Environment:
        Variables:
          TABLE_NAME: !Ref PersonTable
      Events:
        StoreApi:
          Type: Api
            Properties:
              Path: /persons
              Method: PUT
              RestApiId:
                Ref: MyApi
  GetPersonByHTTPParamFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: com.baeldung.lambda.apigateway.APIDemoHandler::handleGetByParam
      Runtime: java8
      Timeout: 15
      MemorySize: 512
      CodeUri: ../target/aws-lambda-0.1.0-SNAPSHOT.jar
      Policies: DynamoDBReadPolicy
      Environment:
        Variables:
          TABLE_NAME: !Ref PersonTable
      Events:
        GetByPathApi:
          Type: Api
            Properties:
              Path: /persons/{id}
              Method: GET
              RestApiId:
                Ref: MyApi
        GetByQueryApi:
          Type: Api
            Properties:
              Path: /persons
              Method: GET
              RestApiId:
                Ref: MyApi
```

正如我们所看到的，每个函数都有相同的属性:

**`Handler` 定义了函数的逻辑。**当我们使用 Java 时，它是类名，包括包，与方法名相关联。

**`Runtime` 定义了函数是如何实现的**，在我们的例子中是 Java 8。

**`Timeout`定义了在 AWS 终止执行之前，代码的执行最多需要多长时间**。

`**MemorySize**` **以 MB 为单位定义分配的内存大小。**重要的是要知道，AWS 按比例分配 CPU 资源给`MemorySize`。因此，在 CPU 密集型函数的情况下，可能需要增加`MemorySize`，即使该函数不需要那么多内存。

`**CodeUri**` **定义了功能代码的位置。**它当前引用我们本地工作区中的目标文件夹。当我们稍后使用 CloudFormation 上传我们的函数时，我们将获得一个引用 S3 对象的更新文件。

`**Policies**` **可以保存一组 AWS 管理的 IAM 策略或特定于 SAM 的策略模板。**我们对`StorePersonFunction`使用特定于 SAM 的策略`DynamoDBCrudPolicy`，对`GetPersonByPathParamFunction`和`GetPersonByQueryParamFunction`使用`DynamoDBReadPolicy`。

`**Environment**` **定义运行时的环境属性。**我们使用一个环境变量来保存 DynamoDB 表的名称。

`**Events**` **可以保存一组 AWS 事件，这些事件应该能够触发该功能。在我们的例子中，我们定义了一个类型为`Api`的`Event`。`path`、HTTP `Method`和`RestApiId`的独特组合将函数链接到我们的 API 的一个方法，我们将在下一节中定义它。**

支持的`Function`属性的完整列表可以在官方规范的[中找到。](https://web.archive.org/web/20220625163453/https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessfunction)

### 4.4。API 定义为 Swagger 文件

在定义了 DynamoDB 表和函数之后，我们现在可以定义 API 了。

第一种可能性是使用 Swagger 格式内联定义我们的 API:

```
AWSTemplateFormatVersion: '2010-09-09'
Transform: 'AWS::Serverless-2016-10-31'
Description: Baeldung Serverless Application Model example

Resources:
  MyApi:
    Type: AWS::Serverless::Api
    Properties:
      StageName: test
      EndpointConfiguration: REGIONAL
      DefinitionBody:
        swagger: "2.0"
        info:
          title: "TestAPI"
        paths:
          /persons:
            get:
              parameters:
              - name: "id"
                in: "query"
                required: true
                type: "string"
              x-amazon-apigateway-request-validator: "Validate query string parameters and\
                \ headers"
              x-amazon-apigateway-integration:
                uri:
                  Fn::Sub: arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${GetPersonByHTTPParamFunction.Arn}/invocations
                responses: {}
                httpMethod: "POST"
                type: "aws_proxy"
            put:
              x-amazon-apigateway-integration:
                uri:
                  Fn::Sub: arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${StorePersonFunction.Arn}/invocations
                responses: {}
                httpMethod: "POST"
                type: "aws_proxy"
          /persons/{id}:
            get:
              parameters:
              - name: "id"
                in: "path"
                required: true
                type: "string"
              responses: {}
              x-amazon-apigateway-integration:
                uri:
                  Fn::Sub: arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${GetPersonByHTTPParamFunction.Arn}/invocations
                responses: {}
                httpMethod: "POST"
                type: "aws_proxy"
        x-amazon-apigateway-request-validators:
          Validate query string parameters and headers:
            validateRequestParameters: true
            validateRequestBody: false
```

我们的`Api `有三个属性:`StageName`定义 API 的阶段， **`EndpointConfiguration`** 定义 API 是区域优化还是边缘优化，`DefinitionBody `包含 API 的实际结构。

在`DefinitionBody`中，我们定义了三个参数:`swagger `版本为`“2.0”`,`info:title:`为`“TestAPI”`，以及一组`paths`。

正如我们所见，`paths`代表 API 结构，我们必须在之前手动定义[。Swagger 中的`paths`相当于 AWS 控制台中的资源。就像这样，每个`path`可以有一个或多个 HTTP 动词，相当于 AWS 控制台中的方法。](/web/20220625163453/https://www.baeldung.com/using-aws-lambda-with-api-gateway#create-api)

每个方法可以有一个或多个参数以及一个请求验证器。

**最令人兴奋的部分是属性`x-amazon-apigateway-integration`，这是对 Swagger 的 AWS 专用扩展:**

`uri`指定应调用哪个 Lambda 函数。

`responses`指定如何转换函数返回的响应的规则。由于我们使用 Lambda 代理集成，我们不需要任何特定的规则。

`type` 定义了我们想要使用 Lambda 代理集成，因此我们必须将`httpMethod` 设置为`“POST”`，因为这是 Lambda 函数所期望的。

支持的`Api`属性的完整列表可以在官方规范的[中找到。](https://web.archive.org/web/20220625163453/https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessapi)

### 4.5。隐式 API 定义

第二种选择是在函数资源中隐式定义 API:

```
AWSTemplateFormatVersion: '2010-09-09'
Transform: 'AWS::Serverless-2016-10-31'
Description: Baeldung Serverless Application Model Example with Implicit API Definition

Globals:
  Api:
    EndpointConfiguration: REGIONAL
    Name: "TestAPI"

Resources:
  StorePersonFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: com.baeldung.lambda.apigateway.APIDemoHandler::handleRequest
      Runtime: java8
      Timeout: 15
      MemorySize: 512
      CodeUri: ../target/aws-lambda-0.1.0-SNAPSHOT.jar
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref PersonTable
      Environment:
        Variables:
          TABLE_NAME: !Ref PersonTable
      Events:
        StoreApi:
          Type: Api
          Properties:
            Path: /persons
            Method: PUT
  GetPersonByHTTPParamFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: com.baeldung.lambda.apigateway.APIDemoHandler::handleGetByParam
      Runtime: java8
      Timeout: 15
      MemorySize: 512
      CodeUri: ../target/aws-lambda-0.1.0-SNAPSHOT.jar
      Policies:
        - DynamoDBReadPolicy:
            TableName: !Ref PersonTable
      Environment:
        Variables:
          TABLE_NAME: !Ref PersonTable
      Events:
        GetByPathApi:
          Type: Api
          Properties:
            Path: /persons/{id}
            Method: GET
        GetByQueryApi:
          Type: Api
          Properties:
            Path: /persons
            Method: GET
```

如我们所见，我们的模板现在略有不同:不再有`AWS::Serverless::Api `资源。

**然而，CloudFormation 将类型`Api`的`Events`属性作为隐式定义，并创建了一个 API。一旦我们测试了我们的应用程序，我们将会看到它的行为与使用 Swagger 显式定义 API 时是一样的。**

此外，还有一个`Globals`部分，在这里我们可以定义 API 的名称，以及我们的端点应该是区域性的。

只有一个限制:当隐式定义 API 时，我们不能设置阶段名。这也是为什么 AWS 无论如何都会创造一个叫做`Prod`的舞台。

## 5。部署和测试

创建模板后，我们现在可以继续进行部署和测试。

为此，我们将在触发实际部署之前将我们的功能代码上传到 S3。

最后，我们可以使用任何 HTTP 客户端测试我们的应用程序。

### 5.1。代码上传到 S3

第一步，我们必须把功能代码上传到 S3。

我们可以通过 AWS CLI 调用 CloudFormation 来实现:

```
$> aws cloudformation package --template-file ./sam-templates/template.yml --s3-bucket baeldung-sam-bucket --output-template-file ./sam-templates/packaged-template.yml
```

使用这个命令，我们触发 CloudFormation 获取`CodeUri:`中指定的功能代码，并将其上传到 S3。CloudFormation 将创建一个`packaged-template.yml`文件，它具有相同的内容，除了`CodeUri:`现在指向 S3 对象。

让我们来看看 CLI 输出:

```
Uploading to 4b445c195c24d05d8a9eee4cd07f34d0 92702076 / 92702076.0 (100.00%)
Successfully packaged artifacts and wrote output template to file packaged-template.yml.
Execute the following command to deploy the packaged template
aws cloudformation deploy --template-file c:\zz_workspace\tutorials\aws-lambda\sam-templates\packaged-template.yml --stack-name <YOUR STACK NAME>
```

### 5.2。部署

现在，我们可以开始实际部署了:

```
$> aws cloudformation deploy --template-file ./sam-templates/packaged-template.yml --stack-name baeldung-sam-stack  --capabilities CAPABILITY_IAM
```

由于我们的堆栈也需要 IAM 角色(比如访问 DynamoDB 表的函数角色)，我们必须通过指定`–capabilities parameter`来明确承认这一点。

CLI 输出应该如下所示:

```
Waiting for changeset to be created..
Waiting for stack create/update to complete
Successfully created/updated stack - baeldung-sam-stack
```

### 5.3。部署审查

部署后，我们可以查看结果:

```
$> aws cloudformation describe-stack-resources --stack-name baeldung-sam-stack
```

CloudFormation 将列出所有资源，这些资源是我们堆栈的一部分。

### 5.4。测试

最后，我们可以使用任何 HTTP 客户端测试我们的应用程序。

让我们来看一些可以用于这些测试的示例`cURL`命令。

`StorePersonFunction`:

```
$> curl -X PUT 'https://0skaqfgdw4.execute-api.eu-central-1.amazonaws.com/test/persons' \
   -H 'content-type: application/json' \
   -d '{"id": 1, "name": "John Doe"}'
```

`GetPersonByPathParamFunction`:

```
$> curl -X GET 'https://0skaqfgdw4.execute-api.eu-central-1.amazonaws.com/test/persons/1' \
   -H 'content-type: application/json'
```

`GetPersonByQueryParamFunction`:

```
$> curl -X GET 'https://0skaqfgdw4.execute-api.eu-central-1.amazonaws.com/test/persons?id=1' \
   -H 'content-type: application/json'
```

### 5.5。清理

最后，我们可以通过移除堆栈和所有包含的资源来进行清理:

```
aws cloudformation delete-stack --stack-name baeldung-sam-stack
```

## 6。结论

在本文中，我们了解了 AWS 无服务器应用程序模型(SAM ),它支持基于模板的描述和自动部署 AWS 上的无服务器应用程序。

我们详细讨论了以下主题:

*   无服务器应用模型(SAM)的基础知识，以及底层云结构
*   使用 SAM 模板语法定义无服务器应用程序
*   使用 CloudFormation CLI 自动部署应用程序

像往常一样，本文的所有代码都可以在 GitHub 的[上获得。](https://web.archive.org/web/20220625163453/https://github.com/eugenp/tutorials/tree/master/aws-modules/aws-lambda)