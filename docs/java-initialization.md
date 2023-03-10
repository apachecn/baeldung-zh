# 用 Java 创建对象的指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-initialization>

## 1。概述

简单地说，在我们能够在 JVM 上使用一个对象之前，它必须被初始化。

在本教程中，我们将研究初始化基本类型和对象的各种方法。

## 2。声明与初始化

让我们首先确保我们在同一页上。

**声明是定义变量**及其类型和名称的过程。

这里我们声明了`id`变量:

```java
int id;
```

**另一方面，初始化就是分配一个值:**

```java
id = 1;
```

为了演示，我们将创建一个具有`name`和`id`属性的`User` 类:

```java
public class User {
    private String name;
    private int id;

    // standard constructor, getters, setters,
}
```

接下来，我们将看到，根据我们要初始化的字段类型，初始化的工作方式会有所不同。

## 3。对象与原语

Java 提供了两种类型的数据表示:基本类型和引用类型。在这一节中，我们将讨论两者在初始化方面的区别。

Java 有八种内置的数据类型，称为 Java 原语类型；这种类型的变量直接保存它们的值。

引用类型保存对对象(类的实例)的引用。不像原始类型在分配变量的内存中保存它们的值，引用不保存它们所引用的对象的值。

相反，**引用通过存储对象所在的内存地址来指向对象。**

注意 Java 不允许我们发现物理内存地址是什么。相反，我们只能用引用来指代对象。

让我们看一个在我们的`User`类之外声明和初始化引用类型的例子:

```java
@Test
public void whenIntializedWithNew_thenInstanceIsNotNull() {
    User user = new User();

    assertThat(user).isNotNull();
}
```

正如我们所看到的，通过使用负责创建新的`User` 对象的关键字`new,` ，可以将一个引用分配给一个新的对象。

让我们继续学习更多关于对象创建的知识。

## 4。创建对象

与基本体不同，对象的创建稍微复杂一些。这是因为我们不仅仅是将值添加到字段中；相反，我们使用关键字`new`触发初始化。这反过来调用一个构造函数并初始化内存中的对象。

让我们更详细地讨论构造函数和关键字`new`。

`new` 关键字是**负责通过构造函数为新对象分配内存。**

**构造函数通常用于初始化代表被创建对象**主要属性的实例变量。

如果我们不显式地提供一个构造函数，编译器会创建一个没有参数的默认构造函数，只是为对象分配内存。

**一个类可以有很多构造函数，只要它们的参数列表不同(`overload` )** 。每个不调用同一个类中另一个构造函数的构造函数都有一个对其父构造函数的调用，无论它是显式编写的还是由编译器通过`super()`插入的。

让我们给我们的`User`类添加一个构造函数:

```java
public User(String name, int id) {
    this.name = name;
    this.id = id;
}
```

现在我们可以使用我们的构造函数创建一个`User`对象，并为其属性设置初始值:

```java
User user = new User("Alice", 1);
```

## 5。可变范围

在接下来的几节中，我们将看看 Java 中变量可以存在的不同类型的作用域，以及这如何影响初始化过程。

### 5.1。实例和类变量

实例和类变量不需要我们去初始化它们。一旦我们声明了这些变量，它们就会被赋予一个默认值:

