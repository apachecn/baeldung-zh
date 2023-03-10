# OSGi 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/osgi>

## 1。简介

一些 Java 关键任务和中间件应用程序有一些硬技术需求。

有些必须支持热部署，以便不中断正在运行的服务，而有些必须能够使用同一软件包的不同版本，以便支持外部遗留系统。

平台代表了支持这种需求的可行解决方案。

**`Open Service Gateway Initiative`是定义基于 Java 的组件系统的规范。**目前由`[OSGi Alliance](https://web.archive.org/web/20220628123428/https://www.osgi.org/)`管理，其第一个版本可追溯到 1999 年。

从那以后，它被证明是组件系统的一个很好的标准，并且现在被广泛使用。例如，`Eclipse IDE`是基于`OSGi`的应用程序。

在本文中，我们将探索利用由`Apache`提供的实现的`OSGi`的一些基本特性。

## 2。OSGi 基础知识

在 OSGi，单个组件被称为捆绑包。

从逻辑上来说，**bundle 是一个具有独立生命周期的功能—**,这意味着它可以独立地启动、停止和删除。

从技术上讲，bundle 只是一个 jar 文件，带有一个包含一些特定于 OSGi 的头的`MANIFEST.MF`文件。

`OSGi`平台提供了一种方式来接收关于软件包变得可用或者当它们从平台上移除时的通知。这将允许适当设计的客户端继续工作，即使其依赖的服务暂时不可用，功能可能会下降。

因此，一个捆绑包必须明确声明它需要访问哪些包，只有在捆绑包本身或已经安装在平台上的其他捆绑包中的依赖项可用时，`OSGi`平台才会启动它。

## 3。获取工具

