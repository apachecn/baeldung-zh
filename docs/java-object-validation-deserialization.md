# 反序列化后的对象验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-object-validation-deserialization>

## 1.概观

在本教程中，我们将看到如何使用 Java 的验证 API 来验证反序列化后的对象。

## 2.手动触发验证

Java 的用于 [bean 验证](/web/20221122042745/https://www.baeldung.com/javax-validation)的 API 在 [JSR 380](https://web.archive.org/web/20221122042745/https://jcp.org/en/jsr/detail?id=380) 中定义。它的一个常见用途是弹簧控制器中的 [`@Valid`注释参数。然而，在本文中，我们将关注控制器之外的验证。](/web/20221122042745/https://www.baeldung.com/spring-boot-bean-validation)

首先，让我们编写一个方法来验证一个对象的内容是否符合它的验证约束。为了做到这一点，我们将从默认的验证器工厂获取`Validator`。然后，我们将对对象应用`validate()`方法。这个方法返回一个 [`Set`](/web/20221122042745/https://www.baeldung.com/java-set-operations#1-what-is-a-set) 的`ConstraintViolation`。一个`ConstraintViolation`封装了一些关于验证错误的提示。为了简单起见，我们只抛出一个`ConstraintViolationException`，以防出现任何验证问题:

```
<T> void validate(T t) {
    Set<ConstraintViolation<T>> violations = validator.validate(t);
    if (!violations.isEmpty()) {
        throw new ConstraintViolationException(violations);
    }
}
```

如果我们在一个对象上调用这个方法，如果这个对象不考虑任何验证约束，它将抛出。可以在带有附加约束的现有对象上的任何点调用此方法。

## 3.将验证纳入反序列化过程

我们现在的目标是将验证合并到反序列化过程中。具体来说，我们将覆盖[杰克森](/web/20221122042745/https://www.baeldung.com/jackson)的反序列化器，以便在反序列化之后立即执行验证。这将确保任何时候我们反序列化一个对象，如果它不兼容，我们不允许任何进一步的处理。

首先，我们需要覆盖默认的`BeanDeserializer`。`BeanDeserializer`是一个可以反序列化对象的类。我们希望调用基本的反序列化方法，然后将 validate()方法应用于创建的实例。我们的 BeanDeserializerWithValidation 如下所示:

```
public class BeanDeserializerWithValidation extends BeanDeserializer {

    protected BeanDeserializerWithValidation(BeanDeserializerBase src) {
        super(src);
    }

    @Override
    public Object deserialize(JsonParser p, DeserializationContext ctxt) throws IOException {
        Object instance = super.deserialize(p, ctxt);
        validate(instance);
        return instance;
    }

}
```

下一步是实现我们自己的`BeanDeserializerModifier`。这将允许我们用`BeanDeserializerWithValidation`中定义的行为改变反序列化过程:

```
public class BeanDeserializerModifierWithValidation extends BeanDeserializerModifier {

    @Override
    public JsonDeserializer<?> modifyDeserializer(DeserializationConfig config, BeanDescription beanDesc, JsonDeserializer<?> deserializer) {
        if (deserializer instanceof BeanDeserializer) {
            return new BeanDeserializerWithValidation((BeanDeserializer) deserializer);
        }

        return deserializer;
    }

}
```

最后，我们需要创建一个 [`ObjectMapper`](/web/20221122042745/https://www.baeldung.com/jackson-object-mapper-tutorial) ，并将我们的`BeanDeserializerModifier`注册为一个`Module`。一个`Module`是扩展 Jackson 默认功能的一种方式。让我们用一种方法来包装它:

```
ObjectMapper getObjectMapperWithValidation() {
    SimpleModule validationModule = new SimpleModule();
    validationModule.setDeserializerModifier(new BeanDeserializerModifierWithValidation());
    ObjectMapper mapper = new ObjectMapper();
    mapper.registerModule(validationModule);
    return mapper;
}
```

## 4.示例用法:从文件中读取并验证对象

我们现在将展示一个如何使用我们的自定义`ObjectMapper`的小例子。首先，让我们定义一个`Student`对象。一个`Student`有名字。名称长度必须在 5 到 10 个字符之间:

```
public class Student {

    @Size(min = 5, max = 10, message = "Student's name must be between 5 and 10 characters")
    private String name;

    public String getName() {
        return name;
    }

}
```

现在让我们创建一个包含有效`Student`对象的 [JSON](/web/20221122042745/https://www.baeldung.com/java-json) 表示的`validStudent.json`文件:

```
{
  "name": "Daniel"
}
```

我们将在`InputStream`中读取这个文件的内容。首先，让我们定义一个方法，将一个`InputStream`解析为一个`Student`对象，并同时验证它。为此，我们想使用我们的`ObjectMapper`:

```
Student readStudent(InputStream inputStream) throws IOException {
    ObjectMapper mapper = getObjectMapperWithValidation();
    return mapper.readValue(inputStream, Student.class);
}
```

我们现在可以编写一个测试，其中我们将:

*   首先将文件内容读入一个`InputStream`
*   将`InputStream`转换为`Student`对象
*   检查`Student`对象的内容是否与预期一致

这个测试看起来像这样:

```
@Test
void givenValidStudent_WhenReadStudent_ThenReturnStudent() throws IOException {
    InputStream inputStream = getClass().getClassLoader().getResourceAsStream(("validStudent.json");
    Student result = readStudent(inputStream);
    assertEquals("Daniel", result.getName());
}
```

类似地，我们可以创建一个包含名称少于 5 个字符的`Student`的 JSON 表示的`invalid.json`文件:

```
{
  "name": "Max"
}
```

现在我们需要修改我们的测试来检查 ConstraintViolationException 是否确实被抛出。此外，我们可以检查错误消息是否正确:

```
@Test
void givenStudentWithInvalidName_WhenReadStudent_ThenThrows() {
    InputStream inputStream = getClass().getClassLoader().getResourceAsStream("invalidStudent.json");
    ConstraintViolationException constraintViolationException = assertThrows(ConstraintViolationException.class, () -> readStudent(inputStream));
    assertEquals("name: Student's name must be between 5 and 10 characters", constraintViolationException.getMessage());
}
```

## 5.结论

在本文中，我们看到了如何覆盖 Jackson 的配置，以便在反序列化后立即验证对象。因此，我们可以保证以后不可能在无效的对象上工作。

和往常一样，相关代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221122042745/https://github.com/eugenp/tutorials/tree/master/javax-validation-advanced)