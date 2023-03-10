# Spring REST API 中二进制数据格式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-rest-api-with-binary-data-formats>

## 1.概观

虽然 JSON 和 XML 是 REST APIs 广泛流行的数据传输格式，但它们并不是唯一可用的选项。

存在许多其他格式，它们具有不同程度的序列化速度和序列化数据大小。

在本文中，我们将探索如何配置一个 Spring REST 机制来使用二进制数据格式——我们用 Kryo 来说明。

此外，我们展示了如何通过添加对 Google 协议缓冲区的支持来支持多种数据格式。

## 2.`HttpMessageConverter`

`HttpMessageConverter`接口基本上是 Spring 的公共 API，用于 REST 数据格式的转换。

指定所需转换器有不同的方法。这里我们实现了`WebMvcConfigurer`并明确地提供了我们想要在被覆盖的`configureMessageConverters`方法中使用的转换器:

```java
@Configuration
@EnableWebMvc
@ComponentScan({ "com.baeldung.web" })
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> messageConverters) {
        //...
    }
}
```

## 3.克里欧

### 3.1。Kryo 概述和 Maven

Kryo 是一种二进制编码格式，与基于文本的格式相比，它提供了良好的序列化和反序列化速度以及较小的传输数据大小。

虽然理论上它可以用来在不同类型的系统之间传输数据，但它主要是为与 Java 组件一起工作而设计的。

我们使用以下 Maven 依赖项添加必要的 Kryo 库:

```java
<dependency>
    <groupId>com.esotericsoftware</groupId>
    <artifactId>kryo</artifactId>
    <version>4.0.0</version>
</dependency>
```

