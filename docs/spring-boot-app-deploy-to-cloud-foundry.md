# 将 Spring Boot 应用部署到 Cloud Foundry

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-app-deploy-to-cloud-foundry>

## 1。概述

将 Spring Boot 应用部署到 Cloud Foundry 是一项简单的工作。在本教程中，我们将告诉你如何做。

## 2。春云相依

由于该项目需要 Spring Cloud 项目的新依赖项，我们将添加 Spring Cloud 依赖项 BOM:

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Greenwhich.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

我们可以在 [Maven Central](https://web.archive.org/web/20220926194005/https://search.maven.org/search?q=g:org.springframework.cloud%20AND%20a:spring-cloud-dependencies&core=gav) 上找到最新版本的`spring-cloud-dependencies`库。

现在，我们想要为 Cloud Foundry 维护一个单独的构建，所以我们将在 Maven `pom.xml.`中创建一个名为 *cloudfoundry* 的概要文件

我们还将添加编译器排除和 Spring Boot 插件来配置包的名称:

```java
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <excludes>
                <exclude>**/logback.xml</exclude>
            </excludes>
        </resource>
    </resources>
    <plugins>
        <plugin>                        
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <finalName>${project.name}-cf</finalName>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <excludes>
                    <exclude>**/cloud/config/*.java</exclude>
                </excludes>
            </configuration>
        </plugin>
    </plugins>
</build>
```

我们还想从普通构建中排除特定于云的文件，因此我们向 Maven 编译器插件添加了一个全局配置文件排除:

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <excludes>
                    <exclude>**/cloud/*.java</exclude>
                </excludes>
            </configuration>
        </plugin>
    </plugins>
</build>
```

然后，我们需要添加 Spring Cloud Starter 和 Spring Cloud Connectors 库，它们为 Cloud Foundry 提供支持:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cloud-connectors</artifactId>
</dependency>
```

## 3。云代工厂配置

要浏览本教程，我们需要在这里注册一个试用版[或者下载](https://web.archive.org/web/20220926194005/https://run.pivotal.io/)[原生 Linux](https://web.archive.org/web/20220926194005/https://github.com/cloudfoundry-incubator/cfdev) 或者[虚拟盒子](https://web.archive.org/web/20220926194005/https://docs.pivotal.io/pcf-dev/)的预配置开发环境。

此外，还需要安装 Cloud Foundry CLI。这里的指令是。

向 Cloud Foundry 提供商注册后，API URL 将变得可用(您可以通过点击左侧的`Tools`选项返回到该 URL)。

应用程序容器允许我们将服务绑定到应用程序。接下来，让我们登录 Cloud Foundry 环境:

```java
cf login -a <url>
```

**Cloud Foundry market place 是一个服务目录**,如数据库、消息、电子邮件、监控、日志等等。大多数服务提供免费或试用计划。

让我们在市场上搜索“MySQL ”,并为我们的应用程序创建一个服务:

```java
cf marketplace | grep MySQL
```

```java
>
cleardb     spark, boost*, amp*, shock*         Highly available MySQL for your Apps. 
```

输出中列出了描述中带有“MySQL”的服务。在 PCF 上，MySQL 服务被命名为`cleardb`，非免费计划标有星号。

接下来，我们使用以下代码列出服务的详细信息:

```java
cf marketplace -s cleardb
```

```java
>
service plan description                                                                 free or paid
spark        Great for getting started and developing your apps                             free
boost        Best for light production or staging your applications                         paid
amp          For apps with moderate data requirements                                       paid
shock        Designed for apps where you need real MySQL reliability, power and throughput  paid
```

现在我们创建一个名为`spring-bootstrap-db`的免费 MySQL 服务实例:

```java
cf create-service cleardb spark spring-bootstrap-db
```

## 4。应用程序配置

接下来，我们添加一个带注释的类`@Configuration`，该类扩展了`AbstractCloudConfig`以在名为`org.baeldung.cloud.config`的包中创建一个`DataSource `:

```java
@Configuration
@Profile("cloud")
public class CloudDataSourceConfig extends AbstractCloudConfig {

    @Bean
    public DataSource dataSource() {
        return connectionFactory().dataSource();
    }
}
```

添加`@Profile(“cloud”)`可以确保当我们进行本地测试时，云连接器不会被激活。我们还将`@ActiveProfiles(profiles = {“local”})`添加到集成测试中。

然后使用以下内容构建应用程序:

```java
mvn clean install spring-boot:repackage -P cloudfoundry
```

此外，我们需要提供一个`manifest.yml`文件，将服务绑定到应用程序。

我们通常将`manifest.yml`文件放在项目文件夹中，但在这种情况下，我们将创建一个`cloudfoundry`文件夹，因为我们将演示部署到多个云原生提供商:

```java
---
applications:
- name: spring-boot-bootstrap
  memory: 768M
  random-route: true
  path: ../target/spring-boot-bootstrap-cf.jar
  env:
    SPRING_PROFILES_ACTIVE: cloud,mysql
  services:
  - spring-bootstrap-db
```

## 5。部署

部署应用程序现在就像:

```java
cd cloudfoundry
cf push
```

Cloud Foundry 将使用 Java buildpack 来部署应用程序，并创建到应用程序的随机路径。

我们可以使用以下命令查看日志文件中的最后几个条目:

```java
cf logs spring-boot-bootstrap --recent
```

或者我们可以跟踪日志文件:

```java
cf logs spring-boot-bootstrap
```

最后，我们需要路由名称来测试应用程序:

```java
cf app spring-boot-bootstrap
```

```java
>
name:              spring-boot-bootstrap
requested state:   started
routes:            spring-boot-bootstrap-delightful-chimpanzee.cfapps.io
last uploaded:     Thu 23 Aug 08:57:20 SAST 2018
stack:             cflinuxfs2
buildpacks:        java-buildpack=v4.15-offline-...

type:           web
instances:      1/1
memory usage:   768M
     state     since                  cpu    memory           disk
#0   running   2018-08-23T06:57:57Z   0.5%   290.9M of 768M   164.7M of 1G 
```

执行以下命令将添加一本新书:

```java
curl -i --request POST \
    --header "Content-Type: application/json" \
    --data '{"title": "The Player of Games", "author": "Iain M. Banks"}' \
    https://<app-route>/api/books
#OR
http POST https://<app-route>/api/books title="The Player of Games" author="Iain M. Banks" 
```

这个命令将列出所有书籍:

```java
curl -i https://<app-route>/api/books 
#OR 
http https://<app-route>/api/books
```

```java
>
HTTP/1.1 200 OK

[
    {
        "author": "Iain M. Banks",
        "id": 1,
        "title": "Player of Games"
    },
    {
        "author": "J.R.R. Tolkien",
        "id": 2,
        "title": "The Hobbit"
    }
] 
```

## 6。扩展应用程序

最后，在 Cloud Foundry 上扩展应用程序就像使用`scale`命令一样简单:

```java
cf scale spring-cloud-bootstrap-cloudfoundry <options>
Options:
-i <instances>
-m <memory-allocated> # Like 512M or 1G
-k <disk-space-allocated> # Like 1G or 2G
-f # Force restart without prompt
```

当我们不再需要该应用程序时，请记住将其删除:

```java
cf delete spring-cloud-bootstrap-cloudfoundry
```

## 7。结论

在本文中，我们介绍了 Spring Cloud libraries，它使用 Spring Boot 简化了云原生应用程序的开发。使用 Cloud Foundry CLI 的部署在[这里](https://web.archive.org/web/20220926194005/https://docs.cloudfoundry.org/cf-cli/cf-help.html)有很好的记录。

CLI 的额外插件可在[插件库](https://web.archive.org/web/20220926194005/https://plugins.cloudfoundry.org/)中获得。

我们这里例子的完整源代码一如既往地在 GitHub 的[上。](https://web.archive.org/web/20220926194005/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-bootstrap)