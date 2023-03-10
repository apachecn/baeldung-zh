# 如何使用 Spring FactoryBean？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-factorybean>

## 1。概述

春豆容器中有两种豆:普通豆和工厂豆。Spring 直接使用前者，而后者可以自己产生对象，由框架管理。

简单地说，我们可以通过实现`org.springframework.beans.factory.FactoryBean`接口来构建工厂 bean。

## 2。工厂豆的基本知识

### 2.1。实施一个`FactoryBean`

我们先来看一下`FactoryBean`界面:

```java
public interface FactoryBean {
    T getObject() throws Exception;
    Class<?> getObjectType();
    boolean isSingleton();
}
```

让我们讨论三种方法:

*   `getObject()`–返回工厂生产的对象，这是 Spring 容器将要使用的对象
*   `getObjectType()`–返回这个`FactoryBean`产生的对象的类型
*   `isSingleton()`–表示此`FactoryBean`产生的对象是否为单例对象

现在，让我们实现一个例子`FactoryBean`。我们将实现一个产生`Tool`类型对象的`ToolFactory` :

```java
public class Tool {

    private int id;

    // standard constructors, getters and setters
}
```

`ToolFactory`本身:

```java
public class ToolFactory implements FactoryBean<Tool> {

    private int factoryId;
    private int toolId;

    @Override
    public Tool getObject() throws Exception {
        return new Tool(toolId);
    }

    @Override
    public Class<?> getObjectType() {
        return Tool.class;
    }

    @Override
    public boolean isSingleton() {
        return false;
    }

    // standard setters and getters
}
```

我们可以看到，`ToolFactory`是一个`FactoryBean`，它可以产生`Tool`对象。

### 2.2。使用`FactoryBean`和基于 XML 的配置

现在让我们来看看如何使用我们的`ToolFactory`。

我们将开始构建一个基于 XML 配置的工具—`factorybean-spring-ctx.xml`:

```java
<beans ...>

    <bean id="tool" class="com.baeldung.factorybean.ToolFactory">
        <property name="factoryId" value="9090"/>
        <property name="toolId" value="1"/>
    </bean>
</beans>
```

接下来，我们可以测试`Tool`对象是否被正确注入:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath:factorybean-spring-ctx.xml" })
public class FactoryBeanXmlConfigTest {
    @Autowired
    private Tool tool;

    @Test
    public void testConstructWorkerByXml() {
        assertThat(tool.getId(), equalTo(1));
    }
}
```

测试结果显示，我们成功地用我们在`factorybean-spring-ctx.xml`中配置的属性注入了由`ToolFactory`产生的工具对象。

测试结果还显示，Spring 容器使用由`FactoryBean`产生的对象而不是它自己来进行依赖注入。

尽管 Spring 容器使用`FactoryBean`的`getObject()`方法的返回值作为 bean，但是您也可以使用`FactoryBean`本身。

**要访问`FactoryBean`，只需在 bean 名称前添加一个“&”。**

让我们尝试获取工厂 bean 及其`factoryId`属性:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath:factorybean-spring-ctx.xml" })
public class FactoryBeanXmlConfigTest {

    @Resource(name = "&tool;")
    private ToolFactory toolFactory;

    @Test
    public void testConstructWorkerByXml() {
        assertThat(toolFactory.getFactoryId(), equalTo(9090));
    }
}
```

### 2.3。使用`FactoryBean`和基于 Java 的配置

将`FactoryBean`用于基于 Java 的配置与基于 XML 的配置略有不同，您必须显式调用`FactoryBean`的`getObject()`方法。

让我们将前一小节中的示例转换成一个基于 Java 的配置示例:

```java
@Configuration
public class FactoryBeanAppConfig {

    @Bean(name = "tool")
    public ToolFactory toolFactory() {
        ToolFactory factory = new ToolFactory();
        factory.setFactoryId(7070);
        factory.setToolId(2);
        return factory;
    }

    @Bean
    public Tool tool() throws Exception {
        return toolFactory().getObject();
    }
}
```

然后，我们测试`Tool`对象是否被正确注入:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = FactoryBeanAppConfig.class)
public class FactoryBeanJavaConfigTest {

    @Autowired
    private Tool tool;

