# Spring Web 流程指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-web-flow>

## 1。概述

Spring Web Flow 建立在 Spring MVC 之上，允许在 Web 应用程序中实现流。它用于创建一系列步骤，引导用户完成一个流程或一些业务逻辑。

在这个快速教程中，我们将浏览**一个用户激活流程**的简单例子。用户会看到一个页面，点击`Activate`按钮继续，或点击`Cancel`按钮取消激活。

这里并不是假设我们已经建立了 Spring MVC web 应用程序。

## 2。设置

让我们从将 Spring Web 流依赖项添加到`pom.xml`开始:

```java
<dependency>
    <groupId>org.springframework.webflow</groupId>
    <artifactId>spring-webflow</artifactId>
    <version>2.5.0.RELEASE</version>
</dependency>
```

Spring Web Flow 的最新版本可以在[中央 Maven 资源库](https://web.archive.org/web/20220628152120/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.webflow%22%20AND%20a%3A%22spring-webflow%22)中找到。

## 3。创建流程

现在让我们创建一个简单的流程。如前所述，流程是指导用户完成一个过程的一系列步骤。目前，这只能通过基于 XML 的配置来实现。

**流程中的每一步称为一个** `**state**.`

对于这个简单的例子，我们将使用一个`view-state`。`view-state`是流程中呈现匹配视图的一个步骤。`view-state`指的是应用程序中的一个页面(`WEB-INF/view),`)，其中`view-state`的 id 是它所指页面的名称。

 *我们还将使用一个`transition`元素。**一个`transition`元素用于处理在一个特定的`state`中发生的事件。**

对于这个示例流程，我们将设置三个`view-states`—`activation`、`success`和`failure`。

这个流程非常简单。起点是`activation`视图。如果一个`activate`事件被触发，它应该转移到`success`视图。如果`cancel`事件被触发，它应该转换到`failure`视图。`transition`元素处理`view-state:`中发生的按钮点击事件

```java
<view-state id="activation">
    <transition on="activate" to="success"/>
    <transition on="cancel" to="failure"/>
</view-state>

<view-state id="success" />

<view-state id="failure" />
```

初始激活页面由 id `activation` 表示，位于`WEB-INF/view/activation.jsp`中，是一个简单的页面，有两个按钮`activate`和`cancel`。点击按钮来触发我们的转换，将用户发送到成功的`view-state` ( `WEB-INF/view/success.jsp`)或失败的`view-state` ( `WEB-INF/view/failure.jsp):`

```java
<body>
    <h2>Click to activate account</h2>

    <form method="post" action="${flowExecutionUrl}">
        <input type="submit" name="_eventId_activate" value="activate" />
        <input type="submit" name="_eventId_cancel" value="cancel" />
    </form>
</body>
```

我们使用`flowExecutionUrl`来访问当前流执行`view-state`的上下文相关的 URI。

## 4。配置流程

接下来，我们将在 Web 环境中配置 Spring Web Flow。我们将通过设置流注册和流构建器服务来实现这一点。

流注册允许我们指定我们的流的位置，并且如果正在使用的话，还可以指定流构建器服务。

流构建器服务帮助我们定制用于构建流的服务和设置。

我们可以定制的服务之一是`view-factory-creator`。 `view-factory-creator`允许我们定制 Spring Web Flow 使用的`ViewFactoryCreator`。因为我们使用的是 Spring MVC，所以我们可以在 Spring MVC 配置中配置 Spring Web Flow 来使用视图解析器。

下面是我们将如何为我们的示例配置 Spring Web Flow:

```java
@Configuration
public class WebFlowConfig extends AbstractFlowConfiguration {

    @Autowired
    private WebMvcConfig webMvcConfig;

    @Bean
    public FlowDefinitionRegistry flowRegistry() {
        return getFlowDefinitionRegistryBuilder(flowBuilderServices())
          .addFlowLocation("/WEB-INF/flows/activation-flow.xml", "activationFlow")
          .build();
    }

    @Bean
    public FlowExecutor flowExecutor() {
        return getFlowExecutorBuilder(flowRegistry()).build();
    }

    @Bean
    public FlowBuilderServices flowBuilderServices() {
        return getFlowBuilderServicesBuilder()
          .setViewFactoryCreator(mvcViewFactoryCreator())
          .setDevelopmentMode(true).build();
    }

    @Bean
    public MvcViewFactoryCreator mvcViewFactoryCreator() {
        MvcViewFactoryCreator factoryCreator = new MvcViewFactoryCreator();
        factoryCreator.setViewResolvers(
          Collections.singletonList(this.webMvcConfig.viewResolver()));
        factoryCreator.setUseSpringBeanBinding(true);
        return factoryCreator;
    }
}
```

我们也可以使用 XML 进行配置:

```java
<bean class="org.springframework.webflow.mvc.servlet.FlowHandlerMapping">
    <property name="flowRegistry" ref="activationFlowRegistry"/>
</bean>

<flow:flow-builder-services id="flowBuilderServices"
  view-factory-creator="mvcViewFactoryCreator"/>

<bean id="mvcViewFactoryCreator" 
  class="org.springframework.webflow.mvc.builder.MvcViewFactoryCreator">
    <property name="viewResolvers" ref="jspViewResolver"/>
</bean>

<flow:flow-registry id="activationFlowRegistry" 
  flow-builder-services="flowBuilderServices">
    <flow:flow-location id="activationFlow" path="/WEB-INF/flows/activation-flow.xml"/>
</flow:flow-registry>

<bean class="org.springframework.webflow.mvc.servlet.FlowHandlerAdapter">
    <property name="flowExecutor" ref="activationFlowExecutor"/>
</bean>
<flow:flow-executor id="activationFlowExecutor" 
  flow-registry="activationFlowRegistry"/>
```

## 5。导航流程

要浏览这些流程，请启动 web 应用程序并转到[http://localhost:8080/{ context-path }/activation flow](https://web.archive.org/web/20220628152120/http://localhost:8080/{context-path}/activationFlow)。要启动应用程序，将其部署在应用服务器上，如 [Tomcat](/web/20220628152120/https://www.baeldung.com/tomcat-deploy-war) 或 [Jetty](/web/20220628152120/https://www.baeldung.com/deploy-to-jetty) 。

这将我们送到流的初始页面，这是在我们的流配置中指定的`activation`页面:

[![activate account 1](img/99f9758df557349fde4b64d60e813cf2.png)](/web/20220628152120/https://www.baeldung.com/wp-content/uploads/2017/05/activate-account-1.png)

您可以点击`activate`按钮进入成功页面:

[![activation successful 1](img/da221f7091458494346da26b31fe58c7.png)](/web/20220628152120/https://www.baeldung.com/wp-content/uploads/2017/05/activation-successful-1-1.png) 或`cancel`按钮进入故障页面:

[![activation failed 1](img/99b2596844daffa80d8c767f1947b655.png)](/web/20220628152120/https://www.baeldung.com/wp-content/uploads/2017/05/activation-failed-1-1.png)

## 6。结论

在本文中，我们使用了一个简单的例子来指导如何使用 Spring Web Flow。

你可以在 GitHub 上找到本文[的完整源代码和所有代码片段。](https://web.archive.org/web/20220628152120/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-webflow)*