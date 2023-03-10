# Spring Boot 的 Enum 映射

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-enum-mapping>

## 1.概观

在本教程中，我们将探索在 Spring Boot 中实现不区分大小写的枚举映射的不同方法。

首先，我们将看到在 Spring 中枚举是如何默认映射的。然后，我们将学习如何应对区分大小写的挑战。

## 2.Spring 中的默认枚举映射

在处理请求参数时，Spring 依靠几个内置的转换器来处理字符串转换。

通常，当我们传递一个 enum 作为请求参数**时，它使用 [`StringToEnumConverterFactory`](https://web.archive.org/web/20221128144746/https://github.com/spring-projects/spring-framework/blob/main/spring-core/src/main/java/org/springframework/core/convert/support/StringToEnumConverterFactory.java) 将传递的`String`转换为`enum`** `.`

按照设计，这个转换器调用`Enum.valueOf(Class, String)` **，这意味着给定的字符串必须与声明的枚举常量**之一完全匹配。

例如，让我们考虑一下`Level`枚举:

```java
public enum Level {
    LOW, MEDIUM, HIGH
}
```

接下来，让我们创建一个接受我们的枚举作为参数的[处理程序方法](/web/20221128144746/https://www.baeldung.com/spring-handler-mappings):

```java
@RestController
@RequestMapping("enummapping")
public class EnumMappingController {

    @GetMapping("/get")
    public String getByLevel(@RequestParam(required = false) Level level){
        return level.name();
    }

}
```

让我们使用 [CURL](/web/20221128144746/https://www.baeldung.com/curl-rest) 向`http://localhost:8080/enummapping/get?level=MEDIUM`发送一个请求:

```java
curl http://localhost:8080/enummapping/get?level=MEDIUM
```

handler 方法发回`MEDIUM`，枚举常量`MEDIUM`的名称。

现在，让我们通过`medium`而不是`MEDIUM`，看看会发生什么:

```java
curl http://localhost:8080/enummapping/get?level=medium
{"timestamp":"2022-11-18T18:41:11.440+00:00","status":400,"error":"Bad Request","path":"/enummapping/get"}
```

正如我们所看到的，请求被认为是无效的，应用程序失败并显示一个错误:

```java
Failed to convert value of type 'java.lang.String' to required type 'com.baeldung.enummapping.enums.Level'; 
nested exception is org.springframework.core.convert.ConversionFailedException: Failed to convert from type [java.lang.String] to type [@org.springframework.web.bind.annotation.RequestParam com.baeldung.enummapping.enums.Level] for value 'medium'; 
...
```

查看堆栈跟踪，我们可以看到 Spring 抛出了`ConversionFailedException.` ，它没有将`medium`识别为枚举常量。

## 3.不区分大小写的枚举映射

Spring 提供了几种方便的方法来解决映射枚举时区分大小写的问题。

让我们仔细看看每种方法。

### 3.1.使用`ApplicationConversionService`

[`ApplicationConversionService`](https://web.archive.org/web/20221128144746/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/convert/ApplicationConversionService.html) 类附带了一组已配置的转换器和格式化程序。

在这些开箱即用的转换器中，我们发现`StringToEnumIgnoringCaseConverterFactory.` **顾名思义，它以不区分大小写的方式将字符串转换为枚举**。

首先，我们需要添加和配置`ApplicationConversionService`:

```java
@Configuration
public class EnumMappingConfig implements WebMvcConfigurer {
    @Override
    public void addFormatters(FormatterRegistry registry) {
        ApplicationConversionService.configure(registry);
    }
}
```

该类使用适合大多数 Spring Boot 应用的现成转换器来配置 [`FormatterRegistry`](https://web.archive.org/web/20221128144746/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/format/FormatterRegistry.html) 。

现在，让我们使用一个测试用例来确认一切都按预期工作:

```java
@RunWith(SpringRunner.class)
@WebMvcTest(EnumMappingController.class)
public class EnumMappingIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void whenPassingLowerCaseEnumConstant_thenConvert() throws Exception {
        mockMvc.perform(get("/enummapping/get?level=medium"))
            .andExpect(status().isOk())
            .andExpect(content().string(Level.MEDIUM.name()));
    }

}
```

我们可以看到，作为参数传递的`medium`值被成功地转换为`MEDIUM`。

### 3.2.使用自定义转换器

另一个解决方案是使用自定义转换器。这里，我们将使用 Apache Commons Lang 3 库。

首先，我们需要添加它的[依赖关系](https://web.archive.org/web/20221128144746/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22):

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

**这里的基本思想是创建一个转换器，将一个`Level` 常量的字符串表示转换成一个真实的`Level`常量:**

```java
public class StringToLevelConverter implements Converter<String, Level> {

    @Override
    public Level convert(String source) {
        if (StringUtils.isBlank(source)) {
            return null;
        }
        return EnumUtils.getEnum(Level.class, source.toUpperCase());
    }

}
```

**从技术的角度来看，自定义转换器是一个简单的类，它实现了`Converter<S,T>`接口**。

正如我们所见，我们将`String`对象转换成了大写。然后，我们使用 Apache Commons Lang 3 库中的 [`EnumUtils`](https://web.archive.org/web/20221128144746/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/EnumUtils.html) 实用程序类从大写字符串中获取`Level`常量。

现在，让我们把最后一块拼图补上。我们需要告诉 Spring 我们新的定制转换器。为此，我们将使用与之前相同的`FormatterRegistry`。**它提供了`addConverter()`方法来注册自定义转换器**:

```java
@Override
public void addFormatters(FormatterRegistry registry) {
    registry.addConverter(new StringToLevelConverter());
}
```

仅此而已。我们的`StringToLevelConverter`现在已经在 [`ConversionService`](https://web.archive.org/web/20221128144746/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/convert/ConversionService.html) 中了。

现在，我们可以像使用任何其他转换器一样使用它:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = EnumMappingMainApplication.class)
public class StringToLevelConverterIntegrationTest {

    @Autowired
    ConversionService conversionService;

    @Test
    public void whenConvertStringToLevelEnumUsingCustomConverter_thenSuccess() {
        assertThat(conversionService.convert("low", Level.class)).isEqualTo(Level.LOW);
    }

}
```

如上图所示，测试用例确认了`“low”` 值被转换为`Level.LOW`。

### 3.3.使用自定义属性编辑器

Spring 使用多个内置的[属性编辑器](/web/20221128144746/https://www.baeldung.com/spring-mvc-custom-property-editor)来管理`String`值和 Java 对象之间的转换。

类似地，我们可以创建一个定制的属性编辑器来将`String`对象映射到`Level` 常量。

例如，让我们将自定义编辑器命名为`LevelEditor`:

```java
public class LevelEditor extends PropertyEditorSupport {

    @Override
    public void setAsText(String text) {
        if (StringUtils.isBlank(text)) {
            setValue(null);
        } else {
            setValue(EnumUtils.getEnum(Level.class, text.toUpperCase()));
        }
    }
}
```

正如我们所看到的，我们需要扩展 [`PropertyEditorSupport`](https://web.archive.org/web/20221128144746/https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/java/beans/PropertyEditorSupport.html) 类并覆盖`setAsText()`方法。

**覆盖 s `etAsText()`的想法是将给定字符串的大写版本转换成`Level` 枚举**。

值得注意的是，`PropertyEditorSupport`也提供了`getAsText()`。将 Java 对象序列化为字符串时会调用它。所以，我们不需要在这里覆盖它。

我们需要注册我们的`LevelEditor`,因为 Spring 不会自动检测定制属性编辑器。**为此，我们需要在我们的[弹簧控制器](/web/20221128144746/https://www.baeldung.com/spring-controllers)** 中创建一个用`@InitBinder`标注的方法:

```java
@InitBinder
public void initBinder(WebDataBinder dataBinder) {
    dataBinder.registerCustomEditor(Level.class, new LevelEditor());
}
```

现在我们把所有的部分放在一起，让我们使用一个测试用例来确认我们的定制属性编辑器`LevelEditor`工作正常:

```java
public class LevelEditorIntegrationTest {

    @Test
    public void whenConvertStringToLevelEnumUsingCustomPropertyEditor_thenSuccess() {
        LevelEditor levelEditor = new LevelEditor();
        levelEditor.setAsText("lOw");

        assertThat(levelEditor.getValue()).isEqualTo(Level.LOW);
    }
}
```

这里要提到的另一件重要的事情是，`EnumUtils.getEnum()`在找到时返回枚举，否则返回`null` 。

因此，为了避免`NullPointerException`，我们需要稍微改变一下我们的处理方法:

```java
public String getByLevel(@RequestParam(required = false) Level level) {
    if (level != null) {
        return level.name();
    }
    return "undefined";
}
```

现在，让我们添加一个简单的测试用例来对此进行测试:

```java
@Test
public void whenPassingUnknownEnumConstant_thenReturnUndefined() throws Exception {
    mockMvc.perform(get("/enummapping/get?level=unknown"))
        .andExpect(status().isOk())
        .andExpect(content().string("undefined"));
}
```

## 4.结论

在本文中，我们学习了在 Spring 中实现不区分大小写的枚举映射的各种方法。

在此过程中，我们研究了一些使用内置和定制转换器实现这一目的的方法。然后，我们看到了如何使用自定义属性编辑器来实现相同的目标。

和往常一样，本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221128144746/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-request-params)