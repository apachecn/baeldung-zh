# Java 泛型面试问题(+答案)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-generics-interview-questions>

[This article is part of a series:](javascript:void(0);)[• Java Collections Interview Questions](/web/20221208143839/https://www.baeldung.com/java-collections-interview-questions)
[• Java Type System Interview Questions](/web/20221208143839/https://www.baeldung.com/java-type-system-interview-questions)
[• Java Concurrency Interview Questions (+ Answers)](/web/20221208143839/https://www.baeldung.com/java-concurrency-interview-questions)
[• Java Class Structure and Initialization Interview Questions](/web/20221208143839/https://www.baeldung.com/java-classes-initialization-questions)
[• Java 8 Interview Questions(+ Answers)](/web/20221208143839/https://www.baeldung.com/java-8-interview-questions)
[• Memory Management in Java Interview Questions (+Answers)](/web/20221208143839/https://www.baeldung.com/java-memory-management-interview-questions)
• Java Generics Interview Questions (+Answers) (current article)[• Java Flow Control Interview Questions (+ Answers)](/web/20221208143839/https://www.baeldung.com/java-flow-control-interview-questions)
[• Java Exceptions Interview Questions (+ Answers)](/web/20221208143839/https://www.baeldung.com/java-exceptions-interview-questions)
[• Java Annotations Interview Questions (+ Answers)](/web/20221208143839/https://www.baeldung.com/java-annotations-interview-questions)
[• Top Spring Framework Interview Questions](/web/20221208143839/https://www.baeldung.com/spring-interview-questions)

## 1。简介

在本文中，我们将浏览一些 Java 泛型面试问题和答案的例子。

泛型是 Java 中的一个核心概念，最初是在 Java 5 中引入的。因此，几乎所有的 Java 代码库都会使用它们，几乎可以保证开发人员会在某个时候碰到它们。这就是为什么正确理解它们是至关重要的，也是为什么它们更有可能在面试过程中被问到。

## 2。问题

### Q1。什么是泛型类型参数？

`Type` 是一个`class`或`interface`的名字。顾名思义，**泛型类型参数是指`type`可以用作类、方法或接口声明中的参数。**

让我们从一个简单的例子开始，一个没有泛型的例子，来说明这一点:

```
public interface Consumer {
    public void consume(String parameter)
}
```

在这种情况下，`consume()`方法的方法参数类型是`String.` ，它不是参数化的，也是不可配置的。

现在让我们用一个我们称之为`T.` 的泛型类型来替换我们的`String` 类型。按照惯例，它是这样命名的:

```
public interface Consumer<T> {
    public void consume(T parameter)
}
```

当我们实现我们的消费者时，我们可以提供我们希望它消费的`type` 作为参数。这是一个泛型类型参数:

```
public class IntegerConsumer implements Consumer<Integer> {
    public void consume(Integer parameter)
}
```

在这种情况下，现在我们可以消耗整数。我们可以把这个`type`换成我们需要的任何东西。

### Q2。使用泛型有什么好处？

使用泛型的一个优点是避免强制转换并提供类型安全。这在处理集合时特别有用。让我们来演示一下:

```
List list = new ArrayList();
list.add("foo");
Object o = list.get(0);
String foo = (String) o;
```

在我们的例子中，列表中的元素类型对于编译器来说是未知的。这意味着唯一可以保证的是它是一个对象。因此，当我们检索元素时，我们得到的是一个`Object` 。作为代码的作者，我们知道它是一个`String,`,但是我们必须将我们的对象转换成一个来明确地解决这个问题。这产生了许多噪音和样板文件。

接下来，如果我们开始考虑人为错误的空间，铸造问题会变得更糟。如果我们的列表中意外出现了一个`Integer`会怎么样？

```
list.add(1)
Object o = list.get(0);
String foo = (String) o;
```

在这种情况下，我们将在运行时得到一个`[ClassCastException](https://web.archive.org/web/20221208143839/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ClassCastException.html)` ，因为`Integer`不能被强制转换为`String.`

现在，让我们试着重复我们自己，这次使用泛型:

```
List<String> list = new ArrayList<>();
list.add("foo");
String o = list.get(0);    // No cast
Integer foo = list.get(0); // Compilation error
```

正如我们所看到的，**通过使用泛型，我们有了一个编译类型检查，它阻止了`ClassCastExceptions`并消除了造型的需要。**

**另一个好处是避免代码重复**。如果没有泛型，我们必须复制和粘贴相同的代码，但用于不同的类型。有了泛型，我们就不必这么做了。我们甚至可以实现适用于泛型类型的算法。

### Q3。什么是类型擦除？

重要的是要认识到泛型类型信息只对编译器可用，对 JVM 不可用。换句话说**，类型擦除意味着泛型类型信息在运行时对 JVM 不可用，只有编译时**。

选择主要实现背后的原因很简单——保持与旧版本 Java 的向后兼容性。当一个泛型代码被编译成字节码时，就好像这个泛型从来没有存在过一样。这意味着编译将:

1.  用对象替换泛型类型
2.  用第一个绑定类替换绑定类型(在后面的问题中会详细讨论)
3.  在检索泛型对象时插入等效的强制转换。

理解类型擦除很重要。否则，开发人员可能会感到困惑，认为他们能够在运行时获得该类型:

```
public foo(Consumer<T> consumer) {
   Type type = consumer.getGenericTypeParameter()
}
```

上面的例子是一个伪代码，相当于没有类型擦除时的样子，但不幸的是，这是不可能的。同样，**泛型类型信息在运行时不可用。**

### Q4。如果在实例化对象时省略了泛型类型，代码还能编译吗？

由于泛型在 Java 5 之前并不存在，所以有可能根本不使用它们。例如，泛型被改造成大多数标准 Java 类，比如集合。如果我们看看问题一的列表，我们会发现已经有一个省略泛型类型的例子了:

```
List list = new ArrayList();
```

尽管能够编译，仍然可能会有来自编译器的警告。这是因为我们正在失去从使用泛型中获得的额外的编译时检查。

要记住的一点是**虽然向后兼容性和类型擦除使得省略泛型类型成为可能，但这是一种不好的做法。**

### Q5。泛型方法和泛型类型有什么不同？

**泛型方法是将类型参数引入到一个方法中，** **活在那个方法的范围内。**让我们用一个例子来试试这个:

```
public static <T> T returnType(T argument) { 
    return argument; 
}
```

我们使用了静态方法，但是如果我们愿意，也可以使用非静态方法。通过利用类型推断(在下一个问题中涉及)，我们可以像调用任何普通方法一样调用它，而不必在这样做时指定任何类型参数。

### Q6。什么是类型推理？

类型推断是指编译器可以查看方法参数的类型来推断泛型类型。例如，如果我们将`T` 传递给一个返回`T,` 的方法，那么编译器可以计算出返回类型。让我们通过调用上一个问题中的泛型方法来尝试一下:

```
Integer inferredInteger = returnType(1);
String inferredString = returnType("String");
```

正如我们所见，不需要强制转换，也不需要传入任何泛型类型参数。参数类型仅推断返回类型。

### Q7。什么是有界类型参数？

到目前为止，我们所有的问题都涵盖了泛型类型的参数，它们是无限的。这意味着我们的泛型类型参数可以是我们想要的任何类型。

当我们使用有界参数时，我们限制了可以用作泛型类型实参的类型。

例如，假设我们想强制我们的泛型类型总是 animal 的子类:

```
public abstract class Cage<T extends Animal> {
    abstract void addAnimal(T animal)
}
```

通过使用扩展`,` ,我们迫使`T` 成为动物`.` 的一个子类，然后我们可以拥有一个猫的笼子:

```
Cage<Cat> catCage;
```

但我们不可能有一个对象的笼子，因为对象不是动物的子类:

```
Cage<Object> objectCage; // Compilation error
```

这样做的一个好处是，animal 的所有方法都可供编译器使用。我们知道我们的类型扩展了它，所以我们可以写一个通用算法，可以在任何动物身上运行。这意味着我们不必为不同的动物子类重复我们的方法:

```
public void firstAnimalJump() {
    T animal = animals.get(0);
    animal.jump();
}
```

### Q8。有可能声明一个多重有界类型参数吗？

为我们的泛型类型声明多个界限是可能的。在前面的示例中，我们指定了一个界限，但是如果我们愿意，也可以指定更多界限:

```
public abstract class Cage<T extends Animal & Comparable>
```

在我们的例子中，animal 是一个类，comparable 是一个接口。现在，我们的类型必须尊重这两个上限。如果我们的类型是 animal 的子类，但是没有实现 comparable，那么代码就不会编译。同样值得记住的是，如果一个上界是一个类，它必须是第一个参数。

### Q9。什么是通配符类型？

**通配符类型代表未知的`type`** 。它是带着一个问号引爆的，如下:

```
public static void consumeListOfWildcardType(List<?> list)
```

这里，我们指定了一个可以是任何`type`的列表。我们可以把任何东西的列表传递给这个方法。

### Q10。什么是上限通配符？

**上限通配符是当通配符类型从具体类型**继承时。这在处理集合和继承时特别有用。

让我们尝试用一个存储动物的 farm 类来演示这一点，首先不使用通配符类型:

```
public class Farm {
  private List<Animal> animals;

  public void addAnimals(Collection<Animal> newAnimals) {
    animals.addAll(newAnimals);
  }
}
```

如果我们有动物`,` 的多个子类，比如猫和狗`,` ，我们可能会做出错误的假设，认为我们可以将它们都添加到我们的农场中:

```
farm.addAnimals(cats); // Compilation error
farm.addAnimals(dogs); // Compilation error
```

这是因为编译器期望一个具体类型 animal `,` 的集合，而不是它的子类。

现在，让我们为添加动物方法引入一个上限通配符:

```
public void addAnimals(Collection<? extends Animal> newAnimals)
```

现在如果我们再试一次，我们的代码将会编译。这是因为我们现在告诉编译器接受任何动物亚型的集合。

### Q11。什么是无界通配符？

无界通配符是没有上限或下限的通配符，可以表示任何类型。

了解通配符类型与 object 不是同义词也很重要。这是因为通配符可以是任何类型，而对象类型是特定的对象(不能是对象的子类)。让我们用一个例子来说明这一点:

```
List<?> wildcardList = new ArrayList<String>(); 
List<Object> objectList = new ArrayList<String>(); // Compilation error
```

同样，第二行不能编译的原因是需要一个对象列表，而不是一个字符串列表。第一行编译，因为任何未知类型的列表都是可以接受的。

### Q12。什么是下界通配符？

下界通配符是指我们通过使用`super`关键字来提供下界，而不是提供上界。换句话说，**一个较低的有界通配符意味着我们强制该类型成为我们的有界类型**的超类。让我们用一个例子来试试这个:

```
public static void addDogs(List<? super Animal> list) {
   list.add(new Dog("tom"))
}
```

通过使用`super,` ,我们可以在对象列表上调用 addDogs:

```
ArrayList<Object> objects = new ArrayList<>();
addDogs(objects);
```

这是有意义的，因为对象是动物的超类。如果我们不使用下界通配符，代码将无法编译，因为对象列表不是动物列表。

如果我们仔细想想，我们不可能把狗添加到任何动物子类的列表中，比如猫，甚至狗。只是动物的一个超类。例如，这不会编译:

```
ArrayList<Cat> objects = new ArrayList<>();
addDogs(objects);
```

### Q13。什么时候你会选择使用下限类型还是上限类型？

在处理集合时，在上限或下限通配符之间进行选择的一个常见规则是 PECS。PECS 代表**生产者延伸，消费者超。**

通过使用一些标准的 Java 接口和类，可以很容易地演示这一点。

`Producer extends`仅仅意味着如果你正在创建一个泛型类型的生产者，那么使用`extends`关键字。让我们试着将这一原则应用于一个系列，看看它为什么有意义:

```
public static void makeLotsOfNoise(List<? extends Animal> animals) {
    animals.forEach(Animal::makeNoise);   
}
```

在这里，我们想对集合中的每种动物调用`makeNoise()` 。这意味着我们的集合是一个生产者`,` ,因为我们所做的只是让它返回动物给我们做手术。如果我们去掉了`extends`，我们将无法传递猫`,` 狗或任何其他动物子类的列表。通过应用生产者扩展原则，我们拥有了最大的灵活性。

`Consumer super` 的意思与`producer extends.` 相反，它的意思是，如果我们正在处理消耗元素的东西，那么我们应该使用`super` 关键字。我们可以通过重复前面的例子来证明这一点:

```
public static void addCats(List<? super Animal> animals) {
    animals.add(new Cat());   
}
```

我们只是在增加我们的动物名单，所以我们的动物名单是一个消费者。这就是我们使用`super`关键字的原因。这意味着我们可以传入任何动物超类的列表，但不能传入子类。例如，如果我们试图传入一个狗或猫的列表，那么代码就不会编译。

最后要考虑的是，如果一个集合既是消费者又是生产者，该怎么办。这方面的一个例子可能是既添加元素又移除元素的集合。在这种情况下，应该使用无界通配符。

### Q14。有没有一些情况下泛型类型信息在运行时是可用的？

在一种情况下，泛型类型在运行时是可用的。当泛型类型是类签名的一部分时，如下所示:

```
public class CatCage implements Cage<Cat>
```

通过使用反射，我们得到这个类型参数:

```
(Class<T>) ((ParameterizedType) getClass()
  .getGenericSuperclass()).getActualTypeArguments()[0];
```

这段代码有些脆弱。例如，它依赖于直接超类中定义的类型参数。但是，它证明了 JVM 确实有这种类型的信息。

Next **»**[Java Flow Control Interview Questions (+ Answers)](/web/20221208143839/https://www.baeldung.com/java-flow-control-interview-questions)**«** Previous[Memory Management in Java Interview Questions (+Answers)](/web/20221208143839/https://www.baeldung.com/java-memory-management-interview-questions)