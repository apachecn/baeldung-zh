# 使用 Maven 进行集成测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-integration-test>

## 1。概述

Maven 是 Java 领域最流行的构建工具，而集成测试是开发过程中必不可少的一部分。因此，**用 Maven 配置和执行集成测试是很自然的选择。**

在本教程中，我们将介绍使用 Maven 进行集成测试以及将集成测试从单元测试中分离出来的多种不同方法。

## 2。准备工作

为了使演示代码接近真实世界的项目，我们将设置一个 JAX 遥感应用程序。该应用程序在执行集成测试之前被部署到服务器上，之后被拆除。

### 2.1。Maven 配置

我们将围绕 Jersey 构建我们的 REST 应用程序——JAX-RS 的参考实现。这种实现需要几个依赖项:

```java
<dependency>
    <groupId>org.glassfish.jersey.containers</groupId>
    <artifactId>jersey-container-servlet-core</artifactId>
    <version>2.27</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.inject</groupId>
    <artifactId>jersey-hk2</artifactId>
    <version>2.27</version>
</dependency>
```

我们可以在这里和这里找到这些依赖关系的最新版本[。](https://web.archive.org/web/20221126234935/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.glassfish.jersey.containers%22%20AND%20a%3A%22jersey-container-servlet-core%22)

我们将使用 Jetty Maven 插件来建立一个测试环境。**这个插件在 Maven 构建生命周期的`pre-integration-test`阶段启动一个 Jetty 服务器，然后在`post-integration-test`阶段停止它。**

下面是我们如何在`pom.xml`中配置 Jetty Maven 插件:

```java
<plugin>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-maven-plugin</artifactId>
    <version>9.4.11.v20180605</version>
    <configuration>
        <httpConnector>
            <port>8999</port>
        </httpConnector>
        <stopKey>quit</stopKey>
        <stopPort>9000</stopPort>
    </configuration>
    <executions>
        <execution>
            <id>start-jetty</id>
            <phase>pre-integration-test</phase>
            <goals>
                <goal>start</goal>
            </goals>
        </execution>
        <execution>
            <id>stop-jetty</id>
            <phase>post-integration-test</phase>
            <goals>
                <goal>stop</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

当 Jetty 服务器启动时，它将监听端口`8999`。`stopKey`和`stopPort`配置元素仅由插件的`stop`目标使用，从我们的角度来看，它们的值并不重要。

这里是找到 Jetty Maven 插件最新版本的地方。

另一件需要注意的事情是，我们必须将`pom.xml`文件中的`packaging`元素设置为`war`，否则 Jetty 插件无法启动服务器:

```java
<packaging>war</packaging>
```

### 2.2。创建 REST 应用程序

应用程序端点非常简单——当 GET 请求命中上下文根时，返回一条欢迎消息:

```java
@Path("/")
public class RestEndpoint {
    @GET
    public String hello() {
        return "Welcome to Baeldung!";
    }
}
```

这是我们向 Jersey 注册端点类的方式:

```java
package com.baeldung.maven.it;

import org.glassfish.jersey.server.ResourceConfig;

public class EndpointConfig extends ResourceConfig {
    public EndpointConfig() {
        register(RestEndpoint.class);
    }
}
```

为了让 Jetty 服务器知道我们的 REST 应用程序，我们可以使用一个经典的`web.xml`部署描述符:

```java
<web-app 

  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
  http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
  version="3.1">
    <servlet>
        <servlet-name>rest-servlet</servlet-name>
        <servlet-class>org.glassfish.jersey.servlet.ServletContainer</servlet-class>
        <init-param>
            <param-name>javax.ws.rs.Application</param-name>
            <param-value>com.baeldung.maven.it.EndpointConfig</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>rest-servlet</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>
</web-app>
```

**该描述符必须放在目录`/src/main/webapp` `/WEB-INF`** 中才能被服务器识别。

### 2.3。客户端测试代码

以下部分中的所有测试类都包含一个方法:

```java
@Test
public void whenSendingGet_thenMessageIsReturned() throws IOException {
    String url = "http://localhost:8999";
    URLConnection connection = new URL(url).openConnection();
    try (InputStream response = connection.getInputStream();
      Scanner scanner = new Scanner(response)) {
        String responseBody = scanner.nextLine();
        assertEquals("Welcome to Baeldung!", responseBody);
    }
}
```

正如我们所看到的，这个方法除了向我们之前设置的 web 应用程序发送 GET 请求并验证响应之外，什么也不做。

## 3。运行中的集成测试

关于集成测试需要注意的一件重要事情是**测试方法通常需要相当长的时间来运行。**

因此，我们应该从默认的构建生命周期中排除集成测试，防止它们在我们每次构建项目时减慢整个过程。

分离集成测试的一种便捷方式是使用构建概要文件。这种配置使我们能够仅在必要时执行集成测试——通过指定合适的概要文件。

在接下来的章节中，我们将用构建概要文件配置所有的集成测试。

## 4。使用故障保护插件进行测试

运行集成测试最简单的方法是使用[Maven`failsafe`插件](/web/20221126234935/https://www.baeldung.com/maven-failsafe-plugin)。

默认情况下，Maven `surefire`插件在`test`阶段执行单元测试，而**`failsafe`插件在`integration-test`阶段**运行集成测试。

我们可以用不同的模式为这些插件命名测试类，以单独提取包含的测试。

**由`surefire`和`failsafe`执行的默认命名约定是不同的，因此我们只需要遵循这些约定来分离单元和集成测试。**

`surefire`插件的执行包括所有名字以`Test`开头，或以`Test`、`Tests`或`TestCase`结尾的类。相比之下，`failsafe`插件在名称以`IT`开头或者以`IT`或者`ITCase`结尾的类中执行测试方法。

[这个](https://web.archive.org/web/20221126234935/https://maven.apache.org/surefire/maven-surefire-plugin/examples/inclusion-exclusion.html)是我们可以找到关于`surefire`测试包含的文档的地方，这里的[是`failsafe`的文档。](https://web.archive.org/web/20221126234935/https://maven.apache.org/surefire/maven-failsafe-plugin/examples/inclusion-exclusion.html)

让我们用默认配置将`failsafe`插件添加到 POM 中:

```java
<profile>
    <id>failsafe</id>
    <build>
        <plugins>
            <plugin>
                <artifactId>maven-failsafe-plugin</artifactId>
                <version>2.22.0</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>integration-test</goal>
                            <goal>verify</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</profile>
```

[这个链接](https://web.archive.org/web/20221126234935/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.maven.plugins%22%20AND%20a%3A%22maven-failsafe-plugin%22)是找到最新版本`failsafe`插件的地方。

使用上述配置，将在`integration-test`阶段执行以下测试方法:

```java
public class RestIT {
    // test method shown in subsection 2.3
}
```

由于 Jetty 服务器在`pre-integration-test`阶段启动，在`post-integration-test`阶段关闭，我们刚刚看到的测试通过了这个命令:

```java
mvn verify -Pfailsafe
```

我们还可以定制命名模式，以包含具有不同名称的类:

```java
<plugin>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>2.22.0</version>
    <configuration>
        <includes>
            <include>**/*RestIT</include>
            <include>**/RestITCase</include>
        </includes>
    </configuration>
    ...
</plugin>
```

## 5。 **使用 Surefire 插件进行测试**

除了`failsafe`插件，**我们还可以使用[插件`surefire`插件](/web/20221126234935/https://www.baeldung.com/maven-surefire-plugin)来执行不同阶段的单元和集成测试。**

假设我们想要用后缀`IntegrationTest`来命名所有的集成测试。由于默认情况下`surefire`插件在`test`阶段使用这样的名称运行测试，我们需要将它们从默认执行中排除:

```java
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>
    <configuration>
        <excludes>
            <exclude>**/*IntegrationTest</exclude>
        </excludes>
    </configuration>
</plugin>
```

这个插件的最新版本是[这里是](https://web.archive.org/web/20221126234935/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.maven.plugins%22%20AND%20a%3A%22maven-surefire-plugin%22)。

我们已经将所有名称以`IntegrationTest`结尾的测试类从构建生命周期中移除。是时候用个人资料把它们放回去了:

```java
<profile>
    <id>surefire</id>
    <build>
        <plugins>
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
                            <includes>
                                <include>**/*IntegrationTest</include>
                            </includes>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</profile>
```

**我们没有将`surefire`插件的`test`目标绑定到`test`构建阶段，而是像往常一样，将其绑定到`integration-test`阶段。**插件将在集成测试过程中启动。

**注意，我们必须将`exclude`元素设置为`none`来覆盖基本配置中指定的排除。**

现在，让我们用我们的命名模式定义一个集成测试类:

```java
public class RestIntegrationTest {
    // test method shown in subsection 2.3
}
```

该测试将使用以下命令运行:

```java
mvn verify -Psurefire
```

## 6。用货物插件进行测试

我们可以将`surefire`插件与 Maven `cargo`插件一起使用。这个插件内置了对嵌入式服务器的支持，这对于集成测试非常有用。

关于这个组合的更多细节可以在[这里](/web/20221126234935/https://www.baeldung.com/integration-testing-with-the-maven-cargo-plugin)找到。

## 7.使用 JUnit 的`@Category`进行测试

有选择地执行测试的一种便捷方式是利用 JUnit 4 框架中的@ `Category`注释。**这个注释让我们从单元测试中排除特定的测试，并将它们包含在集成测试中。**

首先，我们需要一个接口或类作为类别标识符:

```java
package com.baeldung.maven.it;

public interface Integration { }
```

然后我们可以用`@Category`注释和`Integration`标识符来修饰一个测试类:

```java
@Category(Integration.class)
public class RestJUnitTest {
    // test method shown in subsection 2.3
}
```

除了在测试类上声明`@Category`注释，我们还可以在方法级别上使用它来对单个测试方法进行分类。

从`test`构建阶段中排除一个类别很简单:

```java
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>
    <configuration>
        <excludedGroups>com.baeldung.maven.it.Integration</excludedGroups>
    </configuration>
</plugin>
```

在`integration-test`阶段包含`Integration`类别也很简单:

```java
<profile>
    <id>category</id>
        <build>
        <plugins>
            <plugin>
                <artifactId>maven-failsafe-plugin</artifactId>
                <version>2.22.0</version>
                <configuration>
                    <includes>
                        <include>**/*</include>
                    </includes>
                    <groups>com.baeldung.maven.it.Integration</groups>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>integration-test</goal>
                            <goal>verify</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</profile>
```

我们现在可以用 Maven 命令运行集成测试:

```java
mvn verify -Pcategory
```

## 8。为集成测试添加单独的目录

有时为集成测试建立一个单独的目录是可取的。以这种方式组织测试允许我们将集成测试与单元测试完全隔离开来。

为此，我们可以使用 Maven `build helper`插件:

```java
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>build-helper-maven-plugin</artifactId>
    <version>3.0.0</version>
    <executions>
        <execution>
            <id>add-integration-test-source</id>
            <phase>generate-test-sources</phase>
            <goals>
                <goal>add-test-source</goal>
            </goals>
            <configuration>
                <sources>
                    <source>src/integration-test/java</source>
                </sources>
            </configuration>
        </execution>
    </executions>
</plugin>
```

这里是我们可以找到这个插件最新版本的地方。

我们刚刚看到的配置向构建中添加了一个测试源目录。让我们向新目录添加一个类定义:

```java
public class RestITCase {
    // test method shown in subsection 2.3
}
```

是时候在这个类中运行集成测试了:

```java
mvn verify -Pfailsafe
```

由于我们在 3.1 小节中设置的配置，Maven `failsafe`插件将执行这个测试类中的方法。

测试源目录通常伴随着资源目录。我们可以在插件配置的另一个`execution`元素中添加这样一个目录:

```java
<executions>
    ...
    <execution>
        <id>add-integration-test-resource</id>
        <phase>generate-test-resources</phase>
        <goals>
            <goal>add-test-resource</goal>
        </goals>
        <configuration>
            <resources>
                <resource>
                    <directory>src/integration-test/resources</directory>
                </resource>
            </resources>
        </configuration>
    </execution>
</executions>
```

## 9。结论

本文介绍了如何使用 Maven 运行与 Jetty 服务器的集成测试，重点是 Maven `surefire`和`failsafe`插件的配置。

本教程的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221126234935/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-integration-test)