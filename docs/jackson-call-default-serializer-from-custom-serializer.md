# 从 Jackson 中的自定义序列化程序调用默认序列化程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-call-default-serializer-from-custom-serializer>

## 1.介绍

使用所有字段的精确一对一表示将我们的完整数据结构序列化为 JSON 有时可能不合适，或者根本不是我们想要的。相反，**我们可能想要创建数据的扩展或简化视图。**这就是定制 Jackson 序列化程序发挥作用的地方。

然而，实现一个定制的序列化器可能是乏味的，特别是如果我们的模型对象有许多字段、集合或嵌套对象的话。幸运的是，[杰克逊图书馆](/web/20221126232844/https://www.baeldung.com/jackson)有几条规定可以让这项工作简单很多。

在这个简短的教程中，我们将看看定制的 Jackson 序列化器，并展示如何访问定制序列化器中的默认序列化器。

## 2.样本数据模型

在我们深入 Jackson 的定制之前，让我们看一下我们想要序列化的示例`Folder`类:

```
public class Folder {
    private Long id;
    private String name;
    private String owner;
    private Date created;
    private Date modified;
    private Date lastAccess;
    private List<File> files = new ArrayList<>();

    // standard getters and setters
} 
```

以及在我们的`Folder`类中被定义为`List`的`File`类:

```
public class File {
    private Long id;
    private String name;

    // standard getters and setters
} 
```

## 3.Jackson 中的自定义序列化程序

使用自定义序列化程序的主要优点是我们不必修改我们的类结构。另外，我们可以很容易地将我们期望的行为从类本身中分离出来。

因此，让我们假设我们想要我们的`Folder`类的一个简化视图:

```
{
    "name": "Root Folder",
    "files": [
        {"id": 1, "name": "File 1"},
        {"id": 2, "name": "File 2"}
    ]
} 
```

正如我们将在接下来的章节中看到的，有几种方法可以在 Jackson 中实现我们想要的输出。

### 3.1.强力方法

首先，在不使用 Jackson 的默认序列化程序的情况下，我们可以创建一个自定义序列化程序，在其中我们自己完成所有繁重的工作。

让我们为我们的`Folder`类创建一个定制的序列化程序来实现这一点:

```
public class FolderJsonSerializer extends StdSerializer<Folder> {

    public FolderJsonSerializer() {
        super(Folder.class);
    }

    @Override
    public void serialize(Folder value, JsonGenerator gen, SerializerProvider provider)
      throws IOException {
        gen.writeStartObject();
        gen.writeStringField("name", value.getName());

        gen.writeArrayFieldStart("files");
        for (File file : value.getFiles()) {
            gen.writeStartObject();
            gen.writeNumberField("id", file.getId());
            gen.writeStringField("name", file.getName());
            gen.writeEndObject();
        }
        gen.writeEndArray();

        gen.writeEndObject();
    }
}
```

因此，我们可以将我们的`Folder`类序列化为一个精简的视图，只包含我们想要的字段。

### 3.2.使用内部`ObjectMapper`

尽管定制的序列化器为我们提供了改变每个属性细节的灵活性，但是我们可以通过重用 Jackson 的默认序列化器来简化我们的工作。

使用默认序列化程序的一种方法是访问内部的`ObjectMapper`类:

```
@Override
public void serialize(Folder value, JsonGenerator gen, SerializerProvider provider) throws IOException {
    gen.writeStartObject();
    gen.writeStringField("name", value.getName());

    ObjectMapper mapper = (ObjectMapper) gen.getCodec();
    gen.writeFieldName("files");
    String stringValue = mapper.writeValueAsString(value.getFiles());
    gen.writeRawValue(stringValue);

    gen.writeEndObject();
} 
```

所以，Jackson 只是通过序列化`File`对象的`List`来处理繁重的工作，然后我们的输出将是相同的。

### 3.3.使用`SerializerProvider`

调用默认序列化程序的另一种方式是使用`SerializerProvider.`，因此，我们将流程委托给类型为`File`的默认序列化程序。

现在，让我们在`SerializerProvider`的帮助下稍微简化一下我们的代码:

```
@Override
public void serialize(Folder value, JsonGenerator gen, SerializerProvider provider) throws IOException {
    gen.writeStartObject();
    gen.writeStringField("name", value.getName());

    provider.defaultSerializeField("files", value.getFiles(), gen);

    gen.writeEndObject();
} 
```

和以前一样，我们得到相同的输出。

## 4.一个可能的递归问题

根据用例的不同，我们可能需要通过包含更多关于`Folder`的细节来扩展我们的序列化数据。这可能是为了**集成一个遗留系统或外部应用程序，我们没有机会修改**。

让我们更改我们的序列化程序，为我们的序列化数据创建一个`details`字段，以简单地公开`Folder`类的所有字段:

```
@Override
public void serialize(Folder value, JsonGenerator gen, SerializerProvider provider) throws IOException {
    gen.writeStartObject();
    gen.writeStringField("name", value.getName());

    provider.defaultSerializeField("files", value.getFiles(), gen);

    // this line causes exception
    provider.defaultSerializeField("details", value, gen);

    gen.writeEndObject();
} 
```

这次我们得到了一个`StackOverflowError`异常。

**当我们定义一个自定义序列化程序时，Jackson 在内部覆盖了为类型`Folder`创建的原始`BeanSerializer`实例**。因此，我们的`SerializerProvider`每次都找到定制的序列化程序，而不是默认的，这导致了一个无限循环。

那么，我们如何解决这个问题呢？在下一节中，我们将看到这种场景的一个可用解决方案。

## 5.使用`BeanSerializerModifier`

一个可能的解决方法是在 Jackson 内部覆盖类型`Folder` **之前，使用`BeanSerializerModifier` **来存储默认的序列化器**。**

让我们修改我们的序列化程序并添加一个额外的字段— `defaultSerializer`:

```
private final JsonSerializer<Object> defaultSerializer;

public FolderJsonSerializer(JsonSerializer<Object> defaultSerializer) {
    super(Folder.class);
    this.defaultSerializer = defaultSerializer;
} 
```

接下来，我们将创建一个`BeanSerializerModifier`的实现来传递默认的序列化程序:

```
public class FolderBeanSerializerModifier extends BeanSerializerModifier {

    @Override
    public JsonSerializer<?> modifySerializer(
      SerializationConfig config, BeanDescription beanDesc, JsonSerializer<?> serializer) {

        if (beanDesc.getBeanClass().equals(Folder.class)) {
            return new FolderJsonSerializer((JsonSerializer<Object>) serializer);
        }

        return serializer;
    }
} 
```

现在，我们需要将我们的`BeanSerializerModifier`注册为一个模块来使它工作:

```
ObjectMapper mapper = new ObjectMapper();

SimpleModule module = new SimpleModule();
module.setSerializerModifier(new FolderBeanSerializerModifier());

mapper.registerModule(module); 
```

然后，我们将`defaultSerializer`用于`details`字段:

```
@Override
public void serialize(Folder value, JsonGenerator gen, SerializerProvider provider) throws IOException {
    gen.writeStartObject();
    gen.writeStringField("name", value.getName());

    provider.defaultSerializeField("files", value.getFiles(), gen);

    gen.writeFieldName("details");
    defaultSerializer.serialize(value, gen, provider);

    gen.writeEndObject();
} 
```

最后，我们可能希望从`details`中移除`files`字段，因为我们已经将它单独写入序列化数据。

因此，我们简单地忽略了我们的`Folder`类中的`files`字段:

```
@JsonIgnore
private List<File> files = new ArrayList<>(); 
```

最后，问题得到了解决，我们也获得了预期的输出:

```
{
    "name": "Root Folder",
    "files": [
        {"id": 1, "name": "File 1"},
        {"id": 2, "name": "File 2"}
    ],
    "details": {
        "id":1,
        "name": "Root Folder",
        "owner": "root",
        "created": 1565203657164,
        "modified": 1565203657164,
        "lastAccess": 1565203657164
    }
} 
```

## 6.结论

在本教程中，我们学习了如何在 Jackson 库中的自定义序列化程序中调用默认序列化程序。

像往常一样，本教程中使用的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221126232844/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-custom-conversions)