# CDI 便携式扩展和飞行路线

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cdi-portable-extension>

## 1。概述

在本教程中，我们将回顾 CDI(上下文和依赖注入)的一个有趣特性，称为 CDI 可移植扩展。

首先，我们将从理解它是如何工作的开始，然后我们将看到如何编写一个扩展。我们将完成为 Flyway 实现 CDI 集成模块的步骤，这样我们就可以在启动 CDI 容器时运行数据库迁移。

本教程假设您对 CDI 有基本的了解。请看一下[这篇文章](/web/20220523232107/https://www.baeldung.com/java-ee-cdi)对 CDI 的介绍。

## 2.什么是 CDI 可移植扩展？

CDI 可移植扩展是一种机制，通过它我们可以在 CDI 容器之上实现额外的功能。在引导时，CDI 容器扫描类路径并创建关于所发现的类的元数据。

在这个扫描过程中，CDI 容器触发了许多初始化事件，这些事件只能被扩展观察到。这就是 CDI 可移植扩展发挥作用的地方。

CDI 可移植扩展观察这些事件，然后修改或添加信息到由容器创建的元数据中。

## 3。Maven 依赖关系

让我们从在`pom.xml`中添加 CDI API 所需的依赖项开始。这足以实现一个空的扩展。

```java
<dependency>
    <groupId>javax.enterprise</groupId>
    <artifactId>cdi-api</artifactId>
    <version>2.0.SP1</version>
</dependency>
```

为了运行应用程序，我们可以使用任何兼容的 CDI 实现。在本文中，我们将使用 Weld 实现。

```java
<dependency>
    <groupId>org.jboss.weld.se</groupId>
    <artifactId>weld-se-core</artifactId>
    <version>3.0.5.Final</version>
    <scope>runtime</scope>
</dependency>
```

你可以在 Maven Central 上查看是否有新版本的 API 和 T2 实现已经发布。

## 4。在非 CDI 环境中运行 fly way

在我们开始集成`Flyway`和 CDI 之前，我们应该先看看如何在非 CDI 环境中运行它。

让我们来看看下面这个取自`Flyway` `:`的[官方网站的例子](https://web.archive.org/web/20220523232107/https://flywaydb.org/)

```java
DataSource dataSource = //...
Flyway flyway = new Flyway();
flyway.setDataSource(dataSource);
flyway.migrate();
```

正如我们所看到的，我们只使用了一个需要一个`DataSource`实例的`Flyway`实例。

我们的 CDI 可移植扩展稍后将生成`Flyway`和`Datasource`bean。出于这个示例的目的，我们将使用一个嵌入式 H2 数据库，并通过`DataSourceDefinition`注释提供`DataSource`属性。

## 5。CDI 容器初始化事件

在应用程序引导中，CDI 容器从加载和实例化所有 CDI 可移植扩展开始。然后，在每个扩展中，它搜索并注册初始化事件的观察器方法(如果有的话)。之后，它执行以下步骤:

1.  在扫描过程开始前触发`BeforeBeanDiscovery`事件
2.  执行类型发现，扫描归档 beans，并为每个发现的类型触发`ProcessAnnotatedType`事件
3.  激发`AfterTypeDiscovery`事件
4.  执行 bean 发现
5.  激发`AfterBeanDiscovery `事件
6.  执行 bean 验证并检测定义错误
7.  激发`AfterDeploymentValidation` 事件

CDI 可移植扩展的目的是观察这些事件，检查关于发现的 beans 的元数据，修改这些元数据或向其中添加内容。

在 CDI 可移植扩展中，我们只能观察这些事件。

## 6。编写 CDI 可移植扩展

让我们看看如何通过构建我们自己的 CDI 可移植扩展来挂钩这些事件。

### 6.1.实现 SPI 提供程序

一个 CDI 可移植扩展是 Java SPI 接口的提供者`javax.enterprise.inject.spi.Extension.`看看[这篇文章](/web/20220523232107/https://www.baeldung.com/java-spi)对 Java SPI 的介绍。

首先，我们从提供`Extension`实现开始。稍后，我们将向 CDI 容器引导事件添加观察者方法:

```java
public class FlywayExtension implements Extension {
}
```

然后，我们添加一个文件名`META-INF/services/javax.enterprise.inject.spi.Extension`,内容如下:

```java
com.baeldung.cdi.extension.FlywayExtension
```

作为 SPI，这个`Extension`在容器引导之前加载。所以可以注册 CDI 引导事件上的 observer 方法。

### 6.2。定义初始化事件的观察者方法

在这个例子中，我们在扫描过程开始之前让 CDI 容器知道`Flyway`类。这是在`registerFlywayType()`观察器方法中完成的:

```java
public void registerFlywayType(
  @Observes BeforeBeanDiscovery bbdEvent) {
    bbdEvent.addAnnotatedType(
      Flyway.class, Flyway.class.getName());
}
```

这里，我们添加了关于`Flyway`类的元数据。**从现在开始，它会表现得好像被集装箱扫描了一样。**为此，我们使用了`addAnnotatedType()`方法。

接下来，我们将观察`ProcessAnnotatedType`事件，使`Flyway`类成为 CDI 管理的 bean:

```java
public void processAnnotatedType(@Observes ProcessAnnotatedType<Flyway> patEvent) {
    patEvent.configureAnnotatedType()
      .add(ApplicationScoped.Literal.INSTANCE)
      .add(new AnnotationLiteral<FlywayType>() {})
      .filterMethods(annotatedMethod -> {
          return annotatedMethod.getParameters().size() == 1
            && annotatedMethod.getParameters().get(0).getBaseType()
              .equals(javax.sql.DataSource.class);
      }).findFirst().get().add(InjectLiteral.INSTANCE);
}
```

首先，我们用`@ApplicationScoped`和`@FlywayType`注释来注释`Flyway`类，然后我们搜索`Flyway.setDataSource(DataSource dataSource)`方法并用`@Inject.`来注释它

上述操作的最终结果与容器扫描下面的`Flyway` bean 具有相同的效果:

```java
@ApplicationScoped
@FlywayType
public class Flyway {

    //...
    @Inject
    public void setDataSource(DataSource dataSource) {
      //...
    }
}
```

下一步是使`DataSource` bean 可用于注入，因为我们的`Flyway` bean 依赖于`DataSource` bean。

为此，我们将在容器中注册一个`DataSource` Bean，并使用`AfterBeanDiscovery`事件:

```java
void afterBeanDiscovery(@Observes AfterBeanDiscovery abdEvent, BeanManager bm) {
    abdEvent.addBean()
      .types(javax.sql.DataSource.class, DataSource.class)
      .qualifiers(new AnnotationLiteral<Default>() {}, new AnnotationLiteral<Any>() {})
      .scope(ApplicationScoped.class)
      .name(DataSource.class.getName())
      .beanClass(DataSource.class)
      .createWith(creationalContext -> {
          DataSource instance = new DataSource();
          instance.setUrl(dataSourceDefinition.url());
          instance.setDriverClassName(dataSourceDefinition.className());
              return instance;
      });
}
```

正如我们所见，我们需要一个`DataSourceDefinition`来提供数据源属性。

我们可以使用以下注释来注释任何受管 bean:

```java
@DataSourceDefinition(
  name = "ds", 
  className = "org.h2.Driver", 
  url = "jdbc:h2:mem:testdb")
```

为了提取这些属性，我们观察`ProcessAnnotatedType`事件和`@WithAnnotations`注释:

```java
public void detectDataSourceDefinition(
  @Observes @WithAnnotations(DataSourceDefinition.class) ProcessAnnotatedType<?> patEvent) {
    AnnotatedType at = patEvent.getAnnotatedType();
    dataSourceDefinition = at.getAnnotation(DataSourceDefinition.class);
}
```

最后，我们监听`AfterDeployementValidation`事件，从 CDI 容器中获取想要的`Flyway` bean，然后调用`migrate()`方法:

```java
void runFlywayMigration(
  @Observes AfterDeploymentValidation adv, 
  BeanManager manager) {
    Flyway flyway = manager.createInstance()
      .select(Flyway.class, new AnnotationLiteral<FlywayType>() {}).get();
    flyway.migrate();
}
```

## 7.结论

第一次构建 CDI 可移植扩展似乎很困难，但是一旦我们理解了容器初始化生命周期和专用于扩展的 SPI，它就成为一个非常强大的工具，我们可以使用它在 Jakarta EE 之上构建框架。

像往常一样，本文中显示的所有代码示例都可以在 GitHub 上找到[。](https://web.archive.org/web/20220523232107/https://github.com/eugenp/tutorials/tree/master/flyway-cdi-extension)