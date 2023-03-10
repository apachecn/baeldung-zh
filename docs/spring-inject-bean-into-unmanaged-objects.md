# 将 Spring Beans 注入非托管对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-inject-bean-into-unmanaged-objects>

## 1.驱动力

在 Spring 应用程序中，将一个 bean 注入另一个 bean 是非常常见的。然而，有时将 bean 注入一个普通的对象是可取的。例如，我们可能想从一个实体对象中获取对服务的引用。

幸运的是，实现这一目标并不像看起来那么难。接下来的部分将介绍如何使用 [`@Configurable`](https://web.archive.org/web/20221216225702/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/annotation/Configurable.html) 注释和一个 [AspectJ](/web/20221216225702/https://www.baeldung.com/aspectj) weaver 来完成这个任务。

## 2.`@Configurable`注解

该注释允许修饰类的实例保存对 Spring beans 的引用。

### 2.1.定义和注册 Spring Bean

在介绍`@Configurable`注释之前，让我们设置一个 Spring bean 定义:

```java
@Service
public class IdService {
    private static int count;

    int generateId() {
        return ++count;
    }
}
```

这个类是用`@Service`注释修饰的；因此，它可以通过组件扫描向 Spring 上下文注册。

下面是启用该机制的简单配置类:

```java
@ComponentScan
public class AspectJConfig {
}
```

### 2.2.使用`@Configurable`

最简单的形式是，**我们可以使用不带任何元素的`@Configurable`:**

```java
@Configurable
public class PersonObject {
    private int id;
    private String name;

    public PersonObject(String name) {
        this.name = name;
    }

    // getters and other code shown in the next subsection
}
```

在本例中，`@Configurable`注释将`PersonObject`类标记为适合弹簧驱动配置。

### 2.3.将 Spring Bean 注入到非托管对象中

我们可以将`IdService`注入`PersonObject`，就像我们在任何 Spring bean 中做的那样:

```java
@Configurable
public class PersonObject {
    @Autowired
    private IdService idService;

    // fields, constructor and getters - shown in the previous subsection

    void generateId() {
        this.id = idService.generateId();
    }
}
```

然而，注释只有在被处理程序识别和处理时才有用。这就是 AspectJ weaver 发挥作用的地方。具体来说，**`AnnotationBeanConfigurerAspect`将根据`@Configurable`的存在而行动，并进行必要的处理。**

## 3.启用 AspectJ 编织

### 3.1.插件声明

为了启用 AspectJ 编织，我们首先需要 AspectJ Maven 插件:

```java
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>aspectj-maven-plugin</artifactId>
    <version>1.11</version>
    <!-- configuration and executions -->
</plugin>
```

它需要一些额外的配置:

```java
<configuration>
    <complianceLevel>1.8</complianceLevel>
    <Xlint>ignore</Xlint>
    <aspectLibraries>
        <aspectLibrary>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
        </aspectLibrary>
    </aspectLibraries>
</configuration>
```

第一个必需的元素是`complianceLevel`。值`1.8`将源和目标 JDK 版本都设置为 1.8。如果没有明确设置，源版本将是 1.3，目标版本将是 1.1。这些值显然已经过时，对于现代的 Java 应用程序来说是不够的。

要将 bean 注入到非托管对象中，我们必须依赖于在`spring-aspects.jar`中提供的`AnnotationBeanConfigurerAspect`类。由于这是一个预编译的方面，我们需要**将包含的工件添加到插件配置中。**

请注意，这样一个被引用的工件必须作为一个依赖项存在于项目中:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>5.2.7.RELEASE</version>
</dependency>
```

我们可以在 [Maven Central](https://web.archive.org/web/20221216225702/https://search.maven.org/search?q=g:org.springframework%20a:spring-aspects) 上找到`spring-aspects`的最新版本。

### 3.2.插件执行

为了指示插件编织所有相关的类，我们需要这个`executions`配置:

```java
<executions>
    <execution>
        <goals>
            <goal>compile</goal>
        </goals>
    </execution>
</executions>
```

注意**默认情况下，插件的`compile`目标绑定到编译生命周期阶段。**

### 3.2.Bean 配置

启用 AspectJ 编织的最后一步是**将`@EnableSpringConfigured`添加到配置类:**

```java
@ComponentScan
@EnableSpringConfigured
public class AspectJConfig {
}
```

额外的注释配置了`AnnotationBeanConfigurerAspect`，它又向 Spring IoC 容器注册了`PersonObject`实例。

## 4.测试

现在，让我们验证一下`IdService` bean 已经成功地注入到了`PersonObject`中:

```java
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = AspectJConfig.class)
public class PersonUnitTest {
    @Test
    public void givenUnmanagedObjects_whenInjectingIdService_thenIdValueIsCorrectlySet() {
        PersonObject personObject = new PersonObject("Baeldung");
        personObject.generateId();
        assertEquals(1, personObject.getId());
        assertEquals("Baeldung", personObject.getName());
    }
}
```

## 5.将 Bean 注入 JPA 实体

从 Spring 容器的角度来看，实体只不过是一个普通的对象。因此，将 Spring bean 注入 JPA 实体没有什么特别的。

然而，由于注入 JPA 实体是一个典型的用例，让我们更详细地讨论它。

### 5.1.实体类

让我们从实体类的框架开始:

```java
@Entity
@Configurable(preConstruction = true)
public class PersonEntity {
    @Id
    private int id;
    private String name;

    public PersonEntity() {
    }

    // other code - shown in the next subsection
}
```

注意`@Configurable`注释中的`preConstruction`元素:**它使我们能够在对象完全构建之前将依赖注入到对象中。**

### 5.2.服务注入

现在我们可以将`IdService`注入`PersonEntity`，类似于我们对`PersonObject`所做的:

```java
// annotations
public class PersonEntity {
    @Autowired
    @Transient
    private IdService idService;

    // fields and no-arg constructor

    public PersonEntity(String name) {
        id = idService.generateId();
        this.name = name;
    }

    // getters
}
```

`@Transient`注释用于告诉 JPA`idService`是一个不被持久化的字段。

### 5.3.测试方法更新

最后，我们可以更新测试方法，以表明服务可以注入到实体中:

```java
@Test
public void givenUnmanagedObjects_whenInjectingIdService_thenIdValueIsCorrectlySet() {
    // existing statements

    PersonEntity personEntity = new PersonEntity("Baeldung");
    assertEquals(2, personEntity.getId());
    assertEquals("Baeldung", personEntity.getName());
}
```

## 6.警告

尽管从非托管对象访问 Spring 组件很方便，但这样做通常不是一个好的做法。

问题是非托管对象，包括实体，通常是域模型的一部分。这些对象应该只携带可以跨不同服务重用的数据。

将 beans 注入这样的对象会将组件和对象捆绑在一起，使得维护和增强应用程序变得更加困难。

## 7.结论

本教程介绍了将 Spring bean 注入非托管对象的过程。它还提到了一个与依赖注入对象相关的设计问题。

实现代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221216225702/https://github.com/eugenp/tutorials/tree/master/spring-di-2)