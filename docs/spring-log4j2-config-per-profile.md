# 每个弹簧剖面有不同的 Log4j2 配置

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-log4j2-config-per-profile>

## 1.概观

在我们之前的教程中， [Spring Profiles](/web/20221208143835/https://www.baeldung.com/spring-profiles) 和[登录 Spring Boot](/web/20221208143835/https://www.baeldung.com/spring-boot-logging) ，我们展示了如何在 Spring 中激活不同的配置文件和使用 Log4j2。

在这个简短的教程中，我们将学习**如何为每个弹簧轮廓**使用不同的 Log4j2 配置。

## 2.使用不同的属性文件

例如，假设我们有两个文件，`log4j2.xml`和`log4j2-dev.xml`，一个用于默认概要文件，另一个用于“dev”概要文件。

让我们创建我们的`application.properties`文件，并告诉它在哪里可以找到日志配置文件:

```
logging.config=/path/to/log4j2.xml
```

接下来，让我们为名为`application-dev.properties`的“dev”概要文件创建一个新的属性文件，并添加一行类似的代码:

```
logging.config=/path/to/log4j2-dev.xml
```

如果我们有其他概要文件——例如“prod”——我们只需要为我们的“prod”概要文件创建一个名称相似的属性文件—`application-prod.properties`。**配置文件特定的属性总是覆盖默认属性**。

## 3.程序配置

我们可以通过改变我们的 Spring Boot `Application`类，以编程方式选择使用哪个 Log4j2 配置文件:

```
@SpringBootApplication
public class Application implements CommandLineRunner {

    @Autowired
    private Environment env;

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Override
    public void run(String... param) {
        if (Arrays.asList(env.getActiveProfiles()).contains("dev")) {
            Configurator.initialize(null, "/path/to/log4j2-dev.xml");
        } else {
            Configurator.initialize(null, "/path/to/log4j2.xml");
        }
    }
}
```

`Configurator`是 Log4j2 库的一个类。它提供了几种使用配置文件的位置和各种可选参数来构造`LoggerContext`的方法。

**这个解决方案有一个缺点:应用程序启动过程不会使用 Log4j2** 来记录。

## 4.结论

总之，我们已经看到了两种在每个 Spring profile 中使用不同 Log4j2 配置的方法。首先，我们看到我们可以为每个概要文件提供不同的属性文件。然后，我们看到了一种在应用程序启动时基于活动概要文件以编程方式配置 Log4j2 的方法。