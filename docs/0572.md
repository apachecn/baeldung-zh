# 为什么选择 Spring 作为你的 Java 框架？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-why-to-choose>

## 1.概观

在本文中，我们将介绍作为最流行的 Java 框架之一的 [Spring](https://web.archive.org/web/20220627175755/https://spring.io/) 的主要价值主张。

更重要的是，我们将试图理解 Spring 成为我们选择的框架的原因。Spring 及其组成部分的细节已经在我们之前的教程中广泛涉及。因此，我们将跳过介绍性的“如何”部分，主要关注“为什么”。

## 2.为什么要使用任何框架？

在我们开始任何关于 Spring 的讨论之前，让我们首先理解为什么我们需要使用任何框架。

像 Java 这样的通用编程语言能够支持各种各样的应用程序。更不用说 Java 每天都在被积极地开发和改进。

而且，有无数的开源和专有库在这方面支持 Java。

那么我们到底为什么需要一个框架呢？老实说，使用框架来完成一项任务并不是绝对必要的。但是，出于以下几个原因，使用它通常是明智的:

*   帮助我们关注核心任务，而不是与之相关的样板文件
*   以设计模式的形式汇集了多年的智慧
*   帮助我们遵守行业和监管标准
*   降低应用程序的总拥有成本

我们只是触及了表面，我们必须说这些好处是难以忽视的。但这不可能都是积极的，所以有什么问题:

*   迫使我们**以特定的方式编写应用程序**
*   绑定到特定版本的语言和库
*   增加了应用程序的资源占用

坦率地说，软件开发中没有灵丹妙药，框架当然也不例外。因此，选择哪一个框架或不选择任何框架都应该根据上下文来决定。

希望在本文结束时，我们能够更好地做出关于 Spring in Java 的决定。

## 3.春季生态系统概述

在我们开始对 Spring 框架进行定性评估之前，让我们仔细看看 Spring 生态系统是什么样子的。

Spring 在 2003 年的某个时候出现了当时 Java 企业版发展很快，开发企业应用程序令人兴奋，但也很乏味！

Spring 最初是作为 Java 的控制反转(IoC)容器出现的。我们仍然把 Spring 和它联系在一起，事实上，它是框架的核心，也是基于它开发的其他项目的核心。

### 3.1.弹簧框架

spring framework[被划分为模块](https://web.archive.org/web/20220627175755/https://docs.spring.io/spring/docs/current/spring-framework-reference/index.html)，这使得在任何应用程序中挑选和使用部件变得非常容易:

*   核心(Core):提供核心特性，如依赖注入(DI)、国际化、验证和面向方面编程(AOP)
*   [数据访问](https://web.archive.org/web/20220627175755/https://docs.spring.io/spring/docs/5.1.8.RELEASE/spring-framework-reference/data-access.html#spring-data-tier):支持通过 JTA (Java 事务 API)、JPA (Java 持久 API)、JDBC (Java 数据库连接)进行数据访问
*   [Web](https://web.archive.org/web/20220627175755/https://docs.spring.io/spring/docs/5.1.8.RELEASE/spring-framework-reference/web.html#spring-web) :支持 Servlet API ( [Spring MVC](https://web.archive.org/web/20220627175755/https://docs.spring.io/spring/docs/5.1.8.RELEASE/spring-framework-reference/web.html#spring-web) )和最近反应式 API ( [Spring WebFlux](https://web.archive.org/web/20220627175755/https://docs.spring.io/spring/docs/5.1.8.RELEASE/spring-framework-reference/web-reactive.html#spring-webflux) )，另外支持 WebSockets、STOMP 和 WebClient
*   [集成](https://web.archive.org/web/20220627175755/https://docs.spring.io/spring/docs/5.1.8.RELEASE/spring-framework-reference/integration.html#spring-integration):支持通过 JMS (Java 消息服务)、JMX (Java 管理扩展)和 RMI(远程方法调用)集成到企业 Java 中
*   测试:通过模拟对象、测试夹具、上下文管理和缓存广泛支持单元和集成测试

### 3.2.春季项目

但是，让 Spring 更有价值的是一个强大的生态系统，它多年来一直围绕着 Spring 成长，并继续积极进化。这些被结构化为在 Spring 框架之上开发的 [Spring 项目](https://web.archive.org/web/20220627175755/https://spring.io/projects)。

虽然 Spring 项目的列表很长，而且还在不断变化，但是有几个值得一提:

*   [Boot](/web/20220627175755/https://www.baeldung.com/spring-boot) :为我们提供了一套非常有主见但可扩展的模板，用于在短时间内创建各种基于 Spring 的项目。这使得用嵌入式 Tomcat 或类似的容器创建独立的 Spring 应用程序变得非常容易。
*   [Cloud](/web/20220627175755/https://www.baeldung.com/spring-cloud-series) :支持轻松开发一些常见的分布式系统模式，如服务发现、断路器和 API 网关。它帮助我们减少了在本地、远程甚至托管平台上部署这种样板模式的工作量。
*   [安全性](/web/20220627175755/https://www.baeldung.com/security-spring):提供一个健壮的机制，以高度可定制的方式为基于 Spring 的项目开发认证和授权。通过最少的声明性支持，我们可以抵御常见的攻击，如会话固定、点击劫持和跨站点请求伪造。
*   [Mobile](/web/20220627175755/https://www.baeldung.com/spring-mobile) :提供检测设备并相应调整应用程序行为的能力。此外，支持设备感知视图管理以获得最佳用户体验、站点首选项管理和站点切换器。
*   [Batch](/web/20220627175755/https://www.baeldung.com/introduction-to-spring-batch) :为开发数据归档等企业系统的批处理应用程序提供一个轻量级框架。直观地支持调度、重启、跳过、收集指标和日志记录。此外，通过优化和分区支持大容量作业的扩展。

不用说，这是对 Spring 所能提供的非常抽象的介绍。但是对于 Spring 的组织和广度，它为我们进一步讨论提供了足够的基础。

## 4.春天在行动

习惯上添加一个 hello-world 程序来了解任何新技术。

让我们看看 **Spring 如何让编写一个不仅仅是 hello-world** 的程序变得轻而易举。我们将创建一个应用程序，将 CRUD 操作作为 REST APIs 公开给一个域实体，比如由内存数据库支持的 Employee。此外，我们将使用基本身份验证来保护我们的突变端点。最后，没有好的、旧的单元测试，任何应用程序都不可能是完整的。

### 4.1.项目设置

我们将使用 [Spring Initializr](https://web.archive.org/web/20220627175755/https://start.spring.io/) 来设置我们的 Spring Boot 项目，这是一个方便的在线工具，可以使用正确的依赖项来引导项目。我们将添加 Web、JPA、H2 和安全性作为项目依赖项，以正确设置 Maven 配置。

关于自举的更多细节可以在我们之前的文章中找到。

### 4.2.领域模型和持久性

由于要做的事情很少，我们已经准备好定义我们的领域模型和持久性了。

让我们首先将`Employee`定义为一个简单的 JPA 实体:

```
@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    @NotNull
    private String firstName;
    @NotNull
    private String lastName;
    // Standard constructor, getters and setters
}
```

请注意我们在实体定义中包含的自动生成的 id。

现在我们必须为我们的实体定义一个 JPA 存储库。这就是 Spring 让它变得非常简单的地方:

```
public interface EmployeeRepository 
  extends CrudRepository<Employee, Long> {
    List<Employee> findAll();
}
```

我们所要做的就是定义一个这样的接口， **Spring JPA 将为我们提供一个包含默认和定制操作的实现**。相当整洁！在我们的其他文章中可以找到更多关于[使用 Spring Data JPA](/web/20220627175755/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) 的细节。

### 4.3.控制器

现在我们必须定义一个 web 控制器来路由和处理我们的传入请求:

```
@RestController
public class EmployeeController {
    @Autowired
    private EmployeeRepository repository;
    @GetMapping("/employees")
    public List<Employee> getEmployees() {
        return repository.findAll();
    }
    // Other CRUD endpoints handlers
}
```

实际上，我们所要做的就是**注释类并定义路由元信息**以及每个处理程序方法。

使用[弹簧支架控制器](/web/20220627175755/https://www.baeldung.com/building-a-restful-web-service-with-spring-and-java-based-configuration)在我们之前的文章中有详细介绍。

### 4.4.安全性

现在我们已经定义了一切，但是像创建或删除雇员这样的安全操作呢？我们不希望对这些端点进行未经身份验证的访问！

Spring Security 在这一领域大放异彩:

```
@EnableWebSecurity
public class WebSecurityConfig 
  extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) 
      throws Exception {
        http
          .authorizeRequests()
            .antMatchers(HttpMethod.GET, "/employees", "/employees/**")
            .permitAll()
          .anyRequest()
            .authenticated()
          .and()
            .httpBasic();
    }
    // other necessary beans and definitions
}
```

这里有[更多的细节需要注意](/web/20220627175755/https://www.baeldung.com/spring-security-basic-authentication)去理解，但是最重要的一点要注意的是**声明的方式，在这种方式中我们只允许 GET 操作不受限制**。

### 4.5.测试

现在我们已经做了所有的事情，但是等等，我们如何测试呢？

让我们看看 Spring 是否能让为 REST 控制器编写单元测试变得更容易:

```
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
public class EmployeeControllerTests {
    @Autowired
    private MockMvc mvc;
    @Test
    @WithMockUser()
    public void givenNoEmployee_whenCreateEmployee_thenEmployeeCreated() throws Exception {
        mvc.perform(post("/employees").content(
            new ObjectMapper().writeValueAsString(new Employee("First", "Last"))
            .with(csrf()))
          .contentType(MediaType.APPLICATION_JSON)
          .accept(MediaType.APPLICATION_JSON))
          .andExpect(MockMvcResultMatchers.status()
            .isCreated())
          .andExpect(jsonPath("$.firstName", is("First")))
          .andExpect(jsonPath("$.lastName", is("Last")));
    }
    // other tests as necessary
}
```

正如我们所见， **Spring 为我们提供了必要的基础设施来编写简单的单元和集成测试**，否则这些测试将依赖于 Spring 上下文来初始化和配置。

### 4.6.运行应用程序

最后，我们如何运行这个应用程序？这是 Spring Boot 另一个有趣的方面。尽管我们可以将它打包成一个常规的应用程序，并传统地部署在 Servlet 容器上。

但是这有什么好玩！ **Spring Boot 配备了嵌入式 Tomcat 服务器**:

```
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

这是一个作为引导程序的一部分预先创建的类，具有使用嵌入式服务器启动该应用程序所需的所有细节。

而且，[这是高度可定制的](/web/20220627175755/https://www.baeldung.com/spring-boot-application-configuration)。

## 5.弹簧的替代品

虽然选择使用一个框架相对容易，但是在我们所拥有的选择中，在框架之间做出选择常常是令人畏惧的。但为此，我们必须至少对 Spring 所能提供的特性有一个大致的了解。

正如我们之前所讨论的，**Spring 框架及其项目为企业开发人员提供了广泛的选择**。如果我们对当代的 Java 框架做一个快速的评估，它们甚至不能接近 Spring 提供给我们的生态系统。

然而，对于特定的领域，它们确实形成了一个令人信服的论据来作为选择:

*   Guice :为 Java 应用程序提供了一个健壮的 IoC 容器
*   [Play](/web/20220627175755/https://www.baeldung.com/java-intro-to-the-play-framework) :非常适合作为一个具有反应式支持的 Web 框架
*   [Hibernate](/web/20220627175755/https://www.baeldung.com/hibernate-4-spring) :一个已建立的支持 JPA 的数据访问框架

除此之外，最近还增加了一些功能，提供了比特定领域更广泛的支持，但仍然没有涵盖 Spring 必须提供的所有功能:

*   Micronaut :一个基于 JVM 的框架，为云原生微服务量身定制
*   Quarkus :新时代的 Java 堆栈，承诺提供更快的启动时间和更小的占用空间

显然，完全遍历列表既不必要也不可行，但是我们在这里得到了一个大概的想法。

## 6.那么，为什么选择春天呢？

最后，我们已经构建了所有必需的上下文来解决我们的核心问题，为什么是春天？我们知道框架可以帮助我们开发复杂的企业应用程序。

此外，我们确实了解针对特定问题的选项，如 web、数据访问、框架方面的集成，尤其是 Java。

那么，在所有这些当中，春天在哪里发光呢？我们来探索一下。

### 6.1.可用性

任何框架受欢迎的一个关键方面是开发人员使用它有多容易。通过多种配置选项和配置惯例，开发人员可以很容易地开始并配置他们真正需要的东西。

像 **Spring Boot 这样的项目使得引导一个复杂的 Spring 项目变得微不足道。更不用说，它有出色的文档和教程来帮助任何人登上。**

### 6.2.模块性

Spring 受欢迎的另一个关键方面是它高度模块化的特性。我们可以选择使用整个 Spring 框架，或者只使用必要的模块。此外，我们可以根据需要**有选择地包含一个或多个 Spring 项目**。

此外，我们还可以选择使用其他框架，比如 Hibernate 或 Struts！

### 6.3.顺应

尽管 Spring **并不支持所有的 Jakarta EE 规范，但它支持所有的技术**，经常在必要的地方改进对标准规范的支持。例如，Spring 支持基于 JPA 的存储库，因此切换提供者很简单。

此外，Spring 支持行业规范，如 Spring Web Reactive 下的 [Reactive Stream](https://web.archive.org/web/20220627175755/https://www.reactive-streams.org/) 和[Spring hate as 下的 hate as](/web/20220627175755/https://www.baeldung.com/spring-hateoas-tutorial)。

### 6.4.易测性

任何框架的采用在很大程度上也取决于这样一个事实，即测试构建在它之上的应用程序有多容易。spring in the core**提倡并支持测试驱动开发** (TDD)。

Spring 应用程序主要由 POJOs 组成，这自然使得单元测试相对简单得多。然而，Spring 确实为 MVC 这样的场景提供了模拟对象，否则单元测试会变得复杂。

### 6.5。到期日

Spring 在创新、采用和标准化方面有着悠久的历史。这些年来，它已经变得足够成熟，成为大规模企业应用程序开发中面临的大多数常见问题的默认解决方案。

更令人兴奋的是它的开发和维护非常活跃。每天都在开发对新语言特性和企业集成解决方案的支持。

### 6.6.社区支持

最后但同样重要的是，任何框架甚至库都通过创新在行业中生存下来，没有比社区更适合创新的地方了。Spring 是一个开源软件，由 Pivotal Software 领导，由大型组织和个人开发者联盟支持。

这意味着它仍然是有背景的，而且往往是未来的，这从它旗下的项目数量就可以看出来。

## 7.使用弹簧的理由

有各种各样的应用程序可以从不同级别的 Spring 使用中受益，并且随着 Spring 的发展而快速变化。

然而，我们必须明白，Spring 像其他任何框架一样，有助于管理应用程序开发的复杂性。它帮助我们避免常见的陷阱，并在应用程序随时间增长时保持其可维护性。

这种**是以额外的资源占用和学习曲线**为代价的，不管这种代价有多小。如果真的有一个足够简单的应用程序，并且不希望它变得复杂，那么不使用任何框架可能会更好！

## 8.结论

在本文中，我们讨论了在应用程序开发中使用框架的好处。我们进一步简单讨论了 Spring 框架。

在这个主题上，我们还研究了一些可用于 Java 的替代框架。

最后，我们讨论了迫使我们选择 Spring 作为 Java 框架的原因。

不过，我们应该用一条建议来结束这篇文章。无论听起来多么令人信服，在软件开发中通常没有单一的、放之四海而皆准的解决方案。

因此，我们必须运用我们的智慧，为我们要解决的具体问题选择最简单的解决方案。