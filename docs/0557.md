# CDI 拦截器 vs Spring AspectJ

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cdi-interceptor-vs-spring-aspectj>

## 1。简介

拦截器模式通常用于在应用程序中添加新的、横切的功能或逻辑，并且在大量的库中有可靠的支持。

在本文中，我们将介绍并对比其中的两个主要库:CDI 拦截器和 Spring AspectJ。

## 2。CDI 拦截器项目设置

Jakarta EE 官方支持 CDI，但是一些实现支持在 Java SE 环境中使用 CDI。 [Weld](https://web.archive.org/web/20220121025849/http://weld.cdi-spec.org/) 可以被认为是 Java SE 支持的 CDI 实现的一个例子。

为了使用 CDI，我们需要在 POM 中导入焊接库:

```
<dependency>
    <groupId>org.jboss.weld.se</groupId>
    <artifactId>weld-se-core</artifactId>
    <version>3.0.5.Final</version>
</dependency>
```

最新的焊接库可以在 [Maven](https://web.archive.org/web/20220121025849/https://mvnrepository.com/artifact/org.jboss.weld.se/weld-se-core) 库中找到。

现在让我们创建一个简单的拦截器。

## 3。介绍 CDI 拦截器

为了指定我们需要拦截的类，让我们创建拦截器绑定:

```
@InterceptorBinding
@Target( { METHOD, TYPE } )
@Retention( RUNTIME )
public @interface Audited {
}
```

定义了拦截器绑定之后，我们需要定义实际的拦截器实现:

```
@Audited
@Interceptor
public class AuditedInterceptor {
    public static boolean calledBefore = false;
    public static boolean calledAfter = false;

    @AroundInvoke
    public Object auditMethod(InvocationContext ctx) throws Exception {
        calledBefore = true;
        Object result = ctx.proceed();
        calledAfter = true;
        return result;
    }
}
```

每个`@AroundInvoke`方法接受一个`javax.interceptor.InvocationContext`参数，返回一个`java.lang.Object`，并且可以抛出一个`Exception`。

因此，当我们用新的`@Audit`接口注释一个方法时，`auditMethod`将首先被调用，只有这样目标方法才会继续。

## 4。应用 CDI 拦截器

让我们将创建的拦截器应用于一些业务逻辑:

```
public class SuperService {
    @Audited
    public String deliverService(String uid) {
        return uid;
    }
}
```

我们已经创建了这个简单的服务，并用`@Audited`注释注释了我们想要拦截的方法。

要启用 CDI 拦截器，需要在位于`META-INF`目录的`beans.xml`文件中指定完整的类名:

```
<beans 
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
      http://java.sun.com/xml/ns/javaee/beans_1_2.xsd">
    <interceptors>
        <class>com.baeldung.interceptor.AuditedInterceptor</class>
    </interceptors>
</beans>
```

为了验证拦截器确实工作了**，现在让我们运行下面的测试**:

```
public class TestInterceptor {
    Weld weld;
    WeldContainer container;

    @Before
    public void init() {
        weld = new Weld();
        container = weld.initialize();
    }

    @After
    public void shutdown() {
        weld.shutdown();
    }

    @Test
    public void givenTheService_whenMethodAndInterceptorExecuted_thenOK() {
        SuperService superService = container.select(SuperService.class).get();
        String code = "123456";
        superService.deliverService(code);

        Assert.assertTrue(AuditedInterceptor.calledBefore);
        Assert.assertTrue(AuditedInterceptor.calledAfter);
    }
}
```

在这个快速测试中，我们首先从容器中获取 bean `SuperService`,然后调用它的业务方法`deliverService`,并通过验证它的状态变量来检查拦截器`AuditedInterceptor`是否被调用。

我们还有`@Before`和`@After`带注释的方法，在这些方法中，我们分别初始化和关闭焊接容器。

## 5。CDI 注意事项

我们可以指出 CDI 拦截器的以下优点:

*   这是 Jakarta EE 规范的标准特征
*   一些 CDI 实现库可以在 Java SE 中使用
*   当我们的项目对第三方库有严格限制时，可以使用

CDI 拦截器的缺点如下:

*   具有业务逻辑的类和拦截器之间的紧密耦合
*   很难看出项目中截取了哪些类
*   缺乏将拦截器应用于一组方法的灵活机制

## 6。Spring AspectJ

Spring 也使用 AspectJ 语法支持拦截器功能的类似实现。

首先，我们需要向 POM 添加以下 Spring 和 AspectJ 依赖项:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.8.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.2</version>
</dependency>
```

最新版本的 [Spring context](https://web.archive.org/web/20220121025849/https://mvnrepository.com/artifact/org.springframework/spring-context) 、 [aspectjweaver](https://web.archive.org/web/20220121025849/https://mvnrepository.com/artifact/org.aspectj/aspectjweaver) 可以在 Maven 资源库中找到。

我们现在可以使用 AspectJ 注释语法创建一个简单的方面:

```
@Aspect
public class SpringTestAspect {
    @Autowired
    private List accumulator;

    @Around("execution(* com.baeldung.spring.service.SpringSuperService.*(..))")
    public Object auditMethod(ProceedingJoinPoint jp) throws Throwable {
        String methodName = jp.getSignature().getName();
        accumulator.add("Call to " + methodName);
        Object obj = jp.proceed();
        accumulator.add("Method called successfully: " + methodName);
        return obj;
    }
}
```

我们创建了一个适用于所有`SpringSuperService`类方法的方面——为了简单起见，看起来像这样:

```
public class SpringSuperService {
    public String getInfoFromService(String code) {
        return code;
    }
}
```

## 7。Spring AspectJ 方面应用

为了验证方面是否真正适用于服务，让我们编写下面的单元测试:

```
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = { AppConfig.class })
public class TestSpringInterceptor {
    @Autowired
    SpringSuperService springSuperService;

    @Autowired
    private List accumulator;

    @Test
    public void givenService_whenServiceAndAspectExecuted_thenOk() {
        String code = "123456";
        String result = springSuperService.getInfoFromService(code);

        Assert.assertThat(accumulator.size(), is(2));
        Assert.assertThat(accumulator.get(0), is("Call to getInfoFromService"));
        Assert.assertThat(accumulator.get(1), is("Method called successfully: getInfoFromService"));
    }
}
```

在这个测试中，我们注入我们的服务，调用方法并检查结果。

配置如下所示:

```
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {
    @Bean
    public SpringSuperService springSuperService() {
        return new SpringSuperService();
    }

    @Bean
    public SpringTestAspect springTestAspect() {
        return new SpringTestAspect();
    }

    @Bean
    public List getAccumulator() {
        return new ArrayList();
    }
}
```

`@EnableAspectJAutoProxy`注释的一个重要方面——支持处理用 AspectJ 的`@Aspect`注释标记的组件，类似于 Spring 的 XML 元素中的功能。

## 8。Spring AspectJ 考虑事项

让我们指出使用 Spring AspectJ 的一些优点:

*   拦截器从业务逻辑中分离出来
*   拦截器可以从依赖注入中受益
*   拦截器本身拥有所有的配置信息
*   添加新的拦截器不需要增加现有的代码
*   拦截器有灵活的机制来选择拦截的方法
*   可在没有 Jakarta EE 的情况下使用

当然还有一些缺点:

*   您需要知道 AspectJ 语法来开发拦截器
*   AspectJ 拦截器的学习曲线高于 CDI 拦截器

## 9。CDI 拦截器 vs Spring AspectJ

如果您当前的项目使用 Spring，那么考虑 Spring AspectJ 是一个不错的选择。

如果您使用的是成熟的应用服务器，或者您的项目不使用 Spring(或其他框架，如 Google Guice ),并且完全是 Jakarta EE，那么除了选择 CDI 拦截器之外，别无选择。

## 10。结论

在本文中，我们讨论了拦截器模式的两种实现:CDI 拦截器和 Spring AspectJ。我们已经讨论了它们各自的优缺点。

本文示例的源代码可以在我们位于 [GitHub](https://web.archive.org/web/20220121025849/https://github.com/eugenp/tutorials/tree/master/cdi) 的资源库中找到。