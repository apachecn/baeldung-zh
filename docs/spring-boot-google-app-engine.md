# 将 Spring Boot 应用程序部署到 Google App Engine

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-google-app-engine>

## 1。概述

在本教程中，我们将展示如何在谷歌云平台上使用 Spring Boot 教程从我们的 [Bootstrap 简单应用到](/web/20221129020842/https://www.baeldung.com/spring-boot-start)[应用引擎](https://web.archive.org/web/20221129020842/https://cloud.google.com/appengine/)部署应用。

作为其中的一部分，我们将:

*   配置谷歌云平台控制台和 SDK
*   使用云 SQL 创建一个 MySQL 实例
*   为 Spring Cloud GCP 配置应用程序
*   将应用程序部署到 App Engine 并测试它

## 2。谷歌云平台配置

我们可以使用 GCP 控制台为 GCP 准备本地环境。我们可以在官网上找到[安装流程](https://web.archive.org/web/20221129020842/https://cloud.google.com/sdk/)。

让我们使用 [GCP 控制台](https://web.archive.org/web/20221129020842/https://console.cloud.google.com/)在 GCP 上创建一个项目:

```java
gcloud init
```

接下来，让我们配置项目名称:

```java
gcloud config set project baeldung-spring-boot-bootstrap
```

然后，我们将安装应用引擎支持并创建一个应用引擎实例:

```java
gcloud components install app-engine-java
gcloud app create
```

我们的应用程序需要连接到云 SQL 环境中的 MySQL 数据库。由于云 SQL 不提供免费层，我们必须在 GCP 账户上启用[计费](https://web.archive.org/web/20221129020842/https://cloud.google.com/billing/docs/how-to/modify-project)。

我们可以轻松检查可用层:

```java
gcloud sql tiers list 
```

在继续之前，我们应该使用 GCP 网站来启用[云 SQL 管理 API](https://web.archive.org/web/20221129020842/https://console.cloud.google.com/flows/enableapi?apiid=sqladmin) 。

现在，我们可以使用云控制台或 SDK CLI 在[云 SQL](https://web.archive.org/web/20221129020842/https://console.cloud.google.com/sql/instances) 中创建一个 MySQL 实例和数据库。在此过程中，我们将选择区域并提供实例名和数据库名。应用程序和数据库实例必须在同一地区，这一点很重要。

因为我们要将应用程序部署到`europe-west2`，所以让我们对实例做同样的事情:

```java
# create instance
gcloud sql instances create \
  baeldung-spring-boot-bootstrap-db \
    --tier=db-f1-micro \
    --region=europe-west2
# create database
gcloud sql databases create \
  baeldung_bootstrap_db \
    --instance=baeldung-spring-boot-bootstrap-db
```

## 3。春云 GCP 属地

我们的应用程序将需要来自云原生 API 的 [Spring Cloud GCP](https://web.archive.org/web/20221129020842/https://spring.io/projects/spring-cloud-gcp) 项目的依赖。为此，让我们使用名为`cloud-gcp`的 Maven 概要文件:

```java
<profile>
  <id>cloud-gcp</id>
  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-gcp-starter</artifactId>
      <version>1.0.0.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-gcp-starter-sql-mysql</artifactId>
      <version>1.0.0.RELEASE</version>
    </dependency>
  </dependencies>
```

然后我们添加 App Engine Maven 插件:

```java
 <build>
      <plugins>
        <plugin>
          <groupId>com.google.cloud.tools</groupId>
          <artifactId>appengine-maven-plugin</artifactId>
          <version>1.3.2</version>
        </plugin>
      </plugins>
    </build>
</profile>
```

## 4。应用程序配置

现在，让我们定义允许应用程序使用云原生资源(如数据库)的配置。

春云 GCP 使用`spring-cloud-bootstrap.properties`来确定应用名称:

```java
spring.cloud.appId=baeldung-spring-boot-bootstrap
```

对于这个部署，我们将使用名为`gcp`的 Spring 概要文件，并且我们需要配置数据库连接。因此我们创建了`src/main/resources/application-gcp.properties`:

```java
spring.cloud.gcp.sql.instance-connection-name=\
    baeldung-spring-boot-bootstrap:europe-west2:baeldung-spring-boot-bootstrap-db
spring.cloud.gcp.sql.database-name=baeldung_bootstrap_db
```

## 5。部署

Google App Engine 提供了两种 Java 环境:

*   `Standard`环境提供 Jetty 和 JDK8，而`Flexible`环境只提供 JDK8 和
*   灵活的环境是 Spring Boot 应用程序的最佳选择。

我们需要激活`gcp`和`mysql` Spring 概要文件，因此我们通过将`SPRING_PROFILES_ACTIVE`环境变量添加到`src/main/appengine/app.yaml`中的部署配置中来为应用程序提供该变量:

```java
runtime: java
env: flex
runtime_config:
  jdk: openjdk8
env_variables:
  SPRING_PROFILES_ACTIVE: "gcp,mysql"
handlers:
- url: /.*
  script: this field is required, but ignored
manual_scaling: 
  instances: 1
```

现在，**让我们使用`appengine` maven 插件**构建和部署应用程序:

```java
mvn clean package appengine:deploy -P cloud-gcp
```

部署后，我们可以查看或跟踪日志文件:

```java
# view
gcloud app logs read

# tail
gcloud app logs tail
```

现在，**让我们通过添加一本书来验证我们的应用程序正在工作**:

```java
http POST https://baeldung-spring-boot-bootstrap.appspot.com/api/books \
        title="The Player of Games" author="Iain M. Banks" 
```

需要以下输出:

```java
HTTP/1.1 201 
{
    "author": "Iain M. Banks",
    "id": 1,
    "title": "The Player of Games"
}
```

## 6。扩展应用程序

**App Engine 中的默认缩放是自动的。**

在我们理解运行时行为以及相关的预算和成本之前，从手动缩放开始可能更好。我们可以为应用分配资源，并在`app.yaml`中配置自动扩展:

```java
# Application Resources
resources:
  cpu: 2
  memory_gb: 2
  disk_size_gb: 10
  volumes:
  - name: ramdisk1
    volume_type: tmpfs
    size_gb: 0.5
# Automatic Scaling
automatic_scaling: 
  min_num_instances: 1 
  max_num_instances: 4 
  cool_down_period_sec: 180 
  cpu_utilization: 
    target_utilization: 0.6
```

## 7。结论

在本教程中，我们将:

*   配置谷歌云平台和应用引擎
*   用云 SQL 创建了一个 MySQL 实例
*   为使用 MySQL 配置了 Spring Cloud GCP
*   部署我们配置好的 Spring Boot 应用程序，以及
*   测试和扩展应用程序

我们可以随时参考谷歌广泛的[应用引擎文档](https://web.archive.org/web/20221129020842/https://cloud.google.com/appengine/docs/flexible/java/)来了解更多细节。

我们这里例子的完整源代码一如既往地在 GitHub 的[上。](https://web.archive.org/web/20221129020842/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-bootstrap)