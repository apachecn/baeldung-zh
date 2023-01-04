# Spring 4.3 有什么新功能？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/whats-new-in-spring-4-3>

## 1。概述

Spring 4.3 版本对框架的核心容器、缓存、JMS、Web MVC 和测试子模块进行了一些很好的改进。

在本帖中，我们将讨论其中的一些改进，包括:

*   隐式构造函数注入
*   Java 8 默认接口方法支持
*   提高了依赖性的解决方案
*   缓存抽象细化
*   组合的`@RequestMapping`变体
*   @Requestscope、@Sessionscope、@Applicationscope 批注
*   `@RequestAttribute`和`@SessionAttribute`注释
*   库/应用服务器版本支持
*   `InjectionPoint`类

## 2。隐式构造函数注入

考虑以下服务类别:

```
@Service
public class FooService {

    private final FooRepository repository;

    @Autowired
    public FooService(FooRepository repository) {
        this.repository = repository
    }
}
```

这是一个很常见的用例，但是如果您忘记了构造函数上的`@Autowired`注释，容器将抛出一个异常，寻找一个默认的构造函数，除非您显式地进行连接。

所以从 4.3 开始，您不再需要在这种单构造函数场景中指定显式注入注释。这对于根本不带有任何注释的类来说尤其优雅:

```
public class FooService {

    private final FooRepository repository;

    public FooService(FooRepository repository) {
        this.repository = repository
    }
}
```

在 Spring 4.2 和更低版本中，这个 bean 的以下配置将不起作用，因为 Spring 将无法为`FooService`找到默认的构造函数。Spring 4.3 更加智能，将自动连接构造函数:

```
<beans>
    <bean class="com.baeldung.spring43.ctor.FooRepository" />
    <bean class="com.baeldung.spring43.ctor.FooService" />
</beans>
```

类似地，你可能已经注意到`@Configuration`类在历史上不支持构造函数注入。从 4.3 开始，它们允许了，并且它们自然地允许在单构造函数场景中省略`@Autowired`:

```
@Configuration
public class FooConfiguration {

    private final FooRepository repository;

    public FooConfiguration(FooRepository repository) {
        this.repository = repository;
    }

    @Bean
    public FooService fooService() {
        return new FooService(this.repository);
    }
}
```

## 3.Java 8 默认接口方法支持

在 Spring 4.3 之前，不支持默认接口方法。

这并不容易实现，因为即使是 JDK 的 JavaBean 内省器也没有将默认方法检测为访问器。从 Spring 4.3 开始，作为默认接口方法实现的 getters 和 setters 在注入过程中被识别，这允许使用它们作为被访问属性的公共预处理程序，如下例所示:

```
public interface IDateHolder {

    void setLocalDate(LocalDate localDate);

    LocalDate getLocalDate();

    default void setStringDate(String stringDate) {
        setLocalDate(LocalDate.parse(stringDate, 
          DateTimeFormatter.ofPattern("dd.MM.yyyy")));
    }

} 
```

这个 bean 现在可能已经注入了`stringDate`属性:

```
<bean id="dateHolder" 
  class="com.baeldung.spring43.defaultmethods.DateHolder">
    <property name="stringDate" value="15.10.1982"/>
</bean>
```

在默认接口方法上使用类似于`@BeforeTransaction`和`@AfterTransaction` 的测试注释也是如此。JUnit 5 已经支持默认接口方法的测试注释，Spring 4.3 紧随其后。现在，您可以在一个接口中抽象公共测试逻辑，并在测试类中实现它。下面是测试用例的一个接口，它记录测试中事务之前和之后的消息:

```
public interface ITransactionalTest {

    Logger log = LoggerFactory.getLogger(ITransactionalTest.class);

    @BeforeTransaction
    default void beforeTransaction() {
        log.info("Before opening transaction");
    }

    @AfterTransaction
    default void afterTransaction() {
        log.info("After closing transaction");
    }

}
```

