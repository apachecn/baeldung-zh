# Apache CXF 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/introduction-to-apache-cxf>

## 1。概述

Apache CXF 是一个完全符合 JAX-WS 规范的框架。

在 JAX-WS 标准定义的特性之上，Apache CXF 提供了 WSDL 和 Java 类之间的转换能力、用于操作原始 XML 消息的 API、对 JAX-RS 的支持、与 Spring 框架的集成等等。

本教程是 Apache CXF 系列的第一部分，介绍框架的基本特征。它只在源代码中使用 JAX-WS 标准 API，同时仍然在幕后利用 Apache CXF，例如自动生成的 WSDL 元数据和 CXF 默认配置。

## 2。Maven 依赖关系

使用 Apache CXF 需要的关键依赖是`org.apache.cxf:cxf–rt–frontend–` `jaxws`。这提供了一个 JAX-WS 实现来替换内置的 JDK 实现:

```java
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-frontend-jaxws</artifactId>
    <version>3.1.6</version>
</dependency>
```

注意这个工件在`META-INF/services`目录中包含一个名为`javax.xml.ws.spi.Provider`的文件。Java VM 查看该文件的第一行，以确定要利用的 JAX-WS 实现。在本例中，该行的内容是`o` `rg.apache.cxf.jaxws.spi.ProviderImpl`，引用 Apache CXF 提供的实现。

在本教程中，我们不使用 servlet 容器来发布服务，因此需要另一个依赖项来提供必要的 Java 类型定义:

```java
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-transports-http-jetty</artifactId>
    <version>3.1.6</version>
</dependency>
```

