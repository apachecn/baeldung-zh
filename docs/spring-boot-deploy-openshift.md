# 将 Spring Boot 应用程序部署到 OpenShift

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-deploy-openshift>

## 1。概述

在本教程中，我们将展示如何从我们的 [Bootstrap 使用 Spring Boot](/web/20221207181235/https://www.baeldung.com/spring-boot-start) 教程到 [Openshift](https://web.archive.org/web/20221207181235/https://www.openshift.com/) 部署一个简单的应用程序。

作为其中的一部分，我们将:

*   安装和配置 Openshift 开发工具。
*   创建一个 Openshift 项目和 MySQL 部署。
*   为 [Spring Cloud Kubernetes](https://web.archive.org/web/20221207181235/https://github.com/spring-cloud/spring-cloud-kubernetes) 配置应用。
*   使用 [Fabric8 Maven 插件](https://web.archive.org/web/20221207181235/https://github.com/fabric8io/fabric8-maven-plugin)在容器中创建和部署应用程序，并测试和扩展应用程序。

## 2。Openshift 配置

首先，**我们需要[安装 Minishift](https://web.archive.org/web/20221207181235/https://docs.okd.io/latest/minishift/getting-started/installing.html) ，本地单节点 Openshift 集群， [Openshift 客户端](https://web.archive.org/web/20221207181235/https://docs.okd.io/latest/cli_reference/openshift_cli/getting-started-cli.html#installing-the-cli)** 。

在使用 Minishift 之前，我们需要为开发人员用户配置权限:

```java
minishift addons install --defaults
minishift addons enable admin-user
minishift start
oc adm policy --as system:admin add-cluster-role-to-user cluster-admin developer
```

现在我们想使用 Openshift 控制台创建一个 MySQL 服务。我们可以使用以下方式启动浏览器 URL:

```java
minishift console
```

如果您没有自动登录，则使用`developer/developer.`

创建一个名为`baeldung-demo`的项目，然后从目录中创建一个 MySQL 数据库服务。为数据库服务提供`baeldung-db`，为 MySQL 数据库名称提供`baeldung_db `，其他值保留默认值。

我们现在有了访问数据库的服务和秘密。记下数据库连接 url: `mysql://baeldung-db:3306/baeldung_db`

我们还需要允许应用程序读取像 Kubernetes Secrets 和 ConfigMaps 这样的配置:

```java
oc create rolebinding default-view --clusterrole=view \
  --serviceaccount=baeldung-demo:default --namespace=baeldung-demo
```

## 3。春云·库伯内斯依赖

**我们将使用 [Spring Cloud Kubernetes](https://web.archive.org/web/20221207181235/https://github.com/spring-cloud/spring-cloud-kubernetes) 项目为支持 Openshift 的 Kubernetes 启用云原生 API:**

```java
<profile>
  <id>openshift</id>
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-kubernetes-dependencies</artifactId>
        <version>0.3.0.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>Greenwich.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-kubernetes-config</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
  </dependencies>
</profile>
```

**我们还将使用 [Fabric8 Maven 插件](https://web.archive.org/web/20221207181235/https://github.com/fabric8io/fabric8-maven-plugin)来构建和部署容器:**

```java
<plugin>
    <groupId>io.fabric8</groupId>
    <artifactId>fabric8-maven-plugin</artifactId>
    <version>3.5.37</version>
    <executions>
      <execution>
        <id>fmp</id>
        <goals>
          <goal>resource</goal>
          <goal>build</goal>
        </goals>
      </execution>
    </executions>
</plugin> 
```

## 4。应用程序配置

现在**我们需要提供配置来确保正确的 Spring 配置文件和 Kubernetes 秘密作为环境变量被注入**。

**让我们在`src/main/fabric8`** 中创建一个 YAML 片段，以便 Fabric8 Maven 插件在创建部署配置时使用它。

我们还需要为 Spring Boot 执行器添加一个段，因为 Fabric8 中的默认仍然试图访问`/health`而不是`/actuator/health:`

```java
spec:
  template:
    spec:
      containers:
      - env:
        - name: SPRING_PROFILES_ACTIVE
          value: mysql
        - name: SPRING_DATASOURCE_USER
          valueFrom:
            secretKeyRef:
              name: baeldung-db
              key: database-user
        - name: SPRING_DATASOURCE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: baeldung-db
              key: database-password
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 180
        readinessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 30
```

接下来，**我们将在** `**openshift/configmap.yml**,`中保存一个`ConfigMap`，它包含了带有 MySQL URL 的`application.properties`的数据:

```java
apiVersion: v1
kind: ConfigMap
metadata:
  name: spring-boot-bootstrap
data:
  application.properties: |-
    spring.datasource.url=jdbc:mysql://baeldung-db:3306/baeldung_db
```

**在使用命令行客户端与 Openshift 交互之前，我们需要登录**。在 web 控制台的右上角是一个用户图标，我们可以从中选择标有“复制登录命令”的下拉菜单。然后在外壳中使用:

```java
oc login https://192.168.42.122:8443 --token=<some-token>
```

让我们确保使用正确的项目:

```java
oc project baeldung-demo
```

然后**我们上传了`ConfigMap`** `:`

```java
oc create -f openshift/configmap.yml
```

## 5。部署

在部署期间，Fabric8 Maven 插件尝试确定配置的端口。我们的示例应用程序中现有的`application.properties`文件使用一个表达式来定义端口，插件无法解析这个表达式。因此，我们必须注释这一行:

```java
#server.port=${port:8080}
```

从目前的`application.properties`。

我们现在准备好部署了:

```java
mvn clean fabric8:deploy -P openshift 
```

我们可以观察部署进度，直到看到我们的应用程序正在运行:

```java
oc get pods -w
```

应提供一份清单:

```java
NAME                            READY     STATUS    RESTARTS   AGE
baeldung-db-1-9m2cr             1/1       Running   1           1h
spring-boot-bootstrap-1-x6wj5   1/1       Running   0          46s 
```

在测试应用程序之前，我们需要确定路线:

```java
oc get routes
```

将打印当前项目中的路线:

```java
NAME                    HOST/PORT                                                   PATH      SERVICES                PORT      TERMINATION   WILDCARD
spring-boot-bootstrap   spring-boot-bootstrap-baeldung-demo.192.168.42.122.nip.io             spring-boot-bootstrap   8080                    None 
```

现在，让我们通过添加一本书来验证我们的应用程序是否正常工作:

```java
http POST http://spring-boot-bootstrap-baeldung-demo.192.168.42.122.nip.io/api/books \
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

让我们扩展部署以运行 2 个实例:

```java
oc scale --replicas=2 dc spring-boot-bootstrap
```

然后，我们可以使用与前面相同的步骤来观察它的部署、获取路由和测试端点。

Openshift 为[管理性能和扩展](https://web.archive.org/web/20221207181235/https://docs.openshift.com/container-platform/3.11/scaling_performance/index.html)提供了广泛的选项，超出了本文的范围。

## 7 .**。结论**

在本教程中，我们将:

*   安装并配置了 Openshift 开发工具和本地环境
*   部署了 MySQL 服务
*   创建了 ConfigMap 和部署配置以提供数据库连接属性
*   为我们已配置的 Spring Boot 应用程序构建并部署了一个容器，以及
*   测试和扩展应用程序。

更多详情，请查看详细的 Openshift 文档。

我们这里例子的完整源代码一如既往地在 GitHub 的[上。](https://web.archive.org/web/20221207181235/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-bootstrap)