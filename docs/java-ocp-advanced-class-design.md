# OCP 认证-高级 Java 类设计

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-ocp-advanced-class-design>

## 1.概观

在本教程中，我们将讨论 OCP 认证的高级 Java 类设计目标。

## 2.OCP Java 认证

[OCP 认证](https://web.archive.org/web/20221205143313/https://education.oracle.com/oracle-certified-professional-java-se-8-programmer/trackp_357)是对[亚奥理事会认证](https://web.archive.org/web/20221205143313/https://education.oracle.com/oracle-certified-associate-java-se-8-programmer/trackp_333)的升级，但遵循相同的选择题格式。然而，它包含了诸如并发性、泛型和 NIO 等高级主题。

在本教程中，我们将关注考试的高级 Java 类设计目标。事实上，我们将要讨论的一些主题与 OCA 考试的 Java 类设计目标重叠。但是，与此同时， **OCP 也包含了关于高级主题的问题，比如内部类、枚举类型和 lambdas** 。

以下每个部分都致力于考试的一个目标。

## 3.开发使用抽象类和方法的代码

第一个考试目标是`[abstract](/web/20221205143313/https://www.baeldung.com/java-abstract-class) `类和方法的使用。在 Java 中，我们使用`abstract `类在具体的子类之间共享变量和方法。

### 考试提示 3.1:带有`abstract `类的访问修饰符不正确

我们必须在关于`abstract `类和方法的问题中寻找访问修饰符。

例如，尝试解决以下问题:

```java
package animal;
public abstract class Animal {

    abstract boolean canFly();
}

package horse;
import animal.Animal;

public class Horse extends Animal {

    @Override
    boolean canFly() {
        return false;
    }

    public static void main(String[] args) {

        System.out.println(new Horse().canFly());
    }    
}
```

```java
Which of the following is true?
A. The output is false
B. Compilation fails on Line 10
C. Compilation fails on Line 12
D. None of the above
```

值得注意的是，`abstract `方法有一个默认的访问修饰符，由于两个类在**不同的包中，我们不能在`Horse `类**中访问它。因此，正确答案是(B)。

### 考试提示 3.2:类或方法中的语法错误

有些问题要求我们检查给定代码中的错误语法。有了`abstract `类，我们很容易错过这样的错误。

例如，尝试解决以下问题:

```java
public abstract class Animal {

    protected abstract boolean canFly() {
    }

    public abstract void eat() {
        System.out.println("Eat...");
    }
}

public class Amphibian extends Animal {
    @Override
    protected boolean canFly() {
        return false;
    }

    @Override
    public void eat() {

    }

    public abstract boolean swim();
}

public class Frog extends Amphibian {
}
```

```java
Which are true? (Choose all that apply.)
A. Compilation error on line 3
B. Compilation error on line 6
C. Compilation error on line 11
D. Compilation error on line 13
E. Compilation error on line 22
```

这里需要记住的是 **`abstract `方法不能有方法体**。另外， **`abstract `方法不能存在于非`abstract`类**中。所以，(A)、(B)、(C)都是正确答案。

### 考试提示 3.3:缺少`abstract `方法的实现

寻找没有一个`abstract `方法的具体实现的非`abstract`子类。

例如，尝试解决以下问题:

```java
public abstract class Animal {

    protected abstract boolean canFly();

    public abstract void eat();
}

public abstract class Amphibian extends Animal {

    @Override
    public void eat() {
        System.out.println("Eat...");
    }

    public abstract boolean swim();
}

public class Frog extends Amphibian {

    @Override
    protected boolean swim() {
        return false;
    }

}
```

```java
Which are true? (Choose all that apply)
A. Compilation error on line 8
B. Compilation error on line 11
C. Compilation error on line 18
D. Compilation error on line 21
E. No compilation error
```

`Frog`类**没有实现`canFly()`方法**，并且**降低了`swim() `方法**的可见性。因此，(C)和(D)是正确的。

尽管`Amphibian `没有实现`canFly(), `，但它被声明为一个`abstract `类，这就是为什么(A)是不正确的。

### 考试提示 3.4:使用`private`、`final,`或`static`与`abstract `关键字

**`abstract `关键字不能与`static`、`private,`或`final`关键字**组合。因此，不允许使用以下任何语句:

```java
public final abstract class Animal {
}

public abstract class Animal {

    public final abstract void eat();
}

public abstract class Animal {

    private abstract void eat();
}
```

任何这样的声明都会导致编译错误。

## 4.开发使用`final`关键字的代码

Java 中的`[final](/web/20221205143313/https://www.baeldung.com/java-final) `关键字允许我们用一个常量值来声明变量。此外，它还允许我们声明不能扩展或覆盖的类和方法。

### 考试提示 4.1:被覆盖的`final` 类或方法

寻找被声明为`final, `并在子类中被覆盖的方法。

例如，尝试解决以下问题:

```java
public abstract class Animal {

    public final void eat() {
        System.out.println("Eat...");
    }
}

public class Horse extends Animal {

    public void eat() {
        System.out.println("Eat Grass");
    }

    public static void main(String[] args) {
        Animal animal = new Horse();
        animal.eat();
    }
}
```

```java
What is the output?
A. Eat...
B. Eat Grass
C. The code will not compile because of line 3
D. The code will not compile because of line 8
E. The code will not compile because of line 10
```

因为`eat()` 在`Animal `类中被声明为 **`final`，所以我们不能在`Horse `类**中覆盖它。因此，(E)是正确答案。

另外，在方法的参数中寻找`final` 变量。如果给这样的变量赋一个新值，就会导致编译错误。

## 5.内部类

关于[内部类](/web/20221205143313/https://www.baeldung.com/java-nested-classes)的问题通常不像其他题目那么直接。考试中有许多关于主题**的问题，如泛型、集合和使用内部类语法的并发性**，因此我们很难理解问题的意图。

### 考试提示 5.1:非`static`内部类的不正确实例化

实例化非内部类的唯一方法是通过外部类的实例。

例如，尝试解决以下问题:

```java
public class Animal {

    class EatingHabbits {
    }

    private EatingHabbits eatingHabbits() {
        return new EatingHabbits();
    }
}

public class Zookeeper {

    public static void main(String[] args) {
        Zookeeper zookeeper = new Zookeeper();
        zookeeper.feed();
    }

    private void feed() {
        EatingHabbits habbits = new EatingHabbits();
        Animal animal = new Animal();
        Animal.EatingHabbits habbits1 = animal.eatingHabbits();
    }
}
```

```java
What is the result? (Choose all that apply.)
A. Compilation error on line 7
B. Compilation error on line 19
C. Compilation error on line 21
D. No compilation error
```

因为在第 19 行，我们试图在没有外部类的对象的情况下实例化内部类，(B)是正确的答案。

### 考试提示 5.2:在内部类中不正确使用`this` 关键字

查找内部类中`this `关键字的不正确使用:

```java
public class Animal {
    private int age = 10;

    public class EatingHabbits {
        private int numOfTimes = 5;

        public void print() {
            System.out.println("The value of numOfTimes " + this.numOfTimes);
            System.out.println("The value of age " + this.age);
            System.out.println("The value of age " + Animal.this.age);
        }
    }

    public static void main(String[] args) {
        Animal.EatingHabbits habbits = new Animal().new EatingHabbits();
        habbits.print();
    }
}
```

由于 **`this `只能用来访问当前正在执行的对象**，第 9 行将导致编译错误。因此，我们必须密切观察内部类中`this `的使用。

### 考试提示 5.3:局部内部类中的非`final `变量

方法局部类不能访问局部变量，除非它被声明为`final `或者它的值在内部类中保持不变。

例如，尝试解决以下问题:

```java
public class Animal {
    private int age = 10;

    public void printAge() {
        String message = "The age is ";
        class PrintUtility {
            void print() {
                System.out.println(message + age);
            }
        }

        PrintUtility utility = new PrintUtility();
        utility.print();
    }

    public static void main(String[] args) {
        new Animal().printAge();
    }
}
```

```java
What is the result of the following code?

A. The age is 0
B. The age is 10
C. Line 8 generates a compiler error
D. Line 12 generates a compiler error
E. An exception is thrown
```

由于**我们从未更新过`message `字段，所以它实际上是`final`** 。因此，(B)是正确的答案。

### 考试提示 5.4: L **局部内部类不能标记为`private, public, protected,` 或** `**static**`

同样的规则也适用于局部内部类和局部变量。因此，我们必须寻找任何违反这些约束的问题。

此外，任何在`static`方法中声明的局部类只能访问封闭类的`static `成员。

### 考试提示 5.5:`static `内部类中的非`static `成员变量

嵌套类**不能访问外部类的实例变量或非`static`方法。**

因此，注意那些涉及`static`嵌套类但表现为非`static`嵌套类的问题是很重要的:

```java
public class Animal {

    private int age = 10;

    static class EatingHabits {

        private int numOfTimes = 5;

        public void print() {
            System.out.println("The value of x " + age);
            System.out.println("The value of x " + Animal.this.age);
            System.out.println("The value of numOfTimes " + numOfTimes);
        }
    }
}
```

尽管第 10 行和第 11 行对于非`static`嵌套类是有效的，但在这里会导致编译错误。

### 考试提示 5.6:匿名内部类的不正确声明

匿名类和嵌套类一样分散在 OCP 考试中。有很多关于集合、线程和使用匿名内部类的并发性的问题，大多数都有令人困惑的语法。

例如，尝试解决以下问题:

```java
public class Animal {

    public void feed() {
        System.out.println("Eating Grass");
    }
}

public class Zookeeper {

    public static void main(String[] args) {
        Animal animal = new Animal(){
            public void feed(){
                System.out.println("Eating Fish");
            }
        }
        animal.feed();
    }
}
```

```java
What is the result?

A. An exception occurs at runtime
B. Eating Fish
C. Eating Grass
D. Compilation fails because of an error on line 11
E. Compilation fails because of an error on line 12
F. Compilation fails because of an error on line 15
```

由于`Animal` 的**匿名类没有用分号**封闭，所以第 15 行出现编译错误，这就是为什么(F)是正确答案。

### 考试提示 5.7:实例化接口

注意那些试图**实例化一个接口而不是实现它的问题:**

```java
Runnable r = new Runnable(); // compilation error

Runnable r = new Runnable() { // legal statement
    @Override
    public void run() {

    }
};
```

## 6.枚举

枚举是 Java 中表示常量枚举列表的一种方式。它们的行为就像普通的 Java 类，因此可以包含[变量、方法和构造函数](/web/20221205143313/https://www.baeldung.com/java-enum-values)。

尽管相似，枚举的语法确实比常规类复杂。OCP 考试关注的是这种包含枚举的语法不确定性。

### 考试提示 6.1:`enum `声明中的语法错误

注意带有不正确语法错误的`enum `声明。

例如，尝试解决以下问题:

```java
public enum AnimalSpecies {
    MAMMAL(false), FISH(true), BIRD(false),
    REPTILE(false), AMPHIBIAN(true)

    boolean hasFins;

    public AnimalSpecies(boolean hasFins) {
        this.hasFins = hasFins;
    }

    public boolean hasFins() {
        return hasFins;
    }
}
```

```java
What is the result of the following code? (Choose all that apply.)

A. Compiler error on line 2
B. Compiler error on line 3
C. Compiler error on line 7
D. Compiler error on line 11
E. The code compiles successfully
```

这个问题有两个问题:

*   在第 3 行，缺少一个分号(；).记住，如果一个`enum `包含**变量或方法，分号是强制的**
*   这个`enum`中有一个公共构造函数

因此，(B)和(C)都是正确答案。

### 考试提示 6.2: `enum `用`abstract` 的方法

留意实现一个接口或包含一个`abstract `方法的`enum` 问题。

例如，尝试解决以下问题:

```java
public enum AnimalSpecies {
    MAMMAL(false), FISH(true){
        @Override
        boolean canFly() {
            return false;
        }
    }, BIRD(false),
    REPTILE(false), AMPHIBIAN(true);

    boolean hasFins;

    AnimalSpecies(boolean hasFins) {
        this.hasFins = hasFins;
    }

    public boolean hasFins() {
        return hasFins;
    }

    abstract boolean canFly();
}

public class Zookeeper {

    public static void main(String[] args) {
        AnimalSpecies.MAMMAL.canFly();
    }
}
```

```java
What is the result of the following code? (Choose all that apply.)

A. Compilation error on line 2
B. Compilation error on line 4
C. Compilation error on line 20
D. Compilation error on line 26
E. No compilation error
```

因为有一个`abstract `方法，我们必须为每个`enum`常量提供它的实现。因为上面的代码只为`FISH`实现了它，我们会得到一个编译错误。因此，(A)是正确答案。

同样，如果`enum `实现了一个接口`,` **，那么每个常量都必须为该接口`.`的所有方法**提供实现

### 考试提示 6.3:迭代`enum `值

Java 为迭代`enum `值的[提供了静态方法。我们必须预料到要求我们计算一次这样的迭代的输出的问题。](/web/20221205143313/https://www.baeldung.com/java-enum-iteration)

例如，尝试解决以下问题:

```java
public enum AnimalSpecies {
    MAMMAL, FISH, BIRD, REPTILE, AMPHIBIAN
}

public class Zookeeper {

    public static void main(String[] args) {
        AnimalSpecies[] animals = AnimalSpecies.values();
        System.out.println(animals[2]);
    }
}
```

```java
What is the result? (Choose all that apply.)

A. FISH
B. BIRD
C. Compilation fails due to an error on line 2
D. Compilation fails due to an error on line 8
E. Compilation fails due to an error on line 10
```

输出是`BIRD`，因此，(B)是正确的。

## 7.Java 中的接口和`@Override `

在 Java 中，[接口](/web/20221205143313/https://www.baeldung.com/java-interfaces)是为类定义契约的抽象类型。OCP 考试有各种各样的问题来测试考生的继承，方法覆盖和多重继承问题。

### 考试提示 7.1: `abstract `非`abstract `类中的方法实现

注意那些没有实现一个接口的所有`abstract `方法的具体实现。

例如，尝试解决以下问题:

```java
class Bird implements Flyable {
    public void fly() {
    }
}

abstract class Catbirds extends Bird {

}

abstract class Flamingos extends Bird {
    public abstract String color();
}

class GreaterFlamingo extends Flamingos {
    public String color() {
        System.out.println("The color is pink");
    }    
}

interface Flyable {
    void fly();
}
```

```java
What is the result? (Choose all that apply.)

A. Compilation succeeds
B. Compilation fails with an error on line 6
C. Compilation fails with an error on line 10
D. Compilation fails with an error on line 11
E. Compilation fails with an error on line 14
```

因为所有这些都是有效的陈述，(A)是正确的答案。

对于继承的级别，这样的问题有时会很棘手。因此，在试图通过跟踪重写的方法来计算输出之前，我们必须注意任何编译错误。

另一个这样的编译错误来自于`implements `和`extends:`的使用

```java
interface Bird extends Flyable, Wings {}

public class GreaterFlamingo extends Flamingos implements Bird, Vegetarian {}

public class GreaterFlamingo extends Flamingos, Bird {}
```

这里，第 1 & 3 行是有效语句，而第 5 行在 Java 中是不允许的。第 3 行的`GreaterFlamingo` 类现在必须提供所有`abstract` 方法的具体实现。

### 考试提示 7.2: `default `具有相同方法签名的方法

从 JDK 8 开始，接口现在可以有 [`static`和`default `方法](/web/20221205143313/https://www.baeldung.com/java-static-default-methods)。这可能会导致多个接口包含一个具有相同签名的`default `方法的情况。我们会用这样的界面在考试中发现问题。

例如，尝试解决以下问题:

```java
public interface Vegetarian {

    default void eat() {
        System.out.println("Eat Veg");
    }
}

public interface NonVegetarian {

    default void eat() {
        System.out.println("Eat NonVeg");
    }
}

public class Racoon implements Vegetarian, NonVegetarian {

    @Override
    void eat() {
        System.out.println("Eat Something")
    }

    public static void main(String[] args) {
        Racoon racoon = new Racoon();
        racoon.eat();
    }
}
```

```java
What is the result?

A. Eat Veg
B. Eat NonVeg
C. Eat Something
D. The output is unpredictable
E. Compilation fails
F. An exception is thrown at runtime
```

这个问题和[多重继承](/web/20221205143313/https://www.baeldung.com/java-inheritance)有关。值得注意的是，规则说如果从多个接口覆盖了`default `方法，我们必须**提供它的实现。**

现在，因为这段代码确实提供了一个`eat() `方法的实现，所以它可能看起来是一个有效的代码。然而，如果我们仔细观察，我们会发现被覆盖的`eat() `方法不是`public.` ，因此，正确答案是(E)。

### 考试提示 7.3:使用`@Override `

`[@Override](/web/20221205143313/https://www.baeldung.com/java-override) `用于表示 Java 中被覆盖的方法。虽然是可选的，但它提高了可读性，并帮助编译器报告不正确的语法。在考试中寻找这个注释的误用。

例如，尝试解决以下问题:

```java
public abstract class Flamingo {

    public abstract String color();

    public abstract void fly();
}

public class GreaterFlamingo extends Flamingo {
    @Override
    public String color() {
        return "Pink";
    }

    @Override
    public void fly() {
        System.out.println("Flying");
    }

    @Override
    public void eat() {
        System.out.println("Eating");
    }

    public static void main(String[] args) {
        GreaterFlamingo flamingo = new GreaterFlamingo();
        System.out.println(flamingo.color());
    }
}
```

```java
What is the result? (Choose all that apply.)

A. Pink
B. Compilation error on line 8
C. Compilation error on line 19
D. Compilation error on line 20
```

请注意，我们在`eat() `方法上使用了`@Override `。然而，因为在`Flamingo `类中没有这样的`abstract`方法，所以这不是一个被覆盖的方法。因此，(C)是正确的答案。

## 8.创建和使用 Lambda 表达式

高级 Java 类设计最后一个考试目标是关于 lambdas 的。必须记住，lambda 表达式可以用来替代实现[函数接口](/web/20221205143313/https://www.baeldung.com/java-8-functional-interfaces)的匿名内部类。因此，我们会在考试中看到很多问题交替使用这两个词。

lambda 表达式的语法有点复杂。为了找出考试中的语法错误，理解 lambdas 周围的一些[规则是很重要的。](/web/20221205143313/https://www.baeldung.com/java-8-lambda-expressions-tips)

### 考试提示 8.1:Lambda 声明中的非`final `变量

类似于方法局部类，我们只能在 lambda 函数中使用`final `或有效的`final` 变量。考试问题可能不遵守这样的约束。

例如，尝试解决以下问题:

```java
List<String> birds = Arrays.asList("eagle", "seagull", "albatross", "buzzard", "goose");
int longest = 0;
birds.forEach(b -> {
    if (b.length() > longest){
        longest = b.length();
    }
});

System.out.println("Longest bird name is length: " + longest);
```

```java
What is the result?

A. "Longest bird name is length: 9"
B. Compilation fails because of an error on line 3
C. Compilation fails because of an error on line 5
D. A runtime exception occurs on line 5
```

这将导致编译错误，因为我们试图**给 lambda 表达式**中的变量赋值。因此，(C)是正确的答案。

## 9.结论

一般来说，阅读和理解考试中的问题语法是很重要的。**大多数编码题都试图用编译错误来迷惑考生**。因此，在计算输出之前排除任何此类错误是很重要的。

在本文中，我们讨论了一些在考试中经常出现的技巧以及一些示例问题。这些只是样题，用来展示我们在考试中可以期待什么。

当然，破解考试的最好方法是事先练习这些模拟题！