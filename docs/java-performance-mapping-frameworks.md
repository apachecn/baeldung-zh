# Java 映射框架的性能

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-performance-mapping-frameworks>

## 1.**简介**

创建由多层组成的大型 Java 应用程序需要使用多种模型，如持久性模型、域模型或所谓的 dto。为不同的应用层使用多个模型将要求我们提供一种 beans 之间的映射方式。

手动这样做可能会快速创建许多样板代码，并消耗大量时间。幸运的是，Java 有多种对象映射框架。

在本教程中，我们将比较最流行的 Java 映射框架的性能。

## 2。映射框架

### 2.1.**推土机**

**Dozer 是一个映射框架，使用递归将数据从一个对象复制到另一个对象**。该框架不仅能够在 beans 之间复制属性，还能够在不同类型之间自动转换。

为了使用 Dozer 框架，我们需要将这种依赖性添加到我们的项目中:

```java
<dependency>
    <groupId>com.github.dozermapper</groupId>
    <artifactId>dozer-core</artifactId>
    <version>6.5.2</version>
</dependency>
```

关于推土机框架用法的更多信息可在本文[中找到。](/web/20220924193002/https://www.baeldung.com/dozer)

框架的文档可以在[这里](https://web.archive.org/web/20220924193002/https://github.com/DozerMapper/dozer#what-is-dozer)找到，最新版本可以在[这里](https://web.archive.org/web/20220924193002/https://search.maven.org/search?q=com.github.dozermapper)找到。

### 2.2。奥里卡

Orika 是一个 bean 到 bean 的映射框架，它递归地将数据从一个对象复制到另一个对象。

Orika 的一般工作原理类似于推土机。两者的主要区别在于， **Orika 使用字节码生成**。这允许以最小的开销生成更快的映射器。

为了使用它，我们需要向我们的项目添加这样的依赖:

```java
<dependency>
    <groupId>ma.glasnost.orika</groupId>
    <artifactId>orika-core</artifactId>
    <version>1.5.4</version>
</dependency>
```

关于 Orika 用法的更多详细信息可以在这篇文章中找到。

框架的实际文档可以在[这里](https://web.archive.org/web/20220924193002/https://orika-mapper.github.io/orika-docs/)找到，最新版本可以在[这里](https://web.archive.org/web/20220924193002/https://search.maven.org/search?q=ma.glasnost.orika)找到。

**警告:**自 [Java 16](/web/20220924193002/https://www.baeldung.com/java-16-new-features) 、[非法反射访问](/web/20220924193002/https://www.baeldung.com/java-illegal-reflective-access)被默认拒绝。Orika 的 1.5.4 版本使用这样的反射访问，所以 Orika 目前不能与 Java 16 结合使用。随着 1.6.0 版本的发布，这个问题有望在未来得到解决。

### 2.3. **MapStruct**

**MapStruct 是** **一个自动生成 bean mapper 类的代码生成器。**

MapStruct 还能够在不同的数据类型之间进行转换。关于如何使用它的更多信息可以在这篇文章中找到。

要将 MapStruct 添加到我们的项目中，我们需要包含以下依赖项:

```java
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.5.2.Final</version>
</dependency>
```

框架的文档可以在[这里](https://web.archive.org/web/20220924193002/http://mapstruct.org/)找到，最新版本可以在[这里](https://web.archive.org/web/20220924193002/https://search.maven.org/search?q=org.mapstruct)找到。

### 二点四。**模型目录**

ModelMapper 是一个旨在简化对象映射的框架，它通过基于约定确定对象之间的映射方式。它提供了类型安全和重构安全的 API。

关于框架的更多信息可以在[文档](https://web.archive.org/web/20220924193002/http://modelmapper.org/)中找到。

要在我们的项目中包含模型映射器，我们需要添加以下依赖项:

```java
<dependency>
  <groupId>org.modelmapper</groupId>
  <artifactId>modelmapper</artifactId>
  <version>3.1.0</version>
</dependency>
```

框架的最新版本可以在[这里](https://web.archive.org/web/20220924193002/https://search.maven.org/search?q=org.modelmapper)找到。

### 2.5\. **JMapper**

JMapper 是映射框架，旨在提供 Java Beans 之间的易用、高性能的映射。

该框架旨在使用注释和关系映射来应用 DRY 原则。

该框架允许不同的配置方式:基于注释、基于 XML 或基于 API。

关于这个框架的更多信息可以在它的文档中找到。

要将 JMapper 包含在我们的项目中，我们需要添加它的依赖项:

```java
<dependency>
    <groupId>com.googlecode.jmapper-framework</groupId>
    <artifactId>jmapper-core</artifactId>
    <version>1.6.1.CR2</version>
</dependency>
```

框架的最新版本可以在[这里](https://web.archive.org/web/20220924193002/https://search.maven.org/search?q=com.googlecode.jmapper-framework)找到。

## 3。测试 **型号**

为了能够正确地测试映射，我们需要有源和目标模型。我们已经创建了两个测试模型。

第一个只是一个带有一个`String`字段的简单 POJO，这允许我们在简单的情况下比较框架，并检查如果我们使用更复杂的 beans 是否会有任何变化。

简单的源模型如下所示:

```java
public class SourceCode {
    String code;
    // getter and setter
}
```

它的目的地很相似:

```java
public class DestinationCode {
    String code;
    // getter and setter
}
```

源 bean 的真实示例如下所示:

```java
public class SourceOrder {
    private String orderFinishDate;
    private PaymentType paymentType;
    private Discount discount;
    private DeliveryData deliveryData;
    private User orderingUser;
    private List<Product> orderedProducts;
    private Shop offeringShop;
    private int orderId;
    private OrderStatus status;
    private LocalDate orderDate;
    // standard getters and setters
}
```

目标类如下所示:

```java
public class Order {
    private User orderingUser;
    private List<Product> orderedProducts;
    private OrderStatus orderStatus;
    private LocalDate orderDate;
    private LocalDate orderFinishDate;
    private PaymentType paymentType;
    private Discount discount;
    private int shopId;
    private DeliveryData deliveryData;
    private Shop offeringShop;
    // standard getters and setters
}
```

整个模型结构可以在[这里](https://web.archive.org/web/20220924193002/https://github.com/eugenp/tutorials/tree/master/performance-tests/src/main/java/com/baeldung/performancetests/model/source)找到。

## 4。转换器

为了简化测试设置的设计，我们创建了`Converter`接口:

```java
public interface Converter {
    Order convert(SourceOrder sourceOrder);
    DestinationCode convert(SourceCode sourceCode);
}
```

我们所有的自定义映射器都将实现这个接口。

### 4.1.**T2`OrikaConverter`**

Orika 允许完整的 API 实现，这大大简化了映射器的创建:

```java
public class OrikaConverter implements Converter{
    private MapperFacade mapperFacade;

    public OrikaConverter() {
        MapperFactory mapperFactory = new DefaultMapperFactory
          .Builder().build();

        mapperFactory.classMap(Order.class, SourceOrder.class)
          .field("orderStatus", "status").byDefault().register();
        mapperFacade = mapperFactory.getMapperFacade();
    }

    @Override
    public Order convert(SourceOrder sourceOrder) {
        return mapperFacade.map(sourceOrder, Order.class);
    }

    @Override
    public DestinationCode convert(SourceCode sourceCode) {
        return mapperFacade.map(sourceCode, DestinationCode.class);
    }
}
```

### 4.2.**T2`DozerConverter`**

Dozer 需要 XML 映射文件，包含以下部分:

```java
<mappings 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://dozermapper.github.io/schema/bean-mapping
  https://dozermapper.github.io/schema/bean-mapping.xsd">

    <mapping>
        <class-a>com.baeldung.performancetests.model.source.SourceOrder</class-a>
        <class-b>com.baeldung.performancetests.model.destination.Order</class-b>
        <field>
            <a>status</a>
            <b>orderStatus</b>
        </field>
    </mapping>
    <mapping>
        <class-a>com.baeldung.performancetests.model.source.SourceCode</class-a>
        <class-b>com.baeldung.performancetests.model.destination.DestinationCode</class-b>
    </mapping>
</mappings>
```

定义 XML 映射后，我们可以从代码中使用它:

```java
public class DozerConverter implements Converter {
    private final Mapper mapper;

    public DozerConverter() {
        this.mapper = DozerBeanMapperBuilder.create()
          .withMappingFiles("dozer-mapping.xml")
          .build();       
    }

    @Override
    public Order convert(SourceOrder sourceOrder) {
        return mapper.map(sourceOrder,Order.class);
    }

    @Override
    public DestinationCode convert(SourceCode sourceCode) {
        return mapper.map(sourceCode, DestinationCode.class);
    }
}
```

### 4.3.**T2`MapStructConverter`**

MapStruct 定义非常简单，因为它完全基于代码生成:

```java
@Mapper
public interface MapStructConverter extends Converter {
    MapStructConverter MAPPER = Mappers.getMapper(MapStructConverter.class);

    @Mapping(source = "status", target = "orderStatus")
    @Override
    Order convert(SourceOrder sourceOrder);

    @Override
    DestinationCode convert(SourceCode sourceCode);
}
```

### 4.4.**T2`JMapperConverter`**

需要做更多的工作。实现接口后:

```java
public class JMapperConverter implements Converter {
    JMapper realLifeMapper;
    JMapper simpleMapper;

    public JMapperConverter() {
        JMapperAPI api = new JMapperAPI()
          .add(JMapperAPI.mappedClass(Order.class));
        realLifeMapper = new JMapper(Order.class, SourceOrder.class, api);
        JMapperAPI simpleApi = new JMapperAPI()
          .add(JMapperAPI.mappedClass(DestinationCode.class));
        simpleMapper = new JMapper(
          DestinationCode.class, SourceCode.class, simpleApi);
    }

    @Override
    public Order convert(SourceOrder sourceOrder) {
        return (Order) realLifeMapper.getDestination(sourceOrder);
    }

    @Override
    public DestinationCode convert(SourceCode sourceCode) {
        return (DestinationCode) simpleMapper.getDestination(sourceCode);
    }
}
```

我们还需要给目标类的每个字段添加`@JMap `注释。此外，JMapper 本身不能在枚举类型之间转换，它需要我们创建自定义映射函数:

```java
@JMapConversion(from = "paymentType", to = "paymentType")
public PaymentType conversion(com.baeldung.performancetests.model.source.PaymentType type) {
    PaymentType paymentType = null;
    switch(type) {
        case CARD:
            paymentType = PaymentType.CARD;
            break;

        case CASH:
            paymentType = PaymentType.CASH;
            break;

        case TRANSFER:
            paymentType = PaymentType.TRANSFER;
            break;
    }
    return paymentType;
}
```

### 4.5.**T2`ModelMapperConverter`**

`ModelMapperConverter`要求我们只提供我们想要映射的类:

```java
public class ModelMapperConverter implements Converter {
    private ModelMapper modelMapper;

    public ModelMapperConverter() {
        modelMapper = new ModelMapper();
    }

    @Override
    public Order convert(SourceOrder sourceOrder) {
       return modelMapper.map(sourceOrder, Order.class);
    }

    @Override
    public DestinationCode convert(SourceCode sourceCode) {
        return modelMapper.map(sourceCode, DestinationCode.class);
    }
}
```

## 5.简单模型测试

对于性能测试，我们可以使用 Java Microbenchmark Harness，关于如何使用它的更多信息可以在这篇文章中找到。

我们为每个`Converter`创建了一个单独的基准，指定`BenchmarkMode `到`Mode.All`。

### 5.1.**T2`AverageTime`**

JMH 返回了以下平均运行时间的结果(越少越好) :

| 框架名称 | 平均运行时间(每次操作毫秒) |
| MapStruct | 10 ^(-5) |
| JMapper | 10 ^(-5) |
| 奥里卡 | Zero point zero zero one |
| 模型映射器 | Zero point zero zero two |
| 打瞌睡的人 | Zero point zero zero four |

这个基准测试清楚地表明 MapStruct 和 JMapper 都有最好的平均工作时间。

### 5.2.**T2`Throughput`**

在这种模式下，基准返回每秒的操作数。我们收到了以下结果(**越多越好**):

| 框架名称 | 吞吐量(每毫秒操作数) |
| MapStruct | Fifty-eight thousand one hundred and one |
| JMapper | Fifty-three thousand six hundred and sixty-seven |
| 奥里卡 | One thousand one hundred and ninety-five |
| 模型映射器 | Three hundred and seventy-nine |
| 打瞌睡的人 | Two hundred and thirty |

在吞吐量模式下，MapStruct 是测试过的框架中最快的，JMapper 紧随其后。

### 5.3.**T2`SingleShotTime`**

该模式允许测量单次操作从开始到结束的时间。基准测试给出了以下结果(越少越好):

| 框架名称 | 单次发射时间(每次操作毫秒) |
| JMapper | Zero point zero one six |
| MapStruct | One point nine zero four |
| 打瞌睡的人 | Three point eight six four |
| 奥里卡 | Six point five nine three |
| 模型映射器 | Eight point seven eight eight |

这里，我们看到 JMapper 返回的结果比 MapStruct 好。

### 5.4.**T2`SampleTime`**

该模式允许对每次操作的时间进行采样。三个不同百分点的结果如下所示:

|  | 采样时间(每次操作的毫秒数) |
| 框架名称 | p0.90 | p0.999 | p1.0 |
| JMapper | 10 ^(-4) | Zero point zero zero one | One point five two six |
| MapStruct | 10 ^(-4) | 10 ^(-4) | One point nine four eight |
| 奥里卡 | Zero point zero zero one | Zero point zero one eight | Two point three two seven |
| 模型映射器 | Zero point zero zero two | Zero point zero four four | Three point six zero four |
| 打瞌睡的人 | Zero point zero zero three | Zero point zero eight eight | Five point three eight two |

所有基准测试都表明，根据场景不同，MapStruct 和 JMapper 都是不错的选择`.`

## 6.真实模型测试

对于性能测试，我们可以使用 Java Microbenchmark Harness，关于如何使用它的更多信息可以在这篇文章中找到。

我们为每个`Converter`创建了一个单独的基准，指定`BenchmarkMode `到`Mode.All`。

### 6.1.**T2`AverageTime`**

JMH 返回了以下平均运行时间的结果(越少越好) :

| 框架名称 | 平均运行时间(每次操作毫秒) |
| MapStruct | 10 ^(-4) |
| JMapper | 10 ^(-4) |
| 奥里卡 | Zero point zero zero seven |
| 模型映射器 | Zero point one three seven |
| 打瞌睡的人 | Zero point one four five |

### 6.2.**T2`Throughput`**

在这种模式下，基准返回每秒的操作数。对于每个映射器，我们都收到了以下结果(越多越好) :

| 框架名称 | 吞吐量(每毫秒操作数) |
| JMapper | Three thousand two hundred and five |
| MapStruct | Three thousand four hundred and sixty-seven |
| 奥里卡 | One hundred and twenty-one |
| 模型映射器 | seven |
| 打瞌睡的人 | Six point three four two |

### 6.3.**T2`SingleShotTime`**

该模式允许测量单次操作从开始到结束的时间。基准测试给出了以下结果(越少越好):

| 框架名称 | 单次发射时间(每次操作毫秒) |
| JMapper | Zero point seven two two |
| MapStruct | Two point one one one |
| 打瞌睡的人 | Sixteen point three one one |
| 模型映射器 | Twenty-two point three four two |
| 奥里卡 | Thirty-two point four seven three |

### 6.4.**T2`SampleTime`**

该模式允许对每次操作的时间进行采样。采样结果分为百分位数，我们将给出三个不同百分位数 p0.90、p0.999 **、**和 p1.00 的结果:

|  | 采样时间(每次操作的毫秒数) |
| 框架名称 | p0.90 | p0.999 | p1.0 |
| JMapper | 10 ^(-3) | Zero point zero zero six | three |
| MapStruct | 10 ^(-3) | Zero point zero zero six | eight |
| 奥里卡 | Zero point zero zero seven | Zero point one four three | Fourteen |
| 模型映射器 | Zero point one three eight | Zero point nine nine one | Fifteen |
| 打瞌睡的人 | Zero point one three one | Zero point nine five four | seven |

虽然简单例子和真实例子的确切结果明显不同，但它们或多或少遵循相同的趋势。在这两个例子中，我们看到了 JMapper 和 MapStruct 之间争夺头把交椅的激烈竞争。

### 6.5.结论

根据我们在本节中执行的真实模型测试，我们可以看到，最佳性能显然属于 JMapper，尽管 MapStruct 紧随其后。在相同的测试中，我们看到除了`SingleShotTime`之外，Dozer 一直在我们结果表的底部。

## 7.**总结**

在本文中，我们对五个流行的 Java bean 映射框架进行了性能测试:ModelMapper **、** MapStruct **、** Orika **、** Dozer 和 JMapper。

和往常一样，代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20220924193002/https://github.com/eugenp/tutorials/tree/master/performance-tests)