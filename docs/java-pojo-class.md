# 什么是 POJO 类？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-pojo-class>

## 1.概观

在这个简短的教程中，**我们将研究“普通旧 Java 对象”**或简称 POJO 的定义。

我们将看看 POJO 与 JavaBean 的比较，以及将 POJO 转换成 JavaBean 有什么帮助。

## 2.普通的旧 Java 对象

### 2.1.什么是`POJO`？

当我们谈论 POJO 时，我们描述的是一种简单的类型，不涉及任何特定的框架。POJO 对我们的属性和方法没有命名约定。

让我们创建一个基本的员工 POJO。它有三个属性。名字、姓氏和开始日期:

```java
public class EmployeePojo {

    public String firstName;
    public String lastName;
    private LocalDate startDate;

    public EmployeePojo(String firstName, String lastName, LocalDate startDate) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.startDate = startDate;
    }

    public String name() {
        return this.firstName + " " + this.lastName;
    }

    public LocalDate getStart() {
        return this.startDate;
    }
}
```

这个类可以被任何 Java 程序使用，因为它不依赖于任何框架。

但是，我们并没有遵循任何真正的惯例来构造、访问或修改类的状态。

这种惯例的缺乏导致了两个问题:

首先，它增加了试图理解如何使用它的程序员的学习曲线。

第二，它可能会限制一个框架的能力，使其更倾向于约定而不是配置，理解如何使用类，并增加其功能。

为了探索这第二点，让我们使用反射与`EmployeePojo` 一起工作。因此，我们将开始发现它的一些局限性。

### 2.2.用 POJO 反思

让我们将[`commons-beanutils`](https://web.archive.org/web/20220719161511/https://search.maven.org/search?q=g:commons-beanutils%20AND%20a:commons-beanutils) `[dependency](https://web.archive.org/web/20220719161511/https://search.maven.org/search?q=g:commons-beanutils%20AND%20a:commons-beanutils)`添加到我们的项目中:

```java
<dependency>
    <groupId>commons-beanutils</groupId>
    <artifactId>commons-beanutils</artifactId>
    <version>1.9.4</version>
</dependency>
```

现在，让我们检查 POJO 的属性:

```java
List<String> propertyNames =
  PropertyUtils.getPropertyDescriptors(EmployeePojo.class).stream()
    .map(PropertyDescriptor::getDisplayName)
    .collect(Collectors.toList());
```

如果我们将`propertyNames` 输出到控制台，我们只会看到:

```java
[start] 
```

在这里，我们看到我们只获得了`start `作为类的属性。 **`PropertyUtils`未能找到另外两个。**

如果我们使用其他库，比如[杰克森](/web/20220719161511/https://www.baeldung.com/jackson)来处理`EmployeePojo.`，我们会看到同样的结果

理想情况下，我们会看到所有的属性:`firstName`、`lastName,` 和`startDate.` 、T3，好消息是许多 Java 库默认支持 JavaBean 命名约定。

## 3.JavaBeans

### 3.1.什么是`JavaBean`？

JavaBean 仍然是 POJO，但是引入了一套严格的规则来实现它:

*   访问级别——我们的属性是私有的，我们公开 getters 和 setters
*   方法名——我们的 getter 和 setter 遵循`getX`和`setX`约定(在布尔值的情况下，`isX`可以用作 getter)
*   默认构造函数–必须存在无参数构造函数，以便可以在不提供参数的情况下创建实例，例如在反序列化期间
*   serializable——实现`Serializable`接口允许我们存储状态

### 3.2.作为一个 JavaBean

所以，让我们试着将`EmployeePojo`转换成 JavaBean:

```java
public class EmployeeBean implements Serializable {

    private static final long serialVersionUID = -3760445487636086034L;
    private String firstName;
    private String lastName;
    private LocalDate startDate;

    public EmployeeBean() {
    }

    public EmployeeBean(String firstName, String lastName, LocalDate startDate) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.startDate = startDate;
    }

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    //  additional getters/setters

}
```

### 3.3.用 JavaBean 进行反射

当我们用反射检查 bean 时，现在我们得到了属性的完整列表:

```java
[firstName, lastName, startDate]
```

## 4.使用 JavaBeans 时的权衡

因此，我们已经展示了 JavaBeans 有帮助的方式。请记住，每个设计选择都有权衡。

当我们使用 JavaBeans 时，我们也应该注意一些潜在的缺点:

*   我们的 JavaBeans 由于它们的 setter 方法是可变的——这可能导致并发性或一致性问题
*   我们必须为所有的属性引入 getters，为大多数属性引入 setters，这大部分可能是不必要的
*   我们经常需要构造函数中的参数来确保对象在有效的状态下被实例化，但是 JavaBean 标准要求我们提供一个零参数的构造函数

考虑到这些权衡，这些年来框架也适应了其他 bean 约定。

## 5.结论

在本教程中，我们比较了 POJOs 和 JavaBeans。

首先，我们了解到 POJO 是一种不绑定到特定框架的 Java 对象，JavaBean 是一种特殊类型的 POJO，有一套严格的约定。

然后，我们看到了一些框架和库如何利用 JavaBean 命名约定来发现类的属性。

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20220719161511/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-2)