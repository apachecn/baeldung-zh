# 使用 MapStruct 映射集合

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mapstruct-mapping-collections>

## 1.概观

在本教程中，我们将看看如何使用 MapStruct 映射对象集合。

由于本文假设已经对 MapStruct 有了基本的了解，初学者应该先看看我们的 MapStruct 快速指南。

## 2.映射集合

一般来说，**用 MapStruct 映射集合的工作方式与简单类型**相同。

基本上，我们必须创建一个简单的接口或抽象类，并声明映射方法。基于我们的声明，MapStruct 将自动生成映射代码。通常，**生成的代码将在源集合中循环，将每个元素转换成目标类型，并在目标集合中包含每个元素**。

让我们看一个简单的例子。

### 2.1.映射列表

首先，对于我们的示例，让我们考虑一个简单的 POJO 作为映射器的映射源:

```
public class Employee {
    private String firstName;
    private String lastName;

    // constructor, getters and setters
} 
```

目标将是一个简单的 DTO:

```
public class EmployeeDTO {

    private String firstName;
    private String lastName;

    // getters and setters
}
```

接下来，让我们定义我们的映射器:

```
@Mapper
public interface EmployeeMapper {
    List<EmployeeDTO> map(List<Employee> employees);
} 
```

最后，让我们看看从我们的`EmployeeMapper` 接口生成的代码 MapStruct:

```
public class EmployeeMapperImpl implements EmployeeMapper {

    @Override
    public List<EmployeeDTO> map(List<Employee> employees) {
        if (employees == null) {
            return null;
        }

        List<EmployeeDTO> list = new ArrayList<EmployeeDTO>(employees.size());
        for (Employee employee : employees) {
            list.add(employeeToEmployeeDTO(employee));
        }

        return list;
    }

    protected EmployeeDTO employeeToEmployeeDTO(Employee employee) {
        if (employee == null) {
            return null;
        }

        EmployeeDTO employeeDTO = new EmployeeDTO();

        employeeDTO.setFirstName(employee.getFirstName());
        employeeDTO.setLastName(employee.getLastName());

        return employeeDTO;
    }
} 
```

有一件重要的事情需要注意。具体来说， **MapStruct 为我们自动生成了从`Employee`到`EmployeeDTO`到**的映射。

有些情况下这是不可能的。例如，假设我们想要将我们的`Employee`模型映射到下面的模型:

```
public class EmployeeFullNameDTO {

    private String fullName;

    // getter and setter
}
```

在这种情况下，如果我们只声明从`Employee`的`List`到`EmployeeFullNameDTO` 的`List`的映射方法，我们将会收到一个编译时错误或警告，如下所示:

```
Warning:(11, 31) java: Unmapped target property: "fullName". 
  Mapping from Collection element "com.baeldung.mapstruct.mappingCollections.model.Employee employee" to 
  "com.baeldung.mapstruct.mappingCollections.dto.EmployeeFullNameDTO employeeFullNameDTO".
```

基本上，这意味着 **MapStruct 不能为我们** **自动生成映射，在这种情况下**。因此，我们需要手动定义`Employee`和`EmployeeFullNameDTO.`之间的映射

鉴于这几点，让我们手动定义它:

```
@Mapper
public interface EmployeeFullNameMapper {

    List<EmployeeFullNameDTO> map(List<Employee> employees);

    default EmployeeFullNameDTO map(Employee employee) {
        EmployeeFullNameDTO employeeInfoDTO = new EmployeeFullNameDTO();
        employeeInfoDTO.setFullName(employee.getFirstName() + " " + employee.getLastName());

        return employeeInfoDTO;
    }
}
```

**生成的代码将使用我们定义的方法将源`List`的元素映射到目标`List`** 。

这也适用于一般情况。如果我们已经定义了一个将源元素类型映射到目标元素类型的方法，MapStruct 将使用它。

### 2.2.映射集和映射

使用 MapStruct 映射集合的方式与使用列表的方式相同。例如，假设我们想要将一个`Employee` 实例的`Set` 映射到一个`EmployeeDTO` 实例的`Set``.`

和以前一样，我们需要一个映射器:

