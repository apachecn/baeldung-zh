# 用 Swagger 生成 Spring Boot REST 客户端

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-rest-client-swagger-codegen>

## 1。简介

在本文中，我们将使用 [Swagger Codegen](https://web.archive.org/web/20221001011419/https://github.com/swagger-api/swagger-codegen) 和 [OpenAPI Generator](https://web.archive.org/web/20221001011419/https://github.com/OpenAPITools/openapi-generator) 项目从 [OpenAPI/Swagger spec](https://web.archive.org/web/20221001011419/https://swagger.io/specification/) 文件生成 REST 客户端。

此外，我们将创建一个 Spring Boot 项目，其中我们将使用生成的类。

我们将对所有内容使用 [Swagger Petstore](https://web.archive.org/web/20221001011419/http://petstore.swagger.io/) API 示例。

## 2。使用 Swagger Codegen 生成 REST 客户端

Swagger 提供了一个实用工具 jar，允许我们为各种编程语言和多种框架生成 REST 客户端。

### 2.1。下载 Jar 文件

`code-gen_cli.jar`可以从[这里](https://web.archive.org/web/20221001011419/https://search.maven.org/classic/remotecontent?filepath=io/swagger/swagger-codegen-cli/2.2.3/swagger-codegen-cli-2.2.3.jar)下载。

最新版本请查看 [swagger-codegen-cli](https://web.archive.org/web/20221001011419/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22io.swagger%22%20AND%20a%3A%22swagger-codegen-cli%22) 库。

### 2.2。生成客户端

让我们通过执行命令`java -jar swagger-code-gen-cli.jar generate:`来生成我们的客户端

```java
java -jar swagger-codegen-cli.jar generate \
  -i http://petstore.swagger.io/v2/swagger.json \
  --api-package com.baeldung.petstore.client.api \
  --model-package com.baeldung.petstore.client.model \
  --invoker-package com.baeldung.petstore.client.invoker \
  --group-id com.baeldung \
  --artifact-id spring-swagger-codegen-api-client \
  --artifact-version 0.0.1-SNAPSHOT \
  -l java \
  --library resttemplate \
  -o spring-swagger-codegen-api-client
```

提供的参数包括:

*   源 swagger 文件 URL 或路径——使用`-i`参数提供
*   生成类的包名——使用`–api-package`、`–model-package`、`–invoker-package`提供
*   生成的 Maven 项目属性`–group-id`、`–artifact-id`、`–artifact-version`
*   生成的客户端的编程语言——使用`-l`提供
*   实施框架–使用`–library`提供
*   输出目录–使用`-o`提供

要列出所有与 Java 相关的选项，请键入以下命令:

```java
java -jar swagger-codegen-cli.jar config-help -l java
```

Swagger Codegen 支持以下 Java 库(成对的 HTTP 客户端和 JSON 处理库):

*   `jersey1`–球衣 1 +杰克逊
*   `jersey2`–球衣 2 +杰克逊
*   `feign`–open feign+Jackson
*   `okhttp-gson`–ok http+Gson
*   `retrofit`(废弃)–翻新 1/OkHttp + Gson
*   `retrofit2`—改造 2/OkHttp + Gson
*   `rest-template`–Spring rest template+杰克逊
*   `rest-easy`–Resteasy+Jackson

在这篇文章中，我们选择了`rest-template`，因为它是 Spring 生态系统的一部分。

## 3.用 OpenAPI 生成器生成 REST 客户端

OpenAPI Generator 是 Swagger Codegen 的一个分支，能够从任何 OpenAPI 规范 2.0/3.x 文档生成 50 多个客户端。

Swagger Codegen 由 SmartBear 维护，OpenAPI Generator 由一个社区维护，该社区包括 40 多名 Swagger Codegen 的顶级贡献者和模板创建者作为创始团队成员。

### 3.1.装置

也许最简单和最便携的安装方法是使用 [`npm`包](https://web.archive.org/web/20221001011419/https://www.npmjs.com/package/@openapitools/openapi-generator-cli)包装器，它通过在 Java 代码支持的命令行选项上提供一个 CLI 包装器来工作。安装非常简单:

```java
npm install @openapitools/openapi-generator-cli -g
```

对于那些想要 JAR 文件的人，可以在 [Maven Central](https://web.archive.org/web/20221001011419/https://repo1.maven.org/maven2/org/openapitools/openapi-generator-cli) 中找到。现在就下载吧:

```java
wget https://repo1.maven.org/maven2/org/openapitools/openapi-generator-cli/4.2.3/openapi-generator-cli-4.2.3.jar \
  -O openapi-generator-cli.jar 
```

### 3.2.生成客户端

首先，OpenAPI 生成器的选项几乎与 Swagger Codegen 的选项相同。最显著的区别是用`-g`生成器标志替换了`-l`语言标志，它将生成客户端的语言作为参数。

接下来，让我们使用`jar`命令生成一个客户机，它相当于我们用 Swagger Codegen 生成的客户机:

```java
java -jar openapi-generator-cli.jar generate \
  -i http://petstore.swagger.io/v2/swagger.json \
  --api-package com.baeldung.petstore.client.api \
  --model-package com.baeldung.petstore.client.model \
  --invoker-package com.baeldung.petstore.client.invoker \
  --group-id com.baeldung \
  --artifact-id spring-openapi-generator-api-client \
  --artifact-version 0.0.1-SNAPSHOT \
  -g java \
  -p java8=true \
  --library resttemplate \
  -o spring-openapi-generator-api-client
```

要列出所有与 Java 相关的选项，请键入以下命令:

```java
java -jar openapi-generator-cli.jar config-help -g java
```

OpenAPI Generator 支持所有与 Swagger CodeGen 相同的 Java 库，外加一些额外的库。OpenAPI Generator 支持以下 Java 库(HTTP 客户端和 JSON 处理库对):

*   `jersey1`–球衣 1 +杰克逊
*   `jersey2`–球衣 2 +杰克逊
*   `feign`–open feign+Jackson
*   `okhttp-gson`–ok http+Gson
*   `retrofit`(废弃)–翻新 1/OkHttp + Gson
*   `retrofit2`—改造 2/OkHttp + Gson
*   `resttemplate`–Spring rest template+杰克逊
*   `webclient`–Spring 5 WebClient+Jackson(仅限 OpenAPI 生成器)
*   `resteasy`–Resteasy+Jackson
*   `vertx`–VertX+Jackson
*   `google-api-client`–谷歌 API 客户端+杰克逊
*   `rest-assured`–放心+ Jackson/Gson(仅限 Java 8)
*   `native`–Java 原生 HttpClient + Jackson(仅限 Java 11 仅限 OpenAPI 生成器)
*   `microprofile`–micro profile 客户端+ Jackson(仅限 OpenAPI 生成器)

## 4。生成 Spring Boot 项目

现在让我们创建一个新的 Spring Boot 项目。

### 4.1。Maven 依赖关系

我们首先将生成的 API 客户端库的依赖项添加到我们的项目`pom.xml`文件中:

```java
<dependency>
    <groupId>com.baeldung</groupId>
    <artifactId>spring-swagger-codegen-api-client</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

### 4.2。将 API 类公开为 Spring bean

要访问生成的类，我们需要将它们配置为 beans:

```java
@Configuration
public class PetStoreIntegrationConfig {

    @Bean
    public PetApi petApi() {
        return new PetApi(apiClient());
    }

    @Bean
    public ApiClient apiClient() {
        return new ApiClient();
    }
}
```

### 4.3。API 客户端配置

`ApiClient`类用于配置认证、API 的基本路径、公共头，它负责执行所有 API 请求。

例如，如果您正在使用 OAuth:

```java
@Bean
public ApiClient apiClient() {
    ApiClient apiClient = new ApiClient();

    OAuth petStoreAuth = (OAuth) apiClient.getAuthentication("petstore_auth");
    petStoreAuth.setAccessToken("special-key");

    return apiClient;
}
```

### 4.4。弹簧主要应用

我们需要导入新创建的配置:

```java
@SpringBootApplication
@Import(PetStoreIntegrationConfig.class)
public class PetStoreApplication {
    public static void main(String[] args) throws Exception {
        SpringApplication.run(PetStoreApplication.class, args);
    }
}
```

### 4.5。API 用法

因为我们将 API 类配置为 beans，所以我们可以自由地将它们注入到 Spring 管理的类中:

```java
@Autowired
private PetApi petApi;

public List<Pet> findAvailablePets() {
    return petApi.findPetsByStatus(Arrays.asList("available"));
}
```

## 5.替代解决方案

除了执行 Swagger Codegen 或 OpenAPI Generator CLI 之外，还有其他生成 REST 客户端的方法。

### 5.1。Maven 插件

一个可以在你的`pom.xml`中轻松配置的 [swagger-codegen Maven 插件](https://web.archive.org/web/20221001011419/https://github.com/swagger-api/swagger-codegen/blob/master/modules/swagger-codegen-maven-plugin/README.md)允许使用与 Swagger Codegen CLI 相同的选项生成客户端。

这是一个基本的代码片段，我们可以将它包含在项目的`pom.xml`中，以自动生成客户端:

```java
<plugin>
    <groupId>io.swagger</groupId>
    <artifactId>swagger-codegen-maven-plugin</artifactId>
    <version>2.2.3</version>
    <executions>
        <execution>
            <goals>
                <goal>generate</goal>
            </goals>
            <configuration>
                <inputSpec>swagger.yaml</inputSpec>
                <language>java</language>
                <library>resttemplate</library>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### 5.2。Swagger Codegen 在线生成器 API

一个已经发布的 API，它通过向 URL `http://generator.swagger.io/api/gen/clients/java`发送 POST 请求来帮助我们生成客户端，同时在请求体中传递规范 URL 和其他选项。

让我们用一个简单的 curl 命令来做一个例子:

```java
curl -X POST -H "content-type:application/json" \
  -d '{"swaggerUrl":"http://petstore.swagger.io/v2/swagger.json"}' \
  http://generator.swagger.io/api/gen/clients/java
```

响应将是 JSON 格式的，其中包含一个可下载的链接，该链接包含以 zip 格式生成的客户端代码。您可以传递在 Swaager Codegen CLI 中使用的相同选项来自定义输出客户端。

https://generator . swagger . io 包含 API 的 Swagger 文档，我们可以在这里查看它的文档并试用它。

### 5.3。OpenAPI 生成器在线生成器 API

和 Swagger Godegen 一样，OpenAPI Generator 也有一个在线生成器。让我们使用一个简单的 curl 命令来执行一个示例:

```java
curl -X POST -H "content-type:application/json" \
  -d '{"openAPIUrl":"http://petstore.swagger.io/v2/swagger.json"}' \
  http://api.openapi-generator.tech/api/gen/clients/java
```

JSON 格式的响应将包含一个可下载的链接，链接到生成的 zip 格式的客户端代码。您可以传递在 Swagger Codegen CLI 中使用的相同选项来定制输出客户端。

[https://github . com/open api tools/open API-generator/blob/master/docs/online . MD](https://web.archive.org/web/20221001011419/https://github.com/OpenAPITools/openapi-generator/blob/master/docs/online.md)包含 API 的文档。

## 6。结论

Swagger Codegen 和 OpenAPI Generator 使您能够使用多种语言和您选择的库为您的 API 快速生成 REST 客户端。我们可以使用 CLI 工具、Maven 插件或在线 API 来生成客户端库。

这是一个基于 Maven 的项目，包含三个 Maven 模块:生成的 Swagger API 客户端、生成的 OpenAPI 客户端和 Spring Boot 应用程序。

和往常一样，你可以在 GitHub 上找到可用的代码[。](https://web.archive.org/web/20221001011419/https://github.com/eugenp/tutorials/tree/master/spring-swagger-codegen)