# 弹簧类型转换指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-type-conversions>

## 1。简介

在本文中，我们将看看 Spring 的类型转换。

Spring 为内置类型提供了开箱即用的各种转换器；这意味着在像`String, Integer, Boolean`这样的基本类型和许多其他类型之间进行转换。

除此之外，Spring 还提供了一个用于开发定制转换器的固态类型转换 SPI。

## 2。内置的`Converter` s

我们将从春季现成可用的转换器开始；让我们来看看`String`到`Integer` 的转换:

```java
@Autowired
ConversionService conversionService;

@Test
public void whenConvertStringToIntegerUsingDefaultConverter_thenSuccess() {
    assertThat(
      conversionService.convert("25", Integer.class)).isEqualTo(25);
}
```

这里我们唯一需要做的就是自动连接 Spring 提供的`ConversionService`并调用`convert()`方法。第一个参数是我们要转换的值，第二个参数是我们要转换到的目标类型。

除了这个`String`到`Integer`的例子之外，还有很多不同的组合可供我们选择。

## 3。`Converter`创建自定义

让我们看一个将一个`Employee`的`String`表示转换成一个`Employee`实例的例子。

下面是`Employee`类:

```java
public class Employee {

    private long id;
    private double salary;

    // standard constructors, getters, setters
}
```

`String`将是一个逗号分隔的对，代表`id`和`salary.` ，例如，“1，50000.00”。

**为了创建我们的自定义`Converter`，我们需要实现`Converter<S, T>`接口并实现`convert()`方法:**

```java
public class StringToEmployeeConverter
  implements Converter<String, Employee> {

    @Override
    public Employee convert(String from) {
        String[] data = from.split(",");
        return new Employee(
          Long.parseLong(data[0]), 
          Double.parseDouble(data[1]));
    }
}
```

我们还没完呢。我们还需要通过将`StringToEmployeeConverter`添加到`FormatterRegistry`来告诉 Spring 这个新的转换器。这可以通过实现`WebMvcConfigurer`并覆盖`addFormatters()`方法来实现:

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new StringToEmployeeConverter());
    }
}
```

仅此而已。我们的新`Converter`现在对`ConversionService`可用，我们可以像使用任何其他内置`Converter`一样使用它:

```java
@Test
public void whenConvertStringToEmployee_thenSuccess() {
    Employee employee = conversionService
      .convert("1,50000.00", Employee.class);
    Employee actualEmployee = new Employee(1, 50000.00);

    assertThat(conversionService.convert("1,50000.00", 
      Employee.class))
      .isEqualToComparingFieldByField(actualEmployee);
}
```

### 3.1。隐式转换

除了这些使用`ConversionService`， **Spring 的显式转换之外，它还能够在`Controller`方法**中为所有注册的转换器隐式转换值:

```java
@RestController
public class StringToEmployeeConverterController {

