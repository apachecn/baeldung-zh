# Spring 类 PathXmlApplicationContext 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-classpathxmlapplicationcontext>

## 1。概述

简单地说，Spring 框架核心是一个用于管理 beans 的 IoC 容器。

Spring 中有两种基本类型的容器 Bean 工厂和应用程序上下文。前者提供基本功能，这里[介绍](/web/20220815025002/https://www.baeldung.com/spring-beanfactory)；后者是前者的超集，使用最广泛。

`ApplicationContext`是`org.springframework.context`包中的一个接口，它有几个实现，`ClassPathXmlApplicationContext`就是其中之一。

在本文中，我们将重点关注`ClassPathXmlApplicationContext`提供的有用功能。

## 2。基本用法

### 2.1。初始化容器和管理 bean

`ClassPathXmlApplicationContext`可以从类路径加载 XML 配置并管理其 beans:

我们有一门`Student`课:

```java
public class Student {
    private int no;
    private String name;

    // standard constructors, getters and setters
}
```

我们在`classpathxmlapplicationcontext-example.xml`中配置一个`Student` bean，并将其添加到一个类路径中:

```java
<beans ...>
    <bean id="student" class="com.baeldung.applicationcontext.Student">
        <property name="no" value="15"/>
        <property name="name" value="Tom"/>
    </bean>
</beans>
```

现在我们可以使用`ClassPathXmlApplicationContext`来加载 XML 配置并获得`Student` bean:

```java
@Test
public void testBasicUsage() {
    ApplicationContext context 
      = new ClassPathXmlApplicationContext(
        "classpathxmlapplicationcontext-example.xml");

    Student student = (Student) context.getBean("student");
    assertThat(student.getNo(), equalTo(15));
    assertThat(student.getName(), equalTo("Tom"));

    Student sameStudent = context.getBean("student", Student.class);
    assertThat(sameStudent.getNo(), equalTo(15));
    assertThat(sameStudent.getName(), equalTo("Tom"));
}
```

### 2.2。多个 XML 配置

有时我们想使用几个 XML 配置来初始化一个 Spring 容器。在这种情况下，我们只需要在构造`ApplicationContext`时添加几个配置位置:

```java
ApplicationContext context 
  = new ClassPathXmlApplicationContext("ctx.xml", "ctx2.xml");
```

## 3。附加功能

### 3.1。优雅地关闭 Spring IoC 容器

当我们在 web 应用程序中使用 Spring IoC 容器时，Spring 基于 web 的`ApplicationContext`实现将在应用程序关闭时优雅地关闭容器，但如果我们在非 web 环境中使用它，比如独立的桌面应用程序，我们必须自己向 JVM 注册一个关闭挂钩，以确保 Spring IoC 容器优雅地关闭，并调用 destroy 方法来释放资源。

让我们给`Student`类添加一个`destroy()`方法:

```java
public void destroy() {
    System.out.println("Student(no: " + no + ") is destroyed");
}
```

我们现在可以将这个方法配置为`student` bean 的销毁方法:

```java
<beans ...>
    <bean id="student" class="com.baeldung.applicationcontext.Student" 
      destroy-method="destroy">
        <property name="no" value="15"/>
        <property name="name" value="Tom"/>
    </bean>
</beans>
```

我们现在将注册一个关闭挂钩:

```java
@Test
public void testRegisterShutdownHook() {
    ConfigurableApplicationContext context 
      = new ClassPathXmlApplicationContext(
        "classpathxmlapplicationcontext-example.xml");
    context.registerShutdownHook();
}
```

当我们运行测试方法时，我们可以看到`destroy()`方法被调用。

### 3.2。国际化用`MessageSource`

`ApplicationContext`接口扩展了`MessageSource`接口，因此提供了国际化功能。

一个`ApplicationContext`容器在初始化时自动搜索一个`MessageSource` bean，这个 bean 必须被命名为`messageSource`。

下面是一个使用不同语言的例子:

首先，我们在类路径中添加一个`dialog`目录，在对话框目录中添加两个文件:`dialog_en.properties`和`dialog_zh_CN.properties`。

`dialog_en.properties`:

```java
hello=hello
you=you
thanks=thank {0}
```

`dialog_zh_CN.properties`:

```java
hello=\u4f60\u597d
you=\u4f60
thanks=\u8c22\u8c22{0}
```

在`classpathxmlapplicationcontext-internationalization.xml`中配置`messageSource` bean:

```java
<beans ...>
    <bean id="messageSource"
      class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>dialog/dialog</value>
            </list>
        </property>
    </bean>
</beans>
```

然后，让我们用`MessageSource`得到不同语言的对话词:

```java
@Test
public void testInternationalization() {
    MessageSource resources 
      = new ClassPathXmlApplicationContext(
        "classpathxmlapplicationcontext-internationalization.xml");

    String enHello = resources.getMessage(
      "hello", null, "Default", Locale.ENGLISH);
    String enYou = resources.getMessage(
      "you", null, Locale.ENGLISH);
    String enThanks = resources.getMessage(
      "thanks", new Object[] { enYou }, Locale.ENGLISH);

    assertThat(enHello, equalTo("hello"));
    assertThat(enThanks, equalTo("thank you"));

    String chHello = resources.getMessage(
      "hello", null, "Default", Locale.SIMPLIFIED_CHINESE);
    String chYou = resources.getMessage(
      "you", null, Locale.SIMPLIFIED_CHINESE);
    String chThanks = resources.getMessage(
      "thanks", new Object[] { chYou }, Locale.SIMPLIFIED_CHINESE);

    assertThat(chHello, equalTo("你好"));
    assertThat(chThanks, equalTo("谢谢你"));
}
```

## 4。`ApplicationContext`引用

有时我们需要获取它所管理的 beans 内部对`ApplicationContext`的引用，我们可以使用`ApplicationContextAware`或`@Autowired`来实现。让我们看看如何使用`ApplicationContextAware`:

我们有一个名为`Course`的类:

```java
public class Course {

    private String name;

    // standard constructors, getters and setters
}
```

我们有一个`Teacher`类，它根据容器的 beans 组装它的课程:

```java
public class Teacher implements ApplicationContextAware {

    private ApplicationContext context;
    private List<Course> courses = new ArrayList<>();

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.context = applicationContext;
    }

    @PostConstruct
    public void addCourse() {
        if (context.containsBean("math")) {
            Course math = context.getBean("math", Course.class);
            courses.add(math);
        }
        if (context.containsBean("physics")) {
            Course physics = context.getBean("physics", Course.class);
            courses.add(physics);
        }
    }

    // standard constructors, getters and setters
}
```

让我们在`classpathxmlapplicationcontext-example.xml`中配置`course` bean 和`teacher` bean:

```java
<beans ...>
    <bean id="math" class="com.baeldung.applicationcontext.Course">
        <property name="name" value="math"/>
    </bean>

    <bean name="teacher" class="com.baeldung.applicationcontext.Teacher"/>
</beans>
```

然后–测试`courses`属性的注入:

```java
@Test
public void testApplicationContextAware() {
    ApplicationContext context 
       = new ClassPathXmlApplicationContext(
         "classpathxmlapplicationcontext-example.xml");
    Teacher teacher = context.getBean("teacher", Teacher.class);
    List<Course> courses = teacher.getCourses();

    assertThat(courses.size(), equalTo(1));
    assertThat(courses.get(0).getName(), equalTo("math"));
}
```

除了实现`ApplicationContextAware`接口，使用`@Autowired`注释也有同样的效果。

让我们把`Teacher`类改成这样:

```java
public class Teacher {

    @Autowired
    private ApplicationContext context;
    private List<Course> courses = new ArrayList<>();

    @PostConstruct
    public void addCourse() {
        if (context.containsBean("math")) {
            Course math = context.getBean("math", Course.class);
            courses.add(math);
        }
        if (context.containsBean("physics")) {
            Course physics = context.getBean("physics", Course.class);
            courses.add(physics);
        }
    }

    // standard constructors, getters and setters
}
```

然后运行测试，我们可以看到结果是相同。

## 5。结论

`ApplicationContext`是一个 Spring 容器，与`BeanFactory`相比，它具有更多特定于企业的功能，而`ClassPathXmlApplicationContext`是它最常用的实现之一。

所以在本文中，我们介绍了`ClassPathXmlApplicationContext`的几个方面，包括它的基本用法、它的关机注册功能、它的国际化功能以及它的引用的获取。

与往常一样，该示例的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220815025002/https://github.com/eugenp/tutorials/tree/master/spring-core)