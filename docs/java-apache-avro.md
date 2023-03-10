# 阿帕奇 Avro 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-apache-avro>

## 1.概观

数据序列化是一种将数据转换为二进制或文本格式的技术。有多种系统可用于此目的。Apache Avro 就是那些数据序列化系统中的一个。

Avro 是一个独立于语言的、基于模式的数据序列化库。它使用模式来执行序列化和反序列化。此外，Avro 使用 JSON 格式来指定数据结构，这使得它更加强大。

在本教程中，我们将探索更多关于 Avro 设置、执行序列化的 Java API 以及 Avro 与其他数据序列化系统的比较。

我们将主要关注模式创建，这是整个系统的基础。

## 2.阿帕奇 Avro

Avro 是一个独立于语言的序列化库。为了做到这一点，Avro 使用了一个作为核心组件之一的模式。它**将模式存储在一个文件中，用于进一步的数据处理**。

Avro 最适合大数据处理。它在 Hadoop 和 Kafka world 中因其更快的处理速度而颇受欢迎。

Avro 创建一个数据文件，将数据和模式保存在元数据部分。最重要的是，它提供了丰富的数据结构，这使得它比其他类似的解决方案更受欢迎。

要使用 Avro 进行序列化，我们需要遵循下面提到的步骤。

## 3.问题陈述

让我们从定义一个名为`AvroHttRequest`的类开始，我们将在我们的例子中使用它。该类包含基本类型和复杂类型属性:

```java
class AvroHttpRequest {

    private long requestTime;
    private ClientIdentifier clientIdentifier;
    private List<String> employeeNames;
    private Active active;
} 
```

这里，`requestTime`是一个原始值。`ClientIdentifier`是另一个代表复杂类型的类。我们还有`employeeName`，它也是一个复杂的类型。`Active`是描述给定员工列表是否有效的枚举。

**我们的目标是使用 Apache Avro 序列化和反序列化`AvroHttRequest`类。**

## 4.Avro 数据类型

在继续之前，让我们讨论一下 Avro 支持的数据类型。

Avro 支持两种类型的数据:

*   原始类型:Avro 支持所有的原始类型。我们使用基本类型名来定义给定字段的类型。例如，保存`String`的值应该在模式中声明为{"type": "string"}

*   复杂类型:Avro 支持六种复杂类型:记录、枚举、数组、映射、联合和固定

例如，在我们的问题语句中，`ClientIdentifier`是一个记录。

在这种情况下，`ClientIdentifier`的模式应该如下所示:

```java
{
   "type":"record",
   "name":"ClientIdentifier",
   "namespace":"com.baeldung.avro",
   "fields":[
      {
         "name":"hostName",
         "type":"string"
      },
      {
         "name":"ipAddress",
         "type":"string"
      }
   ]
}
```

## 5.使用 Avro

首先，让我们将所需的 Maven 依赖项添加到我们的`pom.xml`文件中。

我们应该包括以下依赖项:

*   Apache Avro–核心组件
*   编译器–针对 Avro IDL 和 Avro 特定 Java APIT 的 Apache Avro 编译器
*   工具–包括 Apache Avro 命令行工具和实用程序
*   用于 Maven 项目的 Apache Avro Maven 插件

对于本教程，我们使用的是版本 1.8.2。

