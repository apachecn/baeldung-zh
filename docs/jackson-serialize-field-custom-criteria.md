# 仅序列化符合 Jackson 自定义标准的字段

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jackson-serialize-field-custom-criteria>

## 1。概述

本教程将说明如何使用 Jackson 来序列化一个满足特定自定义标准的字段。

例如，假设我们只希望序列化一个正值的整数值，如果不是，我们希望完全跳过它。

如果你想更深入地了解和学习其他很酷的事情，你可以用杰克逊 2 号来做——直接去[杰克逊的主要教程](/web/20221129010951/https://www.baeldung.com/jackson "The Jackson JSON Guide")。

## 2。使用杰克逊过滤器控制序列化过程

首先，我们需要在我们的实体上定义过滤器，使用`@JsonFilter`注释:

```
@JsonFilter("myFilter")
public class MyDto {
    private int intValue;

    public MyDto() {
        super();
    }

    public int getIntValue() {
        return intValue;
    }

    public void setIntValue(int intValue) {
        this.intValue = intValue;
    }
}
```

然后，我们需要定义我们的定制`PropertyFilter`:

```
PropertyFilter theFilter = new SimpleBeanPropertyFilter() {
   @Override
   public void serializeAsField
    (Object pojo, JsonGenerator jgen, SerializerProvider provider, PropertyWriter writer)
     throws Exception {
      if (include(writer)) {
         if (!writer.getName().equals("intValue")) {
            writer.serializeAsField(pojo, jgen, provider);
            return;
         }
         int intValue = ((MyDtoWithFilter) pojo).getIntValue();
         if (intValue >= 0) {
            writer.serializeAsField(pojo, jgen, provider);
         }
      } else if (!jgen.canOmitFields()) { // since 2.3
         writer.serializeAsOmittedField(pojo, jgen, provider);
      }
   }
   @Override
   protected boolean include(BeanPropertyWriter writer) {
      return true;
   }
   @Override
   protected boolean include(PropertyWriter writer) {
      return true;
   }
};
```

该过滤器包含基于其值决定`intValue`字段是否将被**序列化或不被**序列化的实际逻辑。

接下来，我们将这个过滤器挂接到`ObjectMapper`中，并序列化一个实体:

```
FilterProvider filters = new SimpleFilterProvider().addFilter("myFilter", theFilter);
MyDto dtoObject = new MyDto();
dtoObject.setIntValue(-1);

ObjectMapper mapper = new ObjectMapper();
String dtoAsString = mapper.writer(filters).writeValueAsString(dtoObject);
```

最后，我们可以检查`intValue`字段确实是**的一部分，而不是被编组的 JSON 输出**的一部分:

```
assertThat(dtoAsString, not(containsString("intValue")));
```

## 3。有条件地跳过对象

现在，让我们讨论如何在基于属性**值**序列化时跳过对象。我们将跳过属性`hidden`为`true`的所有对象:

### 3.1。隐藏类

首先，我们来看看我们的`Hidable`界面:

```
@JsonIgnoreProperties("hidden")
public interface Hidable {
    boolean isHidden();
}
```

我们有两个简单的类来实现这个接口`Person`、`Address`:

`Person`类别:

```
public class Person implements Hidable {
    private String name;
    private Address address;
    private boolean hidden;
}
```

和`Address`类:

```
public class Address implements Hidable {
    private String city;
    private String country;
    private boolean hidden;
}
```

注意:我们使用了`@JsonIgnoreProperties(“hidden”)`来确保`hidden`属性本身不包含在 JSON 中

### 3.2。自定义串行器

接下来——这是我们的自定义序列化程序:

```
public class HidableSerializer extends JsonSerializer<Hidable> {

    private JsonSerializer<Object> defaultSerializer;

    public HidableSerializer(JsonSerializer<Object> serializer) {
        defaultSerializer = serializer;
    }

    @Override
    public void serialize(Hidable value, JsonGenerator jgen, SerializerProvider provider)
      throws IOException, JsonProcessingException {
        if (value.isHidden())
            return;
        defaultSerializer.serialize(value, jgen, provider);
    }

    @Override
    public boolean isEmpty(SerializerProvider provider, Hidable value) {
        return (value == null || value.isHidden());
    }
}
```

请注意:

*   当对象不会被跳过时，我们将序列化委托给默认的注入序列化程序。
*   我们覆盖了方法`isEmpty()`——以确保在隐藏对象是属性的情况下，属性名也被排除在 JSON 之外。

### 3.3。使用`BeanSerializerModifier`

最后，我们将需要使用`BeanSerializerModifier`在我们的自定义`HidableSerializer`中注入默认的序列化程序，如下所示:

```
ObjectMapper mapper = new ObjectMapper();
mapper.setSerializationInclusion(Include.NON_EMPTY);
mapper.registerModule(new SimpleModule() {
    @Override
    public void setupModule(SetupContext context) {
        super.setupModule(context);
        context.addBeanSerializerModifier(new BeanSerializerModifier() {
            @Override
            public JsonSerializer<?> modifySerializer(
              SerializationConfig config, BeanDescription desc, JsonSerializer<?> serializer) {
                if (Hidable.class.isAssignableFrom(desc.getBeanClass())) {
                    return new HidableSerializer((JsonSerializer<Object>) serializer);
                }
                return serializer;
            }
        });
    }
});
```

### 3.4。样本输出

下面是一个简单的序列化示例:

```
Address ad1 = new Address("tokyo", "jp", true);
Address ad2 = new Address("london", "uk", false);
Address ad3 = new Address("ny", "usa", false);
Person p1 = new Person("john", ad1, false);
Person p2 = new Person("tom", ad2, true);
Person p3 = new Person("adam", ad3, false);

System.out.println(mapper.writeValueAsString(Arrays.asList(p1, p2, p3)));
```

输出是:

```
[
    {
        "name":"john"
    },
    {
        "name":"adam",
        "address":{
            "city":"ny",
            "country":"usa"
        }
    }
]
```

### 3.5。测试

最后，这里有几个测试案例:

第一种情况，**无所隐瞒**:

```
@Test
public void whenNotHidden_thenCorrect() throws JsonProcessingException {
    Address ad = new Address("ny", "usa", false);
    Person person = new Person("john", ad, false);
    String result = mapper.writeValueAsString(person);

    assertTrue(result.contains("name"));
    assertTrue(result.contains("john"));
    assertTrue(result.contains("address"));
    assertTrue(result.contains("usa"));
}
```

接下来，**只有地址被隐藏**:

```
@Test
public void whenAddressHidden_thenCorrect() throws JsonProcessingException {
    Address ad = new Address("ny", "usa", true);
    Person person = new Person("john", ad, false);
    String result = mapper.writeValueAsString(person);

    assertTrue(result.contains("name"));
    assertTrue(result.contains("john"));
    assertFalse(result.contains("address"));
    assertFalse(result.contains("usa"));
}
```

现在，**整个人被隐藏起来**:

```
@Test
public void whenAllHidden_thenCorrect() throws JsonProcessingException {
    Address ad = new Address("ny", "usa", false);
    Person person = new Person("john", ad, true);
    String result = mapper.writeValueAsString(person);

    assertTrue(result.length() == 0);
}
```

## 4。结论

这种类型的高级过滤非常强大，在用 Jackson 序列化复杂对象时，允许非常灵活地定制 json。

一个更灵活但也更复杂的替代方案是使用完全定制的序列化程序来控制 JSON 输出——因此，如果这个解决方案不够灵活，可能值得研究一下。

所有这些例子和代码片段的实现**可以在 GitHub** 上找到[——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。](https://web.archive.org/web/20221129010951/https://github.com/eugenp/tutorials/tree/master/jackson-modules/jackson-custom-conversions "Github Project exemplifying how to change the json key of a filed")