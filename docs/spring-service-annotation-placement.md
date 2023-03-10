# Spring @Service 注释应该保存在哪里？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-service-annotation-placement>

## 1.介绍

作为软件开发人员，我们总是在寻找使用给定技术或库的最佳实践。自然，有时会有争论。

其中一个争论是关于 Spring 的`@Service`注释的位置。因为 Spring 提供了定义 beans 的替代方法，所以关注一下原型注释的位置是值得的。

在本教程中，我们将查看`@Service`注释，并检查**将其放在接口、抽象类或具体类**上是否最合适。

## 2.`@Service`在接口上

一些开发人员可能决定将`@Service`放在接口上，因为他们想:

*   明确表明接口应该仅用于服务级别目的
*   定义新的服务实现，并在启动时将它们自动检测为 Spring beans

让我们看看如果我们注释一个接口会是什么样子:

```java
@Service
public interface AuthenticationService {

    boolean authenticate(String username, String password);
}
```

正如我们注意到的，`AuthenticationService`现在变得更加自我描述。`@Service`标记建议开发人员仅将它用于业务层服务，而不要用于数据访问层或任何其他层。

通常情况下，这很好，但有一个缺点。**通过将 Spring 的`@Service`放在接口上，我们创建了一个额外的依赖项，并将我们的接口与外部库耦合起来。**

接下来，为了测试新服务 beans 的自动检测，让我们创建一个`AuthenticationService`的实现:

```java
public class InMemoryAuthenticationService implements AuthenticationService {

    @Override
    public boolean authenticate(String username, String password) {
        //...
    }
}
```

我们应该注意，我们的新实现`InMemoryAuthenticationService`上没有`@Service`注释。我们只在界面上留下了`@Service`，`AuthenticationService`。

因此，让我们借助一个基本的 Spring Boot 设置来运行我们的 Spring 上下文:

```java
@SpringBootApplication
public class AuthApplication {

    @Autowired
    private AuthenticationService authService;

    public static void main(String[] args) {
        SpringApplication.run(AuthApplication.class, args);
    }
}
```

当我们运行我们的应用程序时，**我们得到了臭名昭著的`NoSuchBeanDefinitionException,`** 并且 Spring 上下文无法启动:

```java
org.springframework.beans.factory.NoSuchBeanDefinitionException: 
No qualifying bean of type 'com.baeldung.annotations.service.interfaces.AuthenticationService' available: 
expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: 
...
```

因此，**放置`@Service`在接口上不足以自动检测弹簧组件**。

## 3.`@Service`抽象类上

在抽象类上使用`@Service`注释并不常见。

让我们测试一下，看看它是否达到了让 Spring 自动检测我们的实现类的目的。

我们将从头开始定义一个抽象类，并在上面加上`@Service`注释:

```java
@Service
public abstract class AbstractAuthenticationService {

    public boolean authenticate(String username, String password) {
        return false;
    }
}
```

接下来，我们将`AbstractAuthenticationService`扩展到**创建一个具体的实现，而不注释它**:

```java
public class LdapAuthenticationService extends AbstractAuthenticationService {

    @Override
    public boolean authenticate(String username, String password) { 
        //...
    }
}
```

相应地，我们也更新我们的`AuthApplication`，给**注入新的服务类**:

```java
@SpringBootApplication
public class AuthApplication {

    @Autowired
    private AbstractAuthenticationService authService;

    public static void main(String[] args) {
        SpringApplication.run(AuthApplication.class, args);
    }
}
```

我们应该注意，我们没有试图在这里直接注入抽象类，这是不可能的。相反，**我们打算获取具体类`LdapAuthenticationService`的一个实例，这仅取决于抽象类型**。这是一个很好的实践，正如里斯科夫替代原则所建议的那样。

所以，我们再次运行我们的`AuthApplication`:

```java
org.springframework.beans.factory.NoSuchBeanDefinitionException: 
No qualifying bean of type 'com.baeldung.annotations.service.abstracts.AbstractAuthenticationService' available: 
expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: 
...
```

正如我们所看到的，Spring 上下文没有开始。它**以同样的`NoSuchBeanDefinitionException`** 异常结束。

当然，**在抽象类上使用`@Service`注释在 Spring** 中没有任何效果。

## 4.`@Service`关于具体的类

与我们上面看到的相反，注释实现类而不是抽象类或接口是很常见的做法。

这样，我们的目标主要是告诉 Spring 这个类将是一个 [`@Component`](/web/20221230191547/https://www.baeldung.com/spring-component-repository-service) ，并用一个特殊的构造型来标记它，在我们的例子中是`@Service`。

因此，Spring 将从类路径中自动检测这些类，并自动将它们定义为托管 beans。

所以，这一次让我们把`@Service`放在我们的具体服务类上。我们将有一个类实现我们的接口，第二个类扩展我们之前定义的抽象类:

```java
@Service
public class InMemoryAuthenticationService implements AuthenticationService {

    @Override
    public boolean authenticate(String username, String password) {
        //...
    }
}

@Service
public class LdapAuthenticationService extends AbstractAuthenticationService {

    @Override
    public boolean authenticate(String username, String password) {
        //...
    }
}
```

我们应该注意这里的`AbstractAuthenticationService`没有实现这里的`AuthenticationService`。因此，我们可以独立测试它们。

最后，我们将两个服务类都添加到`AuthApplication`中，并尝试一下:

```java
@SpringBootApplication
public class AuthApplication {

    @Autowired
    private AuthenticationService inMemoryAuthService;

    @Autowired
    private AbstractAuthenticationService ldapAuthService;

    public static void main(String[] args) {
        SpringApplication.run(AuthApplication.class, args);
    }
}
```

**我们的最终测试给了我们一个成功的结果**，Spring 上下文无一例外地启动了。这两个服务都自动注册为 beans。

## 5.结果呢

最终，我们发现唯一可行的方法是将`@Service`放在我们的实现类上，使它们可以自动检测。 **Spring 的[组件扫描](/web/20221230191547/https://www.baeldung.com/spring-component-scanning)不会选择类，除非它们被单独注释，即使它们是从另一个`@Service`注释的接口或抽象类中派生出来的。**

另外， [Spring 的文档](https://web.archive.org/web/20221230191547/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Service.html)还指出，在实现类上使用`@Service`允许组件扫描自动检测它们。

## 6.结论

在本文中，我们研究了使用 Spring 的`@Service`注释的不同地方，并了解了在哪里保留`@Service`来定义服务级 Spring beans，以便在组件扫描期间自动检测它们。

具体来说，我们看到在接口或抽象类上放置`@Service`注释没有任何效果，只有具体的类在用`@Service`注释时才会被组件扫描选中。

和往常一样，GitHub 上的[提供了所有代码示例和更多内容。](https://web.archive.org/web/20221230191547/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-annotations)