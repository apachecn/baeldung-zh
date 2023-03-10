# 通过工厂方法创建 Spring Beans

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-beans-factory-methods>

## 1.介绍

工厂方法对于在单个方法调用中隐藏复杂的创建逻辑来说是一种有用的技术。

虽然我们通常使用[构造函数](/web/20220706111745/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring#constructor-based-dependency-injection)或[字段注入](/web/20220706111745/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring#field-based-dependency-injection)、**在 Spring 中创建 bean，但是我们也可以使用工厂方法**创建[Spring bean](/web/20220706111745/https://www.baeldung.com/spring-bean)。

在本教程中，我们将深入研究使用实例和静态工厂方法创建 Spring beans。

## 2.实例工厂方法

工厂方法模式的标准实现是创建一个返回所需 bean 的实例方法。

此外，**我们可以配置 Spring 来创建我们想要的 bean，不管有没有参数**。

### 2.1.没有争论

我们可以创建一个`Foo`类来表示我们正在创建的 bean:

```java
public class Foo {}
```

然后，我们创建一个包含工厂方法`createInstance`的`InstanceFooFactory`类，它创建了我们的`Foo ` bean:

```java
public class InstanceFooFactory {

    public Foo createInstance() {
        return new Foo();
    }
}
```

之后，我们配置 Spring:

1.  为我们的工厂类(`InstanceFooFactory`)创建一个 bean
2.  使用`factory-bean`属性来引用我们的工厂 bean
3.  使用`factory-method`属性来引用我们的工厂方法(`createInstance`

将此应用于 Spring XML 配置，我们最终会得到:

```java
<beans ...>

    <bean id="instanceFooFactory"
      class="com.baeldung.factorymethod.InstanceFooFactory" />

    <bean id="foo"
      factory-bean="instanceFooFactory"
      factory-method="createInstance" />

</beans>
```

最后，我们自动连接我们想要的`Foo` bean。然后 Spring 将使用我们的`createInstance`工厂方法创建我们的 bean:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("/factorymethod/instance-config.xml")
public class InstanceFooFactoryIntegrationTest {

    @Autowired
    private Foo foo;

    @Test
    public void givenValidInstanceFactoryConfig_whenCreateFooInstance_thenInstanceIsNotNull() {
        assertNotNull(foo);
    }
}
```

### 2.2.带参数

**我们还可以在 Spring 配置中使用`constructor-arg`元素**为实例工厂方法提供参数。

首先，我们创建一个类`Bar`，它利用了一个参数:

```java
public class Bar {

    private String name;

    public Bar(String name) {
        this.name = name;
    }

    // ...getters & setters
}
```

接下来，我们创建一个实例工厂类`InstanceBarFactory`，它具有一个工厂方法，该方法接受一个参数并返回一个`Bar` bean:

```java
public class InstanceBarFactory {

    public Bar createInstance(String name) {
        return new Bar(name);
    }
}
```

最后，我们将一个`constructor-arg`元素添加到我们的`Bar` bean 定义中:

```java
<beans ...>

    <bean id="instanceBarFactory"
      class="com.baeldung.factorymethod.InstanceBarFactory" />

    <bean id="bar"
      factory-bean="instanceBarFactory"
      factory-method="createInstance">
        <constructor-arg value="someName" />
    </bean>

</beans>
```

然后，我们可以像对`Foo` bean 那样自动连接`Bar` bean:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("/factorymethod/instance-bar-config.xml")
public class InstanceBarFactoryIntegrationTest {

    @Autowired
    private Bar instance;

    @Test
    public void givenValidInstanceFactoryConfig_whenCreateInstance_thenNameIsCorrect() {
        assertNotNull(instance);
        assertEquals("someName", instance.getName());
    }
}
```

## 3.静态工厂方法

我们还可以将 Spring 配置为使用静态方法作为工厂方法。

虽然实例工厂方法应该是首选，但是如果我们已经有了生成所需 beans 的现有遗留静态方法，这种技术会很有用。例如，如果一个工厂方法返回一个 singleton，我们可以配置 Spring 来使用这个 singleton 工厂方法。

类似于实例工厂方法，我们可以配置带参数和不带参数的静态方法。

### 3.1.没有争论

使用我们的`Foo`类作为我们想要的 bean，我们可以创建一个类`SingletonFooFactory`，它包含一个`createInstance`工厂方法，该方法返回一个`Foo`的单独实例:

```java
public class SingletonFooFactory {

    private static final Foo INSTANCE = new Foo();

    public static Foo createInstance() {
        return INSTANCE;
    }
}
```

这一次，**我们只需要创建一个 bean。**这个 bean 只需要两个属性:

1.  `class`–声明我们的工厂类(`SingletonFooFactory`)
2.  `factory-method`–声明静态工厂方法(`createInstance`)

将此应用于我们的 Spring XML 配置，我们得到:

```java
<beans ...>

    <bean id="foo"
      class="com.baeldung.factorymethod.SingletonFooFactory"
      factory-method="createInstance" />

</beans>
```

最后，我们使用与之前相同的结构自动连接我们的`Foo` bean:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("/factorymethod/static-foo-config.xml")
public class SingletonFooFactoryIntegrationTest {

    @Autowired
    private Foo singleton;

    @Test
    public void givenValidStaticFactoryConfig_whenCreateInstance_thenInstanceIsNotNull() {
        assertNotNull(singleton);
    }
}
```

### 3.2.带参数

虽然**我们应该避免改变静态对象的状态——比如我们的 singleton——但是如果可能的话**,我们仍然可以将参数传递给我们的静态工厂方法。

为此，我们创建了一个新的工厂方法，它接受我们想要的参数:

```java
public class SingletonBarFactory {

    private static final Bar INSTANCE = new Bar("unnamed");

    public static Bar createInstance(String name) {
        INSTANCE.setName(name);
        return INSTANCE;
    }
}
```

之后，我们使用`constructor-arg`元素配置 Spring 来传入所需的参数:

```java
<beans ...>

    <bean id="bar"
      class="com.baeldung.factorymethod.SingletonBarFactory"
      factory-method="createInstance">
        <constructor-arg value="someName" />
    </bean>

</beans>
```

最后，我们使用与之前相同的结构自动连接我们的`Bar` bean:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("/factorymethod/static-bar-config.xml")
public class SingletonBarFactoryIntegrationTest {

    @Autowired
    private Bar instance;

    @Test
    public void givenValidStaticFactoryConfig_whenCreateInstance_thenNameIsCorrect() {
        assertNotNull(instance);
        assertEquals("someName", instance.getName());
    }
}
```

## 4.结论

在本文中，我们研究了如何配置 Spring 来使用实例和静态工厂方法——有参数和没有参数都可以。

虽然通过构造函数和字段注入创建 beans 更常见，但工厂方法对于复杂的创建步骤和遗留代码来说可能很方便。

本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220706111745/https://github.com/eugenp/tutorials/tree/master/spring-core-4)