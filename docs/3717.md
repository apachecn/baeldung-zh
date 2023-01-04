# Groovy Bean 定义

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-groovy-beans>

## 1。概述

在这篇简短的文章中，我们将关注如何在 Java Spring 项目中使用基于 Groovy 的配置。

## 2。依赖性

在开始之前，我们需要将依赖项添加到我们的`pom.xml`文件中。为了编译 Groovy 文件，我们还需要添加一个插件。

让我们首先将 Groovy 的依赖项添加到我们的`pom.xml`文件中:

```
<dependency>
    <groupId>org.codehaus.groovy</groupId>
    <artifactId>groovy</artifactId>
    <version>2.5.10</version>
</dependency>
```

现在，让我们添加插件:

```
<build>
    <plugins>
        //...
        <plugin>
            <groupId>org.codehaus.gmavenplus</groupId>
            <artifactId>gmavenplus-plugin</artifactId>
            <version>1.9.0</version>
            <executions>
                <execution>
                    <goals>
                        <goal>addSources</goal>
                        <goal>addTestSources</goal>
                        <goal>generateStubs</goal>
                        <goal>compile</goal>
                        <goal>generateTestStubs</goal>
                        <goal>compileTests</goal>
                        <goal>removeStubs</goal>
                        <goal>removeTestStubs</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

在这里，我们使用 [gmavenplus-plugin](https://web.archive.org/web/20221128045814/https://search.maven.org/search?q=gmavenplus-plugin) 和所有的[目标](https://web.archive.org/web/20221128045814/https://github.com/groovy/GMavenPlus/wiki/Usage#why-do-i-need-so-many-goals)。

这些库的最新版本可以在 [Maven Central](https://web.archive.org/web/20221128045814/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.codehaus.groovy%22%20AND%20a%3A%22groovy-all%22) 上找到。

## 3。定义 bean

从版本 4 开始，Spring 提供了对基于 Groovy 的配置的支持。这意味着 **Groovy 类可以是合法的 Spring beans。**

为了说明这一点，我们将使用标准 Java 配置定义一个 bean，然后使用 Groovy 配置同一个 bean。这样，我们就能看出区别。

让我们创建一个具有几个属性的简单类:

```
public class JavaPersonBean {
    private String firstName;
    private String lastName;

    // standard getters and setters
} 
```

记住 getter/setter 很重要——它们对机制的工作至关重要。

### 3.1。Java 配置

我们可以使用基于 Java 的配置来配置同一个 bean:

```
@Configuration
public class JavaBeanConfig {

    @Bean
    public JavaPersonBean javaPerson() {
        JavaPersonBean jPerson = new JavaPersonBean();
        jPerson.setFirstName("John");
        jPerson.setLastName("Doe");

        return jPerson;
    }
}
```

### 3.2。Groovy 配置

现在，当我们使用 Groovy 配置之前创建的 bean 时，我们可以看到不同之处:

```
beans {
    javaPersonBean(JavaPersonBean) {
        firstName = 'John'
        lastName = 'Doe'
    }
}
```

注意，在定义 bean 配置之前，我们还应该导入`JavaPersonBean class.` ，`beans block`中的**，我们可以根据需要定义任意多的 bean。**

我们将字段定义为私有的，尽管 Groovy 看起来像是在直接访问它们，但它是使用提供的 getter/setter 来实现的。

## 4。附加 Bean 设置

与基于 XML 和 Java 的配置一样，我们不仅可以配置 beans。

如果我们需要为 bean 设置一个`alias`,我们可以很容易地做到:

```
registerAlias("bandsBean","bands")
```

如果我们想定义 bean 的`scope:`

```
{ 
    bean ->
        bean.scope = "prototype"
}
```

要为我们的 bean 添加生命周期回调，我们可以:

```
{ 
    bean ->
        bean.initMethod = "someInitMethod"
        bean.destroyMethod = "someDestroyMethod"
}
```

我们还可以在 bean 定义中指定继承:

```
{ 
    bean->
        bean.parent="someBean"
}
```

最后，如果我们需要从 XML 配置中导入一些先前定义的 beans，我们可以使用`importBeans():`来完成

```
importBeans("somexmlconfig.xml")
```

## 5。结论

在本教程中，我们看到了如何创建 Spring Groovy bean 配置。我们还讨论了在 bean 上设置附加属性，比如别名、作用域、父类、初始化或销毁方法，以及如何导入其他 XML 定义的 bean。

虽然这些例子很简单，但是它们可以被扩展并用于创建任何类型的 Spring 配置。

本文中使用的完整示例代码可以在我们的 [GitHub 项目](https://web.archive.org/web/20221128045814/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-groovy)中找到。这是一个 Maven 项目，所以您应该能够导入它并按原样运行它。