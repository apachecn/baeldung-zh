# 将 Mockito Mocks 注入春豆

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/injecting-mocks-in-spring>

## 1。概述

在本教程中，我们将讨论如何使用依赖注入将 Mockito mocks 插入到 Spring Beans 中进行单元测试。

在现实世界的应用程序中，组件经常依赖于访问外部系统，提供适当的测试隔离很重要，这样我们就可以专注于测试给定单元的功能，而不必为每个测试涉及整个类层次结构。

注入 mock 是引入这种隔离的一种干净的方式。

## 2。Maven 依赖关系

对于单元测试和模拟对象，我们需要以下 Maven 依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>2.7.2</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>2.7.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>2.21.0</version>
</dependency>
```

我们决定在这个例子中使用 Spring Boot，但是经典的 Spring 也可以。

## 3。编写测试

### 3.1。商业逻辑

首先，让我们创建一个将要测试的简单服务:

```java
@Service
public class NameService {
    public String getUserName(String id) {
        return "Real user name";
    }
}
```

然后我们将它注入到`UserService`类中:

```java
@Service
public class UserService {

    private NameService nameService;

    @Autowired
    public UserService(NameService nameService) {
        this.nameService = nameService;
    }

    public String getUserName(String id) {
        return nameService.getUserName(id);
    }
}
```

对于本文，给定的类返回单个名称，而不考虑提供的 id。这样做是为了我们不会因为测试任何复杂的逻辑而分心。

我们还需要一个标准的 Spring Boot 主类来扫描 beans 并初始化应用程序:

```java
@SpringBootApplication
public class MocksApplication {
    public static void main(String[] args) {
        SpringApplication.run(MocksApplication.class, args);
    }
}
```

### 3.2。测试

现在让我们继续测试逻辑。首先，我们必须为测试配置应用程序上下文:

```java
@Profile("test")
@Configuration
public class NameServiceTestConfiguration {
    @Bean
    @Primary
    public NameService nameService() {
        return Mockito.mock(NameService.class);
    }
}
```

`@Profile`注释告诉 Spring 只有在“test”概要文件激活时才应用这个配置。`@Primary`注释是为了确保这个实例用于自动连接，而不是一个真实的实例。该方法本身创建并返回我们的`NameService`类的 Mockito mock。

现在我们可以编写单元测试了:

```java
@ActiveProfiles("test")
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = MocksApplication.class)
public class UserServiceUnitTest {

    @Autowired
    private UserService userService;

    @Autowired
    private NameService nameService;

    @Test
    public void whenUserIdIsProvided_thenRetrievedNameIsCorrect() {
        Mockito.when(nameService.getUserName("SomeId")).thenReturn("Mock user name");
        String testName = userService.getUserName("SomeId");
        Assert.assertEquals("Mock user name", testName);
    }
}
```

我们使用`@ActiveProfiles`注释来启用“测试”概要文件，并激活我们之前编写的模拟配置。因此，Spring 自动连接了一个真实的`UserService`类实例，但是模拟了`NameService`类。测试本身是一个相当典型的 JUnit+Mockito 测试。我们配置模拟的期望行为，然后调用我们想要测试的方法，并断言它返回我们期望的值。

在这样的测试中避免使用环境概要文件也是可能的(尽管不推荐)。为此，我们移除了`@Profile` 和 `@ActiveProfiles` 注释，并向`UserServiceTest`类添加了一个`@ContextConfiguration(classes = NameServiceTestConfiguration.class)` 注释。

## 4。结论

在这篇简短的文章中，我们了解了将 Mockito mocks 注入 Spring Beans 是多么容易。

像往常一样，所有的代码样本都可以在 [GitHub](https://web.archive.org/web/20220926183800/https://github.com/eugenp/tutorials/tree/master/testing-modules/spring-mockito) 上找到。