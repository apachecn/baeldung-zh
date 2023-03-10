# 带有继承的 Lombok @Builder

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/lombok-builder-inheritance>

## 1。概述

Lombok 库提供了一种无需编写任何样板代码就能实现`[Builder Pattern](https://web.archive.org/web/20221126223852/https://en.wikipedia.org/wiki/Builder_pattern#Java)`的好方法:注释`[@Builder](https://web.archive.org/web/20221126223852/https://projectlombok.org/features/Builder)`。

在这个简短的教程中，我们将专门学习**在涉及继承时如何处理`@Builder`注释。**我们将展示两种技术。一个依赖于标准的龙目岛特征。另一个利用了 Lombok 1.18 中引入的一个实验特性。

有关构建器注释的更广泛概述，请参考使用 Lombok 的`@Builder`注释的[。](/web/20221126223852/https://www.baeldung.com/lombok-builder)

关于 Lombok 项目库的详细信息也可以在[Lombok 项目简介](/web/20221126223852/https://www.baeldung.com/intro-to-project-lombok)中找到。

## 2。龙目岛`@Builder`和遗产

### 2.1。定义问题

假设我们的`Child`类扩展了一个`Parent`类:

```java
@Getter
@AllArgsConstructor
public class Parent {
    private final String parentName;
    private final int parentAge;
}

@Getter
@Builder
public class Child extends Parent {
    private final String childName;
    private final int childAge;
} 
```

当在扩展另一个类的类上使用`@Builder`时，我们会在注释上得到下面的编译错误:

> 隐式超级构造函数父级()未定义。必须显式调用另一个构造函数

这是因为 Lombok 不考虑超类的字段，而只考虑当前类的字段。

### 2.2。解决问题

幸运的是，有一个简单的解决方法。我们可以(用我们的 IDE 或者甚至手动)生成一个基于字段的构造函数。这也包括超类中的字段。

我们用`@Builder`来注释它，而不是类:

```java
@Getter
@AllArgsConstructor
public class Parent {
    private final String parentName;
    private final int parentAge;
}

@Getter
public class Child extends Parent {
    private final String childName;
    private final int childAge;

    @Builder
    public Child(String parentName, int parentAge, String childName, int childAge) {
        super(parentName, parentAge);
        this.childName = childName;
        this.childAge = childAge;
    }
} 
```

这样，我们将能够从`Child`类访问一个方便的构建器，这将允许我们指定`Parent`类字段:

```java
Child child = Child.builder()
  .parentName("Andrea")
  .parentAge(38)
  .childName("Emma")
  .childAge(6)
  .build();

assertThat(child.getParentName()).isEqualTo("Andrea");
assertThat(child.getParentAge()).isEqualTo(38);
assertThat(child.getChildName()).isEqualTo("Emma");
assertThat(child.getChildAge()).isEqualTo(6);
```

### 2.3。让多个`@Builder`共存

如果超类本身是用`@Builder`注释的，我们在注释`Child`类构造函数时会得到以下错误:

> 返回类型与 Parent.builder()不兼容

这是因为`Child`类试图公开两个同名的`Builder`。

我们可以通过为至少一个构建器方法分配一个唯一的名称来解决这个问题:

```java
@Getter
public class Child extends Parent {
    private final String childName;
    private final int childAge;

    @Builder(builderMethodName = "childBuilder")
    public Child(String parentName, int parentAge, String childName, int childAge) {
        super(parentName, parentAge);
        this.childName = childName;
        this.childAge = childAge;
    }
} 
```

然后我们将能够从`ParentBuilder`到`Child.builder()`获得一个`ChildBuilder`到`Child.childBuilder()`。

### 2.4.支持更大的继承层次结构

在某些情况下，我们可能需要支持更深层次的继承。我们可以利用和以前一样的模式。

让我们创建一个`Child`的子类:

```java
@Getter
public class Student extends Child {

    private final String schoolName;

    @Builder(builderMethodName = "studentBuilder")
    public Student(String parentName, int parentAge, String childName, int childAge, String schoolName) {
        super(parentName, parentAge, childName, childAge);
        this.schoolName = schoolName;
    }
}
```

和以前一样，我们需要手动添加一个构造函数。这需要接受来自所有父类和子类的所有属性作为参数。然后我们像以前一样添加`@Builder`注释。

通过在注释中提供另一个惟一的方法名，我们可以获得`Parent`、`Child`或`Student`的构建器:

```java
Student student = Student.studentBuilder()
  .parentName("Andrea")
  .parentAge(38)
  .childName("Emma")
  .childAge(6)
  .schoolName("Baeldung High School")
  .build();

assertThat(student.getChildName()).isEqualTo("Emma");
assertThat(student.getChildAge()).isEqualTo(6);
assertThat(student.getParentName()).isEqualTo("Andrea");
assertThat(student.getParentAge()).isEqualTo(38);
assertThat(student.getSchoolName()).isEqualTo("Baeldung High School");
```

然后我们可以扩展这个模式来处理任何深度的继承。我们需要创建的构造函数可能会变得很大，但是我们的 IDE 可以帮助我们。

## 3。龙目岛`@SuperBuilder`和遗产

正如我们前面提到的，**Lombok 的 1.18 版本引入了 [`@SuperBuilder`](https://web.archive.org/web/20221126223852/https://projectlombok.org/features/experimental/SuperBuilder) 注释。**我们可以利用这一点以更简单的方式解决我们的问题。

### 3.1.应用注释

我们可以制造一个可以看到其祖先属性的构建器。

为此，我们用`@SuperBuilder`注释来注释我们的类及其祖先。

让我们在这里演示一下我们的三层结构。

请注意，简单父子继承的原则是相同的:

```java
@Getter
@SuperBuilder
public class Parent {
    // same as before...

@Getter
@SuperBuilder
public class Child extends Parent {
   // same as before...

@Getter
@SuperBuilder
public class Student extends Child {
   // same as before...
```

当所有的类都以这种方式标注时，我们得到了一个子类的构建器，它也公开了父类的属性。

注意，我们必须注释所有的类。 **`@SuperBuilder`不能与`@Builder`在同一个类层次内混合。**这样做会导致编译错误。

### 3.2.使用构建器

这次，我们不需要定义任何特殊的构造函数。

由`@SuperBuilder`生成的构建器类的行为就像我们使用主 Lombok `@Builder`生成的一样:

```java
Student student = Student.builder()
  .parentName("Andrea")
  .parentAge(38)
  .childName("Emma")
  .childAge(6)
  .schoolName("Baeldung High School")
  .build();

assertThat(student.getChildName()).isEqualTo("Emma");
assertThat(student.getChildAge()).isEqualTo(6);
assertThat(student.getParentName()).isEqualTo("Andrea");
assertThat(student.getParentAge()).isEqualTo(38);
assertThat(student.getSchoolName()).isEqualTo("Baeldung High School");
```

## 4。结论

我们已经看到了如何处理在利用继承的类中使用`@Builder`注释的常见陷阱。

如果我们使用主 Lombok `@Builder`注释，我们有一些额外的步骤来使它工作。但是如果我们愿意使用实验特性，`@SuperBuilder` 可以简化事情。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221126223852/https://github.com/eugenp/tutorials/tree/master/lombok-modules/lombok)