# 使用 MapStruct 忽略未映射的属性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mapstruct-ignore-unmapped-properties>

## 1。概述

在 Java 应用程序中，我们可能希望将值从一种类型的 Java bean 复制到另一种类型。为了避免冗长且容易出错的代码，我们可以使用一个 bean 映射器，比如 [MapStruct](/web/20221205122114/https://www.baeldung.com/mapstruct) 。

虽然用相同的字段名映射相同的字段非常简单，但是我们经常会遇到不匹配的 beans。在本教程中，我们将看看 MapStruct 如何处理部分映射。

## 2.绘图

MapStruct 是一个 Java 注释处理器。因此，我们需要做的就是定义映射器接口并声明映射方法。MapStruct 将在编译期间生成该接口的实现。

为了简单起见，让我们从两个具有相同字段名的类开始:

```java
public class CarDTO {
    private int id;
    private String name;
}
```

```java
public class Car {
    private int id;
    private String name;
}
```

接下来，让我们创建一个映射器接口:

```java
@Mapper
public interface CarMapper {
    CarMapper INSTANCE = Mappers.getMapper(CarMapper.class);
    CarDTO carToCarDTO(Car car);
}
```

最后，让我们测试一下我们的映射器:

```java
@Test
public void givenCarEntitytoCar_whenMaps_thenCorrect() {
    Car entity = new Car();
    entity.setId(1);
    entity.setName("Toyota");

    CarDTO carDto = CarMapper.INSTANCE.carToCarDTO(entity);

    assertThat(carDto.getId()).isEqualTo(entity.getId());
    assertThat(carDto.getName()).isEqualTo(entity.getName());
}
```

## 3.未映射的属性

由于 MapStruct 在编译时运行，因此它比动态映射框架更快。如果映射不完整，它还可以**生成错误报告**——也就是说，如果没有映射所有的目标属性:

```java
Warning:(X,X) java: Unmapped target property: "propertyName".
```

虽然这在发生事故时是一个有用的警告，但如果这些字段是故意丢失的，我们可能更喜欢以不同的方式处理事情。

让我们用一个映射两个简单对象的例子来探讨这个问题:

```java
public class DocumentDTO {
    private int id;
    private String title;
    private String text;
    private List<String> comments;
    private String author;
}
```

```java
public class Document {
    private int id;
    private String title;
    private String text;
    private Date modificationTime;
}
```

我们在两个类中都有唯一的字段，它们不应该在映射期间填充。它们是:

*   `DocumentDTO`中的`comments`
*   `DocumentDTO`中的`author`
*   `Document`中的`modificationTime`

如果我们定义一个映射器接口，它将在构建期间导致警告消息:

```java
@Mapper
public interface DocumentMapper {
    DocumentMapper INSTANCE = Mappers.getMapper(DocumentMapper.class);

    DocumentDTO documentToDocumentDTO(Document entity);
    Document documentDTOToDocument(DocumentDTO dto);
}
```

由于我们不想映射这些字段，我们可以用几种方法将它们从映射中排除。

## 4.忽略特定字段

为了跳过特定映射方法中的几个属性，我们可以**使用`@Mapping`注释**中的`ignore`属性:

```java
@Mapper
public interface DocumentMapperMappingIgnore {

    DocumentMapperMappingIgnore INSTANCE =
      Mappers.getMapper(DocumentMapperMappingIgnore.class);

    @Mapping(target = "comments", ignore = true)
    @Mapping(target = "author", ignore = true)
    DocumentDTO documentToDocumentDTO(Document entity);

    @Mapping(target = "modificationTime", ignore = true)
    Document documentDTOToDocument(DocumentDTO dto);
}
```

这里，我们提供了字段名作为`target`，并将`ignore`设置为`true`，以表明它不是映射所必需的。

然而，这种技术在某些情况下并不方便。例如，当使用具有大量字段的大模型时，我们可能会发现它很难使用。

## 5.未映射的目标策略

为了使事情更清楚，代码更可读，我们可以**指定未映射的目标策略**。

为此，当没有用于映射的源字段时，我们使用 MapStruct `unmappedTargetPolicy` 来提供我们想要的行为:

*   任何未映射的目标属性都将导致构建失败——这可以帮助我们避免意外的未映射字段
*   `WARN`:(默认)构建期间的警告消息
*   `IGNORE`:无输出或错误

**为了忽略未映射的属性并且不得到输出警告，**我们应该**将`IGNORE`的值赋给`unmappedTargetPolicy`。**根据目的不同，有几种做法。

### 5.1.对每个`Mapper`设置一个策略

我们可以将**中的`unmappedTargetPolicy`设为** **`@Mapper`** 的注释。因此，它的所有方法都将忽略未映射的属性:

```java
@Mapper(unmappedTargetPolicy = ReportingPolicy.IGNORE)
public interface DocumentMapperUnmappedPolicy {
    // mapper methods
}
```

### 5.2.使用共享的`MapperConfig`

我们可以通过 **`@MapperConfig`** 设置**`unmappedTargetPolicy`来忽略几个映射器中未映射的属性，从而在几个映射器之间共享一个设置。**

首先，我们创建一个带注释的接口:

```java
@MapperConfig(unmappedTargetPolicy = ReportingPolicy.IGNORE)
public interface IgnoreUnmappedMapperConfig {
}
```

然后，我们将该共享配置应用于映射器:

```java
@Mapper(config = IgnoreUnmappedMapperConfig.class)
public interface DocumentMapperWithConfig { 
    // mapper methods 
}
```

我们应该注意到，这是一个简单的例子，显示了`@MapperConfig,`的最小使用量，这似乎并不比在每个映射器上设置策略好多少。当有多个设置需要跨多个映射器标准化时，共享配置变得非常有用。

### 5.3.配置选项

最后，我们可以配置 MapStruct 代码生成器的注释处理器选项。当使用 [Maven](/web/20221205122114/https://www.baeldung.com/maven-compiler-plugin) 时，我们可以使用处理器插件的`compilerArgs` 参数传递处理器选项:

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>${maven-compiler-plugin.version}</version>
            <configuration>
                <source>${maven.compiler.source}</source>
                <target>${maven.compiler.target}</target>
                <annotationProcessorPaths>
                    <path>
                        <groupId>org.mapstruct</groupId>
                        <artifactId>mapstruct-processor</artifactId>
                        <version>${org.mapstruct.version}</version>
                    </path>
                </annotationProcessorPaths>
                <compilerArgs>
                    <compilerArg>
                        -Amapstruct.unmappedTargetPolicy=IGNORE
                    </compilerArg>
                </compilerArgs>
            </configuration>
        </plugin>
    </plugins>
</build>
```

在这个例子中，我们忽略了整个项目中未映射的属性。

## 6.优先顺序

我们已经研究了几种可以帮助我们处理部分映射并完全忽略未映射属性的方法。我们也看到了如何在映射器上独立应用它们，但是我们也可以将它们组合起来。

假设我们有一个包含 beans 和映射器的大型代码库，使用默认的 MapStruct 配置。除了少数情况，我们不希望允许部分映射。我们可以很容易地向 bean 或其映射的对应物添加更多的字段，甚至在没有注意到的情况下获得部分映射。

因此，通过 Maven configuration 添加一个全局设置，使构建在部分映射的情况下失败，这可能是一个好主意。

现在，为了允许我们的一些映射器和**中的未映射属性覆盖全局行为**，我们可以结合这些技术，记住优先级的顺序(从最高到最低):

*   忽略映射器方法级别的特定字段
*   关于映射器的策略
*   共享 MapperConfig
*   全局配置

## 7.结论

在本教程中，我们看了如何配置 MapStruct 来忽略未映射的属性。

首先，我们看看未映射的属性对于映射意味着什么。然后我们看到了如何以几种不同的方式允许部分映射而不出错。

最后，我们学习了如何结合这些技术，记住它们的优先顺序。

和往常一样，本教程的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221205122114/https://github.com/eugenp/tutorials/tree/master/mapstruct)