我们将从[此链接](https://web.archive.org/web/20220628123428/https://www.apache.org/dyn/closer.lua/karaf/4.1.3/apache-karaf-4.1.3.zip)下载`Apache Karaf`的最新版本，开始我们在`OSGi`的旅程。`Apache Karaf`是一个运行基于`OSGi-`的应用的平台；它基于被称为`[Apache Felix](https://web.archive.org/web/20220628123428/https://felix.apache.org/)`的`Apache`对`OSGi`规范的实现。

`Karaf`在`Felix`的基础上提供了一些方便的功能，这将帮助我们熟悉`OSGi`，例如，一个命令行界面将允许我们与平台进行交互。

要安装`Karaf`，您可以遵循[官方文档](https://web.archive.org/web/20220628123428/https://karaf.apache.org/manual/latest/#_quick_start)中的安装说明。

## 4.捆绑入口点

要在 OSGi 环境中执行应用程序，我们必须将其打包成一个`OSGi`包，并定义应用程序入口点，这不是通常的`public static void main(String[] args)`方法。

**所以，让我们从构建一个基于`OSGi-`的“Hello World”应用程序开始。**

我们开始建立对核心`OSGi API`的简单依赖:

```java
<dependency>
    <groupId>org.osgi</groupId> 
    <artifactId>org.osgi.core</artifactId>
    <version>6.0.0</version>
    <scope>provided</scope>
</dependency>
```

依赖项被声明为`provided`，因为它将在`OSGi`运行时可用，并且包不需要嵌入它。

现在让我们编写简单的`HelloWorld`类:

```java
public class HelloWorld implements BundleActivator {
    public void start(BundleContext ctx) {
        System.out.println("Hello world.");
    }
    public void stop(BundleContext bundleContext) {
        System.out.println("Goodbye world.");
    }
}
```

**`BundleActivator`是由`OSGi`提供的接口，它必须由作为包入口点的类来实现。**

当包含这个类的包启动时，`OSGi`平台调用`start()`方法。另一方面，`stop()`在束停止之前被调用。

让我们记住，每个包最多可以包含一个`BundleActivator`。提供给这两个方法的`BundleContext`对象允许与`OSGi`运行时交互。我们很快会回来的。

## 5。构建捆绑包

让我们修改`pom.xml`并使它成为一个真正的 OSGi 包。

首先，我们必须明确声明我们将构建一个包，而不是一个 jar:

```java
<packaging>bundle</packaging>
```

然后我们利用`Apache Felix`社区提供的`maven-bundle-plugin,` ，将`HelloWorld`类打包成一个`OSGi`包:

```java
<plugin>
    <groupId>org.apache.felix</groupId>
    <artifactId>maven-bundle-plugin</artifactId>
    <version>3.3.0</version>
    <extensions>true</extensions>
    <configuration>
        <instructions>
            <Bundle-SymbolicName>
                ${pom.groupId}.${pom.artifactId}
            </Bundle-SymbolicName>
            <Bundle-Name>${pom.name}</Bundle-Name>
            <Bundle-Version>${pom.version}</Bundle-Version>
            <Bundle-Activator>
                com.baeldung.osgi.sample.activator.HelloWorld
            </Bundle-Activator>
            <Private-Package>
                com.baeldung.osgi.sample.activator
            </Private-Package>            
        </instructions>
    </configuration>
</plugin>
```

在 instructions 部分，我们指定了希望包含在包的清单文件中的`OSGi`头的值。

`Bundle-Activator`是将用于启动和停止包的`BundleActivator`实现的完全限定名，它指的是我们刚刚编写的类。

`Private-Package`不是一个 OSGi 头，但是它用来告诉插件把这个包包含在包中，但是不让其他插件使用。我们现在可以用通常的命令`mvn clean install`构建这个包。

## 6。安装和运行捆绑包

让我们通过执行命令来启动`Karaf`:

```java
<KARAF_HOME>/bin/karaf start
```

其中`<KARAF_HOME>`是安装`Karaf`的文件夹。当`Karaf`控制台的提示符出现时，我们可以执行以下命令来安装软件包:

```java
> bundle:install mvn:com.baeldung/osgi-intro-sample-activator/1.0-SNAPSHOT
Bundle ID: 63
```

这指示 Karaf 从本地 Maven 仓库加载包。

作为回报，Karaf 打印出分配给捆绑包的数字 ID ，这取决于已经安装的捆绑包的数量，可能会有所不同。软件包现在刚刚安装好，我们现在可以使用以下命令启动它:

```java
> bundle:start 63
Hello World
```

一启动捆绑包，就会立即出现“Hello World”。我们现在可以使用以下命令停止并卸载软件包:

```java
> bundle:stop 63
> bundle:uninstall 63
```

“再见世界”出现在控制台上，对应于`stop()`方法中的代码。

## 7 .**。OSGi 服务**

让我们继续编写一个简单的`OSGi`服务，一个公开问候人们的方法的接口:

```java
package com.baeldung.osgi.sample.service.definition;
public interface Greeter {
    public String sayHiTo(String name);
}
```

让我们编写一个也是`BundleActivator`的实现，这样我们就能够实例化服务，并在捆绑包启动时在平台上注册它:

```java
package com.baeldung.osgi.sample.service.implementation;
public class GreeterImpl implements Greeter, BundleActivator {

    private ServiceReference<Greeter> reference;
    private ServiceRegistration<Greeter> registration;

    @Override
    public String sayHiTo(String name) {
        return "Hello " + name;
    }

    @Override 
    public void start(BundleContext context) throws Exception {
        System.out.println("Registering service.");
        registration = context.registerService(
          Greeter.class, 
          new GreeterImpl(), 
          new Hashtable<String, String>());
        reference = registration
          .getReference();
    }

    @Override 
    public void stop(BundleContext context) throws Exception {
        System.out.println("Unregistering service.");
        registration.unregister();
    }
}
```

**我们使用`BundleContext`作为请求`OSGi`平台注册服务新实例的手段。**

我们还应该提供服务的类型和可能的配置参数的映射，这在我们的简单场景中是不需要的。现在让我们继续配置`maven-bundle-plugin`:

```java
<plugin>
    <groupId>org.apache.felix</groupId>
    <artifactId>maven-bundle-plugin</artifactId>
    <extensions>true</extensions>
    <configuration>
        <instructions>
            <Bundle-SymbolicName>
                ${project.groupId}.${project.artifactId}
            </Bundle-SymbolicName>
            <Bundle-Name>
                ${project.artifactId}
            </Bundle-Name>
            <Bundle-Version>
                ${project.version}
            </Bundle-Version>
            <Bundle-Activator>
                com.baeldung.osgi.sample.service.implementation.GreeterImpl
            </Bundle-Activator>
            <Private-Package>
                com.baeldung.osgi.sample.service.implementation
            </Private-Package>
            <Export-Package>
                com.baeldung.osgi.sample.service.definition
            </Export-Package>
        </instructions>
    </configuration>
</plugin>
```

值得注意的是，这次只导出了`com.baeldung.osgi.sample.service.definition`包，通过`Export-Package`头。

得益于此，`OSGi`将允许其他包只调用服务接口中指定的方法。包`com.baeldung.osgi.sample.service.implementation`被标记为私有，所以没有其他包能够直接访问实现的成员。

## 8。一个 OSGi 客户

现在让我们编写客户端。它只是在启动时查找服务并调用它:

```java
public class Client implements BundleActivator, ServiceListener {
}
```

让我们实现`BundleActivator start()`方法:

```java
private BundleContext ctx;
private ServiceReference serviceReference;

public void start(BundleContext ctx) {
    this.ctx = ctx;
    try {
        ctx.addServiceListener(
          this, "(objectclass=" + Greeter.class.getName() + ")");
    } catch (InvalidSyntaxException ise) {
        ise.printStackTrace();
    }
}
```

`addServiceListener()`方法允许客户端请求平台发送关于符合所提供表达式的服务的通知。

该表达式使用类似于 LDAP 的语法，在我们的例子中，我们请求关于`Greeter`服务的通知。

让我们继续讨论回调方法:

```java
public void serviceChanged(ServiceEvent serviceEvent) {
    int type = serviceEvent.getType();
    switch (type){
        case(ServiceEvent.REGISTERED):
            System.out.println("Notification of service registered.");
            serviceReference = serviceEvent
              .getServiceReference();
            Greeter service = (Greeter)(ctx.getService(serviceReference));
            System.out.println( service.sayHiTo("John") );
            break;
        case(ServiceEvent.UNREGISTERING):
            System.out.println("Notification of service unregistered.");
            ctx.ungetService(serviceEvent.getServiceReference());
            break;
        default:
            break;
    }
}
```

当涉及到`Greeter`服务的一些修改发生时，该方法会得到通知。

当服务注册到平台时，我们获得对它的引用，我们将它存储在本地，然后使用它来获取服务对象并调用它。

当服务器后来被取消注册时，我们使用先前存储的引用来取消对它的访问，这意味着我们告诉平台我们将不再使用它。

我们现在只需要编写`stop()`方法:

```java
public void stop(BundleContext bundleContext) {
    if(serviceReference != null) {
        ctx.ungetService(serviceReference);
    }
}
```

这里，我们再次取消对服务的获取，以涵盖在服务停止之前客户端停止的情况。让我们最后看一下`pom.xml`中的依赖关系:

```java
<dependency>
    <groupId>com.baeldung</groupId>
    <artifactId>osgi-intro-sample-service</artifactId>
    <version>1.0-SNAPSHOT</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.osgi</groupId>
    <artifactId>org.osgi.core</artifactId>
    <version>6.0.0</version>
</dependency>
```

## 9。客户端和服务

现在，让我们通过以下操作在 Karaf 中安装客户端和服务包:

```java
> install mvn:com.baeldung/osgi-intro-sample-service/1.0-SNAPSHOT
Bundle ID: 64
> install mvn:com.baeldung/osgi-intro-sample-client/1.0-SNAPSHOT
Bundle ID: 65
```

请记住，分配给每个包的标识符编号可能会有所不同。

现在让我们开始客户端捆绑包:

```java
> start 65
```

因此，什么都不会发生，因为客户端是活动的，它正在等待服务，我们可以从:

```java
> start 64
Registering service.
Service registered.
Hello John
```

一旦服务的 BundleActivator 启动，服务就会注册到平台。这反过来通知客户机它所等待的服务是可用的。

然后，客户机获得对服务的引用，并使用它来调用通过服务包交付的实现。

## 10。结论

在本文中，我们通过一个简单的例子探索了 OSGi 的基本特征，这足以理解 OSGi 的潜力。

总之，每当我们必须保证单个应用程序的更新不会造成任何损害时，OSGi 是一个可行的解决方案。

这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220628123428/https://github.com/eugenp/tutorials/tree/master/osgi)