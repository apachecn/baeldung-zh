# 春云连接器和 Heroku

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-heroku>

## 1。概述

在本文中，我们将介绍如何使用 Spring Cloud Connectors 在 Heroku 上设置一个 Spring Boot 应用程序。

Heroku 是一种为网络服务提供主机的服务。此外，他们还提供大量第三方服务，称为附加服务，提供从系统监控到数据库存储的一切服务。

除此之外，他们还有一个定制的 CI/CD 管道，可以无缝集成到 Git 中，从而加快产品开发。

Spring 通过它的 Spring 云连接器库支持 Heroku。我们将使用它在应用程序中自动配置 PostgreSQL 数据源。

让我们开始编写应用程序。

## 2。Spring Boot 图书服务

首先，让我们创建一个新的简单的 Spring Boot 服务。

## 3。Heroku 报名

现在，我们需要注册一个 Heroku 账户。我们去 heroku.com点击页面右上角的报名按钮。

现在我们已经有了一个帐户，我们需要获得 CLI 工具。我们需要导航到 [heroku-cli](https://web.archive.org/web/20221208143837/https://devcenter.heroku.com/articles/heroku-cli) 安装页面并安装该软件。这将为我们提供完成本教程所需的工具。

## 4。创建 Heroku 应用程序

现在我们有了 Heroku CLI，让我们回到我们的应用程序。

### 4.1。初始化 Git 存储库

Heroku 在使用 git 作为我们的源代码控制时工作得最好。

让我们从应用程序的根目录开始，这个目录与我们的`pom.xml`文件相同，运行命令`git init`来创建一个 git 存储库。然后运行`git add .`和`git commit -m “first commit”`。

现在，我们已经将应用程序保存到了本地 git 存储库中。

### 4.2。提供 Heroku Web 应用程序

接下来，让我们使用 Heroku CLI 在我们的帐户上提供一个 web 服务器。

首先，我们需要验证我们的 Heroku 帐户。**从命令行运行`heroku login`，按照说明登录并创建一个 SSH 密钥。**

接下来，运行`heroku create`。这将提供 web 服务器并添加一个远程存储库，我们可以将代码推送到该存储库中进行部署。我们还会在控制台中看到一个域，复制这个域以便我们以后可以访问它。

### 4.3。将代码推送到 Heroku

现在我们将使用 git 将代码推送到新的 Heroku 存储库。

运行命令`git push heroku master`将我们的代码发送给 Heroku。

在控制台输出中，我们应该看到指示上传成功的日志，然后他们的系统将下载任何依赖项，构建我们的应用程序，运行测试(如果有)，如果一切顺利，部署应用程序。

就这样——我们现在已经将应用程序公开部署到 web 服务器上了。

## 5。在 Heroku 上测试内存

让我们确保我们的应用程序正常工作。使用创建步骤中的域，让我们测试我们的实际应用程序。

让我们发出一些 HTTP 请求:

```java
POST https://{heroku-domain}/books HTTP
{"author":"baeldung","title":"Spring Boot on Heroku"}
```

我们应该回去:

```java
{
    "title": "Spring Boot on Heroku",
    "author": "baeldung"
}
```

现在让我们试着读取我们刚刚创建的对象:

```java
GET https://{heroku-domain}/books/1 HTTP
```

这应该会返回:

```java
{
    "id": 1,
    "title": "Spring Boot on Heroku",
    "author": "baeldung"
}
```

这看起来不错，但是在生产中，我们应该使用永久的数据存储。

让我们浏览一下提供 PostgreSQL 数据库并配置我们的 Spring 应用程序来使用它。

## 6。添加 PostgreSQL

要添加 PostgreSQL 数据库，运行以下命令`heroku addons:create heroku-postgresql:hobby-dev`

这将为我们的 web 服务器提供一个数据库，并添加一个提供连接信息的环境变量。

**Spring Cloud Connector 配置为检测这个变量，并自动设置数据源**鉴于 Spring 可以检测到我们要使用 PostgreSQL。

为了让 Spring Boot 知道我们正在使用 PostgreSQL，我们需要做两处改动。

首先，我们需要添加一个依赖项来包含 PostgreSQL 驱动程序:

```java
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.2.10</version>
</dependency>
```

接下来，让我们添加属性，以便 Spring Data Connectors 可以根据数据库的可用资源来配置数据库。

在`src/main/resources`中，创建一个 application.properties 文件，并添加以下属性:

```java
spring.datasource.driverClassName=org.postgresql.Driver
spring.datasource.maxActive=10
spring.datasource.maxIdle=5
spring.datasource.minIdle=2
spring.datasource.initialSize=5
spring.datasource.removeAbandoned=true
spring.jpa.hibernate.ddl-auto=create
```

这将汇集我们的数据库连接，并限制我们的应用程序的连接。 **Heroku 将开发层数据库中的活动连接数限制为 10 个**，因此我们将最大值设置为 10。此外，我们将 `hibernate.ddl`属性设置为 create，这样我们的 book 表将被自动创建。

最后，提交这些更改并运行`git push heroku master`。这将把这些变化推送到我们的 Heroku 应用程序上。在我们的应用程序启动后，尝试运行前一部分的测试。

我们需要做的最后一件事是更改 ddl 设置。让我们也更新这个值:

```java
spring.jpa.hibernate.ddl-auto=update
```

这将指示应用程序在应用程序重启时对实体进行更改时更新模式。像以前一样提交并推送此更改，以便将更改推送至我们的 Heroku 应用程序。

我们不需要为此编写定制的数据源集成。这是因为 Spring Cloud Connectors 检测到我们正在使用 Heroku 和 PostgreSQL 运行——并自动连接 Heroku 数据源。

## 5。结论

我们现在在 Heroku 有一个运行中的 Spring Boot 应用程序。

最重要的是，从一个想法到运行应用程序的简单性使得 Heroku 成为一种可靠的部署方式。

为了找到更多关于 Heroku 和所有工具的信息，我们可以阅读更多关于 heroku.com 的文章。

和往常一样，代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143837/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-connectors-heroku)