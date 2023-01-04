# 在 Spring 中使用枚举作为请求参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-enum-request-param>

## 1.介绍

在大多数典型的 web 应用程序中，我们经常需要将请求参数限制为一组预定义的值。枚举是做到这一点的好方法。

在这个快速教程中，我们将演示如何在 Spring MVC 中使用 enums 作为 web 请求参数。

## 2.使用枚举作为请求参数

让我们首先为我们的示例定义一个枚举:

```
public enum Modes {
    ALPHA, BETA;
} 
```

然后我们可以在 Spring 控制器中使用这个枚举作为`RequestParameter`:

```
@GetMapping("/mode2str")
public String getStringToMode(@RequestParam("mode") Modes mode) {
    // ...
} 
```

或者我们可以把它作为一个`PathVariable`:

```
@GetMapping("/findbymode/{mode}")
public String findByEnum(@PathVariable("mode") Modes mode) {
    // ...
} 
```

当我们发出一个 web 请求时，比如说`/mode2str?mode=ALPHA`，请求参数是一个`String`对象。Spring 可以尝试通过使用它的 [`StringToEnumConverterFactory`](https://web.archive.org/web/20221126235036/https://github.com/spring-projects/spring-framework/blob/master/spring-core/src/main/java/org/springframework/core/convert/support/StringToEnumConverterFactory.java) 类将这个`String`对象转换成一个`Enum`对象。

后端转换采用 [`Enum.valueOf`](https://web.archive.org/web/20221126235036/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Enum.html#valueOf(java.lang.Class,java.lang.String)) 的方法。因此，**输入名称字符串必须与声明的枚举值之一**完全匹配。

**当我们使用与我们的枚举值**之一不匹配的字符串值发出 web 请求时，like `/mode2str?mode=unknown, ` Spring 将无法将其转换为指定的枚举类型。在这种情况下，**我们会得到一个`ConversionFailedException` `.`**

## 3.自定义转换器

在 Java 中，用大写字母定义枚举值被认为是一种很好的做法，因为它们是常量。然而，我们可能希望在请求 URL 中支持小写字母。

在这种情况下，我们需要创建一个自定义转换器:

```
public class StringToEnumConverter implements Converter<String, Modes> {
    @Override
    public Modes convert(String source) {
        return Modes.valueOf(source.toUpperCase());
    }
} 
```

要使用我们的定制转换器，我们需要**在 Spring 配置**中注册它:

```
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new StringToEnumConverter());
    }
} 
```

## 4.异常处理

如果我们的 `Modes`枚举没有匹配的常量，`StringToEnumConverter` 中的`Enum.valueOf`方法将抛出一个`IllegalArgumentException`。根据我们的需求，我们可以在自定义转换器中以不同的方式处理这个异常。

例如，我们可以简单地让转换器为不匹配的`String` s 返回`null`:

```
public class StringToEnumConverter implements Converter<String, Modes> {
    @Override
    public Modes convert(String source) {
        try {
            return Modes.valueOf(source.toUpperCase());
        } catch (IllegalArgumentException e) {
            return null;
        }
    }
} 
```

然而，如果我们不在自定义转换器中本地处理异常，Spring 将向调用控制器方法抛出一个`ConversionFailedException`异常。有几种方法可以处理这个异常。

例如，我们可以使用全局异常处理程序类:

```
@ControllerAdvice
public class GlobalControllerExceptionHandler {
    @ExceptionHandler(ConversionFailedException.class)
    public ResponseEntity<String> handleConflict(RuntimeException ex) {
        return new ResponseEntity<>(ex.getMessage(), HttpStatus.BAD_REQUEST);
    }
}
```

## 5.结论

在这篇简短的文章中，我们通过一些代码示例学习了如何在 Spring 中将枚举用作请求参数。

我们还提供了一个定制的转换器示例，它可以将输入字符串映射到一个枚举常量。

最后，我们讨论了如何处理 Spring 在遇到未知输入字符串时抛出的异常。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221126235036/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-3)