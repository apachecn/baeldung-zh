# 用 Spring 注入 YAML 文件中的地图

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-yaml-inject-map>

## 1.概观

在这个快速教程中，我们将仔细看看**如何从 Spring Boot** 的 YAML 文件中注入地图。

首先，我们将开始对 Spring Framework 中的 YAML 文件有一点了解。然后，我们将通过一个实例展示如何将 YAML 属性绑定到一个`Map`。

## 2.Spring 框架中的 YAML 文件

使用 [YAML](https://web.archive.org/web/20220629003655/https://yaml.org/) 文件存储外部配置数据是 Spring 开发人员的常见做法。基本上， **[Spring 支持 YAML](/web/20220629003655/https://www.baeldung.com/spring-yaml-vs-properties) 文档作为属性的替代，并使用 [SnakeYAML](https://web.archive.org/web/20220629003655/https://bitbucket.org/asomov/snakeyaml/src) 来解析它们**。

事不宜迟，让我们看看典型的 YAML 文件是什么样子的:

```
server:
  port: 8090
  application:
    name: myapplication
    url: http://myapplication.com
```

正如我们所看到的， **YAML 文件是不言自明的，更容易阅读。事实上，YAML 提供了一种奇特而简洁的方式来存储分层配置数据。**

默认情况下，Spring Boot 在应用程序启动时从`application.properties`或`application.yml`读取配置属性。但是，我们可以使用 [`@PropertySource`来加载一个自定义的 YAML 文件](/web/20220629003655/https://www.baeldung.com/spring-yaml-propertysource)。

现在我们已经熟悉了什么是 YAML 文件，让我们看看如何在 Spring Boot 注入 YAML 属性作为`Map`。

## 3.如何从 YAML 文件中插入一个`Map`

Spring Boot 通过提供一个名为`@ConfigurationProperties.` 的便捷注释将数据外化提升到了一个新的高度。引入这个注释**是为了轻松地将配置文件中的外部属性直接注入 Java 对象**。

在这一节中，我们将深入讨论如何使用`@ConfigurationProperties` 注释`.`将 YAML 属性绑定到 bean 类中

首先，让我们在`application.yml`中定义一些键值属性:

```
server:
  application:
    name: InjectMapFromYAML
    url: http://injectmapfromyaml.dev
    description: How To Inject a map from a YAML File in Spring Boot
  config:
    ips:
      - 10.10.10.10
      - 10.10.10.11
      - 10.10.10.12
      - 10.10.10.13
    filesystem:
      - /dev/root
      - /dev/md2
      - /dev/md4
  users: 
    root:
      username: root
      password: rootpass
    guest:
      username: guest
      password: guestpass
```

在这个例子中，我们将尝试把`application`映射到一个简单的`Map<String, String>.` 中。同样，我们将把`config` 细节作为`Map<String, List<String>>,` 注入，把`users`作为`Map`注入，并把`String`键和属于用户定义类的对象作为值`.`

其次，让我们创建一个 bean 类—`ServerProperties –` 来封装将我们的配置属性绑定到`Map` s:

```
@Component
@ConfigurationProperties(prefix = "server")
public class ServerProperties {

    private Map<String, String> application;
    private Map<String, List<String>> config;
    private Map<String, Credential> users;

    // getters and setters

    public static class Credential {

        private String username;
        private String password;

        // getters and setters

    }
}
```

正如我们所看到的，我们用`@ConfigurationProperties.` **修饰了`ServerProperties`类，这样，我们告诉 Spring 将所有带有指定前缀的属性映射到一个对象** `**ServerProperties**.`

回想一下，我们的应用程序也需要为配置属性启用[，尽管](/web/20220629003655/https://www.baeldung.com/spring-enable-config-properties)[在大多数 Spring Boot 应用程序中这是自动完成的](/web/20220629003655/https://www.baeldung.com/spring-enable-config-properties#purpose)。

最后，让我们测试一下我们的 YAML 属性是否被正确地注入为`Map` s:

```
@RunWith(SpringRunner.class)
@SpringBootTest
class MapFromYamlIntegrationTest {

    @Autowired
    private ServerProperties serverProperties;

    @Test
    public void whenYamlFileProvidedThenInjectSimpleMap() {
        assertThat(serverProperties.getApplication())
          .containsOnlyKeys("name", "url", "description");

        assertThat(serverProperties.getApplication()
          .get("name")).isEqualTo("InjectMapFromYAML");
    }

    @Test
    public void whenYamlFileProvidedThenInjectComplexMap() {
        assertThat(serverProperties.getConfig()).hasSize(2);

        assertThat(serverProperties.getConfig()
          .get("ips")
          .get(0)).isEqualTo("10.10.10.10");

        assertThat(serverProperties.getUsers()
          .get("root")
          .getUsername()).isEqualTo("root");
    }

}
```

## 4.`@ConfigurationProperties`对`@Value`

现在让我们快速比较一下`@ConfigurationProperties` 和`@Value.`

尽管事实上这两种注释都可以用来从配置文件`,`中注入属性，但是它们是完全不同的。这两种注释之间的主要区别在于，每种注释都有不同的用途。

简而言之， **[`@V` `alue`](/web/20220629003655/https://www.baeldung.com/spring-value-annotation) 允许我们通过键直接注入一个特定的属性**值。**然而，`@ConfigurationProperties` 注释将多个属性**绑定到一个特定的对象，并通过映射的对象提供对属性的访问。

一般来说，在注入配置数据*时，Spring 推荐使用`@ConfigurationProperties` 而不是`@Value` 。`@ConfigurationProperties`* 提供了一种在结构化对象中集中和分组配置属性的好方法，我们可以稍后将它注入到其他 beans 中。

## 5.结论

总而言之，我们首先解释了如何从 Spring Boot 的 YAML 文件中注入一个`Map`。然后，我们强调了`@ConfigurationProperties` 和`@Value.`的区别

和往常一样，这篇文章的完整源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220629003655/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-properties-2)