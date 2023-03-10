# Dropwizard 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-dropwizard>

## 1.概观

Dropwizard 是一个开源的 Java 框架，用于快速开发高性能 RESTful web 服务。它收集了一些流行的库来创建轻量级包。它使用的主要库是 Jetty、Jersey、Jackson、JUnit 和 Guava。此外，它使用自己的名为[度量](/web/20221126223734/https://www.baeldung.com/dropwizard-metrics)的库。

在本教程中，我们将学习如何配置和运行一个简单的 Dropwizard 应用程序。当我们完成时，我们的应用程序将公开一个 RESTful API，允许我们获取存储的品牌列表。

## 2.Maven 依赖性

首先，为了创建我们的服务，我们只需要依赖关系。让我们将它添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>io.dropwizard</groupId>
    <artifactId>dropwizard-core</artifactId>
    <version>2.0.0</version>
</dependency>
```

## 3.配置

现在，我们将创建运行每个 Dropwizard 应用程序所需的必要类。

Dropwizard 应用程序将属性存储在 YML 文件中。因此，我们将在资源目录中创建`introduction-config.yml` 文件:

```java
defaultSize: 5
```

我们可以通过创建一个扩展`io.dropwizard.Configuration`的类来访问该文件中的值:

```java
public class BasicConfiguration extends Configuration {
    @NotNull private final int defaultSize;

    @JsonCreator
    public BasicConfiguration(@JsonProperty("defaultSize") int defaultSize) {
        this.defaultSize = defaultSize;
    }

    public int getDefaultSize() {
        return defaultSize;
    }
}
```

Dropwizard 使用 [Jackson](/web/20221126223734/https://www.baeldung.com/jackson) 将配置文件反序列化到我们的类中。因此，我们使用了杰克逊注释。

接下来，让我们创建主应用程序类，它负责准备我们的服务以供使用:

```java
public class IntroductionApplication extends Application<BasicConfiguration> {

    public static void main(String[] args) throws Exception {
        new IntroductionApplication().run("server", "introduction-config.yml");
    }

    @Override
    public void run(BasicConfiguration basicConfiguration, Environment environment) {
        //register classes
    }

    @Override
    public void initialize(Bootstrap<BasicConfiguration> bootstrap) {
        bootstrap.setConfigurationSourceProvider(new ResourceConfigurationSourceProvider());
        super.initialize(bootstrap);
    }
}
```

首先，`main`方法负责运行应用程序。我们既可以将`args`传递给`run`方法，也可以自己填充。

**第一个参数可以是`server`或`check`。**`check`选项验证配置，而`server`选项运行应用程序。**第二个参数是配置文件的位置。**

此外，`initialize` 方法将配置提供者设置为`ResourceConfigurationSourceProvider`，这允许应用程序在资源目录中找到给定的配置文件。不一定要覆盖这个方法。

最后，`run`方法允许我们访问`Environment`和`BaseConfiguration`，我们将在本文后面使用它们。

## 4.资源

首先，让我们为我们的品牌创建一个域类:

```java
public class Brand {
    private final Long id;
    private final String name;

    // all args constructor and getters
}
```

其次，让我们创建一个负责返回品牌的`BrandRepository`类:

```java
public class BrandRepository {
    private final List<Brand> brands;

    public BrandRepository(List<Brand> brands) {
        this.brands = ImmutableList.copyOf(brands);
    }

    public List<Brand> findAll(int size) {
        return brands.stream()
          .limit(size)
          .collect(Collectors.toList());
    }

    public Optional<Brand> findById(Long id) {
        return brands.stream()
          .filter(brand -> brand.getId().equals(id))
          .findFirst();
    }
}
```

此外，**我们能够使用来自[番石榴](/web/20221126223734/https://www.baeldung.com/guava-collections)的`ImmutableList `，因为它是 Dropwizard 本身的一部分。**

第三，我们将创建一个`BrandResource` 类。**drop wizard 默认使用 [JAX-RS](/web/20221126223734/https://www.baeldung.com/jax-rs-spec-and-implementations) ，用新泽西作为实现**。因此，我们将利用该规范中的注释来公开我们的 REST API 端点:

```java
@Path("/brands")
@Produces(MediaType.APPLICATION_JSON)
public class BrandResource {
    private final int defaultSize;
    private final BrandRepository brandRepository;

    public BrandResource(int defaultSize, BrandRepository brandRepository) {
        this.defaultSize = defaultSize;
        this.brandRepository = brandRepository;
    }

    @GET
    public List<Brand> getBrands(@QueryParam("size") Optional<Integer> size) {
        return brandRepository.findAll(size.orElse(defaultSize));
    }

    @GET
    @Path("/{id}")
    public Brand getById(@PathParam("id") Long id) {
        return brandRepository
          .findById(id)
          .orElseThrow(RuntimeException::new);
    }
}
```

此外，我们已经将`size`定义为`Optional`,以便在没有提供参数的情况下使用配置中的`defaultSize` 。

最后，我们将把`BrandResource`注册到`IntroductionApplicaton`类中。为了做到这一点，让我们实现`run` 方法:

```java
@Override
public void run(BasicConfiguration basicConfiguration, Environment environment) {
    int defaultSize = basicConfiguration.getDefaultSize();
    BrandRepository brandRepository = new BrandRepository(initBrands());
    BrandResource brandResource = new BrandResource(defaultSize, brandRepository);

    environment
      .jersey()
      .register(brandResource);
}
```

所有创建的资源都应该用这种方法注册。

## 5.运行应用程序

在这一节中，我们将学习如何从命令行运行应用程序。

首先，我们将配置我们的项目来使用`[maven-shade-plugin](https://web.archive.org/web/20221126223734/https://search.maven.org/search?q=g:org.apache.maven.plugins%20AND%20a:maven-shade-plugin)`构建一个 JAR 文件:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <configuration>
        <createDependencyReducedPom>true</createDependencyReducedPom>
        <filters>
            <filter>
                <artifact>*:*</artifact>
                <excludes>
                    <exclude>META-INF/*.SF</exclude>
                    <exclude>META-INF/*.DSA</exclude>
                    <exclude>META-INF/*.RSA</exclude>
                </excludes>
            </filter>
        </filters>
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <transformers>
                    <transformer
                      implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>com.baeldung.dropwizard.introduction.IntroductionApplication</mainClass>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

这是插件的建议配置。此外，我们在`<mainClass>`元素中包含了主类的路径。

最后，我们将使用 [Maven](/web/20221126223734/https://www.baeldung.com/maven) 构建应用程序。一旦我们有了 JAR 文件，我们就可以运行应用程序了:

```java
java -jar target/dropwizard-0.0.1-SNAPSHOT.jar
```

没有必要传递参数，因为我们已经将它们包含在了`IntroductionApplication `类中。

之后，控制台日志应该以以下内容结束:

```java
INFO  [2020-01-08 18:55:06,527] org.eclipse.jetty.server.Server: Started @1672ms
```

现在，应用程序正在监听端口 8080，我们可以在`http://localhost:8080/brands`访问我们的品牌端点。

## 6.健康检查

当启动应用程序时，我们被告知应用程序没有任何健康检查。幸运的是， **Dropwizard 提供了一个简单的解决方案来为我们的应用程序**添加健康检查。

让我们从添加一个扩展`com.codahale.metrics.health.HealthCheck`的简单类开始:

```java
public class ApplicationHealthCheck extends HealthCheck {
    @Override
    protected Result check() throws Exception {
        return Result.healthy();
    }
}
```

这个简单的方法将返回关于组件健康状况的信息。我们可以创建多个健康检查，其中一些在某些情况下可能会失败。例如，如果数据库连接失败，我们将返回`Result.unhealthy()`。

最后，我们需要**在我们`IntroductionApplication`类的`run`方法中注册我们的健康检查**:

```java
environment
  .healthChecks()
  .register("application", new ApplicationHealthCheck());
```

运行应用程序后，我们可以在`http://localhost:8081/healthcheck`下**检查健康检查响应**:

```java
{
  "application": {
    "healthy": true,
    "duration": 0
  },
  "deadlocks": {
    "healthy": true,
    "duration": 0
  }
}
```

如我们所见，我们的健康检查已经登记在`application`标签下。

## 7.结论

在本文中，我们已经学习了如何用 Maven 设置 Dropwizard 应用程序。

我们发现应用程序的基本设置非常简单快捷。此外，Dropwizard 包括我们运行高性能 RESTful web 服务所需的每个库。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221126223734/https://github.com/eugenp/tutorials/tree/master/web-modules/dropwizard)