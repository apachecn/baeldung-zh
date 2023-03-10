# Kubernetes 和 Spring Boot 的自我修复应用

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-kubernetes-self-healing-apps>

## 1。简介

在本教程中，我们将讨论 [`Kubernetes`](https://web.archive.org/web/20220523133423/https://kubernetes.io/) 的 [**探测器**](https://web.archive.org/web/20220523133423/https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) ，并演示我们如何利用`Actuator`的 **[`HealthIndicator`](https://web.archive.org/web/20220523133423/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/health/HealthIndicator.html)** 来获得应用程序状态的准确视图。

出于本教程的目的，我们将假设一些关于[`Spring``Boot``Actuator`](/web/20220523133423/https://www.baeldung.com/spring-boot-actuators)[`Kubernetes`](/web/20220523133423/https://www.baeldung.com/kubernetes)和 [`Docker`](/web/20220523133423/https://www.baeldung.com/dockerizing-spring-boot-application) 的预先存在的经验。

## 2 .立方结构探测〔t1〕

`Kubernetes`定义了两种不同的探针，我们可以用它们来定期检查一切是否按预期工作:`liveness`和`readiness`。

### 2.1。活性和就绪性

**通过`Liveness`和`Readiness`探针， [`Kubelet`](https://web.archive.org/web/20220523133423/https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) 可以在检测到有问题时立即采取行动，最大限度地减少应用程序的停机时间。**

两者的配置方式相同，但它们具有不同的语义，并且`Kubelet`根据哪一个被触发而执行不同的动作:

*   `Readiness`–`Readiness`验证我们的`Pod`是否准备好开始接收流量。我们的`P` `od`准备好了，它的所有容器都准备好了
*   `Liveness`–与`readiness`相反，`liveness`检查我们的`Pod`是否应该重启。它可以挑选出我们的应用程序正在运行但处于无法取得进展的状态的用例；例如，它处于死锁状态

我们在容器级别配置两种探测器类型:

```java
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
  - name: goproxy
    image: k8s.gcr.io/goproxy:0.1
    ports:
    - containerPort: 8080
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
      timeoutSeconds: 2
      failureThreshold: 1
      successThreshold: 1
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
      timeoutSeconds: 2
      failureThreshold: 1
      successThreshold: 1
```

我们可以配置许多字段来更精确地控制探测器的行为:

*   `initialDelaySeconds`–创建容器后，**等待`n `秒后开始探测**
*   `periodSeconds`–**该探测器运行的频率**，默认为 10 秒；最短为 1 秒
*   `timeoutSeconds`–**探测超时前等待**的时间，默认为 1 秒；最小值也是 1 秒
*   `failureThreshold`–尝试 **`n`次后放弃**。在`readiness`的情况下，我们的吊舱将被标记为未准备好，而在`liveness`的情况下放弃意味着重启`Pod`。这里的默认值是 3 次失败，最小值是 1 次
*   `successThreshold`–这是**在**失败后，探头被认为成功的最小连续成功次数。它默认为 1 成功，最小值也是 1

在本例中，**我们选择了`tcp`探测器，**然而，我们也可以使用其他类型的探测器。

### 2.2。探针类型

根据我们的用例，一种探针类型可能比另一种更有用。例如，如果我们的容器是一个 web 服务器，使用一个`http`探测器可能比一个`tcp`探测器更可靠。

幸运的是，`Kubernetes`有三种不同类型的探针可供我们使用:

*   `exec`–**在我们的容器**中执行`bash`指令。例如，检查特定文件是否存在。如果指令返回一个失败代码，则探测失败
*   `tcpSocket`–尝试**使用指定的端口**建立到容器的`tcp`连接。如果无法建立连接，则探测失败
*   `httpGet`–**向服务器**发送 HTTP GET 请求，该服务器运行在容器中并监听指定的端口。任何大于或等于 200 且小于 400 的代码都表示成功

值得注意的是，除了我们之前提到的字段之外，`HTTP`探头还有其他字段:

*   `**host**`–要连接的主机名，默认为我们的 pod 的 IP
*   **`scheme`**——应该用于连接的方案，`HTTP`或`HTTPS`，默认为`HTTP`
*   **`path`**–web 服务器上的访问路径
*   **`httpHeaders`**–自定义请求中要设置的标题
*   **`port`**–集装箱中要访问的港口的名称或编号

## 3。弹簧致动器和 Kubernetes 自我修复能力

既然我们已经对`Kubernetes`如何能够检测我们的应用程序是否处于崩溃状态有了一个大致的了解，那么让我们看看**我们** **如何能够** **利用** **的** **的** **Spring 的** **`Actuator`** 的**优势来密切关注我们的应用程序及其依赖关系！**

为了举这些例子，我们打算依靠 [`Minikube`](/web/20220523133423/https://www.baeldung.com/spring-boot-minikube) 。

### 3.1.执行器及其`HealthIndicators`

考虑到 Spring 有许多可以使用的`HealthIndicator`,反映我们的应用程序对`Kubernetes`探测器的一些依赖关系的状态就像给我们的`pom.xml:`添加 [`Actuator`](/web/20220523133423/https://www.baeldung.com/spring-boot-actuators) 依赖关系一样简单

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 3.2。活性示例

让我们从一个将正常启动的应用程序开始， **30** **秒** **后的**将** **转换** **到******断开** **状态**。****

 **我们将通过[创建一个`HealthIndicator`](/web/20220523133423/https://www.baeldung.com/spring-boot-actuators) 来模拟一个中断的状态，验证一个`boolean`变量是否为`true`。我们将变量初始化为`true`，然后我们将安排一个任务在 30 秒后将它更改为`false`:

```java
@Component
public class CustomHealthIndicator implements HealthIndicator {

    private boolean isHealthy = true;

    public CustomHealthIndicator() {
        ScheduledExecutorService scheduled =
          Executors.newSingleThreadScheduledExecutor();
        scheduled.schedule(() -> {
            isHealthy = false;
        }, 30, TimeUnit.SECONDS);
    }

    @Override
    public Health health() {
        return isHealthy ? Health.up().build() : Health.down().build();
    }
}
```

没有

```java
FROM openjdk:8-jdk-alpine
RUN mkdir -p /usr/opt/service
COPY target/*.jar /usr/opt/service/service.jar
EXPOSE 8080
ENTRYPOINT exec java -jar /usr/opt/service/service.jar
```

接下来，我们创建我们的`Kubernetes`模板:

```java
apiVersion: apps/v1
kind: Deployment
metadata:
  name: liveness-example
spec:
  ...
    spec:
      containers:
      - name: liveness-example
        image: dbdock/liveness-example:1.0.0
        ...
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          timeoutSeconds: 2
          periodSeconds: 3
          failureThreshold: 1
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 20
          timeoutSeconds: 2
          periodSeconds: 8
          failureThreshold: 1
```

**我们正在使用一个指向`Actuator's`健康端点的`httpGet`探测器。**我们应用程序状态(及其依赖项)的任何变化都会反映在我们部署的健康性上。

在将我们的应用程序部署到`Kubernetes`之后，我们将能够看到两个探测器都在工作:大约 30 秒后，我们的`Pod`将被标记为未准备好，并从循环中移除；几秒钟后，`Pod`重新启动。

我们可以看到我们的`Pod`执行`kubectl` `describe` `pod` `liveness-example`的事件:

```java
Warning  Unhealthy 3s (x2 over 7s)   kubelet, minikube  Readiness probe failed: HTTP probe failed ...
Warning  Unhealthy 1s                kubelet, minikube  Liveness probe failed: HTTP probe failed ...
Normal   Killing   0s                kubelet, minikube  Killing container with id ...
```

### 3.3。就绪示例

在前面的例子中，我们看到了如何使用一个`HealthIndicator`来反映我们的应用程序在一个`Kubernetes`部署上的健康状态。

让我们在一个不同的用例上使用它:假设我们的应用程序**需要******位****时间** **在** **之前** **能够** **到** **接收** **流量**。例如，它需要将一个文件加载到内存中并验证其内容。**

 **这是我们可以利用`readiness`探测器的一个很好的例子。

让我们修改前面例子中的`HealthIndicator`和`Kubernetes`模板，并使它们适应这个用例:

```java
@Component
public class CustomHealthIndicator implements HealthIndicator {

    private boolean isHealthy = false;

    public CustomHealthIndicator() {
        ScheduledExecutorService scheduled =
          Executors.newSingleThreadScheduledExecutor();
        scheduled.schedule(() -> {
            isHealthy = true;
        }, 40, TimeUnit.SECONDS);
    }

    @Override
    public Health health() {
        return isHealthy ? Health.up().build() : Health.down().build();
    }
}
```

我们将变量初始化为`false`，40 秒后，一个任务将执行并将其设置为`true.`

接下来，我们使用以下模板整理和部署我们的应用程序:

```java
apiVersion: apps/v1
kind: Deployment
metadata:
  name: readiness-example
spec:
  ...
    spec:
      containers:
      - name: readiness-example
        image: dbdock/readiness-example:1.0.0
        ...
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 40
          timeoutSeconds: 2
          periodSeconds: 3
          failureThreshold: 2
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 100
          timeoutSeconds: 2
          periodSeconds: 8
          failureThreshold: 1
```

虽然类似，但我们需要指出探头配置中的一些变化:

*   因为我们知道我们的应用程序需要大约 40 秒来准备接收流量，所以我们将我们的 **`readiness`** 探测器的`**initialDelaySeconds**`增加到 40 秒
*   同样，我们把我们 **`liveness`** 探测器的 **`initialDelaySeconds`** 增加到 100 秒，以避免被`Kubernetes`过早杀死

如果 40 秒后仍未完成，它仍有大约 60 秒时间完成。之后，我们的`liveness`探头将启动并重新启动`Pod.`

## 4。结论

在本文中，我们讨论了`Kubernetes`探针，以及如何使用 Spring 的`Actuator`来改进应用程序的健康监控。

这些例子的完整实现可以在 Github 上找到[。](https://web.archive.org/web/20220523133423/https://github.com/eugenp/tutorials/tree/master/spring-cloud/spring-cloud-kubernetes)****