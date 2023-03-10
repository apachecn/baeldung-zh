# 春季云库伯内特指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-kubernetes>

## 1.概观

当我们构建微服务解决方案时， [Spring Cloud](/web/20220627182516/https://www.baeldung.com/spring-cloud-series) 和 [Kubernetes](/web/20220627182516/https://www.baeldung.com/kubernetes) 都是最佳解决方案，因为它们提供了解决最常见挑战的组件。然而，如果我们决定选择 Kubernetes 作为我们解决方案的主要容器管理器和部署平台，我们仍然可以主要通过 [Spring Cloud Kubernetes](https://web.archive.org/web/20220627182516/https://cloud.spring.io/spring-cloud-static/spring-cloud-kubernetes/2.1.0.RC1/single/spring-cloud-kubernetes.html) 项目来使用 Spring Cloud 的有趣功能。

这个相对较新的项目无疑为 Spring Boot 和 T2 的应用提供了与 Kubernetes 的轻松集成。在开始之前，看看如何在 Minikube (一个本地 Kubernetes 环境`.`)上部署一个 [Spring Boot 应用程序可能会有所帮助](/web/20220627182516/https://www.baeldung.com/spring-boot-minikube)

在本教程中，我们将:

*   在本地机器上安装 minitube
*   开发一个微服务架构示例，其中两个独立的 Spring Boot 应用程序通过 REST 进行通信
*   使用 Minikube 在单节点集群上设置应用程序
*   使用`YAML`配置文件部署应用程序

## 2.方案

在我们的示例中，我们使用的场景是旅行社向不时查询旅行社服务的客户提供各种交易。我们将使用它来演示:

*   **通过 Spring Cloud Kubernetes 进行服务发现**
*   **配置管理**并使用 Spring Cloud Kubernetes Config 将 Kubernetes 配置图和秘密注入到应用 pods 中
*   **负载平衡**使用 Spring Cloud Kubernetes Ribbon

## 3.环境设置

首先，我们需要**在我们的本地机器**上安装 [Minikube](https://web.archive.org/web/20220627182516/https://kubernetes.io/docs/setup/minikube/) ，最好是一个 VM 驱动程序，比如 [VirtualBox](https://web.archive.org/web/20220627182516/https://www.virtualbox.org/) 。在进行这个环境设置之前，还建议先看看 [Kubernetes](https://web.archive.org/web/20220627182516/https://kubernetes.io/) 及其主要特性。

让我们启动本地单节点 Kubernetes 集群:

```java
minikube start --vm-driver=virtualbox
```

此命令使用 VirtualBox 驱动程序创建运行 Minikube 集群的虚拟机。`kubectl`中的默认上下文现在将是`minikube`。然而，为了能够在上下文之间切换，我们使用:

```java
kubectl config use-context minikube
```

启动 Minikube 后，我们可以**连接到 Kubernetes 仪表板**以轻松访问日志和监控我们的服务、pod、配置图和机密:

```java
minikube dashboard 
```

### 3.1.部署

首先，让我们从 [GitHub](https://web.archive.org/web/20220627182516/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-kubernetes) 中得到我们的例子。

此时，我们可以从父文件夹中运行“deployment-travel-client.sh”脚本，或者逐个执行每个指令，以便很好地掌握该过程:

```java
### build the repository
mvn clean install

### set docker env
eval $(minikube docker-env)

### build the docker images on minikube
cd travel-agency-service
docker build -t travel-agency-service .
cd ../client-service
docker build -t client-service .
cd ..

### secret and mongodb
kubectl delete -f travel-agency-service/secret.yaml
kubectl delete -f travel-agency-service/mongo-deployment.yaml

kubectl create -f travel-agency-service/secret.yaml
kubectl create -f travel-agency-service/mongo-deployment.yaml

### travel-agency-service
kubectl delete -f travel-agency-service/travel-agency-deployment.yaml
kubectl create -f travel-agency-service/travel-agency-deployment.yaml

### client-service
kubectl delete configmap client-service
kubectl delete -f client-service/client-service-deployment.yaml

kubectl create -f client-service/client-config.yaml
kubectl create -f client-service/client-service-deployment.yaml

# Check that the pods are running
kubectl get pods
```

## 4.服务发现

这个项目为我们提供了 Kubernetes 中的`ServiceDiscovery`接口的实现。在微服务环境中，通常有多个 pod 运行相同的服务。 **Kubernetes 将服务公开为端点的集合**，可以从运行在同一个 Kubernetes 集群的 pod 中的 Spring Boot 应用程序中获取和访问这些端点。

例如，在我们的例子中，我们有旅行社服务的多个副本，从我们的客户端服务作为`http://travel-agency-service:8080`访问。然而，这在内部会转化为访问不同的 pod，例如`travel-agency-service-7c9cfff655-4hxnp`。

Spring Cloud Kubernetes Ribbon 利用这一特性在服务的不同端点之间实现负载平衡。

我们可以通过在客户端应用程序上添加[spring-cloud-starter-kubernetes](https://web.archive.org/web/20220627182516/https://search.maven.org/search?q=g:org.springframework.cloud%20a:spring-cloud-starter-kubernetes)依赖项来轻松使用服务发现:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-kubernetes</artifactId>
</dependency>
```

同样，我们应该添加`@EnableDiscoveryClient`，并通过在我们的类中使用`@Autowired`将`DiscoveryClient`注入到`ClientController`中:

```java
@SpringBootApplication
@EnableDiscoveryClient
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

```java
@RestController
public class ClientController {
    @Autowired
    private DiscoveryClient discoveryClient;
}
```

## 5.配置映射

通常，**微服务需要某种配置管理**。例如，在 Spring Cloud 应用程序中，我们将使用 Spring Cloud 配置服务器。

然而，我们可以通过使用 Kubernetes 提供的 [ConfigMaps](https://web.archive.org/web/20220627182516/https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) 来实现这一点——前提是我们打算仅将它用于非敏感、未加密的信息。或者，如果我们想要分享的信息是敏感的，那么我们应该选择使用[机密](https://web.archive.org/web/20220627182516/https://kubernetes.io/docs/concepts/configuration/secret/)来代替。

在我们的例子中，我们在`client-service` Spring Boot 应用程序上使用配置映射。让我们创建一个`client-config.` yaml 文件来定义`client-service`的配置图:

```java
apiVersion: v1 by d
kind: ConfigMap
metadata:
  name: client-service
data:
  application.properties: |-
    bean.message=Testing reload! Message from backend is: %s <br/> Services : %s
```

**配置图的名称必须与我们的“application.properties”文件中指定的应用程序名称**相匹配，这一点很重要。这种情况下是 `client-service`。接下来，我们应该在 Kubernetes 上为`client-service`创建配置图:

```java
kubectl create -f client-config.yaml
```

现在，让我们用`@Configuration`和`@ConfigurationProperties`创建一个配置类`ClientConfig`，并注入到`ClientController`中:

```java
@Configuration
@ConfigurationProperties(prefix = "bean")
public class ClientConfig {

    private String message = "Message from backend is: %s <br/> Services : %s";

    // getters and setters
}
```

```java
@RestController
public class ClientController {

    @Autowired
    private ClientConfig config;

    @GetMapping
    public String load() {
        return String.format(config.getMessage(), "", "");
    }
}
```

如果我们不指定 ConfigMap，那么我们应该会看到默认消息，它是在类中设置的。但是，当我们创建配置映射时，这个默认消息会被那个属性覆盖。

此外，每次我们决定更新配置映射时，页面上的消息都会相应地改变:

```java
kubectl edit configmap client-service
```

## 6.秘密

让我们通过查看示例中 MongoDB 连接设置的规范来看看[秘密](https://web.archive.org/web/20220627182516/https://kubernetes.io/docs/concepts/configuration/secret/)是如何工作的。我们将在 Kubernetes 上创建环境变量，然后将这些变量注入到 Spring Boot 应用程序中。

### 6.1.创造一个秘密

第一步是创建一个`secret.yaml`文件，将`username`和`password`编码为`Base 64`:

```java
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
data:
  username: dXNlcg==
  password: cDQ1NXcwcmQ=
```

让我们在 Kubernetes 集群上应用这个秘密配置:

```java
kubectl apply -f secret.yaml
```

### 6.2.创建一个 MongoDB 服务

我们现在应该创建 MongoDB 服务和部署文件`travel-agency-deployment.yaml`。特别是，在部署部分，我们将使用我们之前定义的秘密`username`和`password`:

```java
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: mongo
spec:
  replicas: 1
  template:
    metadata:
      labels:
        service: mongo
      name: mongodb-service
    spec:
      containers:
      - args:
        - mongod
        - --smallfiles
        image: mongo:latest
        name: mongo
        env:
          - name: MONGO_INITDB_ROOT_USERNAME
            valueFrom:
              secretKeyRef:
                name: db-secret
                key: username
          - name: MONGO_INITDB_ROOT_PASSWORD
            valueFrom:
              secretKeyRef:
                name: db-secret
                key: password
```

默认情况下，`mongo:latest`映像将在名为`admin.`的数据库上创建一个带有`username`和`password`的用户

### 6.3.在旅行社服务上设置 MongoDB

更新应用程序属性以添加数据库相关信息非常重要。虽然我们可以自由地指定数据库名称`admin`，但是这里我们隐藏了最敏感的信息，比如`username`和`password`:

```java
spring.cloud.kubernetes.reload.enabled=true
spring.cloud.kubernetes.secrets.name=db-secret
spring.data.mongodb.host=mongodb-service
spring.data.mongodb.port=27017
spring.data.mongodb.database=admin
spring.data.mongodb.username=${MONGO_USERNAME}
spring.data.mongodb.password=${MONGO_PASSWORD}
```

现在，让我们看一下我们的`travel-agency-deployment`属性文件，用连接到`mongodb-service`所需的用户名和密码信息来更新服务和部署。

下面是文件的相关部分，其中一部分与 MongoDB 连接相关:

```java
env:
  - name: MONGO_USERNAME
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: username
  - name: MONGO_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: password
```

## 7.用丝带交流

在微服务环境中，我们通常需要复制服务的 pods 列表，以便执行负载平衡。这是通过使用 Spring Cloud Kubernetes Ribbon 提供的机制来实现的。这个机制可以**自动发现并到达特定服务**的所有端点，随后，它用关于端点的信息填充 Ribbon `ServerList`。

让我们从将`[spring-cloud-starter-kubernetes-ribbon](https://web.archive.org/web/20220627182516/https://search.maven.org/search?q=g:org.springframework.cloud%20a:spring-cloud-starter-kubernetes-ribbon)`依赖项添加到我们的`client-service` pom.xml 文件开始:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-kubernetes-ribbon</artifactId>
</dependency>
```

下一步是将注释`@RibbonClient`添加到我们的`client-service`应用程序中:

```java
@RibbonClient(name = "travel-agency-service")
```

当端点列表被填充时，Kubernetes 客户机将搜索当前名称空间/项目中与使用`@RibbonClient`注释定义的服务名相匹配的注册端点。

我们还需要在应用程序属性中启用功能区客户端:

```java
ribbon.http.client.enabled=true
```

## 8.附加功能

### 8.1.高起鳞癣

[**Hystrix**](/web/20220627182516/https://www.baeldung.com/introduction-to-hystrix) **有助于构建容错和弹性应用程序。**它的主要目标是快速失败和快速恢复。

特别是，在我们的例子中，我们使用 Hystrix 通过用`@EnableCircuitBreaker`注释 Spring Boot 应用程序类来实现`client-server`上的断路器模式。

此外，我们通过用`@HystrixCommand()`注释方法`TravelAgencyService.getDeals()`来使用回退功能。这意味着在回退的情况下，将调用`getFallBackName()`并返回“回退”消息:

```java
@HystrixCommand(fallbackMethod = "getFallbackName", commandProperties = { 
    @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "1000") })
public String getDeals() {
    return this.restTemplate.getForObject("http://travel-agency-service:8080/deals", String.class);
}

private String getFallbackName() {
    return "Fallback";
}
```

### 8.2.Pod 健康指示器

我们可以利用 Spring Boot `HealthIndicator`和 Spring Boot[致动器](/web/20220627182516/https://www.baeldung.com/spring-boot-actuators)向用户展示健康相关信息。

特别是，Kubernetes 健康指标提供:

*   下的 name
*   国际电脑互联网地址
*   命名空间
*   服务帐户
*   节点名
*   一个标志，指示 Spring Boot 应用程序是在 Kubernetes 内部还是外部

## 9.结论

在本文中，我们提供了 Spring Cloud Kubernetes 项目的全面概述。

那么我们为什么要使用它呢？如果我们支持 Kubernetes 作为一个微服务平台，但仍然欣赏 Spring Cloud 的功能，那么 Spring Cloud Kubernetes 为我们提供了两个世界的最佳选择。

这个例子的完整源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220627182516/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-kubernetes)