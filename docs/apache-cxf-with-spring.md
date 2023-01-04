# 使用 Spring 的 Apache CXF 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-cxf-with-spring>

## 1。概述

本教程着重于配置和使用 [Apache CXF](https://web.archive.org/web/20220815041941/https://cxf.apache.org/) 框架和 Spring——Java 或 XML 配置。

这是 Apache CXF 系列的第二篇；第一篇聚焦于 CXF 作为 JAX-WS 标准 API 实现的基础知识。

## 2。Maven 依赖关系

与前面的教程类似，需要包含以下两个依赖项:

```
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-frontend-jaxws</artifactId>
    <version>3.1.6</version>
</dependency>
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-transports-http</artifactId>
    <version>3.1.6</version>
</dependency>
```

关于最新版本的 Apache CXF 工件，请查看 [apache-cxf](https://web.archive.org/web/20220815041941/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.cxf%22) 。

此外，以下依赖项是支持 Spring 所必需的:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>4.3.1.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>4.3.1.RELEASE</version>
</dependency>
```

Spring 工件的最新版本可以在[这里](https://web.archive.org/web/20220815041941/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework%22)找到。

最后，因为我们将使用 Java Servlet 3.0+ API 而不是传统的`web.xml`部署描述符来编程配置应用程序，所以我们将需要下面的工件:

```
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
</dependency>
```

这个是我们可以找到 Servlet API 最新版本的地方。

## 3。服务器端组件

现在让我们来看看为了发布 web 服务端点，服务器端需要出现的组件。

### 3.1。`WebApplicationInitilizer`界面

实现`WebApplicationInitializer`接口是为了以编程方式为应用程序配置`ServletContext`接口。当出现在类路径上时，它的`onStartup`方法被 servlet 容器自动调用，然后`ServletContext`被实例化和初始化。

下面是如何定义一个类来实现`WebApplicationInitializer`接口:

```
public class AppInitializer implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext container) {
        // Method implementation
    }
}
```

使用下面显示的代码片段实现了`onStartup()`方法。

首先，创建并配置一个 Spring 应用程序上下文来注册一个包含配置元数据的类:

```
AnnotationConfigWebApplicationContext context 
  = new AnnotationConfigWebApplicationContext();
context.register(ServiceConfiguration.class);
```

用`@Configuration`注释对`ServiceConfiguration`类进行了注释，以提供 bean 定义。这个类将在下一小节中讨论。

下面的代码片段展示了如何将 Spring 应用程序上下文添加到 servlet 上下文中:

```
container.addListener(new ContextLoaderListener(context));
```

由 Apache CXF 定义的`CXFServlet`类被生成并注册以处理传入的请求:

```
ServletRegistration.Dynamic dispatcher 
  = container.addServlet("dispatcher", new CXFServlet());
