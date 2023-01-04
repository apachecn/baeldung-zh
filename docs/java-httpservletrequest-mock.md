# 如何模仿 HttpServletRequest

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-httpservletrequest-mock>

## 1.概观

在这个快速教程中，**我们将看看模仿`HttpServletRequest`物体**的几种方法。

首先，我们将从 Spring 测试库中的一个全功能模拟类型——[`MockHttpServletRequest`](https://web.archive.org/web/20220813180250/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/mock/web/MockHttpServletRequest.html)开始。然后，我们将看到如何使用两个流行的模仿库进行测试——mock ITO 和 JMockit。最后，我们将看到如何使用匿名子类进行测试。

## 2.测试`HttpServletRequest`

当我们想要模拟客户端请求信息(如 [`HttpServletRequest`](https://web.archive.org/web/20220813180250/https://tomcat.apache.org/tomcat-5.5-doc/servletapi/javax/servlet/http/HttpServletRequest.html) )时，测试[servlet](/web/20220813180250/https://www.baeldung.com/tag/servlet/)可能会很棘手。此外，这个接口定义了各种方法，有不同的方法可以模仿这些方法。

让我们看看我们想要测试的目标`UserServlet`类:

```java
public class UserServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String firstName = request.getParameter("firstName");
        String lastName = request.getParameter("lastName");

        response.getWriter().append("Full Name: " + firstName + " " + lastName);
    }
}
```

为了对`doGet()`方法进行单元测试，我们需要模拟`request`和`response`参数来模拟实际的运行时行为。

## 3.使用来自弹簧的`MockHttpServletRequest`

[Spring-Test](https://web.archive.org/web/20220813180250/https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#integration-testing-overview) 库提供了实现`HttpServletRequest`接口的全功能类`[MockHttpServletRequest](https://web.archive.org/web/20220813180250/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/mock/web/MockHttpServletRequest.html)` 。

尽管这个库主要是为了测试 Spring 应用程序，**我们可以使用它的`MockHttpServletRequest`类，而不需要实现任何 Spring 特有的功能。换句话说，即使应用程序不使用 Spring，我们仍然可以拥有这种依赖来模仿`HttpServletRequest` 对象`.`**

让我们将这个[依赖关系](https://web.archive.org/web/20220813180250/https://search.maven.org/search?q=a:spring-test%20AND%20g:org.springframework)添加到 pom.xml:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.3.20</version>
    <scope>test</scope>
</dependency>
```

现在，让我们看看如何使用这个类来测试`UserServlet`:

```java
@Test
void givenHttpServletRequest_whenUsingMockHttpServletRequest_thenReturnsParameterValues() throws IOException {
    MockHttpServletRequest request = new MockHttpServletRequest();
    request.setParameter("firstName", "Spring");
    request.setParameter("lastName", "Test");
    MockHttpServletResponse response = new MockHttpServletResponse();

    servlet.doGet(request, response);

    assertThat(response.getContentAsString()).isEqualTo("Full Name: Spring Test");
}
```

在这里，我们可以注意到没有实际的嘲讽。我们已经使用了全功能的请求和响应对象，并且只用几行代码就测试了目标类。结果，测试代码是干净的，可读的，可维护的。

## 4.使用模仿框架

或者，**mock 框架提供了一个干净简单的 API 来测试模仿原始对象**运行时行为的 [mock 对象](/web/20220813180250/https://www.baeldung.com/mockito-vs-easymock-vs-jmockit#3-mock-concepts-and-definition)。

它们的一些优点是它们的可表达性和模仿`static`和`private`方法的现成能力。此外，我们可以避免模仿所需的大部分样板代码(与定制实现相比),而是专注于测试。

### 4.1.使用 Mockito

[Mockito](/web/20220813180250/https://www.baeldung.com/tag/mockito/) 是一个流行的开源测试自动化框架，它在内部使用 [Java 反射](/web/20220813180250/https://www.baeldung.com/java-reflection) API 来创建模拟对象。

让我们从将 mockito-core [依赖项](https://web.archive.org/web/20220813180250/https://search.maven.org/search?q=mockito-core)添加到我们的`pom.xml`开始:

```java
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>4.4.0</version>
    <scope>test</scope>
</dependency>
```

接下来，让我们看看如何从`HttpServletRequest`对象模拟`getParameter()`方法:

```java
@Test
void givenHttpServletRequest_whenMockedWithMockito_thenReturnsParameterValues() throws IOException {
    // mock HttpServletRequest & HttpServletResponse
    HttpServletRequest request = mock(HttpServletRequest.class);
    HttpServletResponse response = mock(HttpServletResponse.class);

    // mock the returned value of request.getParameterMap()
    when(request.getParameter("firstName")).thenReturn("Mockito");
    when(request.getParameter("lastName")).thenReturn("Test");
    when(response.getWriter()).thenReturn(new PrintWriter(writer));

    servlet.doGet(request, response);

    assertThat(writer.toString()).isEqualTo("Full Name: Mockito Test");
}
```

### 4.2.使用 JMockit

[JMockit](/web/20220813180250/https://www.baeldung.com/jmockit-101) 是一个模仿 API，它提供了有用的记录和验证语法(我们可以用它来处理 [JUnit](https://web.archive.org/web/20220813180250/http://junit.org/junit4/) 和 [TestNG](https://web.archive.org/web/20220813180250/http://testng.org/doc/index.html) 的语法)。这是一个用于 Java EE 和基于 Spring 的应用程序的容器外集成测试库。让我们看看如何使用 JMockit 模仿`HttpServletRequest`。

首先，我们将把`jmockit` [依赖项](https://web.archive.org/web/20220813180250/https://search.maven.org/search?q=a:jmockit%20AND%20g:org.jmockit)添加到我们的项目中:

```java
<dependency> 
    <groupId>org.jmockit</groupId> 
    <artifactId>jmockit</artifactId> 
    <version>1.49</version>
    <scope>test</scope>
</dependency>
```

接下来，让我们继续测试类中的模拟实现:

```java
@Mocked
HttpServletRequest mockRequest;
@Mocked
HttpServletResponse mockResponse;

@Test
void givenHttpServletRequest_whenMockedWithJMockit_thenReturnsParameterValues() throws IOException {
    new Expectations() {{
        mockRequest.getParameter("firstName"); result = "JMockit";
        mockRequest.getParameter("lastName"); result = "Test";
        mockResponse.getWriter(); result = new PrintWriter(writer);
    }};

    servlet.doGet(mockRequest, mockResponse);

    assertThat(writer.toString()).isEqualTo("Full Name: JMockit Test");
}
```

正如我们在上面看到的，通过几行设置，我们已经成功地用一个模拟`HttpServletRequest`对象测试了目标类。

因此，**模仿框架可以节省我们大量的跑腿工作，并使编写单元测试更快。相反，要使用模拟对象，需要理解模拟 API，并且通常需要一个单独的框架。**

## 5.使用匿名子类

一些项目可能有依赖约束，或者更喜欢直接控制他们自己的测试类实现。具体来说，这在较大的 servlet 代码库的情况下可能很有用，在这种情况下，定制实现的可重用性很重要。在这些情况下，匿名类就派上了用场。

**[匿名类](/web/20220813180250/https://www.baeldung.com/java-anonymous-classes)是没有名字的内部类**。**此外，它们可以快速实现并提供对实际对象的直接控制。如果我们不想包含额外的测试依赖，可以考虑这种方法。**

现在，让我们创建一个实现`HttpServletRequest` 接口的匿名子类，并用它来测试`doGet()`方法:

```java
public static HttpServletRequest getRequest(Map<String, String[]> params) {
    return new HttpServletRequest() {
        public Map<String, String[]> getParameterMap() {
            return params;
        }

        public String getParameter(String name) {
            String[] values = params.get(name);
            if (values == null || values.length == 0) {
                return null;
            }
            return values[0];
        }

        // More methods to implement
    }
};
```

接下来，让我们将这个请求传递给被测试的类:

```java
@Test
void givenHttpServletRequest_whenUsingAnonymousClass_thenReturnsParameterValues() throws IOException {
    final Map<String, String[]> params = new HashMap<>();
    params.put("firstName", new String[] { "Anonymous Class" });
    params.put("lastName", new String[] { "Test" });

    servlet.doGet(getRequest(params), getResponse(writer));

    assertThat(writer.toString()).isEqualTo("Full Name: Anonymous Class Test");
}
```

**这种解决方案的缺点是需要为所有抽象方法创建一个带有虚拟实现的匿名类。此外，** **像`HttpSession`这样的嵌套对象可能需要特定的实现**。

## 6.结论

在本文中，我们讨论了在为 servlets 编写单元测试时模仿`HttpServletRequest` 对象的一些选项。除了使用 mocking 框架之外，我们看到用`MockHttpServletRequest` 类进行测试似乎比定制实现更加干净和高效。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220813180250/https://github.com/eugenp/tutorials/tree/master/javax-servlets-2)