```
@Mapper
public interface EmployeeMapper {

    Set<EmployeeDTO> map(Set<Employee> employees);
}
```

MapStruct 将生成适当的代码:

```
public class EmployeeMapperImpl implements EmployeeMapper {

    @Override
    public Set<EmployeeDTO> map(Set<Employee> employees) {
        if (employees == null) {
            return null;
        }

        Set<EmployeeDTO> set = 
          new HashSet<EmployeeDTO>(Math.max((int)(employees.size() / .75f ) + 1, 16));
        for (Employee employee : employees) {
            set.add(employeeToEmployeeDTO(employee));
        }

        return set;
    }

    protected EmployeeDTO employeeToEmployeeDTO(Employee employee) {
        if (employee == null) {
            return null;
        }

        EmployeeDTO employeeDTO = new EmployeeDTO();

        employeeDTO.setFirstName(employee.getFirstName());
        employeeDTO.setLastName(employee.getLastName());

        return employeeDTO;
    }
}
```

这同样适用于地图。假设我们想将一个`Map<String, Employee>` 映射到一个`Map<String, EmployeeDTO>`。

然后，我们可以遵循与之前相同的步骤:

```
@Mapper
public interface EmployeeMapper {

    Map<String, EmployeeDTO> map(Map<String, Employee> idEmployeeMap);
}
```

MapStruct 完成了它的工作:

```
public class EmployeeMapperImpl implements EmployeeMapper {

    @Override
    public Map<String, EmployeeDTO> map(Map<String, Employee> idEmployeeMap) {
        if (idEmployeeMap == null) {
            return null;
        }

        Map<String, EmployeeDTO> map = new HashMap<String, EmployeeDTO>(Math.max((int)(idEmployeeMap.size() / .75f) + 1, 16));

        for (java.util.Map.Entry<String, Employee> entry : idEmployeeMap.entrySet()) {
            String key = entry.getKey();
            EmployeeDTO value = employeeToEmployeeDTO(entry.getValue());
            map.put(key, value);
        }

        return map;
    }

    protected EmployeeDTO employeeToEmployeeDTO(Employee employee) {
        if (employee == null) {
            return null;
        }

        EmployeeDTO employeeDTO = new EmployeeDTO();

        employeeDTO.setFirstName(employee.getFirstName());
        employeeDTO.setLastName(employee.getLastName());

        return employeeDTO;
    }
}
```

## 3.集合映射策略

通常，我们需要映射具有父子关系的数据类型。通常，我们有一个**数据类型(父类型),它有另一个数据类型(子类型)的字段`Collection`。**

对于这种情况， **MapStruct 提供了一种选择如何设置或添加子类型到父类型的方法。**特别是`@Mapper` 注释有一个`collectionMappingStrategy` 属性，可以是`ACCESSOR_ONLY`、`SETTER_PREFERRED`、`ADDER_PREFERRED`或`TARGET_IMMUTABLE`。

所有这些值都是指子类型应该被设置或添加到父类型的方式。**默认值是`ACCESSOR_ONLY,`** ，这意味着只有访问器可以用来设置子节点的`Collection`。

当`Collection`字段的**设置器不可用，但我们有一个加法器时，这个选项很方便。**另一个有用的例子是**，此时`Collection`在父类型**上是不可变的。通常，我们会在生成的目标类型中遇到这些情况。

### 3.1.`ACCESSOR_ONLY`集合映射策略

让我们举一个例子来更好地理解这是如何工作的。

对于我们的例子，让我们创建一个`Company`类作为我们的映射源:

```
public class Company {

    private List<Employee> employees;

   // getter and setter
}
```

我们映射的目标将是一个简单的 DTO:

```
public class CompanyDTO {

    private List<EmployeeDTO> employees;

    public List<EmployeeDTO> getEmployees() {
        return employees;
    }

    public void setEmployees(List<EmployeeDTO> employees) {
        this.employees = employees;
    }

    public void addEmployee(EmployeeDTO employeeDTO) {
        if (employees == null) {
            employees = new ArrayList<>();
        }

        employees.add(employeeDTO);
    }
}
```

