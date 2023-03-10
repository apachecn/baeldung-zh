# 如何实现一个 quartus 扩展

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/quarkus-extension-java>

## 1。概述

Quarkus 是由一个核心和一组扩展组成的框架。核心基于上下文和依赖注入(CDI ),扩展通常通过将主要组件公开为 CDI beans 来集成第三方框架。

在本教程中，假设对 [Quarkus](/web/20221121100642/https://www.baeldung.com/quarkus-io) 有基本的了解，我们将关注如何编写一个 Quarkus 扩展。

## 2。什么是 Quakus 扩展

Quarkus 扩展只是一个可以在 Quarkus 应用程序上运行的模块。Quarkus 应用程序本身是一个核心模块，带有一组其他扩展。

这种扩展最常见的用例是在 Quarkus 应用程序上运行第三方框架。

## 3。在普通 Java 应用程序中运行 liqui base

让我们尝试实现一个扩展来集成 [Liquibase，](https://web.archive.org/web/20221121100642/https://www.liquibase.org/)一个数据库变更管理工具。

但是在我们开始之前，我们首先需要展示如何从 Java main 方法运行 Liquibase 迁移。这将极大地方便扩展的实现。

Liquibase 框架的入口点是 Liquibase API。要使用它，我们需要一个 changelog 文件、一个用于访问该文件的`ClassLoader`和一个用于底层数据库的`Connection`:

```java
Connection c = DriverManager.getConnection("jdbc:h2:mem:testdb", "user", "password");
ResourceAccessor resourceAccessor = new ClassLoaderResourceAccessor();
String changLogFile = "db/liquibase-changelog-master.xml";
Liquibase liquibase = new Liquibase(changLogFile, resourceAccessor, new JdbcConnection(c));
```

有了这个实例，我们只需调用`update()`方法来更新数据库以匹配 changelog 文件。

```java
liquibase.update(new Contexts());
```

**目标是将 Liquibase 公开为 Quarkus 扩展**。也就是说，通过 Quarkus Configuration 提供数据库配置和 changelog 文件，然后将 Liquibase API 生成为 CDI bean。这提供了一种记录迁移调用的方法，以便以后执行。

## 4。如何写一个 Quarkus 扩展

从技术上讲，Quarkus 扩展是由两个模块组成的 Maven 多模块项目。第一个是运行时模块，我们在其中实现需求。第二个是部署模块，用于处理配置和生成运行时代码。

因此，让我们首先创建一个名为`quarkus-liquibase-parent`的 Maven 多模块项目，它包含两个子模块`runtime`和`deployment`:

```java
<modules>
    <module>runtime</module>
    <module>deployment</module>
</modules>
```

## 5.实现运行时模块

在运行时模块中，我们将实现:

*   用于捕获 Liquibase changelog 文件的配置类
*   用于公开 Liquibase API 的 CDI 生成器
*   以及充当记录调用调用的代理的记录器

### 5.1.Maven 依赖项和插件

运行时模块将依赖于`[quarkus-core](https://web.archive.org/web/20221121100642/https://search.maven.org/search?q=g:io.quarkus%20AND%20a:quarkus-core&core=gav)`模块，并最终依赖于所需扩展的运行时模块。这里，我们需要[`quarkus-agroal`依赖项](https://web.archive.org/web/20221121100642/https://search.maven.org/artifact/io.quarkus/quarkus-agroal)，因为我们的扩展需要一个`Datasource.`，我们还将在这里包含 [Liquibase 库](https://web.archive.org/web/20221121100642/https://search.maven.org/search?q=g:org.liquibase%20AND%20a:liquibase-core&core=gav):

```java
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-core</artifactId>
    <version>${quarkus.version}</version>
</dependency>
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-agroal</artifactId>
    <version>${quarkus.version}</version>
</dependency>
<dependency>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-core</artifactId>
    <version>3.8.1</version>
</dependency>
```

还有，我们可能需要加上 [`quarkus-bootstrap-maven-plugin`](https://web.archive.org/web/20221121100642/https://search.maven.org/artifact/io.quarkus/quarkus-bootstrap-maven-plugin) 。这个插件通过调用`extension-descriptor`目标自动生成 Quarkus 扩展描述符。

或者，我们可以省略这个插件，手动生成描述符。

无论哪种方式，我们都可以找到位于`META-INF/quarkus-extension.properties`下的扩展描述符:

```java
deployment-artifact=com.baeldung.quarkus.liquibase\:deployment\:1.0-SNAPSHOT
```

### 5.2.公开配置

为了提供 changelog 文件，我们需要实现一个配置类:

```java
@ConfigRoot(name = "liquibase", phase = ConfigPhase.BUILD_AND_RUN_TIME_FIXED)
public final class LiquibaseConfig {
    @ConfigItem
    public String changeLog;
}
```

我们用`@ConfigRoot`注释类，用`@ConfigItem.`注释属性，因此，`changeLog`字段，即变更日志的 camel case 形式，将通过位于 Quarkus 应用程序类路径中的`application.properties`文件中的`quarkus.liquibase.change-log` 键提供:

```java
quarkus.liquibase.change-log=db/liquibase-changelog-master.xml
```

我们还可以注意到`ConfigRoot.phase`值，它指示何时解析`change-log`键。在这种情况下，`BUILD_AND_RUN_TIME_FIXED,`密钥在部署时被读取，并在运行时可供应用程序使用。

### 5.3.将 Liquibase API 作为 CDI Bean 公开

我们已经在上面看到了如何从 main 方法运行 Liquibase 迁移。

现在，我们将复制相同的代码，但是作为一个 CDI bean，我们将使用一个 CDI 生成器来实现这个目的:

```java
@Produces
public Liquibase produceLiquibase() throws Exception {
    ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
    ResourceAccessor resourceAccessor = new ClassLoaderResourceAccessor(classLoader);
    DatabaseConnection jdbcConnection = new JdbcConnection(dataSource.getConnection());
    Liquibase liquibase = new Liquibase(liquibaseConfig.changeLog, resourceAccessor, jdbcConnection);
    return liquibase;
}
```

### 5.4.记录字节码

在这一步，我们将编写一个记录器类，作为记录字节码和设置运行时逻辑的代理:

```java
@Recorder
public class LiquibaseRecorder {

    public BeanContainerListener setLiquibaseConfig(LiquibaseConfig liquibaseConfig) {
        return beanContainer -> {
            LiquibaseProducer producer = beanContainer.instance(LiquibaseProducer.class);
            producer.setLiquibaseConfig(liquibaseConfig);
        };
    }

    public void migrate(BeanContainer container) throws LiquibaseException {
        Liquibase liquibase = container.instance(Liquibase.class);
        liquibase.update(new Contexts());
    }

}
```

这里，我们必须记录两次调用。`setLiquibaseConfig`用于设置配置，`migrate`用于执行迁移。接下来，我们将查看部署构建步骤处理器如何调用这些记录器方法，我们将在部署模块中实现这些方法。

**注意，当我们在构建时调用这些记录器方法时，指令不会被执行，而是被记录下来供以后在启动时执行。**

## 6.实现部署模块

**夸尔库斯扩展中的核心组件是`Build Step Processors`** 。它们是标注为`@BuildStep`的方法，通过记录器生成字节码，在构建期间通过 Quarkus 应用程序中配置的`quarkus-maven-plugin` 的构建目标来执行。

`@BuildStep`由于`BuildItem`而被订购。它们消耗由早期构建步骤生成的构建项目，并为后面的构建步骤生成构建项目。

应用程序部署模块中所有有序构建步骤生成的代码实际上是运行时代码。

### 6.1.Maven 依赖性

部署模块应该依赖于相应的运行时模块，并最终依赖于所需扩展的部署模块:

```java
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-core-deployment</artifactId>
    <version>${quarkus.version}</version>
</dependency>
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-arc-deployment</artifactId>
    <version>${quarkus.version}</version>
</dependency>
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-agroal-deployment</artifactId>
    <version>${quarkus.version}</version>
</dependency>

<dependency>
    <groupId>com.baeldung.quarkus.liquibase</groupId>
    <artifactId>runtime</artifactId>
    <version>${project.version}</version>
</dependency>
```

Quarkus 扩展的最新稳定版本与运行时模块相同。

### 6.2.实现构建步骤处理器

现在，让我们实现两个构建步骤处理器来记录字节码。第一个构建步骤处理器是`build()`方法，它将记录在静态 init 方法中执行的字节码。我们通过`STATIC_INIT`值对此进行配置:

```java
@Record(ExecutionTime.STATIC_INIT)
@BuildStep
void build(BuildProducer<AdditionalBeanBuildItem> additionalBeanProducer,
  BuildProducer<FeatureBuildItem> featureProducer,
  LiquibaseRecorder recorder,
  BuildProducer<BeanContainerListenerBuildItem> containerListenerProducer,
  DataSourceInitializedBuildItem dataSourceInitializedBuildItem) {

    featureProducer.produce(new FeatureBuildItem("liquibase"));

    AdditionalBeanBuildItem beanBuilItem = AdditionalBeanBuildItem.unremovableOf(LiquibaseProducer.class);
    additionalBeanProducer.produce(beanBuilItem);

    containerListenerProducer.produce(
      new BeanContainerListenerBuildItem(recorder.setLiquibaseConfig(liquibaseConfig)));
}
```

首先，我们创建一个`FeatureBuildItem`来标记扩展的类型或名称。然后，我们创建一个`AdditionalBeanBuildItem` ，这样`LiquibaseProducer` bean 将可用于 Quarkus 容器`.`

最后，我们创建一个`BeanContainerListenerBuildItem`，以便在 Quarkus `BeanContainer`启动后触发`BeanContainerListener`。这里，在监听器中，我们将配置传递给 Liquibase bean。

反过来，`processMigration(),`将记录在 main 方法中执行的调用，因为它是使用用于记录的`RUNTIME_INIT`参数配置的。

```java
@Record(ExecutionTime.RUNTIME_INIT)
@BuildStep
void processMigration(LiquibaseRecorder recorder, 
  BeanContainerBuildItem beanContainer) throws LiquibaseException {
    recorder.migrate(beanContainer.getValue());
}
```

这里，在这个处理器中，我们只是调用了`migrate()` recorder 方法，该方法又记录了`update()` Liquibase 方法，供以后执行。

## 7.测试 Liquibase 扩展

为了测试我们的扩展，我们将首先使用`quarkus-maven-plugin`创建一个 Quarkus 应用程序:

```java
mvn io.quarkus:quarkus-maven-plugin:1.0.0.CR1:create\
-DprojectGroupId=com.baeldung.quarkus.app\
-DprojectArtifactId=quarkus-app
```

接下来，除了对应于底层数据库的 Quarkus JDBC 扩展之外，我们将添加我们的扩展作为依赖项:

```java
<dependency>
    <groupId>com.baeldung.quarkus.liquibase</groupId>
    <artifactId>runtime</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-jdbc-h2</artifactId>
    <version>1.0.0.CR1</version>
</dependency>
```

接下来，我们需要在 pom 文件中有 [`quarkus-maven-plugin`](https://web.archive.org/web/20221121100642/https://search.maven.org/artifact/io.quarkus/quarkus-maven-plugin) :

```java
<plugin>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-maven-plugin</artifactId>
    <version>${quarkus.version}</version>
    <executions>
        <execution>
            <goals>
                <goal>build</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

这对于使用`dev`目标运行应用程序或者使用`build`目标构建可执行文件特别有用。

接下来，我们将通过位于`src/main/resources`中的`application.properties`文件提供数据源配置:

```java
quarkus.datasource.url=jdbc:h2:mem:testdb
quarkus.datasource.driver=org.h2.Driver
quarkus.datasource.username=user
quarkus.datasource.password=password
```

接下来，我们将为我们的 changelog 文件提供 changelog 配置:

```java
quarkus.liquibase.change-log=db/liquibase-changelog-master.xml
```

最后，我们可以在开发模式下启动应用程序:

```java
mvn compile quarkus:dev
```

或者在生产模式下:

```java
mvn clean package
java -jar target/quarkus-app-1.0-SNAPSHOT-runner.jar
```

## 8.结论

在本文中，我们实现了一个 Quarkus 扩展。例如，我们展示了如何让 Liquibase 在 Quarkus 应用程序上运行。

GitHub 上的[提供了完整的源代码。](https://web.archive.org/web/20221121100642/https://github.com/eugenp/tutorials/tree/master/quarkus-modules/quarkus-extension)