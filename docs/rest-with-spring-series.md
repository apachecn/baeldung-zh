# 用弹簧休息教程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-with-spring-series>

构建 REST API 不是一项简单的任务——从高层次的 RESTful 约束到让一切正常运行的细节。

Spring 让 REST 成为了一等公民，这个平台也在飞速发展。随着 Spring 5 的发布， **REST 现在已经久经沙场，完全成熟了**。

通过这篇指南，我的目标是组织关于这个主题的大量信息，并引导你正确地构建一个 API。

该指南从基础知识开始，引导 REST API，Spring MVC 配置，基本定制。

然后，它深入到 REST 的更高级的领域——hate OAS 和分页、错误处理和测试。

![rest - icon](img/567fd3c8428cec1f5acf165e125dc085.png)

## REST API 基础

*   ***[自举一个 Web 应用](/web/20220912073704/https://www.baeldung.com/bootstraping-a-web-application-with-spring-and-java-based-configuration)***
**   ***[建筑休息 API](/web/20220912073704/https://www.baeldung.com/building-a-restful-web-service-with-spring-and-java-based-configuration)*****   ***[弹簧@控制器和@RestController 注解](/web/20220912073704/https://www.baeldung.com/spring-controller-vs-restcontroller)*****   ***[错误处理为休息](/web/20220912073704/https://www.baeldung.com/exception-handling-for-rest-with-spring "Exception Handling for REST with Spring 3") **(流行)*******   ***[实体到 DTO 转换为一个弹簧休息 API](/web/20220912073704/https://www.baeldung.com/entity-to-and-from-dto-for-a-java-spring-application "Exception Handling for REST with Spring 3")*****   ***[春天的请求体和响应体注释](/web/20220912073704/https://www.baeldung.com/spring-request-response-body)*****   ***[如何读取 Spring REST 控制器中的 HTTP 头](/web/20220912073704/https://www.baeldung.com/spring-rest-http-headers)*****   ***[使用 Spring @ResponseStatus 设置 HTTP 状态码](/web/20220912073704/https://www.baeldung.com/spring-response-status)*****   ***[使用 Spring ResponseEntity 来操纵 HTTP 响应](/web/20220912073704/https://www.baeldung.com/spring-response-entity)***********