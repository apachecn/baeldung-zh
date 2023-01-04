# 构建基本的 UAA 安全 JHipster 微服务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jhipster-uaa-secured-micro-service>

## 1.概观

在之前的文章中，我们已经介绍了 JHipster 的[基础知识，以及如何使用它来生成基于](/web/20220628145118/https://www.baeldung.com/jhipster)[微服务的应用](/web/20220628145118/https://www.baeldung.com/jhipster-microservices)。

在本教程中，**我们将探索 JHipster 的用户账户和授权服务**——简称 UAA——以及如何使用它来保护一个完全成熟的基于 jhi pster 的微服务应用。更好的是，这一切都可以在**不用写一行代码**的情况下实现！

## 2.UAA 的核心特征

我们在以前的文章中构建的应用程序的一个重要特征是用户帐户是它们不可或缺的一部分。现在，当我们只有一个应用程序时，这很好，但是如果我们想要在多个 JHipster 生成的应用程序之间共享用户帐户呢？这就是吉普斯特的 UAA 的用武之地。

JHipster 的 UAA 是一种微服务，它是独立于我们应用程序中的其他服务而构建、部署和运行的。它的作用是:

*   基于 Spring Boot 实现的 OAuth2 授权服务器
*   身份管理服务器，公开用户帐户 CRUD API

JHipster UAA 还支持自助注册和“记住我”等典型的登录功能。当然，它完全集成了 JHipster 的其他服务。

## 3.开发环境设置

在开始任何开发之前，我们必须首先确保我们的环境已经设置好了所有的先决条件。除了我们的[JHipster 简介文章](/web/20220628145118/https://www.baeldung.com/jhipster)中描述的所有工具之外，我们还需要一个运行中的 JHipster 注册中心。简单回顾一下，注册服务允许我们将要创建的不同服务互相查找和交流。

生成和运行注册表的完整过程在我们的微服务架构文章的 [JHipster 的第 4.1 节中有所描述，因此我们在此不再赘述。一个](/web/20220628145118/https://www.baeldung.com/jhipster-microservices) [Docker 图像也是可用的](https://web.archive.org/web/20220628145118/https://hub.docker.com/r/jhipster/jhipster-registry)，可以作为一个替代。

## 4.生成新的捷普斯特 UAA 服务

让我们使用 JHipster 命令行实用程序来生成我们的 UAA 服务:

```
$ mkdir uaa
$ cd uaa
$ jhipster 
```

我们必须回答的第一个问题是我们想要生成哪种类型的应用程序。使用箭头键，我们将选择“JHipster UAA(用于微服务 OAuth2 身份验证)”选项:

[![app type](img/6fc514634a670fdbe6d75a7744b5409a.png)](/web/20220628145118/https://www.baeldung.com/wp-content/uploads/2018/12/app-type.png)

接下来，系统会提示我们几个关于所生成服务的具体细节的问题，例如应用程序名称、服务器端口和服务发现:

[![jhipster uaa setup](img/1b715752972008624afa2a47ac4ac3e3.png)](/web/20220628145118/https://www.baeldung.com/wp-content/uploads/2018/12/jhipster-uaa-setup.png)

在大多数情况下，默认答案是好的。至于应用程序的基本名称**，它影响了许多生成的工件**，我们选择了`“uaa”`(小写)——一个合理的名称。如果我们愿意，我们可以改变其他值，但是它不会改变生成的项目的主要特性。

回答完这些问题后，JHipster 将创建所有项目文件并安装`npm`包依赖项(在本例中并没有真正用到)。

我们现在可以使用本地 Maven 脚本来构建和运行我们的 UAA 服务:

```
$ ./mvnw
... build messages omitted
2018-10-14 14:07:17.995  INFO 18052 --- [  restartedMain] com.baeldung.jhipster.uaa.UaaApp         :
----------------------------------------------------------
        Application 'uaa' is running! Access URLs:
        Local:          http://localhost:9999/
        External:       http://192.168.99.1:9999/
        Profile(s):     [dev, swagger]
----------------------------------------------------------
2018-10-14 14:07:18.000  INFO 18052 --- [  restartedMain] com.baeldung.jhipster.uaa.UaaApp         :
----------------------------------------------------------
        Config Server:  Connected to the JHipster Registry config server!
---------------------------------------------------------- 
```

这里需要注意的关键信息是，UAA 与 JHipster 注册中心有关联。该消息表明 UAA 能够注册自身，并且将可被其他微服务和网关发现。

## 5.测试 UAA 服务

由于生成的 UAA 服务本身没有 UI，我们必须使用直接的 API 调用来测试它是否按预期工作。

**在与其他部分或我们的系统一起使用之前，我们必须确保两个功能正常工作:** OAuth2 令牌生成和帐户检索。

首先，让**使用一个简单的`curl` 命令`:`从我们 UAA 的 OAuth 端点**获取一个新令牌

```
$ curl -X POST --data \
 "username=user&password;=user&grant;_type=password&scope;=openid" \
 http://web_app:[[email protected]](/web/20220628145118/https://www.baeldung.com/cdn-cgi/l/email-protection):9999/oauth/token 
```

这里，我们使用了`password grant `流，使用了两对凭证。在这种流程中，我们使用基本的 HTTP 身份验证发送客户端凭证，我们直接在 URL 中对其进行编码。

最终用户凭据作为正文的一部分发送，使用标准用户名和密码参数。**我们还使用名为`“user”`的用户帐户，它在测试配置文件中默认可用。**

假设我们正确地提供了所有细节，我们将得到一个包含访问令牌和刷新令牌的答案:

```
{
  "access_token" : "eyJh...(token omitted)",
  "token_type" : "bearer",
  "refresh_token" : "eyJ...(token omitted)",
  "expires_in" : 299,
  "scope" : "openid",
  "iat" : 1539650162,
  "jti" : "8066ab12-6e5e-4330-82d5-f51df16cd70f"
}
```

我们现在可以**使用返回的`access_token`来获取相关账户的信息，使用`account`资源**，该资源在 UAA 服务中可用:

```
$ curl -H "Authorization: Bearer eyJh...(access token omitted)" \ 
 http://localhost:9999/api/account
{
  "id" : 4,
  "login" : "user",
  "firstName" : "User",
  "lastName" : "User",
  "email" : "[[email protected]](/web/20220628145118/https://www.baeldung.com/cdn-cgi/l/email-protection)",
  "imageUrl" : "",
  "activated" : true,
  "langKey" : "en",
  "createdBy" : "system",
  "createdDate" : "2018-10-14T17:07:01.336Z",
  "lastModifiedBy" : "system",
  "lastModifiedDate" : null,
  "authorities" : [ "ROLE_USER" ]
} 
```

**请注意，我们必须在访问令牌到期**之前发出此命令。默认情况下，UAA 服务颁发有效期为五分钟的令牌，这对于生产来说是一个合理的值。

我们可以通过编辑与运行应用程序的配置文件相对应的`application-<profile>.yml`文件并设置`uaa.web-client-configuration.access-token-validity-in-seconds`键来轻松更改有效令牌的寿命。设置文件位于我们 UAA 项目的`src/main/resources/config`目录中。

## 6.生成支持 UAA 的网关

既然我们确信我们的 UAA 服务和服务注册正在工作，那么让我们为它们的交互创建一个生态系统。最后，我们会添加:

*   基于角度的前端
*   微服务后端
*   面向这两者的 API 网关

让我们实际上从网关开始，因为它将是与 UAA 协商身份验证的服务。它将托管我们的前端应用程序，并将 API 请求路由到其他微服务。

我们将再次在新创建的目录中使用 JHipster 命令行工具:

```
$ mkdir gateway
$ cd gateway
$ jhipster
```

和以前一样，为了生成项目，我们必须回答几个问题。其中最重要的如下:

*   `Application type` : **必须是“微服务网关”**
*   这次我们将使用“网关”
*   `Service discovery`:选择“JHipster 注册表”
*   `Authentication type:` **我们必须在这里选择**的“与 JHipster UAA 服务器认证”选项
*   `UI Framework:`我们来挑“棱角 6”

一旦 JHipster 生成了所有的工件，我们就可以用提供的 Maven 包装器脚本构建并运行网关:

```
$ ./mwnw
... many messages omitted
----------------------------------------------------------
        Application 'gateway' is running! Access URLs:
        Local:          http://localhost:8080/
        External:       http://192.168.99.1:8080/
        Profile(s):     [dev, swagger]
----------------------------------------------------------
2018-10-15 23:46:43.011  INFO 21668 --- [  restartedMain] c.baeldung.jhipster.gateway.GatewayApp   :
----------------------------------------------------------
        Config Server:  Connected to the JHipster Registry config server!
---------------------------------------------------------- 
```

根据上面的消息，我们可以通过将浏览器指向`[http://localhost:8080](https://web.archive.org/web/20220628145118/http://localhost:8080/),` 来访问我们的应用程序，这将显示默认生成的主页:

[![jhipster home](img/d4e1ac899d760ba995479e0e493b8436.png)](/web/20220628145118/https://www.baeldung.com/wp-content/uploads/2018/12/jhipster-home.png)

让我们导航到`Account > Login`菜单项，登录我们的应用程序。我们将使用`admin/admin `作为凭证，这是 JHipster 默认自动创建的。如果一切顺利，欢迎页面将显示一条确认成功登录的消息:

[![jhipster success ok 1](img/63728c9056ddd1bc2bb39cd802466c6f.png)](/web/20220628145118/https://www.baeldung.com/wp-content/uploads/2018/12/jhipster-success-ok-1.png)

让我们回顾一下发生了什么:首先，网关将我们的凭证发送到 UAA 的 OAuth2 令牌端点，该端点对它们进行验证，并生成包含访问和刷新 JWT 令牌的响应。然后，网关获取这些令牌，并将其作为 cookies 发送回浏览器。

接下来，棱角分明的前端调用了`/uaa/api/account` API，它再次被网关转发到 UAA。在这个过程中，网关获取包含访问令牌的 cookie，并使用其值将授权头添加到请求中。

如果需要，我们可以通过检查 UAA 和网关的日志来查看所有这些流程的细节。我们还可以通过将`org.apache.http.wire` logger 级别设置为 DEBUG 来获得完整的线路级数据。

## 7.生成支持 UAA 的微服务

现在，我们的应用程序环境已经启动并运行，是时候为它添加一个简单的微服务了。我们将创建一个“quotes”微服务，它将公开一个完整的 REST API，允许我们创建、查询、修改和删除一组股票报价。每个报价只有三个属性:

*   报价的贸易符号
*   它的价格，和
*   最后一笔交易的时间戳

让我们回到我们的终端，使用 JHipster 的命令行工具来生成我们的项目:

```
$ mkdir quotes
$ cd quotes
$ jhipster 
```

这一次，我们将要求 JHipster 生成一个`Microservice application`，我们称之为“报价”。这些问题和我们以前回答过的问题相似。除了这三个以外，我们可以保留大部分的默认值:

*   选择“JHipster Registry ”,因为我们已经在我们的架构中使用了它
*   `Path to the UAA application:`因为我们将所有项目目录保存在同一个文件夹下，这将是`../uaa `(当然，除非我们已经改变了它)
*   `Authentication Type:`选择“捷普斯特 UAA 服务器”

在我们的案例中，典型的答案序列如下:

[![jhipster microservice](img/b643731f4e38c0745f3868bebddca5f3.png)](/web/20220628145118/https://www.baeldung.com/wp-content/uploads/2018/12/jhipster-microservice.png)

一旦 JHipster 完成了项目的生成，我们就可以开始构建它了:

```
$ mvnw
... many, many messages omitted
----------------------------------------------------------
        Application 'quotes' is running! Access URLs:
        Local:          http://localhost:8081/
        External:       http://192.168.99.1:8081/
        Profile(s):     [dev, swagger]
----------------------------------------------------------
2018-10-19 00:16:05.581  INFO 16092 --- [  restartedMain] com.baeldung.jhipster.quotes.QuotesApp   :
----------------------------------------------------------
        Config Server:  Connected to the JHipster Registry config server!
----------------------------------------------------------
... more messages omitted 
```

消息“连接到 JHipster 注册表配置服务器！”这就是我们要找的。它的存在告诉我们，微服务向注册中心注册了自己，因此，一旦我们创建了它，网关就能够将请求路由到我们的“quotes”资源，并在一个漂亮的 UI 上显示它。由于我们使用的是微服务架构，因此我们将此任务分为两部分:

*   创建“报价”资源后端服务
*   在前端创建“报价”用户界面(网关项目的一部分)

### 7.1.添加报价资源

首先，我们需要**确保 quotes 微服务应用程序已经停止** —我们可以在之前运行它的同一个控制台窗口上点击 CTRL-C。

现在，让我们使用 JHipster 的工具为项目添加一个实体。这一次我们将使用`import-jdl`命令，这将把我们从繁琐且容易出错的单独提供所有细节的过程中解救出来。有关 JDL 格式的更多信息，请参考[完整 JDL 参考](https://web.archive.org/web/20220628145118/https://www.jhipster.tech/jdl/)。

接下来，我们**创建一个名为`quotes.jh`的文本文件，包含我们的`Quote `实体定义**，以及一些代码生成指令:

```
entity Quote {
  symbol String required unique,
  price BigDecimal required,
  lastTrade ZonedDateTime required
}
dto Quote with mapstruct
paginate Quote with pagination
service Quote with serviceImpl
microservice Quote with quotes
filter Quote
clientRootFolder Quote with quotes 
```

我们现在可以**将这个实体定义**导入到我们的项目中:

```
$ jhipster import-jdl quotes.jh 
```

**注意:在导入过程中，JHipster 会抱怨在对`master.xml`文件应用更改时出现冲突。在这种情况下，我们可以安全地选择`overwrite`选项。**

现在，我们可以使用`mvnw. `再次构建和运行我们的微服务。一旦微服务启动，我们可以验证网关是否选择了访问`Gateway`视图的新路由，该视图可从`Administration`菜单中获得。这一次，我们可以看到有一个用于`“/quotes/**”`路由的条目，显示后端已经准备好供 UI 使用。

### 7.2.添加报价用户界面

最后，让我们在 gateway 项目中生成 CRUD UI，我们将用它来访问我们的报价。我们将使用来自“quotes”微服务项目的同一个 JDL 文件来生成 UI 组件，并且我们将使用 JHipster 的`import-jdl`命令来导入它:

```
$ jhipster import-jdl ../jhipster-quotes/quotes.jh
...messages omitted
? Overwrite webpack\webpack.dev.js? <b>y</b>
... messages omitted
Congratulations, JHipster execution is complete! 
```

在导入过程中，JHipster 会提示几次应该对冲突文件采取的措施。在我们的例子中，我们可以简单地覆盖现有的资源，因为我们没有做任何定制。

现在我们可以重新启动网关，看看我们完成了什么。让我们将浏览器指向 [`http://localhost:8080`](https://web.archive.org/web/20220628145118/http://localhost:8080/) 的网关，确保我们刷新了它的内容。现在,`Entities`菜单应该有一个新条目用于`Quotes` 资源:

[![jhipster gateway entities menu](img/75d5ed30653e90a87eb88cdab0d36a19.png)](/web/20220628145118/https://www.baeldung.com/wp-content/uploads/2018/12/jhipster-gateway-entities-menu.png)

点击此菜单选项，调出`Quotes`列表屏幕:

[![jhipster gateway quotes](img/d2f14c61856a59d9d85eccf46c99c1a3.png)](/web/20220628145118/https://www.baeldung.com/wp-content/uploads/2018/12/jhipster-gateway-quotes.png)

正如所料，列表是空的—我们还没有添加任何报价！让我们尝试通过单击该屏幕右上角的“创建新报价按钮”来添加一个报价，这将把我们带到创建/编辑表单:

[![jhipster gateway add quote](img/ca20921713c88059ccda31bffeecc66d.png)](/web/20220628145118/https://www.baeldung.com/wp-content/uploads/2018/12/jhipster-gateway-add_quote.png)

我们可以看到生成的表单具有所有预期的特性:

*   必填字段标有红色指示器，一旦填写完毕，指示器将变为绿色
*   日期/时间和数字字段使用本地组件来帮助输入数据
*   我们可以取消此活动，这将保持数据不变，或者保存我们的新实体或修改后的实体

填写完这个表格并点击`Save, `后，我们将在列表屏幕上看到结果。我们现在可以在数据网格中看到新的`Quotes`实例:

[![jhipster gateway quotes2](img/261b8b707dc03e3e9bf3bf6cbb18ce86.png)](/web/20220628145118/https://www.baeldung.com/wp-content/uploads/2018/12/jhipster-gateway-quotes2.png)

作为管理员，我们还可以访问 API 菜单项，它将我们带到标准的 Swagger API 开发者门户。在此屏幕中，我们可以选择一个可用的 API 进行练习:

*   显示可用路线的网关自己的 API
*   `uaa`:账户和用户 API
*   `quotes: `报价 API

## 8.后续步骤

到目前为止，我们构建的应用程序运行正常，为进一步开发提供了坚实的基础。根据需求的复杂程度，我们肯定还需要编写一些(或许多)定制代码。可能需要做一些工作的一些领域是:

*   由于前端应用程序的结构方式，这通常很容易——我们可以通过摆弄 CSS 和添加一些图像来走很长的路
*   一些组织已经有了某种内部用户存储库(例如 LDAP 目录)，这需要对 UAA 进行更改，但好的一面是我们只需要更改一次
*   `Finer grained authorization on entities:` **由生成的实体后端使用的标准安全模型没有任何类型的实例级和/或字段级安全** —由开发人员在适当的级别(API 或服务，取决于具体情况)添加这些限制

尽管如此，在开发一个新的应用程序时，使用像 JHispter 这样的工具还是很有帮助的。它将带来一个坚实的基础，并且随着系统和开发人员的发展，可以在我们的代码库中保持良好的一致性。

## 9.结论

在本文中，我们展示了如何使用 JHipster 创建一个基于微服务架构和 jhi pster UAA 服务器的工作应用程序。我们没有写一行 Java 代码就做到了这一点，这令人印象深刻。

像往常一样，本文中介绍的项目的完整代码可以在我们的 [GitHub 资源库](https://web.archive.org/web/20220628145118/https://github.com/eugenp/tutorials/tree/master/jhipster-modules/jhipster-uaa)中获得。