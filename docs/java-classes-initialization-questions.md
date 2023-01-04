# Java 类结构和初始化面试问题

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-classes-initialization-questions>

[This article is part of a series:](javascript:void(0);)[• Java Collections Interview Questions](/web/20221208143855/https://www.baeldung.com/java-collections-interview-questions)
[• Java Type System Interview Questions](/web/20221208143855/https://www.baeldung.com/java-type-system-interview-questions)
[• Java Concurrency Interview Questions (+ Answers)](/web/20221208143855/https://www.baeldung.com/java-concurrency-interview-questions)
• Java Class Structure and Initialization Interview Questions (current article)[• Java 8 Interview Questions(+ Answers)](/web/20221208143855/https://www.baeldung.com/java-8-interview-questions)
[• Memory Management in Java Interview Questions (+Answers)](/web/20221208143855/https://www.baeldung.com/java-memory-management-interview-questions)
[• Java Generics Interview Questions (+Answers)](/web/20221208143855/https://www.baeldung.com/java-generics-interview-questions)
[• Java Flow Control Interview Questions (+ Answers)](/web/20221208143855/https://www.baeldung.com/java-flow-control-interview-questions)
[• Java Exceptions Interview Questions (+ Answers)](/web/20221208143855/https://www.baeldung.com/java-exceptions-interview-questions)
[• Java Annotations Interview Questions (+ Answers)](/web/20221208143855/https://www.baeldung.com/java-annotations-interview-questions)
[• Top Spring Framework Interview Questions](/web/20221208143855/https://www.baeldung.com/spring-interview-questions)

## 1。简介

类结构和初始化是每个 Java 程序员都应该熟悉的基础。这篇文章为你可能遇到的一些面试问题提供了答案。

### Q1。描述 Final 关键字应用于类、方法、字段或局部变量时的含义。

`final`关键字在应用于不同的语言结构时有多种不同的含义:

*   一个`final`类是一个不能被子类化的类
*   `final`方法是不能在子类中被覆盖的方法
*   一个`final`字段是一个必须在构造器或初始化器块中初始化的字段，并且在那之后不能被修改
*   一个`final`变量是一个只能被赋值(并且必须被赋值)一次的变量，并且在那之后不会被修改

### Q2。什么是默认方法？

在 Java 8 之前，接口只能有抽象方法，即没有主体的方法。从 Java 8 开始，接口方法可以有一个默认的实现。如果实现类不重写此方法，则使用默认实现。这样的方法被适当地标记上一个`default`关键字。

`default`方法的一个突出用例是向现有接口添加方法。如果不将这样的接口方法标记为`default`，那么这个接口的所有现有实现都将中断。添加一个具有`default`实现的方法确保了遗留代码与该接口新版本的二进制兼容性。

一个很好的例子是`Iterator`接口，它允许一个类成为 for-each 循环的目标。这个接口最早出现在 Java 5 中，但是在 Java 8 中它接受了两个额外的方法，`forEach`和`spliterator`。它们被定义为实现的默认方法，因此不会破坏向后兼容性:

```java
public interface Iterable<T> {

    Iterator<T> iterator();

    default void forEach(Consumer<? super T> action) { /* */ }

    default Spliterator<T> spliterator() { /* */ }
}
```

### Q3。什么是静态类成员？

一个类的字段和方法不绑定到一个类的特定实例。相反，它们被绑定到类对象本身。对一个`static`方法的调用或者对一个`static`字段的寻址在编译时被解析，因为与实例方法和字段相反，我们不需要遍历引用并确定我们所引用的实际对象。

### Q4。如果一个类没有任何抽象成员，它可以被声明为抽象的吗？这堂课的目的是什么？

是的，一个类可以被声明为`abstract`，即使它不包含任何`abstract`成员。作为一个抽象类，它不能被实例化，但是它可以作为某个层次结构的根对象，提供对其实现有用的方法。

### Q5。什么是构造函数链接？

构造函数链接是一种简化对象构造的方法，它提供多个构造函数，这些构造函数按顺序相互调用。

最具体的构造函数可以接受所有可能的参数，并可用于最详细的对象配置。一个不太具体的构造函数可以通过为它的一些参数提供默认值来调用更具体的构造函数。在链的顶端，无参数构造函数可以用默认值实例化一个对象。

这里有一个例子，有一个类，它用一定天数内可用的百分比来模拟折扣。如果在使用无参数构造函数时不指定默认值 10%和 2 天，则使用它们:

```java
public class Discount {

    private int percent;

    private int days;

    public Discount() {
        this(10);
    }

    public Discount(int percent) {
        this(percent, 2);
    }

    public Discount(int percent, int days) {
        this.percent = percent;
        this.days = days;
    }

}
```

### Q6。什么是方法的重写和重载？它们有什么不同？

当你用和超类中相同的签名定义一个方法时，方法的覆盖是在子类中完成的。这允许运行时根据调用方法的实际对象类型来选择方法。方法`toString`、`equals`和`hashCode`在子类中经常被覆盖。

方法的重载发生在同一个类中。当您创建名称相同但类型或参数数量不同的方法时，就会发生重载。这允许您根据所提供的参数类型执行特定的代码，而方法的名称保持不变。

这里有一个在`java.io.Writer`抽象类中重载的例子。下面的方法都被命名为`write`，但是其中一个方法接收一个`int`，而另一个方法接收一个`char`数组。

```java
public abstract class Writer {

    public void write(int c) throws IOException {
        // ...
    }

    public void write(char cbuf[]) throws IOException {
        // ...
    }

}
```

### Q7。你能重写一个静态方法吗？

不，你不能。根据定义，只有当方法的实现在运行时由实际实例的类型决定时，才可以重写该方法(这个过程称为动态方法查找)。`static`方法的实现是在编译时使用引用的类型来确定的，所以重写没有多大意义。尽管可以向子类中添加一个与超类中完全相同的签名的`static`方法，但这在技术上并不是重写。

### Q8。什么是不可变类，如何创建一个不可变类？

不可变类的实例在创建后不能更改。我们所说的更改是指通过修改实例的字段值来改变状态。不可变类有很多优点:它们是线程安全的，当你没有可变状态要考虑时，推理它们要容易得多。

要使类不可变，您应该确保以下几点:

*   所有字段都应声明为`private`和`final`；这意味着它们应该在构造函数中被初始化，并且从那以后不再被改变；
*   该类不应有改变字段值的 setters 或其他方法；
*   通过构造函数传递的类的所有字段也应该是不可变的，或者它们的值应该在字段初始化之前被复制(或者我们可以通过保留这些值并修改它们来改变这个类的状态)；
*   该类的方法不应是可重写的；要么所有方法都应该是`final`，要么构造函数应该是`private`，并且只能通过`static`工厂方法调用。

### Q9。如何比较两个枚举值:用`equals()`还是用==？

其实两个都可以用。`enum`值是对象，所以可以和`equals()`进行比较，但是它们也是作为`static`常量在引擎盖下实现的，所以你不妨和`==`进行比较。这主要是代码风格的问题，但是如果你想节省字符空间(并且可能跳过不必要的方法调用)，你应该用`==`来比较枚举。

### Q10。什么是初始化块？什么是静态初始化块？

初始化程序块是类范围内的花括号代码块，在实例创建期间执行。您可以用它来初始化比就地初始化一行程序更复杂的字段。

实际上，编译器只是将这个块复制到每个构造函数中，所以这是从所有构造函数中提取公共代码的好方法。

静态初始化程序块是一个带花括号的代码块，前面有 `static`修饰符。它在类加载期间执行一次，可用于初始化静态字段或一些副作用。

### Q11。什么是标记接口？Java 中有哪些值得注意的标记接口的例子？

标记接口是没有任何方法的接口。它通常由一个类实现或由另一个接口扩展来表示某个属性。标准 Java 库中最广为人知的标记接口如下:

*   `Serializable` 用来明确表示这个类可以序列化；
*   `Cloneable` 允许使用`clone`方法克隆对象(如果没有`Cloneable`接口，这个方法抛出一个`CloneNotSupportedException`)；
*   在 RMI 中使用`Remote`来指定一个可以被远程调用的接口。

### Q12。什么是 Singleton，如何用 Java 实现？

Singleton 是面向对象编程的一种模式。单例类可能只有一个实例，通常是全局可见和可访问的。

在 Java 中有多种方法可以创建一个 singleton。下面是一个最简单的例子，其中有一个就地初始化的`static`字段。初始化是线程安全的，因为`static`字段保证以线程安全的方式初始化。构造函数是`private`，所以外部代码无法创建该类的多个实例。

```java
public class SingletonExample {

    private static SingletonExample instance = new SingletonExample();

    private SingletonExample() {}

    public static SingletonExample getInstance() {
        return instance;
    }
} 
```

但是这种方法有一个严重的缺点——当第一次访问这个类时，实例将被实例化。如果这个类的初始化是一个繁重的操作，我们可能希望推迟到实际需要实例的时候(可能永远不需要)，但同时保持它的线程安全。在这种情况下，我们应该使用一种称为**双重检查锁定**的技术。

### Q13。什么是 Var-Arg？Var-Arg 有什么限制？如何在方法体内使用它？

Var-arg 是方法的可变长度参数。一个方法只能有一个 var-arg，并且它必须在参数列表的最后。它被指定为类型名，后跟省略号和参数名。在方法体中，var-arg 用作指定类型的数组。

这里有一个来自标准库的例子——`Collections.addAll`方法接收一个集合，可变数量的元素，并将所有元素添加到集合中:

```java
public static <T> boolean addAll(
  Collection<? super T> c, T... elements) {
    boolean result = false;
    for (T element : elements)
        result |= c.add(element);
    return result;
}
```

### Q14。你能访问一个超类的重写方法吗？你能以类似的方式访问一个超超类的重写方法吗？

要访问超类的重写方法，可以使用`super`关键字。但是您没有访问超超类的重写方法的类似方法。

作为标准库中的一个例子，`LinkedHashMap`类扩展了`HashMap`并且大部分重用了它的功能，在它的值上添加了一个链表来保持迭代顺序。`LinkedHashMap`重用其超类的`clear`方法，然后清除其链表的头尾引用:

```java
public void clear() {
    super.clear();
    head = tail = null;
} 
```

Next **»**[Java 8 Interview Questions(+ Answers)](/web/20221208143855/https://www.baeldung.com/java-8-interview-questions)**«** Previous[Java Concurrency Interview Questions (+ Answers)](/web/20221208143855/https://www.baeldung.com/java-concurrency-interview-questions)