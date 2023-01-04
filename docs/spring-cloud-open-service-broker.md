# Spring 云开放服务代理快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-open-service-broker>

## 1.概观

在本教程中，**我们将介绍[Spring Cloud Open Service Broker](https://web.archive.org/web/20220628163443/https://spring.io/projects/spring-cloud-open-service-broker)项目，并学习如何实现 [Open Service Broker API](https://web.archive.org/web/20220628163443/https://www.openservicebrokerapi.org/)** 。

首先，我们将深入研究开放服务代理 API 的规范。然后，我们将学习如何使用 Spring Cloud Open Service Broker 来构建实现 API 规范的应用程序。

最后，我们将探索可以用来保护 service broker 端点的安全机制。

## 2.开放式服务代理 API

Open Service Broker API 项目**允许我们快速为运行在 Cloud Foundry 和 Kubernetes** 等云原生平台上的应用提供后台服务。实质上，API 规范描述了一组 REST 端点，通过它们我们可以提供并连接到这些服务。

特别是，我们可以在云原生平台中使用服务代理来:

*   宣传后勤服务目录
*   供应服务实例
*   创建和删除支持服务和客户端应用程序之间的绑定
*   取消供应服务实例

spring Cloud Open Service Broker**通过提供所需的 web 控制器、域对象和配置**为 Open Service Broker API 兼容实现创建基础。此外，我们需要通过实现适当的[服务代理接口](https://web.archive.org/web/20220628163443/https://docs.spring.io/spring-cloud-open-service-broker/docs/current/apidocs/org/springframework/cloud/servicebroker/service/package-summary.html)来提出我们的业务逻辑。

## 3.自动配置

**为了在我们的应用程序中使用 Spring Cloud Open Service Broker，我们需要添加相关的`starter`工件**。我们可以使用 Maven Central 来搜索最新版本的 [`open-service-broker`启动器](https://web.archive.org/web/20220628163443/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-open-service-broker%22)。

除了云启动器，我们还需要包含一个 Spring Boot web 启动器，以及 Spring WebFlux 或 Spring MVC，来激活自动配置:

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-open-service-broker</artifactId>
    <version>3.1.1.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

自动配置机制为服务代理所需的大多数组件配置默认实现。如果我们愿意，我们可以通过提供我们自己的`open-service-broker` Spring 相关 beans 的实现来覆盖默认行为。

### 3.1.Service Broker 端点路径配置

默认情况下，注册 service broker 端点的上下文路径是“/”。

如果这不理想，我们想改变它，最直接的方法是在我们的应用程序属性或 YAML 文件中设置属性`spring.cloud.openservicebroker.base-path`:

```
spring:
  cloud:
    openservicebroker:
      base-path: /broker
```

在这种情况下，为了查询 service broker 端点，我们首先需要在我们的请求前面加上前缀`/broker/` base-path。

## 4.服务代理示例

让我们使用 Spring Cloud Open service broker 库创建一个 Service Broker 应用程序，并探索 API 是如何工作的。

通过我们的示例，我们将使用 service broker 来提供并连接到一个后备邮件系统。为了简单起见，我们将使用代码示例中提供的虚拟邮件 API。

### 4.1.服务目录

首先，**为了控制我们的服务代理提供哪些服务，我们需要定义一个服务目录**。为了快速初始化服务目录，在我们的示例中，我们将提供一个类型为`[Catalog](https://web.archive.org/web/20220628163443/https://docs.spring.io/spring-cloud-open-service-broker/docs/current/apidocs//org/springframework/cloud/servicebroker/model/catalog/Catalog.html)`的 Spring bean:

```
@Bean
public Catalog catalog() {
    Plan mailFreePlan = Plan.builder()
        .id("fd81196c-a414-43e5-bd81-1dbb082a3c55")
        .name("mail-free-plan")
        .description("Mail Service Free Plan")
        .free(true)
        .build();

    ServiceDefinition serviceDefinition = ServiceDefinition.builder()
        .id("b92c0ca7-c162-4029-b567-0d92978c0a97")
        .name("mail-service")
        .description("Mail Service")
        .bindable(true)
        .tags("mail", "service")
        .plans(mailFreePlan)
        .build();

    return Catalog.builder()
        .serviceDefinitions(serviceDefinition)
        .build();
}
```

如上所示，服务目录包含描述我们的 service broker 可以提供的所有可用服务的元数据。此外，服务的定义有意地宽泛**，因为它可以指数据库、消息队列，或者在我们的例子中，指邮件服务**。

另一个关键点是，每个服务都是从计划中建立起来的，这是另一个通用术语。本质上，**每个计划可以提供不同的功能，花费不同的金额**。

最终，服务目录通过 service broker `/v2/catalog`端点提供给云原生平台:

```
curl http://localhost:8080/broker/v2/catalog

{
    "services": [
        {
            "bindable": true,
            "description": "Mail Service",
            "id": "b92c0ca7-c162-4029-b567-0d92978c0a97",
            "name": "mail-service",
            "plans": [
                {
                    "description": "Mail Service Free Plan",
                    "free": true,
                    "id": "fd81196c-a414-43e5-bd81-1dbb082a3c55",
                    "name": "mail-free-plan"
                }
            ],
            "tags": [
                "mail",
                "service"
            ]
        }
    ]
}
```

因此，云原生平台将向所有服务代理查询服务代理目录端点，以呈现服务目录的聚合视图。

### 4.2.服务供应

一旦我们开始广告服务，我们还需要在我们的代理中提供机制，以便在云平台中提供和管理它们的生命周期。

此外，供应代表的内容因代理而异。在某些情况下，**供应可能涉及启动空数据库、创建消息代理，或者简单地提供一个帐户来访问外部 API**。

就术语而言，由服务代理创建的服务将被称为服务实例。

使用 Spring Cloud Open Service Broker，我们可以通过实现 [`ServiceInstanceService`](https://web.archive.org/web/20220628163443/https://docs.spring.io/spring-cloud-open-service-broker/docs/current/apidocs/org/springframework/cloud/servicebroker/service/ServiceInstanceService.html) 接口来管理服务生命周期。例如，为了在我们的 service broker 中管理服务供应请求，我们必须为`createServiceInstance`方法提供一个实现:

```
@Override
public Mono<CreateServiceInstanceResponse> createServiceInstance(
    CreateServiceInstanceRequest request) {
    return Mono.just(request.getServiceInstanceId())
        .flatMap(instanceId -> Mono.just(CreateServiceInstanceResponse.builder())
            .flatMap(responseBuilder -> mailService.serviceInstanceExists(instanceId)
                .flatMap(exists -> {
                    if (exists) {
                        return mailService.getServiceInstance(instanceId)
                            .flatMap(mailServiceInstance -> Mono.just(responseBuilder
                                .instanceExisted(true)
                                .dashboardUrl(mailServiceInstance.getDashboardUrl())
                                .build()));
                    } else {
                        return mailService.createServiceInstance(
                            instanceId, request.getServiceDefinitionId(), request.getPlanId())
                            .flatMap(mailServiceInstance -> Mono.just(responseBuilder
                                .instanceExisted(false)
                                .dashboardUrl(mailServiceInstance.getDashboardUrl())
                                .build()));
                    }
                })));
}
```

这里，如果不存在具有相同服务实例 id 的邮件服务，我们在内部映射中分配一个新的邮件服务，并提供一个仪表板 URL。我们可以将仪表板视为服务实例的 web 管理界面。

服务供应通过`/v2/service_instances/{instance_id}`端点提供给云原生平台:

```
curl -X PUT http://localhost:8080/broker/v2/service_instances/[[email protected]](/web/20220628163443/https://www.baeldung.com/cdn-cgi/l/email-protection) 
  -H 'Content-Type: application/json' 
  -d '{
    "service_id": "b92c0ca7-c162-4029-b567-0d92978c0a97", 
    "plan_id": "fd81196c-a414-43e5-bd81-1dbb082a3c55"
  }' 

{"dashboard_url":"http://localhost:8080/mail-dashboard/[[email protected]](/web/20220628163443/https://www.baeldung.com/cdn-cgi/l/email-protection)"}
```

简而言之，**当我们提供一项新服务时，我们需要通过服务目录**中广告的`service_id`和`plan_id`。此外，我们需要提供一个唯一的`instance_id`，我们的服务代理将在未来的绑定和取消供应请求中使用它。

### 4.3.服务绑定

在我们提供服务之后，我们希望我们的客户端应用程序开始与它通信。从服务代理的角度来看，这被称为服务绑定。

类似于服务实例和计划，我们应该将绑定视为另一种灵活的抽象，可以在我们的服务代理中使用。一般来说，**我们将提供服务绑定来公开用于访问服务实例**的凭证。

在我们的例子中，如果广告服务的`bindable`字段设置为`true`，我们的服务代理必须提供一个`[ServiceInstanceBindingService](https://web.archive.org/web/20220628163443/https://docs.spring.io/spring-cloud-open-service-broker/docs/current/apidocs/org/springframework/cloud/servicebroker/service/ServiceInstanceBindingService.html)`接口的实现。否则，云平台不会从我们的服务代理调用服务绑定方法。

让我们通过为`createServiceInstanceBinding`方法提供一个实现来处理服务绑定创建请求:

```
@Override
public Mono<CreateServiceInstanceBindingResponse> createServiceInstanceBinding(
    CreateServiceInstanceBindingRequest request) {
    return Mono.just(CreateServiceInstanceAppBindingResponse.builder())
        .flatMap(responseBuilder -> mailService.serviceBindingExists(
            request.getServiceInstanceId(), request.getBindingId())
            .flatMap(exists -> {
                if (exists) {
                    return mailService.getServiceBinding(
                        request.getServiceInstanceId(), request.getBindingId())
                        .flatMap(serviceBinding -> Mono.just(responseBuilder
                            .bindingExisted(true)
                            .credentials(serviceBinding.getCredentials())
                            .build()));
                } else {
                    return mailService.createServiceBinding(
                        request.getServiceInstanceId(), request.getBindingId())
                        .switchIfEmpty(Mono.error(
                            new ServiceInstanceDoesNotExistException(
                                request.getServiceInstanceId())))
                        .flatMap(mailServiceBinding -> Mono.just(responseBuilder
                            .bindingExisted(false)
                            .credentials(mailServiceBinding.getCredentials())
                            .build()));
                }
            }));
}
```

上面的代码生成了一组唯一的凭证——用户名、密码和 URI——通过这些凭证，我们可以连接到新的邮件服务实例并进行身份验证。

Spring Cloud Open Service Broker 框架通过`/v2/service_instances/{instance_id}/service_bindings/{binding_id}`端点公开服务绑定操作:

```
curl -X PUT 
  http://localhost:8080/broker/v2/service_instances/[[email protected]](/web/20220628163443/https://www.baeldung.com/cdn-cgi/l/email-protection)/service_bindings/admin 
  -H 'Content-Type: application/json' 
  -d '{ 
    "service_id": "b92c0ca7-c162-4029-b567-0d92978c0a97", 
    "plan_id": "fd81196c-a414-43e5-bd81-1dbb082a3c55" 
  }'

{
    "credentials": {
        "password": "bea65996-3871-4319-a6bb-a75df06c2a4d",
        "uri": "http://localhost:8080/mail-system/[[email protected]](/web/20220628163443/https://www.baeldung.com/cdn-cgi/l/email-protection)",
        "username": "admin"
    }
}
```

就像服务实例供应一样，我们在绑定请求中使用服务目录中公布的`service_id`和`plan_id`。此外，我们还传递一个惟一的`binding_id`，代理将它用作我们凭证集的用户名。

## 5.Service Broker API 安全性

通常，当服务代理和云原生平台相互通信时，需要一种认证机制。

不幸的是，Open Service Broker API 规范目前并没有涵盖 Service Broker 端点的身份验证部分。正因为如此，Spring Cloud Open Service Broker 库也没有实现任何安全配置。

幸运的是，如果我们需要保护我们的 service broker 端点，我们可以快速使用 Spring Security 来实现[基本认证](/web/20220628163443/https://www.baeldung.com/spring-security-basic-authentication)或 [OAuth 2.0](/web/20220628163443/https://www.baeldung.com/rest-api-spring-oauth2-angular) 机制。在这种情况下，我们应该使用我们选择的身份验证机制对所有 service broker 请求进行身份验证，并在身份验证失败时返回一个`401 Unauthorized`响应。

## 6.结论

在本文中，我们探索了 Spring Cloud 开放服务代理项目。

首先，我们了解了什么是 Open Service Broker API，以及它如何允许我们提供和连接到后台服务。随后，我们看到了如何使用 Spring Cloud Open Service Broker 库快速构建一个符合 Service Broker API 的项目。

最后，我们讨论了如何使用 Spring Security 来保护我们的 service broker 端点。

和往常一样，本教程的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220628163443/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-open-service-broker)