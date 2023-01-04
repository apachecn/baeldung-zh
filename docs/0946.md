# 在 Spring 中将资源作为字符串加载

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-load-resource-as-string>

## 1.概观

在本教程中，我们将研究各种方法来将包含文本的资源内容作为字符串注入到我们的 Spring beans 中。

我们将看看如何定位资源并读取其内容。

此外，我们将演示如何在几个 beans 之间共享加载的资源。我们将通过使用与依赖注入相关的[注释来展示这一点，尽管同样的事情也可以通过使用](/web/20220630130841/https://www.baeldung.com/spring-annotations-resource-inject-autowire)[基于 XML 的注入](/web/20220630130841/https://www.baeldung.com/spring-xml-injection)并在 XML 属性文件中声明 beans 来实现。

## 2.使用`Resource`

我们可以通过使用 [`Resource`](/web/20220630130841/https://www.baeldung.com/spring-classpath-file-access) 界面来简化资源文件的定位。Spring 帮助我们使用资源加载器找到并读取资源，资源加载器根据提供的路径决定选择哪个`Resource`实现。`Resource`实际上是访问资源内容的一种方式，而不是内容本身。

让我们来看看[获取类路径](/web/20220630130841/https://www.baeldung.com/spring-classpath-file-access)上资源的`Resource`实例的一些方法。

### 2.1.使用`ResourceLoader`

如果我们喜欢使用延迟加载，我们可以使用类`ResourceLoader`:

```
ResourceLoader resourceLoader = new DefaultResourceLoader();
Resource resource = resourceLoader.getResource("classpath:resource.txt");
```

我们也可以用`@Autowired`将`ResourceLoader`注入到 bean 中:

```
@Autowired
private ResourceLoader resourceLoader;
```

### 2.2.使用`@Resource`

我们可以用`@Value`将`Resource`直接注入到 Spring bean 中:

```
@Value("classpath:resource.txt")
private Resource resource;
```

## 3.从`Resource`转换到`String`

一旦我们访问了`Resource`，我们就需要能够将它读入`String`。让我们用一个静态方法`asString`创建一个`ResourceReader`实用程序类来为我们做这件事。

首先，我们必须获得一个`InputStream`:

```
InputStream inputStream = resource.getInputStream();
```

我们的下一步是把这个`InputStream`转换成一个`String`。我们可以用春天自己的`FileCopyUtils#copyToString`方法:

```
public class ResourceReader {

    public static String asString(Resource resource) {
        try (Reader reader = new InputStreamReader(resource.getInputStream(), UTF_8)) {
            return FileCopyUtils.copyToString(reader);
        } catch (IOException e) {
            throw new UncheckedIOException(e);
        }
    }

    // more utility methods
}
```

有许多其他的方法可以实现这个，例如，使用 Spring 的`StreamUtils`类的`copyToString`

让我们也创建另一个实用方法`readFileToString,`，它将检索路径的`Resource`，并调用`asString`方法将其转换为`String`。

```
public static String readFileToString(String path) {
    ResourceLoader resourceLoader = new DefaultResourceLoader();
    Resource resource = resourceLoader.getResource(path);
    return asString(resource);
}
```

## 4.添加一个`Configuration`类

如果每个 bean 都必须单独注入资源`String`，那么拥有自己的`String`副本的 bean 就有可能出现代码重复和更多的内存使用。

通过在加载应用程序上下文时将资源的内容注入一个或多个 Spring beans，我们可以实现一个更简洁的解决方案。通过这种方式，我们可以对需要使用该内容的各种 beans 隐藏读取资源的实现细节。

```
@Configuration
public class LoadResourceConfig {

    // Bean Declarations
}
```

### 4.1.使用包含资源字符串的 Bean

让我们声明 beans 来保存一个`@Configuration`类中的资源内容:

```
@Bean
public String resourceString() {
    return ResourceReader.readFileToString("resource.txt");
}
```

现在让我们通过添加一个 [`@Autowired`](/web/20220630130841/https://www.baeldung.com/spring-autowire) 注释将注册的 beans 注入到字段中:

```
public class LoadResourceAsStringIntegrationTest {
    private static final String EXPECTED_RESOURCE_VALUE = "...";  // The string value of the file content

    @Autowired
    @Qualifier("resourceString")
    private String resourceString;

    @Test
    public void givenUsingResourceStringBean_whenConvertingAResourceToAString_thenCorrect() {
        assertEquals(EXPECTED_RESOURCE_VALUE, resourceString);
    }
}
```

在这种情况下，我们使用`@Qualifier`注释和 bean 的名称，因为**我们可能需要注入相同类型的多个字段**–`String`。

我们应该注意，限定符中使用的 bean 名称是从在配置类中创建 bean 的方法的名称中派生出来的。

## 5.使用 SpEL

最后，让我们看看如何使用 Spring Expression 语言来描述将资源文件直接加载到类的字段中所需的代码。

让我们使用`@Value`注释将文件内容注入到字段`resourceStringUsingSpel`中:

```
public class LoadResourceAsStringIntegrationTest {
    private static final String EXPECTED_RESOURCE_VALUE = "..."; // The string value of the file content

    @Value(
      "#{T(com.baeldung.loadresourceasstring.ResourceReader).readFileToString('classpath:resource.txt')}"
    )
    private String resourceStringUsingSpel;

    @Test
    public void givenUsingSpel_whenConvertingAResourceToAString_thenCorrect() {
        assertEquals(EXPECTED_RESOURCE_VALUE, resourceStringUsingSpel);
    }
}
```

这里我们调用了`ResourceReader#readFileToString`,通过在我们的`@Value` 注释中使用前缀为`“classpath:” –`的路径来描述文件的位置。

为了减少 SpEL 中的代码量，我们在类`ResourceReader`中创建了一个 helper 方法，它使用 Apache Commons `FileUtils`从提供的路径访问文件:

```
public class ResourceReader {
    public static String readFileToString(String path) throws IOException {
        return FileUtils.readFileToString(ResourceUtils.getFile(path), StandardCharsets.UTF_8);
    }
}
```

## 6.结论

在本教程中，我们回顾了一些将资源转换成 T2 的方法。

首先，我们看到了如何产生一个`Resource`来访问文件，以及如何从`Resource`读取到`String.`

接下来，我们还展示了如何隐藏资源加载实现，并通过在`@Configuration`中创建合格的 bean 来允许跨 bean 共享字符串内容，从而允许字符串自动连接。

最后，我们使用了 SpEL，它提供了一个紧凑而直接的解决方案，尽管它需要一个定制的助手函数来防止它变得过于复杂。

和往常一样，例子的代码可以在 GitHub 上找到