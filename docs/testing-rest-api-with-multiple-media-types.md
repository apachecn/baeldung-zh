# 使用多种 MIME 类型测试 REST

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/testing-rest-api-with-multiple-media-types>

## 1。概述

**本文将重点介绍使用多种媒体类型/表示来测试 REST 服务。**

我们将编写能够在 API 支持的多种类型的表示之间切换的集成测试。目标是能够运行完全相同的测试，使用完全相同的服务 URIs，只是要求不同的媒体类型。

## 2。目标

任何 REST API 都需要将其资源公开为使用一种或多种媒体类型的表示。**客户端将设置`Accept`头来选择它向服务请求的表示类型。**

由于资源可以有多种表示，服务器必须实现一种负责选择正确表示的机制。这也称为内容协商。

因此，如果客户机请求`application/xml`，那么它应该获得资源的 XML 表示。如果它要求`application/json`，那么它应该得到 JSON。

## 3。测试基础设施

我们将从定义一个简单的编组器接口开始。这将是允许测试在不同媒体类型之间切换的主要抽象:

```
public interface IMarshaller {
    ...
    String getMime();
}
```

然后，我们需要一种基于某种形式的外部配置来初始化正确的编组器的方法。

**为此，我们将使用一个 Spring `FactoryBean`来初始化编组器，并使用一个简单的属性来确定使用哪个编组器**:

```
@Component
@Profile("test")
public class TestMarshallerFactory implements FactoryBean<IMarshaller> {

    @Autowired
    private Environment env;

    public IMarshaller getObject() {
        String testMime = env.getProperty("test.mime");
        if (testMime != null) {
            switch (testMime) {
            case "json":
                return new JacksonMarshaller();
            case "xml":
                return new XStreamMarshaller();
            default:
                throw new IllegalStateException();
            }
        }

        return new JacksonMarshaller();
    }

    public Class<IMarshaller> getObjectType() {
        return IMarshaller.class;
    }

    public boolean isSingleton() {
        return true;
    }
}
```

让我们来看看这个:

*   首先，这里使用了 Spring 3.1 中引入的新的`Environment`抽象——关于这一点的更多信息，请查看关于使用 Spring 属性的[详细文章](/web/20220120013454/https://www.baeldung.com/properties-with-spring "Properties Files usage in Spring 3.1")
*   我们从环境中检索`test.mime`属性，并使用它来确定创建哪个编组器——这里使用了一些 Java 7 `switch on String`语法
*   接下来，如果该属性根本没有定义，默认的封送拆收器将成为 JSON 支持的 Jackson 封送拆收器
*   最后——这个`BeanFactory`只在测试场景中是活动的，因为我们使用的是`@Profile`支持，也是在 Spring 3.1 中引入的

就是这样——该机制能够根据`test.mime` 属性的值在编组器之间切换。

## 4。JSON 和 XML 编组器

接下来，我们需要实际的编组器实现——每个支持的媒体类型一个。

对于 JSON，我们将使用`Jackson`作为底层库:

```
public class JacksonMarshaller implements IMarshaller {
    private ObjectMapper objectMapper;

    public JacksonMarshaller() {
        super();
        objectMapper = new ObjectMapper();
    }

    ...

    @Override
    public String getMime() {
        return MediaType.APPLICATION_JSON.toString();
    }
}
```

对于 XML 支持，编组器使用`XStream`:

```
public class XStreamMarshaller implements IMarshaller {
    private XStream xstream;

    public XStreamMarshaller() {
        super();
        xstream = new XStream();
    }

    ...

    public String getMime() {
        return MediaType.APPLICATION_XML.toString();
    }
}
```

**注意，这些编组员本身不是春豆**。原因是它们将通过`TestMarshallerFactory;`引导到 Spring 上下文中，没有必要直接把它们做成组件。

## 5。使用 JSON 和 XML 消费服务

此时，我们应该能够针对已部署的服务运行完整的集成测试。使用编组器很简单:我们将在测试中注入一个 `IMarshaller`:

```
@ActiveProfiles({ "test" })
public abstract class SomeRestLiveTest {

    @Autowired
    private IMarshaller marshaller;

    // tests
    ...

}
```

**Spring 将根据`test.mime`属性的值决定注入哪个封送拆收器。**

如果我们不为这个属性提供一个值，`TestMarshallerFactory`将简单地返回到默认的编组器 JSON 编组器。

## 6。梅文和詹金斯

如果 Maven 设置为针对已经部署的 REST 服务运行集成测试，那么我们可以使用:

```
mvn test -Dtest.mime=xml
```

或者，如果构建使用 Maven 生命周期的`integration-test`阶段:

```
mvn integration-test -Dtest.mime=xml
```

关于如何设置 Maven 构建来运行集成测试的更多细节，请参见 [`Integration Testing with Maven`](/web/20220120013454/https://www.baeldung.com/integration-testing-with-the-maven-cargo-plugin "How to set up Integration Testing with the Maven Cargo plugin") 一文。

对于 Jenkins，我们必须配置以下工作:

```
This build is parametrized
```

并增加了`String parameter` : `test.mime=xml`。【T2

一个常见的 Jenkins 配置将不得不对部署的服务运行相同的集成测试集——一个使用 XML，另一个使用 JSON 表示。

## 7。结论

本文展示了如何测试使用多种表示的 REST API。大多数 API 确实在多种表示下发布它们的资源，所以测试所有这些表示是至关重要的。事实上，我们可以在所有的测试中使用完全相同的测试，这很酷。

这种机制的完整实现——使用实际的集成测试并验证 XML 和 JSON 表示——可以在 GitHub 项目中找到。