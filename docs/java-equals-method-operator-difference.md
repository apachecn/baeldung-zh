# Java 中==和 equals()的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-equals-method-operator-difference>

## 1.概观

在本教程中，我们将描述 Java 中的两种基本的相等检查——引用相等和值相等。我们将比较它们，展示例子，并强调它们之间的主要区别。

此外，我们将关注`null`检查，并理解为什么我们在处理对象时应该使用引用相等而不是值相等。

## 2.参考等式

我们将从理解引用比较开始，引用比较由等号运算符(`==`)表示。当两个引用指向内存中的同一个对象时，引用相等。

### 2.1.具有基本类型的相等运算符

我们知道 Java 中的[原语类型是简单的非类原始值。当我们对基本类型使用等式运算符时，我们只是比较它们的值:](/web/20221208143839/https://www.baeldung.com/java-primitives)

```java
int a = 10;
int b = 15;
assertFalse(a == b);

int c = 10;
assertTrue(a == c);

int d = a;
assertTrue(a == d);
```

如上所示，**相等和引用检查对原语**的作用是相同的。当我们用相同的值初始化一个新的原语时，检查返回`true.`此外，如果我们将原始值重新分配给新变量并比较它，操作符返回相同的结果。

现在让我们执行`null`检查:

```java
int e = null; // compilation error
assertFalse(a == null); // compilation error
assertFalse(10 == null); // compilation error
```

Java 禁止将`null`赋给原语。一般来说，**我们不能用等式操作符对原始变量**或值执行任何`null`检查。

### 2.2.对象类型的相等运算符

至于 Java 、**中的[对象类型，相等运算符只执行参照相等比较](/web/20221208143839/https://www.baeldung.com/java-classes-objects)**，忽略对象值。在我们实现测试之前，让我们创建一个简单的定制类:

```java
public class Person {
    private String name;
    private int age;

    // constructor, getters, setters...
}
```

现在，让我们初始化一些类对象并检查等式运算符的结果:

```java
Person a = new Person("Bob", 20);
Person b = new Person("Mike", 40);
assertFalse(a == b);

Person c = new Person("Bob", 20);
assertFalse(a == c);

Person d = a;
assertTrue(a == d);
```

结果和以前大不相同。第二次检查返回`false`，而我们已经得到了原语的`true`。正如我们前面提到的，相等运算符在比较时会忽略对象的内部值。它只**检查两个变量是否引用同一个内存地址**。

与原语不同，我们可以在处理对象时使用`null`:

`assertFalse(a == null);
Person e = null;
assertTrue(e == null);`

**通过使用等式运算符和比较`null,`，我们检查分配给变量的对象是否已经初始化**。

## 3.价值平等

现在让我们关注值相等测试。当两个独立的对象碰巧具有相同的值或状态时，值相等。

这种比较值与[`Object's equals()`方法](/web/20221208143839/https://www.baeldung.com/java-comparing-objects#equals-instance)密切相关。像以前一样，让我们比较它与原语和对象类型的使用，看看关键的区别。

### 3.1.`equals()`具有原始类型的方法

正如我们所知，原语是只有一个值的基本类型，不实现任何方法。因此，**不可能使用原语**直接调用`equals()`方法:

`int a = 10;
assertTrue(a.equals(10)); // compilation error`

然而，由于每个**原语都有自己的[包装类](/web/20221208143839/https://www.baeldung.com/java-wrapper-classes)** ，我们可以使用`boxing mechanism`将其转换为对象表示。然后，我们可以像使用对象类型一样轻松地调用`equals()`方法:

```java
int a = 10;
Integer b = a;

assertTrue(b.equals(10));
```

### 3.2.`equals()`方法与对象类型

让我们回到我们的`Person`班。为了让`equals() `方法正确工作，[我们需要通过考虑类中包含的字段来覆盖自定义类中的方法](/web/20221208143839/https://www.baeldung.com/java-eclipse-equals-and-hashcode):

```java
public class Person {
    // other fields and methods omitted

    @Override
    public boolean equals(Object o) {
        if (this == o) 
            return true;
        if (o == null || getClass() != o.getClass()) 
            return false;
        Person person = (Person) o;
        return age == person.age && Objects.equals(name, person.name);
    }
}
```

首先，`equals()`方法在给定值有相同引用的情况下返回`true`，由引用操作符检查。如果没有，我们开始平等测试。

此外，我们测试两个值的`Class` 对象的相等性。如果它们不同，我们返回`false`。否则，我们继续检查是否相等。最后，我们返回分别比较每个属性的组合结果。

现在，让我们修改前面的测试并检查结果:

```java
Person a = new Person("Bob", 20);
Person b = new Person("Mike", 40);
assertFalse(a.equals(b));

Person c = new Person("Bob", 20);
assertTrue(a.equals(c));

Person d = a;
assertTrue(a.equals(d));
```

正如我们所看到的，第二次检查返回`true`，而不是引用相等。我们覆盖的`equals()`方法比较对象的内部值。

**如果我们不覆盖`equals()`方法，就会使用父类`Object `的方法。由于`Object.equals()`方法只做引用相等性检查，当比较`Person `对象时，行为可能不是我们所期望的。**

虽然我们没有在上面展示`hashCode()`方法，但我们应该注意，每当我们覆盖`equals()`方法 [时，覆盖它是很重要的，以确保这些方法](/web/20221208143839/https://www.baeldung.com/java-equals-hashcode-contracts)之间的一致性。

## 4.零相等

最后，让我们检查一下`equals()`方法如何处理`null` 值:

```java
Person a = new Person("Bob", 20);
Person e = null;
assertFalse(a.equals(e));
assertThrows(NullPointerException.class, () -> e.equals(a));
```

当我们使用`equals()`方法对另一个对象进行检查时，根据这些变量的顺序，我们会得到两种不同的结果。最后一条语句抛出了一个异常，因为我们在`null`引用上调用了`equals()`方法。要修复最后一个语句，我们应该首先调用等式运算符 check:

```java
assertFalse(e != null && e.equals(a));
```

现在，条件的左边返回`false`，使得整个语句等于`false`，防止`NullPointerException`被抛出。因此，我们必须记住**首先检查我们正在调用的`equals()`方法上的值不是`null`** ，否则，它会导致令人讨厌的 bug。

此外，从 Java 7 开始，我们可以使用一个空安全的 [`Objects#equals()`](https://web.archive.org/web/20221208143839/https://docs.oracle.com/en/java/javase/16/docs/api/java.base/java/util/Objects.html#equals(java.lang.Object,java.lang.Object)) `static`方法来执行等式检查:

```java
assertFalse(Objects.equals(e, a));
assertTrue(Objects.equals(null, e));
```

这个帮助器方法执行额外的检查以防止抛出`NullPointerException`，当两个参数都是`null`时返回`true`。

## 5.结论

在本文中，我们讨论了针对原语和对象值的引用相等和值相等检查。

为了测试引用的相等性，我们使用了`==`操作符。该操作符对原始值和对象的作用略有不同。**当我们对原语使用等式运算符时，它会比较值。另一方面，当我们对对象使用它时，它检查内存引用。**通过将其与一个`null`值进行比较，我们简单地检查对象是否在内存中初始化。

**为了在 Java 中执行值相等测试，我们使用从`Object`继承的`equals()`方法。基元是简单的非类值，因此如果不进行包装，就不能调用该方法。**

我们还需要记住只对实例化的对象调用`equals()`方法。否则，将引发异常。为了防止这种情况，如果我们怀疑一个`null` 值，我们应该用`==`操作符检查这个值。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20221208143839/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-5)