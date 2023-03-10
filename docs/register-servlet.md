# 如何在 Java 中注册 Servlet

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/register-servlet>

## 1。简介

本文将概述如何在 Jakarta EE 和 Spring Boot 注册 servlet。具体来说，我们将看看在 Jakarta EE 中注册 Java Servlet 的两种方法——一种使用`web.xml`文件，另一种使用注释。然后，我们将使用 XML 配置、Java 配置和可配置属性在 Spring Boot 注册 servlets。

一篇关于 servlets 的很棒的介绍性文章可以在这里找到。

## 2。在雅加达注册 servlet EE

让我们看一下在 Jakarta EE 中注册 servlet 的两种方法。首先，我们可以通过`web.xml`注册一个 servlet。或者，我们可以使用 Jakarta EE `@WebServlet`注释。

### 2.1。Via `web.xml`

在 Jakarta EE 应用程序中注册 servlet 的最常见方式是将其添加到您的`web.xml`文件中:

```java
<welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.htm</welcome-file>
    <welcome-file>index.jsp</welcome-file>
</welcome-file-list>
<servlet>
    <servlet-name>Example</servlet-name>
    <servlet-class>com.baeldung.Example</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>Example</servlet-name>
    <url-pattern>/Example</url-pattern>
</servlet-mapping>
```

如您所见，这涉及到两个步骤:(1)将我们的 servlet 添加到`servlet`标记中，确保也指定 servlet 所在的类的源路径，以及(2)指定 servlet 将在`url-pattern`标记中暴露的 URL 路径。

Jakarta EE `web.xml`文件通常位于`WebContent/WEB-INF`中。

### 2.2。通过注释

现在让我们使用自定义 servlet 类上的`@WebServlet`注释来注册我们的 servlet。这消除了在`server.xml`中 servlet 映射和在`web.xml`中注册 servlet 的需要:

```java
@WebServlet(
  name = "AnnotationExample",
  description = "Example Servlet Using Annotations",
  urlPatterns = {"/AnnotationExample"}
)
public class Example extends HttpServlet {	

    @Override
    protected void doGet(
      HttpServletRequest request, 
      HttpServletResponse response) throws ServletException, IOException {

        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        out.println("<p>Hello World!</p>");
    }
}
```

上面的代码演示了如何将注释直接添加到 servlet 中。servlet 仍然可以在与以前相同的 URL 路径上使用。

## 3。在 Spring Boot 注册 Servlets】

现在我们已经展示了如何在 Jakarta EE 中注册 servlet，让我们看看在 Spring Boot 应用程序中注册 servlet 的几种方法。

### 3.1。程序化注册

Spring Boot 支持 web 应用程序的 100%编程配置。

首先，我们将实现 `WebApplicationInitializer`接口，然后实现`WebMvcConfigurer`接口，该接口允许您覆盖预设的默认值，而不是必须指定每个特定的配置设置，从而节省您的时间，并允许您使用一些现成的可靠设置。

让我们来看一个示例`WebApplicationInitializer`实现:

```java
public class WebAppInitializer implements WebApplicationInitializer {

    public void onStartup(ServletContext container) throws ServletException {
        AnnotationConfigWebApplicationContext ctx
          = new AnnotationConfigWebApplicationContext();
        ctx.register(WebMvcConfigure.class);
        ctx.setServletContext(container);

        ServletRegistration.Dynamic servlet = container.addServlet(
          "dispatcherExample", new DispatcherServlet(ctx));
        servlet.setLoadOnStartup(1);
        servlet.addMapping("/");
     }
}
```

接下来，让我们实现`WebMvcConfigurer`接口:

```java
@Configuration
public class WebMvcConfigure implements WebMvcConfigurer {

    @Bean
    public ViewResolver getViewResolver() {
        InternalResourceViewResolver resolver
          = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/");
        resolver.setSuffix(".jsp");
        return resolver;
    }

    @Override
    public void configureDefaultServletHandling(
      DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
          .addResourceLocations("/resources/").setCachePeriod(3600)
          .resourceChain(true).addResolver(new PathResourceResolver());
    }
}
```

上面我们明确指定了 JSP servlets 的一些默认设置，以便支持`.jsp`视图和静态资源服务。

### 3.2。XML 配置

在 Spring Boot 内配置和注册 servlets 的另一种方法是通过`web.xml`:

```java
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/dispatcher.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

用于在 Spring 中指定配置的`web.xml`与 Jakarta EE 中的类似。上面，您可以看到我们如何通过`servlet`标签下的属性指定更多的参数。

这里我们使用另一个 XML 来完成配置:

```java
<beans ...>

    <context:component-scan base-package="com.baeldung"/>

    <bean 
      class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

记住你的春天`web.xml`通常会住在`src/main/webapp/WEB-INF`。

