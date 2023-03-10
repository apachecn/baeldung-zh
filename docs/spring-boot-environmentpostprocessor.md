# Spring Boot 的环境后处理器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-environmentpostprocessor>

## 1.概观

从 Spring Boot 1.3 开始，我们能够在应用上下文被刷新之前使用`EnvironmentPostProcessor` 到 **自定义应用的`Environment`**。

在本教程中，让我们看看如何将自定义属性加载并转换到`Environment,`中，然后访问这些属性`.`

## 2.弹簧`Environment`

Spring 中的`Environment`抽象表示当前应用程序运行的环境。同时，它倾向于统一访问各种属性源中的属性的方式，例如属性文件、JVM 系统属性、系统环境变量和 servlet 上下文参数。

**所以在大多数情况下，定制`Environment`意味着在将各种属性暴露给我们的 beans 之前对它们进行操作。**首先，请访问我们之前关于使用 Spring 操纵属性的文章[。](/web/20220627174907/https://www.baeldung.com/properties-with-spring)

## 3.一个简单的例子

现在让我们构建一个简单的价格计算应用程序。它会以总额或净额的方式计算价格。来自第三方的系统环境变量将决定选择哪种计算模式。

### 3.1.实施`EnvironmentPostProcessor`

为此，让我们实现`EnvironmentPostProcessor`接口。

我们将使用它来读取几个环境变量:

```java
calculation_mode=GROSS 
gross_calculation_tax_rate=0.15
```

我们将使用后处理器以特定于应用的方式公开这些内容，在本例中使用自定义前缀:

```java
com.baeldung.environmentpostprocessor.calculation.mode=GROSS
com.baeldung.environmentpostprocessor.gross.calculation.tax.rate=0.15
```

然后，我们可以非常简单地将新属性添加到`Environment`中:

```java
@Order(Ordered.LOWEST_PRECEDENCE)
public class PriceCalculationEnvironmentPostProcessor implements EnvironmentPostProcessor {

    @Override
    public void postProcessEnvironment(ConfigurableEnvironment environment, 
      SpringApplication application) {
        PropertySource<?> system = environment.getPropertySources()
          .get(SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME);
        if (!hasOurPriceProperties(system)) {
          // error handling code omitted
        }
        Map<String, Object> prefixed = names.stream()
          .collect(Collectors.toMap(this::rename, system::getProperty));
        environment.getPropertySources()
          .addAfter(SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME, new MapPropertySource("prefixer", prefixed));
    }

}
```

让我们看看我们在这里做了什么。首先，**我们要求`environment`给我们环境变量的`PropertySource`。**调用结果`system.getProperty`类似于调用 Java 的`System.getenv().get.`

然后，只要这些属性存在于环境中，**我们将创建一个新的映射，** `**prefixed.**`为了简洁起见，我们将跳过`rename`的内容，但是查看完整实现的代码示例。**生成的映射与`system`具有相同的值，但带有前缀关键字。**

最后，我们现在将把我们的新`PropertySource `添加到`Environment.`中，如果一只豆子请求`com.baeldung.environmentpostprocessor.calculation.mode`，`Environment`将查询我们的地图。

顺便注意一下， [`EnvironmentPostProcessor`的 Javadoc](https://web.archive.org/web/20220627174907/https://github.com/spring-projects/spring-boot/blob/v2.1.3.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/env/EnvironmentPostProcessor.java#L31) 鼓励我们要么实现`Ordered` 接口，要么[使用 `@Order` 注释](/web/20220627174907/https://www.baeldung.com/spring-order)。

当然，**这只是一个单一的资产来源** `.` Spring Boot 允许我们迎合众多的来源和格式。

### 3.2.注册在`spring.factories`

为了调用 Spring Boot 引导过程中的实现，我们需要在`META-INF/spring.factories`中注册这个类:

```java
org.springframework.boot.env.EnvironmentPostProcessor=
  com.baeldung.environmentpostprocessor.PriceCalculationEnvironmentPostProcessor
```

### 3.3.使用`@Value`注释访问属性

让我们在几节课中使用这些。在这个示例中，我们有一个带有两个实现的`PriceCalculator `接口:`GrossPriceCalculator`和`NetPriceCalculator.`

在我们的实现中， **[我们可以只使用`@Value`](/web/20220627174907/https://www.baeldung.com/spring-value-annotation) 来检索我们的新属性:**

```java
public class GrossPriceCalculator implements PriceCalculator {
    @Value("${com.baeldung.environmentpostprocessor.gross.calculation.tax.rate}")
    double taxRate;

    @Override
    public double calculate(double singlePrice, int quantity) {
        //calcuation implementation omitted
    }
}
```

这很好，因为这与我们访问任何其他属性的方式相同，比如我们在`application.properties.`中定义的那些属性

### 3.4.访问 Spring Boot 自动配置中的属性

现在，让我们来看一个复杂的例子，我们在 Spring Boot 自动配置中访问前面的属性。

我们将创建自动配置类来读取这些属性。该类将根据不同的属性值初始化并连接应用程序上下文中的 beans:

```java
@Configuration
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
public class PriceCalculationAutoConfig {
    @Bean
    @ConditionalOnProperty(name = 
      "com.baeldung.environmentpostprocessor.calculation.mode", havingValue = "NET")
    @ConditionalOnMissingBean
    public PriceCalculator getNetPriceCalculator() {
        return new NetPriceCalculator();
    }

    @Bean
    @ConditionalOnProperty(name = 
      "com.baeldung.environmentpostprocessor.calculation.mode", havingValue = "GROSS")
    @ConditionalOnMissingBean
    public PriceCalculator getGrossPriceCalculator() {
        return new GrossPriceCalculator();
    }
}
```

类似于`EnvironmentPostProcessor`实现，自动配置类也需要在`META-INF/spring.factories`中注册:

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=
  com.baeldung.environmentpostprocessor.autoconfig.PriceCalculationAutoConfig
```

这是因为**定制`EnvironmentPostProcessor`的实现在 Spring Boot 自动配置**之前就开始了。这种组合使得 Spring Boot 自动配置更加强大。

另外，关于 Spring Boot 自动配置的更多细节，请看看关于使用 Spring Boot 定制自动配置的文章。

## 4.测试自定义实现

现在是时候测试我们的代码了。我们可以通过运行以下命令在 Windows 中设置系统环境变量:

```java
set calculation_mode=GROSS
set gross_calculation_tax_rate=0.15
```

或者在 Linux/Unix 中，我们可以将它们改为:

```java
export calculation_mode=GROSS 
export gross_calculation_tax_rate=0.15
```

之后，我们可以用`mvn spring-boot:run`命令开始测试:

```java
mvn spring-boot:run
  -Dstart-class=com.baeldung.environmentpostprocessor.PriceCalculationApplication
  -Dspring-boot.run.arguments="100,4"
```

## 5.结论

**总而言之，`EnvironmentPostProcessor`实现能够从不同的位置加载各种格式的任意文件。**此外，我们可以做任何我们需要的转换，使属性在`Environment` 中随时可用，以备后用。当我们将基于 Spring Boot 的应用程序与第三方配置集成时，这种自由肯定是有用的。

源代码可以在[的 GitHub 库](https://web.archive.org/web/20220627174907/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-environment)中找到。