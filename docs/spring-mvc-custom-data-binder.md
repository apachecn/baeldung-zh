# Spring MVC 中的自定义数据绑定器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-custom-data-binder>

## 1。概述

本文将展示如何使用 Spring 的数据绑定机制，通过将自动原语应用于对象转换来使我们的代码更加清晰易读。

默认情况下，Spring 只知道如何转换简单类型。换句话说，一旦我们将数据提交给控制器`Int`、`String`或`Boolean`类型的数据，它将自动绑定到适当的 Java 类型。

但是在现实世界的项目中，这还不够，因为**我们可能需要绑定更复杂类型的对象**。

## 2。将单个对象绑定到请求参数

先从简单开始，先绑定一个简单类型；我们必须提供一个自定义的 [`Converter<S, T>`](https://web.archive.org/web/20220629014624/https://docs.spring.io/spring/docs/3.0.x/spring-framework-reference/htmlsingle/spring-framework-reference.html#core-convert-Converter-API) 接口实现，其中`S`是我们要转换的类型，`T`是我们要转换的类型:

```
@Component
public class StringToLocalDateTimeConverter
  implements Converter<String, LocalDateTime> {

    @Override
    public LocalDateTime convert(String source) {
        return LocalDateTime.parse(
          source, DateTimeFormatter.ISO_LOCAL_DATE_TIME);
    }
}
```

现在，我们可以在控制器中使用以下语法:

```
@GetMapping("/findbydate/{date}")
public GenericEntity findByDate(@PathVariable("date") LocalDateTime date) {
    return ...;
}
```

### 2.1。使用枚举作为请求参数

接下来，我们将看到**如何使用 e `num`作为`RequestParameter`** 。

在这里，我们有一个简单的`enum` `Modes`:

```
public enum Modes {
    ALPHA, BETA;
}
```

我们将构建一个`String`到`enum Converter`，如下所示:

```
public class StringToEnumConverter implements Converter<String, Modes> {

    @Override
    public Modes convert(String from) {
        return Modes.valueOf(from);
    }
}
```

然后，我们需要注册我们的`Converter`:

```
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new StringToEnumConverter());
    }
}
```

现在我们可以用我们的`Enum`作为`RequestParameter`:

```
@GetMapping
public ResponseEntity<Object> getStringToMode(@RequestParam("mode") Modes mode) {
    // ...
}
```

或者作为`PathVariable`:

```
@GetMapping("/entity/findbymode/{mode}")
public GenericEntity findByEnum(@PathVariable("mode") Modes mode) {
    // ...
}
```

## 3。绑定对象层次结构

有时我们需要转换对象层次结构的整个树，使用更集中的绑定比一组单独的转换器更有意义。

在这个例子中，我们有`AbstractEntity`我们的基类:

```
public abstract class AbstractEntity {
    long id;
    public AbstractEntity(long id){
        this.id = id;
    }
}
```

子类`Foo`和`Bar`:

```
public class Foo extends AbstractEntity {
    private String name;

    // standard constructors, getters, setters
}
```

```
public class Bar extends AbstractEntity {
    private int value;

    // standard constructors, getters, setters
}
```

在这种情况下，**我们可以实现`ConverterFactory<S, R>`，其中 S 是我们要转换的类型，R 是基本类型**，它定义了我们可以转换的类的范围:

```
public class StringToAbstractEntityConverterFactory 
  implements ConverterFactory<String, AbstractEntity>{

    @Override
    public <T extends AbstractEntity> Converter<String, T> getConverter(Class<T> targetClass) {
        return new StringToAbstractEntityConverter<>(targetClass);
    }

    private static class StringToAbstractEntityConverter<T extends AbstractEntity>
      implements Converter<String, T> {

        private Class<T> targetClass;

        public StringToAbstractEntityConverter(Class<T> targetClass) {
            this.targetClass = targetClass;
        }

        @Override
        public T convert(String source) {
            long id = Long.parseLong(source);
            if(this.targetClass == Foo.class) {
                return (T) new Foo(id);
            }
            else if(this.targetClass == Bar.class) {
                return (T) new Bar(id);
            } else {
                return null;
            }
        }
    }
}
```

正如我们所见，必须实现的唯一方法是`getConverter()`，它返回所需类型的转换器。然后，转换过程被委托给该转换器。

然后，我们需要注册我们的`ConverterFactory`:

```
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverterFactory(new StringToAbstractEntityConverterFactory());
    }
}
```

最后，我们可以在控制器中随心所欲地使用它:

```
@RestController
@RequestMapping("/string-to-abstract")
public class AbstractEntityController {

    @GetMapping("/foo/{foo}")
    public ResponseEntity<Object> getStringToFoo(@PathVariable Foo foo) {
        return ResponseEntity.ok(foo);
    }

    @GetMapping("/bar/{bar}")
    public ResponseEntity<Object> getStringToBar(@PathVariable Bar bar) {
        return ResponseEntity.ok(bar);
    }
}
```

## 4。绑定域对象

有些情况下，我们希望将数据绑定到对象，但它要么以非直接的方式出现(例如，来自`Session`、`Header`或`Cookie`变量)，要么甚至存储在数据源中。在这些情况下，我们需要使用不同的解决方案。

### 4.1。自定义参数解析器

首先，我们将为这些参数定义一个注释:

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.PARAMETER)
public @interface Version {
}
```

然后，我们将实现一个自定义`[HandlerMethodArgumentResolver](https://web.archive.org/web/20220629014624/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/method/support/HandlerMethodArgumentResolver.html):`

```
public class HeaderVersionArgumentResolver
  implements HandlerMethodArgumentResolver {

    @Override
    public boolean supportsParameter(MethodParameter methodParameter) {
        return methodParameter.getParameterAnnotation(Version.class) != null;
    }

    @Override
    public Object resolveArgument(
      MethodParameter methodParameter, 
      ModelAndViewContainer modelAndViewContainer, 
      NativeWebRequest nativeWebRequest, 
      WebDataBinderFactory webDataBinderFactory) throws Exception {

        HttpServletRequest request 
          = (HttpServletRequest) nativeWebRequest.getNativeRequest();

        return request.getHeader("Version");
    }
}
```

最后一件事是让 Spring 知道在哪里搜索它们:

```
@Configuration
public class WebConfig implements WebMvcConfigurer {

    //...

    @Override
    public void addArgumentResolvers(
      List<HandlerMethodArgumentResolver> argumentResolvers) {
        argumentResolvers.add(new HeaderVersionArgumentResolver());
    }
}
```

就是这样。现在我们可以在控制器中使用它:

```
@GetMapping("/entity/{id}")
public ResponseEntity findByVersion(
  @PathVariable Long id, @Version String version) {
    return ...;
}
```

正如我们所见，`HandlerMethodArgumentResolver`的`resolveArgument()`方法返回一个`Object.` ，换句话说，我们可以返回任何对象，而不仅仅是`String`。

## 5。结论

结果，我们摆脱了许多常规转换，让 Spring 为我们做了大部分事情。最后，让我们总结一下:

*   对于单个简单的类型到对象的转换，我们应该使用`Converter`实现
*   对于封装一系列对象的转换逻辑，我们可以尝试`ConverterFactory` 实现
*   对于任何间接数据或需要应用额外逻辑来检索相关数据的数据，最好使用`HandlerMethodArgumentResolver`

像往常一样，所有的例子都可以在我们的 [GitHub 库](https://web.archive.org/web/20220629014624/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-3)中找到。