```

应用程序上下文加载配置文件中定义的 Spring 元素。在这种情况下，servlet 的名称是`cxf`，因此默认情况下，上下文在名为`cxf-servlet.xml`的文件中查找这些元素。

最后，CXF servlet 被映射到一个相对 URL:

```
dispatcher.addMapping("/services");
```

### 3.2。`web.xml`

或者，如果我们想利用(有点过时的)部署描述符而不是`WebApplicationInitilizer`接口，相应的`web.xml`文件应该包含以下 servlet 定义:

```
<servlet>
    <servlet-name>cxf</servlet-name>
    <servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
    <servlet-mapping>
    <servlet-name>cxf</servlet-name>
    <url-pattern>/services/*</url-pattern>
</servlet-mapping>
```

### 3.3。`ServiceConfiguration`阶级

现在让我们来看看服务配置——首先是一个基本框架，它包含了 web 服务端点的 bean 定义:

```
@Configuration
public class ServiceConfiguration {
    // Bean definitions
}
```

第一个必需的 bean 是`SpringBus`——它为 Apache CXF 提供扩展，以便与 Spring 框架一起工作:

```
@Bean
public SpringBus springBus() {
    return new SpringBus();
}
```

还需要使用`SpringBus` bean 和 web 服务`implementor`创建一个`EnpointImpl` bean。此 bean 用于在给定的 HTTP 地址发布端点:

```
@Bean
public Endpoint endpoint() {
    EndpointImpl endpoint = new EndpointImpl(springBus(), new BaeldungImpl());
    endpoint.publish("http://localhost:8080/services/baeldung");
    return endpoint;
}
```

`BaeldungImpl`类用于实现 web 服务接口。它的定义在下一小节中给出。

或者，我们也可以在 XML 配置文件中声明服务器端点。具体来说，下面的`cxf-servlet.xml`文件与 3.1 小节中定义的`web.xml`部署描述符一起工作，并描述了完全相同的端点:

```
<jaxws:endpoint
  id="baeldung"
  implementor="com.baeldung.cxf.spring.BaeldungImpl"
  address="http://localhost:8080/services/baeldung" />
```

注意，XML 配置文件是以部署描述符中定义的 servlet 名称命名的，即`cxf`。

### 3.4。类型定义

接下来——这是前面小节中已经提到的`implementor`的定义:

```
@WebService(endpointInterface = "com.baeldung.cxf.spring.Baeldung")
public class BaeldungImpl implements Baeldung {
    private int counter;

    public String hello(String name) {
        return "Hello " + name + "!";
    }

    public String register(Student student) {
        counter++;
        return student.getName() + " is registered student number " + counter;
    }
}
```

这个类为 Apache CXF 将包含在发布的 WSDL 元数据中的`Baeldung`端点接口提供了一个实现:

```
@WebService
public interface Baeldung {
    String hello(String name);
    String register(Student student);
}
```

端点接口和`implementor`都使用了`Student`类，定义如下:

```
public class Student {
    private String name;

    // constructors, getters and setters
}
```

## 4。客户端 bean

为了利用 Spring 框架，我们在一个`@Configuration`注释类中声明了一个 bean:

```
@Configuration
public class ClientConfiguration {
    // Bean definitions
}
```

定义了一个名为`client`的 bean:

```
@Bean(name = "client")
public Object generateProxy() {
    return proxyFactoryBean().create();
}
```

`client` bean 代表了`Baeldung` web 服务的代理。它是通过调用一个`JaxWsProxyFactoryBean` bean 上的`create`方法创建的，这个 bean 是一个用于创建 JAX-WS 代理的工厂。

通过以下方法创建和配置`JaxWsProxyFactoryBean`对象:

```
@Bean
public JaxWsProxyFactoryBean proxyFactoryBean() {
    JaxWsProxyFactoryBean proxyFactory = new JaxWsProxyFactoryBean();
    proxyFactory.setServiceClass(Baeldung.class);
    proxyFactory.setAddress("http://localhost:8080/services/baeldung");
    return proxyFactory;
}
```

工厂的`serviceClass`属性表示 web 服务接口，而`address`属性表示代理进行远程调用的 URL 地址。

对于客户端的 Spring beans，也可以恢复到 XML 配置文件。以下元素声明了与我们刚刚在上面以编程方式配置的 beanss 相同的 bean:

```
<bean id="client" factory-bean="clientFactory" factory-method="create" />
<bean id="clientFactory" class="org.apache.cxf.jaxws.JaxWsProxyFactoryBean">
    <property name="serviceClass" value="com.baeldung.cxf.spring.Baeldung" />
    <property name="address" value="http://localhost:8080/services/baeldung" />
</bean>
```

## 5。测试用例

本节描述了用于说明 Apache CXF 对 Spring 支持的测试用例。测试用例在名为`StudentTest`的类中定义。

首先，我们需要从前面提到的 *ServiceConfiguration* 配置类中加载一个 Spring 应用程序上下文，并将其缓存在`context`字段中:

```
private ApplicationContext context 
  = new AnnotationConfigApplicationContext(ClientConfiguration.class);
```

接下来，从应用程序上下文中声明并加载服务端点接口的代理:

```
private Baeldung baeldungProxy = (Baeldung) context.getBean("client");
```

这个`Baeldung`代理将在下面描述的测试用例中使用。

在第一个测试案例中，我们证明了当在代理上本地调用`hello`方法时，响应与端点`implementor`从远程 web 服务返回的完全相同:

```
@Test
public void whenUsingHelloMethod_thenCorrect() {
    String response = baeldungProxy.hello("John Doe");
    assertEquals("Hello John Doe!", response);
}
```

在第二个测试案例中，学生通过本地调用代理上的`register`方法注册 Baeldung 课程，该方法又调用 web 服务。然后，远程服务将计算学生人数，并将其返回给调用者。以下代码片段证实了我们的预期:

```
@Test
public void whenUsingRegisterMethod_thenCorrect() {
    Student student1 = new Student("Adam");
    Student student2 = new Student("Eve");
    String student1Response = baeldungProxy.register(student1);
    String student2Response = baeldungProxy.register(student2);

    assertEquals("Adam is registered student number 1", student1Response);
    assertEquals("Eve is registered student number 2", student2Response);
}
```

## 6。集成测试

为了在服务器上部署为 web 应用程序，本教程中的代码片段需要首先打包到一个 WAR 文件中。这可以通过在 POM 文件中声明`packaging`属性来实现:

```
<packaging>war</packaging>
```

打包工作由 Maven WAR 插件实现:

```
<plugin>
    <artifactId>maven-war-plugin</artifactId>
    <version>2.6</version>
    <configuration>
        <failOnMissingWebXml>false</failOnMissingWebXml>
    </configuration>
</plugin>
```

这个插件将编译后的源代码打包成一个 WAR 文件。由于我们使用 Java 代码配置 servlet 上下文，传统的`web.xml`部署描述符不需要存在。因此，必须将`failOnMissingWebXml`属性设置为`false`，以避免插件执行失败。

我们可以通过[这个链接](https://web.archive.org/web/20220815041941/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.maven.plugins%22%20AND%20a%3A%22maven-war-plugin%22)获得最新版本的 Maven WAR 插件。

为了说明 web 服务的操作，我们创建一个集成测试。该测试首先生成一个 WAR 文件并启动一个嵌入式服务器，然后让客户端调用 web 服务，验证后续响应，最后停止服务器。

Maven POM 文件中需要包含以下插件。更多详情，请查看[本集成测试教程](/web/20220815041941/https://www.baeldung.com/apache-cxf-with-spring)。

以下是 Maven Surefire 插件:

```
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>
    <configuration>
        <excludes>
            <exclude>StudentTest.java</exclude>
        </excludes>
    </configuration>
</plugin>
```

这个插件的最新版本可以在[这里](https://web.archive.org/web/20220815041941/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.maven.plugins%22%20AND%20a%3A%22maven-surefire-plugin%22)找到。

声明带有`integration`的`id`的`profile`段，以便于集成测试；

```
<profiles>
   <profile>
      <id>integration</id>
      <build>
         <plugins>
            ...
         </plugins>
      </build>
   </profile>
</profiles>
```

Maven Cargo 插件包含在`integration`档案中:

```
<plugin>
    <groupId>org.codehaus.cargo</groupId>
    <artifactId>cargo-maven2-plugin</artifactId>
    <version>1.5.0</version>
    <configuration>
        <container>
            <containerId>jetty9x</containerId>
            <type>embedded</type>
        </container>
        <configuration>
            <properties>
                <cargo.hostname>localhost</cargo.hostname>
                <cargo.servlet.port>8080</cargo.servlet.port>
            </properties>
        </configuration>
    </configuration>
    <executions>
        <execution>
            <id>start-server</id>
            <phase>pre-integration-test</phase>
            <goals>
                <goal>start</goal>
            </goals>
        </execution>
        <execution>
            <id>stop-server</id>
            <phase>post-integration-test</phase>
            <goals>
                <goal>stop</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

请注意，`cargo.hostname`和`cargo.servlet.port`配置属性只是为了清楚起见而包含在内。由于这些配置属性的值与默认值相同，因此可以省略它们，而不会对应用程序产生任何影响。这个插件启动服务器，等待连接，最后停止服务器来释放系统资源。

[这个链接](https://web.archive.org/web/20220815041941/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.maven.plugins%22%20AND%20a%3A%22maven-war-plugin%22)允许我们查看最新版本的 Maven Cargo 插件。

在`integration`概要文件中再次声明 Maven Surefire 插件，以覆盖其在主`build`部分中的配置，并执行前一部分中描述的测试用例:

```
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>
    <executions>
        <execution>
            <phase>integration-test</phase>
            <goals>
                <goal>test</goal>
            </goals>
            <configuration>
                <excludes>
                    <exclude>none</exclude>
                </excludes>
            </configuration>
        </execution>
    </executions>
</plugin>
```

现在，整个过程可以通过命令运行:`mvn -Pintegration clean install`。

## 7。结论

本教程演示了 Apache CXF 对 Spring 的支持。特别是，已经展示了如何使用 Spring 配置文件发布 web 服务，以及客户端如何通过 Apache CXF 代理工厂创建的代理与该服务进行交互，该代理是在另一个配置文件中声明的。

所有这些例子和代码片段的实现都可以在[链接的 GitHub 项目](https://web.archive.org/web/20220815041941/https://github.com/eugenp/tutorials/tree/master/apache-cxf-modules/cxf-spring)中找到。