[![init1](img/a0680042e1fcb8b9ae80ea3eada58dd8.png)](/web/20221208143921/https://www.baeldung.com/wp-content/uploads/2017/12/init1.png)

现在让我们尝试定义一些与实例和类相关的变量，并测试它们是否有默认值:

```java
@Test
public void whenValuesAreNotInitialized_thenUserNameAndIdReturnDefault() {
    User user = new User();

    assertThat(user.getName()).isNull();
    assertThat(user.getId() == 0);
}
```

### 5.2。局部变量

在使用之前，局部变量必须被初始化，因为它们没有默认值，编译器不允许我们使用未初始化的值。

例如，下面的代码会生成一个编译器错误:

```java
public void print(){
    int i;
    System.out.println(i);
}
```

## 6。`Final`关键词

应用于字段的关键字`final`意味着字段的值在初始化后不能再更改。这样，我们就可以在 Java 中定义常量了。

让我们给我们的`User`类添加一个常量:

```java
private static final int YEAR = 2000;
```

常量必须在声明时或在构造函数中初始化。

## 7。**Java 中的初始值**

在 Java 中，**初始化器是一个没有关联名称或数据类型**的代码块，它被放置在任何方法、构造函数或另一个代码块之外。

Java 提供了两种类型的初始化器，静态初始化器和实例初始化器。让我们看看如何使用它们。

### 7.1。实例初始化器

我们可以用这些来初始化实例变量。

为了演示，我们将使用我们的`User`类中的实例初始化器为用户`id`提供一个值:

```java
{
    id = 0;
}
```

### 7.2。静态初始化块

静态初始化器或静态块是用于初始化`static`字段的代码块。换句话说，这是一个简单的初始化器，标有关键字`static:`

```java
private static String forum;
static {
    forum = "Java";
}
```

## 8。初始化顺序

当编写初始化不同类型字段的代码时，我们必须注意初始化的顺序。

在 Java 中，初始化语句的顺序如下:

*   静态变量和静态初始值设定项
*   实例变量和实例初始值设定项
*   构造器

## 9。对象生命周期

现在我们已经学习了如何声明和初始化对象，让我们来看看对象不使用时会发生什么。

与我们必须担心对象破坏的其他语言不同，Java 通过其垃圾收集器来处理过时的对象。

Java 中的所有对象都存储在我们程序的堆内存中。事实上，堆代表了为我们的 Java 应用程序分配的大量未使用的内存。

另一方面，**垃圾收集器是一个 Java 程序，它通过删除不再可达的对象来负责自动内存管理**。

对于变得不可访问的 Java 对象，它必须遇到以下情况之一:

*   该对象不再有任何指向它的引用。
*   所有指向该对象的引用都超出了范围。

总之，一个对象首先是从一个类中创建的，通常使用关键字`new.` ，然后对象就有了自己的生命，并为我们提供了对它的方法和字段的访问。

最后，当不再需要它时，垃圾收集器会销毁它。

## 10。创建对象的其他方法

在这一节中，我们将简要地看一下创建对象的**方法，而不是`new `关键字，并学习如何应用它们，特别是反射、克隆和序列化**。

反射是一种我们可以用来在运行时检查类、字段和方法的机制。这里有一个使用反射创建我们的`User`对象的例子:

```java
@Test
public void whenInitializedWithReflection_thenInstanceIsNotNull() 
  throws Exception {
    User user = User.class.getConstructor(String.class, int.class)
      .newInstance("Alice", 2);

    assertThat(user).isNotNull();
}
```

在这种情况下，我们使用反射来查找和调用`User`类的构造函数。

下一种方法，**克隆，是一种创建一个对象的精确副本的方法。**为此，我们的`User`类必须实现`Cloneable`接口:

```java
public class User implements Cloneable { //... }
```

现在我们可以使用`clone()`方法创建一个新的`clonedUser` 对象，它与`user`对象具有相同的属性值:

```java
@Test
public void whenCopiedWithClone_thenExactMatchIsCreated() 
  throws CloneNotSupportedException {
    User user = new User("Alice", 3);
    User clonedUser = (User) user.clone();

    assertThat(clonedUser).isEqualTo(user);
}
```

**我们也可以使用`[sun.misc.Unsafe](/web/20221208143921/https://www.baeldung.com/java-unsafe)`类为一个对象分配内存，而不需要调用构造函数:**

```java
User u = (User) unsafeInstance.allocateInstance(User.class);
```

## 11。结论

在本文中，我们讨论了 Java 中的字段初始化。然后我们研究了 Java 中不同的数据类型以及如何使用它们。我们还探索了用 Java 创建对象的几种方法。

这篇文章的完整实现可以在 Github 上找到[。](https://web.archive.org/web/20221208143921/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-syntax)