关于这些依赖项的最新版本，请查看 Maven 中央存储库中的 [`cxf-rt-frontend-jaxws`](https://web.archive.org/web/20220815042407/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.cxf%22%20AND%20a%3A%22cxf-rt-frontend-jaxws%22) 和 [`cxf-rt-transports-http-jetty`](https://web.archive.org/web/20220815042407/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.cxf%22%20AND%20a%3A%22cxf-rt-transports-http-jetty%22) 。

## 3。Web 服务端点

让我们从用于配置服务端点的实现类开始:

```java
@WebService(endpointInterface = "com.baeldung.cxf.introduction.Baeldung")
public class BaeldungImpl implements Baeldung {
    private Map<Integer, Student> students 
      = new LinkedHashMap<Integer, Student>();

    public String hello(String name) {
        return "Hello " + name;
    }

    public String helloStudent(Student student) {
        students.put(students.size() + 1, student);
        return "Hello " + student.getName();
    }

    public Map<Integer, Student> getStudents() {
        return students;
    }
}
```

这里需要注意的最重要的事情是在`@WebService`注释中出现了`endpointInterface`属性。该属性指向一个为 web 服务定义抽象契约的接口。

端点接口中声明的所有方法签名都需要实现，但是实现接口并不是必需的。

这里的`BaeldungImpl`实现类仍然实现下面的端点接口，以表明接口的所有声明方法都已实现，但这是可选的:

```java
@WebService
public interface Baeldung {
    public String hello(String name);

    public String helloStudent(Student student);

    @XmlJavaTypeAdapter(StudentMapAdapter.class)
    public Map<Integer, Student> getStudents();
}
```

默认情况下，Apache CXF 使用 JAXB 作为其数据绑定架构。然而，由于 JAXB 不直接支持绑定从`getStudents`方法返回的`Map`，**，我们需要一个适配器将`Map`转换成 JAXB 可以使用的 Java 类**。

此外，为了将契约元素从它们的实现中分离出来，我们将`Student`定义为一个接口，而 JAXB 也不直接支持接口，因此我们需要另外一个适配器来处理这个问题。事实上，为了方便起见，我们可以将`Student`声明为一个类。使用这种类型作为接口只是使用适配类的又一个例子。

适配器在下面一节中演示。

## 4。定制适配器

本节说明了如何使用适配类来支持 Java 接口和使用 JAXB 的`Map`的绑定。

### 4.1。接口适配器

这就是`Student`接口的定义:

```java
@XmlJavaTypeAdapter(StudentAdapter.class)
public interface Student {
    public String getName();
}
```

该接口只声明了一个返回`String`的方法，并将`StudentAdapter`指定为适配类，以将自身映射到可以应用 JAXB 绑定的类型，或者从该类型映射。

`StudentAdapter`类定义如下:

```java
public class StudentAdapter extends XmlAdapter<StudentImpl, Student> {
    public StudentImpl marshal(Student student) throws Exception {
        if (student instanceof StudentImpl) {
            return (StudentImpl) student;
        }
        return new StudentImpl(student.getName());
    }

    public Student unmarshal(StudentImpl student) throws Exception {
        return student;
    }
}
```

一个适配类必须实现`XmlAdapter`接口，并为`marshal`和`unmarshal`方法提供实现。`marshal`方法将绑定类型(`Student`，JAXB 不能直接处理的接口)转换为值类型(`StudentImpl`，JAXB 可以处理的具体类)。`unmarshal`方法以相反的方式做事情。

下面是`StudentImpl`类的定义:

```java
@XmlType(name = "Student")
public class StudentImpl implements Student {
    private String name;

    // constructors, getter and setter
}
```

### 4.2。`Map`适配器

`Baeldung`端点接口的`getStudents`方法返回一个`Map`，并指示一个适配类将`Map`转换为 JAXB 可以处理的类型。类似于`StudentAdapter`类，这个适配类必须实现`XmlAdapter`接口的`marshal`和`unmarshal`方法:

```java
public class StudentMapAdapter 
  extends XmlAdapter<StudentMap, Map<Integer, Student>> {
    public StudentMap marshal(Map<Integer, Student> boundMap) throws Exception {
        StudentMap valueMap = new StudentMap();
        for (Map.Entry<Integer, Student> boundEntry : boundMap.entrySet()) {
            StudentMap.StudentEntry valueEntry  = new StudentMap.StudentEntry();
            valueEntry.setStudent(boundEntry.getValue());
            valueEntry.setId(boundEntry.getKey());
            valueMap.getEntries().add(valueEntry);
        }
        return valueMap;
    }

    public Map<Integer, Student> unmarshal(StudentMap valueMap) throws Exception {
        Map<Integer, Student> boundMap = new LinkedHashMap<Integer, Student>();
        for (StudentMap.StudentEntry studentEntry : valueMap.getEntries()) {
            boundMap.put(studentEntry.getId(), studentEntry.getStudent());
        }
        return boundMap;
    }
}
```

`StudentMapAdapter`类将`Map<Integer, Student>`映射到`StudentMap`值类型，并从值类型映射出，定义如下:

```java
@XmlType(name = "StudentMap")
public class StudentMap {
    private List<StudentEntry> entries = new ArrayList<StudentEntry>();

    @XmlElement(nillable = false, name = "entry")
    public List<StudentEntry> getEntries() {
        return entries;
    }

    @XmlType(name = "StudentEntry")
    public static class StudentEntry {
        private Integer id;
        private Student student;

        // getters and setters
    }
}
```

## 5.部署

### 5.1。`Server`定义

为了部署上面讨论的 web 服务，我们将利用标准的 JAX-WS API。因为我们使用的是 Apache CXF，所以框架做了一些额外的工作，例如生成和发布 WSDL 模式。下面是服务服务器的定义:

```java
public class Server {
    public static void main(String args[]) throws InterruptedException {
        BaeldungImpl implementor = new BaeldungImpl();
        String address = "http://localhost:8080/baeldung";
        Endpoint.publish(address, implementor);
        Thread.sleep(60 * 1000);        
        System.exit(0);
    }
}
```

在服务器处于活动状态一段时间以便于测试之后，应该将其关闭以释放系统资源。通过向`Thread.sleep`方法传递一个`long`参数，您可以根据自己的需要为服务器指定任何工作持续时间。

### 5.2。`Server` 的部署

在本教程中，我们使用`org.codehaus.mojo:exec-maven-plugin`插件来实例化上述服务器并控制其生命周期。这在 Maven POM 文件中声明如下:

```java
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>exec-maven-plugin</artifactId>
    <configuration>
        <mainClass>com.baeldung.cxf.introduction.Server</mainClass>
    </configuration>
</plugin>
```

`mainClass`配置指的是发布 web 服务端点的`Server`类。在运行了这个插件的`java`目标之后，我们可以通过访问 URL*http://localhost:8080/bael dung 来查看 Apache CXF 自动生成的 WSDL 模式？wsdl* 。

## 6。测试用例

本节将带您完成编写测试用例的步骤，这些测试用例用于验证我们之前创建的 web 服务。

请注意，在运行任何测试之前，我们需要执行`exec:java`目标来启动 web 服务服务器。

### 6.1。准备工作

第一步是为测试类声明几个字段:

```java
public class StudentTest {
    private static QName SERVICE_NAME 
      = new QName("http://introduction.cxf.baeldung.com/", "Baeldung");
    private static QName PORT_NAME 
      = new QName("http://introduction.cxf.baeldung.com/", "BaeldungPort");

    private Service service;
    private Baeldung baeldungProxy;
    private BaeldungImpl baeldungImpl;

    // other declarations
}
```

以下初始化程序块用于在运行任何测试之前初始化`javax.xml.ws.Service`类型的`service`字段:

```java
{
    service = Service.create(SERVICE_NAME);
    String endpointAddress = "http://localhost:8080/baeldung";
    service.addPort(PORT_NAME, SOAPBinding.SOAP11HTTP_BINDING, endpointAddress);
}
```

将 JUnit 依赖项添加到 POM 文件后，我们可以使用下面代码片段中的`@Before`注释。该方法在每次测试之前运行，以重新实例化`Baeldung`字段:

```java
@Before
public void reinstantiateBaeldungInstances() {
    baeldungImpl = new BaeldungImpl();
    baeldungProxy = service.getPort(PORT_NAME, Baeldung.class);
}
```

`baeldungProxy`变量是 web 服务端点的代理，而`baeldungImpl`只是一个简单的 Java 对象。该对象用于比较通过代理调用远程端点方法和调用本地方法的结果。

注意，`QName`实例由两部分标识:名称空间 URI 和本地部分。如果省略了`Service.getPort`方法的`QName`类型的`PORT_NAME`参数，Apache CXF 将假设该参数的名称空间 URI 是端点接口的包名(顺序相反),其本地部分是接口名加上`Port`,这与`PORT_NAME.`的值完全相同。因此，在本教程中，我们可以省略该参数。

### 6.2。测试实施

我们在这一小节中展示的第一个测试用例是验证从服务端点上的`hello`方法的远程调用返回的响应:

```java
@Test
public void whenUsingHelloMethod_thenCorrect() {
    String endpointResponse = baeldungProxy.hello("Baeldung");
    String localResponse = baeldungImpl.hello("Baeldung");
    assertEquals(localResponse, endpointResponse);
}
```

很明显，远程端点方法返回与本地方法相同的响应，这意味着 web 服务按预期工作。

下一个测试案例演示了`helloStudent`方法的使用:

```java
@Test
public void whenUsingHelloStudentMethod_thenCorrect() {
    Student student = new StudentImpl("John Doe");
    String endpointResponse = baeldungProxy.helloStudent(student);
    String localResponse = baeldungImpl.helloStudent(student);
    assertEquals(localResponse, endpointResponse);
}
```

在这种情况下，客户端向端点提交一个`Student`对象，并收到一条包含学生姓名的消息作为回报。像前面的测试用例一样，来自远程和本地调用的响应是相同的。

我们在这里展示的最后一个测试案例更加复杂。根据服务端点实现类的定义，每次客户端调用端点上的`helloStudent`方法时，提交的`Student`对象将被存储在缓存中。这个缓存可以通过在端点上调用`getStudents`方法来检索。下面的测试用例确认了`students`缓存的内容代表了客户端发送给 web 服务的内容:

```java
@Test
public void usingGetStudentsMethod_thenCorrect() {
    Student student1 = new StudentImpl("Adam");
    baeldungProxy.helloStudent(student1);

    Student student2 = new StudentImpl("Eve");
    baeldungProxy.helloStudent(student2);

    Map<Integer, Student> students = baeldungProxy.getStudents();       
    assertEquals("Adam", students.get(1).getName());
    assertEquals("Eve", students.get(2).getName());
}
```

## 7。结论

本教程介绍了 Apache CXF，这是一个用 Java 处理 web 服务的强大框架。它关注于框架作为标准 JAX-WS 实现的应用，同时仍然在运行时利用框架的特定功能。

所有这些例子和代码片段的实现都可以在 GitHub 项目中找到。