### 3.3。结合 XML 和编程注册

让我们将 XML 配置方法与 Spring 的编程配置结合起来:

```java
public void onStartup(ServletContext container) throws ServletException {
   XmlWebApplicationContext xctx = new XmlWebApplicationContext();
   xctx.setConfigLocation('classpath:/context.xml');
   xctx.setServletContext(container);

   ServletRegistration.Dynamic servlet = container.addServlet(
     "dispatcher", new DispatcherServlet(ctx));
   servlet.setLoadOnStartup(1);
   servlet.addMapping("/");
}
```

让我们也配置一下 dispatcher servlet:

```java
<beans ...>

    <context:component-scan base-package="com.baeldung"/>
    <bean class="com.baeldung.configuration.WebAppInitializer"/>
</beans>
```

### 3.4。Bean 注册

我们还可以使用`ServletRegistrationBean`以编程方式配置和注册我们的 servlets。下面我们这样做是为了注册一个`HttpServlet`(它实现了`javax.servlet.Servlet`接口):

```java
@Bean
public ServletRegistrationBean exampleServletBean() {
    ServletRegistrationBean bean = new ServletRegistrationBean(
      new CustomServlet(), "/exampleServlet/*");
    bean.setLoadOnStartup(1);
    return bean;
}
```

这种方法的主要优点是，它使您能够向 Spring 应用程序添加多个 servlet 以及不同种类的 servlet。

我们将使用一个更简单的`HttpServlet`子类实例，它通过四个函数公开四个基本的`HttpRequest` 操作:`doGet()`、`doPost()`、`doPut()`和`doDelete()`，就像在 Jakarta EE 中一样。

记住 HttpServlet 是一个抽象类(所以不能实例化)。不过，我们可以很容易地创建一个自定义扩展:

```java
public class CustomServlet extends HttpServlet{
    ...
}
```

## 4。用属性注册 Servlets】

另一种不常见的配置和注册 servlets 的方法是使用自定义属性文件，通过`PropertyLoader, PropertySource,` 或 `PropertySources` 实例对象`.`加载到应用程序中

这提供了一种中间类型的配置和定制`application.properties`的能力，这为非嵌入式 servlets 提供了很少的直接配置。

### 4.1。系统属性方法

我们可以向我们的 `application.properties`文件或另一个属性文件添加一些自定义设置。让我们添加一些设置来配置我们的`DispatcherServlet`:

```java
servlet.name=dispatcherExample
servlet.mapping=/dispatcherExampleURL
```

让我们将自定义属性加载到应用程序中:

```java
System.setProperty("custom.config.location", "classpath:custom.properties");
```

现在，我们可以通过以下方式访问这些属性:

```java
System.getProperty("custom.config.location");
```

### 4.2。自定义属性方法

让我们从一个`custom.properties`文件开始:

```java
servlet.name=dispatcherExample
servlet.mapping=/dispatcherExampleURL
```

然后，我们可以使用一般的属性加载器:

```java
public Properties getProperties(String file) throws IOException {
  Properties prop = new Properties();
  InputStream input = null;
  input = getClass().getResourceAsStream(file);
  prop.load(input);
  if (input != null) {
      input.close();
  }
  return prop;
}
```

现在我们可以将这些自定义属性作为常量添加到我们的`WebApplicationInitializer`实现中:

```java
private static final PropertyLoader pl = new PropertyLoader(); 
private static final Properties springProps
  = pl.getProperties("custom_spring.properties"); 

public static final String SERVLET_NAME
  = springProps.getProperty("servlet.name"); 
public static final String SERVLET_MAPPING
  = springProps.getProperty("servlet.mapping");
```

例如，我们可以使用它们来配置我们的 dispatcher servlet:

```java
ServletRegistration.Dynamic servlet = container.addServlet(
  SERVLET_NAME, new DispatcherServlet(ctx));
servlet.setLoadOnStartup(1);
servlet.addMapping(SERVLET_MAPPING);
```

这种方法的优点是没有`.xml`维护，但是易于修改配置设置，不需要重新部署代码库。

### 4.3。`PropertySource`接近

完成以上任务的一个更快的方法是利用 Spring 的`PropertySource`,它允许访问和加载配置文件。

`PropertyResolver` 是由`ConfigurableEnvironment,` 实现的接口，它使应用程序属性在 servlet 启动和初始化时可用:

```java
@Configuration 
@PropertySource("classpath:/com/yourapp/custom.properties") 
public class ExampleCustomConfig { 
    @Autowired 
    ConfigurableEnvironment env; 

    public String getProperty(String key) { 
        return env.getProperty(key); 
    } 
}
```

上面，我们将一个依赖项自动连接到类中，并指定自定义属性文件的位置。然后，我们可以通过调用函数 `getProperty()`传入字符串值来获取我们的显著属性。

