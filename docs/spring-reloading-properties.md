# 在 Spring 中重载属性文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-reloading-properties>

## 1.概观

在本教程中，我们将展示如何在 Spring 应用程序中重新加载[属性。](/web/20220628145405/https://www.baeldung.com/properties-with-spring)

## 2.春季阅读属性

在 Spring 中，我们有不同的选项来访问属性:

1.  `Environment` —我们可以注入`Environment` ，然后使用`Environment#getProperty` 来读取给定的属性。`Environment`包含不同的属性来源，如系统属性、`-D`参数和`application.properties (.yml)`。另外，可以使用`@PropertySource`将额外的属性源添加到`Environment`中。
2.  `Properties` —我们可以将属性文件加载到一个`Properties`实例中，然后通过调用`properties.get(“property”).`在 bean 中使用它
3.  [`@Value`](/web/20220628145405/https://www.baeldung.com/spring-value-annotation) —我们可以用`@Value(${‘property'})`注释在 bean 中注入特定的属性。
4.  `[@ConfigurationProperties](/web/20220628145405/https://www.baeldung.com/configuration-properties-in-spring-boot)` —我们可以使用`@ConfigurationProperties`在 bean 中加载分层属性。

## 3.从外部文件重新加载属性

为了在运行时改变文件的属性，我们应该把文件放在 jar 之外的某个地方。然后，我们将通过命令行参数告诉 Spring 它在哪里。或者，我们可以把它放在`application.properties.`

在基于文件的属性中，我们必须选择一种方式来重新加载文件。例如，我们可以开发一个端点或调度程序来读取文件并更新属性。

一个方便的重新加载文件的库是 Apache 的`commons-configuration`。我们可以用 [`PropertiesConfiguration`](https://web.archive.org/web/20220628145405/https://commons.apache.org/proper/commons-configuration/apidocs/org/apache/commons/configuration2/PropertiesConfiguration.html) 搭配不同的 `ReloadingStrategy`。

让我们将 [`commons-configuration`](https://web.archive.org/web/20220628145405/https://search.maven.org/search?q=g:commons-configuration%20a:commons-configuration) 添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>commons-configuration</groupId>
    <artifactId>commons-configuration</artifactId>
    <version>1.10</version>
</dependency>
```

然后，我们添加一个方法来创建一个`PropertiesConfiguration` bean，我们稍后会用到它:

```java
@Bean
@ConditionalOnProperty(name = "spring.config.location", matchIfMissing = false)
public PropertiesConfiguration propertiesConfiguration(
  @Value("${spring.config.location}") String path) throws Exception {
    String filePath = new File(path.substring("file:".length())).getCanonicalPath();
    PropertiesConfiguration configuration = new PropertiesConfiguration(
      new File(filePath));
    configuration.setReloadingStrategy(new FileChangedReloadingStrategy());
    return configuration;
}
```

在上面的代码中，我们将`FileChangedReloadingStrategy`设置为具有默认刷新延迟的重载策略。这意味着`PropertiesConfiguration`检查文件修改日期**，如果它的最后一次检查是在 5000 毫秒之前**。

我们可以使用`FileChangedReloadingStrategy#setRefreshDelay.`自定义延迟

### 3.1.重新加载`Environment`属性

如果我们想重新加载通过`Environment` 实例加载的属性，我们必须**扩展`PropertySource`，然后使用`PropertiesConfiguration`从外部属性文件**返回新值。

让我们从扩展`PropertySource`开始:

```java
public class ReloadablePropertySource extends PropertySource {

    PropertiesConfiguration propertiesConfiguration;

    public ReloadablePropertySource(String name, PropertiesConfiguration propertiesConfiguration) {
        super(name);
        this.propertiesConfiguration = propertiesConfiguration;
    }

    public ReloadablePropertySource(String name, String path) {
        super(StringUtils.hasText(name) ? path : name);
        try {
            this.propertiesConfiguration = new PropertiesConfiguration(path);
            this.propertiesConfiguration.setReloadingStrategy(new FileChangedReloadingStrategy());
        } catch (Exception e) {
            throw new PropertiesException(e);
        }
    }

    @Override
    public Object getProperty(String s) {
        return propertiesConfiguration.getProperty(s);
    }
}
```

我们已经覆盖了`getProperty`方法，将其委托给`PropertiesConfiguration#getProperty.` ，因此，它将根据我们的刷新延迟检查更新的值。

现在，我们将把我们的`ReloadablePropertySource`添加到`Environment`的财产来源中:

```java
@Configuration
public class ReloadablePropertySourceConfig {

    private ConfigurableEnvironment env;

    public ReloadablePropertySourceConfig(@Autowired ConfigurableEnvironment env) {
        this.env = env;
    }

    @Bean
    @ConditionalOnProperty(name = "spring.config.location", matchIfMissing = false)
    public ReloadablePropertySource reloadablePropertySource(PropertiesConfiguration properties) {
        ReloadablePropertySource ret = new ReloadablePropertySource("dynamic", properties);
        MutablePropertySources sources = env.getPropertySources();
        sources.addFirst(ret);
        return ret;
    }
}
```

**我们添加了新的属性源作为第一项**,因为我们希望它覆盖任何具有相同键的现有属性。

让我们创建一个 bean 来读取来自`Environment`的属性:

```java
@Component
public class EnvironmentConfigBean {

    private Environment environment;

    public EnvironmentConfigBean(@Autowired Environment environment) {
        this.environment = environment;
    }

    public String getColor() {
        return environment.getProperty("application.theme.color");
    }
}
```

如果我们需要添加其他可重载的外部属性源，首先我们必须实现我们的自定义 `PropertySourceFactory`:

```java
public class ReloadablePropertySourceFactory extends DefaultPropertySourceFactory {
    @Override
    public PropertySource<?> createPropertySource(String s, EncodedResource encodedResource)
      throws IOException {
        Resource internal = encodedResource.getResource();
        if (internal instanceof FileSystemResource)
            return new ReloadablePropertySource(s, ((FileSystemResource) internal)
              .getPath());
        if (internal instanceof FileUrlResource)
            return new ReloadablePropertySource(s, ((FileUrlResource) internal)
              .getURL()
              .getPath());
        return super.createPropertySource(s, encodedResource);
    }
}
```

然后我们可以用`@PropertySource`注释一个组件的类:

```java
@PropertySource(value = "file:path-to-config", factory = ReloadablePropertySourceFactory.class)
```

### 3.2.正在重新加载属性实例

`Environment`是比`Properties`更好的选择，尤其是当我们需要从文件中重新加载属性的时候。但是，如果我们需要，我们可以延长`java.util.Properties`:

```java
public class ReloadableProperties extends Properties {
    private PropertiesConfiguration propertiesConfiguration;

    public ReloadableProperties(PropertiesConfiguration propertiesConfiguration) throws IOException {
        super.load(new FileReader(propertiesConfiguration.getFile()));
        this.propertiesConfiguration = propertiesConfiguration;
    }

    @Override
    public String getProperty(String key) {
        String val = propertiesConfiguration.getString(key);
        super.setProperty(key, val);
        return val;
    }

    // other overrides
}
```

我们已经覆盖了`getProperty`及其重载，然后将其委托给一个`PropertiesConfiguration`实例。现在，我们可以创建这个类的 bean，并将其注入到我们的组件中。

### 3.3.用`@ConfigurationProperties`重新加载 Bean

为了用`@ConfigurationProperties`获得同样的效果，我们需要重建实例。

但是，Spring 只会创建一个具有`prototype`或`request`范围的组件的新实例。

因此，我们重新加载环境的技术也适用于它们，但是对于单例来说，我们别无选择，只能实现一个端点来销毁和重新创建 bean，或者在 bean 本身内部处理属性 reload。

### 3.4.用`@Value`重新加载 Bean

`@Value`注释呈现出与 [`@ConfigurationProperties`](#reloading-config-prop) 相同的限制。

## 4.通过执行器和云重新加载属性

[Spring Actuator](/web/20220628145405/https://www.baeldung.com/spring-boot-actuators) 为健康、度量和配置提供了不同的端点，但是没有为刷新 beans 提供任何东西。因此，我们需要 Spring Cloud 为它添加一个`/refresh`端点。该端点重新加载`Environment`的所有属性源，然后发布一个 [`EnvironmentChangeEvent`](https://web.archive.org/web/20220628145405/https://static.javadoc.io/org.springframework.cloud/spring-cloud-commons-parent/1.1.9.RELEASE/org/springframework/cloud/context/environment/EnvironmentChangeEvent.html) `.`

春云也推出了 [`@RefreshScope`](https://web.archive.org/web/20220628145405/https://static.javadoc.io/org.springframework.cloud/spring-cloud-commons-parent/1.1.4.RELEASE/org/springframework/cloud/context/scope/refresh/RefreshScope.html) ，我们可以用它来配置类或者 beans。因此，默认范围将是`refresh`而不是 [`singleton`](/web/20220628145405/https://www.baeldung.com/spring-bean-scopes) 。

使用`refresh` 范围，Spring 将在`EnvironmentChangeEvent`上清除这些组件的内部缓存。然后，在下一次访问 bean 时，会创建一个新实例。

让我们从将`spring-boot-starter-actuator`添加到`pom.xml`开始:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency> 
```

那么，我们也导入 [`spring-cloud-dependencies`](https://web.archive.org/web/20220628145405/https://search.maven.org/search?q=g:org.springframework.cloud%20a:spring-cloud-dependencies) :

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<properties>
    <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
</properties> 
```

然后我们加上`spring-cloud-starter`:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter</artifactId>
</dependency>
```

最后，让我们启用刷新端点:

```java
management.endpoints.web.exposure.include=refresh
```

当我们使用 Spring Cloud 时，我们可以设置一个[配置服务器](/web/20220628145405/https://www.baeldung.com/spring-cloud-configuration)来管理属性，但是我们也可以继续使用我们的外部文件。现在，我们可以处理另外两种读取属性的方法:`@Value`和`@ConfigurationProperties`。

### 4.1.用`@ConfigurationProperties`刷新 Beans

让我们展示一下如何将`@ConfigurationProperties`与`@RefreshScope`一起使用:

```java
@Component
@ConfigurationProperties(prefix = "application.theme")
@RefreshScope
public class ConfigurationPropertiesRefreshConfigBean {
    private String color;

    public void setColor(String color) {
        this.color = color;
    }

    //getter and other stuffs
}
```

我们的 bean 正在从根`“application`读取`color”`属性。`theme”` 属性`.` **注意，根据 Spring 的文档，我们确实需要 setter 方法。**

在我们更改了外部配置文件中的值`application.theme.color`之后，我们可以调用`/refresh`，这样我们就可以在下次访问时从 bean 中获取新值。

### 4.2.用`@Value`刷新 Beans

让我们创建示例组件:

```java
@Component
@RefreshScope
public class ValueRefreshConfigBean {
    private String color;

    public ValueRefreshConfigBean(@Value("${application.theme.color}") String color) {
        this.color = color;
    } 
    //put getter here 
}
```

刷新的过程同上。

**然而，需要注意的是，`/refresh`对于具有明确`singleton`范围的 beans 不起作用。**

## 5.结论

在本教程中，我们演示了如何使用或不使用 Spring Cloud 功能来重新加载属性。此外，我们还展示了每种技术的缺陷和例外。

完整的代码在我们的 GitHub 项目中[可用。](https://web.archive.org/web/20220628145405/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-properties)