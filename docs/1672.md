# Java 14 记录 vs. Lombok

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-record-vs-lombok>

## 1.概观

[Java 的记录关键字](/web/20220906213109/https://www.baeldung.com/java-record-keyword)是 Java 14 中引入的新语义特性。**记录对于创建小型不可变对象**非常有用。另一方面， **[Lombok](/web/20220906213109/https://www.baeldung.com/intro-to-project-lombok) 是一个 Java 库，可以自动生成一些已知的模式**作为 Java 字节码。尽管它们都可以用来减少样板代码，但它们是不同的工具。因此，我们应该在给定的上下文中使用更适合我们需要的那个。

在本文中，我们将探索各种用例，包括 java 记录的一些限制。对于每个例子，我们将看到 Lombok 是如何派上用场的，并比较这两种解决方案。

## 2.小型不可变对象

对于我们的第一个例子，我们将使用`Color`对象。一个`Color` 由代表红色、绿色和蓝色通道的三个整数值组成。此外，颜色将暴露其十六进制表示。例如，带有`RGB(255,0,0)`的颜色将有一个十六进制表示`#FF0000`。此外，我们希望两种颜色是`equal`，如果它们有相同的 RGB 值。

出于这些原因，在这种情况下选择一个`record `会非常有意义:

```
public record ColorRecord(int red, int green, int blue) {

    public String getHexString() {
        return String.format("#%02X%02X%02X", red, green, blue);
    }
}
```

类似地，Lombok 允许我们使用`@Value`注释创建不可变的对象:

```
@Value
public class ColorValueObject {
    int red;
    int green;
    int blue;

    public String getHexString() {
        return String.format("#%02X%02X%02X", red, green, blue);
    }
}
```

然而，从 Java 14 开始，`records`将是这些用例的自然方式。

## 3.透明数据载体

按照 JDK 增强提案( [JEP 395](https://web.archive.org/web/20220906213109/https://openjdk.org/jeps/359) )，**记录是作为不可变数据的透明载体的类。因此，我们无法阻止记录公开其成员字段。**例如，我们不能强迫前面例子中的`ColorRecord`只暴露`hexString` 而完全隐藏三个整数字段。

但是，Lombok 允许我们定制 getters 的名称、访问级别和返回类型。让我们相应地更新`ColorValueObject`:

```
@Value
@Getter(AccessLevel.NONE)
public class ColorValueObject {
    int red;
    int green;
    int blue;

    public String getHexString() {
        return String.format("#%02X%02X%02X", red, green, blue);
    }
}
```

因此，如果我们需要不可变的数据对象，记录是一个很好的解决方案。

然而，如果我们想隐藏成员字段，只暴露一些使用它们执行的操作，Lombok 会更适合。

## 4.具有许多字段的类

我们已经看到了记录是如何代表一种创建小的、不可变的对象的非常方便的方式。让我们看看如果数据模型需要更多的字段，记录会是什么样子。对于这个例子，让我们考虑一下`Student`数据模型:

```
public record StudentRecord(
  String firstName, 
  String lastName, 
  Long studentId, 
  String email, 
  String phoneNumber, 
  String address, 
  String country, 
  int age) {
}
```

我们已经可以猜到 StudentRecord 的实例化将很难阅读和理解，特别是如果一些字段不是强制的:

```
StudentRecord john = new StudentRecord(
  "John", "Doe", null, "[[email protected]](/web/20220906213109/https://www.baeldung.com/cdn-cgi/l/email-protection)", null, null, "England", 20); 
```

为了方便这些用例，Lombok 提供了一个 [Builder 设计模式](/web/20220906213109/https://www.baeldung.com/creational-design-patterns#builder)的实现。

为了使用它，我们只需要用`@Builder:`来注释我们的类

```
@Getter
@Builder
public class StudentBuilder {
    private String firstName;
    private String lastName;
    private Long studentId;
    private String email;
    private String phoneNumber;
    private String address;
    private String country;
    private int age;
}
```

现在，让我们使用`StudentBuilder`创建一个具有相同属性的对象:

```
StudentBuilder john = StudentBuilder.builder()
  .firstName("John")
  .lastName("Doe")
  .email("[[email protected]](/web/20220906213109/https://www.baeldung.com/cdn-cgi/l/email-protection)")
  .country("England")
  .age(20)
  .build();
```

如果我们比较这两者，我们可以注意到使用构建器模式是有利的，可以产生更干净的代码。

**总之，记录更适合较小的物体。但是，对于有许多字段的对象，缺乏创造性的模式将使 Lombok 的`@Builder`成为更好的选择。**

## 5.可变数据

我们可以将 java 记录专门用于不可变的数据。**如果上下文需要一个可变的 java 对象，我们可以使用 Lombok 的`@Data`对象来代替:**

```
@Data
@AllArgsConstructor
public class ColorData {

    private int red;
    private int green;
    private int blue;

    public String getHexString() {
        return String.format("#%02X%02X%02X", red, green, blue);
    }

} 
```

一些框架可能需要带有 setters 或默认构造函数的对象。例如，Hibernate 就属于这一类。当创建一个`@Entity, `时，我们必须使用 Lombok 的注释或普通 Java。

## 6.遗产

**Java 记录不支持继承。**因此，他们不能延续或继承其他阶级。另一方面，Lombok 的`@Value`对象可以扩展其他类，但它们是最终的:

```
@Value
public class MonochromeColor extends ColorData {

    public MonochromeColor(int grayScale) {
        super(grayScale, grayScale, grayScale);
    }
}
```

此外，`@Data`对象既可以扩展其他类，也可以被扩展。**总之，如果我们需要继承，我们应该坚持 Lombok 的解决方案。**

## 7.结论

在本文中，我们已经看到 Lombok 和 java 记录是不同的工具，服务于不同的目的。此外，我们发现 Lombok 更加灵活，可以用于记录有限的场景。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220906213109/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-14)