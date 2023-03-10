# YAML 在 Spring Boot 的对象列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-yaml-list>

## 1.概观

在这个简短的教程中，我们将仔细看看如何将 YAML 列表映射到 Spring Boot 的。

我们将从如何在 YAML 定义列表的一些背景开始。

然后我们将深入探讨如何将 YAML 列表绑定到对象上。

## 2.快速回顾 YAML 的列表

简而言之， [YAML](https://web.archive.org/web/20221126233544/https://yaml.org/spec/1.2/spec.html) 是一种人类可读的数据序列化标准，它提供了一种简洁明了的方式来编写配置文件。**YAML 的好处在于它支持多种数据类型，比如`List`、`Map`和标量类型。**

YAML 列表中的元素是使用“-”字符定义的，它们都共享相同的缩进级别:

```java
yamlconfig:
  list:
    - item1
    - item2
    - item3
    - item4
```

作为比较，基于属性的等效使用指数:

```java
yamlconfig.list[0]=item1
yamlconfig.list[1]=item2
yamlconfig.list[2]=item3
yamlconfig.list[3]=item4
```

更多的例子，请随意阅读我们关于如何使用 YAML 和属性文件定义[列表和地图的文章。](/web/20221126233544/https://www.baeldung.com/spring-yaml-vs-properties#lists-and-maps)

事实上，与属性文件相比， **YAML 的分层特性显著增强了可读性。**YAML 的另一个有趣的特性是可以为不同的弹簧轮廓定义[不同的属性](/web/20221126233544/https://www.baeldung.com/spring-yaml#spring-yaml-file)。从引导版本 2.4.0 开始，这对于属性文件也是可能的。

值得一提的是，Spring Boot 为 YAML 配置提供了开箱即用的支持。按照设计，Spring Boot 在启动时从`application.yml`加载配置属性，不需要任何额外的工作。

## 3.将 YAML 列表绑定到一个简单的对象`List`

Spring Boot 为**提供了`[@ConfigurationProperties](/web/20221126233544/https://www.baeldung.com/configuration-properties-in-spring-boot)`注释，简化了将外部配置数据映射到对象模型的逻辑。**

在这一节中，我们将使用`@ConfigurationProperties`将 YAML 列表绑定到`List<Object>`中。

我们首先在`application.yml`中定义一个简单的列表:

```java
application:
  profiles:
    - dev
    - test
    - prod
    - 1
    - 2
```

然后我们将创建一个简单的`ApplicationProps` [POJO](/web/20221126233544/https://www.baeldung.com/java-pojo-class) 来保存将我们的 YAML 列表绑定到对象的`List`的逻辑:

```java
@Component
@ConfigurationProperties(prefix = "application")
public class ApplicationProps {

    private List<Object> profiles;

    // getter and setter

}
```

`ApplicationProps` 类需要用`@ConfigurationProperties` **修饰，以表达将所有带有指定前缀的 YAML 属性映射到一个`ApplicationProps`对象的意图。**

为了绑定`profiles`列表，我们只需要定义一个类型为`List`的字段，剩下的工作由`@ConfigurationProperties` 注释完成。

注意，我们使用`@Component`将`ApplicationProps` 类注册为普通的 Spring bean。**因此，我们可以像其他任何 Spring bean 一样将它注入到其他类中。**

最后，我们将`ApplicationProps` bean 注入到一个测试类中，并验证我们的`profiles` YAML 列表是否被正确地注入为`List<Object>`:

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(initializers = ConfigDataApplicationContextInitializer.class)
@EnableConfigurationProperties(value = ApplicationProps.class)
class YamlSimpleListUnitTest {

    @Autowired
    private ApplicationProps applicationProps;

    @Test
    public void whenYamlList_thenLoadSimpleList() {
        assertThat(applicationProps.getProfiles().get(0)).isEqualTo("dev");
        assertThat(applicationProps.getProfiles().get(4).getClass()).isEqualTo(Integer.class);
        assertThat(applicationProps.getProfiles().size()).isEqualTo(5);
    }
}
```

## 4.将 YAML 列表绑定到复杂列表

现在让我们更深入，看看如何将嵌套的 YAML 列表注入到复杂的结构化`List`中

首先，让我们给`application.yml`添加一些嵌套列表:

```java
application:
  // ...
  props: 
    -
      name: YamlList
      url: http://yamllist.dev
      description: Mapping list in Yaml to list of objects in Spring Boot
    -
      ip: 10.10.10.10
      port: 8091
    -
      email: [[email protected]](/web/20221126233544/https://www.baeldung.com/cdn-cgi/l/email-protection)
      contact: http://yamllist.dev/contact
  users:
    -
      username: admin
      password: [[email protected]](/web/20221126233544/https://www.baeldung.com/cdn-cgi/l/email-protection)@
      roles:
        - READ
        - WRITE
        - VIEW
        - DELETE
    -
      username: guest
      password: [[email protected]](/web/20221126233544/https://www.baeldung.com/cdn-cgi/l/email-protection)
      roles:
        - VIEW
```

在这个例子中，我们将把`props` 属性绑定到一个`List<Map<String, Object>>`。同样，我们将把`users` 映射成`User`对象的`List`。

由于`props` 条目的每个元素持有不同的键，我们可以将其作为 `Map`的`List` 注入。请务必查看我们的文章，了解如何[从 Spring Boot 的 YAML 文件](/web/20221126233544/https://www.baeldung.com/spring-yaml-inject-map) `.`注入地图

然而`,` 在`users`的情况下，所有的条目共享相同的关键字**，所以为了简化它的映射，我们可能需要创建一个专用的`User`类来将关键字封装成字段**:

```java
public class ApplicationProps {

    // ...

    private List<Map<String, Object>> props;
    private List<User> users;

    // getters and setters

    public static class User {

        private String username;
        private String password;
        private List<String> roles;

        // getters and setters

    }
}
```

现在，我们验证嵌套的 YAML 列表是否被正确映射:

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(initializers = ConfigDataApplicationContextInitializer.class)
@EnableConfigurationProperties(value = ApplicationProps.class)
class YamlComplexListsUnitTest {

    @Autowired
    private ApplicationProps applicationProps;

    @Test
    public void whenYamlNestedLists_thenLoadComplexLists() {
        assertThat(applicationProps.getUsers().get(0).getPassword()).isEqualTo("[[email protected]](/web/20221126233544/https://www.baeldung.com/cdn-cgi/l/email-protection)@");
        assertThat(applicationProps.getProps().get(0).get("name")).isEqualTo("YamlList");
        assertThat(applicationProps.getProps().get(1).get("port").getClass()).isEqualTo(Integer.class);
    }

}
```

## 5.结论

在本文中，我们学习了如何将 YAML 列表映射到 Java 中

我们还检查了如何将复杂列表绑定到定制 POJOs。

和往常一样，本文的完整源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221126233544/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-properties-2)