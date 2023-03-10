# @在春天查找注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-lookup>

## 1。简介

在这个快速教程中，我们将通过`@Lookup`注释来看看 Spring 的方法级依赖注入支持。

## 2。为什么是`@Lookup`？

**用`@Lookup`标注的方法告诉 Spring 在我们调用它时返回该方法的返回类型的一个实例。**

本质上，Spring 将覆盖我们带注释的方法，并使用我们方法的返回类型和参数作为`BeanFactory#getBean.`的参数

`@Lookup`适用于:

*   将原型范围的 bean 注入到单独 bean 中(类似于`Provider`)
*   程序化地注入依赖关系

还要注意的是，`@Lookup` 是 XML 元素`lookup-method`的 Java 等价物。

## 3。使用`@Lookup`

### 3.1。将原型范围的 Bean 注入到单例 Bean 中

如果我们碰巧决定拥有一个原型 Spring bean，那么我们几乎马上就会面临这样的问题:我们的单例 Spring bean 如何访问这些原型 Spring bean？

现在，`Provider`当然是一种方式，尽管`@Lookup` 在某些方面更加通用。

首先，让我们创建一个原型 bean，稍后我们将把它注入到一个单独的 bean 中:

```java
@Component
@Scope("prototype")
public class SchoolNotification {
    // ... prototype-scoped state
}
```

如果我们创建一个使用`@Lookup`的单例 bean:

```java
@Component
public class StudentServices {

    // ... member variables, etc.

    @Lookup
    public SchoolNotification getNotification() {
        return null;
    }

    // ... getters and setters
}
```

使用`@Lookup`，我们可以通过我们的 singleton bean 获得一个`SchoolNotification`的实例:

```java
@Test
public void whenLookupMethodCalled_thenNewInstanceReturned() {
    // ... initialize context
    StudentServices first = this.context.getBean(StudentServices.class);
    StudentServices second = this.context.getBean(StudentServices.class);

    assertEquals(first, second); 
    assertNotEquals(first.getNotification(), second.getNotification()); 
}
```

注意，在`StudentServices`中，我们将`getNotification`方法作为一个存根。

这是因为 Spring 通过调用`beanFactory.getBean(StudentNotification.class)`覆盖了这个方法，所以我们可以让它为空。

### 3.2。程序化地注入依赖关系

然而，更强大的是，`@Lookup`允许我们程序化地注入依赖，这是我们不能用`Provider`做的。

让我们用一些状态来增强`StudentNotification` :

```java
@Component
@Scope("prototype")
public class SchoolNotification {
    @Autowired Grader grader;

    private String name;
    private Collection<Integer> marks;

    public SchoolNotification(String name) {
        // ... set fields
    }

    // ... getters and setters

    public String addMark(Integer mark) {
        this.marks.add(mark);
        return this.grader.grade(this.marks);
    }
}
```

现在，它依赖于一些 Spring 上下文，以及我们将在程序上提供的附加上下文。

然后，我们可以向`StudentServices` 添加一个方法，获取学生数据并持久化它:

```java
public abstract class StudentServices {

    private Map<String, SchoolNotification> notes = new HashMap<>();

    @Lookup
    protected abstract SchoolNotification getNotification(String name);

    public String appendMark(String name, Integer mark) {
        SchoolNotification notification
          = notes.computeIfAbsent(name, exists -> getNotification(name)));
        return notification.addMark(mark);
    }
} 
```

在运行时，Spring 将以同样的方式实现该方法，并增加了一些技巧。

首先，注意它可以调用一个复杂的构造函数，也可以注入其他 Spring beans，这允许我们将`SchoolNotification`处理得更像一个 Spring 感知方法。

它通过调用`beanFactory.getBean(SchoolNotification.class, name)`来实现`getSchoolNotification`来做到这一点。

第二，我们有时可以让`@Lookup-`带注释的方法变得抽象，就像上面的例子。

使用`abstract`比 stub 好看一点，但是我们只能在**不** **`component-scan`或者`@Bean`——管理**周围 bean 的时候使用它:

```java
@Test
public void whenAbstractGetterMethodInjects_thenNewInstanceReturned() {
    // ... initialize context

    StudentServices services = context.getBean(StudentServices.class);    
    assertEquals("PASS", services.appendMark("Alex", 89));
    assertEquals("FAIL", services.appendMark("Bethany", 78));
    assertEquals("PASS", services.appendMark("Claire", 96));
}
```

有了这个设置，我们可以向`SchoolNotification`添加 Spring 依赖项和方法依赖项。

## 4。局限性

尽管`@Lookup`功能多样，但还是有一些明显的局限性:

*   `@Lookup`-带注释的方法，如`getNotification,`，当周围的类，如`Student,`被组件扫描时，必须是具体的。这是因为组件扫描跳过了抽象 beans。
*   当周围的类由`@Bean`管理时，带注释的方法根本不起作用。

在这种情况下，如果我们需要将一个原型 bean 注入到一个 singleton 中，我们可以使用`Provider` 作为替代。

## 5。结论

在这篇简短的文章中，我们学习了如何以及何时使用 Spring 的`@Lookup`注释，包括如何使用它将原型范围的 bean 注入到单例 bean 中，以及如何使用它程序化地注入依赖关系。

本教程使用的所有代码都可以在 Github 上找到[。](https://web.archive.org/web/20220627170037/https://github.com/eugenp/tutorials/tree/master/spring-di-3)