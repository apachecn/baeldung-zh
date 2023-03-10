# JAX WS 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jax-ws>

## 1。概述

[Java API for XML Web Services(JAX-WS)](https://web.archive.org/web/20220526040446/http://jax-ws.java.net/)是用于创建和消费 SOAP(简单对象访问协议)Web 服务的标准化 API。

在本文中，我们将创建一个 SOAP web 服务，并使用 JAX-WS 连接到它。

## 2。肥皂

SOAP 是通过网络发送消息的 XML 规范。SOAP 消息独立于任何操作系统，可以使用多种通信协议，包括 HTTP 和 SMTP。

SOAP 大量使用 XML，因此最好与工具/框架一起使用。JAX-WS 是一个简化使用 SOAP 的框架。它是标准 Java 的一部分。

## 3。自上而下与自下而上

构建 SOAP web 服务有两种方式。我们可以采用自上而下或自下而上的方法。

在自顶向下(契约优先)的方法中，创建一个 WSDL 文档，并从 WSDL 中生成必要的 Java 类。在自底向上(契约最后)的方法中，Java 类被编写，WSDL 从 Java 类生成。

根据你的 web 服务的复杂程度，编写一个 WSDL 文件可能相当困难。这使得自底向上的方法成为一种更容易的选择。另一方面，由于您的 WSDL 是从 Java 类中生成的，代码中的任何变化都可能导致 WSDL 的变化。自上而下的方法却不是这样。

在本文中，我们将看看这两种方法。

## 4。Web 服务定义语言(WSDL)

WSDL 是可用服务的契约定义。它是输入/输出消息以及如何调用 web 服务的规范。它是语言中立的，是用 XML 定义的。

让我们看看 WSDL 文件的主要内容。

### 4.1。定义

`definitions`元素是所有 WSDL 文档的根元素。它定义了名称、命名空间等。如你所见，这里非常宽敞:

```java
<definitions  
  xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/" 
  xmlns:tns="http://jaxws.baeldung.com/" 
  xmlns:wsam="http://www.w3.org/2007/05/addressing/metadata"
  xmlns:wsp="http://www.w3.org/ns/ws-policy" 
  xmlns:wsp1_2="http://schemas.xmlsoap.org/ws/2004/09/policy"
  xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd" 
  xmlns:xsd="http://www.w3.org/2001/XMLSchema" 
  targetNamespace="http://jaxws.baeldung.com/" 
  name="EmployeeService">
  ...
</definitions>
```

### 4.2。类型

`types`元素定义了 web 服务使用的数据类型。WSDL 使用 [XSD](https://web.archive.org/web/20220526040446/https://www.w3.org/TR/xmlschema-2/) (XML 模式定义)作为帮助实现互操作性的类型系统:

```java
<definitions ...>
    ...
    <types>
        <xsd:schema>
            <xsd:import namespace="http://jaxws.baeldung.com/" 
              schemaLocation = "http://localhost:8080/employeeservice?xsd=1" />
        </xsd:schema>
    </types>
    ...
</definitions>
```

### 4.3。消息

元素提供了被传输数据的抽象定义。每个`message`元素描述了服务方法的输入或输出以及可能的异常:

```java
<definitions ...>
    ...
    <message name="getEmployee">
        <part name="parameters" element="tns:getEmployee" />
    </message>
    <message name="getEmployeeResponse">
        <part name="parameters" element="tns:getEmployeeResponse" />
    </message>
    <message name="EmployeeNotFound">
        <part name="fault" element="tns:EmployeeNotFound" />
    </message>
    ...
</definitions>
```

### 4.4。操作和端口类型

`portType`元素描述了每个可以执行的`operation`和所有涉及的`message`元素。例如，`getEmployee`操作指定了 web 服务`operation`抛出的请求`input`、`output`和可能的`fault`异常:

```java
<definitions ...>
    ...
    <portType name="EmployeeService">
        <operation name="getEmployee">
            <input 
              wsam:Action="http://jaxws.baeldung.com/EmployeeService/getEmployeeRequest" 
              message="tns:getEmployee" />
            <output 
              wsam:Action="http://jaxws.baeldung.com/EmployeeService/getEmployeeResponse" 
              message="tns:getEmployeeResponse" />
            <fault message="tns:EmployeeNotFound" name="EmployeeNotFound" 
              wsam:Action="http://jaxws.baeldung.com/EmployeeService/getEmployee/Fault/EmployeeNotFound" />
        </operation>
    ....
    </portType>
    ...
</definitions> 
```

### 4.5。绑定

`binding`元素为每个`portType`提供协议和数据格式细节:

```java
<definitions ...>
    ...
    <binding name="EmployeeServiceImplPortBinding" 
      type="tns:EmployeeService">
        <soap:binding transport="http://schemas.xmlsoap.org/soap/http" 
          style="document" />
        <operation name="getEmployee">
            <soap:operation soapAction="" />
            <input>
                <soap:body use="literal" />
            </input>
            <output>
                <soap:body use="literal" />
            </output>
            <fault name="EmployeeNotFound">
                <soap:fault name="EmployeeNotFound" use="literal" />
            </fault>
        </operation>
        ...
    </binding>
    ...
</definitions>
```

### 4.6。服务和端口

`service`元素定义了 web 服务支持的端口。`service`中的`port`元素定义了服务的`name`、`binding`和`address`:

```java
<definitions ...>
    ...
    <service name="EmployeeService">
        <port name="EmployeeServiceImplPort" 
          binding="tns:EmployeeServiceImplPortBinding">
            <soap:address 
              location="http://localhost:8080/employeeservice" />
        </port>
    </service>
    ...
</definitions>
```

## 5。自上而下(合同优先)的方法

让我们从自顶向下的方法开始，创建一个 WSDL 文件`employeeservicetopdown.wsdl.` 为了简单起见，它只有一个方法:

```java
<?xml version="1.0" encoding="UTF-8"?>
<definitions 
  xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/"
  xmlns:tns="http://topdown.server.jaxws.baeldung.com/"
  xmlns:xsd="http://www.w3.org/2001/XMLSchema"

  targetNamespace="http://topdown.server.jaxws.baeldung.com/"
  qname="EmployeeServiceTopDown">
    <types>
        <xsd:schema 
          targetNamespace="http://topdown.server.jaxws.baeldung.com/">
            <xsd:element name="countEmployeesResponse" type="xsd:int"/>
        </xsd:schema>
    </types>

    <message name="countEmployees">
    </message>
    <message name="countEmployeesResponse">
        <part name="parameters" element="tns:countEmployeesResponse"/>
    </message>
    <portType name="EmployeeServiceTopDown">
        <operation name="countEmployees">
            <input message="tns:countEmployees"/>
            <output message="tns:countEmployeesResponse"/>
        </operation>
    </portType>
    <binding name="EmployeeServiceTopDownSOAP" 
      type="tns:EmployeeServiceTopDown">
        <soap:binding transport="http://schemas.xmlsoap.org/soap/http" 
          style="document"/>
        <operation name="countEmployees">
            <soap:operation 
              soapAction="http://topdown.server.jaxws.baeldung.com/
              EmployeeServiceTopDown/countEmployees"/>
            <input>
                <soap:body use="literal"/>
            </input>
            <output>
                <soap:body use="literal"/>
            </output>
        </operation>
    </binding>
    <service name="EmployeeServiceTopDown">
        <port name="EmployeeServiceTopDownSOAP" 
          binding="tns:EmployeeServiceTopDownSOAP">
            <soap:address 
              location="http://localhost:8080/employeeservicetopdown"/>
        </port>
    </service>
</definitions>
```

### 5.1。从 WSDL 生成 **Web 服务源文件**

有几种方法可以从 WSDL 文档生成 web 服务源文件。

一种方法是使用`wsimport`工具，它是 JDK 的一部分(在 [$JAVA_HOME](/web/20220526040446/https://www.baeldung.com/java-home-on-windows-7-8-10-mac-os-x-linux) /bin)，直到 JDK 8。

在命令提示符下:

```java
wsimport -s . -p com.baeldung.jaxws.server.topdown employeeservicetopdown.wsdl
```

使用的命令行选项: **-p** 指定目标包。 **-s** 指定生成的源文件放在哪里。

对于后来的 JDK 版本，我们可以使用 MojoHaus 的`[jaxws-maven-plugin](https://web.archive.org/web/20220526040446/https://www.mojohaus.org/jaxws-maven-plugin/index.html)`，如这里的所述。

或者，`org.jvnet.jaxb2`的`[maven-jaxb2-plugin](https://web.archive.org/web/20220526040446/https://search.maven.org/search?q=g:org.jvnet.jaxb2.maven2%20a:maven-jaxb2-plugin)`可以派上用场，详见[在春天](/web/20220526040446/https://www.baeldung.com/spring-soap-web-service#1-generate-client-code)调用 SOAP Web 服务。

生成的文件:

*   `EmployeeServiceTopDown.java`–是包含方法定义的服务端点接口(SEI)
*   `ObjectFactory.java`–包含以编程方式创建模式派生类实例的工厂方法
*   `EmployeeServiceTopDown_Service.java`–是 JAX-WS 客户端可以使用的服务提供者类

### 5.2。 **Web 服务端点接口**

`wsimport` 工具已经生成了 web 服务端点接口 `EmployeeServiceTopDown`。它声明了 web 服务方法:

```java
@WebService(
  name = "EmployeeServiceTopDown", 
  targetNamespace = "http://topdown.server.jaxws.baeldung.com/")
@SOAPBinding(parameterStyle = SOAPBinding.ParameterStyle.BARE)
@XmlSeeAlso({
    ObjectFactory.class
})
public interface EmployeeServiceTopDown {
    @WebMethod(
      action = "http://topdown.server.jaxws.baeldung.com/"
      + "EmployeeServiceTopDown/countEmployees")
    @WebResult(
      name = "countEmployeesResponse", 
      targetNamespace = "http://topdown.server.jaxws.baeldung.com/", 
      partName = "parameters")
    public int countEmployees();
}
```

### 5.3。 **Web 服务实现**

`wsimport`工具已经创建了 web 服务的结构。我们必须创建 web 服务的实现:

```java
@WebService(
  name = "EmployeeServiceTopDown", 
  endpointInterface = "com.baeldung.jaxws.server.topdown.EmployeeServiceTopDown",
  targetNamespace = "http://topdown.server.jaxws.baeldung.com/")
public class EmployeeServiceTopDownImpl 
  implements EmployeeServiceTopDown {

    @Inject 
    private EmployeeRepository employeeRepositoryImpl;

    @WebMethod
    public int countEmployees() {
        return employeeRepositoryImpl.count();
    }
}
```

## 6。自下而上(合同最后)的方法

在自底向上的方法中，我们必须创建端点接口和实现类。当 web 服务发布时，从类中生成 WSDL。

让我们创建一个 web 服务，它将对`Employee`数据执行简单的 CRUD 操作。

### 6.1。模型类

`Employee`模型类:

```java
public class Employee {
    private int id;
    private String firstName;

    // standard getters and setters
}
```

### 6.2。 **Web 服务端点接口**

声明 web 服务方法的 web 服务端点接口:

```java
@WebService
public interface EmployeeService {
    @WebMethod
    Employee getEmployee(int id);

    @WebMethod
    Employee updateEmployee(int id, String name);

    @WebMethod
    boolean deleteEmployee(int id);

    @WebMethod
    Employee addEmployee(int id, String name);

    // ...
}
```

这个接口为 web 服务定义了一个抽象契约。使用的注释:

*   `@WebService`表示它是一个 web 服务接口
*   `@WebMethod`用于定制一个 web 服务操作
*   `@WebResult` 用于定制表示返回值的 XML 元素的名称

### 6.3。 **Web 服务实现**

web 服务端点接口的实现类:

```java
@WebService(endpointInterface = "com.baeldung.jaxws.EmployeeService")
public class EmployeeServiceImpl implements EmployeeService {

    @Inject 
    private EmployeeRepository employeeRepositoryImpl;

    @WebMethod
    public Employee getEmployee(int id) {
        return employeeRepositoryImpl.getEmployee(id);
    }

    @WebMethod
    public Employee updateEmployee(int id, String name) {
        return employeeRepositoryImpl.updateEmployee(id, name);
    }

    @WebMethod
    public boolean deleteEmployee(int id) {
        return employeeRepositoryImpl.deleteEmployee(id);
    }

    @WebMethod
    public Employee addEmployee(int id, String name) {
        return employeeRepositoryImpl.addEmployee(id, name);
    }

    // ...
}
```

## 7。发布 Web 服务端点

为了发布 web 服务(自顶向下和自底向上)，我们需要将 web 服务实现的地址和实例传递给`javax.xml.ws.Endpoint` 类的`publish()`方法:

```java
public class EmployeeServicePublisher {
    public static void main(String[] args) {
        Endpoint.publish(
          "http://localhost:8080/employeeservicetopdown", 
           new EmployeeServiceTopDownImpl());

        Endpoint.publish("http://localhost:8080/employeeservice", 
          new EmployeeServiceImpl());
    }
}
```

我们现在可以运行`EmployeeServicePublisher` 来启动 web 服务。为了利用 CDI 特性，可以将 web 服务作为 WAR 文件部署到 WildFly 或 GlassFish 等应用服务器。

## 8。远程网络服务客户端

现在让我们创建一个 JAX-WS 客户端来远程连接到`EmployeeService` web 服务。

### 8.1。生成客户端工件

为了生成 JAX-WS 客户端工件，我们可以再次使用`wsimport` 工具:

```java
wsimport -keep -p com.baeldung.jaxws.client http://localhost:8080/employeeservice?wsdl
```

生成的`EmployeeService_Service` 类封装了使用`URL`和`QName`获取服务器端口的逻辑。

### 8.2。连接到网络服务

web 服务客户端使用生成的`EmployeeService_Service`连接到服务器并远程调用 web 服务:

```java
public class EmployeeServiceClient {
    public static void main(String[] args) throws Exception {
        URL url = new URL("http://localhost:8080/employeeservice?wsdl");

        EmployeeService_Service employeeService_Service 
          = new EmployeeService_Service(url);
        EmployeeService employeeServiceProxy 
          = employeeService_Service.getEmployeeServiceImplPort();

        List<Employee> allEmployees 
          = employeeServiceProxy.getAllEmployees();
    }
}
```

## 9。结论

这篇文章是对使用 JAX-WS 的 SOAP Web 服务的快速介绍。

我们使用自底向上和自顶向下的方法来创建使用 JAX-WS API 的 SOAP Web 服务。我们还编写了一个 JAX-WS 客户端，它可以远程连接到服务器并进行 web 服务调用。

完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220526040446/https://github.com/eugenp/tutorials/tree/master/jee-7)