关于注释`@BeforeTransaction,` `@AfterTransaction`和`@Transactional`的另一个改进是放松了注释方法应该是`public`的要求——现在它们可以有任何可见性级别。

## 4。提高了依赖性的解决方案

最新版本还引入了`ObjectProvider`，这是对现有`ObjectFactory`接口的扩展，带有方便的签名，如`getIfAvailable`和`getIfUnique`，以便仅在 bean 存在或可以确定单个候选项的情况下检索 bean(特别是:在多个匹配 bean 的情况下作为主要候选项)。

```
@Service
public class FooService {

    private final FooRepository repository;

    public FooService(ObjectProvider<FooRepository> repositoryProvider) {
        this.repository = repositoryProvider.getIfUnique();
    }
}
```

如上所示，在初始化期间，您可以使用这样的`ObjectProvider`句柄进行自定义解析，或者将句柄存储在一个字段中，用于以后的按需解析(就像您通常对`ObjectFactory`所做的那样)。

## 5。缓存抽象细化

缓存抽象主要用于缓存消耗 CPU 和 IO 的值。在特定的用例中，给定的密钥可能被几个线程(即客户端)并行请求，尤其是在启动时。同步缓存支持是一个长期要求的特性，现在已经实现了。假设如下:

```
@Service
public class FooService {

    @Cacheable(cacheNames = "foos", sync = true)
    public Foo getFoo(String id) { ... }

}
```

注意`sync = true`属性，它告诉框架在计算值时阻塞任何并发线程。这将确保在并发访问的情况下，这个密集的操作只被调用一次。

Spring 4.3 还改进了缓存抽象，如下所示:

*   缓存相关注释中的 SpEL 表达式现在可以引用 beans(即`@beanName.method()`)。
*   `ConcurrentMapCacheManager`和`ConcurrentMapCache`现在通过一个新的`storeByValue`属性支持缓存条目的序列化。
*   `@Cacheable`、`@CacheEvict`、`@CachePut`和`@Caching`现在可以用作元注释来创建带有属性覆盖的定制组合注释。

## 6。沉稳`@RequestMapping`变种

Spring Framework 4.3 引入了下面的方法级组合变体`@RequestMapping`注释，它们有助于简化常见 HTTP 方法的映射，并更好地表达带注释的处理程序方法的语义。

*   `@GetMapping`
*   `@PostMapping`
*   `@PutMapping`
*   `@DeleteMapping`
*   `@PatchMapping`

例如，`@GetMapping`是`@RequestMapping(method = RequestMethod.GET)`的一种更简短的说法。下面的例子展示了一个用组合的`@GetMapping`注释简化的 MVC 控制器。

```
@Controller
@RequestMapping("/appointments")
public class AppointmentsController {

    private final AppointmentBook appointmentBook;

    @Autowired
    public AppointmentsController(AppointmentBook appointmentBook) {
        this.appointmentBook = appointmentBook;
    }

    @GetMapping
    public Map<String, Appointment> get() {
        return appointmentBook.getAppointmentsForToday();
    }
}
```

## 7。`@RequestScope`、`@SessionScope`、`@ApplicationScope`注解

当使用注释驱动的组件或 Java 配置时，`@RequestScope`、`@SessionScope`和`@ApplicationScope`注释可用于将组件分配到所需的范围。这些注释不仅设置了 bean 的作用域，还将作用域代理模式设置为`ScopedProxyMode.TARGET_CLASS.`

`TARGET_CLASS` mode 表示 CGLIB 代理将用于代理这个 bean，并确保它可以被注入到任何其他 bean 中，甚至是更广的范围。模式不仅允许接口代理，也允许类代理`.`

```
@RequestScope
@Component
public class LoginAction {
    // ...
}
```

```
@SessionScope
@Component
public class UserPreferences {
    // ...
}
```

```
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}
```

## 8.`@RequestAttribute`和`@SessionAttribute`注释

