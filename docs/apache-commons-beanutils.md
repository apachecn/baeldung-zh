# Apache Commons BeanUtils

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-commons-beanutils>

## 1。概述

Apache Commons BeansUtils 包含了使用 Java beans 所需的所有工具。

简单地说，bean 是一个简单的 Java 类，包含字段、getter/setter 和一个无参数构造函数。

Java 提供反射和自省功能来识别 getter-setter 方法并动态调用它们。然而，这些 API 可能很难学习，可能需要开发人员编写样板代码来执行最简单的操作。

## 2。Maven 依赖关系

下面是在使用它之前需要包含在 POM 文件中的 Maven 依赖项:

```java
<dependency>
    <groupId>commons-beanutils</groupId>
    <artifactId>commons-beanutils</artifactId>
    <version>1.9.4</version>
</dependency>
```

最新版本可以在这里找到[。](https://web.archive.org/web/20221128113811/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22commons-beanutils%22%20AND%20a%3A%22commons-beanutils%22)

## 3。创建 Java Bean

让我们用典型的 getter 和 setter 方法创建两个 bean 类`Course`和`Student`。

```java
public class Course {
    private String name;
    private List<String> codes;
    private Map<String, Student> enrolledStudent = new HashMap<>();

    //  standard getters/setters
} 
```

```java
public class Student {
    private String name;

    //  standard getters/setters
}
```

我们有一个`Course`类，它有一个课程名称、课程代码和多个注册学生。注册的学生由唯一的注册 Id 标识。`Course`类在一个`Map`对象中维护注册的学生，其中注册 Id 是一个键，学生对象将是值。

## 4。属性访问

Bean 属性可以分为三类。

### 4.1。简单属性

单值属性也称为简单属性或标量属性。

它们的值可能是一个原语(如 int、float)或复杂类型的对象。BeanUtils 有一个`PropertyUtils`类，允许我们修改 Java Bean 中的简单属性。

下面是设置属性的示例代码:

```java
Course course = new Course();
String name = "Computer Science";
List<String> codes = Arrays.asList("CS", "CS01");

PropertyUtils.setSimpleProperty(course, "name", name);
PropertyUtils.setSimpleProperty(course, "codes", codes);
```

### 4.2。索引属性

索引属性有一个集合作为值，可以使用索引号单独访问。作为 JavaBean 的扩展，BeanUtils 也认为`java.util.List`类型值是有索引的。

我们可以使用`PropertyUtils's` `setIndexedProperty`方法修改索引属性的单个值。

下面是修改索引属性的示例代码:

```java
PropertyUtils.setIndexedProperty(course, "codes[1]", "CS02");
```

### 4.3。映射属性

任何将`java.util.Map`作为底层类型的属性都被称为映射属性。BeanUtils 允许我们使用一个`String-valued`键来更新映射中的单个值。

以下是修改映射属性中的值的示例代码:

```java
Student student = new Student();
String studentName = "Joe";
student.setName(studentName);

PropertyUtils.setMappedProperty(course, "enrolledStudent(ST-1)", student);
```

## 5。嵌套属性访问

如果一个属性值是一个对象，我们需要访问该对象内部的属性值——这将是访问一个嵌套的属性。`PropertyUtils`允许我们访问和**修改嵌套属性**。

假设我们想通过`Course`对象访问`Student`类的 name 属性。我们可能会写:

```java
String name = course.getEnrolledStudent("ST-1").getName();
```

我们可以使用`getNestedProperty`访问嵌套的属性值，并使用`PropertyUtils`中的`setNestedProperty`方法修改嵌套的属性。代码如下:

```java
Student student = new Student();
String studentName = "Joe";
student.setName(studentName);

String nameValue 
  = (String) PropertyUtils.getNestedProperty(
  course, "enrolledStudent(ST-1).name");
```

## 6。复制 Bean 属性

对于开发人员来说，将一个对象的属性复制到另一个对象通常是乏味且容易出错的。 **`BeanUtils`类提供了一个 *copyProperties* 方法，将源对象的属性复制到目标对象**中，其中两个对象中的属性名称相同。

让我们创建另一个 bean 类，就像上面创建的`Course` 一样，具有相同的属性，只是它没有`enrolledStudent`属性，而是属性名为`students`。让我们把这个类命名为`CourseEntity`。该类看起来像这样:

```java
public class CourseEntity {
    private String name;
    private List<String> codes;
    private Map<String, Student> students = new HashMap<>();

    //  standard getters/setters
}
```

现在我们将把*课程*对象的属性复制到*课程实体*对象:

```java
Course course = new Course();
course.setName("Computer Science");
course.setCodes(Arrays.asList("CS"));
course.setEnrolledStudent("ST-1", new Student());

CourseEntity courseEntity = new CourseEntity();
BeanUtils.copyProperties(courseEntity, course);
```

请记住，这只会复制具有相同名称的属性。因此，它不会复制 `Course`类中的属性`enrolledStudent`，因为`CourseEntity`类中没有同名的属性。

## 7。结论

在这篇简短的文章中，我们回顾了由`BeanUtils`提供的实用程序类。我们还研究了不同类型的属性，以及如何访问和修改它们的值。

最后，我们研究了访问嵌套属性值和将一个对象的属性复制到另一个对象。

当然，Java SDK 中的反射和自省功能也允许我们动态地访问属性，但是这可能很难学习，并且需要一些样板代码。允许我们通过一个方法调用来访问和修改这些值。

代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20221128113811/https://github.com/eugenp/tutorials/tree/master/libraries-apache-commons)