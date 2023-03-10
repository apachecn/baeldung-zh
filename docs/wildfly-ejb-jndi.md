# WildFly 应用服务器上的 EJB JNDI 查找简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/wildfly-ejb-jndi>

## 1。概述

[企业 Java bean](https://web.archive.org/web/20220627185241/https://docs.oracle.com/javaee/6/tutorial/doc/gijsz.html)(EJB)是 [Java EE 规范](https://web.archive.org/web/20220627185241/https://en.wikipedia.org/wiki/Java_Platform,_Enterprise_Edition)的核心部分，旨在简化分布式企业级应用的开发。EJB 的生命周期由应用服务器处理，比如 [JBoss WildFly](https://web.archive.org/web/20220627185241/http://wildfly.org/) 或 [Oracle GlassFish](https://web.archive.org/web/20220627185241/https://www.oracle.com/middleware/technologies/glassfish-server.html) 。

EJB 提供了一个健壮的编程模型，它促进了企业级软件模块的实现，因为由应用服务器来处理与业务逻辑无关的问题，如事务处理、组件生命周期管理或依赖注入。

此外，我们已经发表了两篇文章，涵盖了 EJB 的基本概念，所以请随意查看[这里](/web/20220627185241/https://www.baeldung.com/ejb-intro)和[这里](/web/20220627185241/https://www.baeldung.com/ejb-session-beans)。

在本教程中，我们将展示如何在 WildFly 上实现一个基本的 EJB 模块，并通过一个 [JNDI](https://web.archive.org/web/20220627185241/https://en.wikipedia.org/wiki/Java_Naming_and_Directory_Interface) 从远程客户端调用一个 EJB。

## 2。实施 EJB 模块

业务逻辑由一个或多个本地/远程业务接口(也称为本地/远程视图)实现，或者直接通过不实现任何接口的类(非视图接口)实现。

值得注意的是，当从驻留在相同环境(即相同的 EAR 或 WAR 文件)中的客户机访问 bean 时，使用本地业务接口，而当从不同的环境(即不同的 JVM 或应用服务器)访问 bean 时，则需要远程业务接口。

让我们创建一个基本的 EJB 模块，它将只由一个 bean 组成。bean 的业务逻辑很简单，仅限于将给定的`String`转换成大写版本。

### 2.1。定义远程业务接口

让我们首先定义一个单独的远程业务接口，用 *@Remote* 注释进行修饰。根据 [EJB 3.x 规范](https://web.archive.org/web/20220627185241/https://download.oracle.com/otn-pub/jcp/ejb-3.1-fr-eval-oth-JSpec/ejb-3_1-fr-spec.pdf)，这是强制性的，因为 bean 将从远程客户端访问:

```java
@Remote
public interface TextProcessorRemote {
    String processText(String text);
}
```

### 2.2。定义无状态 Bean

接下来，让我们通过实现上述远程接口来实现业务逻辑:

```java
@Stateless
public class TextProcessorBean implements TextProcessorRemote {
    public String processText(String text) {
        return text.toUpperCase();
    }
}
```

TextProcessorBean 类是一个简单的 Java 类，用*@无状态*注释修饰。

根据定义，无状态 beans 不维护与客户的任何对话状态，即使它们可以跨不同的请求维护实例状态。它们的对应物，有状态 beans，确实保留了它们的对话状态，因此为应用服务器创建它们的成本更高。

在这种情况下，上面的类没有任何实例状态，它可以变成无状态的。如果它有一个状态，那么在不同的客户端请求中使用它根本没有意义。

bean 的行为是确定的，也就是说，它没有副作用，正如一个设计良好的 bean 应该有的那样:它只是接受一个输入`String`并返回它的大写版本。

### 2.3。Maven 依赖关系

接下来，我们需要将 [*javaee-api*](https://web.archive.org/web/20220627185241/https://search.maven.org/classic/#search%7Cga%7C1%7Cjavaee-api) Maven 工件添加到模块中，它提供了所有的 Java EE 7 规范 api，包括 EJB 所需的 API:

```java
<dependency>
    <groupId>javax</groupId>
    <artifactId>javaee-api</artifactId>
    <version>7.0</version>
    <scope>provided</scope>
</dependency>
```

至此，我们已经成功创建了一个基本的功能性 EJB 模块。为了使它对所有潜在的客户可用，我们必须将工件作为 JAR 文件添加到我们的本地 Maven 存储库中。

### 2.4。将 EJB 模块安装到本地存储库中

有几种方法可以实现这一点。最简单的方法是执行 Maven 生命周期的*清理-安装*构建阶段:

```java
mvn clean install
```

这个命令将 EJB 模块作为`ejbmodule-1.0.jar (`或者在 `pom.xml`文件中指定的任意工件 id 安装到我们的本地存储库中。关于如何用 Maven 安装本地 JAR 的更多信息，请查看[这篇文章](/web/20220627185241/https://www.baeldung.com/install-local-jar-with-maven/)。

假设 EJB 模块已经正确安装到我们的本地存储库中，下一步是开发一个远程客户端应用程序，该应用程序使用我们的 *TextProcessorBean* API。

## 3。远程 EJB 客户端

我们将保持远程 EJB 客户端的业务逻辑非常简单:首先，它执行 JNDI 查找来获得一个 *TextProcessorBean* 代理。之后，它调用代理的`processText()`方法。

### 3.1。Maven 依赖关系

我们需要包含以下 Maven 工件，以便 EJB 客户端能够按预期工作:

```java
<dependency>
    <groupId>javax</groupId>
    <artifactId>javaee-api</artifactId>
    <version>7.0</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.wildfly</groupId>
    <artifactId>wildfly-ejb-client-bom</artifactId>
    <version>10.1.0.Final</version>
</dependency>
<dependency>
    <groupId>com.beldung.ejbmodule</groupId>
    <artifactId>ejbmodule</artifactId>
    <version>1.0</version>
</dependency>
```

虽然我们包含 *javaee-api* 工件的原因很明显，但是包含[*wildly-EJ B- client-BOM*](https://web.archive.org/web/20220627185241/https://search.maven.org/classic/#search%7Cga%7C1%7Cwildfly-ejb-client-bom)却不明显。在 WildFly 上执行远程 EJB 调用需要这个工件。

最后但同样重要的是，我们需要让之前的 EJB 模块对客户端可用，所以这也是我们添加了 *ejbmodule* 依赖项的原因。

### 3.2。EJB 客户端类

考虑到 EJB 客户端调用了一个 *TextProcessorBean* 的代理，我们将非常务实地将客户端类命名为 *TextApplication* :

```java
public class TextApplication {

    public static void main(String[] args) throws NamingException {
        TextProcessorRemote textProcessor = EJBFactory
          .createTextProcessorBeanFromJNDI("ejb:");
        System.out.print(textProcessor.processText("sample text"));
    }

    private static class EJBFactory {

        private static TextProcessorRemote createTextProcessorBeanFromJNDI
          (String namespace) throws NamingException {
            return lookupTextProcessorBean(namespace);
        }

        private static TextProcessorRemote lookupTextProcessorBean
          (String namespace) throws NamingException {
            Context ctx = createInitialContext();
            String appName = "";
            String moduleName = "EJBModule";
            String distinctName = "";
            String beanName = TextProcessorBean.class.getSimpleName();
            String viewClassName = TextProcessorRemote.class.getName();
            return (TextProcessorRemote) ctx.lookup(namespace 
              + appName + "/" + moduleName 
              + "/" + distinctName + "/" + beanName + "!" + viewClassName);
        }

        private static Context createInitialContext() throws NamingException {
            Properties jndiProperties = new Properties();
            jndiProperties.put(Context.INITIAL_CONTEXT_FACTORY, 
              "org.jboss.naming.remote.client.InitialContextFactory");
            jndiProperties.put(Context.URL_PKG_PREFIXES, 
              "org.jboss.ejb.client.naming");
            jndiProperties.put(Context.PROVIDER_URL, 
               "http-remoting://localhost:8080");
            jndiProperties.put("jboss.naming.client.ejb.context", true);
            return new InitialContext(jndiProperties);
        }
    }
}
```

简而言之， *TextApplication* 类所做的就是检索 bean 代理，并使用示例字符串调用其 *processText()* 方法。

实际的查找由嵌套类 *EJBFactory* 执行，它首先创建一个 JNDI*[initial context](https://web.archive.org/web/20220627185241/https://docs.oracle.com/en/java/javase/11/docs/api/java.naming/javax/naming/InitialContext.html)*实例，然后将所需的 JNDI 参数传递给构造函数，最后用它来查找 bean 代理。

注意，查找是通过使用 WildFly 专有的“ejb:”名称空间来执行的。这优化了查找过程，因为客户端将连接推迟到服务器，直到显式调用代理。

同样值得注意的是，根本不需要借助“ejb”名称空间就可以查找 bean 代理。然而，**我们将失去惰性网络连接的所有额外好处，从而使客户端的性能大大降低**。

### 3.3。设置 EJB 环境

客户端应该知道与哪个主机和端口建立连接来执行 bean 查找。在这个意义上，**客户端需要设置专有的 WildFly EJB 上下文，这个上下文是用放置在其类路径中的*JBoss-EJ b-client . properties*文件**定义的，通常在 *src/main/resources* 文件夹下:

```java
endpoint.name=client-endpoint
remote.connectionprovider.create.options.org.xnio.Options.SSL_ENABLED=false
remote.connections=default
remote.connection.default.host=127.0.0.1
remote.connection.default.port=8080
remote.connection.default.connect.options.org.xnio.Options
  .SASL_POLICY_NOANONYMOUS=false
remote.connection.default.username=myusername
remote.connection.default.password=mypassword
```

该文件是不言自明的，因为它提供了建立到 WildFly 的连接所需的所有参数，包括默认的远程连接数、默认的主机和端口，以及用户凭证。在这种情况下，连接是不加密的，但是在启用 SSL 时可以加密。

最后要考虑的是**如果连接需要认证，就有必要通过[【add-user.sh/add-user.bat】T2 实用程序](https://web.archive.org/web/20220627185241/https://docs.jboss.org/author/display/WFLY8/add-user%20utility.html)将用户添加到 WildFly。**

## 4。结论

只要我们严格遵守概述的过程，在 WildFly 上执行 EJB 查找是简单的。

像往常一样，本文包含的所有示例都可以在 GitHub [这里](https://web.archive.org/web/20220627185241/https://github.com/eugenp/tutorials/tree/master/spring-ejb/spring-ejb-remote)和[这里](https://web.archive.org/web/20220627185241/https://github.com/eugenp/tutorials/tree/master/spring-ejb/spring-ejb-client)找到。