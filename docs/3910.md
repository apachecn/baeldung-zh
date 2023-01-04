# 使用反射从 Java 类中检索字段

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-reflection-class-fields>

## 1.概观

[反射](/web/20220926182111/https://www.baeldung.com/java-reflection)是计算机软件在运行时检查自身结构的能力。在 Java 中，我们通过使用`Java Reflection API`实现这一点。它允许我们在运行时检查类的元素，比如字段、方法甚至内部类。

本教程将重点介绍如何检索 Java 类的字段，包括私有字段和继承字段。

## 2.从类中检索字段

让我们先看看如何检索一个类的字段，不管它们是否可见。稍后，我们将看到如何获得继承字段。

让我们从一个有两个字段`String`的`Person`类的例子开始:`lastName`和`firstName`。前者是*保护的*(稍后会有用)而后者是`private:`

```
public class Person {
    protected String lastName;
    private String firstName;
}
```

我们希望使用反射获得`lastName`和`firstName`字段。我们将通过使用`Class::getDeclaredFields`方法来实现这一点。顾名思义，它以`Field`数组:的形式返回一个类的所有`declared`字段

```
public class PersonAndEmployeeReflectionUnitTest {

    /* ... constants ... */

    @Test
    public void givenPersonClass_whenGetDeclaredFields_thenTwoFields() {
        Field[] allFields = Person.class.getDeclaredFields();

        assertEquals(2, allFields.length);

        assertTrue(Arrays.stream(allFields).anyMatch(field ->
          field.getName().equals(LAST_NAME_FIELD)
            && field.getType().equals(String.class))
        );
        assertTrue(Arrays.stream(allFields).anyMatch(field ->
          field.getName().equals(FIRST_NAME_FIELD)
            && field.getType().equals(String.class))
        );
    }

}
```

正如我们所看到的，我们得到了`Person`类的两个字段。我们检查与`Person`类中的字段定义相匹配的名称和类型。

## 3.检索继承的字段

现在让我们看看如何获得 Java 类的继承字段。

为了说明这一点，让我们创建第二个名为`Employee`的扩展`Person`的类，它有自己的字段:

```
public class Employee extends Person {
    public int employeeId;
}
```

### 3.1.检索简单类层次结构上的继承字段

**使用`Employee.class.getDeclaredFields()`只会返回`employeeId`字段**，因为这个方法不会返回超类中声明的字段。为了获得继承的字段，我们还必须获得`Person`超类的字段。

当然，我们可以对 *Person* 和 *Employee* 类使用`getDeclaredFields()`方法，并将它们的结果合并到一个数组中。但是如果我们不想显式地指定超类呢？

在这种情况下，**我们可以利用`Java Reflection API` : `Class::getSuperclass`** 的另一种方法。这给了我们另一个类的超类，而我们不需要知道那个超类是什么。

让我们收集`Employee.class`和`Employee.class.getSuperclass()`上`getDeclaredFields()`的结果，并将它们合并成一个数组:

```
@Test
public void givenEmployeeClass_whenGetDeclaredFieldsOnBothClasses_thenThreeFields() {
    Field[] personFields = Employee.class.getSuperclass().getDeclaredFields();
    Field[] employeeFields = Employee.class.getDeclaredFields();
    Field[] allFields = new Field[employeeFields.length + personFields.length];
    Arrays.setAll(allFields, i -> 
      (i < personFields.length ? personFields[i] : employeeFields[i - personFields.length]));

    assertEquals(3, allFields.length);

    Field lastNameField = allFields[0];
    assertEquals(LAST_NAME_FIELD, lastNameField.getName());
    assertEquals(String.class, lastNameField.getType());

    Field firstNameField = allFields[1];
    assertEquals(FIRST_NAME_FIELD, firstNameField.getName());
    assertEquals(String.class, firstNameField.getType());

    Field employeeIdField = allFields[2];
    assertEquals(EMPLOYEE_ID_FIELD, employeeIdField.getName());
    assertEquals(int.class, employeeIdField.getType());
}
```

我们可以看到，我们已经收集了两个字段`Person `和一个字段`Employee`。

但是，`Person`的`private`字段真的是继承字段吗？没有那么多。对于`package-private`油田来说也是如此。**只有`public`和`protected`字段被认为是继承的。**

### 3.2.过滤`public`和`protected`字段

不幸的是，Java API 中没有任何方法允许我们从一个类及其超类中收集`public `和`protected`字段。`Class::getFields`方法接近我们的目标，因为它返回一个类及其超类的所有`public`字段，但不返回`protected`字段。

我们必须只获取继承字段的唯一方法是使用`getDeclaredFields()`方法，就像我们刚才做的那样，并使用`Field::getModifiers `方法过滤它的结果。这个函数返回一个代表当前字段修饰符的`int`。每个可能的修改器被分配一个在`2^0`和`2^7`之间的 2 的幂。

比如`public`是 `2^0`，`static`是`2^3`。因此，在`public `和`static`字段上调用`getModifiers()`方法将返回 9。

然后，可以在这个值和特定修饰符的值之间执行一个`bitwise and`,看看那个字段是否有那个修饰符。如果操作返回的不是 0，则应用修饰符，否则不应用。

我们很幸运，因为 Java 为我们提供了一个实用程序类来检查修饰符是否出现在由`getModifiers()`返回的值中。**让我们使用`isPublic()`和`isProtected()`方法只收集我们示例中的继承字段:**

```
List<Field> personFields = Arrays.stream(Employee.class.getSuperclass().getDeclaredFields())
  .filter(f -> Modifier.isPublic(f.getModifiers()) || Modifier.isProtected(f.getModifiers()))
  .collect(Collectors.toList());

assertEquals(1, personFields.size());

assertTrue(personFields.stream().anyMatch(field ->
  field.getName().equals(LAST_NAME_FIELD)
    && field.getType().equals(String.class))
);
```

正如我们所看到的，结果不再携带`private`字段。

### 3.3.检索深层类层次结构上的继承字段

在上面的例子中，我们处理了一个单一的类层次结构。如果我们有一个更深的类层次结构，并希望收集所有继承的字段，我们现在该怎么办？

假设我们有一个子类`Employee`或者一个超类`Person –`，那么获取整个层次结构的字段将需要检查所有的超类。

我们可以通过创建贯穿层次结构的实用方法来实现这一点，为我们构建完整的结果:

```
List<Field> getAllFields(Class clazz) {
    if (clazz == null) {
        return Collections.emptyList();
    }

    List<Field> result = new ArrayList<>(getAllFields(clazz.getSuperclass()));
    List<Field> filteredFields = Arrays.stream(clazz.getDeclaredFields())
      .filter(f -> Modifier.isPublic(f.getModifiers()) || Modifier.isProtected(f.getModifiers()))
      .collect(Collectors.toList());
    result.addAll(filteredFields);
    return result;
}
```

**这个递归方法将在类层次结构中搜索`public`和`protected`字段，并返回在`List`中找到的所有字段。**

让我们用一个新的`MonthEmployee`类的小测试来说明它，扩展了`Employee`类:

```
public class MonthEmployee extends Employee {
    protected double reward;
}
```

这个类定义了一个新的字段-`reward`。**给定所有的层次类，我们的方法应该给我们以下字段**定义:`Person::lastName, Employee::employeeId`和 `MonthEmployee::reward`。

让我们调用`MonthEmployee`上的`getAllFields()`方法:

```
@Test
public void givenMonthEmployeeClass_whenGetAllFields_thenThreeFields() {
    List<Field> allFields = getAllFields(MonthEmployee.class);

    assertEquals(3, allFields.size());

    assertTrue(allFields.stream().anyMatch(field ->
      field.getName().equals(LAST_NAME_FIELD)
        && field.getType().equals(String.class))
    );
    assertTrue(allFields.stream().anyMatch(field ->
      field.getName().equals(EMPLOYEE_ID_FIELD)
        && field.getType().equals(int.class))
    );
    assertTrue(allFields.stream().anyMatch(field ->
      field.getName().equals(MONTH_EMPLOYEE_REWARD_FIELD)
        && field.getType().equals(double.class))
    );
}
```

正如所料，我们收集了所有的`public`和`protected`字段。

## 4.结论

在本文中，我们看到了如何使用`Java Reflection API`检索 Java 类的字段。

我们首先学习了如何检索一个类的声明字段。之后，我们还看到了如何检索它的超类字段。然后，我们学习了过滤掉非`public` 和非`protected` 字段。

最后，我们看到了如何应用所有这些来收集多类层次结构的继承字段。

像往常一样，这篇文章的完整代码可以在我们的 GitHub 上找到[。](https://web.archive.org/web/20220926182111/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-reflection)