# 用 Java 调用 SOAP Web 服务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-soap-web-service>

## 1.概观

在本教程中，**我们将学习如何在 Java 8 和 11 中用 [JAX-WS RI](https://web.archive.org/web/20220628094602/https://javaee.github.io/metro-jax-ws/) 构建一个 SOAP 客户端。**

首先，我们将使用`wsimport`实用程序生成客户机代码，然后使用 JUnit 测试它。

对于那些刚开始的人，我们的[对 JAX-WS](/web/20220628094602/https://www.baeldung.com/jax-ws) 的介绍提供了关于这个主题的很好的背景。

## 2.网络服务

在我们开始构建客户端之前，我们需要一个服务器。在这种情况下，我们需要一个公开 JAX-WS web 服务的服务器。

出于本教程的目的，我们将使用一个 web 服务来获取一个国家的数据，给定它的名称。

### 2.1.实施摘要

因为我们的重点是构建客户端，所以我们不会讨论服务的实现细节。

假设一个接口`CountryService`用于向外部世界公开 web 服务。为了简单起见，**我们将使用我们的类`CountryServicePublisher`中的 [`javax.xml.ws.Endpoint` API](https://web.archive.org/web/20220628094602/https://docs.oracle.com/javase/7/docs/api/javax/xml/ws/Endpoint.html) 来构建和部署 web 服务。**

我们将把`CountryServicePublisher`作为一个 Java 应用程序来运行，以发布一个接受传入请求的端点。换句话说，这将是我们的服务器。

启动服务器后，点击 URL [`http://localhost:8888/ws/country?wsdl`](https://web.archive.org/web/20220628094602/http://localhost:8888/ws/country?wsdl) 给我们 web 服务描述文件。**WSDL 充当了理解服务内容并为客户生成实现代码的向导。**

### 2.2.Web 服务描述语言

让我们来看看我们的 web 服务的 WSDL，`country`:

```java
<?xml version="1.0" encoding="UTF-8"?>
<definitions <!-- namespace declarations -->
    targetNamespace="http://server.ws.soap.baeldung.com/" name="CountryServiceImplService">
    <types>
        <xsd:schema>
            <xsd:import namespace="http://server.ws.soap.baeldung.com/" 
              schemaLocation="http://localhost:8888/ws/country?xsd=1"></xsd:import>
        </xsd:schema>
    </types>
    <message name="findByName">
        <part name="arg0" type="xsd:string"></part>
    </message>
    <message name="findByNameResponse">
        <part name="return" type="tns:country"></part>
    </message>
    <portType name="CountryService">
        <operation name="findByName">
            <input wsam:Action="http://server.ws.soap.baeldung.com/CountryService/findByNameRequest" 
              message="tns:findByName"></input>
            <output wsam:Action="http://server.ws.soap.baeldung.com/CountryService/findByNameResponse" 
              message="tns:findByNameResponse"></output>
        </operation>
    </portType>
    <binding name="CountryServiceImplPortBinding" type="tns:CountryService">
        <soap:binding transport="http://schemas.xmlsoap.org/soap/http" style="rpc"></soap:binding>
        <operation name="findByName">
            <soap:operation soapAction=""></soap:operation>
            <input>
                <soap:body use="literal" namespace="http://server.ws.soap.baeldung.com/"></soap:body>
            </input>
            <output>
                <soap:body use="literal" namespace="http://server.ws.soap.baeldung.com/"></soap:body>
            </output>
        </operation>
    </binding>
    <service name="CountryServiceImplService">
        <port name="CountryServiceImplPort" binding="tns:CountryServiceImplPortBinding">
            <soap:address location="http://localhost:8888/ws/country"></soap:address>
        </port>
    </service>
</definitions>
```

简而言之，这是它提供的有用信息:

*   我们可以用一个`string`参数调用方法`findByName`。
*   作为响应，服务将返回给我们一个自定义类型`country`。
*   类型在 [`http://localhost:8888/ws/country?xsd=1`](https://web.archive.org/web/20220628094602/http://localhost:8888/ws/country?xsd=1) 位置生成的`xsd`模式中定义:

```java
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema <!-- namespace declarations -->
    targetNamespace="http://server.ws.soap.baeldung.com/">
    <xs:complexType name="country">
        <xs:sequence>
            <xs:element name="capital" type="xs:string" minOccurs="0"></xs:element>
            <xs:element name="currency" type="tns:currency" minOccurs="0"></xs:element>
            <xs:element name="name" type="xs:string" minOccurs="0"></xs:element>
            <xs:element name="population" type="xs:int"></xs:element>
        </xs:sequence>
    </xs:complexType>
    <xs:simpleType name="currency">
        <xs:restriction base="xs:string">
            <xs:enumeration value="EUR"></xs:enumeration>
            <xs:enumeration value="INR"></xs:enumeration>
            <xs:enumeration value="USD"></xs:enumeration>
        </xs:restriction>
    </xs:simpleType>
</xs:schema>
```

这就是我们实现客户端所需的全部内容。

让我们在下一节看看如何实现。

## 3.使用`wsimport`生成客户端代码

### 3.1.对于 JDK 8

首先，让我们看看如何使用 JDK 8 生成客户端代码。

首先，让我们给我们的`pom.xml`添加一个插件，通过 Maven 使用这个工具:

```java
<plugin> 
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>jaxws-maven-plugin</artifactId>
    <version>2.6</version>
    <executions> 
        <execution> 
            <id>wsimport-from-jdk</id>
            <goals>
                <goal>wsimport</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <wsdlUrls>
            <wsdlUrl>http://localhost:8888/ws/country?wsdl</wsdlUrl> 
        </wsdlUrls>
        <keep>true</keep> 
        <packageName>com.baeldung.soap.ws.client.generated</packageName> 
        <sourceDestDir>src/main/java</sourceDestDir>
    </configuration>
</plugin>
```

其次，让我们执行这个插件:

```java
mvn clean jaxws:wsimport
```

仅此而已！上面的命令将在插件配置中提供的`sourceDestDir`内的指定包`com.baeldung.soap.ws.client.generated`中生成代码。

**另一种实现相同目的的方法是使用 [`wsimport`](https://web.archive.org/web/20220628094602/https://docs.oracle.com/javase/7/docs/technotes/tools/share/wsimport.html) 实用程序。**它来自标准的 JDK 8 发行版，可以在`JAVA_HOME/bin`目录下找到。

要使用`wsimport`生成客户端代码，我们可以导航到项目的根目录并运行以下命令:

```java
JAVA_HOME/bin/wsimport -s src/main/java/ -keep -p com.baeldung.soap.ws.client.generated "http://localhost:8888/ws/country?wsdl"
```

重要的是要记住，为了成功地执行插件或命令，服务端点应该是可用的。

### 3.2.对于 JDK 11 号

从 JDK 11 开始， **`wsimport`作为 JDK 的一部分被删除，不再随标准发行版一起提供。**

然而，它对 Eclipse 基金会是开源的。

为了使用`wsimport`为 Java 11 及以上版本生成客户端代码，我们需要在 [`jaxws-maven-plugin`](https://web.archive.org/web/20220628094602/https://search.maven.org/search?q=g:com.sun.xml.ws%20AND%20a:jaxws-maven-plugin) 之外添加[`jakarta.xml.ws-api`](https://web.archive.org/web/20220628094602/https://search.maven.org/search?q=g:jakarta.xml.ws%20AND%20a:jakarta.xml.ws-api)[`jaxws-rt`](https://web.archive.org/web/20220628094602/https://search.maven.org/search?q=g:com.sun.xml.ws%20AND%20a:jaxws-rt)和 [`jaxws-ri`](https://web.archive.org/web/20220628094602/https://search.maven.org/search?q=g:com.sun.xml.ws%20AND%20a:jaxws-ri) 依赖项:

```java
<dependencies>
    <dependency>
        <groupId>jakarta.xml.ws</groupId
        <artifactId>jakarta.xml.ws-api</artifactId
        <version>3.0.0</version>
    </dependency>
    <dependency>
        <groupId>com.sun.xml.ws</groupId>
        <artifactId>jaxws-rt</artifactId>
        <version>3.0.0</version
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>com.sun.xml.ws</groupId>
        <artifactId>jaxws-ri</artifactId>
        <version>2.3.1</version
        <type>pom</type>
    </dependency>
</dependencies>
<build>
    <plugins>        
        <plugin>
            <groupId>com.sun.xml.ws</groupId>
            <artifactId>jaxws-maven-plugin</artifactId>
            <version>2.3.2</version>
            <configuration>
                <wsdlUrls>
                    <wsdlUrl>http://localhost:8888/ws/country?wsdl</wsdlUrl>
                </wsdlUrls>
                <keep>true</keep>
                <packageName>com.baeldung.soap.ws.client.generated</packageName>
                <sourceDestDir>src/main/java</sourceDestDir>
            </configuration>
        </plugin>
    </plugins>
</build> 
```

现在，为了在包`com.baeldung.soap.ws.client.generated`中生成客户端代码，我们将需要与之前相同的 Maven 命令:

```java
mvn clean jaxws:wsimport
```

接下来，让我们看看两个 Java 版本中相同的生成的工件。

### 3.3.生成的 POJOs

基于我们之前看到的`xsd`，该工具将生成一个名为`Country.java`的文件:

```java
@XmlAccessorType(XmlAccessType.FIELD)
@XmlType(name = "country", propOrder = { "capital", "currency", "name", "population" })
public class Country {
    protected String capital;
    @XmlSchemaType(name = "string")
    protected Currency currency;
    protected String name;
    protected int population;
    // standard getters and setters
}
```

正如我们看到的，生成的类用 [JAXB 注释](/web/20220628094602/https://www.baeldung.com/jaxb)修饰，用于将对象与 XML 进行编组和解组。

此外，它还会生成一个`Currency`枚举:

```java
@XmlType(name = "currency")
@XmlEnum
public enum Currency {
    EUR, INR, USD;
    public String value() {
        return name();
    }
    public static Currency fromValue(String v) {
        return valueOf(v);
    }
}
```

### 3.4.`CountryService`

第二个生成的工件是一个接口，它充当实际 web 服务的代理。

接口`CountryService`声明了与我们的服务器`findByName`相同的方法:

```java
@WebService(name = "CountryService", targetNamespace = "http://server.ws.soap.baeldung.com/")
@SOAPBinding(style = SOAPBinding.Style.RPC)
@XmlSeeAlso({ ObjectFactory.class })
public interface CountryService {
    @WebMethod
    @WebResult(partName = "return")
    @Action(input = "http://server.ws.soap.baeldung.com/CountryService/findByNameRequest", 
      output = "http://server.ws.soap.baeldung.com/CountryService/findByNameResponse")
    public Country findByName(@WebParam(name = "arg0", partName = "arg0") String arg0);
}
```

值得注意的是，该接口被标记为`javax.jws.WebService`，带有一个 [`SOAPBinding.Style`](https://web.archive.org/web/20220628094602/https://docs.oracle.com/javase/7/docs/api/javax/jws/soap/SOAPBinding.Style.html) 作为由服务的 WSDL 定义的 RPC。

对方法`findByName`进行了注释，声明它是一个`javax.jws.WebMethod`，带有预期的输入和输出参数类型。

### 3.5.`CountryServiceImplService`

我们下一个生成的类`CountryServiceImplService`，扩展了`javax.xml.ws.Service`。

它的注释`[WebServiceClient](https://web.archive.org/web/20220628094602/https://docs.oracle.com/javase/7/docs/api/javax/xml/ws/WebServiceClient.html) `表示它是服务的客户端视图:

```java
@WebServiceClient(name = "CountryServiceImplService", 
  targetNamespace = "http://server.ws.soap.baeldung.com/", 
  wsdlLocation = "http://localhost:8888/ws/country?wsdl")
public class CountryServiceImplService extends Service {

    private final static URL COUNTRYSERVICEIMPLSERVICE_WSDL_LOCATION;
    private final static WebServiceException COUNTRYSERVICEIMPLSERVICE_EXCEPTION;
    private final static QName COUNTRYSERVICEIMPLSERVICE_QNAME = 
      new QName("http://server.ws.soap.baeldung.com/", "CountryServiceImplService");

    static {
        URL url = null;
        WebServiceException e = null;
        try {
            url = new URL("http://localhost:8888/ws/country?wsdl");
        } catch (MalformedURLException ex) {
            e = new WebServiceException(ex);
        }
        COUNTRYSERVICEIMPLSERVICE_WSDL_LOCATION = url;
        COUNTRYSERVICEIMPLSERVICE_EXCEPTION = e;
    }

    public CountryServiceImplService() {
        super(__getWsdlLocation(), COUNTRYSERVICEIMPLSERVICE_QNAME);
    }

    // other constructors 

    @WebEndpoint(name = "CountryServiceImplPort")
    public CountryService getCountryServiceImplPort() {
        return super.getPort(new QName("http://server.ws.soap.baeldung.com/", "CountryServiceImplPort"), 
          CountryService.class);
    }

    private static URL __getWsdlLocation() {
        if (COUNTRYSERVICEIMPLSERVICE_EXCEPTION != null) {
            throw COUNTRYSERVICEIMPLSERVICE_EXCEPTION;
        }
        return COUNTRYSERVICEIMPLSERVICE_WSDL_LOCATION;
    }

}
```

这里要注意的重要方法是`getCountryServiceImplPort`。给定服务端点的限定名或`QName`，以及动态代理的服务端点接口名，它返回一个代理实例。

为了调用 web 服务，我们需要使用这个代理，我们很快就会看到。

使用代理使我们看起来好像是在本地调用服务，抽象掉了远程调用的复杂性。

## 4.测试客户端

接下来，我们将编写一个 JUnit 测试，使用生成的客户端代码连接到 web 服务。

在这样做之前，我们需要在客户端获得服务的代理实例:

```java
@BeforeClass
public static void setup() {
    CountryServiceImplService service = new CountryServiceImplService();
    CountryService countryService = service.getCountryServiceImplPort();
}
```

对于更高级的场景，比如启用或禁用一个 [`WebServiceFeature`](https://web.archive.org/web/20220628094602/https://docs.oracle.com/javase/7/docs/api/javax/xml/ws/WebServiceFeature.html) ，我们可以为`CountryServiceImplService`使用其他生成的构造函数。

现在我们来看一些测试:

```java
@Test
public void givenCountryService_whenCountryIndia_thenCapitalIsNewDelhi() {
    assertEquals("New Delhi", countryService.findByName("India").getCapital());
}

@Test
public void givenCountryService_whenCountryFrance_thenPopulationCorrect() {
    assertEquals(66710000, countryService.findByName("France").getPopulation());
}

@Test
public void givenCountryService_whenCountryUSA_thenCurrencyUSD() {
    assertEquals(Currency.USD, countryService.findByName("USA").getCurrency());
} 
```

正如我们所看到的，调用远程服务的方法变得和本地调用方法一样简单。代理的`findByName`方法返回了一个与我们提供的`name` 匹配的`Country`实例。然后我们使用 POJO 的各种 getters 来断言期望值。

## 5.结论

在本文中，我们看到了如何使用 JAX-WS RI 和用于 Java 8 和 11 的`wsimport`实用程序来**调用 Java 中的 SOAP web 服务。**

或者，我们可以使用其他 JAX-WS 实现，比如 Apache CXF、Apache Axis2 和 Spring 来做同样的事情。

和往常一样，GitHub 上有 JDK 8 和 T2 JDK 11 的源代码。