出现了另外两个用于将 HTTP 请求的参数注入到`Controller`方法中的注释，即`@RequestAttribute`和`@SessionAttribute`。它们允许您访问一些预先存在的、全局管理的属性(即在 `Controller`之外)。这些属性的值可以由例如`javax.servlet.Filter`或`org.springframework.web.servlet.HandlerInterceptor`的注册实例提供。

假设我们已经注册了下面的`HandlerInterceptor`实现，它解析请求并将`login`参数添加到会话中，并将另一个`query`参数添加到请求中:

```
public class ParamInterceptor extends HandlerInterceptorAdapter {

    @Override
    public boolean preHandle(HttpServletRequest request, 
      HttpServletResponse response, Object handler) throws Exception {
        request.getSession().setAttribute("login", "john");
        request.setAttribute("query", "invoices");
        return super.preHandle(request, response, handler);
    }

}
```

这样的参数可以被注入到一个`Controller`实例中，在方法参数上有相应的注释:

```
@GetMapping
public String get(@SessionAttribute String login, 
  @RequestAttribute String query) {
    return String.format("login = %s, query = %s", login, query);
}
```

## 9。库/应用服务器版本支持

Spring 4.3 支持以下库版本和服务器版本:

*   Hibernate ORM 5.2(仍然支持 4.2/4.3 和 5.0/5.1，现在不支持 3.6)
*   杰克逊 2.8(截至 4.3 年春季，最低提升至杰克逊 2.6 以上)
*   OkHttp 3.x(仍然并排支持 OkHttp 2.x)
*   Netty 4.1
*   逆流 1.4
*   Tomcat 8.5.2 和 9.0 M6

此外，Spring 4.3 在`spring-core.jar`中嵌入了更新后的 ASM 5.1 和 Objenesis 2.4。

## 10。`InjectionPoint`

`InjectionPoint`类是 Spring 4.3 中引入的一个新类，它**提供了关于特定 bean 在何处被注入**的信息，无论它是方法/构造函数参数还是字段。

使用该类可以找到的信息类型有:

*   `Field`对象——如果 bean 被注入到一个字段中，您可以通过使用`getField()`方法获得被包装为`Field`对象的注入点
*   `MethodParameter`——如果 bean 被注入到一个参数中，可以调用`getMethodParameter()`方法来获取被包装为`MethodParameter`对象的注入点
*   `Member`–调用`getMember()`方法将返回包含被注入的 bean 的实体，该 bean 被包装到`Member`对象中
*   `Class<?>`–使用`getDeclaredType()`获取注入 bean 的参数或字段的声明类型
*   `Annotation[]`–通过使用`getAnnotations()`方法，您可以检索注释对象的数组，这些对象表示与字段或参数相关联的注释
*   `AnnotatedElement`–调用`getAnnotatedElement()`将注入点包装成一个`AnnotatedElement`对象

这个类非常有用的一个例子是当我们想要基于 beans 所属的类创建`Logger`bean 时:

```
@Bean
@Scope("prototype")
public Logger logger(InjectionPoint injectionPoint) {
    return Logger.getLogger(
      injectionPoint.getMethodParameter().getContainingClass());
}
```

bean 必须用一个`prototype`作用域来定义，以便为每个类创建一个不同的记录器。如果创建一个`singleton` bean 并在多个地方注入，Spring 将返回第一个遇到的注入点。

然后，我们可以将 bean 注入到我们的`AppointmentsController`:

```
@Autowired
private Logger logger;
```

## 11。结论

在本文中，我们讨论了 Spring 4.3 引入的一些新特性。

我们已经讨论了消除样板文件的有用注释、新的有用的依赖性查找和注入方法，以及 web 和缓存设施中的几项重大改进。

你可以在 GitHub 上找到文章[的源代码。](https://web.archive.org/web/20220521214436/https://github.com/eugenp/tutorials/tree/master/spring-4)