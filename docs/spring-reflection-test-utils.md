# 单元测试的反射测试工具指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-reflection-test-utils>

## 1。简介

`[ReflectionTestUtils](https://web.archive.org/web/20220713215745/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/util/ReflectionTestUtils.html)`是 Spring 测试上下文框架的一部分。它是在单元和集成测试场景中使用的基于反射的实用方法的集合，用于设置非公共字段、调用非公共方法和注入依赖关系。

在本教程中，我们将通过几个例子来看看如何在单元测试中使用`ReflectionTestUtils`。

## 2。Maven 依赖关系

让我们首先将我们的示例所需的所有必需依赖项的最新版本添加到我们的 `pom.xml`:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.1.2.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.1.2.RELEASE</version>
    <scope>test</scope>
</dependency>
```

最新的`[spring-context](https://web.archive.org/web/20220713215745/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-context%22), [spring-test](https://web.archive.org/web/20220713215745/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-test%22) `依赖项可以从 Maven 中央存储库下载。

## 3.使用`ReflectionTestUtils`设置非公共字段的值

假设我们需要在单元测试中使用一个具有私有字段的类的实例，而没有公共 setter 方法。

让我们从创建它开始:

```java
public class Employee {
    private Integer id;
    private String name;

    // standard getters/setters
}
```

通常我们不能访问私有字段`id`来为测试赋值，因为它没有公共的 setter 方法。

然后我们可以使用`ReflectionTestUtils.setField`方法给私有成员`id`赋值:

```java
@Test
public void whenNonPublicField_thenReflectionTestUtilsSetField() {
    Employee employee = new Employee();
    ReflectionTestUtils.setField(employee, "id", 1);

    assertTrue(employee.getId().equals(1));
}
```

## 4.使用`ReflectionTestUtils`调用非公共方法

现在让我们假设在`Employee`类中有一个私有方法`employeeToString`:

```java
private String employeeToString(){
    return "id: " + getId() + "; name: " + getName();
}
```

我们可以为下面的`employeeToString`方法编写一个单元测试，即使它没有任何来自`Employee`类外部的访问:

```java
@Test
public void whenNonPublicMethod_thenReflectionTestUtilsInvokeMethod() {
    Employee employee = new Employee();
    ReflectionTestUtils.setField(employee, "id", 1);
    employee.setName("Smith, John");

    assertTrue(ReflectionTestUtils.invokeMethod(employee, "employeeToString")
      .equals("id: 1; name: Smith, John"));
}
```

## 5.使用`ReflectionTestUtils`注入依赖关系

假设想要为下面的 Spring 组件编写一个单元测试，该组件有一个带有`@Autowired`注释的私有字段:

```java
@Component
public class EmployeeService {

    @Autowired
    private HRService hrService;

    public String findEmployeeStatus(Integer employeeId) {
        return "Employee " + employeeId + " status: " + hrService.getEmployeeStatus(employeeId);
    }
}
```

我们现在可以如下实现`HRService`组件:

```java
@Component
public class HRService {

    public String getEmployeeStatus(Integer employeeId) {
        return "Inactive";
    }
}
```

此外，让我们通过使用 [Mockito](/web/20220713215745/https://www.baeldung.com/mockito-annotations) 为`HRService`类创建一个模拟实现。我们将把这个模拟注入到`EmployeeService`实例中，并在我们的单元测试中使用它:

```java
HRService hrService = mock(HRService.class);
when(hrService.getEmployeeStatus(employee.getId())).thenReturn("Active");
```

因为`hrService`是一个没有公共 setter 的私有字段，我们将使用`ReflectionTestUtils.setField`方法将上面创建的 mock 注入这个私有字段。

```java
EmployeeService employeeService = new EmployeeService();
ReflectionTestUtils.setField(employeeService, "hrService", hrService);
```

最后，我们的单元测试看起来类似于这样:

```java
@Test
public void whenInjectingMockOfDependency_thenReflectionTestUtilsSetField() {
    Employee employee = new Employee();
    ReflectionTestUtils.setField(employee, "id", 1);
    employee.setName("Smith, John");

    HRService hrService = mock(HRService.class);
    when(hrService.getEmployeeStatus(employee.getId())).thenReturn("Active");
    EmployeeService employeeService = new EmployeeService();

    // Inject mock into the private field
    ReflectionTestUtils.setField(employeeService, "hrService", hrService);  

    assertEquals(
      "Employee " + employee.getId() + " status: Active", 
      employeeService.findEmployeeStatus(employee.getId()));
}
```

我们应该注意，这种技术是我们在 bean 类中使用字段注入这一事实的一种变通方法。如果我们切换到[构造函数注入](/web/20220713215745/https://www.baeldung.com/constructor-injection-in-spring)，那么这种方法就没有必要了。

## 6.结论

在本教程中，我们通过几个例子展示了如何在单元测试中使用`ReflectionTestUtils`。

代码样本一如既往地可以在 Github 上找到[。](https://web.archive.org/web/20220713215745/https://github.com/eugenp/tutorials/tree/master/testing-modules/spring-testing)