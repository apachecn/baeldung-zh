# 将 Spring Boot 应用程序部署到 AWS Beanstalk

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-deploy-aws-beanstalk>

## 1。概述

在本教程中，我们将展示如何从我们的 [Bootstrap 一个简单的应用程序使用 Spring Boot](https://web.archive.org/web/20220630015858/https://www.baeldung.com/spring-boot-start) 教程到 [AWS Elastic Beanstalk](https://web.archive.org/web/20220630015858/https://docs.aws.amazon.com/elastic-beanstalk/index.html#lang/en_us) 部署一个应用程序。

作为其中的一部分，我们将:

*   安装和配置 AWS CLI 工具
*   创建一个 Beanstalk 项目和 MySQL 部署
*   在 AWS RDS 中为 MySQL 配置应用程序
*   部署、测试和扩展应用程序

## 2。AWS 弹性豆茎配置

**作为先决条件，我们应该在 AWS 上注册，并且[在 Elastic Beanstalk](https://web.archive.org/web/20220630015858/https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.environments.html) 上创建了一个 Java 8 环境。我们还需要[安装 AWS CLI](https://web.archive.org/web/20220630015858/https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) ，这将允许我们连接到我们的环境。**

因此，鉴于此，我们需要登录并初始化我们的应用程序:

```
cd .../spring-boot-bootstrap
eb init 
```

```
>
Select a default region
1) us-east-1 : US East (N. Virginia)
2) us-west-1 : US West (N. California)
3) us-west-2 : US West (Oregon)
4) eu-west-1 : EU (Ireland)
5) eu-central-1 : EU (Frankfurt)
6) ap-south-1 : Asia Pacific (Mumbai)
7) ap-southeast-1 : Asia Pacific (Singapore)
8) ap-southeast-2 : Asia Pacific (Sydney)
9) ap-northeast-1 : Asia Pacific (Tokyo)
10) ap-northeast-2 : Asia Pacific (Seoul)
11) sa-east-1 : South America (Sao Paulo)
12) cn-north-1 : China (Beijing)
13) cn-northwest-1 : China (Ningxia)
14) us-east-2 : US East (Ohio)
15) ca-central-1 : Canada (Central)
16) eu-west-2 : EU (London)
17) eu-west-3 : EU (Paris)
18) eu-north-1 : EU (Stockholm)
(default is 3):
```

如上所示，系统会提示我们选择一个区域。

最后，我们可以选择应用程序:

```
>
Select an application to use
1) baeldung-demo
2) [ Create new Application ]
(default is 2): 
```

此时，**CLI 将创建一个名为** `**.elasticbeanstalk/config.yml**.`的文件。该文件将保留项目的默认值。

## 3。数据库

现在，我们可以在 AWS Web 控制台上或通过 CLI 使用以下命令创建数据库:

```
eb create --single --database
```

我们需要按照说明提供用户名和密码。

创建好数据库后，现在让我们为应用程序配置 RDS 凭证。我们将通过在应用程序中创建`src/main/resources/application-beanstalk.properties` 在名为`beanstalk`的 Spring 概要文件中这样做:

```
spring.datasource.url=jdbc:mysql://${rds.hostname}:${rds.port}/${rds.db.name}
spring.datasource.username=${rds.username}
spring.datasource.password=${rds.password} 
```

**Spring 将搜索名为`rds.hostname`的属性作为名为`RDS_HOSTNAME`的环境变量。同样的逻辑将适用于其余部分。**

## 4。应用程序

现在，我们将为`pom.xml`添加一个 Beanstalk `–`特有的 Maven 概要文件:

```
<profile>
    <id>beanstalk</id>
    <build>
        <finalName>${project.name}-eb</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
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
</profile>
```

**接下来，我们将把工件指定到弹性豆茎配置文件`.elasticbeanstalk/config.yml` :**

```
deploy:
  artifact: target/spring-boot-bootstrap-eb.jar 
```

最后，我们将在弹性豆茎中包含两个环境变量。第一个将指定活动的 Spring 配置文件，第二个将确保使用 Beanstalk 期望的默认端口 5000:

```
eb setenv SPRING_PROFILES_ACTIVE=beanstalk,mysql
eb setenv SERVER_PORT=5000
```

## 5。部署和测试

**现在我们已经准备好构建和部署:**

```
mvn clean package spring-boot:repackage
eb deploy 
```

接下来，我们将检查状态并确定部署的应用程序的 DNS 名称:

```
eb status
```

我们的输出应该是这样的:

```
Environment details for: BaeldungDemo-env
  Application name: baeldung-demo
  Region: us-east-2
  Deployed Version: app-181216_154233
  Environment ID: e-42mypzuc2x
  Platform: arn:aws:elasticbeanstalk:us-east-2::platform/Java 8 running on 64bit Amazon Linux/2.7.7
  Tier: WebServer-Standard-1.0
  CNAME: BaeldungDemo-env.uv3tr7qfy9.us-east-2.elasticbeanstalk.com
  Updated: 2018-12-16 13:43:22.294000+00:00
  Status: Ready
  Health: Green
```

我们现在可以测试应用程序了——请注意使用 CNAME 字段作为 DNS 来完成 URL。

现在让我们向图书馆添加一本书:

```
http POST http://baeldungdemo-env.uv3tr7qfy9.us-east-2.elasticbeanstalk.com/api/books title="The Player of Games" author="Iain M. Banks"
```

如果一切顺利的话，我们应该会得到这样的结果:

```
HTTP/1.1 201 
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Connection: keep-alive
Content-Type: application/json;charset=UTF-8
Date: Wed, 19 Dec 2018 15:36:31 GMT
Expires: 0
Pragma: no-cache
Server: nginx/1.12.1
Transfer-Encoding: chunked
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block

{
    "author": "Iain M. Banks",
    "id": 5,
    "title": "The Player of Games"
}
```

## 6。扩展应用程序

最后，我们扩展部署以运行两个实例:

```
eb scale 2
```

**Beanstalk 现在将运行该应用程序的两个实例，并在两个实例之间平衡流量负载。**

用于生产的自动缩放涉及到更多的，所以我们将把它留到下一天。

## 7。结论

在本教程中，我们将:

*   安装并配置了 AWS Beanstalk CLI，并配置了在线环境
*   部署了 MySQL 服务并配置了数据库连接属性
*   构建并部署我们已配置的 Spring Boot 应用程序，以及
*   测试和扩展应用程序

要了解更多细节，请查看 Beanstalk Java 文档。

和往常一样，我们示例的完整源代码在这里，在 GitHub 上的[。](https://web.archive.org/web/20220630015858/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-bootstrap)