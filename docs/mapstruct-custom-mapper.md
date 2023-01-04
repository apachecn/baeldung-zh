# 带有 MapStruct 的自定义映射器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mapstruct-custom-mapper>

## 1.概观

在本文中，我们将学习如何将自定义映射器与 [MapStruct 库](/web/20221204132225/https://www.baeldung.com/mapstruct)一起使用。

**map struct 库用于 Java bean 类型之间的映射**。通过使用带有 MapStruct **，** **的自定义映射器，我们可以自定义默认的映射方法。**

## 2.Maven 依赖性

让我们将 [mapstruct](https://web.archive.org/web/20221204132225/https://search.maven.org/search?q=g:org.mapstruct%20a:mapstruct) 库添加到我们的 Maven `pom.xml`中:

```
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.3.1.Final</version> 
</dependency>
```

要查看项目的`target folder`中自动生成的方法，我们必须将`annotationProcessorPaths` 添加到`maven-compiler-plugin`插件中:

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.5.1</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <annotationProcessorPaths>
            <path>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct</artifactId>
                <version>1.3.1.Final</version>
            </path>
        </annotationProcessorPaths>
    </configuration>
</plugin>
```

## 3.自定义映射器

自定义映射器用于解决特定的转换要求。为了实现这一点，我们必须定义一个方法来进行转换。然后，我们必须将该方法通知 MapStruct。最后，MapStruct 将调用方法进行从源到目标的转换。

例如，假设我们有一个计算用户身体质量指数(身体质量指数)报告的应用程序。为了计算身体质量指数，我们必须收集用户的身体值。要将英制单位转换为公制单位，我们可以使用自定义映射器方法。

通过 MapStruct 使用自定义映射器有两种方式。**我们可以通过在`@Mapping`注释的`qualifiedByName`属性中键入自定义方法来调用它，或者我们可以为它创建一个注释。**

在开始之前，我们必须定义一个 DTO 类来保存帝国价值观:

```
public class UserBodyImperialValuesDTO {
    private int inch;
    private int pound;
    // constructor, getters, and setters
}
```

接下来，让我们定义一个 DTO 类来保存度量值:

```
public class UserBodyValues {
    private double kilogram;
    private double centimeter;
    // constructor, getters, and setters
}
```

### 3.1.使用方法的自定义映射器

要开始使用定制映射器，让我们用`@Mapper`注释创建一个接口:

```
@Mapper 
public interface UserBodyValuesMapper {
    //...
}
```

其次，让我们用我们想要的返回类型和我们需要转换的参数创建我们的自定义方法。我们必须使用带有 value 参数的@ `Named`注释来通知 MapStruct 关于自定义映射器方法的信息:

```
@Mapper
public interface UserBodyValuesMapper {

    @Named("inchToCentimeter")
    public static double inchToCentimeter(int inch) {
        return inch * 2.54;
    }

    //...
}
```

最后，让我们用`@Mapping`注释定义 mapper 接口方法。在这个注释中，我们将告诉 MapStruct 关于源类型、目标类型以及它将使用的方法:

```
@Mapper
public interface UserBodyValuesMapper {
    UserBodyValuesMapper INSTANCE = Mappers.getMapper(UserBodyValuesMapper.class);

    @Mapping(source = "inch", target = "centimeter", qualifiedByName = "inchToCentimeter")
    public UserBodyValues userBodyValuesMapper(UserBodyImperialValuesDTO dto);

    @Named("inchToCentimeter") 
    public static double inchToCentimeter(int inch) { 
        return inch * 2.54; 
    }
}
```

让我们测试一下我们的自定义映射器:

```
UserBodyImperialValuesDTO dto = new UserBodyImperialValuesDTO();
dto.setInch(10);

UserBodyValues obj = UserBodyValuesMapper.INSTANCE.userBodyValuesMapper(dto);

assertNotNull(obj);
assertEquals(25.4, obj.getCentimeter(), 0); 
```

### 3.2.带有注释的自定义映射器

要使用带有注释的自定义映射器，我们必须定义一个注释，而不是`@Named`注释。然后，我们必须通过指定@ `Mapping`注释的`qualifiedByName` 参数`.`来通知 MapStruct 新创建的注释

让我们看看如何定义注释:

```
@Qualifier
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.CLASS)
public @interface PoundToKilogramMapper {
}
```

让我们将`@PoundToKilogramMapper`注释添加到`poundToKilogram`方法中:

```
@PoundToKilogramMapper
public static double poundToKilogram(int pound) {
    return pound * 0.4535;
} 
```

现在，让我们用`@Mapping`注释定义 mapper 接口方法。在映射注释中，我们将告诉 MapStruct 关于源类型、目标类型和它将使用的注释类的信息:

```
@Mapper
public interface UserBodyValuesMapper {
    UserBodyValuesMapper INSTANCE = Mappers.getMapper(UserBodyValuesMapper.class);

    @Mapping(source = "pound", target = "kilogram", qualifiedBy = PoundToKilogramMapper.class)
    public UserBodyValues userBodyValuesMapper(UserBodyImperialValuesDTO dto);

    @PoundToKilogramMapper
    public static double poundToKilogram(int pound) {
        return pound * 0.4535;
    }
}
```

最后，让我们测试我们的定制映射器:

```
UserBodyImperialValuesDTO dto = new UserBodyImperialValuesDTO();
dto.setPound(100);

UserBodyValues obj = UserBodyValuesMapper.INSTANCE.userBodyValuesMapper(dto);

assertNotNull(obj);
assertEquals(45.35, obj.getKilogram(), 0); 
```

## 4.结论

在本文中，我们学习了如何使用 MapStruct 库的自定义映射器。

GitHub 上的[提供了这些例子和测试的实现。](https://web.archive.org/web/20221204132225/https://github.com/eugenp/tutorials/tree/master/mapstruct)