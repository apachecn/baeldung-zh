# 用 Spring 注释实例化同一个类的多个 Beans

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-same-class-multiple-beans>

## 1.概观

Spring IoC 容器创建并管理 Spring beans，它是我们应用程序的核心。创建 bean 的实例等同于从普通 Java 类创建对象。然而，生成同一个类的几个 beans 是有挑战性的。

在本教程中，我们将学习如何在 Spring 框架中使用注释来创建同一个类的多个 beans。

## 2.使用 Java 配置

这是使用注释创建同一个类的多个 beans 的最简单和最容易的方法。在这种方法中，我们将使用一个基于 Java 的配置类来配置同一个类的多个 beans。

让我们考虑一个简单的例子。我们有一个`Person`类，它有两个类成员`firstName`和 `lastName`:

```java
public class Person {
    private String firstName;
    private String lastName;

    public Person(String firstName, String secondName) {
        super();
        this.firstName = firstName;
        this.lastName = secondName;
    }

    @Override
    public String toString() {
        return "Person [firstName=" + firstName + ", secondName=" + lastName + "]";
    }
}
```

接下来，我们将构建一个名为`PersonConfig`的配置类，并在其中定义多个`Person`类的 beans:

```java
@Configuration
public class PersonConfig {
    @Bean
    public Person personOne() {
        return new Person("Harold", "Finch");
    }

    @Bean
    public Person personTwo() {
        return new Person("John", "Reese");
    }
}
```

