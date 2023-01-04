# EJB 设置指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ejb-intro>

## 1。概述

在本文中，我们将讨论如何开始企业 JavaBean (EJB)开发。

Enterprise JavaBeans 用于开发可伸缩的、分布式的服务器端组件，通常封装应用程序的业务逻辑。

我们将使用 [*WildFly 10.1.0*](https://web.archive.org/web/20220815025944/http://wildfly.org/) 作为我们的首选服务器解决方案，但是，您可以自由使用您选择的任何 Java 企业应用服务器。

## 2。设置

让我们首先讨论 EJB 3.2 开发所需的 Maven 依赖性，以及如何使用 Maven Cargo 插件或手动配置 WildFly 应用服务器。

### 2.1。Maven 依赖关系

为了使用 EJB 3.2 **，**确保将最新版本添加到`pom.xml` 文件的`dependencies`部分:

```
<dependency>
    <groupId>javax</groupId>
    <artifactId>javaee-api</artifactId>
    <version>7.0</version>
    <scope>provided</scope>
</dependency>
```

You will find the latest dependency in the [Maven Repository](https://web.archive.org/web/20220815025944/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22javax%22%20AND%20a%3A%22javaee-api%22). This dependency ensures that all Java EE 7 APIs are available during compile time. The `provided` scope ensures that once deployed, the dependency will be provided by the container where it has been deployed.

### 2.2。Maven Cargo 的 WildFly 设置

我们来谈谈如何使用 Maven Cargo 插件来设置服务器。

下面是 Maven 配置文件的代码，它提供了 WildFly 服务器:

```
<profile>
    <id>wildfly-standalone</id>
    <build>
        <plugins>
            <plugin>
                <groupId>org.codehaus.cargo</groupId>
                <artifactId>cargo-maven2-plugin</artifactId>
                <version>${cargo-maven2-plugin.version</version>
                <configuration>
                    <container>
                        <containerId>wildfly10x</containerId>
                        <zipUrlInstaller>
                            <url>
                                http://download.jboss.org/
                                  wildfly/10.1.0.Final/
                                    wildfly-10.1.0.Final.zip
                            </url>
                        </zipUrlInstaller>
                    </container>
                    <configuration>
                        <properties>
                            <cargo.hostname>127.0.0.0</cargo.hostname>
                            <cargo.jboss.management-http.port>
                                9990
                            </cargo.jboss.management-http.port>
                            <cargo.servlet.users>
                                testUser:admin1234!
                            </cargo.servlet.users>
                        </properties>
                    </configuration>
                </configuration>
            </plugin>
        </plugins>
    </build>
</profile> 
```

我们使用插件直接从 WildFly 的网站下载`WildFly 10.1` zip。然后通过确保`hostname`是`127.0.0.1` 并将端口设置为 9990 来配置。

然后，我们使用`cargo.servlet.users` 属性创建一个测试用户，用户 id 为`testUser` ，密码为 `admin1234!.`

现在插件的配置已经完成，我们应该能够调用一个 Maven 目标，并下载、安装、启动服务器和部署应用程序。

为此，导航到`ejb-remote` 目录并运行以下命令:

```
mvn clean package cargo:run
```

当您第一次运行此命令时，它将下载 WildFly 10.1 zip 文件，提取它并执行安装，然后启动它。它还将添加上面讨论的测试用户。任何进一步的执行将不会再次下载该 zip 文件。

### 2.3。手动设置 WildFly

为了手动设置 WildFly，你必须自己从[wildfly.org](https://web.archive.org/web/20220815025944/http://wildfly.org/downloads/)网站下载安装 zip 文件。以下步骤是 WildFly 服务器设置过程的高级视图:

将文件内容下载并解压缩到要安装服务器的位置后，配置以下环境变量:

```
JBOSS_HOME=/Users/$USER/../wildfly.x.x.Final
JAVA_HOME=`/usr/libexec/java_home -v 1.8`
```

然后在`bin`目录中，对于基于 Linux 的操作系统运行`./standalone.sh`或者对于 Windows 运行`./standalone.bat`。

在这之后，您必须添加一个用户。该用户将用于连接到远程 EJB bean。要了解如何添加用户，您应该查看一下[“添加用户”文档](https://web.archive.org/web/20220815025944/https://docs.jboss.org/author/display/WFLY/add-user%20utility.html)。

有关详细的设置说明，请访问 WildFly 的[入门文档](https://web.archive.org/web/20220815025944/https://docs.jboss.org/author/display/WFLY/Getting%20Started%20Guide.html)。

通过设置两个概要文件，项目 POM 已经被配置为与 Cargo 插件和手动服务器配置一起工作。默认情况下，选择 Cargo 插件。然而，要将应用程序部署到已经安装、配置并运行的 Wildfly 服务器，请在`ejb-remote`目录中执行以下命令:

```
mvn clean install wildfly:deploy -Pwildfly-runtime
```

## 3。`Remote`vs`Local`

bean 的业务接口可以是`local`或`remote.`

只有当 `@Local`带注释的 bean 与进行调用的 bean 在同一个应用程序中时，才能访问它，也就是说，如果它们驻留在同一个`.ear`或`.war`中。

可以从不同的应用程序访问 `@Remote`带注释的 bean，即驻留在不同的`JVM`或应用服务器中的应用程序。

**在设计包含 EJB 的解决方案时，需要记住一些要点:**

*   当用`@Local`或`@Remote`声明 bean 时，由 `javax.ejb`包定义的 `java.io.Serializable`、`java.io.Externalizable`和接口总是被排除
*   如果 bean 类是远程的，那么所有实现的接口都是远程的
*   如果 bean 类不包含注释或者指定了`@Local` 注释，那么所有实现的接口都被认为是本地的
*   为不包含接口的 bean 显式定义的任何接口都必须声明为`@Local`
*   EJB 3.2 版本倾向于为需要显式定义本地和远程接口的情况提供更多的粒度

## 4。创造了`Remote` EJB

让我们首先创建 bean 的接口，并将其命名为`HelloWorld:`

```
@Remote
public interface HelloWorld {
    String getHelloWorld();
}
```

现在我们将实现上面的接口，并将具体实现命名为`HelloWorldBean:`

```
@Stateless(name = "HelloWorld")
public class HelloWorldBean implements HelloWorld {

    @Resource
    private SessionContext context;

    @Override
    public String getHelloWorld() {
        return "Welcome to EJB Tutorial!";
    }
}
```

注意类声明上的`@Stateless` 注释。它表示这个 bean 是一个无状态会话 bean。**这种 bean 没有任何关联的客户端状态**，但是它可以保留其实例状态，并且通常用于执行独立的操作。

`@Resource` 注释将会话上下文注入远程 bean。

**`SessionContext`接口提供对容器为会话 bean 实例**提供的运行时会话上下文的访问。在创建实例之后，容器将`SessionContext`接口传递给实例。会话上下文在其生存期内保持与该实例的关联。

EJB 容器通常会创建一个无状态 bean 的对象池，并使用这些对象来处理客户端请求。由于这种池机制，实例变量值不能保证在查找方法调用中得到维护。

## 5。远程设置

在这一节中，我们将讨论如何设置 Maven 在服务器上构建和运行应用程序。

我们一个一个来看插件。

### 5.1。EJB 插件

下面给出的 EJB 插件是用来打包 EJB 模块的。我们已经指定 EJB 版本为 3.2。

以下插件配置用于为 bean 设置目标 JAR:

```
<plugin>
    <artifactId>maven-ejb-plugin</artifactId>
    <version>2.4</version>
    <configuration>
        <ejbVersion>3.2</ejbVersion>
    </configuration>
</plugin>
```

### 5.2。部署远程 EJB

要在 WildFly 服务器中部署 bean，请确保该服务器已启动并正在运行。

然后，为了执行远程设置，我们需要对`ejb-remote`项目中的 pom 文件运行以下 Maven 命令:

```
mvn clean install 
```

那我们应该跑:

```
mvn wildfly:deploy
```

或者，我们可以作为一个`admin`用户从应用服务器`.` 的管理控制台手动部署它

## 6。客户端设置

在创建远程 bean 之后，我们应该通过创建一个客户机来测试已部署的 bean。

首先，让我们讨论客户机项目的 Maven 设置。

### 6.1。客户端 Maven 设置

为了启动 EJB3 客户端，我们需要添加以下依赖项:

```
<dependency>
    <groupId>org.wildfly</groupId>
    <artifactId>wildfly-ejb-client-bom</artifactId>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

我们依靠这个应用程序的 EJB 远程业务接口来运行客户端。因此，我们需要指定 EJB 客户端 JAR 依赖关系。我们在父 pom 中添加了以下内容:

```
<dependency>
    <groupId>com.baeldung.ejb</groupId>
    <artifactId>ejb-remote</artifactId>
    <type>ejb</type>
</dependency>
```

将`<type>` 指定为`ejb`。

### 6.2。访问远程 Bean

我们需要在`src/main/resources`下创建一个文件，并将其命名为 `jboss-ejb-client.properties` ，该文件将包含访问已部署 bean 所需的所有属性:

```
remote.connections=default
remote.connection.default.host=127.0.0.1
remote.connection.default.port=8080
remote.connection.default.connect.options.org.xnio.Options
  .SASL_POLICY_NOANONYMOUS = false
remote.connection.default.connect.options.org.xnio.Options
  .SASL_POLICY_NOPLAINTEXT = false
remote.connection.default.connect.options.org.xnio.Options
  .SASL_DISALLOWED_MECHANISMS = ${host.auth:JBOSS-LOCAL-USER}
remote.connection.default.username=testUser
remote.connection.default.password=admin1234! 
```

## 7。创建客户端

将访问和使用远程`HelloWorld` bean 的类已经在`EJBClient.java`中创建，它在`com.baeldung.ejb.client`包中。

### 7.1。远程 Bean URL

远程 bean 通过符合以下格式的 URL 定位:

```
ejb:${appName}/${moduleName}/${distinctName}/${beanName}!${viewClassName}
```

*   `${appName}`是部署的应用程序名称。这里我们没有使用任何 EAR 文件，而是使用了一个简单的 JAR 或 WAR 部署，所以应用程序名将为空
*   `${moduleName}`是我们早先为我们的部署设定的名称，所以是`ejb-remote`
*   `${distinctName}`是一个特定的名称，可以有选择地分配给部署在服务器上的部署。如果一个部署不使用`distinct-name`，那么我们可以在 JNDI 名称中使用一个空字符串作为`distinct-name`，正如我们在示例中所做的那样
*   `${beanName}` 变量是 EJB 的实现类的简单名称，所以在我们的例子中它是`HelloWorld`
*   `${viewClassName}`表示远程接口的全限定接口名

### 7.2。查逻辑

接下来，让我们看看我们简单的查找逻辑:

```
public HelloWorld lookup() throws NamingException { 
    String appName = ""; 
    String moduleName = "remote"; 
    String distinctName = ""; 
    String beanName = "HelloWorld"; 
    String viewClassName = HelloWorld.class.getName();
    String toLookup = String.format("ejb:%s/%s/%s/%s!%s",
      appName, moduleName, distinctName, beanName, viewClassName);
    return (HelloWorld) context.lookup(toLookup);
}
```

为了连接到我们刚刚创建的`bean`,我们需要一个 URL，我们可以将它输入到上下文中。

### 7.3。`Initial Context`

我们现在将创建/初始化会话上下文:

```
public void createInitialContext() throws NamingException {
    Properties prop = new Properties();
    prop.put(Context.URL_PKG_PREFIXES, "org.jboss.ejb.client.naming");
    prop.put(Context.INITIAL_CONTEXT_FACTORY, 
      "org.jboss.naming.remote.client.InitialContextFacto[ERROR]
    prop.put(Context.PROVIDER_URL, "http-remoting://127.0.0.1:8080");
    prop.put(Context.SECURITY_PRINCIPAL, "testUser");
    prop.put(Context.SECURITY_CREDENTIALS, "admin1234!");
    prop.put("jboss.naming.client.ejb.context", false);
    context = new InitialContext(prop);
}
```

为了连接到远程 bean，我们需要一个 JNDI 上下文。上下文工厂由 Maven 工件`org.jboss:jboss-remote-naming`提供，这将创建一个 JNDI 上下文，它将把在`lookup`方法中构造的 URL 解析为远程应用服务器进程的代理。

### 7.4。定义查找参数

我们用参数`Context.INITIAL_CONTEXT_FACTORY.`定义工厂类

`Context.URL_PKG_PREFIXES`用于定义一个包来扫描附加的命名上下文。

参数`org.jboss.ejb.client.scoped.context = false`告诉上下文从提供的映射中读取连接参数(比如连接主机和端口),而不是从类路径配置文件中读取。如果我们想要创建一个应该能够连接到不同主机的 JAR 包，这就特别有用。

参数`Context.PROVIDER_URL`定义了连接模式，应该以`http-remoting://`开始。

## 8.测试

为了测试部署和检查设置，我们可以运行以下测试来确保一切正常工作:

```
@Test
public void testEJBClient() {
    EJBClient ejbClient = new EJBClient();
    HelloWorldBean bean = new HelloWorldBean();

    assertEquals(bean.getHelloWorld(), ejbClient.getEJBRemoteMessage());
}
```

随着测试的通过，我们现在可以确信一切都在按预期运行。

## 9。结论

因此，我们创建了一个 EJB 服务器和一个客户端，它调用远程 EJB 上的一个方法。通过适当地添加服务器的依赖项，该项目可以在任何应用服务器上运行。

整个项目可以在 GitHub 上找到[。](https://web.archive.org/web/20220815025944/https://github.com/eugenp/tutorials/tree/master/spring-ejb-modules)