# 弹簧轮廓

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-profiles>

## 1。概述

在本教程中，我们将重点介绍 Spring 中的概要文件。

概要文件是框架的一个核心特性— **允许我们将 beans 映射到不同的概要文件** —例如，`dev`、`test`和`prod`。

然后，我们可以在不同的环境中激活不同的概要文件，只引导我们需要的 beans。

## 延伸阅读:

## [为测试配置单独的 Spring 数据源](/web/20221001115718/https://www.baeldung.com/spring-testing-separate-data-source)

A quick, practical tutorial on how to configure a separate data source for testing in a Spring application.[Read more](/web/20221001115718/https://www.baeldung.com/spring-testing-separate-data-source) →

## [弹簧和 Spring Boot 的特性](/web/20221001115718/https://www.baeldung.com/properties-with-spring)

Tutorial for how to work with properties files and property values in Spring.[Read more](/web/20221001115718/https://www.baeldung.com/properties-with-spring) →

## 2。在豆子上使用`@Profile`

让我们从简单开始，看看如何让一个 bean 属于一个特定的概要。**我们使用`@Profile`注释——我们将 bean 映射到那个特定的概要文件**；这个注释只是接受一个(或者多个)概要文件的名称。

考虑一个基本的场景:我们有一个 bean，它应该只在开发期间活动，而不在生产中部署。

我们用一个`dev`概要文件注释这个 bean，它将只在开发期间出现在容器中。在生产中，`dev`不会被激活:

```java
@Component
@Profile("dev")
public class DevDatasourceConfig
```

作为一个小提示，概要文件名称也可以以 NOT 操作符为前缀，例如`!dev`，以将它们从概要文件中排除。

在本例中，仅当`dev`配置文件未激活时，组件才被激活:

```java
@Component
@Profile("!dev")
public class DevDatasourceConfig
```

## 3。用 XML 声明概要文件

概要文件也可以用 XML 来配置。`<beans>`标签有一个`profile`属性，它采用逗号分隔的适用概要文件值:

```java
<beans profile="dev">
    <bean id="devDatasourceConfig" 
      class="org.baeldung.profiles.DevDatasourceConfig" />
</beans>
```

## 4。设置配置文件

下一步是激活和设置概要文件，以便在容器中注册相应的 beans。

这可以通过多种方式实现，我们将在接下来的小节中探讨这些方式。

### 4.1。通过`WebApplicationInitializer`接口编程

在 web 应用程序中， `WebApplicationInitializer` 可用于以编程方式配置`ServletContext` 。

这也是一个非常方便的以编程方式设置活动配置文件的位置:

```java
@Configuration
public class MyWebApplicationInitializer 
  implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {

        servletContext.setInitParameter(
          "spring.profiles.active", "dev");
    }
}
```

### 4.2。通过`ConfigurableEnvironment`编程

我们也可以直接在环境中设置配置文件:

```java
@Autowired
private ConfigurableEnvironment env;
...
env.setActiveProfiles("someProfile");
```

### 4.3。`web.xml`中的上下文参数

类似地，我们可以在 web 应用程序的`web.xml`文件中定义活动概要文件，使用一个上下文参数:

```java
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/app-config.xml</param-value>
</context-param>
<context-param>
    <param-name>spring.profiles.active</param-name>
    <param-value>dev</param-value>
</context-param>
```

### 4.4。JVM 系统参数

配置文件名也可以通过 JVM 系统参数传递。这些配置文件将在应用程序启动时激活:

```java
-Dspring.profiles.active=dev
```

### 4.5。环境变量

在 Unix 环境中，**配置文件也可以通过环境变量**激活:

```java
export spring_profiles_active=dev
```

### 4.6。Maven 简介

Spring 概要文件也可以通过 Maven 概要文件激活，通过**指定`spring.profiles.active `配置属性**。

在每个 Maven 概要文件中，我们可以设置一个`spring.profiles.active`属性:

```java
<profiles>
    <profile>
        <id>dev</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <spring.profiles.active>dev</spring.profiles.active>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <spring.profiles.active>prod</spring.profiles.active>
        </properties>
    </profile>
</profiles>
```

**它的值将被用来替换`application.properties` :** 中的`@[[email protected]](/web/20221001115718/https://www.baeldung.com/cdn-cgi/l/email-protection)`占位符

```java
[[email protected]](/web/20221001115718/https://www.baeldung.com/cdn-cgi/l/email-protection)@
```

现在我们需要在`pom.xml`中启用资源过滤:

```java
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
    ...
</build>
```

并附加一个`-P`参数来切换将应用哪个 Maven 概要文件:

```java
mvn clean package -Pprod
```

该命令将为`prod`配置文件打包应用程序。它还在运行时将`spring.profiles.active `值`prod`应用于该应用程序。

### 4.7。在测试中

通过使用`@ActiveProfile` 注释来启用特定的概要文件，测试可以非常容易地指定哪些概要文件是活动的:

```java
@ActiveProfiles("dev")
```

到目前为止，我们已经研究了多种激活个人资料的方法。现在让我们来看看哪一个比另一个优先级高，如果我们使用多个优先级，从高到低会发生什么:

1.  `web.xml`中的上下文参数
2.  `WebApplicationInitializer`
3.  JVM 系统参数
4.  环境变量
5.  Maven 简介

## 5。默认配置文件

任何没有指定概要文件的 bean 都属于`default`概要文件。

Spring 还提供了一种在没有其他概要文件处于活动状态时设置默认概要文件的方法——通过使用`spring.profiles.default`属性。

## 6。获取活动档案

Spring 的活动概要文件驱动了启用/禁用 beans 的`@Profile`注释的行为。然而，我们也可能希望以编程方式访问活动概要文件的列表。

我们有两种方法，**使用`Environment` 或** `**spring.profiles.active**` **。**

### 6.1.使用`Environment`

我们可以通过注入从`Environment`对象中访问活动概要文件:

```java
public class ProfileManager {
    @Autowired
    private Environment environment;

    public void getActiveProfiles() {
        for (String profileName : environment.getActiveProfiles()) {
            System.out.println("Currently active profile - " + profileName);
        }  
    }
}
```

### 6.2.使用`spring.profiles.active`

或者，我们可以通过注入属性`spring.profiles.active`来访问概要文件:

```java
@Value("${spring.profiles.active}")
private String activeProfile;
```

这里，我们的`activeProfile`变量**将包含当前激活的**配置文件的名称，如果有几个，它将包含用逗号分隔的名称。

然而，我们应该**考虑如果根本没有活动概要文件会发生什么。**在上面的代码中，缺少活动概要文件会阻止应用程序上下文的创建。由于缺少注入变量的占位符，这将导致一个`IllegalArgumentException`。

为了避免这种情况，我们可以**定义一个默认值**:

```java
@Value("${spring.profiles.active:}")
private String activeProfile;
```

现在，如果没有配置文件是活动的，我们的`activeProfile`将只包含一个空字符串。

如果我们想像前面的例子一样访问它们的列表，我们可以通过[拆分](/web/20221001115718/https://www.baeldung.com/java-split-string)变量`activeProfile`来实现:

```java
public class ProfileManager {
    @Value("${spring.profiles.active:}")
    private String activeProfiles;

    public String getActiveProfiles() {
        for (String profileName : activeProfiles.split(",")) {
            System.out.println("Currently active profile - " + profileName);
        }
    }
}
```

## 7。示例:使用概要文件分离数据源配置

既然基本的都已经说了，我们来看一个真实的例子。

考虑一个场景，其中**我们必须为开发和生产环境**维护数据源配置。

让我们创建一个需要由两个数据源实现来实现的公共接口`DatasourceConfig`:

```java
public interface DatasourceConfig {
    public void setup();
}
```

以下是开发环境的配置:

```java
@Component
@Profile("dev")
public class DevDatasourceConfig implements DatasourceConfig {
    @Override
    public void setup() {
        System.out.println("Setting up datasource for DEV environment. ");
    }
}
```

和生产环境的配置:

```java
@Component
@Profile("production")
public class ProductionDatasourceConfig implements DatasourceConfig {
    @Override
    public void setup() {
       System.out.println("Setting up datasource for PRODUCTION environment. ");
    }
}
```

现在让我们创建一个测试并注入我们的 DatasourceConfig 接口；根据活动配置文件，Spring 将注入`DevDatasourceConfig`或`ProductionDatasourceConfig` bean:

```java
public class SpringProfilesWithMavenPropertiesIntegrationTest {
    @Autowired
    DatasourceConfig datasourceConfig;

    public void setupDatasource() {
        datasourceConfig.setup();
    }
}
```

当`dev`概要文件被激活时，Spring 注入`DevDatasourceConfig`对象，当调用`setup()`方法时，输出如下:

```java
Setting up datasource for DEV environment.
```

## 8。Spring Boot 简介

Spring Boot 支持到目前为止概述的所有概要文件配置，还有一些额外的特性。

### 8.1.激活或设置配置文件

第 4 节中介绍的初始化参数`spring.profiles.active`也可以在 Spring Boot 中设置为一个属性，以定义当前活动的配置文件。这是一个标准属性，Spring Boot 会自动选择:

```java
spring.profiles.active=dev
```

然而，从 Spring Boot 2.4 开始，该属性不能与`spring.config.activate.on-profile`一起使用，因为这会引发一个`ConfigDataException ` ( `i.e.`、`InvalidConfigDataPropertyException` 或`InactiveConfigDataAccessException`)。

为了以编程方式设置概要文件，我们也可以使用`SpringApplication`类:

```java
SpringApplication.setAdditionalProfiles("dev");
```

要在 Spring Boot 使用 Maven 设置概要文件，我们可以在`pom.xm` `l`中的`spring-boot-maven-plugin`下指定概要文件名称:

```java
<plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
            <profiles>
                <profile>dev</profile>
            </profiles>
        </configuration>
    </plugin>
    ...
</plugins>
```

并执行 Spring Boot 特有的 Maven 目标:

```java
mvn spring-boot:run
```

### 8.2.特定于配置文件的属性文件

然而，Spring Boot 带来的最重要的与概要文件相关的特性是**特定于概要文件的属性文件。**这些必须以`application-{profile}.properties`的格式命名。

Spring Boot 将自动加载所有概要文件的属性到一个`application.properties`文件中，并且只加载特定概要文件的属性到指定概要文件的`.properties`文件中。

例如，我们可以通过使用名为`application-dev.properties`和`application-production.properties`的两个文件为`dev`和`production`配置文件配置不同的数据源:

在`application-production.properties`文件中，我们可以设置一个`MySql`数据源:

```java
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/db
spring.datasource.username=root
spring.datasource.password=root
```

然后我们可以为`application-dev.properties`文件中的`dev`配置文件配置相同的属性，以使用内存中的`H2`数据库:

```java
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:mem:db;DB_CLOSE_DELAY=-1
spring.datasource.username=sa
spring.datasource.password=sa
```

这样，我们可以很容易地为不同的环境提供不同的配置。

在 Spring Boot 2.4 之前，可以从特定于配置文件的文档中激活配置文件。但现在情况不再是这样了；在以后的版本中，框架会在这些情况下抛出一个`InvalidConfigDataPropertyException` 或者一个`InactiveConfigDataAccessException`。

### 8.3.多文档文件

为了进一步简化为不同环境定义属性，我们甚至可以将所有属性放在同一个文件中，并使用分隔符来表示配置文件。

从 2.4 版本开始，除了以前支持的 [YAML](/web/20221001115718/https://www.baeldung.com/spring-yaml) 之外，Spring Boot 还扩展了对属性文件的多文档文件支持。所以现在，**我们可以在同一个`application.properties`** 中指定`dev`和`production`属性:

```java
my.prop=used-always-in-all-profiles
#---
spring.config.activate.on-profile=dev
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/db
spring.datasource.username=root
spring.datasource.password=root
#---
spring.config.activate.on-profile=production
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:mem:db;DB_CLOSE_DELAY=-1
spring.datasource.username=sa
spring.datasource.password=sa
```

Spring Boot 按照从上到下的顺序阅读这个文件。也就是说，如果某个属性，比如说`my.prop`，在上面的例子中再次出现在末尾，那么将考虑最末尾的值。

### 8.4.配置文件组

Boot 2.4 中添加的另一个功能是配置文件组。顾名思义，**它允许我们将相似的配置文件分组在一起**。

让我们考虑一个用例，在这个用例中，我们的生产环境有多个配置文件。比如说，`production`环境中的数据库用`proddb`，调度程序用`prodquartz`。

为了通过我们的`application.properties`文件一次启用这些概要文件，我们可以指定:

```java
spring.profiles.group.production=proddb,prodquartz
```

因此，激活`production`配置文件也会激活`proddb`和`prodquartz`。

## 9。结论

在本文中，我们讨论了如何**在 bean 上定义概要文件**，以及如何**在我们的应用程序中启用正确的概要文件**。

最后，我们用一个简单但真实的例子来验证我们对概要文件的理解。

本教程的实现可以在[GitHub 项目](https://web.archive.org/web/20221001115718/https://github.com/eugenp/tutorials/tree/master/spring-core-2 "The example project for using Profiles on Github")中找到。