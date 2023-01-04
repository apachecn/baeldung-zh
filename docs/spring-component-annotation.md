# Spring @组件注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-component-annotation>

## 1.概观

在本教程中，我们将全面了解一下 [Spring `@Component`](/web/20220827110142/https://www.baeldung.com/spring-component-repository-service) 注释和相关区域。我们将看到使用它来集成一些核心 Spring 功能的不同方式，以及如何利用它的诸多优势。

## 2.弹簧`ApplicationContext`

在我们能够理解`@Component`的值之前，我们首先需要稍微了解一下[弹簧`ApplicationContext`](/web/20220827110142/https://www.baeldung.com/spring-application-context) 。

Spring `ApplicationContext`是 Spring 保存对象实例的地方，这些对象被它标识为自动管理和分发。这些叫做豆子。

Bean 管理和依赖注入的机会是 Spring 的一些主要特性。

使用[反转控制原理](/web/20220827110142/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)， **Spring 从我们的应用程序中收集 bean 实例，并在适当的时候使用它们。**我们可以向 Spring 显示 bean 依赖关系，而不需要处理这些对象的设置和实例化。

使用像 [`@Autowired`](/web/20220827110142/https://www.baeldung.com/spring-autowire) 这样的注释将 Spring 管理的 beans 注入到我们的应用程序中的能力是在 Spring 中创建强大和可伸缩代码的驱动力。

那么，我们如何告诉 Spring 我们希望它为我们管理的 beans 呢？我们应该通过在我们的类上使用原型注释来利用 Spring 的自动 bean 检测。

## 3.`@Component`

`@Component`是一个注释，允许 Spring 自动检测我们的定制 beans。

换句话说，无需编写任何显式代码，Spring 将:

*   扫描我们的应用程序，寻找用`@Component`标注的类
*   实例化它们并将任何指定的依赖项注入其中
*   注射到任何需要的地方

然而，大多数开发人员更喜欢使用更加专门化的原型注释来服务这个功能。

### 3.1.Spring 原型注释

Spring 提供了一些专门的原型注释:`@Controller`、`@Service`和`@Repository`。它们都提供与`@Component`相同的功能。

**它们的行为都是一样的，因为它们都是用`@Component`作为元注释的复合注释。**它们类似于`@Component`别名，具有 Spring 自动检测或依赖注入之外的特殊用途和含义。

如果我们真的想这么做，理论上我们可以选择使用`@Component`专门满足我们的 bean 自动检测需求。另一方面，我们也可以[使用`@Component`编写我们自己的专用注释](/web/20220827110142/https://www.baeldung.com/java-custom-annotation)。

然而，Spring 的其他领域专门寻找 Spring 的专用注释来提供额外的自动化好处。因此，我们可能应该在大多数时候坚持使用已建立的专业化。

让我们假设在我们的 Spring Boot 项目中有这些情况的例子:

```java
@Controller
public class ControllerExample {
}

@Service
public class ServiceExample {
}

@Repository
public class RepositoryExample {
}

@Component
public class ComponentExample {
}

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Component
public @interface CustomComponent {
}

@CustomComponent
public class CustomComponentExample {
}
```

我们可以编写一个测试，证明 Spring 自动检测到每一个，并将其添加到`ApplicationContext`:

```java
@SpringBootTest
@ExtendWith(SpringExtension.class)
public class ComponentUnitTest {

    @Autowired
    private ApplicationContext applicationContext;

    @Test
    public void givenInScopeComponents_whenSearchingInApplicationContext_thenFindThem() {
        assertNotNull(applicationContext.getBean(ControllerExample.class));
        assertNotNull(applicationContext.getBean(ServiceExample.class));
        assertNotNull(applicationContext.getBean(RepositoryExample.class));
        assertNotNull(applicationContext.getBean(ComponentExample.class));
        assertNotNull(applicationContext.getBean(CustomComponentExample.class));
    }
}
```

### 3.2.`@ComponentScan`

在我们完全依赖`@Component`之前，我们必须明白它只是一个普通的注释。注释的目的是将 beans 与其他对象(如域对象)区分开来。

然而， **Spring 使用`@ComponentScan`注释将它们全部聚集到它的`ApplicationContext`中。**

如果我们正在编写一个 Spring Boot 应用程序，知道`@SpringBootApplication`是一个包含`@ComponentScan`的组合注释是很有帮助的。只要我们的`@SpringBootApplication` 类是我们项目的根，它就会扫描我们默认定义的每一个`@Component`。

但是万一我们的`@SpringBootApplication`类不能位于我们项目的根，或者我们想要扫描外部源代码，[我们可以显式地配置`@ComponentScan`](/web/20220827110142/https://www.baeldung.com/spring-component-scanning#component-scan) 来查看我们指定的任何包，只要它存在于类路径中。

让我们定义一个超出范围的`@Component` bean:

```java
package com.baeldung.component.scannedscope;

@Component
public class ScannedScopeExample {
}
```

接下来，我们可以通过对我们的`@ComponentScan`注释的显式指令来包含它:

```java
package com.baeldung.component.inscope;

@SpringBootApplication
@ComponentScan({"com.baeldung.component.inscope", "com.baeldung.component.scannedscope"})
public class ComponentApplication {
    //public static void main(String[] args) {...}
}
```

最后，我们可以测试它是否存在:

```java
@Test
public void givenScannedScopeComponent_whenSearchingInApplicationContext_thenFindIt() {
    assertNotNull(applicationContext.getBean(ScannedScopeExample.class));
}
```

实际上，当我们想要扫描包含在项目中的外部依赖项时，这种情况更有可能发生。

### 3.3.`@Component`限制

在一些场景中，当我们不能使用`@Component`时，我们希望某个对象成为 Spring 管理的 bean。

让我们在项目外部的包中定义一个用`@Component`注释的对象:

```java
package com.baeldung.component.outsidescope;

@Component
public class OutsideScopeExample {
}
```

这里有一个测试证明`ApplicationContext`不包括外部组件:

```java
@Test
public void givenOutsideScopeComponent_whenSearchingInApplicationContext_thenFail() {
    assertThrows(NoSuchBeanDefinitionException.class, () -> applicationContext.getBean(OutsideScopeExample.class));
}
```

此外，我们可能无法访问源代码，因为它来自第三方来源，并且我们无法添加`@Component`注释。或者，我们可能希望根据我们运行的环境有条件地使用一个 bean 实现而不是另一个。**自动检测在大多数时候是足够的，但是当它不够时，我们可以使用`@Bean`。**

## 4.`@Component`对`@Bean`

`@Bean`也是 Spring 在运行时用来收集 beans 的注释，但它不是在类级别使用的。相反，**我们用`@Bean`注释方法，这样 Spring 可以将方法的结果存储为 Spring bean。**

我们将首先创建一个没有注释的 POJO:

```java
public class BeanExample {
}
```

在我们用`@Configuration`注释的类内部，我们可以创建一个 bean 生成方法:

```java
@Bean
public BeanExample beanExample() {
    return new BeanExample();
}
```

可能代表一个本地类，也可能是一个外部类。这无关紧要，因为我们只需要返回它的一个实例。

然后，我们可以编写一个测试来验证 Spring 确实拾取了 bean:

```java
@Test
public void givenBeanComponent_whenSearchingInApplicationContext_thenFindIt() {
    assertNotNull(applicationContext.getBean(BeanExample.class));
}
```

由于`@Component`和`@Bean`之间的差异，我们应该注意一些重要的含义。

*   `@Component`是类级别的注释，但是`@Bean`是在方法级别，所以`@Component`只是当类的源代码可编辑时的一个选项。`@Bean`可以一直用，但是比较啰嗦。
*   `@Component`兼容 Spring 的自动检测，但是`@Bean`需要手动类实例化。
*   使用`@Bean`将 bean 的实例化与其类定义解耦。这就是为什么我们可以用它把甚至第三方的类做成春豆。这也意味着我们可以引入逻辑来决定 bean 使用几个可能的实例选项中的哪一个。

## 5.结论

我们刚刚探讨了 Spring `@Component`注释以及其他相关主题。首先，我们讨论了各种 Spring 原型注释，它们只是`@Component`的特殊版本。

然后我们了解到`@Component`不做任何事情，除非它能被`@ComponentScan`找到。

最后，由于不可能在没有源代码的类上使用`@Component`，我们学习了如何使用`@Bean`注释。

所有这些代码示例以及更多内容都可以在 GitHub 上找到[。](https://web.archive.org/web/20220827110142/https://github.com/eugenp/tutorials/tree/master/spring-core-5)