### 4.4。PropertySource 编程方法

我们可以将上面的方法(包括获取属性值)与下面的方法(允许我们以编程方式指定这些值)结合起来:

```java
ConfigurableEnvironment env = new StandardEnvironment(); 
MutablePropertySources props = env.getPropertySources(); 
Map map = new HashMap(); map.put("key", "value"); 
props.addFirst(new MapPropertySource("Map", map));
```

我们已经创建了一个映射，将一个键链接到一个值，然后将该映射添加到`PropertySources`中，根据需要启用调用。

## 5。注册嵌入式 servlet

最后，我们还将看一下嵌入式 servlets 在 Spring Boot 中的基本配置和注册。

嵌入式 servlet 提供完整的 web 容器(Tomcat、Jetty 等)。)功能，而不必单独安装或维护 web 容器。

您可以为简单的 live server 部署添加所需的依赖项和配置，只要这些功能能够轻松、简洁、快速地得到支持。

我们将只研究如何实现这个 Tomcat，但是同样的方法也可以用于 Jetty 和替代方法。

让我们在`pom.xml`中指定嵌入式 Tomcat 8 web 容器的依赖关系:

```java
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
     <artifactId>tomcat-embed-core</artifactId>
     <version>8.5.11</version>
</dependency>
```

现在让我们添加成功地将 Tomcat 添加到 Maven 在构建时生成的`.war`中所需的标签:

```java
<build>
    <finalName>embeddedTomcatExample</finalName>
    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>appassembler-maven-plugin</artifactId>
            <version>2.0.0</version>
            <configuration>
                <assembleDirectory>target</assembleDirectory>
                <programs>
                    <program>
                        <mainClass>launch.Main</mainClass>
                        <name>webapp</name>
                    </program>
            </programs>
            </configuration>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>assemble</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

如果您使用的是 Spring Boot，您可以将 Spring 的`spring-boot-starter-tomcat` 依赖项添加到您的 `pom.xml`中:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
```

### 5.1。通过属性注册

Spring Boot 支持通过`application.properties`配置大多数可能的弹簧设置。在将必要的嵌入式 servlet 依赖项添加到您的`pom.xml`之后，您可以使用几个这样的配置选项定制和配置您的嵌入式 servlet:

```java
server.jsp-servlet.class-name=org.apache.jasper.servlet.JspServlet 
server.jsp-servlet.registered=true
server.port=8080
server.servlet-path=/
```

以上是一些可用于配置`DispatcherServlet` 和静态资源共享的应用程序设置。还提供了嵌入式 servlets、SSL 支持和会话的设置。

这里列出的配置参数实在太多了，但是您可以在 [Spring Boot 文档](https://web.archive.org/web/20220623120353/https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html)中看到完整的列表。

### 5.2。通过 YAML 进行配置

类似地，我们可以使用 YAML 配置我们的嵌入式 servlet 容器。这需要使用专门的 YAML 属性加载器——`YamlPropertySourceLoader`,它公开了我们的 YAML，并使其中的键和值在我们的应用程序中可用。

```java
YamlPropertySourceLoader sourceLoader = new YamlPropertySourceLoader();
PropertySource<?> yamlProps = sourceLoader.load("yamlProps", resource, null);
```

### 5.3。通过 TomcatEmbeddedServletContainerFactory 进行编程配置

嵌入式 servlet 容器的编程配置可以通过`EmbeddedServletContainerFactory`的子类化实例来实现。例如，您可以使用`TomcatEmbeddedServletContainerFactory`来配置您的嵌入式 Tomcat servlet。

`TomcatEmbeddedServletContainerFactory`包装了 `org.apache.catalina.startup.Tomcat`对象，提供了额外的配置选项:

```java
@Bean
public ConfigurableServletWebServerFactory servletContainer() {
    TomcatServletWebServerFactory tomcatContainerFactory
      = new TomcatServletWebServerFactory();
    return tomcatContainerFactory;
}
```

然后我们可以配置返回的实例:

```java
tomcatContainerFactory.setPort(9000);
tomcatContainerFactory.setContextPath("/springboottomcatexample");
```

这些特定设置中的每一个都可以使用前面描述的任何方法进行配置。

我们还可以直接访问和操作 `org.apache.catalina.startup.Tomcat`对象:

```java
Tomcat tomcat = new Tomcat();
tomcat.setPort(port);
tomcat.setContextPath("/springboottomcatexample");
tomcat.start();
```

## 6。结论

在本文中，我们回顾了几种在 Jakarta EE 和 Spring Boot 应用程序中注册 Servlet 的方法。

本教程中使用的源代码可以在 [Github 项目](https://web.archive.org/web/20220623120353/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-mvc-4)中获得。