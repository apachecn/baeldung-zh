# 地图结构快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mapstruct>

## 1。概述

在本教程中，我们将探索使用 [MapStruct](https://web.archive.org/web/20220706110125/http://www.mapstruct.org/) ，简单地说，它是一个 Java Bean 映射器。

这个 API 包含在两个 Java Beans 之间自动映射的函数。有了 MapStruct，我们只需要创建接口，库就会在编译时自动创建一个具体的实现。

## 延伸阅读:

## [带有 MapStruct 的自定义映射器](/web/20220706110125/https://www.baeldung.com/mapstruct-custom-mapper)

Learn how to use custom mapper with the MapStruct library[Read more](/web/20220706110125/https://www.baeldung.com/mapstruct-custom-mapper) →

## [用 MapStruct 忽略未映射的属性](/web/20220706110125/https://www.baeldung.com/mapstruct-ignore-unmapped-properties)

MapStruct allows us to copy between Java beans. There are a few ways we can configure it to handle missing fields.[Read more](/web/20220706110125/https://www.baeldung.com/mapstruct-ignore-unmapped-properties) →

## [通过 MapStruct 使用多个源对象](/web/20220706110125/https://www.baeldung.com/mapstruct-multiple-source-objects)

Learn how to use multiple source objects with MapStruct.[Read more](/web/20220706110125/https://www.baeldung.com/mapstruct-multiple-source-objects) →

## 2。MapStruct 和 Transfer 对象模式

对于大多数应用程序，您会注意到许多将 POJO 转换成其他 POJO 的样板代码。

例如，一种常见的转换发生在持久性支持的实体和发送到客户端的 dto 之间。

这就是 MapStruct 解决的问题:手动创建 bean 映射器非常耗时。但是库**可以自动生成 bean mapper 类。**

## 3。肚子

让我们将下面的依赖关系添加到我们的 Maven `pom.xml`中:

```
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.4.2.Final</version> 
</dependency>
```

最新稳定版的 [MapStruct](https://web.archive.org/web/20220706110125/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.mapstruct%22%20AND%20a%3A%22mapstruct%22) 和它的[处理器](https://web.archive.org/web/20220706110125/https://search.maven.org/classic/#search|ga|1|g%3A%22org.mapstruct%22%20AND%20a%3A%22mapstruct-processor%22)都可以从 Maven Central Repository 获得。

让我们也将`annotationProcessorPaths`部分添加到`maven-compiler-plugin`插件的配置部分。

`mapstruct-processor`用于在构建期间生成映射器实现:

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
                <artifactId>mapstruct-processor</artifactId>
                <version>1.4.2.Final</version>
            </path>
        </annotationProcessorPaths>
    </configuration>
</plugin>
```

## 4。基本映射

### 4.1。创建 POJO

让我们首先创建一个简单的 Java POJO:

```
public class SimpleSource {
    private String name;
    private String description;
    // getters and setters
}

public class SimpleDestination {
    private String name;
    private String description;
    // getters and setters
}
```

### 4.2。映射器接口

```
@Mapper
public interface SimpleSourceDestinationMapper {
    SimpleDestination sourceToDestination(SimpleSource source);
    SimpleSource destinationToSource(SimpleDestination destination);
}
```

注意，我们没有为我们的`SimpleSourceDestinationMapper —`创建实现类，因为 MapStruct 为我们创建了它。

### 4.3。新的映射器

我们可以通过执行`mvn clean install`来触发 MapStruct 处理。

这将在`/target/generated-sources/annotations/`下生成实现类。

下面是 MapStruct 为我们自动创建的类:

```
public class SimpleSourceDestinationMapperImpl
  implements SimpleSourceDestinationMapper {
    @Override
    public SimpleDestination sourceToDestination(SimpleSource source) {
        if ( source == null ) {
            return null;
        }
        SimpleDestination simpleDestination = new SimpleDestination();
        simpleDestination.setName( source.getName() );
        simpleDestination.setDescription( source.getDescription() );
        return simpleDestination;
    }
    @Override
    public SimpleSource destinationToSource(SimpleDestination destination){
        if ( destination == null ) {
            return null;
        }
        SimpleSource simpleSource = new SimpleSource();
        simpleSource.setName( destination.getName() );
        simpleSource.setDescription( destination.getDescription() );
        return simpleSource;
    }
}
```

### 4.4。测试用例

最后，生成所有内容后，让我们编写一个测试用例，显示`SimpleSource`中的值与`SimpleDestination`中的值匹配:

```
public class SimpleSourceDestinationMapperIntegrationTest {
    private SimpleSourceDestinationMapper mapper
      = Mappers.getMapper(SimpleSourceDestinationMapper.class);
    @Test
    public void givenSourceToDestination_whenMaps_thenCorrect() {
        SimpleSource simpleSource = new SimpleSource();
        simpleSource.setName("SourceName");
        simpleSource.setDescription("SourceDescription");
        SimpleDestination destination = mapper.sourceToDestination(simpleSource);

        assertEquals(simpleSource.getName(), destination.getName());
        assertEquals(simpleSource.getDescription(), 
          destination.getDescription());
    }
    @Test
    public void givenDestinationToSource_whenMaps_thenCorrect() {
        SimpleDestination destination = new SimpleDestination();
        destination.setName("DestinationName");
        destination.setDescription("DestinationDescription");
        SimpleSource source = mapper.destinationToSource(destination);
        assertEquals(destination.getName(), source.getName());
        assertEquals(destination.getDescription(),
          source.getDescription());
    }
}
```

## 5。依赖注入映射

接下来，让我们通过调用`Mappers.getMapper(YourClass.class)`来获得 MapStruct 中映射器的实例。

当然，这是一种非常手工的获取实例的方式。然而，一个更好的选择是在我们需要的地方直接注入映射器(如果我们的项目使用任何依赖注入解决方案)。

**幸运的是，MapStruct 对 Spring 和 CDI 都有坚实的支持** ( `Contexts and Dependency Injection`)。

为了在我们的映射器中使用 Spring IoC，我们需要用值`spring`将`componentModel` 属性添加到`@Mapper`中，对于 CDI，它将是`cdi`。

### 5.1。修改映射器

将以下代码添加到`SimpleSourceDestinationMapper`:

```
@Mapper(componentModel = "spring")
public interface SimpleSourceDestinationMapper
```

### 5.2.将 Spring 组件注入到映射器中

有时，我们需要在映射逻辑中使用其他 Spring 组件。在这种情况下，**我们必须使用一个抽象类来代替接口**:

```
@Mapper(componentModel = "spring")
public abstract class SimpleDestinationMapperUsingInjectedService
```

然后，我们可以使用众所周知的`@Autowired`注释轻松地注入所需的组件，并在我们的代码中使用它:

```
@Mapper(componentModel = "spring")
public abstract class SimpleDestinationMapperUsingInjectedService {

    @Autowired
    protected SimpleService simpleService;

    @Mapping(target = "name", expression = "java(simpleService.enrichName(source.getName()))")
    public abstract SimpleDestination sourceToDestination(SimpleSource source);
}
```

**我们一定要记住不要把注入的 bean 私有！**这是因为 MapStruct 要访问生成的实现类中的对象。

## 6。映射具有不同字段名的字段

在前面的例子中，MapStruct 能够自动映射我们的 beans，因为它们具有相同的字段名。那么，如果我们将要映射的 bean 有不同的字段名呢？

在这个例子中，我们将创建一个名为`Employee`和`EmployeeDTO`的新 bean。

### 6.1。新 POJO

```
public class EmployeeDTO {
    private int employeeId;
    private String employeeName;
    // getters and setters
}
```

```
public class Employee {
    private int id;
    private String name;
    // getters and setters
}
```

### 6.2。映射器接口

当映射不同的字段名时，我们需要将其源字段配置为其目标字段，为此，我们需要添加`@Mappings`注释。该注释接受一组`@Mapping`注释，我们将使用它来添加目标和源属性。

在 MapStruct 中，我们还可以使用点符号来定义 bean 的成员:

```
@Mapper
public interface EmployeeMapper {
    @Mappings({
      @Mapping(target="employeeId", source="entity.id"),
      @Mapping(target="employeeName", source="entity.name")
    })
    EmployeeDTO employeeToEmployeeDTO(Employee entity);
    @Mappings({
      @Mapping(target="id", source="dto.employeeId"),
      @Mapping(target="name", source="dto.employeeName")
    })
    Employee employeeDTOtoEmployee(EmployeeDTO dto);
}
```

### 6.3。测试用例

同样，我们需要测试源和目标对象值是否匹配:

```
@Test
public void givenEmployeeDTOwithDiffNametoEmployee_whenMaps_thenCorrect() {
    EmployeeDTO dto = new EmployeeDTO();
    dto.setEmployeeId(1);
    dto.setEmployeeName("John");

    Employee entity = mapper.employeeDTOtoEmployee(dto);

    assertEquals(dto.getEmployeeId(), entity.getId());
    assertEquals(dto.getEmployeeName(), entity.getName());
}
```

更多测试用例可以在 [GitHub 项目](https://web.archive.org/web/20220706110125/https://github.com/eugenp/tutorials/tree/master/mapstruct)中找到。

## 7。映射带有子 bean 的 bean

接下来，我们将展示如何将一个 bean 映射到对其他 bean 的引用。

### 7.1。修改 POJO

让我们给`Employee`对象添加一个新的 bean 引用:

```
public class EmployeeDTO {
    private int employeeId;
    private String employeeName;
    private DivisionDTO division;
    // getters and setters omitted
}
```

```
public class Employee {
    private int id;
    private String name;
    private Division division;
    // getters and setters omitted
}
```

```
public class Division {
    private int id;
    private String name;
    // default constructor, getters and setters omitted
}
```

### 7.2。修改映射器

这里我们需要添加一个方法来将`Division`转换为`DivisionDTO`，反之亦然；如果 MapStruct 检测到需要转换对象类型，并且要转换的方法存在于同一个类中，它将自动使用该方法。

让我们将它添加到映射器中:

```
DivisionDTO divisionToDivisionDTO(Division entity);

Division divisionDTOtoDivision(DivisionDTO dto);
```

### 7.3。修改测试用例

让我们修改并添加一些测试用例到现有的用例中:

```
@Test
public void givenEmpDTONestedMappingToEmp_whenMaps_thenCorrect() {
    EmployeeDTO dto = new EmployeeDTO();
    dto.setDivision(new DivisionDTO(1, "Division1"));
    Employee entity = mapper.employeeDTOtoEmployee(dto);
    assertEquals(dto.getDivision().getId(), 
      entity.getDivision().getId());
    assertEquals(dto.getDivision().getName(), 
      entity.getDivision().getName());
}
```

## 8。类型转换映射

MapStruct 还提供了一些现成的隐式类型转换，对于我们的示例，我们将尝试将一个字符串 date 转换为一个实际的`Date`对象。

有关隐式类型转换的更多细节，请查看 [MapStruct 参考指南](https://web.archive.org/web/20220706110125/https://mapstruct.org/documentation/stable/reference/html/)。

### 8.1。修改豆子

我们为员工添加了一个开始日期:

```
public class Employee {
    // other fields
    private Date startDt;
    // getters and setters
}
```

```
public class EmployeeDTO {
    // other fields
    private String employeeStartDt;
    // getters and setters
}
```

### 8.2。修改映射器

我们修改映射器，并为我们的开始日期提供`dateFormat`:

```
@Mappings({
  @Mapping(target="employeeId", source = "entity.id"),
  @Mapping(target="employeeName", source = "entity.name"),
  @Mapping(target="employeeStartDt", source = "entity.startDt",
           dateFormat = "dd-MM-yyyy HH:mm:ss")})
EmployeeDTO employeeToEmployeeDTO(Employee entity);
@Mappings({
  @Mapping(target="id", source="dto.employeeId"),
  @Mapping(target="name", source="dto.employeeName"),
  @Mapping(target="startDt", source="dto.employeeStartDt",
           dateFormat="dd-MM-yyyy HH:mm:ss")})
Employee employeeDTOtoEmployee(EmployeeDTO dto);
```

### 8.3。修改测试用例

让我们再添加几个测试用例来验证转换是否正确:

```
private static final String DATE_FORMAT = "dd-MM-yyyy HH:mm:ss";
@Test
public void givenEmpStartDtMappingToEmpDTO_whenMaps_thenCorrect() throws ParseException {
    Employee entity = new Employee();
    entity.setStartDt(new Date());
    EmployeeDTO dto = mapper.employeeToEmployeeDTO(entity);
    SimpleDateFormat format = new SimpleDateFormat(DATE_FORMAT);

    assertEquals(format.parse(dto.getEmployeeStartDt()).toString(),
      entity.getStartDt().toString());
}
@Test
public void givenEmpDTOStartDtMappingToEmp_whenMaps_thenCorrect() throws ParseException {
    EmployeeDTO dto = new EmployeeDTO();
    dto.setEmployeeStartDt("01-04-2016 01:00:00");
    Employee entity = mapper.employeeDTOtoEmployee(dto);
    SimpleDateFormat format = new SimpleDateFormat(DATE_FORMAT);

    assertEquals(format.parse(dto.getEmployeeStartDt()).toString(),
      entity.getStartDt().toString());
}
```

## 9。用抽象类映射

有时，我们可能希望以超出@Mapping 功能的方式定制我们的映射器。

例如，除了类型转换之外，我们可能希望以某种方式转换值，如下例所示。

在这种情况下，我们可以创建一个抽象类并实现我们想要定制的方法，并将那些应该由 MapStruct 生成的方法留在抽象类中。

### 9.1。基本型号

在这个例子中，我们将使用下面的类:

```
public class Transaction {
    private Long id;
    private String uuid = UUID.randomUUID().toString();
    private BigDecimal total;

    //standard getters
}
```

和一个匹配的 DTO:

```
public class TransactionDTO {

    private String uuid;
    private Long totalInCents;

    // standard getters and setters
}
```

这里棘手的部分是将`BigDecimal` `total` 美元转换成`Long totalInCents`。

### 9.2。定义映射器

我们可以通过将`Mapper` 创建为抽象类来实现这一点:

```
@Mapper
abstract class TransactionMapper {

    public TransactionDTO toTransactionDTO(Transaction transaction) {
        TransactionDTO transactionDTO = new TransactionDTO();
        transactionDTO.setUuid(transaction.getUuid());
        transactionDTO.setTotalInCents(transaction.getTotal()
          .multiply(new BigDecimal("100")).longValue());
        return transactionDTO;
    }

    public abstract List<TransactionDTO> toTransactionDTO(
      Collection<Transaction> transactions);
}
```

这里，我们为单个对象转换实现了完全定制的映射方法。

另一方面，我们留下了方法，这意味着将`Collection` 映射到`List` 抽象，所以`MapStruct` 将为我们实现它。

### 9.3。生成的结果

因为我们已经实现了将单个`Transaction` 映射到一个`TransactionDTO`的方法，我们期望`MapStruct` 在第二个方法中使用它。

将生成以下内容:

```
@Generated
class TransactionMapperImpl extends TransactionMapper {

    @Override
    public List<TransactionDTO> toTransactionDTO(Collection<Transaction> transactions) {
        if ( transactions == null ) {
            return null;
        }

        List<TransactionDTO> list = new ArrayList<>();
        for ( Transaction transaction : transactions ) {
            list.add( toTransactionDTO( transaction ) );
        }

        return list;
    }
}
```

正如我们在第 12 行看到的，`MapStruct` 在生成的方法中使用我们的实现。

## 10.映射前和映射后注释

这里有另一种通过使用`@BeforeMapping`和`@AfterMapping`注释来定制`@Mapping`功能的方法。**注释用于标记在映射逻辑前后调用的方法。**

在我们可能希望将这个**行为应用于所有映射的超类型的场景中，它们非常有用。**

让我们看一个将`Car` `ElectricCar`和`BioDieselCar`的子类型映射到`CarDTO`的例子。

在映射时，我们希望将类型的概念映射到 DTO 中的`FuelType` 枚举字段。映射完成后，我们想把 DTO 的名字改成大写。

### 10.1.基本模型

我们将使用以下类:

```
public class Car {
    private int id;
    private String name;
}
```

`Car`的子类型:

```
public class BioDieselCar extends Car {
}
```

```
public class ElectricCar extends Car {
}
```

具有枚举字段类型`FuelType`的 `CarDTO`:

```
public class CarDTO {
    private int id;
    private String name;
    private FuelType fuelType;
}
```

```
public enum FuelType {
    ELECTRIC, BIO_DIESEL
}
```

### 10.2.定义映射器

现在让我们继续编写将`Car`映射到`CarDTO`的抽象映射器类:

```
@Mapper
public abstract class CarsMapper {
    @BeforeMapping
    protected void enrichDTOWithFuelType(Car car, @MappingTarget CarDTO carDto) {
        if (car instanceof ElectricCar) {
            carDto.setFuelType(FuelType.ELECTRIC);
        }
        if (car instanceof BioDieselCar) { 
            carDto.setFuelType(FuelType.BIO_DIESEL);
        }
    }

    @AfterMapping
    protected void convertNameToUpperCase(@MappingTarget CarDTO carDto) {
        carDto.setName(carDto.getName().toUpperCase());
    }

    public abstract CarDTO toCarDto(Car car);
}
```

**`@MappingTarget`** 是一个参数注释，在`@BeforeMapping` 和`@AfterMapping` 注释方法的情况下，**在映射逻辑执行之前填充目标映射 DTO。**

### 10.3.结果

上面定义的 **`CarsMapper` 生成******实现**:**

```
@Generated
public class CarsMapperImpl extends CarsMapper {

    @Override
    public CarDTO toCarDto(Car car) {
        if (car == null) {
            return null;
        }

        CarDTO carDTO = new CarDTO();

        enrichDTOWithFuelType(car, carDTO);

        carDTO.setId(car.getId());
        carDTO.setName(car.getName());

        convertNameToUpperCase(carDTO);

        return carDTO;
    }
}
```

注意**带注释的方法调用在实现中是如何包围映射逻辑**的。

## 11。支持龙目语

在 MapStruct 的最新版本中，宣布了 Lombok 支持。因此，我们可以很容易地使用 Lombok 映射源实体和目的实体。

为了启用 Lombok 支持，我们需要在注释处理器路径中添加[依赖关系](https://web.archive.org/web/20220706110125/https://search.maven.org/search?q=a:lombok)。从 Lombok 1 . 18 . 16 版本开始，我们还必须添加对`lombok-mapstruct-binding`的依赖。现在我们在 Maven 编译器插件中有了`mapstruct-processor`和 Lombok:

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
                <artifactId>mapstruct-processor</artifactId>
                <version>1.4.2.Final</version>
            </path>
            <path>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
	        <version>1.18.4</version>
            </path>
            <path>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok-mapstruct-binding</artifactId>
	        <version>0.2.0</version>
            </path>
        </annotationProcessorPaths>
    </configuration>
</plugin>
```**  **让我们使用 Lombok 注释来定义源实体:

```
@Getter
@Setter
public class Car {
    private int id;
    private String name;
}
```

和目的地数据传输对象:

```
@Getter
@Setter
public class CarDTO {
    private int id;
    private String name;
}
```

这个映射器接口与我们之前的示例相似:

```
@Mapper
public interface CarMapper {
    CarMapper INSTANCE = Mappers.getMapper(CarMapper.class);
    CarDTO carToCarDTO(Car car);
}
```

## 12。`defaultExpression`支持

从版本 1.3.0 开始，**我们可以使用`@Mapping`注释的`defaultExpression`属性来指定一个表达式，如果源字段是`null`，该表达式将确定目标字段的值。**这是对现有`defaultValue`属性功能的补充。

源实体:

```
public class Person {
    private int id;
    private String name;
}
```

目标数据传输对象:

```
public class PersonDTO {
    private int id;
    private String name;
}
```

如果源实体的`id`字段是`null`，我们希望生成一个随机的`id`，并将其分配给目标，同时保持其他属性值不变:

```
@Mapper
public interface PersonMapper {
    PersonMapper INSTANCE = Mappers.getMapper(PersonMapper.class);

    @Mapping(target = "id", source = "person.id", 
      defaultExpression = "java(java.util.UUID.randomUUID().toString())")
    PersonDTO personToPersonDTO(Person person);
}
```

让我们添加一个测试用例来验证表达式的执行:

```
@Test
public void givenPersonEntitytoPersonWithExpression_whenMaps_thenCorrect() 
    Person entity  = new Person();
    entity.setName("Micheal");
    PersonDTO personDto = PersonMapper.INSTANCE.personToPersonDTO(entity);
    assertNull(entity.getId());
    assertNotNull(personDto.getId());
    assertEquals(personDto.getName(), entity.getName());
}
```

## 13。结论

本文介绍了 MapStruct。我们已经介绍了映射库的大部分基础知识，以及如何在我们的应用程序中使用它。

这些例子和测试的实现可以在 [GitHub](https://web.archive.org/web/20220706110125/https://github.com/eugenp/tutorials/tree/master/mapstruct) 项目中找到。这是一个 Maven 项目，因此应该很容易导入和运行。**