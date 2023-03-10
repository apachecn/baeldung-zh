# Spring 和 EJB 的集成指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-ejb>

## 1。概述

在本文中，我们将展示如何**集成 Spring 和远程企业 Java Beans (EJB)** 。

为此，我们将创建一些 EJB 和必要的远程接口，然后在 JEE 容器中运行它们。之后，我们将启动我们的 Spring 应用程序，并使用远程接口实例化我们的 beans，以便它们可以执行远程调用。

如果对 EJB 是什么或者它们如何工作有任何疑问，我们已经在这里发表了一篇关于主题[的介绍性文章。](/web/20220524003411/https://www.baeldung.com/ejb-intro)

## 2。EJB 设置

我们需要创建我们的远程接口和 EJB 实现。为了使它们可用，我们还需要一个容器来保存和管理 beans。

### 2.1。EJB 远程接口

让我们从定义两个非常简单的 beans 开始——一个无状态，一个有状态。

我们将从它们的接口开始:

```java
@Remote
public interface HelloStatefulWorld {
    int howManyTimes();
    String getHelloWorld();
} 
```

```java
@Remote
public interface HelloStatelessWorld {
    String getHelloWorld();
}
```

### 2.2。EJB 实施

现在，让我们实现我们的远程 EJB 接口:

```java
@Stateful(name = "HelloStatefulWorld")
public class HelloStatefulWorldBean implements HelloStatefulWorld {

    private int howManyTimes = 0;

    public int howManyTimes() {
        return howManyTimes;
    }

    public String getHelloWorld() {
        howManyTimes++;
        return "Hello Stateful World";
    }
} 
```

```java
@Stateless(name = "HelloStatelessWorld")
public class HelloStatelessWorldBean implements HelloStatelessWorld {

    public String getHelloWorld() {
        return "Hello Stateless World!";
    }
} 
```

如果有状态和无状态 beans 听起来不熟悉，这篇介绍文章可能会派上用场。

### 2.3。EJB 集装箱

我们可以在任何 JEE 容器中运行我们的代码，但是出于实用目的，我们将使用 Wildfly 和`cargo` Maven 插件来为我们完成繁重的工作:

```java
<plugin>
    <groupId>org.codehaus.cargo</groupId>
    <artifactId>cargo-maven2-plugin</artifactId>
    <version>1.6.1</version>
    <configuration>
        <container>
            <containerId>wildfly10x</containerId>
            <zipUrlInstaller>
                <url>
                  http://download.jboss.org/wildfly/10.1.0.Final/wildfly-10.1.0.Final.zip
                </url>
            </zipUrlInstaller>
        </container>
        <configuration>
            <properties>
                <cargo.hostname>127.0.0.1</cargo.hostname>
                <cargo.jboss.configuration>standalone-full</cargo.jboss.configuration>
                <cargo.jboss.management-http.port>9990</cargo.jboss.management-http.port>
                <cargo.servlet.users>testUser:admin1234!</cargo.servlet.users>
            </properties>
        </configuration>
    </configuration>
</plugin>
```

### 2.4。运行 EJB

配置好这些之后，我们可以直接从 Maven 命令行运行容器:

```java
mvn clean package cargo:run -Pwildfly-standalone
```

我们现在有了一个 Wildfly 托管我们的 beans 的工作实例。我们可以通过日志行来确认这一点:

```java
java:global/ejb-remote-for-spring/HelloStatefulWorld!com.baeldung.ejb.tutorial.HelloStatefulWorld
java:app/ejb-remote-for-spring/HelloStatefulWorld!com.baeldung.ejb.tutorial.HelloStatefulWorld
java:module/HelloStatefulWorld!com.baeldung.ejb.tutorial.HelloStatefulWorld
java:jboss/exported/ejb-remote-for-spring/HelloStatefulWorld!com.baeldung.ejb.tutorial.HelloStatefulWorld
java:global/ejb-remote-for-spring/HelloStatefulWorld
java:app/ejb-remote-for-spring/HelloStatefulWorld
java:module/HelloStatefulWorld 
```

```java
java:global/ejb-remote-for-spring/HelloStatelessWorld!com.baeldung.ejb.tutorial.HelloStatelessWorld
java:app/ejb-remote-for-spring/HelloStatelessWorld!com.baeldung.ejb.tutorial.HelloStatelessWorld
java:module/HelloStatelessWorld!com.baeldung.ejb.tutorial.HelloStatelessWorld
java:jboss/exported/ejb-remote-for-spring/HelloStatelessWorld!com.baeldung.ejb.tutorial.HelloStatelessWorld
java:global/ejb-remote-for-spring/HelloStatelessWorld
java:app/ejb-remote-for-spring/HelloStatelessWorld
java:module/HelloStatelessWorld
```

## 3。弹簧设置

既然我们已经启动并运行了 JEE 容器，并且部署了 EJB，我们就可以启动我们的 Spring 应用程序了。我们将使用`spring-boot-web`来使手动测试更容易，但是对于远程调用来说，这不是强制性的。

### 3.1。Maven 依赖关系

为了能够连接到远程 EJB，我们需要`Wildfly EJB Client`库和我们的远程接口:

```java
<dependency>
    <groupId>org.wildfly</groupId>
    <artifactId>wildfly-ejb-client-bom</artifactId>
    <version>10.1.0.Final</version>
    <type>pom</type>
</dependency>
<dependency>
    <groupId>com.baeldung.spring.ejb</groupId>
    <artifactId>ejb-remote-for-spring</artifactId>
    <version>1.0.1</version>
    <type>ejb</type>
</dependency>
```

最后版本的`wildfly-ejb-client-bom`可以在[这里](https://web.archive.org/web/20220524003411/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.wildfly%22%20AND%20a%3A%22wildfly-ejb-client-bom%22)找到。

### 3.2。命名策略上下文

有了类路径中的这些依赖项，我们可以**实例化一个`javax.naming.Context`来查找我们的远程 bean**。我们将把它创建为一个 Spring Bean，这样我们可以在需要时自动连接它:

```java
@Bean   
public Context context() throws NamingException {
    Properties jndiProps = new Properties();
    jndiProps.put("java.naming.factory.initial", 
      "org.jboss.naming.remote.client.InitialContextFactory");
    jndiProps.put("jboss.naming.client.ejb.context", true);
    jndiProps.put("java.naming.provider.url", 
      "http-remoting://localhost:8080");
    return new InitialContext(jndiProps);
}
```

这些属性是**通知远程`URL`和命名策略上下文**所必需的。

### 3.3。JNDI 模式

在我们将远程 beans 连接到 Spring 容器之前，我们需要知道如何到达它们。为此，我们将使用他们的 JNDI 绑定。让我们看看这些绑定的标准模式:

```java
${appName}/${moduleName}/${distinctName}/${beanName}!${viewClassName}
```

请记住，**因为我们部署了一个简单的`jar`而不是一个`ear`，并且没有明确设置名称，所以我们没有一个`appName`和一个`distinctName`T5。在我们的 [EJB 介绍文章](/web/20220524003411/https://www.baeldung.com/ejb-intro)中有更多的细节，以防有些事情看起来很奇怪。**

我们将使用这种模式将远程 beans 绑定到我们的 Spring bean。

### 3.4。构建我们的春豆

为了访问我们的 EJB，我们将使用前面提到的 JNDI。还记得我们用来检查我们的企业 beans 是否被部署的日志行吗？

我们将看到现在正在使用的信息:

```java
@Bean
public HelloStatelessWorld helloStatelessWorld(Context context) 
  throws NamingException {

    return (HelloStatelessWorld) 
      context.lookup(this.getFullName(HelloStatelessWorld.class));
} 
```

```java
@Bean
public HelloStatefulWorld helloStatefulWorld(Context context) 
  throws NamingException {

    return (HelloStatefulWorld) 
      context.lookup(this.getFullName(HelloStatefulWorld.class));
} 
```

```java
private String getFullName(Class classType) {
    String moduleName = "ejb-remote-for-spring/";
    String beanName = classType.getSimpleName();
    String viewClassName = classType.getName();
    return moduleName + beanName + "!" + viewClassName;
}
```

**我们需要非常小心正确的完全 JNDI 绑定**，否则上下文将无法到达远程 EJB 并创建必要的底层基础设施。

请记住，`Context` 中的方法`lookup` 会抛出一个`NamingException`，以防它没有找到您需要的 bean。

## 4。整合

一切就绪后，我们可以**将 beans 注入控制器**，这样我们就可以测试连接是否正确:

```java
@RestController
public class HomeEndpoint {

    // ...

    @GetMapping("/stateless")
    public String getStateless() {
        return helloStatelessWorld.getHelloWorld();
    }

    @GetMapping("/stateful")
    public String getStateful() {
        return helloStatefulWorld.getHelloWorld()
          + " called " + helloStatefulWorld.howManyTimes() + " times";
    }
}
```

让我们启动 Spring 服务器并检查一些日志。我们将看到下面一行，表示一切正常:

```java
EJBCLIENT000013: Successful version handshake completed
```

现在，让我们测试我们的无状态 bean。我们可以尝试一些`curl`命令来验证它们是否按预期运行:

```java
curl http://localhost:8081/stateless
Hello Stateless World!
```

让我们检查一下有状态的:

```java
curl http://localhost:8081/stateful
Hello Stateful World called 1 times

curl http://localhost:8081/stateful
Hello Stateful World called 2 times
```

## 5。结论

在本文中，我们学习了如何将 Spring 集成到 EJB，并远程调用 JEE 容器。我们创建了两个远程 EJB 接口，并且能够以透明的方式调用那些使用 Spring Beans 的接口。

尽管 Spring 被广泛采用，EJB 在企业环境中仍然很流行，在这个简单的例子中，我们已经展示了利用 Jakarta EE 的分布式优势和 Spring 应用程序的易用性是可能的。

和往常一样，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524003411/https://github.com/eugenp/tutorials/tree/master/spring-ejb)