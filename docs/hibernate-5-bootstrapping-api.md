# Hibernate 5 引导 API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-5-bootstrapping-api>

## 1。概述

在本教程中，我们将探索初始化和启动 Hibernate `SessionFactory.`的新机制，我们将特别关注新的本机引导过程，因为它在 5.0 版本中被重新设计。

在 5.0 版本之前，应用程序必须使用`Configuration`类来引导`SessionFactory.`。这种方法现在已被否决，因为 Hibernate 文档推荐使用基于`ServiceRegistry.`的新 API

简而言之，**构建一个`SessionFactory`就是拥有一个`ServiceRegistry`实现，它包含 Hibernate** 在启动和运行时所需的`Services`。

## 2。Maven 依赖关系

在我们开始探索新的引导过程之前，我们需要将`hibernate-core` jar 文件添加到项目类路径中。在基于 Maven 的项目中，我们只需要在`pom.xml`文件中声明这个依赖关系:

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.4.0.Final</version>
</dependency>
```

由于 Hibernate 是一个 JPA 提供者，这也将包括 JPA API 依赖关系。

我们还需要正在使用的数据库的 JDBC 驱动程序。在本例中，我们将使用嵌入式 H2 数据库:

```java
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.197</version>
</dependency>
```

可以在 Maven Central `.`上随意查看`[hibernate-core](https://web.archive.org/web/20221108062316/https://search.maven.org/search?q=g:org.hibernate%20AND%20a:hibernate-core&core=gav)`和 [`H2 driver`](https://web.archive.org/web/20221108062316/https://search.maven.org/search?q=g:com.h2database%20AND%20a:h2&core=gav) 的最新版本

## 3。引导 API

自举是指构建和初始化一个`SessionFactory.`的过程

**为了达到这个目的，我们需要有一个`ServiceRegistry`来保存 Hibernate 需要的`Services`。从这个注册表中，我们可以构建一个代表应用程序的域模型及其到数据库**的映射的`Metadata`对象。

让我们更详细地探索这些主要物体。

### 3.1。`Service`

在我们深入探讨`ServiceRegistry`的概念之前，我们首先需要了解什么是`Service` 在 Hibernate 5.0 中`.`，一个`Service`是一种由同名接口表示的功能类型:

```java
org.hibernate.service.Service
```

默认情况下，Hibernate 为最常见的`Services`提供了实现，它们在大多数情况下已经足够了。否则，我们可以构建自己的`Services`来修改原有的 Hibernate 功能或添加新的功能。

在下一小节中，我们将展示 Hibernate 如何通过一个名为`ServiceRegistry.`的轻量级容器使这些`Services`可用

### 3.2。`ServiceRegistry`

构建`SessionFactory`的第一步是创建一个`ServiceRegistry.`,它允许保存各种提供 Hibernate 所需功能的`Services`,并基于 [Java SPI](/web/20221108062316/https://www.baeldung.com/java-spi) 功能。

从技术上来说，我们可以把`ServiceRegistry`看作是一个轻量级的依赖注入工具，其中 beans 的类型只有`Service.`

有两种类型的`ServiceRegistry`并且它们是分层的`.` **第一种是`BootstrapServiceRegistry`，它没有父节点并且拥有这三个必需的服务**:

*   `ClassLoaderService:`允许 Hibernate 与各种运行时环境的`ClassLoader`交互
*   `IntegratorService:`控制`Integrator`服务的发现和管理，允许第三方应用程序与 Hibernate 集成
*   `StrategySelector:`解决各种战略契约的实施

**为了构建一个`BootstrapServiceRegistry`实现，我们使用了`BootstrapServiceRegistryBuilder `工厂类**，它允许以类型安全的方式定制这三个服务:

```java
BootstrapServiceRegistry bootstrapServiceRegistry = new BootstrapServiceRegistryBuilder()
  .applyClassLoader()
  .applyIntegrator()
  .applyStrategySelector()
  .build();
```

**第二个`ServiceRegistry`是`StandardServiceRegistry`，它建立在前面的`BootstrapServiceRegistry` 的基础上，容纳了前面提到的三个`Services`**`.`另外，它还包含了 Hibernate 需要的其他各种`Services`，列在 [`StandardServiceInitiators`](https://web.archive.org/web/20221108062316/https://github.com/hibernate/hibernate-orm/blob/master/hibernate-core/src/main/java/org/hibernate/service/StandardServiceInitiators.java) 类中。

像前面的注册表一样，我们使用`StandardServiceRegistryBuilder `来创建`StandardServiceRegistry:`的一个实例

```java
StandardServiceRegistryBuilder standardServiceRegistry =
  new StandardServiceRegistryBuilder();
```

在幕后，`StandardServiceRegistryBuilder`创建并使用`BootstrapServiceRegistry.`的一个实例，我们也可以使用一个重载的构造函数来传递一个已经创建的实例:

```java
BootstrapServiceRegistry bootstrapServiceRegistry = 
  new BootstrapServiceRegistryBuilder().build();
StandardServiceRegistryBuilder standardServiceRegistryBuilder = 
  new StandardServiceRegistryBuilder(bootstrapServiceRegistry);
```

我们使用这个构建器从一个资源文件中加载一个配置，比如默认的`hibernate.cfg.xml`，最后，我们调用`build()`方法来获得`StandardServiceRegistry.`的一个实例

```java
StandardServiceRegistry standardServiceRegistry = standardServiceRegistryBuilder
  .configure()
  .build();
```

### 3.3。`Metadata`

配置好实例化`BootstrapServiceRegistry or StandardServiceRegistry, ` **类型的`ServiceRegistry` 所需的所有`Services`之后，我们现在需要提供应用程序的域模型及其数据库映射的表示。**

`MetadataSources`类对此负责:

```java
MetadataSources metadataSources = new MetadataSources(standardServiceRegistry);
metadataSources.addAnnotatedClass();
metadataSources.addResource()
```

接下来，我们得到一个`Metadata`的实例，我们将在最后一步中使用它:

```java
Metadata metadata = metadataSources.buildMetadata();
```

### 3.4。`SessionFactory`

最后一步是从先前创建的`Metadata:`创建`SessionFactory`

```java
SessionFactory sessionFactory = metadata.buildSessionFactory();
```

我们现在可以打开一个`Session`并开始保存和读取实体:

```java
Session session = sessionFactory.openSession();
Movie movie = new Movie(100L);
session.persist(movie);
session.createQuery("FROM Movie").list();
```

## 4。结论

在本文中，我们探索了构建一个`SessionFactory.`所需的步骤。虽然这个过程看起来很复杂，但我们可以将其总结为三个主要步骤:首先创建一个`StandardServiceRegistry`的实例，然后构建一个`Metadata`对象，最后构建`SessionFactory.`

这些例子的完整代码可以在 Github 上找到[。](https://web.archive.org/web/20221108062316/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate5)