    @Resource(name = "&tool;")
    private ToolFactory toolFactory;

    @Test
    public void testConstructWorkerByJava() {
        assertThat(tool.getId(), equalTo(2));
        assertThat(toolFactory.getFactoryId(), equalTo(7070));
    }
}
```

测试结果显示了与前面基于 XML 的配置测试相似的效果。

## 3。初始化的方法

有时你需要在设置了`FactoryBean`之后，调用`getObject()`方法之前执行一些操作，比如属性检查。

您可以通过实现`InitializingBean`接口或使用`@PostConstruct`注释来实现这一点。

关于使用这两种解决方案的更多细节已经在另一篇文章中介绍过:[Spring](/web/20220122144103/https://www.baeldung.com/running-setup-logic-on-startup-in-spring)启动时运行逻辑指南。

## 4。`AbstractFactoryBean`

Spring 提供了`AbstractFactoryBean`作为`FactoryBean`实现的简单模板超类。有了这个基类，我们现在可以更方便地实现一个工厂 bean 来创建一个单例或原型对象。

让我们实现一个`SingleToolFactory`和一个`NonSingleToolFactory`来展示如何为单例类型和原型类型使用`AbstractFactoryBean`:

```java
public class SingleToolFactory extends AbstractFactoryBean<Tool> {

    private int factoryId;
    private int toolId;

    @Override
    public Class<?> getObjectType() {
        return Tool.class;
    }

    @Override
    protected Tool createInstance() throws Exception {
        return new Tool(toolId);
    }

    // standard setters and getters
}
```

现在是非单一实现:

```java
public class NonSingleToolFactory extends AbstractFactoryBean<Tool> {

    private int factoryId;
    private int toolId;

    public NonSingleToolFactory() {
        setSingleton(false);
    }

    @Override
    public Class<?> getObjectType() {
        return Tool.class;
    }

    @Override
    protected Tool createInstance() throws Exception {
        return new Tool(toolId);
    }

    // standard setters and getters
}
```

此外，这些工厂 beans 的 XML 配置:

```java
<beans ...>

    <bean id="singleTool" class="com.baeldung.factorybean.SingleToolFactory">
        <property name="factoryId" value="3001"/>
        <property name="toolId" value="1"/>
    </bean>

    <bean id="nonSingleTool" class="com.baeldung.factorybean.NonSingleToolFactory">
        <property name="factoryId" value="3002"/>
        <property name="toolId" value="2"/>
    </bean>
</beans>
```

现在我们可以测试`Worker`对象的属性是否如我们预期的那样被注入:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath:factorybean-abstract-spring-ctx.xml" })
public class AbstractFactoryBeanTest {

    @Resource(name = "singleTool")
    private Tool tool1;

    @Resource(name = "singleTool")
    private Tool tool2;

    @Resource(name = "nonSingleTool")
    private Tool tool3;

    @Resource(name = "nonSingleTool")
    private Tool tool4;

    @Test
    public void testSingleToolFactory() {
        assertThat(tool1.getId(), equalTo(1));
        assertTrue(tool1 == tool2);
    }

    @Test
    public void testNonSingleToolFactory() {
        assertThat(tool3.getId(), equalTo(2));
        assertThat(tool4.getId(), equalTo(2));
        assertTrue(tool3 != tool4);
    }
}
```

正如我们从测试中看到的，`SingleToolFactory`产生单例对象，而`NonSingleToolFactory`产生原型对象。

注意，不需要在`SingleToolFactory`中设置 singleton 属性，因为在`AbstractFactory`中，singleton 属性的默认值是`true`。

## 5。结论

使用`FactoryBean`可以很好地封装复杂的构造逻辑，或者使在 Spring 中配置高度可配置的对象变得更容易。

因此，在本文中，我们介绍了如何实现我们的`FactoryBean`的基础知识，如何在基于 XML 的配置和基于 Java 的配置中使用它，以及`FactoryBean`的一些其他方面，比如`FactoryBean`和`AbstractFactoryBean`的初始化。

和往常一样，完整的源代码在 GitHub 上[。](https://web.archive.org/web/20220122144103/https://github.com/eugenp/tutorials/tree/master/spring-core-3)