    @GetMapping("/string-to-employee")
    public ResponseEntity<Object> getStringToEmployee(
      @RequestParam("employee") Employee employee) {
        return ResponseEntity.ok(employee);
    }
}
```

这是使用`Converter`的一种更自然的方式。让我们添加一个测试来看看它的实际效果:

```java
@Test
public void getStringToEmployeeTest() throws Exception {
    mockMvc.perform(get("/string-to-employee?employee=1,2000"))
      .andDo(print())
      .andExpect(jsonPath("$.id", is(1)))
      .andExpect(jsonPath("$.salary", is(2000.0)))
}
```

如您所见，测试将打印请求和响应的所有细节。以下是作为响应的一部分返回的 JSON 格式的`Employee`对象:

```java
{"id":1,"salary":2000.0}
```

## 4。创造一个`ConverterFactory`

也可以创建一个`ConverterFactory`来按需创建`Converter`。这对于为`Enums`创建`Converter`特别有帮助。

让我们来看看一个非常简单的枚举:

```java
public enum Modes {
    ALPHA, BETA;
}
```

接下来，让我们创建一个`StringToEnumConverterFactory`，它可以生成将一个`String`转换成任何一个`Enum`的`Converter`:

```java
@Component
public class StringToEnumConverterFactory 
  implements ConverterFactory<String, Enum> {

    private static class StringToEnumConverter<T extends Enum> 
      implements Converter<String, T> {

        private Class<T> enumType;

        public StringToEnumConverter(Class<T> enumType) {
            this.enumType = enumType;
        }

        public T convert(String source) {
            return (T) Enum.valueOf(this.enumType, source.trim());
        }
    }

    @Override
    public <T extends Enum> Converter<String, T> getConverter(
      Class<T> targetType) {
        return new StringToEnumConverter(targetType);
    }
}
```

正如我们所看到的，工厂类在内部使用了一个`Converter`接口的实现。

这里需要注意的一点是，虽然我们将使用我们的`Modes Enum`来演示用法，但是我们在`StringToEnumConverterFactory`中没有提到`Enum`。**我们的工厂类足够通用，可以按需为任何`Enum`类型**生成`Converter` s。

下一步是注册这个工厂类，就像我们在前一个例子中注册我们的`Converter`一样:

```java
@Override
public void addFormatters(FormatterRegistry registry) {
    registry.addConverter(new StringToEmployeeConverter());
    registry.addConverterFactory(new StringToEnumConverterFactory());
}
```

现在`ConversionService`准备将`String` s 转换为`Enum` s:

```java
@Test
public void whenConvertStringToEnum_thenSuccess() {
    assertThat(conversionService.convert("ALPHA", Modes.class))
      .isEqualTo(Modes.ALPHA);
}
```

## 5。创造一个`GenericConverter`

**一个`GenericConverter`为我们提供了更多的灵活性来创建一个`Converter`用于更一般的用途，代价是失去一些类型安全。**

让我们考虑一个将`Integer`、`Double`或`String`转换为`BigDecimal`值的例子。我们不需要为此写三个`Converter`。一个简单的`GenericConverter`就能达到目的。

第一步是告诉 Spring 支持什么类型的转换。我们通过创建一个`ConvertiblePair`的`Set`来做到这一点:

```java
public class GenericBigDecimalConverter 
  implements GenericConverter {

@Override
public Set<ConvertiblePair> getConvertibleTypes () {

    ConvertiblePair[] pairs = new ConvertiblePair[] {
          new ConvertiblePair(Number.class, BigDecimal.class),
          new ConvertiblePair(String.class, BigDecimal.class)};
        return ImmutableSet.copyOf(pairs);
    }
}
```

下一步是在同一个类中覆盖`convert()`方法:

```java
@Override
public Object convert (Object source, TypeDescriptor sourceType, 
  TypeDescriptor targetType) {

    if (sourceType.getType() == BigDecimal.class) {
        return source;
    }

    if(sourceType.getType() == String.class) {
        String number = (String) source;
        return new BigDecimal(number);
    } else {
        Number number = (Number) source;
        BigDecimal converted = new BigDecimal(number.doubleValue());
        return converted.setScale(2, BigDecimal.ROUND_HALF_EVEN);
    }
}
```

`convert()`方法非常简单。然而，`TypeDescriptor`在获取关于源和目标类型的细节方面为我们提供了很大的灵活性。

您可能已经猜到了，下一步是注册这个`Converter`:

```java
@Override
public void addFormatters(FormatterRegistry registry) {
    registry.addConverter(new StringToEmployeeConverter());
    registry.addConverterFactory(new StringToEnumConverterFactory());
    registry.addConverter(new GenericBigDecimalConverter());
}
```

使用这个`Converter`类似于我们已经看到的其他例子:

```java
@Test
public void whenConvertingToBigDecimalUsingGenericConverter_thenSuccess() {
    assertThat(conversionService
      .convert(Integer.valueOf(11), BigDecimal.class))
      .isEqualTo(BigDecimal.valueOf(11.00)
      .setScale(2, BigDecimal.ROUND_HALF_EVEN));
    assertThat(conversionService
      .convert(Double.valueOf(25.23), BigDecimal.class))
      .isEqualByComparingTo(BigDecimal.valueOf(Double.valueOf(25.23)));
    assertThat(conversionService.convert("2.32", BigDecimal.class))
      .isEqualTo(BigDecimal.valueOf(2.32));
}
```

## 6。结论

在本教程中，我们已经通过各种例子看到了如何使用和扩展 Spring 的类型转换系统。

和往常一样，本文的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220525121838/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-basic-customization-2)