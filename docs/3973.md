# BeanFactory 和 ApplicationContext 之间的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-beanfactory-vs-applicationcontext>

## 1.概观

Spring 框架附带了两个 IOC 容器—`[BeanFactory](/web/20221001115718/https://www.baeldung.com/spring-beanfactory)`和`[ApplicationContext](/web/20221001115718/https://www.baeldung.com/spring-classpathxmlapplicationcontext)`。`BeanFactory` 是 IOC 容器的最基本版本，`ApplicationContext` 扩展了`BeanFactory`的特性。

在这个快速教程中，我们将通过实例来理解这两个 IOC 容器之间的显著差异。

## 2.延迟加载与急切加载

**`BeanFactory`按需加载 bean，而`ApplicationContext`在启动**时加载所有 bean。因此，`BeanFactory`与`ApplicationContext`相比是轻量级的。我们用一个例子来理解一下。

### 2.1.用`BeanFactory`延迟加载

假设我们有一个名为`Student` 的[单例 bean](/web/20221001115718/https://www.baeldung.com/spring-bean-scopes#singleton) 类，它有一个方法:

```
public class Student {
    public static boolean isBeanInstantiated = false;

    public void postConstruct() {
        setBeanInstantiated(true);
    }

    //standard setters and getters
}
```

我们将`postConstruct()`方法定义为`BeanFactory`配置文件中的 [`init-method`](/web/20221001115718/https://www.baeldung.com/running-setup-logic-on-startup-in-spring#4-the-bean-initmethod-attribute) ，`ioc-container-difference-example.xml`:

```
<bean id="student" class="com.baeldung.ioccontainer.bean.Student" init-method="postConstruct"/>
```

现在，让我们编写一个测试用例，创建一个`BeanFactory`来检查它是否加载了`Student` bean:

```
@Test
public void whenBFInitialized_thenStudentNotInitialized() {
    Resource res = new ClassPathResource("ioc-container-difference-example.xml");
    BeanFactory factory = new XmlBeanFactory(res);

    assertFalse(Student.isBeanInstantiated());
}
```

这里，**`Student`对象未被初始化**。换句话说，**只有`BeanFactory`被初始化**。只有当我们显式调用`getBean()`方法`.`时，才会加载在我们的`BeanFactory`中定义的 beans

让我们检查一下我们手动调用`getBean()`方法的`Student` bean 的初始化:

```
@Test
public void whenBFInitialized_thenStudentInitialized() {
    Resource res = new ClassPathResource("ioc-container-difference-example.xml");
    BeanFactory factory = new XmlBeanFactory(res);
    Student student = (Student) factory.getBean("student");

    assertTrue(Student.isBeanInstantiated());
}
```

这里，`Student` bean 加载成功。因此，`BeanFactory` 只在需要的时候加载 bean。

### 2.2.用`ApplicationContext`急切加载

现在，让我们用`ApplicationContext` 代替`BeanFactory.`

我们将只定义`ApplicationContext,` ,它将通过使用急切加载策略来立即加载所有的 beans:

```
@Test
public void whenAppContInitialized_thenStudentInitialized() {
    ApplicationContext context = new ClassPathXmlApplicationContext("ioc-container-difference-example.xml");

    assertTrue(Student.isBeanInstantiated());
}
```

**在这里，`Student`对象被创建，即使我们没有调用`getBean()`方法。**

被认为是一个沉重的 IOC 容器，因为它的急切加载策略在启动时加载所有的 beans。相比之下，`BeanFactory`是轻量级的，在内存受限的系统中非常方便。然而，**我们将在接下来的章节中看到为什么`ApplicationContext`是大多数用例**的首选。

## 3.企业应用功能

`ApplicationContext` 以更加面向框架的风格增强了`BeanFactory`,并提供了几个适合企业应用程序的特性。

例如，它**提供了[消息传递(i18n 或国际化)](/web/20221001115718/https://www.baeldung.com/spring-classpathxmlapplicationcontext#2-internationalization-with-messagesource)** 功能、 **[事件发布](/web/20221001115718/https://www.baeldung.com/spring-events)** 功能、**基于注释的依赖注入**，以及**与 Spring AOP 特性的轻松集成**。

除此之外，`ApplicationContext`支持几乎所有类型的 bean 作用域，但是`BeanFactory`只支持两种作用域——`Singleton`和`Prototype`。因此，在构建复杂的企业应用程序时，最好使用`ApplicationContext` 。

## 4.`BeanFactoryPostProcessor`和`BeanPostProcessor`的自动注册

**T0 在启动时自动注册`BeanFactoryPostProcessor`和`BeanPostProcessor`T5。另一方面，`BeanFactory`不会自动注册这些接口。**

### 4.1.在`BeanFactory`中注册

为了理解，我们先写两个类。

首先，我们有`CustomBeanFactoryPostProcessor` 类，它实现了`BeanFactoryPostProcessor`:

```
public class CustomBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    private static boolean isBeanFactoryPostProcessorRegistered = false;

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory){
        setBeanFactoryPostProcessorRegistered(true);
    }

    // standard setters and getters
}
```

在这里，我们已经覆盖了 [`postProcessBeanFactory()`](https://web.archive.org/web/20221001115718/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanFactoryPostProcessor.html#postProcessBeanFactory-org.springframework.beans.factory.config.ConfigurableListableBeanFactory-) 方法来检查它的注册。

其次，我们有另一个类`CustomBeanPostProcessor`，它实现了`BeanPostProcessor`:

```
public class CustomBeanPostProcessor implements BeanPostProcessor {
    private static boolean isBeanPostProcessorRegistered = false;

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName){
        setBeanPostProcessorRegistered(true);
        return bean;
    }

    //standard setters and getters
}
```

在这里，我们已经覆盖了 [`postProcessBeforeInitialization()`](https://web.archive.org/web/20221001115718/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html#postProcessBeforeInitialization-java.lang.Object-java.lang.String-) 方法来检查它的注册。

此外，我们已经在`ioc-container-difference-example.xml` 配置文件中配置了这两个类:

```
<bean id="customBeanPostProcessor" 
  class="com.baeldung.ioccontainer.bean.CustomBeanPostProcessor" />
<bean id="customBeanFactoryPostProcessor" 
  class="com.baeldung.ioccontainer.bean.CustomBeanFactoryPostProcessor" />
```

让我们看一个测试案例，检查这两个类是否在启动时自动注册:

```
@Test
public void whenBFInitialized_thenBFPProcessorAndBPProcessorNotRegAutomatically() {
    Resource res = new ClassPathResource("ioc-container-difference-example.xml");
    ConfigurableListableBeanFactory factory = new XmlBeanFactory(res);

    assertFalse(CustomBeanFactoryPostProcessor.isBeanFactoryPostProcessorRegistered());
    assertFalse(CustomBeanPostProcessor.isBeanPostProcessorRegistered());
}
```

从我们的测试中可以看出，**自动注册没有发生**。

**现在，让我们看一个测试用例，它将它们手动添加到`BeanFactory` :**

```
@Test
public void whenBFPostProcessorAndBPProcessorRegisteredManually_thenReturnTrue() {
    Resource res = new ClassPathResource("ioc-container-difference-example.xml");
    ConfigurableListableBeanFactory factory = new XmlBeanFactory(res);

    CustomBeanFactoryPostProcessor beanFactoryPostProcessor 
      = new CustomBeanFactoryPostProcessor();
    beanFactoryPostProcessor.postProcessBeanFactory(factory);
    assertTrue(CustomBeanFactoryPostProcessor.isBeanFactoryPostProcessorRegistered());

    CustomBeanPostProcessor beanPostProcessor = new CustomBeanPostProcessor();
    factory.addBeanPostProcessor(beanPostProcessor);
    Student student = (Student) factory.getBean("student");
    assertTrue(CustomBeanPostProcessor.isBeanPostProcessorRegistered());
}
```

这里，我们使用 [`postProcessBeanFactory()`](https://web.archive.org/web/20221001115718/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanFactoryPostProcessor.html#postProcessBeanFactory-org.springframework.beans.factory.config.ConfigurableListableBeanFactory-) 方法注册`CustomBeanFactoryPostProcessor`，使用 [`addBeanPostProcessor()`](https://web.archive.org/web/20221001115718/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/ConfigurableBeanFactory.html#addBeanPostProcessor-org.springframework.beans.factory.config.BeanPostProcessor-) 方法注册`CustomBeanPostProcessor`。在这种情况下，两者都注册成功。

### 4.2.在`ApplicationContext`中注册

正如我们前面提到的，`ApplicationContext` 自动注册这两个类，无需编写额外的代码。

让我们在单元测试中验证这一行为:

```
@Test
public void whenAppContInitialized_thenBFPostProcessorAndBPostProcessorRegisteredAutomatically() {
    ApplicationContext context 
      = new ClassPathXmlApplicationContext("ioc-container-difference-example.xml");

    assertTrue(CustomBeanFactoryPostProcessor.isBeanFactoryPostProcessorRegistered());
    assertTrue(CustomBeanPostProcessor.isBeanPostProcessorRegistered());
}
```

正如我们所看到的，**在这种情况下，两个类的自动注册都是成功的**。

因此，**使用`ApplicationContext`** 总是明智的，因为 Spring 2.0(及以上版本)大量使用`BeanPostProcessor.`

同样值得注意的是**如果你使用的是普通的`BeanFactory,` ，那么像事务和 AOP 这样的特性将不会生效**(至少不会在不编写额外代码的情况下生效)。这可能会导致混乱，因为配置看起来没有任何问题。

## 5.结论

在本文中，我们已经通过实例看到了`ApplicationContext`和`BeanFactory`之间的主要区别。

`ApplicationContext` 带有高级功能，包括几个面向企业应用的功能，而`BeanFactory` 只有基本功能。因此，一般建议使用`ApplicationContext,` 和**，只有在内存消耗很关键的时候才应该使用`BeanFactory`。**

和往常一样，这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221001115718/https://github.com/eugenp/tutorials/tree/master/spring-core-3)