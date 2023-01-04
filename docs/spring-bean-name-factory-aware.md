# Spring 中的 BeanNameAware 和 BeanFactoryAware 接口

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-bean-name-factory-aware>

## 1。概述

在这个快速教程中，**我们将关注 Spring 框架**中的`BeanNameAware`和`BeanFactoryAware`接口。

我们将分别描述每个接口及其用法的优缺点。

## 2。`Aware`界面

`BeanNameAware`和`BeanFactoryAware`都属于`org.springframework.beans.factory.Aware`根标记接口。这使用 setter 注入在应用程序上下文启动期间获取一个对象。

**`Aware`接口混合了回调、监听器和观察者设计模式**。它表明 bean 有资格通过回调方法得到 Spring 容器的通知。

## 3。`BeanNameAware`

**`BeanNameAware`让对象知道容器**中定义的 bean 名称。

让我们来看一个例子:

```
public class MyBeanName implements BeanNameAware {

    @Override
    public void setBeanName(String beanName) {
        System.out.println(beanName);
    }
}
```

`beanName`属性表示在 Spring 容器中注册的 bean id。在我们的实现中，我们只是显示 bean 名称。

接下来，让我们在 Spring 配置类中注册一个这种类型的 bean:

```
@Configuration
public class Config {

    @Bean(name = "myCustomBeanName")
    public MyBeanName getMyBeanName() {
        return new MyBeanName();
    }
}
```

这里我们已经用`@Bean(name` = `“myCustomBeanName”)`行明确地给我们的`MyBeanName`类指定了一个名称。

现在，我们可以启动应用程序上下文并从中获取 bean:

```
AnnotationConfigApplicationContext context 
  = new AnnotationConfigApplicationContext(Config.class);

MyBeanName myBeanName = context.getBean(MyBeanName.class);
```

正如我们所料，`setBeanName`方法打印出了`“myCustomBeanName”`。

如果我们从`@Bean`注释中移除`name = “…”`代码，在这种情况下，容器会将`getMyBeanName()`方法名分配给 bean。所以输出会是`“getMyBeanName”`。

## 4。`BeanFactoryAware`

**`BeanFactoryAware`用于注入`BeanFactory`对象**。这样我们就可以访问创建这个对象的`BeanFactory`。

下面是一个`MyBeanFactory`类的例子:

```
public class MyBeanFactory implements BeanFactoryAware {

    private BeanFactory beanFactory;

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }

    public void getMyBeanName() {
        MyBeanName myBeanName = beanFactory.getBean(MyBeanName.class);
        System.out.println(beanFactory.isSingleton("myCustomBeanName"));
    }
}
```

在`setBeanFactory()`方法的帮助下，我们将来自 IoC 容器的`BeanFactory`引用分配给`beanFactory property`。

之后，我们可以像在`getMyBeanName()`函数中一样直接使用它。

让我们初始化`MyBeanFactory`并调用`getMyBeanName()`方法:

```
MyBeanFactory myBeanFactory = context.getBean(MyBeanFactory.class);
myBeanFactory.getMyBeanName();
```

由于我们已经在前面的例子中实例化了`MyBeanName`类，Spring 将在这里调用现有的实例。

`beanFactory.isSingleton(“myCustomBeanName”)`行验证了这一点。

## 5。什么时候用？

`BeanNameAware`的典型用例可能是获取 bean 名称用于日志记录或连接目的。对于`BeanFactoryAware`来说，可以使用遗留代码中的 spring bean。

在大多数情况下，我们应该避免使用任何`Aware`接口，除非我们需要它们。**实现这些接口会将代码耦合到 Spring 框架。**

## 6。结论

在这篇文章中，我们学习了`BeanNameAware`和`BeanFactoryAware`接口以及如何在实践中使用它们。

和往常一样，这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220626205725/https://github.com/eugenp/tutorials/tree/master/spring-core)