# 在 java 中携带

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-currying>

## 1.介绍

从 Java 8 开始，我们可以在 Java 中定义单参数和双参数函数，允许我们将它们的行为作为参数传递给其他函数。但是对于有更多参数的函数，我们依赖于像 [Vavr](/web/20221208143921/https://www.baeldung.com/vavr) 这样的外部库。

另一种选择是使用[curry](https://web.archive.org/web/20221208143921/https://en.wikipedia.org/wiki/Currying)。通过结合 currying 和[功能接口](/web/20221208143921/https://www.baeldung.com/java-8-functional-interfaces)，我们甚至可以定义易读的构建器，强制用户提供所有输入。

在本教程中，我们将定义奉承并介绍其用法。

## 2.简单的例子

让我们考虑一个带有多个参数的字母的具体例子。

我们简化的第一版只需要一个正文和一个称呼:

```java
class Letter {
    private String salutation;
    private String body;

    Letter(String salutation, String body){
        this.salutation = salutation;
        this.body = body;
    }
}
```

### 2.1.按方法创作

这种对象可以通过以下方法轻松创建:

```java
Letter createLetter(String salutation, String body){
    return new Letter(salutation, body);
}
```

### 2.2.用`BiFunction`创作

上面的方法工作得很好，但是我们可能需要将这种行为提供给以函数风格编写的东西。从 Java 8 开始，我们可以使用`BiFunction`来达到这个目的:

```java
BiFunction<String, String, Letter> SIMPLE_LETTER_CREATOR 
  = (salutation, body) -> new Letter(salutation, body); 
```

### 2.3.一系列功能的创造

我们也可以把它重新表述为一系列函数，每个函数都有一个参数:

```java
Function<String, Function<String, Letter>> SIMPLE_CURRIED_LETTER_CREATOR 
  = salutation -> body -> new Letter(salutation, body);
```

我们看到`salutation`映射到一个函数。结果函数映射到新的`Letter`对象上。从`BiFunction`看返回类型是如何变化的。我们只使用`Function`类。**这种到函数序列的转换叫做 currying。**

## 3.高级示例

为了展示 curry 的优势，让我们用更多的参数来扩展我们的`Letter`类构造函数:

```java
class Letter {
    private String returningAddress;
    private String insideAddress;
    private LocalDate dateOfLetter;
    private String salutation;
    private String body;
    private String closing;

    Letter(String returningAddress, String insideAddress, LocalDate dateOfLetter, 
      String salutation, String body, String closing) {
        this.returningAddress = returningAddress;
        this.insideAddress = insideAddress;
        this.dateOfLetter = dateOfLetter;
        this.salutation = salutation;
        this.body = body;
        this.closing = closing;
    }
}
```

### 3.1.按方法创作

像以前一样，我们可以用一种方法创建对象:

```java
Letter createLetter(String returnAddress, String insideAddress, LocalDate dateOfLetter, 
  String salutation, String body, String closing) {
    return new Letter(returnAddress, insideAddress, dateOfLetter, salutation, body, closing);
}
```

### 3.2.任意 Arity 的函数

Arity 是一个函数接受的参数数量的度量。Java 提供了现有的[一元](/web/20221208143921/https://www.baeldung.com/java-8-functional-interfaces) ( `Supplier`)、[一元](/web/20221208143921/https://www.baeldung.com/java-8-functional-interfaces) ( `Function`)、二进制(`BiFunction`)的函数接口，但仅此而已。如果不定义一个新的函数接口，我们就不能提供一个有六个输入参数的函数。

奉承是我们的出路。它将一个任意的 arity 转换成一个一元函数序列。对于我们的例子，我们得到:

```java
Function<String, Function<String, Function<LocalDate, Function<String,
  Function<String, Function<String, Letter>>>>>> LETTER_CREATOR =
  returnAddress
    -> closing
    -> dateOfLetter
    -> insideAddress
    -> salutation
    -> body
    -> new Letter(returnAddress, insideAddress, dateOfLetter, salutation, body, closing);
```

### 3.3.详细类型

显然，上面的类型不太可读。在这个表单中，我们使用了六次`‘apply'`来创建一个`Letter`:

```java
LETTER_CREATOR
  .apply(RETURNING_ADDRESS)
  .apply(CLOSING)
  .apply(DATE_OF_LETTER)
  .apply(INSIDE_ADDRESS)
  .apply(SALUTATION)
  .apply(BODY);
```

### 3.4.预填充值

有了这个函数链，我们可以创建一个帮助器，它预先填写第一个值，并返回函数以继续完成 letter 对象:

```java
Function<String, Function<LocalDate, Function<String, Function<String, Function<String, Letter>>>>> 
  LETTER_CREATOR_PREFILLED = returningAddress -> LETTER_CREATOR.apply(returningAddress).apply(CLOSING);
```

注意，为了使这个有用，**我们必须仔细选择原始函数中参数的顺序，使不太具体的参数成为第一个。**

## 4.构建器模式

为了克服不友好的类型定义和标准`apply`方法的重复使用，也就是说你不知道输入的正确顺序，我们可以使用[构建器模式](/web/20221208143921/https://www.baeldung.com/creational-design-patterns#builder):

```java
AddReturnAddress builder(){
    return returnAddress
      -> closing
      -> dateOfLetter
      -> insideAddress
      -> salutation
      -> body
      -> new Letter(returnAddress, insideAddress, dateOfLetter, salutation, body, closing);
}
```

我们使用一系列功能接口，而不是一系列功能。注意上面定义的返回类型是`AddReturnAddress`。在下文中，我们只需定义中间接口:

```java
interface AddReturnAddress {
    Letter.AddClosing withReturnAddress(String returnAddress);
}

interface AddClosing {
    Letter.AddDateOfLetter withClosing(String closing);
}

interface AddDateOfLetter {
    Letter.AddInsideAddress withDateOfLetter(LocalDate dateOfLetter);
}

interface AddInsideAddress {
    Letter.AddSalutation withInsideAddress(String insideAddress);
}

interface AddSalutation {
    Letter.AddBody withSalutation(String salutation);
}

interface AddBody {
    Letter withBody(String body);
}
```

所以用这个来创建一个`Letter` 是不言自明的:

```java
Letter.builder()
  .withReturnAddress(RETURNING_ADDRESS)
  .withClosing(CLOSING)
  .withDateOfLetter(DATE_OF_LETTER)
  .withInsideAddress(INSIDE_ADDRESS)
  .withSalutation(SALUTATION)
  .withBody(BODY));
```

像以前一样，我们可以预先填充字母对象:

```java
AddDateOfLetter prefilledLetter = Letter.builder().
  withReturnAddress(RETURNING_ADDRESS).withClosing(CLOSING);
```

注意**接口确保填充顺序**。所以，不能只预填`closing`。

## 5.结论

我们已经了解了如何应用 currying，因此我们不会受到标准 Java 函数接口所支持的有限参数数量的限制。此外，我们可以轻松地预填充前几个参数。此外，我们还学习了如何使用它来创建一个可读的构建器。

和往常一样，完整的代码样本可以在 GitHub 上找到。