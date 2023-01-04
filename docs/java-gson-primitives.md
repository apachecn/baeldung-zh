# 在 Gson 中使用原始值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-gson-primitives>

## 1.**概述**

在本教程中，我们将**学习如何用 Gson 序列化和反序列化原始值。** Google 开发了 Gson 库来序列化和反序列化 JSON。此外，我们将了解 Gson 库在处理原语时的一些特殊情况。

另一方面，如果我们需要处理数组、集合、嵌套对象或其他定制，我们有关于用 Gson 序列化[和用 Gson](/web/20220523233733/https://www.baeldung.com/gson-serialization-guide) 反序列化[的附加教程。](/web/20220523233733/https://www.baeldung.com/gson-deserialization-guide)

## 2。Maven 依赖关系

要使用 Gson，我们必须将 [Gson](https://web.archive.org/web/20220523233733/https://search.maven.org/search?q=g:com.google.code.gson%20AND%20a:gson) 依赖项添加到 pom:

```java
<dependency> 
    <groupId>com.google.code.gson</groupId> 
    <artifactId>gson</artifactId> 
    <version>2.8.5</version> 
</dependency>
```

## 3。序列化原始类型

用 Gson 序列化非常简单。我们将使用以下模型作为示例:

```java
public class PrimitiveBundle {
    public byte byteValue;
    public short shortValue;
    public int intValue;
    public long longValue;
    public float floatValue;
    public double doubleValue;
    public boolean booleanValue;
    public char charValue;
}
```

首先，让我们用一些测试值初始化一个实例:

```java
PrimitiveBundle primitiveBundle = new PrimitiveBundle();
primitiveBundle.byteValue = (byte) 0x00001111;
primitiveBundle.shortValue = (short) 3;
primitiveBundle.intValue = 3;
primitiveBundle.longValue = 3;
primitiveBundle.floatValue = 3.5f;
primitiveBundle.doubleValue = 3.5;
primitiveBundle.booleanValue = true;
primitiveBundle.charValue = 'a';
```

接下来，我们可以将其序列化:

```java
Gson gson = new Gson();
String json = gson.toJson(primitiveBundle);
```

最后，我们可以看到序列化的结果:

```java
{  
   "byteValue":17,
   "shortValue":3,
   "intValue":3,
   "longValue":3,
   "floatValue":3.5,
   "doubleValue":3.5,
   "booleanValue":true,
   "charValue":"a"
}
```

我们应该从我们的例子中注意一些细节。首先，字节值不像在模型中那样序列化为一串位。除此之外，没有短整型、整型和长整型之分。此外，float 和 double 之间没有区别。

另一件要注意的事情是字符串代表字符值。

实际上，这最后三件事与 Gson 没有任何关系，但它是 JSON 的定义方式。

### 3.1。序列化特殊浮点值

Java 有常数`Float.POSITIVE_INFINITY`和`NEGATIVE_INFINITY`来表示无穷大。Gson 无法序列化这些特殊值:

```java
public class InfinityValuesExample {
    public float negativeInfinity;
    public float positiveInfinity;
}
```

```java
InfinityValuesExample model = new InfinityValuesExample();
model.negativeInfinity = Float.NEGATIVE_INFINITY;
model.positiveInfinity = Float.POSITIVE_INFINITY;

Gson gson = new Gson();
gson.toJson(model);
```

试图这样做会引发一个`IllegalArgumentException`。

试图序列化`NaN`也会引发一个`IllegalArgumentException`，因为 JSON 规范不允许这个值。

出于同样的原因，试图序列化`Double.`正无穷大、负无穷大或`NaN`也会抛出一个`IllegalArgumentException.`

## 4。反序列化原始类型

现在让我们看看如何反序列化在前面的例子中获得的 JSON 字符串。

反序列化和序列化一样简单:

```java
Gson gson = new Gson();
PrimitiveBundle model = gson.fromJson(json, PrimitiveBundle.class);
```

最后，我们可以验证模型是否包含所需的值:

```java
assertEquals(17, model.byteValue);
assertEquals(3, model.shortValue);
assertEquals(3, model.intValue);
assertEquals(3, model.longValue);
assertEquals(3.5, model.floatValue, 0.0001);
assertEquals(3.5, model.doubleValue, 0.0001);
assertTrue(model.booleanValue);
assertEquals('a', model.charValue); 
```

### 4.1。反序列化字符串值

当一个有效的值被放入一个字符串中时，Gson 会对它进行解析并按预期进行处理:

```java
String json = "{\"byteValue\": \"15\", \"shortValue\": \"15\", "
  + "\"intValue\": \"15\", \"longValue\": \"15\", \"floatValue\": \"15.0\""
  + ", \"doubleValue\": \"15.0\"}";

Gson gson = new Gson();
PrimitiveBundleInitialized model = gson.fromJson(json, PrimitiveBundleInitialized.class); 
```

```java
assertEquals(15, model.byteValue);
assertEquals(15, model.shortValue);
assertEquals(15, model.intValue);
assertEquals(15, model.longValue);
assertEquals(15, model.floatValue, 0.0001);
assertEquals(15, model.doubleValue, 0.0001);
```

值得注意的是，字符串值不能反序列化为布尔类型。

### 4.2。 **反序列化** **空字符串值**

另一方面，让我们尝试用空字符串反序列化下面的 JSON:

```java
String json = "{\"byteValue\": \"\", \"shortValue\": \"\", "
  + "\"intValue\": \"\", \"longValue\": \"\", \"floatValue\": \"\""
  + ", \"doubleValue\": \"\"}";

Gson gson = new Gson();
gson.fromJson(json, PrimitiveBundleInitialized.class);
```

这引发了一个`JsonSyntaxException` ，因为在反序列化原语`.`时不期望空字符串

### 4.3。反序列化空值

尝试反序列化值为`null`的字段将导致 Gson 忽略该字段。例如，使用下面的类:

```java
public class PrimitiveBundleInitialized {
    public byte byteValue = (byte) 1;
    public short shortValue = (short) 1;
    public int intValue = 1;
    public long longValue = 1L;
    public float floatValue = 1.0f;
    public double doubleValue = 1;
}
```

Gson 忽略空字段:

```java
String json = "{\"byteValue\": null, \"shortValue\": null, "
  + "\"intValue\": null, \"longValue\": null, \"floatValue\": null"
  + ", \"doubleValue\": null}";

Gson gson = new Gson();
PrimitiveBundleInitialized model = gson.fromJson(json, PrimitiveBundleInitialized.class);

assertEquals(1, model.byteValue);
assertEquals(1, model.shortValue);
assertEquals(1, model.intValue);
assertEquals(1, model.longValue);
assertEquals(1, model.floatValue, 0.0001);
assertEquals(1, model.doubleValue, 0.0001);
```

### 4.4。反序列化溢出的值

这是 Gson 意外处理的一个很有意思的案例。正在尝试反序列化:

```java
{"value": 300}
```

使用模型:

```java
class ByteExample {
    public byte value;
}
```

因此，该对象的值为 44。它处理得很糟糕，因为在这些情况下可能会引发异常。这将防止无法检测的错误在应用程序中传播。

### 4.5。反序列化浮点数

接下来，让我们尝试将下面的 JSON 反序列化为一个`ByteExample`对象:

```java
{"value": 2.3}
```

Gson 在这里做了正确的事情，并引发了一个子类型为`NumberFormatException`的`JsonSyntaxException`。不管我们使用哪种离散类型(`byte`、`short`、`int `还是`long`，我们都会得到相同的结果。

如果该值以“. 0”结尾，Gson 将按预期反序列化该数字。

### 4.6。反序列化数字布尔值

有时，布尔值被编码为 0 或 1，而不是“真”或“假”。Gson 默认不允许这样。例如，如果我们尝试反序列化:

```java
{"value": 1}
```

进入模型:

```java
class BooleanExample {
    public boolean value;
}
```

Gson 引发一个异常子类型为`IllegalStateException`的`JsonSyntaxException`。**这与数字与**不符时引发的`NumberFormatException`形成对比。如果我们想改变这一点，我们可以使用一个定制的反串行化器。

### 4.7。反序列化 Unicode 字符

值得注意的是，Unicode 字符的反序列化不需要额外的配置。

例如，JSON:

```java
{"value": "\u00AE"}
```

会产生这样的性格。

## 5。结论

正如我们所看到的，Gson 提供了一种处理 JSON 和 Java 原语类型的简单方法。即使在处理简单的基本类型时，也要注意一些意外的行为。

本文的完整实现可以在 GitHub 项目中找到——这是一个基于 Eclipse 的项目，因此应该很容易导入和运行。