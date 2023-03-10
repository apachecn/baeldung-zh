# 用 Eclipse 生成 equals()和 hashCode()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-eclipse-equals-and-hashcode>

## 1。简介

在本文中，我们探索使用 Eclipse IDE 生成`equals()`和`hashCode()`方法。我们将展示 Eclipse 的代码自动生成是多么强大和方便，同时强调勤奋的代码测试仍然是必要的。

## 2。规则

Java 中的`equals()`用于检查两个对象是否等价。测试这一点的好方法是确保对象是对称的、自反的和可传递的。也就是说，对于三个非空对象`a`、`b`和`c`:

*   对称-当且仅当 b .等于(a)时，a .等于(b)
*   反身– `a.equals(a)`
*   可传递——如果`a.equals(b)`和`b.equals(c)`那么`a.equals(c)`

`hashCode()`必须遵守一条规则:

*   2 个为`equals()`的对象必须具有相同的`hashCode()`值

## 3。具有原语的类

让我们考虑一个仅由原始成员变量组成的 Java 类:

```java
public class PrimitiveClass {

    private boolean primitiveBoolean;
    private int primitiveInt;

    // constructor, getters and setters
}
```

我们使用 Eclipse IDE 生成`equals`()和`hashCode`()使用‘源- >生成`hashCode()`和`equals()`。Eclipse 提供了这样一个对话框:

