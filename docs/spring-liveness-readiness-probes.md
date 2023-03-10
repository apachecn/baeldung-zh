# Spring Boot 的活跃度和就绪度调查

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-liveness-readiness-probes>

## 1.概观

在本教程中，我们将看到 [Spring Boot 2.3](https://web.archive.org/web/20220628163033/https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.3-Release-Notes) 如何与 [Kubernetes probes](/web/20220628163033/https://www.baeldung.com/spring-boot-kubernetes-self-healing-apps) 集成，以创建更加愉快的云原生体验。

首先，我们将从 Kubernetes 探测器的一些背景开始。然后，我们将切换话题，看看 Spring Boot 2.3 如何支持这些探针。

## 2.库比厄探测器

当使用 Kubernetes 作为我们的编排平台时，每个节点中的 [kubelet](https://web.archive.org/web/20220628163033/https://kubernetes.io/docs/admin/kubelet/) 负责保持该节点中的 pod 健康。

例如，有时我们的应用程序可能需要一点时间才能接受请求。kubelet 可以确保应用程序只在准备好的时候接收请求。此外，如果一个 pod 的主进程由于某种原因崩溃，kubelet 将重新启动容器。

为了履行这些职责，Kubernetes 有两个探针:活性探针和就绪探针。

kubelet 将使用就绪探测器来确定应用程序何时准备好接受请求。更具体地说，当一个 pod 的所有容器都准备好时，它就准备好了。

类似地，**kube let 可以通过活性探针**检查一个 pod 是否仍然活着。基本上，活跃度探测器可以帮助 kubelet 知道何时应该重启容器。

现在我们已经熟悉了这些概念，让我们看看 Spring Boot 积分是如何工作的。

## 3.执行器的活性和就绪性

从 Spring Boot 2.3 开始，`[LivenessStateHealthIndicator](https://web.archive.org/web/20220628163033/https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/availability/LivenessStateHealthIndicator.java) `和`[ReadinessStateHealthIndicator](https://web.archive.org/web/20220628163033/https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/availability/ReadinessStateHealthIndicator.java) `类将公开应用程序的活性和就绪状态。当我们将应用程序部署到 Kubernetes 时，Spring Boot 将自动注册这些健康指标。

**因此，我们可以使用`/actuator/health/liveness` 和`/actuator/health/readiness` 端点分别作为我们的活性和就绪探针。**

例如，我们可以将这些添加到我们的 pod 定义中，以将活跃度探测配置为 HTTP GET 请求:

```java
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
    initialDelaySeconds: 3
    periodSeconds: 3
```

我们通常会让 Spring Boot 决定何时为我们竖起这些探针。但是，如果我们愿意，我们可以在我们的`application.properties.`中手动启用它们

如果我们使用的是 Spring Boot 2.3.0 或 2.3.1，我们可以通过一个配置属性来启用上述探测器:

```java
management.health.probes.enabled=true
```

然而，**从 Spring Boot 2.3.2 开始，由于[配置混乱](https://web.archive.org/web/20220628163033/https://github.com/spring-projects/spring-boot/issues/22107)，该属性被弃用。**

如果我们使用 Spring Boot 2.3.2，我们可以使用新的属性来启用活性和就绪性探测:

```java
management.endpoint.health.probes.enabled=true
management.health.livenessState.enabled=true
management.health.readinessState.enabled=true
```

### 3.1.就绪和活性状态转换

Spring Boot 使用两个枚举来封装不同的就绪和活动状态。对于就绪状态，有一个名为`[ReadinessState](https://web.archive.org/web/20220628163033/https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/availability/ReadinessState.java) `的枚举，其值如下:

*   `ACCEPTING_TRAFFIC `状态表示应用程序准备好接受流量
*   `REFUSING_TRAFFIC `状态意味着应用程序还不愿意接受任何请求

类似地，`[LivenessState](https://web.archive.org/web/20220628163033/https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/availability/LivenessState.java) `枚举用两个值表示应用的活动状态:

*   `CORRECT `值意味着应用程序正在运行，其内部状态是正确的
*   另一方面，`BROKEN `值意味着应用程序运行时会出现一些致命的故障

下面是 Spring 中应用程序生命周期事件的就绪性和活性状态的变化:

1.  注册侦听器和初始化器
2.  准备`Environment`
3.  准备`ApplicationContext`
4.  正在加载 bean 定义
5.  **将活性状态更改为`CORRECT`**
6.  调用应用程序和命令行运行程序
7.  **将准备状态更改为`ACCEPTING_TRAFFIC`**

一旦应用程序启动并运行，我们(和 Spring 本身)可以通过发布适当的 [`AvailabilityChangeEvents`](https://web.archive.org/web/20220628163033/https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/availability/AvailabilityChangeEvent.java) 来改变这些状态。

## 4.管理应用程序可用性

应用程序组件可以通过注入`[ApplicationAvailability](https://web.archive.org/web/20220628163033/https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/availability/ApplicationAvailability.java) `接口来检索当前的就绪和活动状态:

```java
@Autowired private ApplicationAvailability applicationAvailability;
```

那么我们可以如下使用它:

```java
assertThat(applicationAvailability.getLivenessState())
  .isEqualTo(LivenessState.CORRECT);
assertThat(applicationAvailability.getReadinessState())
  .isEqualTo(ReadinessState.ACCEPTING_TRAFFIC);
assertThat(applicationAvailability.getState(ReadinessState.class))
  .isEqualTo(ReadinessState.ACCEPTING_TRAFFIC);
```

### 4.1.更新可用性状态

我们还可以通过发布一个`AvailabilityChangeEvent `事件来更新应用程序状态:

```java
assertThat(applicationAvailability.getLivenessState())
  .isEqualTo(LivenessState.CORRECT);
mockMvc.perform(get("/actuator/health/liveness"))
  .andExpect(status().isOk())
  .andExpect(jsonPath("$.status").value("UP"));

AvailabilityChangeEvent.publish(context, LivenessState.BROKEN);

assertThat(applicationAvailability.getLivenessState())
  .isEqualTo(LivenessState.BROKEN);
mockMvc.perform(get("/actuator/health/liveness"))
  .andExpect(status().isServiceUnavailable())
  .andExpect(jsonPath("$.status").value("DOWN"));
```

如上所示，在发布任何事件之前，`/actuator/health/liveness `端点返回一个 200 OK 响应，带有以下 JSON:

```java
{
    "status": "OK"
}
```

然后，在打破活跃度状态之后，同一个端点返回 503 服务不可用响应，其中包含以下 JSON:

```java
{
    "status": "DOWN"
}
```

当我们改变到准备状态`REFUSING_TRAFFIC, `时，`status `值将是`OUT_OF_SERVICE:`

```java
assertThat(applicationAvailability.getReadinessState())
  .isEqualTo(ReadinessState.ACCEPTING_TRAFFIC);
mockMvc.perform(get("/actuator/health/readiness"))
  .andExpect(status().isOk())
  .andExpect(jsonPath("$.status").value("UP"));

AvailabilityChangeEvent.publish(context, ReadinessState.REFUSING_TRAFFIC);

assertThat(applicationAvailability.getReadinessState())
  .isEqualTo(ReadinessState.REFUSING_TRAFFIC);
mockMvc.perform(get("/actuator/health/readiness"))
  .andExpect(status().isServiceUnavailable())
  .andExpect(jsonPath("$.status").value("OUT_OF_SERVICE"));
```

### 4.2.倾听变化

我们可以注册事件侦听器，以便在应用程序可用性状态发生变化时得到通知:

```java
@Component
public class LivenessEventListener {

    @EventListener
    public void onEvent(AvailabilityChangeEvent<LivenessState> event) {
        switch (event.getState()) {
        case BROKEN:
            // notify others
            break;
        case CORRECT:
            // we're back
        }
    }
}
```

在这里，我们监听应用程序活动状态的任何变化。

## 5.自动配置

在结束之前，让我们看看 Spring Boot 如何在 Kubernetes 部署中自动配置这些探针。`[AvailabilityProbesAutoConfiguration](https://web.archive.org/web/20220628163033/https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-actuator-autoconfigure/src/main/java/org/springframework/boot/actuate/autoconfigure/availability/AvailabilityProbesAutoConfiguration.java) `类负责有条件地注册活跃度和就绪性探测。

事实上，有一个特殊的[条件](https://web.archive.org/web/20220628163033/https://github.com/spring-projects/spring-boot/blob/ba7a42088e86e442c327c7a5143cd275a92ae844/spring-boot-project/spring-boot-actuator-autoconfigure/src/main/java/org/springframework/boot/actuate/autoconfigure/availability/AvailabilityProbesAutoConfiguration.java#L74)，当下列条件之一为`true:`时，它会注册探针

*   Kubernetes 是部署环境
*   `[management.health.probes.enabled](https://web.archive.org/web/20220628163033/https://github.com/spring-projects/spring-boot/blob/ba7a42088e86e442c327c7a5143cd275a92ae844/spring-boot-project/spring-boot-actuator-autoconfigure/src/main/java/org/springframework/boot/actuate/autoconfigure/availability/AvailabilityProbesAutoConfiguration.java#L82) `属性被设置为`true`

当应用程序满足这些条件之一时，自动配置会注册`LivenessStateHealthIndicator `和`ReadinessStateHealthIndicator.`的 beans

## 6.结论

在本文中，我们看到了如何使用 Spring Boot 为 Kubernetes 集成提供的两个健康探针。

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220628163033/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-actuator)