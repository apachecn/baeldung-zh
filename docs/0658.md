# 孔与的入口控制器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-kong-ingress>

## 1.概观

[Kubernetes (K8s)](/web/20221112225153/https://www.baeldung.com/ops/kubernetes) 是自动化软件开发和部署的 orchestrator，是当今 API 托管的流行选择，可在内部或云服务上运行，如谷歌云 Kubernetes 服务(GKS)或亚马逊弹性 Kubernetes 服务(EKS)。另一方面， [Spring](/web/20221112225153/https://www.baeldung.com/spring-boot-minikube) 已经成为最流行的 Java 框架之一。

在本教程中，我们将演示如何使用 Kong Ingress Controller (KIC)在 Kubernetes 上设置一个受保护的环境来部署我们的 Spring Boot 应用程序。我们还将演示 KIC 的高级用法，为我们的应用程序实现一个简单的速率限制器，无需任何编码。

## 2.改进的安全性和访问控制

现代应用程序部署，尤其是 API，面临着许多挑战，例如:隐私法(例如，GPDR)、安全问题(DDOS)和使用跟踪(例如，API 配额和速率限制)。在这种情况下，现代应用程序和 API 需要额外的保护级别来应对所有这些挑战，如防火墙、反向代理、速率限制器和相关服务。尽管 K8s 环境保护我们的应用程序免受这些威胁，但是我们仍然需要采取一些措施来保证应用程序的安全。其中一个措施是部署一个入口控制器，并设置它对您的应用程序的访问规则。

入口是一个对象，它通过向部署的应用程序公开 HTTP / HTTPS 路由并对其实施访问规则，来管理对 K8s 集群和部署在其上的应用程序的外部访问。为了公开应用程序以允许外部访问，我们需要定义入口规则并使用入口控制器，这是一种专用的反向代理和负载平衡器。一般来说，ingress 控制器是由第三方公司提供的，功能各异，比如本文使用的 [Kong Ingress 控制器](https://web.archive.org/web/20221112225153/https://docs.konghq.com/kubernetes-ingress-controller/latest/)。

## 3.设置环境

为了在 Spring Boot 应用程序中演示 Kong Ingress Controller (KIC)的使用，我们需要访问 K8s 集群，这样我们就可以使用完整的 Kubernetes、内部安装或云提供的，或者使用 Minikube 开发我们的示例[应用程序。在启动我们的 K8s 环境之后，我们需要在我们的集群](/web/20221112225153/https://www.baeldung.com/spring-boot-minikube)上部署 Kong 入口控制器[。Kong 公开了一个外部 IP，我们需要使用它来访问我们的应用程序，因此使用该地址创建一个环境变量是一个很好的做法:](https://web.archive.org/web/20221112225153/https://docs.konghq.com/kubernetes-ingress-controller/2.7.x/guides/getting-started/)

```
export PROXY_IP=$(minikube service -n kong kong-proxy --url | head -1)
```

就是这样！安装了 Kong 入口控制器，我们可以通过访问那个`PROXY_IP`来测试它是否在运行:

```
curl -i $PROXY_IP
```

响应应该是 404 错误，没错，因为我们还没有部署任何应用程序，所以应该是没有与这些值匹配的路由。是时候创建一个示例应用程序了，但是在此之前，如果没有 Docker 的话，我们可能需要安装它。为了将我们的应用程序部署到 K8s，我们需要一种创建容器映像的方法，我们可以使用 Docker 来完成。

## 4.创建一个示例 Spring Boot 应用程序

现在我们需要一个 Spring Boot 应用程序，并将其部署到 K8s 集群。要用至少一个公开的 web 资源生成一个简单的 HTTP 服务器应用程序，我们可以这样做:

```
curl https://start.spring.io/starter.tgz -d dependencies=webflux,actuator -d type=maven-project | tar -xzvf -
```

一件重要的事情是选择默认的 Java 版本。如果我们需要使用旧版本，那么就需要一个`javaVersion`属性:

`curl https://start.spring.io/starter.tgz -d dependencies=webflux,actuator -d type=maven-project -d javaVersion=11 | tar -xzvf -`

在这个示例应用程序中，我们选择了`webflux,`，它使用 [Spring WebFlux 和 Netty](/web/20221112225153/https://www.baeldung.com/spring-webflux) 生成了一个反应式 web 应用程序。但是增加了另一个重要的依赖项。`[actuator](/web/20221112225153/https://www.baeldung.com/spring-boot-actuators),`这是一个 Spring 应用的监控工具，已经公开了一些 web 资源，这正是我们需要和孔一起测试的。这样，我们的应用程序已经公开了一些我们可以使用的 web 资源。让我们来建造它:

```
./mvnw install
```

生成的 jar 是可执行的，因此我们可以通过运行它来测试应用程序:

java -jar target/*。罐子

为了测试应用程序，我们需要打开另一个终端并键入以下命令:

```
curl -i http://localhost:8080/actuator/health
```

响应必须是应用程序的健康状态，由执行器提供:

```
HTTP/1.1 200 OK
Content-Type: application/vnd.spring-boot.actuator.v3+json
Content-Length: 15

{"status":"UP"}
```

## 5.从应用程序生成容器图像

将应用程序部署到 Kubernetes 集群的过程包括创建容器映像并将其部署到集群可访问的存储库中。在现实生活中，我们会将图像推送到 DockerHub 或我们自己的私有容器图像注册中心。但是，当我们使用 Minikube 时，让我们将 Docker 客户机环境变量指向 Minikube 的 Docker:

```
$(minikube docker-env)
```

我们可以构建应用程序映像:

```
./mvnw spring-boot:build-image
```

## 6.部署应用程序

现在是时候在我们的 K8s 集群上部署应用程序了。我们需要创建一些 K8s 对象来部署和测试我们的应用程序，所有需要的文件都可以在演示的存储库中找到:

*   带有容器规范的应用程序的部署对象
*   为我们的 Pod 分配集群 IP 地址的服务定义
*   将 Kong 的代理 IP 地址用于我们的路由的入口规则

一个部署对象只是创建运行我们的映像所必需的 pod，这是创建它的 YAML 文件:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: demo
  name: demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: demo
    spec:
      containers:
      - image: docker.io/library/demo:0.0.1-SNAPSHOT
        name: demo
        resources: {}
        imagePullPolicy: Never

status: {} 
```

我们指向 Minikube 内部创建的图像，并获得它的全名。请注意，有必要将`imagePullPolicy`属性指定为`Never `，因为我们没有使用图像注册服务器，所以我们不希望 K8s 尝试下载图像，而是使用已经在其内部 Docker 存档中的图像。我们可以用`kubectl`来部署它:

```
kubectl apply -f serviceDeployment.yaml
```

如果部署成功，我们可以看到消息:

```
deployment.apps/demo created
```

为了让我们的应用程序有一个统一的 IP 地址，我们需要创建一个服务，为它分配一个内部集群范围的 IP 地址，这是创建它的 YAML 文件:

```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: demo
  name: demo
spec:
  ports:
  - name: 8080-8080
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: demo
  type: ClusterIP
status:
  loadBalancer: {} 
```

现在我们也可以用`kubectl`来部署它:

```
kubectl apply -f clusterIp.yaml
```

请注意，我们选择的标签`demo`指向我们部署的应用程序。为了能够从外部访问(在 K8s 集群之外)，我们需要创建一个入口规则，在我们的例子中，我们将它指向路径`/actuator/health`和端口 8080:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo
spec:
  ingressClassName: kong
  rules:
  - http:
      paths:
      - path: /actuator/health
        pathType: ImplementationSpecific
        backend:
          service:
            name: demo
            port:
              number: 8080
```

最后，我们用`kubectl`部署它:

```
kubectl apply -f ingress-rule.yaml 
```

现在我们可以使用 Kong 的代理 IP 地址进行外部访问:

```
$ curl -i $PROXY_IP/actuator/health
HTTP/1.1 200 OK
Content-Type: application/vnd.spring-boot.actuator.v3+json
Content-Length: 49
Connection: keep-alive
X-Kong-Upstream-Latency: 325
X-Kong-Proxy-Latency: 1
Via: kong/3.0.0
```

## 7.演示限速器

我们设法在 Kubernetes 上部署了一个 Spring Boot 应用程序，并使用 Kong Ingress 控制器来提供对它的访问。但是 KIC 做的远不止这些:身份验证、负载平衡、监控、速率限制和其他功能。为了展示 Kong 的强大功能，我们将在应用程序中实现一个简单的速率限制器，将访问限制为每分钟只有五个请求。为此，我们需要在 K8s 集群中创建一个名为`KongClusterPlugin`的对象。YAML 的文件做到了这一点:

```
apiVersion: configuration.konghq.com/v1
kind: KongClusterPlugin
metadata:
  name: global-rate-limit
  annotations:
    kubernetes.io/ingress.class: kong
  labels:
    global: true
config:
  minute: 5
  policy: local
plugin: rate-limiting
```

插件配置允许我们为应用程序指定额外的访问规则，我们将对它的访问限制为每分钟五个请求。让我们应用这个配置并测试结果:

```
kubectl apply -f rate-limiter.yaml
```

为了测试它，我们可以在一分钟内重复之前使用的 CURL 命令五次以上，我们将得到一个 429 错误:

```
curl -i $PROXY_IP/actuator/health
HTTP/1.1 429 Too Many Requests
Date: Sun, 06 Nov 2022 19:33:36 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
RateLimit-Reset: 24
Retry-After: 24
X-RateLimit-Remaining-Minute: 0
X-RateLimit-Limit-Minute: 5
RateLimit-Remaining: 0
RateLimit-Limit: 5
Content-Length: 41
X-Kong-Response-Latency: 0
Server: kong/3.0.0

{
  "message":"API rate limit exceeded"
}
```

我们可以看到响应 HTTP 头通知客户端速率限制。

## 8.清理资源

为了清理这个演示，我们需要按 LIFO 顺序删除所有对象:

```
kubectl delete -f rate-limiter.yaml
kubectl delete -f ingress-rule.yaml
kubectl delete -f clusterIp.yaml
kubectl delete -f serviceDeployment.yaml
```

并停止 Minikube 集群:

```
minikube stop
```

## 9.结论

在本文中，我们演示了如何使用 Kong 入口控制器来管理对部署在 K8s 集群上的 Spring Boot 应用程序的访问。

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221112225153/https://github.com/eugenp/tutorials/tree/master/kubernetes-modules/kubernetes-spring)