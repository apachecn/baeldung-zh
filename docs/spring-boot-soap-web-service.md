# 用 Spring 创建 SOAP Web 服务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-soap-web-service>

## 1。概述

在本教程中，我们将学习如何用 Spring Boot 启动 web 服务创建一个基于 SOAP 的 Web 服务。

## 2。SOAP 网络服务

简而言之，web 服务是一种机器对机器、独立于平台的服务，允许通过网络进行通信。

SOAP 是一种消息协议。消息(请求和响应)是 HTTP 上的 **XML 文档。**XML 契约由 WSDL** (Web 服务描述语言)定义。它提供了一组规则来定义服务的消息、绑定、操作和位置。**

SOAP 中使用的 XML 可能会变得非常复杂。出于这个原因，最好使用带有框架的 SOAP，如 [JAX-WS](/web/20221126222213/https://www.baeldung.com/jax-ws) 或 Spring，我们将在本教程中看到。

## 3。契约优先开发风格

创建 web 服务时有两种可能的方法:契约-最后一个和 [契约-第一个](https://web.archive.org/web/20221126222213/https://docs.spring.io/spring-ws/sites/1.5/reference/html/why-contract-first.html) 。当我们使用契约式方法时，我们从 Java 代码开始，从类中生成 web 服务契约(WSDL)。当使用契约优先时，我们从 WSDL 契约开始，从中我们生成 Java 类。

Spring-WS 只支持契约优先的开发风格。

## 4。建立 Spring Boot 项目

我们将创建一个 [Spring Boot](/web/20221126222213/https://www.baeldung.com/spring-boot) 项目，在这里我们将定义我们的 SOAP WS 服务器。

### 4.1.Maven 依赖性

让我们首先将 [`spring-boot-starter-parent`](https://web.archive.org/web/20221126222213/https://search.maven.org/search?q=g:org.springframework.boot%20a:spring-boot-starter-parent) 添加到我们的项目中:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.2</version>
</parent> 
```

接下来，让我们添加 [`spring-boot-starter-web-services`](https://web.archive.org/web/20221126222213/https://search.maven.org/search?q=g:org.springframework.boot%20a:spring-boot-starter-web-services) 和`[wsdl4j](https://web.archive.org/web/20221126222213/https://search.maven.org/search?q=g:wsdl4j%20%20a:wsdl4j)`的依赖关系:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web-services</artifactId>
</dependency>
<dependency>
    <groupId>wsdl4j</groupId>
    <artifactId>wsdl4j</artifactId>
</dependency> 
```

### 4.2.XSD 档案

契约优先的方法要求我们首先为服务创建域(方法和参数)。我们将使用一个 XML 模式文件(XSD ), Spring-WS 会将其自动导出为 WSDL:

```java
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:tns="http://www.baeldung.com/springsoap/gen"
           targetNamespace="http://www.baeldung.com/springsoap/gen" elementFormDefault="qualified">

    <xs:element name="getCountryRequest">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="name" type="xs:string"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>

    <xs:element name="getCountryResponse">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="country" type="tns:country"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>

    <xs:complexType name="country">
        <xs:sequence>
            <xs:element name="name" type="xs:string"/>
            <xs:element name="population" type="xs:int"/>
            <xs:element name="capital" type="xs:string"/>
            <xs:element name="currency" type="tns:currency"/>
        </xs:sequence>
    </xs:complexType>

    <xs:simpleType name="currency">
        <xs:restriction base="xs:string">
            <xs:enumeration value="GBP"/>
            <xs:enumeration value="EUR"/>
            <xs:enumeration value="PLN"/>
        </xs:restriction>
    </xs:simpleType>
</xs:schema> 
```

**在这个文件中，我们可以看到`getCountryRequest` web 服务请求**的格式。我们将其定义为接受一个类型为`string`的参数。

接下来，我们将定义响应的格式，它包含一个类型为`country`的对象。

最后，我们可以看到在`country`对象中使用的`currency`对象。

### 4.3.生成域 Java 类

现在我们将从上一节定义的 XSD 文件中生成 Java 类。 [`jaxb2-maven-plugin`](https://web.archive.org/web/20221126222213/https://search.maven.org/search?q=g:org.codehaus.mojo%20a:jaxb2-maven-plugin) 会在构建时自动完成这项工作。该插件使用 XJC 工具作为代码生成引擎。XJC 将 XSD 模式文件编译成完全带注释的 Java 类。

让我们在 pom.xml 中添加和配置插件:

```java
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>jaxb2-maven-plugin</artifactId>
    <version>1.6</version>
    <executions>
        <execution>
            <id>xjc</id>
            <goals>
                <goal>xjc</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <schemaDirectory>${project.basedir}/src/main/resources/</schemaDirectory>
        <outputDirectory>${project.basedir}/src/main/java</outputDirectory>
        <clearOutputDir>false</clearOutputDir>
    </configuration>
</plugin> 
```

这里我们注意到两个重要的配置:

*   `<schemaDirectory>${project.basedir}/src/main/resources</schemaDirectory>`–XSD 文件的位置
*   `<outputDirectory>${project.basedir}/src/main/java</outputDirectory>`–我们希望生成 Java 代码的地方

为了生成 Java 类，我们可以使用 Java 安装中的 XJC 工具。不过，在我们的 Maven 项目中，这甚至更简单，因为在通常的 Maven 构建过程中，**类会自动生成**:

```java
mvn compile
```

### 4.4.添加 SOAP Web 服务端点

SOAP web 服务端点类将处理服务的所有传入请求。它将启动处理，并发回响应。

在定义它之前，我们将创建一个`Country`存储库，以便向 web 服务提供数据:

```java
@Component
public class CountryRepository {

    private static final Map<String, Country> countries = new HashMap<>();

    @PostConstruct
    public void initData() {
        // initialize countries map
    }

    public Country findCountry(String name) {
        return countries.get(name);
    }
} 
```

接下来，我们将配置端点:

```java
@Endpoint
public class CountryEndpoint {

    private static final String NAMESPACE_URI = "http://www.baeldung.com/springsoap/gen";

    private CountryRepository countryRepository;

    @Autowired
    public CountryEndpoint(CountryRepository countryRepository) {
        this.countryRepository = countryRepository;
    }

    @PayloadRoot(namespace = NAMESPACE_URI, localPart = "getCountryRequest")
    @ResponsePayload
    public GetCountryResponse getCountry(@RequestPayload GetCountryRequest request) {
        GetCountryResponse response = new GetCountryResponse();
        response.setCountry(countryRepository.findCountry(request.getName()));

        return response;
    }
} 
```

以下是一些需要注意的细节:

*   `@Endpoint`–向 Spring WS 注册类作为 Web 服务端点
*   `@PayloadRoot`–**根据`namespace` 和`localPart` 属性**定义处理程序方法
*   `@ResponsePayload`–表示此方法返回一个要映射到响应有效负载的值
*   `@RequestPayload`–表示该方法接受从传入请求映射的参数

### 4.5.SOAP Web 服务配置 Beans

现在让我们创建一个类来配置 Spring message dispatcher servlet 以接收请求:

```java
@EnableWs
@Configuration
public class WebServiceConfig extends WsConfigurerAdapter {
    // bean definitions
}
```

在此 Spring Boot 应用程序中启用 SOAP Web 服务功能。`WebServiceConfig`类扩展了`WsConfigurerAdapter`基类，后者配置注释驱动的 Spring-WS 编程模型。

让我们创建一个用于处理 SOAP 请求的`MessageDispatcherServlet,`:

```java
@Bean
public ServletRegistrationBean messageDispatcherServlet(ApplicationContext applicationContext) {
    MessageDispatcherServlet servlet = new MessageDispatcherServlet();
    servlet.setApplicationContext(applicationContext);
    servlet.setTransformWsdlLocations(true);
    return new ServletRegistrationBean(servlet, "/ws/*");
} 
```

我们将设置`servlet,`的注入的`ApplicationContext`对象，以便 Spring-WS 可以找到其他 Spring beans。

我们还将启用 WSDL 位置 servlet 转换。这将转换 WSDL 中`soap:address`的位置属性，以便它反映传入请求的 URL。

最后，我们将创建一个`DefaultWsdl11Definition`对象。这公开了一个使用 XsdSchema 的标准 WSDL 1.1。WSDL 名称将与 bean 名称相同:

```java
@Bean(name = "countries")
public DefaultWsdl11Definition defaultWsdl11Definition(XsdSchema countriesSchema) {
    DefaultWsdl11Definition wsdl11Definition = new DefaultWsdl11Definition();
    wsdl11Definition.setPortTypeName("CountriesPort");
    wsdl11Definition.setLocationUri("/ws");
    wsdl11Definition.setTargetNamespace("http://www.baeldung.com/springsoap/gen");
    wsdl11Definition.setSchema(countriesSchema);
    return wsdl11Definition;
}

@Bean
public XsdSchema countriesSchema() {
    return new SimpleXsdSchema(new ClassPathResource("countries.xsd"));
} 
```

## 5.测试 SOAP 项目

一旦项目配置完成，我们就可以测试它了。

### 5.1.构建并运行项目

可以创建一个 WAR 文件并将其部署到外部应用服务器。相反，我们将使用 Spring Boot，这是启动和运行应用程序的更快更简单的方法。

首先，我们将添加以下类来使应用程序可执行:

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
} 
```

注意，我们没有使用任何 XML 文件(如 web.xml)来创建这个应用程序。都是纯 Java。

现在我们已经准备好构建和运行应用程序了:

```java
mvn spring-boot:run
```

为了检查应用程序是否正常运行，我们可以通过 URL 打开 WSDL:[http://localhost:8080/ws/countries . wsdl](https://web.archive.org/web/20221126222213/http://localhost:8080/ws/countries.wsdl)

### 5.2.测试 SOAP 请求

为了测试一个请求，我们将创建以下文件并将其命名为 request.xml:

```java
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:gs="http://www.baeldung.com/springsoap/gen">
    <soapenv:Header/>
    <soapenv:Body>
        <gs:getCountryRequest>
            <gs:name>Spain</gs:name>
        </gs:getCountryRequest>
    </soapenv:Body>
</soapenv:Envelope> 
```

要将请求发送到我们的测试服务器，我们可以使用外部工具，比如 SoapUI 或 Google Chrome extension Wizdler。另一种方法是在我们的 shell 中运行以下命令:

```java
curl --header "content-type: text/xml" -d @request.xml http://localhost:8080/ws
```

如果没有缩进或换行符，得到的响应可能不容易阅读。

要查看它的格式，我们可以将它复制粘贴到我们的 IDE 或其他工具中。如果我们已经安装了 xmllib2，我们可以将 curl 命令的输出通过管道传输到`xmllint`:

```java
curl [command-line-options] | xmllint --format -
```

回复应包含关于西班牙的信息:

```java
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/">
<SOAP-ENV:Header/>
<SOAP-ENV:Body>
    <ns2:getCountryResponse xmlns:ns2="http://www.baeldung.com/springsoap/gen">
        <ns2:country>
            <ns2:name>Spain</ns2:name>
            <ns2:population>46704314</ns2:population>
            <ns2:capital>Madrid</ns2:capital>
            <ns2:currency>EUR</ns2:currency>
        </ns2:country>
    </ns2:getCountryResponse>
</SOAP-ENV:Body>
</SOAP-ENV:Envelope> 
```

## 6.结论

在本文中，我们学习了如何使用 Spring Boot 创建 SOAP web 服务。我们还演示了如何从 XSD 文件生成 Java 代码。最后，我们配置了处理 SOAP 请求所需的 Spring beans。

完整的源代码可以在 [GitHub](https://web.archive.org/web/20221126222213/https://github.com/eugenp/tutorials/tree/master/spring-soap) 上找到。