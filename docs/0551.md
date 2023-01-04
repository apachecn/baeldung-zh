# juice vs spring——依赖注入

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guice-spring-dependency-injection>

## 1.介绍

[`Google Guice`](/web/20220930092230/https://www.baeldung.com/guice) 和 [`Spring`](/web/20220930092230/https://www.baeldung.com/spring-tutorial) 是两个用于依赖注入的健壮框架。两个框架都涵盖了依赖注入的所有概念，但是每个框架都有自己的实现方式。

在本教程中，我们将讨论 Guice 和 Spring 框架在配置和实现上的不同。

## 2.Maven 依赖性

让我们首先将 Guice 和 Spring Maven 依赖项添加到我们的`pom.xml`文件中:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.1.4.RELEASE</version>
</dependency>

<dependency>
    <groupId>com.google.inject</groupId>
    <artifactId>guice</artifactId>
    <version>4.2.2</version>
</dependency>
```

我们总是可以从 Maven Central 访问最新的`[spring-context](https://web.archive.org/web/20220930092230/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-context%22) `或`[guice](https://web.archive.org/web/20220930092230/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.google.inject%22%20a%3Aguice)`依赖项。

## 3.依赖注入配置

依赖注入是一种编程技术，我们用它来使我们的类独立于它们的依赖。

在这一节中，我们将提到 Spring 和 Guice 在配置依赖注入方面的几个核心特性。

### 3.1.弹簧布线

Spring 在一个特殊的配置类中声明了依赖注入配置。这个类必须通过`@Configuration`注释来注释。Spring 容器使用这个类作为 bean 定义的来源。

**Spring 管理的类称为** **[春豆](/web/20220930092230/https://www.baeldung.com/spring-bean)。**

**弹簧使用 [`@Autowired`](/web/20220930092230/https://www.baeldung.com/spring-autowire) 标注为[自动关联](/web/20220930092230/https://www.baeldung.com/spring-annotations-resource-inject-autowire)** 。`@Autowired`是 [Spring 内置核心注释](/web/20220930092230/https://www.baeldung.com/spring-core-annotations)的一部分。我们可以在成员变量、setter 方法和构造函数上使用`@Autowired`。

Spring 还支持`@Inject. @Inject`是 [Java CDI(上下文和依赖注入)](/web/20220930092230/https://www.baeldung.com/java-ee-cdi)的一部分，它定义了依赖注入的标准。

假设我们想自动将一个依赖项连接到一个成员变量。我们可以简单地用`@Autowired`来注释它:

```
@Component
public class UserService {
    @Autowired
    private AccountService accountService;
}
```

```
@Component
public class AccountServiceImpl implements AccountService {
}
```

其次，让我们创建一个配置类，在加载应用程序上下文时用作 beans 的源:

```
@Configuration
@ComponentScan("com.baeldung.di.spring")
public class SpringMainConfig {
}
```

注意，我们还用`@Component`注释了 *UserService* 和`AccountServiceImpl`，将它们注册为 beans。**是`@ComponentScan`注释告诉 Spring 在哪里搜索**带注释的组件。

尽管我们已经注释了`AccountServiceImpl**,**` ，但是 Spring 可以将它映射到`AccountService `，因为它实现了`AccountService`。

然后，我们需要定义一个应用程序上下文来访问 beans。请注意，我们将在所有的 Spring 单元测试中引用这个上下文:

```
ApplicationContext context = new AnnotationConfigApplicationContext(SpringMainConfig.class);
```

现在在运行时，我们可以从我们的`UserService` bean 中检索`A` `ccountService`实例:

```
UserService userService = context.getBean(UserService.class);
assertNotNull(userService.getAccountService());
```

### 3.2。Guice 绑定

**Guice 在一个叫做模块的特殊类中管理它的依赖关系。**Guice 模块必须扩展`AbstractModule `类并覆盖其`configure()`方法。

Guice 使用绑定作为 Spring 中连接的等价物。简单地说，**绑定允许我们定义如何将依赖注入到一个类**中。Guice 绑定是在我们模块的`configure() `方法中声明的。

**代替`@Autowired`，Guice 使用 [`@Inject`](/web/20220930092230/https://www.baeldung.com/spring-annotations-resource-inject-autowire) 注释来注入依赖关系。**

让我们创建一个等效的果汁示例:

```
public class GuiceUserService {
    @Inject
    private AccountService accountService;
}
```

其次，我们将创建模块类，它是我们绑定定义的来源:

```
public class GuiceModule extends AbstractModule {
    @Override
    protected void configure() {
        bind(AccountService.class).to(AccountServiceImpl.class);
    }
}
```

通常，如果在`configure()`方法中没有明确定义任何绑定，我们希望 Guice 从它们的默认构造函数中实例化每个依赖对象。**但是由于接口不能被直接实例化，我们需要定义绑定**来告诉 Guice 哪个接口将与哪个实现配对。

然后，我们需要使用`GuiceModule`定义一个`Injector`来获取我们的类的实例。请注意，我们所有的 Guice 测试都将使用这个`Injector`:

```
Injector injector = Guice.createInjector(new GuiceModule());
```

最后，在运行时，我们检索一个具有非空`accountService`依赖关系的`GuiceUserService`实例:

```
GuiceUserService guiceUserService = injector.getInstance(GuiceUserService.class);
assertNotNull(guiceUserService.getAccountService());
```

### 3.3.Spring 的@Bean 注释

**Spring 还提供了一个方法级注释`@Bean`来注册 bean**，作为其类级注释`@Component`的替代。一个`@Bean`注释方法的返回值被注册为容器中的一个 bean。

假设我们有一个`BookServiceImpl`的实例，我们想让它可用于注入。我们可以使用`@Bean`来注册我们的实例:

```
@Bean 
public BookService bookServiceGenerator() {
    return new BookServiceImpl();
}
```

现在我们可以得到一个`BookService` bean:

```
BookService bookService = context.getBean(BookService.class);
assertNotNull(bookService);
```

### 3.4。Guice 的@提供了注释

作为 Spring 的`@Bean`注释的等价物， **Guice 有一个内置的注释`[@Provides](https://web.archive.org/web/20220930092230/https://github.com/google/guice/blob/master/core/src/com/google/inject/Provides.java) `来做同样的工作**。和`@Bean`一样，`@Provides`只适用于方法。

现在让我们用 Guice 实现前面的 Spring bean 示例。我们需要做的就是将下面的代码添加到我们的模块类中:

```
@Provides
public BookService bookServiceGenerator() {
    return new BookServiceImpl();
}
```

现在，我们可以检索一个`BookService`的实例:

```
BookService bookService = injector.getInstance(BookService.class);
assertNotNull(bookService);
```

### 3.5。Spring 中的类路径组件扫描

Spring 提供了一个 **`@ComponentScan`注释，通过扫描预定义的包自动检测和实例化带注释的组件**。

`@ComponentScan` 注释告诉 Spring 哪些包将被扫描以寻找带注释的组件。它与`@Configuration`注释一起使用。

### 3.6。Guice 中的类路径组件扫描

与 Spring 不同， **Guice 没有这样的组件扫描特性**。但是模拟起来并不难。有一些类似 [`Governator`](https://web.archive.org/web/20220930092230/https://github.com/Netflix/governator) 的插件可以把这个特性带入 Guice。

### 3.7.Spring 中的对象识别

Spring 通过名称来识别对象。**弹簧将物体保持在一个大致类似于`Map<String, Object>`的结构中。**这意味着我们不能有两个同名的对象。

由于拥有多个同名 Bean 而导致的 Bean 冲突是 Spring 开发者遇到的一个常见问题。例如，让我们考虑以下 bean 声明:

```
@Configuration
@Import({SpringBeansConfig.class})
@ComponentScan("com.baeldung.di.spring")
public class SpringMainConfig {
    @Bean
    public BookService bookServiceGenerator() {
        return new BookServiceImpl();
    }
}
```

```
@Configuration
public class SpringBeansConfig {
    @Bean
    public AudioBookService bookServiceGenerator() {
        return new AudioBookServiceImpl();
    }
}
```

正如我们所记得的，我们已经在`SpringMainConfig`类中为`BookService`定义了一个 bean。

为了在这里创建 bean 冲突，我们需要用相同的名称声明 bean 方法。但是我们不允许在一个类中有两个同名的不同方法。出于这个原因，我们在另一个配置类中声明了`AudioBookService` bean。

现在，让我们在单元测试中引用这些 beans:

```
BookService bookService = context.getBean(BookService.class);
assertNotNull(bookService); 
AudioBookService audioBookService = context.getBean(AudioBookService.class);
assertNotNull(audioBookService);
```

单元测试将失败，出现以下情况:

```
org.springframework.beans.factory.NoSuchBeanDefinitionException:
No qualifying bean of type 'AudioBookService' available
```

首先，Spring 在其 bean 映射中注册了名为`“bookServiceGenerator”`的`AudioBookService` bean。然后，由于`HashMap`数据结构的 **`“no duplicate names allowed”`** 性质，它必须通过`BookService`的 bean 定义覆盖它。

最后，**我们可以通过使 bean 方法名唯一或者将`name`属性设置为每个`@Bean`的唯一名称来解决这个问题。**

### 3.8.Guice 中的对象识别

与 Spring 不同， **Guice 基本上有一个`Map` <级<？>，对象>结构**。这意味着，如果不使用额外的元数据，我们就不能拥有同一个类型的多个绑定。

Guice 提供了[绑定注释](https://web.archive.org/web/20220930092230/https://github.com/google/guice/wiki/BindingAnnotations)来为同一类型定义多个绑定。让我们看看如果在 Guice 中同一类型有两个不同的绑定会发生什么。

```
public class Person {
}
```

现在，让我们为`Person`类声明两个不同的绑定:

```
bind(Person.class).toConstructor(Person.class.getConstructor());
bind(Person.class).toProvider(new Provider<Person>() {
    public Person get() {
        Person p = new Person();
        return p;
    }
});
```

下面是我们如何获得一个`Person`类的实例:

```
Person person = injector.getInstance(Person.class);
assertNotNull(person);
```

这将失败，原因是:

```
com.google.inject.CreationException: A binding to Person was already configured at GuiceModule.configure()
```

我们可以通过简单地丢弃`Person`类的一个绑定来解决这个问题。

### 3.9.Spring 中的可选依赖项

[可选依赖关系](/web/20220930092230/https://www.baeldung.com/spring-autowire#dependencies)是自动连接或注入 beans 时不需要的依赖关系。

对于已经注释了`@Autowired`的字段，如果在上下文中没有找到匹配数据类型的 bean，Spring 将抛出`NoSuchBeanDefinitionException`。

然而，**有时我们可能想跳过一些依赖项的自动连接，让它们作为`null`** 而不抛出异常:

现在让我们看看下面的例子:

```
@Component
public class BookServiceImpl implements BookService {
    @Autowired
    private AuthorService authorService;
}
```

```
public class AuthorServiceImpl implements AuthorService {
}
```

从上面的代码中我们可以看到，`AuthorServiceImpl`类还没有被注释为组件。我们将假设在我们的配置文件中没有它的 bean 声明方法。

现在，让我们运行下面的测试来看看会发生什么:

```
BookService bookService = context.getBean(BookService.class);
assertNotNull(bookService);
```

毫不奇怪，它会失败:

```
org.springframework.beans.factory.NoSuchBeanDefinitionException: 
No qualifying bean of type 'AuthorService' available
```

**我们可以通过使用 [Java 8 的`Optional`类型](/web/20220930092230/https://www.baeldung.com/java-optional)来避免这种异常，从而使`authorService`依赖成为可选的。**

```
public class BookServiceImpl implements BookService {
    @Autowired
    private Optional<AuthorService> authorService;
}
```

现在，我们的`authorService`依赖更像是一个容器，可能包含也可能不包含`AuthorService`类型的 bean。即使在我们的应用程序上下文中没有`AuthorService`的 bean，我们的`authorService`字段仍然是非`null`空容器。因此，Spring 没有任何理由抛出`NoSuchBeanDefinitionException`。

作为对`Optional`的替代，我们可以使用`@Autowired`的`required`属性，默认设置为`true`，使依赖项可选。**我们可以将`required`属性设置为`false`,使自动连接的依赖项可选。**

因此，如果其数据类型的 bean 在上下文中不可用，Spring 将跳过注入依赖项。依赖性将保持设置为`null:`

```
@Component
public class BookServiceImpl implements BookService {
    @Autowired(required = false)
    private AuthorService authorService;
}
```

有时将依赖项标记为可选会很有用，因为并非所有的依赖项都是必需的。

记住这一点，我们应该记住，我们需要在开发过程中使用额外的谨慎和`null`-检查，以避免由于`null`依赖性而导致的任何`NullPointerException`。

### 3.10.Guice 中的可选依赖项

就像`Spring`、**、`Guice`也可以使用 Java 8 的`Optional`类型，使依赖项可选。**

假设我们想要创建一个具有`Foo`依赖关系的类:

```
public class FooProcessor {
    @Inject
    private Foo foo;
}
```

现在，让我们为`Foo`类定义一个绑定:

```
bind(Foo.class).toProvider(new Provider<Foo>() {
    public Foo get() {
        return null;
    }
});
```

现在让我们试着在单元测试中获得一个`FooProcessor`的实例:

```
FooProcessor fooProcessor = injector.getInstance(FooProcessor.class);
assertNotNull(fooProcessor);
```

我们的单元测试将会失败:

```
com.google.inject.ProvisionException:
null returned by binding at GuiceModule.configure(..)
but the 1st parameter of FooProcessor.[...] is not @Nullable
```

为了跳过这个异常，我们可以通过简单的更新使`foo`依赖项可选:

```
public class FooProcessor {
    @Inject
    private Optional<Foo> foo;
}
```

`@Inject`没有将依赖项标记为可选的`required`属性。在 Guice 中，使**成为可选依赖项的另一种方法是使用`@Nullable`注释。**

Guice 允许在使用`@Nullable`的情况下注入`null`值，如上面的异常消息所示。让我们套用一下`@Nullable`的注解:

```
public class FooProcessor {
    @Inject
    @Nullable
    private Foo foo;
}
```

## 4.依赖注入类型的实现

在这一节中，我们将看看[依赖注入类型](/web/20220930092230/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)，并通过几个例子比较 Spring 和 Guice 提供的实现。

### 4.1.春季构造函数注入

**在[基于构造函数的依赖注入](/web/20220930092230/https://www.baeldung.com/constructor-injection-in-spring)中，我们在实例化时将所需的依赖传递给一个类。**

假设我们想要一个 Spring 组件，并想通过它的构造函数添加依赖项。我们可以用`@Autowired`来注释构造函数:

```
@Component
public class SpringPersonService {

    private PersonDao personDao;

    @Autowired
    public SpringPersonService(PersonDao personDao) {
        this.personDao = personDao;
    }
}
```

从 Spring 4 开始，如果类只有一个构造函数，这种类型的注入就不需要`@Autowired`依赖。

让我们在测试中检索一个`SpringPersonService` bean:

```
SpringPersonService personService = context.getBean(SpringPersonService.class);
assertNotNull(personService);
```

### 4.2。Guice 中的构造函数注入

我们可以重新安排前面的例子，让**在 Guice** 中实现构造函数注入。注意 Guice 用的是`@Inject`而不是`@Autowired`。

```
public class GuicePersonService {

    private PersonDao personDao;

    @Inject
    public GuicePersonService(PersonDao personDao) {
        this.personDao = personDao;
    }
}
```

下面是我们如何在测试中从`injector`中获取`GuicePersonService`类的实例:

```
GuicePersonService personService = injector.getInstance(GuicePersonService.class);
assertNotNull(personService);
```

### 4.3.设定器或方法在春天注入

**在基于 setter 的依赖注入中，容器在调用构造函数实例化组件后，会调用类的 setter 方法。**

假设我们希望 Spring 使用 setter 方法自动连接一个依赖项。我们可以用`@Autowired`来注释 setter 方法:

```
@Component
public class SpringPersonService {

    private PersonDao personDao;

    @Autowired
    public void setPersonDao(PersonDao personDao) {
        this.personDao = personDao;
    }
}
```

每当我们需要一个`SpringPersonService`类的实例时，Spring 就会通过调用`setPersonDao()`方法自动连接`personDao`字段。

我们可以得到一个`SpringPersonService` bean，并在下面的测试中访问它的`personDao`字段:

```
SpringPersonService personService = context.getBean(SpringPersonService.class);
assertNotNull(personService);
assertNotNull(personService.getPersonDao());
```

### 4.4。Guice 中的 Setter 或方法注入

我们将简单地稍微改变一下我们的例子，以在 Guice 中实现 **setter 注入。**

```
public class GuicePersonService {

    private PersonDao personDao;

    @Inject
    public void setPersonDao(PersonDao personDao) {
        this.personDao = personDao;
    }
}
```

每当我们从注入器获得一个`GuicePersonService`类的实例时，我们将把`personDao`字段传递给上面的 setter 方法。

下面是我们如何创建一个`GuicePersonService`类的实例，并在测试中访问它的`personDao`字段:

```
GuicePersonService personService = injector.getInstance(GuicePersonService.class);
assertNotNull(personService);
assertNotNull(personService.getPersonDao());
```

### 4.5.春季野外注射

在我们所有的例子中，我们已经看到了如何为 Spring 和 Guice 应用字段注入。所以，对我们来说这不是一个新概念。但为了完整起见，我们还是再列出来。

**在基于字段的依赖注入的情况下，我们通过用`@Autowired`或`@Inject`标记它们来注入依赖。**

### 4.6。现场注入导管

正如我们在上一节中提到的，我们已经使用`@Inject` 介绍了 Guice 的**字段注入。**

## 5.结论

在本教程中，我们探讨了 Guice 和 Spring 框架在实现依赖注入方面的几个核心差异。和往常一样， [Guice](https://web.archive.org/web/20220930092230/https://github.com/eugenp/tutorials/tree/master/guice) 和 [Spring](https://web.archive.org/web/20220930092230/https://github.com/eugenp/tutorials/tree/master/spring-di) 代码示例在 GitHub 上结束。