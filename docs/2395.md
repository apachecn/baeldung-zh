# org . spring 框架

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/org-springframework>

## 1。简介

Spring 框架为可以在任何部署平台上运行的现代基于 Java 的企业应用程序提供了一个清晰而富于表现力的编程和配置模型。

本文涵盖了 Spring 框架的高级概述，主要是提供依赖注入、事务管理、web 应用程序、数据访问、消息传递、测试等支持的 org.springframework 包。

## 2。功能

Spring framework 提供了一系列全面的特性:

*   Spring MVC web 应用程序和 RESTful web 服务框架
*   面向方面的编程，包括 Spring 的声明式事务管理
*   依赖注入
*   控制反转

还有更多。

## 3。Maven 依赖关系

如果你想把 Spring 添加到你的 Maven 项目中，你可以在这里找到更多关于它的信息。

## 4。春季项目

该框架包括许多不同的模块和项目。从配置到安全性，从 web 应用到大数据——无论您的应用需要什么样的基础设施，总有一个 Spring 项目可以帮助您构建它。

从小处着手，按需使用——Spring 是模块化设计。让我们看看这里的一些项目。

### 4.1。Spring Web MVC

[Web MVC](https://web.archive.org/web/20220120203614/https://docs.spring.io/spring/docs/current/spring-framework-reference/html/mvc.html) 框架提供模型-视图-控制器架构，围绕一个`DispatcherServlet`设计，处理所有 HTTP 请求和响应，使应用程序松散耦合。

最棒的是它允许你使用任何对象作为命令或表单对象——不需要实现一个框架特定的接口或基类。它的数据绑定非常灵活:例如，它将类型不匹配视为应用程序可以评估的验证错误，而不是系统错误。

在这里你可以找到完整的[指南](https://web.archive.org/web/20220120203614/https://spring.io/guides/gs/serving-web-content/)。

### 4.2。Spring IO 平台

IO 平台定义了一组依赖项(Spring 框架依赖项都是第三方库),可以包含在 Java 项目中，允许你选择必要的依赖项，而不用担心它们版本之间的兼容性(因为 Spring IO 保证了这一点)。

IO 平台已经过认证，可以与 Java 7 和 8 一起工作。

看看[GitHub 项目](https://web.archive.org/web/20220120203614/https://github.com/spring-io/platform)。

### 4.3。Spring Boot

[Spring Boot](https://web.archive.org/web/20220120203614/https://projects.spring.io/spring-boot/) 让创建基于 Spring 的独立生产级应用变得简单，你可以“直接运行”。这使得用最少的工作创建一个 Spring 驱动的应用程序变得非常容易。

用它创建的应用程序可以很大程度上自动配置一些合理的缺省值，接下来可以用度量标准(多少个请求，请求需要多长时间等等)来改进。).

它由几个(可选)模块组成:

1.  CLI–基于 Groovy 的命令行界面，用于启动/停止 spring boot 创建的应用程序。
2.  [引导核心](https://web.archive.org/web/20220120203614/https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot)——其他模块的基础。
3.  [自动配置](https://web.archive.org/web/20220120203614/https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-autoconfigure)–自动配置各种 Spring 项目的模块。它将检测某些框架的可用性(Spring Batch，Spring Data JPA，Hibernate，JDBC)。
4.  [Actuator](https://web.archive.org/web/20220120203614/https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-actuator)–这个项目添加后，将为您的应用程序启用某些企业功能(安全性、指标、默认错误页面)。
5.  [Starters](https://web.archive.org/web/20220120203614/https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters)–将不同的快速启动项目作为依赖项包含在您的 Maven 或 Gradle 构建文件中。它将拥有该类型应用程序所需的依赖关系。目前，有一个 web 项目的启动项目(基于 tomcat 和 jetty)，Spring Batch，Spring Data JPA，Spring Integration，Spring Security exist。
6.  [工具](https://web.archive.org/web/20220120203614/https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-tools)–Maven 和 Gradle 构建工具以及定制的 Spring Boot 加载器(用于单个可执行文件 jar/war)包含在这个项目中。

我们可以在这里找到 Maven 神器[，看看](https://web.archive.org/web/20220120203614/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter)[的 GitHub 项目](https://web.archive.org/web/20220120203614/https://github.com/spring-projects/spring-boot)。

### 4.4。春季数据

Spring Data 的使命是为数据访问提供一个熟悉的、一致的、基于 Spring 的编程模型，同时仍然保留底层数据存储的特性。

这个项目的主要目标是使构建 Spring 驱动的应用程序变得更加容易，这些应用程序使用新的数据访问技术，如非关系数据库、map-reduce 框架和基于云的数据服务，并提供对关系数据库技术的改进支持。

这是一个伞状项目，包含许多特定于给定数据库的子项目(如 [JPA](https://web.archive.org/web/20220120203614/https://projects.spring.io/spring-data-jpa/) 、 [MongoDB](https://web.archive.org/web/20220120203614/https://projects.spring.io/spring-data-mongodb/) 、 [Redis](https://web.archive.org/web/20220120203614/https://projects.spring.io/spring-data-redis/) 、 [Apache Solr](https://web.archive.org/web/20220120203614/https://projects.spring.io/spring-data-solr/) 、 [Gemfire](https://web.archive.org/web/20220120203614/https://projects.spring.io/spring-data-gemfire/) 、 [Apache Cassandra](https://web.archive.org/web/20220120203614/https://projects.spring.io/spring-data-cassandra/) )。这些项目是通过与这些令人兴奋的技术背后的许多公司和开发商合作开发的。

### 4.5。弹簧安全

Spring Security 是一个专注于为 Java 应用程序提供认证和授权的框架。像所有的 Spring 项目一样，Spring Security 的真正强大之处在于它可以很容易地扩展以满足定制需求。它是在 [Apache 2.0 许可下发布的](https://web.archive.org/web/20220120203614/https://github.com/spring-projects/spring-security/blob/master/LICENSE.txt)，所以你可以放心地在你的项目中使用它。

它也易于学习、部署和管理。它有专门的安全命名空间，为最常见的操作提供指令，只需几行 XML 就可以实现完整的应用程序安全性，并可以保护您的应用程序免受会话固定、点击劫持、跨站点请求伪造等攻击。

Spring Security 还集成了许多其他 Spring 技术，包括 Spring Web Flow、Spring Web Services 和 Pivotal tc Server。

一定要看看 Spring security 的[FAQ](https://web.archive.org/web/20220120203614/https://docs.spring.io/spring-security/site/docs/current/reference/html/appendix.html#appendix-faq)以获得更深入的见解，以及 [Maven](https://web.archive.org/web/20220120203614/https://mvnrepository.com/artifact/org.springframework.security/spring-security-core) 依赖页面。另外，看看 Spring 安全教程的[认证](/web/20220120203614/https://www.baeldung.com/spring-security-authentication-and-registration)、[注册](/web/20220120203614/https://www.baeldung.com/spring-security-registration)，以及[设置](/web/20220120203614/https://www.baeldung.com/spring-security-with-maven) Spring 安全与 Maven **。**

### 4.6。春季社交

Spring Social 是一个框架的扩展，支持应用程序连接软件即服务提供商，如 Twitter、脸书和其他基于认证的 API。它为基于 web 的应用程序提供了一个现成的 OAuth 认证框架。

#### 特点:

*   一个可扩展的服务提供商框架，极大地简化了将本地用户帐户连接到托管提供商帐户的过程。
*   一个连接控制器，处理 Java/Spring web 应用程序、服务提供者和用户之间的授权流。
*   Java 绑定到流行的服务提供商 API，如脸书、Twitter、LinkedIn、TripIt 和 GitHub。
*   一种登录控制器，使用户能够通过服务提供商登录来验证您的应用程序。

#### 入门指南:

*   [访问脸书数据](https://web.archive.org/web/20220120203614/https://projects.spring.io/spring-social-facebook/)
*   [春季社交推特设置](/web/20220120203614/https://www.baeldung.com/spring_social_twitter_setup)
*   [辅助脸书登录](/web/20220120203614/https://www.baeldung.com/facebook-authentication-with-spring-security-and-social)

Spring 提供了很多 GitHub 项目的例子来帮助你快速开始， [Spring Social reference](https://web.archive.org/web/20220120203614/https://docs.spring.io/spring-social/docs/current/reference/htmlsingle/) 也很方便，还有一个[快速开始](https://web.archive.org/web/20220120203614/https://github.com/spring-projects/spring-social/wiki/Quick-Start)页面。

### 4.7。弹簧壳

Spring Shell 是一个交互式的 Shell，可以使用基于 Spring 的编程模型通过命令轻松扩展。

shell 项目的用户可以通过依赖 Spring Shell jar 并添加他们自己的命令(作为 spring beans 上的方法)来轻松构建一个全功能的 shell ( `aka`命令行)应用程序。创建一个命令行应用程序对于与项目的 REST API 交互或者处理本地文件内容会很有用。

[GitHub 项目](https://web.archive.org/web/20220120203614/https://github.com/spring-projects/spring-shell)可以在这里找到。

### 4.8。春天移动

[Spring Mobile](https://web.archive.org/web/20220120203614/https://projects.spring.io/spring-mobile/) 是框架和 [Spring Web MVC](https://web.archive.org/web/20220120203614/https://docs.spring.io/spring/docs/current/spring-framework-reference/html/mvc.html) 的扩展，旨在简化移动 Web 应用的开发。

Spring Mobile 是一个框架，它提供了检测向 Spring 网站发出请求的设备类型的能力，并提供基于该设备的可选视图。像所有的 Spring 项目一样，Spring Mobile 的真正强大之处在于它可以很容易地被扩展。
特点:

*   用于移动和平板设备的服务器端检测的设备解析器抽象
*   站点首选项管理，允许用户指明他或她更喜欢“普通”、“移动”还是“平板”体验
*   一个站点切换器，能够根据用户的设备将用户切换到最合适的站点，无论是移动设备、平板电脑还是普通设备，并且可选地指示一个站点偏好
*   设备感知视图管理，用于组织和管理特定设备的不同视图。

这个[示例应用程序](https://web.archive.org/web/20220120203614/https://github.com/spring-projects/spring-mobile-samples)将帮助您快速入门。

你还可以[检测设备](https://web.archive.org/web/20220120203614/https://docs.spring.io/spring-mobile/docs/current/reference/html/device.html)，处理[网站偏好](https://web.archive.org/web/20220120203614/https://docs.spring.io/spring-mobile/docs/current/reference/html/device.html#site-preference)或者使用 Spring MVC 提供移动网络内容。

### 4.9。春季批次

Spring Batch 是一个轻量级的综合框架，旨在支持对企业系统的日常操作至关重要的批处理应用程序的开发。

在这种情况下，批处理应用程序指的是面向批量数据处理的自动化离线系统。Spring Batch 自动化了这个基本的批处理迭代，提供了将类似的事务作为一个集合进行处理的能力，通常是在没有任何用户交互的离线环境中。

Spring Batch 的工作方式是从数据源读取具有可配置块大小的数据，对其进行处理，最后将其写入资源。

读者的数据源可以是平面文件(文本文件、XML 文件、CSV 文件……)、关系数据库(MySQL……)、MongoDB。类似地，作者可以将数据写入平面文件、关系数据库、MongoDB、mailer 等。

通过[创建一个批处理服务](https://web.archive.org/web/20220120203614/https://spring.io/projects/spring-batch)和其他 Spring 批处理[资源](https://web.archive.org/web/20220120203614/https://spring.io/projects/spring-batch)快速入门。

## 5。核心弹簧组件

在这里，让我们来看看核心弹簧包。

*   [org . spring framework . cache](https://web.archive.org/web/20220120203614/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/cache/package-summary.html)——这个包支持声明式缓存管理的子包和类，在[咖啡因](https://web.archive.org/web/20220120203614/https://github.com/ben-manes/caffeine/)库中设置开源缓存，支持开源缓存的类 [EhCache 2.x](https://web.archive.org/web/20220120203614/http://ehcache.sourceforge.net/) 。

*   org . spring framework . context–这个包建立在 beans 包的基础上，增加了对消息源和观察者设计模式的支持，以及应用程序对象使用一致的 API 获取资源的能力。

*   org . spring framework . core–提供异常处理和版本检测的基本类，以及其他不特定于框架任何部分的核心帮助程序。

*   org . spring framework . expression——这个包提供了`Spring Expression Language`背后的核心抽象。

*   这个包包含了客户端/服务器端 http 的基本抽象。

*   org . spring framework . jdbc–这个包中的类使 JDBC 更容易使用，并减少了常见错误的可能性。
*   这个包包含 jms 的集成类，允许 Spring 风格的 JMS 访问。

*   org . spring framework . jndi–这个包中的类使 JNDI 更容易使用，方便了对存储在 JNDI 的配置的访问，并为 JNDI 访问类提供了有用的超类。

*   [org . Spring framework . ORM . Hibernate 5](https://web.archive.org/web/20220120203614/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/orm/hibernate5/)——提供 [Hibernate 5.x](https://web.archive.org/web/20220120203614/http://www.hibernate.org/) 与 Spring 概念集成的包。

*   org . spring framework . test . util–在单元和集成测试中使用的通用工具类。

这个列表是有限的，仅仅描述了 Spring framework 的核心包。你可以在这里找到完整的列表[。](https://web.archive.org/web/20220120203614/https://docs.spring.io/spring/docs/current/javadoc-api/overview-summary.html)

## 6。结论

在这篇快速概述文章中，我们看了 Spring 生态系统中存在的各种项目，并收集了丰富的 Maven 依赖项、GitHub 项目以及每个项目提供的综合功能，使我们的 web 应用程序安全、可伸缩且易于使用。

我们还看了一下核心包，它们让我们能够专注于应用程序的逻辑方面。