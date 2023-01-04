# 春季 MVC 内容协商

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-content-negotiation-json-xml>

## 1。概述

本文描述了如何在 Spring MVC 项目中实现内容协商。

通常，有三个选项来确定请求的媒体类型:

*   `**(Deprecated)**`在请求中使用 URL 后缀(扩展名)(例如`.xml/.json`
*   在请求中使用 URL 参数(例如`?format=json`)
*   在请求中使用`Accept`报头

默认情况下，这是 Spring 内容协商管理器尝试使用这三种策略的顺序。如果这些都没有启用，我们可以指定一个默认内容类型的后备。

## 2。内容协商策略

让我们从必要的依赖项开始——我们正在使用 JSON 和 XML 表示，因此对于本文，我们将使用 Jackson for JSON:

```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.10.2</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.10.2</version>
</dependency> 
```

对于 XML 支持，我们可以使用 JAXB、XStream 或更新的 Jackson-XML 支持。

既然我们已经在关于 `[HttpMessageConverters](/web/20220727020632/https://www.baeldung.com/spring-httpmessageconverter-rest),` 的早期文章[中解释了`Accept`头的使用，那么让我们深入关注前两种策略。](/web/20220727020632/https://www.baeldung.com/spring-httpmessageconverter-rest)

## 3。URL 后缀策略

在 Spring Boot 2.6.x 版本中，根据注册的 Spring MVC 处理程序映射匹配请求路径的默认策略从`AntPathMatcher`变成了`PathPatternParser`。

由于 PathPatternParser 不支持后缀模式匹配，所以在使用这种策略之前，我们首先需要使用遗留的路径匹配器。

我们可以在 application.properties 文件中添加**spring . MVC . path match . matching-strategy**，将默认切换回`AntPathMatcher`。

默认情况下，此策略是禁用的，我们需要通过在 application.properties 中设置`spring.mvc.pathmatch.use-suffix-pattern`为 true 来启用它:

```
spring.mvc.pathmatch.use-suffix-pattern=true
spring.mvc.pathmatch.matching-strategy=ant-path-matcher
```

一旦启用，框架就可以检查 URL 的路径扩展，以确定输出内容类型。

在进入配置之前，让我们快速看一个例子。在典型的 Spring 控制器中，我们有以下简单的 API 方法实现:

```
@RequestMapping(
  value = "/employee/{id}", 
  produces = { "application/json", "application/xml" }, 
  method = RequestMethod.GET)
public @ResponseBody Employee getEmployeeById(@PathVariable long id) {
    return employeeMap.get(id);
} 
```

让我们通过使用 JSON 扩展来指定资源的媒体类型来调用它:

```
curl http://localhost:8080/spring-mvc-basics/employee/10.json
```

如果使用 JSON 扩展，我们可能会得到以下结果:

```
{
    "id": 10,
    "name": "Test Employee",
    "contactNumber": "999-999-9999"
}
```

下面是 XML 的请求-响应的样子:

```
curl http://localhost:8080/spring-mvc-basics/employee/10.xml
```

响应正文:

```
<employee>
    <contactNumber>999-999-9999</contactNumber>
    <id>10</id>
    <name>Test Employee</name>
</employee>
```

现在，**如果我们不使用任何扩展**或使用未配置的扩展，将返回默认内容类型:

```
curl http://localhost:8080/spring-mvc-basics/employee/10
```

现在让我们看看如何设置这个策略——使用 Java 和 XML 配置。

### 3.1。Java 配置

```
public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
    configurer.favorPathExtension(true).
    favorParameter(false).
    ignoreAcceptHeader(true).
    useJaf(false).
    defaultContentType(MediaType.APPLICATION_JSON); 
}
```

让我们检查一下细节。

首先，我们启用了路径扩展策略。**同样值得一提的是，从 [Spring Framework 5.2.4](https://web.archive.org/web/20220727020632/https://github.com/spring-projects/spring-framework/issues/24179) 开始，`favorPathExtension(boolean)` 方法已经被弃用，目的是阻止使用路径扩展进行内容协商。**

然后，我们禁用了 URL 参数策略和`Accept`头策略——因为我们只想依靠路径扩展的方式来确定内容的类型。

然后我们关闭 Java 激活框架；如果传入的请求与我们配置的任何策略都不匹配，可以使用 JAF 作为后备机制来选择输出格式。我们禁用它是因为我们要将 JSON 配置为默认的内容类型。请注意**`useJaf()`方法从 Spring Framework 5** 开始被弃用。

最后，我们将 JSON 设置为默认设置。这意味着如果这两种策略都不匹配，所有传入的请求都将被映射到一个服务于 JSON 的控制器方法。

### 3.2。XML 配置

让我们快速看一下完全相同的配置，只使用 XML:

```
<bean id="contentNegotiationManager" 
  class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
    <property name="favorPathExtension" value="true" />
    <property name="favorParameter" value="false"/>
    <property name="ignoreAcceptHeader" value="true" />
    <property name="defaultContentType" value="application/json" />
    <property name="useJaf" value="false" />
</bean> 
```

## 4。URL 参数策略

我们已经在前一节中使用了路径扩展——现在让我们设置 Spring MVC 来使用路径参数。

我们可以通过将`favorParameter`属性的值设置为 true 来启用这个策略。

让我们快速看一下前面的例子是如何工作的:

```
curl http://localhost:8080/spring-mvc-basics/employee/10?mediaType=json
```

下面是 JSON 响应体的内容:

```
{
    "id": 10,
    "name": "Test Employee",
    "contactNumber": "999-999-9999"
}
```

如果我们使用 XML 参数，输出将是 XML 格式:

```
curl http://localhost:8080/spring-mvc-basics/employee/10?mediaType=xml
```

响应正文:

```
<employee>
    <contactNumber>999-999-9999</contactNumber>
    <id>10</id>
    <name>Test Employee</name>
</employee>
```

现在让我们进行配置——同样，首先使用 Java，然后使用 XML。

### 4.1。Java 配置

```
public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
    configurer.favorPathExtension(false).
    favorParameter(true).
    parameterName("mediaType").
    ignoreAcceptHeader(true).
    useJaf(false).
    defaultContentType(MediaType.APPLICATION_JSON).
    mediaType("xml", MediaType.APPLICATION_XML). 
    mediaType("json", MediaType.APPLICATION_JSON); 
} 
```

让我们通读一下这个配置。

首先，当然，路径扩展和`Accept`头策略被禁用(还有 JAF)。

其余配置相同。

### 4.2。XML 配置

```
<bean id="contentNegotiationManager" 
  class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
    <property name="favorPathExtension" value="false" />
    <property name="favorParameter" value="true"/>
    <property name="parameterName" value="mediaType"/>
    <property name="ignoreAcceptHeader" value="true" />
    <property name="defaultContentType" value="application/json" />
    <property name="useJaf" value="false" />

    <property name="mediaTypes">
        <map>
            <entry key="json" value="application/json" />
            <entry key="xml" value="application/xml" />
        </map>
    </property>
</bean>
```

此外，我们可以让**两种策略(扩展和参数)同时启用**:

```
public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
    configurer.favorPathExtension(true).
    favorParameter(true).
    parameterName("mediaType").
    ignoreAcceptHeader(true).
    useJaf(false).
    defaultContentType(MediaType.APPLICATION_JSON).
    mediaType("xml", MediaType.APPLICATION_XML). 
    mediaType("json", MediaType.APPLICATION_JSON); 
}
```

在这种情况下，Spring 将首先寻找路径扩展名，如果不存在，那么将寻找路径参数。如果这两者在输入请求中都不可用，那么将返回默认的内容类型。

## 5。`Accept`头球攻略

如果启用了`Accept`头，Spring MVC 将在传入的请求中查找它的值，以确定表示类型。

我们必须将`ignoreAcceptHeader`的值设置为 false 来启用这种方法，我们正在禁用其他两种策略，这样我们就知道我们只依赖于`Accept`头。

### 5.1。Java 配置

```
public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
    configurer.favorPathExtension(true).
    favorParameter(false).
    parameterName("mediaType").
    ignoreAcceptHeader(false).
    useJaf(false).
    defaultContentType(MediaType.APPLICATION_JSON).
    mediaType("xml", MediaType.APPLICATION_XML). 
    mediaType("json", MediaType.APPLICATION_JSON); 
}
```

### 5.2。XML 配置

```
<bean id="contentNegotiationManager" 
  class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
    <property name="favorPathExtension" value="true" />
    <property name="favorParameter" value="false"/>
    <property name="parameterName" value="mediaType"/>
    <property name="ignoreAcceptHeader" value="false" />
    <property name="defaultContentType" value="application/json" />
    <property name="useJaf" value="false" />

    <property name="mediaTypes">
        <map>
            <entry key="json" value="application/json" />
            <entry key="xml" value="application/xml" />
        </map>
    </property>
</bean>
```

最后，我们需要通过将内容协商管理器插入到整体配置中来打开它:

```
<mvc:annotation-driven content-negotiation-manager="contentNegotiationManager" /> 
```

## 6。结论

我们结束了。我们研究了 Spring MVC 中的内容协商是如何工作的，并且我们关注了几个设置它以使用各种策略来确定内容类型的例子。

这篇文章的完整实现可以在 GitHub 上找到[。](https://web.archive.org/web/20220727020632/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics)