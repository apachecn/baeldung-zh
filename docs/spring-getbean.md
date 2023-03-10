# 理解 Spring 中的 getBean()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-getbean>

## 1.介绍

在本教程中，我们将经历`BeanFactory.getBean()`方法的不同变体。

简单地说，正如方法名所示，**这个** **负责从 Spring 容器**中检索一个 bean 实例。

## 2.Spring Beans 设置

首先，让我们定义几个用于测试的 Spring beans。有几种方法可以为 Spring 容器提供 bean 定义，但是在我们的例子中，我们将使用基于注释的 Java 配置:

```java
@Configuration
class AnnotationConfig {

    @Bean(name = {"tiger", "kitty"})
    @Scope(value = "prototype")
    Tiger getTiger(String name) {
        return new Tiger(name);
    }

    @Bean(name = "lion")
    Lion getLion() {
        return new Lion("Hardcoded lion name");
    }

    interface Animal {}
} 
```

我们已经创建了两个 beans。`Lion`具有默认的单例范围。`Tiger`被明确设置为[原型范围](/web/20221205124057/https://www.baeldung.com/spring-bean-scopes)。此外，请注意，我们为将在后续请求中使用的每个 bean 定义了名称。

## 3.`getBean()`API

[`BeanFactory`](/web/20221205124057/https://www.baeldung.com/spring-beanfactory) 提供了`getBean()`方法的五种不同的签名，我们将在下面的小节中研究这些签名。

### 3.1.按名称检索 Bean

让我们看看如何使用名称检索一个`Lion` bean 实例:

```java
Object lion = context.getBean("lion");

assertEquals(Lion.class, lion.getClass());
```

在这个变体中，我们提供一个名称，作为回报，如果应用程序上下文中存在具有给定名称的 bean，我们将获得一个`Object `类的实例。否则，如果 bean 查找失败，这个实现和所有其他实现都会抛出 [`NoSuchBeanDefinitionException`](/web/20221205124057/https://www.baeldung.com/spring-nosuchbeandefinitionexception) 。

主要的缺点是**在检索到 bean 之后，我们必须[将它转换成期望的类型](/web/20221205124057/https://www.baeldung.com/java-type-casting)。如果返回的 bean 的类型不同于我们预期的类型**，这可能会产生另一个异常 **。**

假设当我们将结果转换为`Tiger`时，我们试图使用名称`“lion”.` 获得一个`Tiger`，它将抛出一个`ClassCastException`:

```java
assertThrows(ClassCastException.class, () -> {
    Tiger tiger = (Tiger) context.getBean("lion");
});
```

### 3.2.按名称和类型检索 Bean

这里我们需要指定请求的 bean 的名称和类型:

```java
Lion lion = context.getBean("lion", Lion.class);
```

与前面的方法相比，这种方法更安全，因为我们可以立即获得关于类型不匹配的信息:

```java
assertThrows(BeanNotOfRequiredTypeException.class, () -> 
    context.getBean("lion", Tiger.class));
}
```

### 3.3.按类型检索 Bean

使用`getBean(),`的第三个变体，只需指定 bean 类型就足够了:

```java
Lion lion = context.getBean(Lion.class);
```

在这种情况下，我们需要**特别注意一个潜在的模糊结果**:

```java
assertThrows(NoUniqueBeanDefinitionException.class, () -> 
    context.getBean(Animal.class));
}
```

在上面的例子中，因为`Lion`和`Tiger`都实现了`Animal`接口，仅仅指定类型不足以明确地确定结果。因此，我们得到一个`NoUniqueBeanDefinitionException`。

### 3.4.使用构造函数参数按名称检索 Bean

除了 bean 名称，我们还可以传递构造函数参数:

```java
Tiger tiger = (Tiger) context.getBean("tiger", "Siberian");
```

**这个方法有点不同，因为它只适用于具有原型作用域**的 beans。

在单例中，我们将得到一个`[BeanDefinitionStoreException](/web/20221205124057/https://www.baeldung.com/spring-beandefinitionstoreexception).`

因为原型 bean 每次从应用程序容器请求时都会返回一个新创建的实例，**我们可以在调用`getBean()`时即时提供构造函数参数**:

```java
Tiger tiger = (Tiger) context.getBean("tiger", "Siberian");
Tiger secondTiger = (Tiger) context.getBean("tiger", "Striped");

assertEquals("Siberian", tiger.getName());
assertEquals("Striped", secondTiger.getName());
```

正如我们所看到的，根据我们在请求 bean 时指定的第二个参数，每个`Tiger`都有不同的名称。

### 3.5.使用构造函数参数按类型检索 Bean

这个方法类似于最后一个方法，但是我们需要传递类型而不是名称作为第一个参数:

```java
Tiger tiger = context.getBean(Tiger.class, "Shere Khan");

assertEquals("Shere Khan", tiger.getName());
```

类似于使用构造函数参数通过名称检索 bean，**这个方法只适用于具有原型作用域**的 bean。

## 4.使用注意事项

尽管是在`BeanFactory` 接口中定义的，但是`getBean()` 方法通常是通过`ApplicationContext.` 、**来访问的，我们不想在我们的程序**中直接使用`getBean()`方法。

Beans 应该由容器来管理。如果我们想使用其中一个，我们应该依靠[依赖注入](/web/20221205124057/https://www.baeldung.com/spring-dependency-injection)，而不是直接调用`ApplicationContext.getBean()` `.` ，这样，我们可以避免将应用程序逻辑与框架相关的细节混淆。

## 5.结论

在这个快速教程中，我们浏览了来自`BeanFactory` 接口的`getBean() `方法的所有实现，并描述了每个实现的优缺点。

这里显示的所有代码示例都可以在 GitHub 的[上找到。](https://web.archive.org/web/20221205124057/https://github.com/eugenp/tutorials/tree/master/spring-core-3)