要查看最新版本的`kryo` 你可以让[看一下这里的](https://web.archive.org/web/20220120134716/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.esotericsoftware%22%20AND%20a%3A%22kryo%22)。

### 3.2。Kryo in Spring REST

为了利用 Kryo 作为数据传输格式，我们创建了一个定制的`HttpMessageConverter`并实现了必要的序列化和反序列化逻辑。此外，我们为 Kryo: `application/x-kryo`定义了自定义的 HTTP 头。这是一个完全简化的工作示例，我们用于演示目的:

```java
public class KryoHttpMessageConverter extends AbstractHttpMessageConverter<Object> {

    public static final MediaType KRYO = new MediaType("application", "x-kryo");

    private static final ThreadLocal<Kryo> kryoThreadLocal = new ThreadLocal<Kryo>() {
        @Override
        protected Kryo initialValue() {
            Kryo kryo = new Kryo();
            kryo.register(Foo.class, 1);
            return kryo;
        }
    };

    public KryoHttpMessageConverter() {
        super(KRYO);
    }

    @Override
    protected boolean supports(Class<?> clazz) {
        return Object.class.isAssignableFrom(clazz);
    }

    @Override
    protected Object readInternal(
      Class<? extends Object> clazz, HttpInputMessage inputMessage) throws IOException {
        Input input = new Input(inputMessage.getBody());
        return kryoThreadLocal.get().readClassAndObject(input);
    }

    @Override
    protected void writeInternal(
      Object object, HttpOutputMessage outputMessage) throws IOException {
        Output output = new Output(outputMessage.getBody());
        kryoThreadLocal.get().writeClassAndObject(output, object);
        output.flush();
    }

    @Override
    protected MediaType getDefaultContentType(Object object) {
        return KRYO;
    }
}
```

请注意，我们在这里使用`ThreadLocal`只是因为创建 Kryo 实例会很昂贵，我们希望尽可能多地重用这些实例。

控制器方法很简单(注意，不需要任何定制的特定于协议的数据类型，我们使用普通的`Foo` DTO):

```java
@RequestMapping(method = RequestMethod.GET, value = "/foos/{id}")
@ResponseBody
public Foo findById(@PathVariable long id) {
    return fooRepository.findById(id);
}
```

以及一个快速测试来证明我们已经正确地将所有东西连接在一起:

```java
RestTemplate restTemplate = new RestTemplate();
restTemplate.setMessageConverters(Arrays.asList(new KryoHttpMessageConverter()));

HttpHeaders headers = new HttpHeaders();
headers.setAccept(Arrays.asList(KryoHttpMessageConverter.KRYO));
HttpEntity<String> entity = new HttpEntity<String>(headers);

ResponseEntity<Foo> response = restTemplate.exchange("http://localhost:8080/spring-rest/foos/{id}",
  HttpMethod.GET, entity, Foo.class, "1");
Foo resource = response.getBody();

assertThat(resource, notNullValue());
```

## 4.支持多种数据格式

通常，您会希望为同一服务提供对多种数据格式的支持。客户端在`Accept` HTTP 头中指定所需的数据格式，并调用相应的消息转换器来序列化数据。

通常，你只需要注册另一个转换器就可以开箱即用了。Spring 根据`Accept`头中的值和转换器中声明的支持的媒体类型自动选择合适的转换器。

例如，要添加对 JSON 和 Kryo 的支持，需要注册`KryoHttpMessageConverter`和`MappingJackson2HttpMessageConverter`:

```java
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> messageConverters) {
    messageConverters.add(new MappingJackson2HttpMessageConverter());
    messageConverters.add(new KryoHttpMessageConverter());
    super.configureMessageConverters(messageConverters);
} 
```

现在，让我们假设我们也想将 Google 协议缓冲区添加到列表中。对于这个例子，我们假设有一个由`protoc`编译器基于下面的`proto`文件生成的类`FooProtos.Foo`:

```java
package baeldung;
option java_package = "com.baeldung.web.dto";
option java_outer_classname = "FooProtos";
message Foo {
    required int64 id = 1;
    required string name = 2;
}
```

Spring 自带了一些对协议缓冲区的内置支持。我们需要做的就是将`ProtobufHttpMessageConverter`包含在支持的转换器列表中:

```java
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> messageConverters) {
    messageConverters.add(new MappingJackson2HttpMessageConverter());
    messageConverters.add(new KryoHttpMessageConverter());
    messageConverters.add(new ProtobufHttpMessageConverter());
}
```

然而，我们必须定义一个单独的返回`FooProtos.Foo`实例的控制器方法(JSON 和 Kryo 都处理`Foo` s，所以不需要在控制器中做任何改变来区分这两者)。

有两种方法可以解决调用哪个方法的模糊性。第一种方法是对 protobuf 和其他格式使用不同的 URL。例如，对于 protobuf:

```java
@RequestMapping(method = RequestMethod.GET, value = "/fooprotos/{id}")
@ResponseBody
public FooProtos.Foo findProtoById(@PathVariable long id) { … } 
```

对于其他人来说:

```java
@RequestMapping(method = RequestMethod.GET, value = "/foos/{id}")
@ResponseBody
public Foo findById(@PathVariable long id) { … } 
```

注意，对于 protobuf，我们使用`value = “/fooprotos/{id}”`，对于其他格式，我们使用`value = “/foos/{id}”`。

**第二种也是更好的方法是使用相同的 URL，但是在 protobuf 的请求映射中明确指定生成的数据格式:**

```java
@RequestMapping(
  method = RequestMethod.GET, 
  value = "/foos/{id}", 
  produces = { "application/x-protobuf" })
@ResponseBody
public FooProtos.Foo findProtoById(@PathVariable long id) { … } 
```

注意，通过在`produces`注释属性中指定媒体类型，我们根据客户端提供的`Accept`头中的值向底层 Spring 机制提供了关于需要使用哪个映射的提示，因此对于需要为`“foos/{id}”` URL 调用哪个方法没有歧义。

第二种方法使我们能够为所有数据格式的客户端提供统一一致的 REST API。

最后，如果您对在 Spring REST API 中使用协议缓冲区感兴趣，可以看看参考文章。

## 5.注册额外的消息转换器

值得注意的是，当您覆盖`configureMessageConverters`方法时，您会丢失所有默认的消息转换器。只会使用您提供的信息。

虽然有时这正是您想要的，但在许多情况下，您只是想添加新的转换器，同时仍然保留默认的转换器，这些转换器已经处理了 JSON 等标准数据格式。为此，重写`extendMessageConverters`方法:

```java
@Configuration
@EnableWebMvc
@ComponentScan({ "com.baeldung.web" })
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> messageConverters) {
        messageConverters.add(new ProtobufHttpMessageConverter());
        messageConverters.add(new KryoHttpMessageConverter());
    }
}
```

## 6.结论

在本教程中，我们看到了在 Spring MVC 中使用任何数据传输格式是多么容易，我们以 Kryo 为例对此进行了检验。

我们还展示了如何添加对多种格式的支持，以便不同的客户端能够使用不同的格式。

Spring REST API 教程[中这种二进制数据格式的实现当然是在 Github](https://web.archive.org/web/20220120134716/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-simple) 上。这是一个基于 Maven 的项目，因此应该很容易导入和运行。