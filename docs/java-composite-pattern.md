# Java 中的复合设计模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-composite-pattern>

## 1。简介

在这个快速教程中，我们将介绍 Java 中的复合设计模式。

我们将描述它的结构和用途。

## 2。结构

**组合模式意味着允许以同样的方式处理单个对象和对象的组合，或“组合”。**

它可以被视为由继承基类型的类型组成的树结构，它可以表示对象的单个部分或整个层次结构。

我们可以将模式分解为:

*   组件–是组合中所有对象的基本接口。它应该是一个接口或者是一个抽象类，具有管理子复合的通用方法。
*   leaf–实现基本组件的默认行为。它不包含对其他对象的引用。
*   复合–具有叶元素。它实现了基本组件方法，并定义了与子组件相关的操作。
*   客户端–可以通过使用基本组件对象来访问组合元素。

## 3。实际例子

现在，让我们深入到实现中。让我们假设我们想要在一个公司里建立一个部门的层级结构。

### 3.1。底座组件

作为一个组件对象，我们将定义一个简单的`Department`接口:

```java
public interface Department {
    void printDepartmentName();
}
```

### 3.2。树叶

对于叶组件，让我们定义财务和销售部门的类:

```java
public class FinancialDepartment implements Department {

    private Integer id;
    private String name;

    public void printDepartmentName() {
        System.out.println(getClass().getSimpleName());
    }

    // standard constructor, getters, setters
}
```

`SalesDepartment,`第二叶类，也类似:

```java
public class SalesDepartment implements Department {

    private Integer id;
    private String name;

    public void printDepartmentName() {
        System.out.println(getClass().getSimpleName());
    }

    // standard constructor, getters, setters
}
```

这两个类都从基本组件实现了`printDepartmentName()`方法，并打印出了它们各自的类名。

此外，因为它们是叶类，所以不包含其他的`Department`对象。

接下来，让我们看看一个复合类。

### 3.3。复合元素

作为一个复合类，让我们创建一个`HeadDepartment`类:

```java
public class HeadDepartment implements Department {
    private Integer id;
    private String name;

    private List<Department> childDepartments;

    public HeadDepartment(Integer id, String name) {
        this.id = id;
        this.name = name;
        this.childDepartments = new ArrayList<>();
    }

    public void printDepartmentName() {
        childDepartments.forEach(Department::printDepartmentName);
    }

    public void addDepartment(Department department) {
        childDepartments.add(department);
    }

    public void removeDepartment(Department department) {
        childDepartments.remove(department);
    }
}
```

**这是一个复合类，因为它包含了一组`Department`组件**，以及在列表中添加和删除元素的方法。

复合`printDepartmentName()`方法是通过迭代叶子元素列表并为每个元素调用适当的方法来实现的。

## 4。测试

出于测试的目的，让我们来看看一个`CompositeDemo`类:

```java
public class CompositeDemo {
    public static void main(String args[]) {
        Department salesDepartment = new SalesDepartment(
          1, "Sales department");
        Department financialDepartment = new FinancialDepartment(
          2, "Financial department");

        HeadDepartment headDepartment = new HeadDepartment(
          3, "Head department");

        headDepartment.addDepartment(salesDepartment);
        headDepartment.addDepartment(financialDepartment);

        headDepartment.printDepartmentName();
    }
}
```

首先，我们为财务和销售部门创建两个实例。之后，我们实例化 head department，并将之前创建的实例添加到其中。

最后可以测试一下`printDepartmentName()`的构图方法。如我们所料，**输出包含每个叶组件的类名**:

```java
SalesDepartment
FinancialDepartment
```

## 5。结论

在本文中，我们学习了复合设计模式。文章突出了主要结构，并通过实际例子演示了使用方法。

像往常一样，完整的代码可以在 [Github 项目](https://web.archive.org/web/20221206080838/https://github.com/eugenp/tutorials/tree/master/patterns-modules/design-patterns-structural)中获得。