[![eclipse-equals-hascode](img/bed64b7de91a2080c3d51693590fd2d2.png)](/web/20220625215941/https://www.baeldung.com/wp-content/uploads/2016/10/eclipse-equals-hascode.png)

我们可以通过选择“全选”来确保包含所有成员变量。

请注意，插入点:下面列出的选项会影响生成代码的样式。这里，我们不选择这些选项中的任何一个，选择“OK ”,这些方法就被添加到我们的类中:

```java
@Override
public int hashCode() {
    final int prime = 31;
    int result = 1;
    result = prime * result + (primitiveBoolean ? 1231 : 1237);
    result = prime * result + primitiveInt;
    return result;
}

@Override
public boolean equals(Object obj) {
    if (this == obj) return true;
    if (obj == null) return false;
    if (getClass() != obj.getClass()) return false;
    PrimitiveClass other = (PrimitiveClass) obj;
    if (primitiveBoolean != other.primitiveBoolean) return false;
    if (primitiveInt != other.primitiveInt) return false;
    return true;
}
```

生成的`hashCode()`方法从一个质数(31)的声明开始，对原始对象执行各种操作，并根据对象的状态返回结果。

`equals()`首先检查两个对象是否是同一个实例(==)，如果是则返回 true。

接下来，它检查比较对象是否为非空，以及两个对象是否属于同一类，如果不是，则返回 false。

最后，`equals()`检查每个成员变量的相等性，如果其中任何一个不相等，则返回 false。

所以我们可以编写简单的测试:

```java
PrimitiveClass aObject = new PrimitiveClass(false, 2);
PrimitiveClass bObject = new PrimitiveClass(false, 2);
PrimitiveClass dObject = new PrimitiveClass(true, 2);

assertTrue(aObject.equals(bObject) && bObject.equals(aObject));
assertTrue(aObject.hashCode() == bObject.hashCode());

assertFalse(aObject.equals(dObject));
assertFalse(aObject.hashCode() == dObject.hashCode());
```

## 4。具有集合和泛型的类

现在，让我们考虑一个更复杂的具有集合和泛型的 Java 类:

```java
public class ComplexClass {

    private List<?> genericList;
    private Set<Integer> integerSet;

    // constructor, getters and setters
}
```

我们再次使用 Eclipse 的“Source-> Generate”`hashCode()`和`equals()'.` 注意，`hashCode()` 使用`instanceOf`来比较类对象，因为我们在对话框的 Eclipse 选项中选择了“使用‘instance of’来比较类型”。我们得到:

```java
@Override
public int hashCode() {
    final int prime = 31;
    int result = 1;
    result = prime * result + ((genericList == null)
      ? 0 : genericList.hashCode());
    result = prime * result + ((integerSet == null)
      ? 0 : integerSet.hashCode());
    return result;
}

@Override
public boolean equals(Object obj) {
    if (this == obj) return true;
    if (obj == null) return false;
    if (!(obj instanceof ComplexClass)) return false;
    ComplexClass other = (ComplexClass) obj;
    if (genericList == null) {
        if (other.genericList != null)
            return false;
    } else if (!genericList.equals(other.genericList))
        return false;
    if (integerSet == null) {
        if (other.integerSet != null)
            return false;
    } else if (!integerSet.equals(other.integerSet))
        return false;
    return true;
}
```

生成的`hashCode()`方法依赖于`AbstractList.hashCode()`和`AbstractSet.hashCode()`核心 Java 方法。这些函数遍历一个集合，对每一项的`hashCode()`值求和并返回一个结果。

类似地，生成的`equals()`方法使用`AbstractList.equals()`和`AbstractSet.equals()`，它们通过比较集合的字段来比较集合是否相等。

我们可以通过测试一些例子来验证健壮性:

```java
ArrayList<String> strArrayList = new ArrayList<String>();
strArrayList.add("abc");
strArrayList.add("def");
ComplexClass aObject = new ComplexClass(strArrayList, new HashSet<Integer>(45,67));
ComplexClass bObject = new ComplexClass(strArrayList, new HashSet<Integer>(45,67));

ArrayList<String> strArrayListD = new ArrayList<String>();
strArrayListD.add("lmn");
strArrayListD.add("pqr");
ComplexClass dObject = new ComplexClass(strArrayListD, new HashSet<Integer>(45,67));

assertTrue(aObject.equals(bObject) && bObject.equals(aObject));
assertTrue(aObject.hashCode() == bObject.hashCode());

assertFalse(aObject.equals(dObject));
assertFalse(aObject.hashCode() == dObject.hashCode());
```

## 5。继承

让我们考虑使用继承的 Java 类:

```java
public abstract class Shape {
    public abstract double area();

    public abstract double perimeter();
}

public class Rectangle extends Shape {
    private double width;
    private double length;

    @Override
    public double area() {
        return width * length;
    }

    @Override
    public double perimeter() {
        return 2 * (width + length);
    }
    // constructor, getters and setters
}

public class Square extends Rectangle {
    Color color;
    // constructor, getters and setters
}
```

如果我们尝试对`Square`类执行‘源->生成`hashCode()`和`equals()`，Eclipse 会警告我们‘超类‘矩形’没有重新声明`equals()`和`hashCode()`:结果代码可能无法正常工作’。

类似地，当我们试图在`Rectangle`类上生成`hashCode()`和`equals()`时，我们会得到一个关于超类‘Shape’的警告。

日蚀将允许我们不顾警告勇往直前。在`Rectangle`的例子中，它扩展了一个抽象的`Shape`类，该类不能实现`hashCode()`或`equals()`，因为它没有具体的成员变量。对于这种情况，我们可以忽略 Eclipse。

然而，`Square`类从 Rectangle 继承了`width`和`length`成员变量，以及它自己的颜色变量。在`Square`中创建`hashCode()`和`equals()`而不先为`Rectangle`做同样的事情，意味着在`equals()` / `hashCode()`中只使用`color`:

```java
@Override
public int hashCode() {
    final int prime = 31;
    int result = 1;
    result = prime * result + ((color == null) ? 0 : color.hashCode());
    return result;
}
@Override
public boolean equals(Object obj) {
    if (this == obj) return true;
    if (obj == null) return false;
    if (getClass() != obj.getClass()) return false;
    Square other = (Square) obj;
    if (color == null) {
        if (other.color != null)
            return false;
    } else if (!color.equals(other.color))
        return false;
    return true;
}
```

快速测试表明，如果仅仅是`width`不同，那么`Square`的`equals()` / `hashCode()`是不够的，因为`width`不包括在`equals()` / `hashCode()`计算中:

```java
Square aObject = new Square(10, Color.BLUE);     
Square dObject = new Square(20, Color.BLUE);

Assert.assertFalse(aObject.equals(dObject));
Assert.assertFalse(aObject.hashCode() == dObject.hashCode()); 
```

让我们通过使用 Eclipse 为`Rectangle`类生成`equals()` / `hashCode()`来解决这个问题:

```java
@Override
public int hashCode() {
    final int prime = 31;
    int result = 1;
    long temp;
    temp = Double.doubleToLongBits(length);
    result = prime * result + (int) (temp ^ (temp >>> 32));
    temp = Double.doubleToLongBits(width);
    result = prime * result + (int) (temp ^ (temp >>> 32));
    return result;
}

@Override
public boolean equals(Object obj) {
    if (this == obj) return true;
    if (obj == null) return false;
    if (getClass() != obj.getClass()) return false;
    Rectangle other = (Rectangle) obj;
    if (Double.doubleToLongBits(length)
      != Double.doubleToLongBits(other.length)) return false;
    if (Double.doubleToLongBits(width)
      != Double.doubleToLongBits(other.width)) return false;
    return true;
}
```

我们必须在`Square`类中重新生成`equals()` / `hashCode()`，所以`Rectangle`的`equals()` / `hashCode()`被调用。在这一代代码中，我们选择了 Eclipse 对话框中的所有选项，因此我们看到了注释、`instanceOf`比较和`if`块:

```java
@Override
public int hashCode() {
    final int prime = 31;
    int result = super.hashCode();
    result = prime * result + ((color == null) ? 0 : color.hashCode());
    return result;
}

@Override
public boolean equals(Object obj) {
    if (this == obj) {
        return true;
    }
    if (!super.equals(obj)) {
        return false;
    }
    if (!(obj instanceof Square)) {
        return false;
    }
    Square other = (Square) obj;
    if (color == null) {
        if (other.color != null) {
            return false;
       }
    } else if (!color.equals(other.color)) {
        return false;
    }
    return true;
}
```

从上面重新运行我们的测试，我们现在通过了，因为`Square`的`hashCode()` / `equals()`计算正确。

## 6。结论

Eclipse IDE 非常强大，允许自动生成样板代码——getter/setter、各种类型的构造函数、`equals()`和`hashCode()`。

通过理解 Eclipse 正在做什么，我们可以减少花费在这些编码任务上的时间。然而，我们仍然必须小心谨慎，用测试来验证我们的代码，以确保我们已经处理了所有预期的情况。

代码片段一如既往地可以在 GitHub 上找到[。](https://web.archive.org/web/20220625215941/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang)