# 简单二进制编码指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-sbe>

## 1.介绍

效率和性能是现代数据服务的两个重要方面，尤其是当我们传输大量数据时。当然，使用高性能编码减少消息大小是实现这一目标的关键。

然而，内部编码/解码算法可能会很麻烦和脆弱，这使得它们很难长期维护。

幸运的是，[简单的二进制编码](https://web.archive.org/web/20221022195507/https://real-logic.github.io/simple-binary-encoding/)可以帮助我们以实用的方式实现和维护一个定制的编码/解码系统。

在本教程中，我们将讨论什么是简单二进制编码(SBE ),以及如何使用它的代码样本。

## 2.什么是 SBE？

SBE 是编码/解码消息的二进制表示，以支持低延迟流。它也是金融数据编码标准 [FIX SBE](https://web.archive.org/web/20221022195507/https://github.com/FIXTradingCommunity/fix-simple-binary-encoding) 标准的参考实现。

### 2.1.消息结构

为了保持流语义，**消息必须能够被顺序读取或写入，没有回溯**。这消除了额外的操作，如解引用、处理位置指针、管理附加状态等。–更好地利用硬件支持来保持最高的性能和效率。

让我们来看看 SBE 的信息是如何组织的: [![](img/618277e5402b1615c07f55795a213c57.png)](/web/20221022195507/https://www.baeldung.com/wp-content/uploads/2022/10/sbe-message-structure.png)

*   Header:它包含强制字段，如消息的版本。必要时，它还可以包含更多字段。
*   根字段:消息的静态字段。它们的块大小是预定义的，不能更改。它们也可以被定义为可选的。
*   重复组:这些表示集合类型的演示。组可以包含字段和内部组，以便能够表示更复杂的结构。
*   可变数据字段:这些是我们无法预先确定其大小的字段。字符串和 Blob 数据类型就是两个例子。他们会在信息的最后。

接下来，我们将了解为什么这个消息结构很重要。

### 2.2.SBE 什么时候(不)有用？

SBE 的力量源于它的信息结构。它针对数据的顺序访问进行了优化。因此， **SBE 非常适合固定大小的数据，如数字、位集、枚举和数组**。

SBE 的一个常见用例是金融数据流——大多包含数字和枚举——这是 SBE 专门设计的。

另一方面， **SBE 不太适合可变长度的数据类型，比如字符串和 blob** 。原因是我们很可能不知道未来确切的数据量。因此，这将导致在流式传输时进行额外的计算，以检测消息中数据的边界。毫不奇怪，如果我们谈论毫秒级延迟，这可能会影响我们的业务。

尽管 SBE 仍然支持字符串和 Blob 数据类型，**它们总是被放在消息的末尾，以使可变长度计算的影响最小**。

## 3.建立图书馆

为了使用 SBE 库，让我们将下面的 Maven [依赖项](https://web.archive.org/web/20221022195507/https://search.maven.org/search?q=g:uk.co.real-logic%20AND%20a:sbe-tool)添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>uk.co.real-logic</groupId>
    <artifactId>sbe-all</artifactId>
    <version>1.27.0</version>
</dependency>
```

## 4.生成 Java 存根

显然，在生成 Java 存根之前，我们需要形成消息模式。SBE 提供了通过 XML 定义我们的模式的能力。

接下来，我们将看到如何为我们的消息定义一个模式，该模式传输样本市场交易数据。

### 4.1.创建消息模式

我们的模式将是一个基于 FIX 协议的特殊的 T2 XSD T3 的 T0 XML T1 文件。它将定义我们的消息格式。

因此，让我们创建我们的模式文件:

```java
<?xml version="1.0" encoding="UTF-8"?>
<sbe:messageSchema xmlns:sbe="http://fixprotocol.io/2016/sbe"
  package="com.baeldung.sbe.stub" id="1" version="0" semanticVersion="5.2"
  description="A schema represents stock market data.">
    <types>
        <composite name="messageHeader" 
          description="Message identifiers and length of message root.">
            <type name="blockLength" primitiveType="uint16"/>
            <type name="templateId" primitiveType="uint16"/>
            <type name="schemaId" primitiveType="uint16"/>
            <type name="version" primitiveType="uint16"/>
        </composite>
        <enum name="Market" encodingType="uint8">
            <validValue name="NYSE" description="New York Stock Exchange">0</validValue>
            <validValue name="NASDAQ" 
              description="National Association of Securities Dealers Automated Quotations">1</validValue>
        </enum>
        <type name="Symbol" primitiveType="char" length="4" characterEncoding="ASCII" 
          description="Stock symbol"/>
        <composite name="Decimal">
            <type name="mantissa" primitiveType="uint64" minValue="0"/>
            <type name="exponent" primitiveType="int8"/>
        </composite>
        <enum name="Currency" encodingType="uint8">
            <validValue name="USD" description="US Dollar">0</validValue>
            <validValue name="EUR" description="Euro">1</validValue>
        </enum>
        <composite name="Quote" 
          description="A quote represents the price of a stock in a market">
            <ref name="market" type="Market"/>
            <ref name="symbol" type="Symbol"/>
            <ref name="price" type="Decimal"/>
            <ref name="currency" type="Currency"/>
        </composite>
    </types>
    <sbe:message name="TradeData" id="1" description="Represents a quote and amount of trade">
        <field name="quote" id="1" type="Quote"/>
        <field name="amount" id="2" type="uint16"/>
    </sbe:message>
</sbe:messageSchema>
```

如果我们仔细观察这个模式，我们会注意到它有两个主要部分，`<types>`和`<sbe:message>`。我们将首先开始定义`<types>`。

作为我们的第一种类型，我们创建了`messageHeader`。这是必填字段，也有四个必填字段:

```java
<composite name="messageHeader" description="Message identifiers and length of message root.">
    <type name="blockLength" primitiveType="uint16"/>
    <type name="templateId" primitiveType="uint16"/>
    <type name="schemaId" primitiveType="uint16"/>
    <type name="version" primitiveType="uint16"/>
</composite>
```

*   `blockLength`:表示为消息中的根字段保留的总空间。它不计算重复字段或可变长度字段，如字符串和 blob。
*   `templateId`:消息模板的标识。
*   `schemaId`:消息模式的标识符。模式总是包含一个模板。
*   `version`:定义消息时消息模式的版本。

接下来，我们定义一个枚举，`Market`:

```java
<enum name="Market" encodingType="uint8">
    <validValue name="NYSE" description="New York Stock Exchange">0</validValue>
    <validValue name="NASDAQ" 
      description="National Association of Securities Dealers Automated Quotations">1</validValue>
</enum>
```

我们的目标是保存一些众所周知的交换名称，我们可以将它们硬编码到模式文件中。它们不会经常改变或增加。因此，type < `enum>`非常适合这里。

通过设置`encodingType=”uint8″,`,我们保留了 8 位空间用于在单个消息中存储市场名称。这允许我们支持`2^8 = 256`不同的市场(0 到 255)——无符号 8 位整数的大小。

紧接着，我们定义了另一种类型，`Symbol`。这将是一个 3 或 4 个字符的字符串，用于标识金融工具，如 AAPL(苹果)、MSFT(微软)等。：

```java
<type name="Symbol" primitiveType="char" length="4" characterEncoding="ASCII" description="Instrument symbol"/>
```

正如我们看到的，我们用`characterEncoding=”ASCII”`限制字符–7 位，最多 128 个字符–我们用`length=”4″`设置上限，不允许超过 4 个字符。因此，我们可以尽可能地减小尺寸。

之后，我们需要一个价格数据的复合类型。因此，我们创建了类型`Decimal`:

```java
<composite name="Decimal">
    <type name="mantissa" primitiveType="uint64" minValue="0"/>
    <type name="exponent" primitiveType="int8"/>
</composite>
```

`Decimal`由两种类型组成:

*   `mantissa`:十进制数的有效位数
*   `exponent`:十进制数的刻度

例如，值`mantissa=98765`和`exponent=-3`代表数字 98.765。

接下来，非常类似于`Market`，我们创建另一个`<enum>`来表示`Currency`，其值被映射为`uint8`:

```java
<enum name="Currency" encodingType="uint8">
    <validValue name="USD" description="US Dollar">0</validValue>
    <validValue name="EUR" description="Euro">1</validValue>
</enum>
```

最后，我们通过组合之前创建的其他类型来定义`Quote`:

```java
<composite name="Quote" description="A quote represents the price of an instrument in a market">
    <ref name="market" type="Market"/>
    <ref name="symbol" type="Symbol"/>
    <ref name="price" type="Decimal"/>
    <ref name="currency" type="Currency"/>
</composite>
```

最后，我们完成了类型定义。

然而，我们仍然需要定义一个消息。那么，让我们来定义我们的信息，`TradeData`:

```java
<sbe:message name="TradeData" id="1" description="Represents a quote and amount of trade">
    <field name="quote" id="1" type="Quote"/>
    <field name="amount" id="2" type="uint16"/>
</sbe:message>
```

当然，在类型方面，我们可以从[规范](https://web.archive.org/web/20221022195507/https://github.com/FIXTradingCommunity/fix-simple-binary-encoding/blob/master/v1-0-STANDARD/doc/publication/Simple_Binary_Encoding_V1.0_with_Errata_November_2020.pdf)中找到更多的细节。

在接下来的两节中，我们将讨论如何使用我们的模式来生成 Java 代码，我们最终将使用这些代码来编码/解码我们的消息。

### 4.2.使用`SbeTool`

生成 Java 存根的一种简单方法是使用 SBE jar 文件。这将自动运行实用程序类`SbeTool`:

```java
java -jar -Dsbe.output.dir=target/generated-sources/java 
  <local-maven-directory>/repository/uk/co/real-logic/sbe-all/1.26.0/sbe-all-1.26.0.jar 
  src/main/resources/schema.xml
```

我们应该注意，我们必须用本地 Maven 路径调整占位符`<local-maven-directory>`来运行命令。

成功生成后，我们将在文件夹`target/generated-sources/java`中看到生成的 Java 代码。

### 4.3.将`SbeTool`与 Maven 一起使用

使用`SbeTool`非常容易，但是我们甚至可以通过将它集成到 Maven 中来使它更加实用。

因此，让我们将以下 Maven 插件添加到我们的`pom.xml`中:

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>exec-maven-plugin</artifactId>
            <version>1.6.0</version>
            <executions>
                <execution>
                    <phase>generate-sources</phase>
                    <goals>
                        <goal>java</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <includeProjectDependencies>false</includeProjectDependencies>
                <includePluginDependencies>true</includePluginDependencies>
                <mainClass>uk.co.real_logic.sbe.SbeTool</mainClass>
                <systemProperties>
                    <systemProperty>
                        <key>sbe.output.dir</key>
                        <value>${project.build.directory}/generated-sources/java</value>
                    </systemProperty>
                </systemProperties>
                <arguments>
                    <argument>${project.basedir}/src/main/resources/schema.xml</argument>
                </arguments>
                <workingDirectory>${project.build.directory}/generated-sources/java</workingDirectory>
            </configuration>
            <dependencies>
                <dependency>
                    <groupId>uk.co.real-logic</groupId>
                    <artifactId>sbe-tool</artifactId>
                    <version>1.27.0</version>
                </dependency>
            </dependencies>
        </plugin>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>build-helper-maven-plugin</artifactId>
            <version>3.0.0</version>
            <executions>
                <execution>
                    <id>add-source</id>
                    <phase>generate-sources</phase>
                    <goals>
                        <goal>add-source</goal>
                    </goals>
                    <configuration>
                        <sources>
                            <source>${project.build.directory}/generated-sources/java/</source>
                        </sources>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

结果，**一个典型的 Maven `clean install`命令自动生成我们的 Java 存根**。

此外，我们可以随时查看 [SBE 的 Maven 文档](https://web.archive.org/web/20221022195507/https://github.com/real-logic/simple-binary-encoding/wiki/SBE-Tool-Maven)以获得更多配置选项。

## 5.基本信息

既然我们已经准备好了 Java 存根，让我们看看如何使用它们

首先，我们需要一些数据来进行测试。因此，我们创建了一个类，`MarketData`:

```java
public class MarketData {

    private int amount;
    private double price;
    private Market market;
    private Currency currency;
    private String symbol;

    // Constructor, getters and setters
}
```

我们应该注意到我们的`MarketData`组成了 SBE 为我们生成的`Market`和`Currency`类。

接下来，让我们定义一个`MarketData`对象，稍后在单元测试中使用:

```java
private MarketData marketData;

@BeforeEach
public void setup() {
    marketData = new MarketData(2, 128.99, Market.NYSE, Currency.USD, "IBM");
}
```

既然我们已经准备好了一个`MarketData`，我们将在接下来的章节中看到如何将它写入和读取到我们的`TradeData`中。

### 5.1.写消息

大多数情况下，我们希望将数据写入一个`ByteBuffer`，因此我们创建了一个初始容量为的`ByteBuffer`以及我们生成的编码器、`MessageHeaderEncoder`和`TradeDataEncoder`:

```java
@Test
public void givenMarketData_whenEncode_thenDecodedValuesMatch() {
    // our buffer to write encoded data, initial cap. 128 bytes
    UnsafeBuffer buffer = new UnsafeBuffer(ByteBuffer.allocate(128));
    MessageHeaderEncoder headerEncoder = new MessageHeaderEncoder();
    TradeDataEncoder dataEncoder = new TradeDataEncoder();

    // we'll write the rest of the code here
}
```

在写入数据之前，我们需要将价格数据解析成两部分，尾数和指数:

```java
BigDecimal priceDecimal = BigDecimal.valueOf(marketData.getPrice());
int priceMantissa = priceDecimal.scaleByPowerOfTen(priceDecimal.scale()).intValue();
int priceExponent = priceDecimal.scale() * -1;
```

我们应该注意到我们在这个转换中使用了 [`BigDecimal`](/web/20221022195507/https://www.baeldung.com/java-bigdecimal-biginteger#bigdecimal) 。**在处理货币值时使用`BigDecimal`总是一个好习惯，因为我们不想失去精度**。

最后，让我们编码并编写我们的`TradeData`:

```java
TradeDataEncoder encoder = dataEncoder.wrapAndApplyHeader(buffer, 0, headerEncoder);
encoder.amount(marketData.getAmount());
encoder.quote()
  .market(marketData.getMarket())
  .currency(marketData.getCurrency())
  .symbol(marketData.getSymbol())
  .price()
    .mantissa(priceMantissa)
    .exponent((byte) priceExponent);
```

### 5.2.阅读信息

为了读取消息，我们将使用写入数据的同一个缓冲区实例。然而，我们需要解码器，`MessageHeaderDecoder`和`TradeDataDecoder`，这次:

```java
MessageHeaderDecoder headerDecoder = new MessageHeaderDecoder();
TradeDataDecoder dataDecoder = new TradeDataDecoder();
```

接下来，我们解码我们的`TradeData`:

```java
dataDecoder.wrapAndApplyHeader(buffer, 0, headerDecoder);
```

类似地，我们需要从尾数和指数这两个部分解码价格数据，以便将价格数据转换成一个`double`值。当然，我们会再次使用`BigDecimal`:

```java
double price = BigDecimal.valueOf(dataDecoder.quote().price().mantissa())
  .scaleByPowerOfTen(dataDecoder.quote().price().exponent())
  .doubleValue();
```

最后，让我们确保解码值与原始值匹配:

```java
Assertions.assertEquals(2, dataDecoder.amount());
Assertions.assertEquals("IBM", dataDecoder.quote().symbol());
Assertions.assertEquals(Market.NYSE, dataDecoder.quote().market());
Assertions.assertEquals(Currency.USD, dataDecoder.quote().currency());
Assertions.assertEquals(128.99, price);
```

## 6.结论

在本文中，我们学习了如何设置 SBE，通过 XML 定义消息结构，并使用它来编码/解码 Java 中的消息。

和往常一样，我们可以在 GitHub 上找到所有的代码样本和更多的[。](https://web.archive.org/web/20221022195507/https://github.com/eugenp/tutorials/tree/master/libraries-7)