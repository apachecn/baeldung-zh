# <annotation-config>vs <component-scan>的区别</component-scan></annotation-config>

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-contextannotation-contextcomponentscan>

## 1.概观

在本教程中，我们将了解 Spring 的两个主要 XML 配置元素之间的差异:`<context:annotation-config>`和`<context:component-scan>`。

## 2.Bean 定义

众所周知，Spring 为我们提供了两种方式来定义我们的[bean](/web/20220629004916/https://www.baeldung.com/spring-bean)和依赖: [XML 配置](/web/20220629004916/https://www.baeldung.com/spring-xml-injection)和 Java 注释。我们还可以将 Spring 的注释分为两类:[依赖注入注释](/web/20220629004916/https://www.baeldung.com/spring-core-annotations)和 [bean 注释](/web/20220629004916/https://www.baeldung.com/spring-core-annotations)。

在使用注释之前，我们必须在 XML 配置文件中手动定义所有的 beans 和依赖项。现在多亏了 Spring 的注释，**它可以自动发现并为我们连接所有的 beans 和依赖关系**。因此，我们至少可以消除 beans 和依赖项所需的 XML。

然而，我们应该记住**注释是没有用的，除非我们激活它们**。**为了激活它们，我们可以在 XML 文件上添加`<context:annotation-config>`或`<context:component-scan> `。**

在这一节中，我们将看到`<context:annotation-config>`和`<context:component-scan>` 在激活注释的方式上有什么不同。

## 3.<`context:annotation-config>`激活注释

`<context:annotation-config>`注释主要用于激活依赖注入注释。 [`@Autowired`](/web/20220629004916/https://www.baeldung.com/spring-autowire) ， [`@Qualifier`](/web/20220629004916/https://www.baeldung.com/spring-core-annotations) ， [`@PostConstruct`](/web/20220629004916/https://www.baeldung.com/spring-postconstruct-predestroy) ， [`@PreDestroy`](/web/20220629004916/https://www.baeldung.com/spring-postconstruct-predestroy) ， [`@Resource`](/web/20220629004916/https://www.baeldung.com/spring-annotations-resource-inject-autowire) 都是`<context:annotation-config>` 可以解决的。

让我们举一个简单的例子来看看`<context:annotation-config>`如何为我们简化 XML 配置。

首先，让我们创建一个具有依赖字段的类:

```
public class UserService {
    @Autowired
    private AccountService accountService;
}
```

```
public class AccountService {}
```

现在，让我们定义我们的 beans。

```
<bean id="accountService" class="AccountService"></bean>

<bean id="userService" class="UserService"></bean>
```

在继续之前，让我们指出我们仍然需要在 XML 中声明 beans。这是因为 **`<context:annotation-config> `只为已经在应用程序上下文**中注册的 beans 激活注释。

正如这里可以看到的，我们使用`@Autowired`注释了`accountService`字段。 **`@Autowired`告诉 Spring 这个字段是一个依赖项，需要由一个匹配的 bean 自动连接。**

如果我们不使用`@Autowired`，那么我们需要手动设置`accountService`依赖关系:

```
<bean id="userService" class="UserService">
    <property name="accountService" ref="accountService"></property>
</bean>
```

现在，我们可以在单元测试中引用我们的 beans 和依赖项:

```
@Test
public void givenContextAnnotationConfig_whenDependenciesAnnotated_thenNoXMLNeeded() {
    ApplicationContext context
      = new ClassPathXmlApplicationContext("classpath:annotationconfigvscomponentscan-beans.xml");

    UserService userService = context.getBean(UserService.class);
    AccountService accountService = context.getBean(AccountService.class);

    Assert.assertNotNull(userService);
    Assert.assertNotNull(accountService);
    Assert.assertNotNull(userService.getAccountService());
}
```

嗯，这里有点不对劲。看起来 Spring 没有连接`accountService`，尽管我们用`@Autowired`对它进行了注释。看样子`@Autowired `并不活跃。为了解决这个问题，我们只需在 XML 文件的顶部添加以下代码行:

```
<context:annotation-config/>
```

## 4.<`context:component-scan>`激活注释

与`<context:annotation-config>`类似，`<context:component-scan>`也可以识别和处理依赖注入注释。此外， **`<context:component-scan>`识别出`<context:annotation-config>`没有检测到** `.` 的 bean 注释

基本上， **`<context:component-scan>`通过包扫描**来检测注释。换句话说，它告诉 Spring 需要扫描哪些包来寻找带注释的 beans 或组件。

[`@Component`](/web/20220629004916/https://www.baeldung.com/spring-component-repository-service) ， [`@Repository`](/web/20220629004916/https://www.baeldung.com/spring-component-repository-service) ， [`@Service`](/web/20220629004916/https://www.baeldung.com/spring-component-repository-service) ， [`@Controller`](/web/20220629004916/https://www.baeldung.com/spring-controller-vs-restcontroller) ， [`@RestController`](/web/20220629004916/https://www.baeldung.com/spring-controller-vs-restcontroller) ， [`@Configuration`](/web/20220629004916/https://www.baeldung.com/spring-mvc-tutorial) 是`<context:component-scan>`可以检测到的几个`.`

现在，让我们看看如何简化前面的示例:

```
@Component
public class UserService {
    @Autowired
    private AccountService accountService;
} 
```

```
@Component
public class AccountService {}
```

这里， **`@Component`注释将我们的类标记为 bean**。现在，我们可以从 XML 文件中清除所有的 bean 定义。当然，我们需要保持`<context:component-scan>`在它上面:

```
<context:component-scan
  base-package="com.baeldung.annotationconfigvscomponentscan.components" />
```

最后，我们注意到 Spring 将在由`base-package`属性指示的包下寻找带注释的 beans 和依赖项。

## 5.结论

在本教程中，我们看了一下`<context:annotation-config>`和`<context:component-scan>`之间的区别。

代码样本一如既往地在 GitHub 上[结束。](https://web.archive.org/web/20220629004916/https://github.com/eugenp/tutorials/tree/master/spring-5)