**这里，** **[`@Bean`](/web/20220820090132/https://www.baeldung.com/spring-bean) 实例化两个 id 与方法名相同的 beans，并在`BeanFactory` (Spring 容器)接口**中注册。接下来，我们可以初始化 Spring 容器，并从 Spring 容器中请求任何 beans。

这种策略也使得实现[依赖注入](/web/20220820090132/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)变得简单。我们可以直接将一个 bean，比如说`personOne,`注入同类型的另一个 bean，比如说`personTwo`使用[自动连线](/web/20220820090132/https://www.baeldung.com/spring-autowire)。

**这种方法的局限性在于，在典型的基于 Java 的配置风格中，我们需要使用`new`关键字手动实例化 beans。**

**所以，如果同一个类的 bean 数量增加，我们需要先注册，在配置类中创建 bean。**这使得它更像是一种特定于 Java 的方法，而不是特定于 Spring 的方法。

## 3.使用`@Component`注释

在这种方法中，我们将使用 [`@Component`](/web/20220820090132/https://www.baeldung.com/spring-component-annotation) 注释来创建多个从`Person`类继承其属性的 beans。

首先，我们将创建多个子类，即扩展`Person`超类的`PersonOne`和`PersonTwo,`:

```java
@Component
public class PersonOne extends Person {

    public PersonOne() {
        super("Harold", "Finch");
    }
}
```

```java
@Component
public class PersonTwo extends Person {

    public PersonTwo() {
        super("John", "Reese");
    }
}
```

接下来，在`PersonConfig`文件中，我们将使用 [`@ComponentScan`](/web/20220820090132/https://www.baeldung.com/spring-component-scanning) 注释来启用对整个包的组件扫描。**这使得 Spring 容器能够自动创建用`@Component` :** 注释的任何类的 beans

```java
@Configuration
@ComponentScan("com.baeldung.multibeaninstantiation.solution2")
public class PersonConfig {

}
```

现在，我们可以使用 Spring 容器中的`PersonOne`或`PersonTwo`bean。在其他任何地方，我们都可以使用`Person` 类 bean。

这种方法的问题在于它不会创建同一个类的多个实例。相反，它创建从超类继承属性的类的 beans。因此，我们只能在继承的类没有定义任何附加属性的情况下使用这个解决方案。**此外，** **继承的使用增加了代码的整体复杂度。**

## 4.使用`BeanFactoryPostProcessor`

**第三种也是最后一种方法利用 [`BeanFactoryPostProcessor`](/web/20220820090132/https://www.baeldung.com/spring-beanpostprocessor) 接口的定制实现来创建同一个类**的多个 bean 实例。这可以通过以下步骤实现:

*   创建一个定制 bean 类，并使用`FactoryBean`接口对其进行配置
*   使用`BeanFactoryPostProcessor`接口实例化多个相同类型的 beans

### 4.1.自定义 Bean 实现

为了更好地理解这种方法，我们将进一步扩展这个例子。假设有一个`Human` 类依赖于`Person`类的多个实例:

```java
public class Human implements InitializingBean {

    private Person personOne;

    private Person personTwo;

    @Override
    public void afterPropertiesSet() throws Exception {
        Assert.notNull(personOne, "Harold is alive!");
        Assert.notNull(personTwo, "John is alive!");
    }

    /* Setter injection */
    @Autowired
    public void setPersonOne(Person personOne) {
        this.personOne = personOne;
        this.personOne.setFirstName("Harold");
        this.personOne.setSecondName("Finch");
    }

    @Autowired
    public void setPersonTwo(Person personTwo) {
        this.personTwo = personTwo;
        this.personTwo.setFirstName("John");
        this.personTwo.setSecondName("Reese");
    }
}
```

**[`InitializingBean`](https://web.archive.org/web/20220820090132/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/InitializingBean.html)接口调用`afterPropertiesSet()`方法来检查`BeanFactory`是否已经设置了所有的 bean 属性并满足其他依赖**。此外，我们使用 setter 注入来注入和初始化两个`Person`类 bean`personOne`和`personTwo`。

接下来，我们将创建一个实现 [`FactoryBean`](/web/20220820090132/https://www.baeldung.com/spring-factorybean) 接口的`Person`类。**一个`FactoryBean`充当在 IoC 容器中创建其他 beans 的工厂。**

该接口旨在创建实现它的 bean 的更多实例。在我们的例子中，它生成类型`Person`类的实例并自动配置它:

```java
@Qualifier(value = "personOne, personTwo")
public class Person implements FactoryBean<Object> {
    private String firstName;
    private String secondName;

    public Person() {
        // initialization code (optional)
    }

    @Override
    public Class<Person> getObjectType() {
        return Person.class;
    }

    @Override
    public Object getObject() throws Exception {
        return new Person();
    }

    public boolean isSingleton() {
        return true;
    }

    // code for getters & setters
}
```

**这里要注意的第二件重要的事情是** **对 [`@Qualifier`](/web/20220820090132/https://www.baeldung.com/spring-qualifier-annotation) 注释的使用，该注释包含类级别**的多个`Person`类型的名称或 bean ids。在这个例子中，在类级别使用`@Qualifier`是有原因的，我们接下来会看到。

### 4.2.自定义`BeanFactory`实现

现在，我们将使用一个自定义实现的 [`BeanFactoryPostProcessor`](https://web.archive.org/web/20220820090132/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanFactoryPostProcessor.html) 接口。**任何实现`BeanFactoryPostProcessor`的类都在任何 Spring bean 创建之前执行。**这允许我们配置和操作 bean 的生命周期。

**`BeanFactoryPostProcessor`扫描所有用`@Qualifier.` 注释的类。此外，它从注释中提取名称(bean ids)并手动创建具有指定名称的类类型的实例:**

```java
public class PersonFactoryPostProcessor implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        Map<String, Object> map = beanFactory.getBeansWithAnnotation(Qualifier.class);
        for (Map.Entry<String, Object> entry : map.entrySet()) {
            createInstances(beanFactory, entry.getKey(), entry.getValue());
        }
    }

    private void createInstances(ConfigurableListableBeanFactory beanFactory, String beanName, Object bean) {
        Qualifier qualifier = bean.getClass().getAnnotation(Qualifier.class);
        for (String name : extractNames(qualifier)) {
            Object newBean = beanFactory.getBean(beanName);
            beanFactory.registerSingleton(name.trim(), newBean);
        }
    }

    private String[] extractNames(Qualifier qualifier) {
        return qualifier.value().split(",");
    }
}
```

**在这里，** **一旦 Spring 容器被初始化**，定制的`BeanFactoryPostProcessor`实现就会被调用。

接下来，为了简单起见，我们将使用一个 Java 配置类来初始化自定义和`BeanFactory`实现:

```java
@Configuration
public class PersonConfig {
    @Bean
    public PersonFactoryPostProcessor PersonFactoryPostProcessor() {
        return new PersonFactoryPostProcessor();
    }

    @Bean
    public Person person() {
        return new Person();
    }

    @Bean
    public Human human() {
        return new Human();
    }
}
```

这种方法的局限性在于它的复杂性。此外，不鼓励使用它，因为这不是在典型的 Spring 应用程序中配置 beans 的自然方式。

尽管有这些限制，但这种方法更加具体于 Spring，并且可以使用注释实例化多个相似类型的 beans。

## 5.结论

在本文中，我们已经学习了使用 Spring 注解通过三种不同的方法实例化同一个类的多个 beans。

前两种方法是实例化多个 Spring beans 的简单且特定于 Java 的方法。然而，第三个有点棘手和复杂。但是，它服务于使用注释创建 bean 的目的。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20220820090132/https://github.com/eugenp/tutorials/tree/master/spring-core-6/)