然而，总是建议在 [Maven Central](https://web.archive.org/web/20220625233241/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22avro%22%20AND%20g%3A%22org.apache.avro%22) 上找到最新版本:

```java
<dependency>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro-compiler</artifactId>
    <version>1.8.2</version>
</dependency>
<dependency>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro-maven-plugin</artifactId>
    <version>1.8.2</version>
</dependency>
```

添加 maven 依赖项后，接下来的步骤将是:

*   模式创建
*   读取我们程序中的模式
*   使用 Avro 序列化我们的数据
*   最后，反序列化数据

## 6.模式创建

Avro 使用 JSON 格式描述它的模式。对于给定的 Avro 模式，主要有四个属性:

*   `Type-`描述模式的类型，无论是复杂类型还是原始值
*   `Namespace-`描述给定模式所属的名称空间
*   `Name`–模式的名称
*   `Fields-`它告诉我们与给定模式相关的字段。**字段可以是原始类型，也可以是复杂类型**。

创建模式的一种方法是编写 JSON 表示，正如我们在前面几节中看到的。

我们还可以使用`SchemaBuilder`创建一个模式，这无疑是一个更好、更有效的创建方式。

### 6.1.`SchemaBuilder`效用

类`org.apache.avro.SchemaBuilder`对于创建模式很有用。

首先，让我们为`ClientIdentifier:`创建模式

```java
Schema clientIdentifier = SchemaBuilder.record("ClientIdentifier")
  .namespace("com.baeldung.avro")
  .fields().requiredString("hostName").requiredString("ipAddress")
  .endRecord();
```

现在，让我们用它来创建一个`avroHttpRequest`模式:

```java
Schema avroHttpRequest = SchemaBuilder.record("AvroHttpRequest")
  .namespace("com.baeldung.avro")
  .fields().requiredLong("requestTime")
  .name("clientIdentifier")
    .type(clientIdentifier)
    .noDefault()
  .name("employeeNames")
    .type()
    .array()
    .items()
    .stringType()
    .arrayDefault(null)
  .name("active")
    .type()
    .enumeration("Active")
    .symbols("YES","NO")
    .noDefault()
  .endRecord();
```

这里需要注意的是，我们已经将`clientIdentifier`指定为`clientIdentifier`字段的类型。在本例中，`clientIdentifier`用来定义的类型与我们之前创建的模式相同。

稍后我们可以应用`toString`方法得到`Schema`的`JSON`结构。

**模式文件使用。avsc 扩展**。让我们将生成的模式保存到`“src/main/resources/avroHttpRequest-schema.avsc”`文件中。

## 7.读取模式

读取模式或多或少是为了给定的模式创建 Avro 类。一旦创建了 Avro 类，我们就可以用它们来序列化和反序列化对象。

创建 Avro 类有两种方法:

*   以编程方式生成 Avro 类:可以使用`SchemaCompiler`生成类。有几个 API 可以用来生成 Java 类。我们可以在 GitHub 上找到生成类的代码。
*   **使用 Maven 生成类**

我们有一个 maven 插件可以很好地完成这项工作。我们需要包含插件并运行`mvn clean install`。

让我们将插件添加到我们的`pom.xml`文件中:

```java
<plugin>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro-maven-plugin</artifactId>
    <version>${avro.version}</version>
        <executions>
            <execution>
                <id>schemas</id>
                <phase>generate-sources</phase>
                <goals>
                    <goal>schema</goal>
                    <goal>protocol</goal>
                    <goal>idl-protocol</goal>
                </goals>
                <configuration>
                    <sourceDirectory>${project.basedir}/src/main/resources/</sourceDirectory>
                    <outputDirectory>${project.basedir}/src/main/java/</outputDirectory>
                </configuration>
            </execution>
        </executions>
</plugin> 
```

## 8.用 Avro 进行序列化和反序列化

当我们完成了模式生成之后，让我们继续探索序列化部分。

Avro 支持两种数据序列化格式:JSON 格式和二进制格式。

首先，我们将关注 JSON 格式，然后我们将讨论二进制格式。

在继续下一步之前，我们应该了解几个关键接口。我们可以使用下面的接口和类进行序列化:

我们应该用它在给定的模式上写数据。在我们的例子中，我们将使用`SpecificDatumWriter`实现，然而，`DatumWriter`也有其他实现。其他实现有`GenericDatumWriter, Json.Writer, ProtobufDatumWriter, ReflectDatumWriter, ThriftDatumWriter.`

`Encoder`:如前所述，编码器用于定义格式。`EncoderFactory`提供两种类型的编码器，二进制编码器和 JSON 编码器。

`DatumReader<D>:`用于反序列化的单一接口。同样，它有多个实现，但是我们将在我们的例子中使用`SpecificDatumReader`。其他实现有- `GenericDatumReader, Json.ObjectReader, Json.Reader, ProtobufDatumReader, ReflectDatumReader, ThriftDatumReader.`

`Decoder:`解串数据时使用解码器。`Decoderfactory`提供两种解码器:二进制解码器和 JSON 解码器。

接下来，我们来看看在 Avro 中序列化和反序列化是如何发生的。

### 8.1.序列化

我们将以`AvroHttpRequest`类为例，尝试使用 Avro 对其进行序列化。

首先我们用 JSON 格式序列化一下:

```java
public byte[] serealizeAvroHttpRequestJSON(
  AvroHttpRequest request) {

    DatumWriter<AvroHttpRequest> writer = new SpecificDatumWriter<>(
      AvroHttpRequest.class);
    byte[] data = new byte[0];
    ByteArrayOutputStream stream = new ByteArrayOutputStream();
    Encoder jsonEncoder = null;
    try {
        jsonEncoder = EncoderFactory.get().jsonEncoder(
          AvroHttpRequest.getClassSchema(), stream);
        writer.write(request, jsonEncoder);
        jsonEncoder.flush();
        data = stream.toByteArray();
    } catch (IOException e) {
        logger.error("Serialization error:" + e.getMessage());
    }
    return data;
} 
```

让我们来看看这个方法的一个测试案例:

```java
@Test
public void whenSerialized_UsingJSONEncoder_ObjectGetsSerialized(){
    byte[] data = serealizer.serealizeAvroHttpRequestJSON(request);
    assertTrue(Objects.nonNull(data));
    assertTrue(data.length > 0);
}
```

这里我们使用了`jsonEncoder`方法，并将模式传递给它。

如果我们想使用二进制编码器，我们需要用`binaryEncoder():`替换`jsonEncoder()`方法

```java
Encoder jsonEncoder = EncoderFactory.get().binaryEncoder(stream,null);
```

### 8.2.反序列化

为此，我们将使用上述的`DatumReader`和`Decoder`接口。

正如我们使用`EncoderFactory`来获得一个`Encoder,`一样，我们将使用`DecoderFactory`来获得一个`Decoder`对象。

让我们使用 JSON 格式反序列化数据:

```java
public AvroHttpRequest deSerealizeAvroHttpRequestJSON(byte[] data) {
    DatumReader<AvroHttpRequest> reader
     = new SpecificDatumReader<>(AvroHttpRequest.class);
    Decoder decoder = null;
    try {
        decoder = DecoderFactory.get().jsonDecoder(
          AvroHttpRequest.getClassSchema(), new String(data));
        return reader.read(null, decoder);
    } catch (IOException e) {
        logger.error("Deserialization error:" + e.getMessage());
    }
} 
```

让我们看看测试案例:

```java
@Test
public void whenDeserializeUsingJSONDecoder_thenActualAndExpectedObjectsAreEqual(){
    byte[] data = serealizer.serealizeAvroHttpRequestJSON(request);
    AvroHttpRequest actualRequest = deSerealizer
      .deSerealizeAvroHttpRequestJSON(data);
    assertEquals(actualRequest,request);
    assertTrue(actualRequest.getRequestTime()
      .equals(request.getRequestTime()));
}
```

同样，我们可以使用二进制解码器:

```java
Decoder decoder = DecoderFactory.get().binaryDecoder(data, null);
```

## 9.结论

Apache Avro 在处理大数据时特别有用。它提供了二进制和 JSON 格式的数据序列化，可以根据用例来使用。

Avro 序列化过程更快，也更节省空间。Avro 不保存每个字段的字段类型信息；相反，它在模式中创建元数据。

最后但并非最不重要的是，Avro 与广泛的编程语言有很好的绑定，这给了它一个优势。

和往常一样，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220625233241/https://github.com/eugenp/tutorials/tree/master/apache-libraries)