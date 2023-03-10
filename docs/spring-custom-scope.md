# 春季自定义范围

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-custom-scope>

## 1。概述

Spring 提供了两个现成的标准 bean 作用域(`“singleton”`和`“prototype”`)，可以在任何 Spring 应用程序中使用，另外还有三个 bean 作用域(`“request”`、`“session”`和`“globalSession”`)，只能在 web 感知应用程序中使用。

标准的 bean 作用域不能被覆盖，覆盖 web 感知的作用域通常被认为是一种不好的做法。但是，您可能有一个应用程序需要与所提供的范围不同的或额外的功能。

例如，如果您正在开发一个多租户系统，您可能希望为每个租户提供一个特定 bean 或一组 bean 的单独实例。Spring 为这样的场景提供了一种创建自定义范围的机制。

在这个快速教程中，我们将演示**如何在 Spring 应用程序**中创建、注册和使用自定义作用域。

## 2。创建自定义范围类

为了创建一个自定义范围，**我们必须实现 [`Scope`接口](https://web.archive.org/web/20220626074922/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/Scope.html)** 。在这样做的时候，我们还必须**确保实现是线程安全的**，因为作用域可以同时被多个 bean 工厂使用。

### 2.1。管理作用域对象和回调

当实现一个定制的`Scope`类时，首先要考虑的事情之一是如何存储和管理作用域对象和析构回调。例如，这可以使用映射或专用类来完成。

对于本文，我们将使用同步映射以线程安全的方式来实现这一点。

让我们开始定义我们的自定义范围类:

```java
public class TenantScope implements Scope {
    private Map<String, Object> scopedObjects
      = Collections.synchronizedMap(new HashMap<String, Object>());
    private Map<String, Runnable> destructionCallbacks
      = Collections.synchronizedMap(new HashMap<String, Runnable>());
...
}
```

### 2.2。从作用域中检索对象

为了从我们的作用域中按名称检索一个对象，让我们实现`getObject`方法。正如 JavaDoc 所声明的，**如果命名的对象不存在于作用域中，这个方法必须创建并返回一个新的对象**。

在我们的实现中，我们检查命名对象是否在我们的地图中。如果是，我们返回它，如果不是，我们使用`ObjectFactory`创建一个新对象，将它添加到我们的地图中，并返回它:

```java
@Override
public Object get(String name, ObjectFactory<?> objectFactory) {
    if(!scopedObjects.containsKey(name)) {
        scopedObjects.put(name, objectFactory.getObject());
    }
    return scopedObjects.get(name);
}
```

在由`Scope`接口定义的五个方法中，**只有`get`方法需要具有所描述行为的完整实现**。其他四个方法是可选的，如果它们不需要或者不支持某个功能，可能会抛出`UnsupportedOperationException`。

### 2.3。注册销毁回调

我们还必须实现`registerDestructionCallback`方法。此方法提供了一个回调函数，当命名对象被销毁或者作用域本身被应用程序销毁时，将执行该回调函数:

```java
@Override
public void registerDestructionCallback(String name, Runnable callback) {
    destructionCallbacks.put(name, callback);
}
```

### 2.4。从范围中移除对象

接下来，让我们实现`remove`方法，该方法从作用域中移除命名对象，同时移除其注册的销毁回调，返回移除的对象:

```java
@Override
public Object remove(String name) {
    destructionCallbacks.remove(name);
    return scopedObjects.remove(name);
}
```

注意**实际执行回调并销毁移除的对象**是调用者的责任。

### 2.5。获取对话 ID

现在，让我们实现`getConversationId`方法。如果您的作用域支持对话 ID 的概念，您应该在此处返回它。否则，约定是返回`null`:

```java
@Override
public String getConversationId() {
    return "tenant";
}
```

### 2.6。解析上下文对象

最后，让我们实现`resolveContextualObject`方法。如果您的范围支持多个上下文对象，那么您应该将每个对象与一个键值相关联，并且您应该返回对应于所提供的`key`参数的对象。否则，惯例是退回`null`:

```java
@Override
public Object resolveContextualObject(String key) {
    return null;
}
```

## 3。注册自定义范围

为了让 Spring 容器知道您的新范围，您需要通过`ConfigurableBeanFactory`实例上的`registerScope`方法将它注册到**中。让我们来看看这个方法的定义:**

```java
void registerScope(String scopeName, Scope scope);
```

第一个参数`scopeName`用于通过唯一的名称来标识/指定范围。第二个参数`scope`，是您希望注册和使用的自定义`Scope`实现的实际实例。

让我们创建一个定制的`BeanFactoryPostProcessor`并使用一个`ConfigurableListableBeanFactory`注册我们的定制范围:

```java
public class TenantBeanFactoryPostProcessor implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory factory) throws BeansException {
        factory.registerScope("tenant", new TenantScope());
    }
}
```

现在，让我们编写一个 Spring 配置类来加载我们的`BeanFactoryPostProcessor`实现:

```java
@Configuration
public class TenantScopeConfig {

    @Bean
    public static BeanFactoryPostProcessor beanFactoryPostProcessor() {
        return new TenantBeanFactoryPostProcessor();
    }
}
```

## 4。使用自定义范围

现在我们已经注册了我们的自定义作用域，我们可以将它应用于我们的任何 bean，就像我们应用于使用除了`singleton`(默认作用域)以外的作用域的任何其他 bean 一样——通过使用`@Scope`注释并通过名称指定我们的自定义作用域。

让我们创建一个简单的`TenantBean`类——稍后我们将声明这种类型的租户范围的 beans:

```java
public class TenantBean {

    private final String name;

    public TenantBean(String name) {
        this.name = name;
    }

    public void sayHello() {
        System.out.println(
          String.format("Hello from %s of type %s",
          this.name, 
          this.getClass().getName()));
    }
}
```

注意，我们没有在这个类上使用类级别的`@Component`和`@Scope`注释。

现在，让我们在配置类中定义一些租户范围的 beans:

```java
@Configuration
public class TenantBeansConfig {

    @Scope(scopeName = "tenant")
    @Bean
    public TenantBean foo() {
        return new TenantBean("foo");
    }

    @Scope(scopeName = "tenant")
    @Bean
    public TenantBean bar() {
        return new TenantBean("bar");
    }
}
```

## 5。测试自定义范围

让我们编写一个测试来测试我们的自定义范围配置，方法是加载一个`ApplicationContext`，注册我们的`Configuration`类，并检索我们的租户范围 beans:

```java
@Test
public final void whenRegisterScopeAndBeans_thenContextContainsFooAndBar() {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    try{
        ctx.register(TenantScopeConfig.class);
        ctx.register(TenantBeansConfig.class);
        ctx.refresh();

        TenantBean foo = (TenantBean) ctx.getBean("foo", TenantBean.class);
        foo.sayHello();
        TenantBean bar = (TenantBean) ctx.getBean("bar", TenantBean.class);
        bar.sayHello();
        Map<String, TenantBean> foos = ctx.getBeansOfType(TenantBean.class);

        assertThat(foo, not(equalTo(bar)));
        assertThat(foos.size(), equalTo(2));
        assertTrue(foos.containsValue(foo));
        assertTrue(foos.containsValue(bar));

        BeanDefinition fooDefinition = ctx.getBeanDefinition("foo");
        BeanDefinition barDefinition = ctx.getBeanDefinition("bar");

        assertThat(fooDefinition.getScope(), equalTo("tenant"));
        assertThat(barDefinition.getScope(), equalTo("tenant"));
    }
    finally {
        ctx.close();
    }
}
```

我们测试的结果是:

```java
Hello from foo of type org.baeldung.customscope.TenantBean
Hello from bar of type org.baeldung.customscope.TenantBean
```

## 6。结论

在这个快速教程中，我们展示了如何在 Spring 中定义、注册和使用自定义范围。

你可以在 [Spring 框架参考](https://web.archive.org/web/20220626074922/https://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-factory-scopes-custom)中阅读更多关于自定义作用域的内容。你也可以在 GitHub 上的 [Spring 框架库中看看 Spring 对各种`Scope`类的实现。](https://web.archive.org/web/20220626074922/https://github.com/spring-projects/spring-framework)

像往常一样，你可以在 [GitHub 项目](https://web.archive.org/web/20220626074922/https://github.com/eugenp/tutorials/tree/master/spring-core-3)上找到本文中使用的代码示例。