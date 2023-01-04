# 春豆名称

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-bean-names>

## 1.概观

当我们有相同类型的多个实现时，命名一个 [Spring bean](/web/20221001115719/https://www.baeldung.com/spring-bean) 非常有用。这是因为如果我们的 bean 没有惟一的名称，那么 Spring 注入 bean 将是不明确的。

通过控制 bean 的命名，我们可以告诉 Spring 我们想要将哪个 bean 注入到目标对象中。

在本文中，我们将讨论 Spring bean 的命名策略，并探索如何为一种类型的 bean 赋予多个名称。

## 2.默认 Bean 命名策略

Spring 为创建 beans 提供了多个[注释](/web/20221001115719/https://www.baeldung.com/spring-bean-annotations)。我们可以在不同的层次上使用这些注释。例如，我们可以将一些注释放在 bean 类上，将其他注释放在创建 bean 的方法上。

首先，让我们看看 Spring 的默认命名策略。当我们只指定注释而没有任何值时，Spring 如何命名我们的 bean？

### 2.1.类级注释

让我们从在类级别使用的注释的默认命名策略开始。为了给 bean 命名， **Spring 使用类名并将第一个字母转换成小写的**。

让我们来看一个例子:

```java
@Service
public class LoggingService {
}
```

这里，Spring 为类`LoggingService` 创建了一个 bean，并使用名称“`loggingService`注册它。

这个相同的默认命名策略适用于所有用于创建 Spring bean 的类级注释，比如`[@Component](/web/20221001115719/https://www.baeldung.com/spring-bean-annotations#component)`、`[@Service](/web/20221001115719/https://www.baeldung.com/spring-bean-annotations#service)`和`[@Controller](/web/20221001115719/https://www.baeldung.com/spring-bean-annotations#controller)`。

### 2.2.方法级注释

Spring 提供了类似于 [`@Bean`](/web/20221001115719/https://www.baeldung.com/spring-bean-annotations) 和 [`@Qualifier`](/web/20221001115719/https://www.baeldung.com/spring-qualifier-annotation) 的注释，用于创建 bean 的方法。

让我们看一个例子来理解`@Bean` 注释的默认命名策略:

```java
@Configuration
public class AuditConfiguration {
    @Bean
    public AuditService audit() {
          return new AuditService();
    }
}
```

在这个配置类中，Spring 将一个类型为`AuditService`的 bean 注册在名称`audit`下，因为**当我们在一个方法上使用`@Bean`注释时，** **Spring 使用方法名称作为 bean 名称**。

我们也可以在方法上使用`@Qualifier`注释，我们将在下面看到一个例子。

## 3.Beans 的自定义命名

当我们需要在同一个 Spring 上下文中创建多个相同类型的 bean 时，我们可以给 bean 定制名称，并使用这些名称来引用它们。

那么，让我们看看如何给我们的 Spring bean 起一个自定义名称:

```java
@Component("myBean")
public class MyCustomComponent {
}
```

这一次，Spring 将创建类型为`MyCustomComponent`的 bean，名称为“`myBean`”。

由于我们显式地给 bean 命名，Spring 将使用这个名称，然后可以用它来引用或访问 bean。

与`@Component(“myBean”)`类似，我们可以使用其他注释来指定名称，比如`@Service(“myService”)`、`@Controller(“myController”)`和`@Bean(“myCustomBean”)`，然后 Spring 将使用给定的名称注册该 bean。

## 4.用`@Bean`和`@Qualifier`命名 Bean

### 4.1.`@Bean`带值

正如我们前面看到的，`@Bean`注释应用于方法级，默认情况下，Spring 使用方法名作为 bean 名。

这个默认的 bean 名称可以被覆盖——我们可以使用 `@Bean` 注释来指定这个值:

```java
@Configuration
public class MyConfiguration {
    @Bean("beanComponent")
    public MyCustomComponent myComponent() {
        return new MyCustomComponent();
    }
}
```

在这种情况下，当我们想要获得一个类型为`MyCustomComponent`的 bean 时，我们可以通过使用名称“`beanComponent`”来引用这个 bean。

Spring `@Bean`注释通常在配置类方法中声明。它可能通过直接调用同一个类中的其他`@Bean`方法来引用它们。

### 4.2.`@Qualifier`带值

我们还可以使用`@Qualifier`注释来命名 bean。

首先，让我们创建一个将由多个类实现的接口`Animal`:

```java
public interface Animal {
    String name();
}
```

现在，让我们定义一个实现类`Cat `，并用值“`cat`”向它添加`@Qualifier `注释:

```java
@Component 
@Qualifier("cat") 
public class Cat implements Animal { 
    @Override 
     public String name() { 
        return "Cat"; 
     } 
}
```

让我们添加`Animal`的另一个实现，并用`@Qualifier` 和值`dog`:对其进行注释

```java
@Component
@Qualifier("dog")
public class Dog implements Animal {
    @Override
    public String name() {
        return "Dog";
    }
}
```

现在，让我们编写一个类`PetShow`，我们可以在其中注入`Animal`的两个不同实例:

```java
@Service 
public class PetShow { 
    private final Animal dog; 
    private final Animal cat; 

    public PetShow (@Qualifier("dog")Animal dog, @Qualifier("cat")Animal cat) { 
      this.dog = dog; 
      this.cat = cat; 
    }
    public Animal getDog() { 
      return dog; 
    }
    public Animal getCat() { 
      return cat; 
    }
}
```

在类`Pet` `Show,` 中，我们已经通过在构造函数参数上使用`@Qualifier`注释注入了类型`Animal` 的两个实现，每个注释的值属性中都有限定的 bean 名称。每当我们使用这个限定名时，Spring 会将带有该限定名的 bean 注入到目标 bean 中。

## 5.验证 Bean 名称

到目前为止，我们已经看到了不同的例子来演示如何给 Spring beans 命名。现在的问题是，我们如何验证或测试这一点？

让我们看一个单元测试来验证这个行为:

```java
@ExtendWith(SpringExtension.class)
public class SpringBeanNamingUnitTest {
    private AnnotationConfigApplicationContext context;

    @BeforeEach
    void setUp() {
        context = new AnnotationConfigApplicationContext();
        context.scan("com.baeldung.springbean.naming");
        context.refresh();
    } 
```

```java
 @Test
    void givenMultipleImplementationsOfAnimal_whenFieldIsInjectedWithQualifiedName_thenTheSpecificBeanShouldGetInjected() {
        PetShow petShow = (PetShow) context.getBean("petShow");
        assertThat(petShow.getCat().getClass()).isEqualTo(Cat.class);
        assertThat(petShow.getDog().getClass()).isEqualTo(Dog.class);
    }
```

在这个 JUnit 测试中，我们正在初始化`setUp` 方法中的`AnnotationConfigApplicationContext`，它用于获取 bean。

然后，我们简单地使用标准断言来验证我们的 Spring beans 的类。

## 6.结论

在这篇简短的文章中，我们研究了默认和定制的 Spring bean 命名策略。

我们还了解了定制 Spring bean 命名在我们需要管理多个相同类型的 bean 的用例中是如何有用的。

和往常一样，这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221001115719/https://github.com/eugenp/tutorials/tree/master/spring-core-5)