注意，我们有设置器`setEmployees,`和加法器`addEmployee,`可用。另外，**对于加法器，我们负责集合初始化。**

现在，假设我们想要将一个`Company`映射到一个`CompanyDTO.` ，和之前一样，我们需要一个映射器:

```
@Mapper(uses = EmployeeMapper.class)
public interface CompanyMapper {
    CompanyDTO map(Company company);
}
```

注意，我们重用了`EmployeeMapper` 和默认的`collectionMappingStrategy.`

现在，让我们看看 MapStruct 生成的代码:

```
public class CompanyMapperImpl implements CompanyMapper {

    private final EmployeeMapper employeeMapper = Mappers.getMapper(EmployeeMapper.class);

    @Override
    public CompanyDTO map(Company company) {
        if (company == null) {
            return null;
        }

        CompanyDTO companyDTO = new CompanyDTO();

        companyDTO.setEmployees(employeeMapper.map(company.getEmployees()));

        return companyDTO;
    }
}
```

可以看出， **MapStruct 使用 setter`setEmployees`来设置`EmployeeDTO`实例**的`List`。发生这种情况是因为这里我们使用了默认的`collectionMappingStrategy,` `ACCESSOR_ONLY.`

此外，MapStruct 在 `EmployeeMapper` 中找到了一个将`List<Employee>` 映射到`List<EmployeeDTO>` 的方法，并重用了它。

### 3.2.`ADDER_PREFERRED`集合映射策略

相比之下，让我们考虑使用`ADDER_PREFERRED` 作为`collectionMappingStrategy`:

```
@Mapper(collectionMappingStrategy = CollectionMappingStrategy.ADDER_PREFERRED,
        uses = EmployeeMapper.class)
public interface CompanyMapperAdderPreferred {
    CompanyDTO map(Company company);
}
```

同样，我们希望重用`EmployeeMapper`。然而，**我们需要显式地添加一个方法，该方法可以将单个`Employee`转换为第一个`EmployeeDTO`**:

```
@Mapper
public interface EmployeeMapper {
    EmployeeDTO map(Employee employee);
    List map(List employees);
    Set map(Set employees);
    Map<String, EmployeeDTO> map(Map<String, Employee> idEmployeeMap);
}
```

这是因为 MapStruct 会使用加法器将 **`EmployeeDTO`实例逐个添加到目标`CompanyDTO`实例中**:

```
public class CompanyMapperAdderPreferredImpl implements CompanyMapperAdderPreferred {

    private final EmployeeMapper employeeMapper = Mappers.getMapper( EmployeeMapper.class );

    @Override
    public CompanyDTO map(Company company) {
        if ( company == null ) {
            return null;
        }

        CompanyDTO companyDTO = new CompanyDTO();

        if ( company.getEmployees() != null ) {
            for ( Employee employee : company.getEmployees() ) {
                companyDTO.addEmployee( employeeMapper.map( employee ) );
            }
        }

        return companyDTO;
    }
}
```

在加法器不可用的情况下，将使用设置器。

我们可以在 MapStruct 的[参考文档](https://web.archive.org/web/20220706110116/https://mapstruct.org/documentation/stable/reference/html/#collection-mapping-strategies)中找到所有集合映射策略的完整描述。

## 4.目标集合的实现类型

MapStruct 支持集合接口作为映射方法的目标类型。

在这种情况下，生成的代码中使用了一些默认实现。例如，从上面的例子中可以看出，`List`的默认实现是`ArrayList`。

我们可以在[参考文档](https://web.archive.org/web/20220706110116/https://mapstruct.org/documentation/stable/reference/html/#implementation-types-for-collection-mappings)中找到 MapStruct 支持的接口的完整列表以及它为每个接口使用的默认实现。

## 5.结论

在本文中，我们探讨了如何使用 MapStruct 映射集合。

首先，我们看了如何映射不同类型的集合。然后，我们看到了如何使用集合映射策略定制父子关系映射器。

在这个过程中，我们强调了使用 MapStruct 映射集合时需要记住的要点和事项。

和往常一样，完整的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220706110116/https://github.com/eugenp/tutorials/tree/master/mapstruct)