# 避免在 Java 中检查空语句

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-avoid-null-check>

## 1.概观

一般来说，变量、引用和集合在 Java 代码中很难处理。它们不仅难以识别，而且处理起来也很复杂。

事实上，在处理`null`时的任何失误在编译时都无法识别，并在运行时导致一个`NullPointerException` 。

在本教程中，我们将看看在 Java 中检查`null` 的必要性，以及帮助我们在代码中避免`null`检查的各种替代方法。

## 延伸阅读:

## [使用 NullAway 避免 NullPointerExceptions](/web/20220812145536/https://www.baeldung.com/java-nullaway)

Learn how to avoid NullPointerExceptions using NullAway.[Read more](/web/20220812145536/https://www.baeldung.com/java-nullaway) →

## [弹簧零安全注释](/web/20220812145536/https://www.baeldung.com/spring-null-safety-annotations)

A quick and practical guide to null-safety annotations in Spring.[Read more](/web/20220812145536/https://www.baeldung.com/spring-null-safety-annotations) →

## [空对象模式介绍](/web/20220812145536/https://www.baeldung.com/java-null-object-pattern)

Learn about the Null Object Pattern and how to implement it in Java[Read more](/web/20220812145536/https://www.baeldung.com/java-null-object-pattern) →

## 2.什么是`NullPointerException`？

根据`NullPointerException` 的 [Javadoc，在需要一个对象的情况下，当应用程序试图使用`null`时抛出，比如:](https://web.archive.org/web/20220812145536/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/NullPointerException.html)

*   调用一个`null`对象的实例方法
*   访问或修改`null`对象的字段
*   将`null`的长度视为一个数组
*   像访问数组一样访问或修改`null`的槽
*   抛出`null`，好像它是一个`Throwable`值

让我们快速看几个导致这个异常的 Java 代码示例:

```java
public void doSomething() {
    String result = doSomethingElse();
    if (result.equalsIgnoreCase("Success")) 
        // success
    }
}

private String doSomethingElse() {
    return null;
}
```

在这里，**我们试图为一个`null` 引用调用一个方法。这将导致一个`NullPointerException`。**

另一个常见的例子是，如果我们试图访问一个`null` 数组:

```java
public static void main(String[] args) {
    findMax(null);
}

private static void findMax(int[] arr) {
    int max = arr[0];
    //check other elements in loop
}
```

这导致第 6 行出现`NullPointerException`。

因此，访问一个`null` 对象的任何字段、方法或索引都会导致一个`NullPointerException`，从上面的例子可以看出。

避免`NullPointerException` 的常见方法是检查`null`:

```java
public void doSomething() {
    String result = doSomethingElse();
    if (result != null && result.equalsIgnoreCase("Success")) {
        // success
    }
    else
        // failure
}

private String doSomethingElse() {
    return null;
}
```

在现实世界中，程序员发现很难识别哪些对象可以是`null.` **一个积极安全的策略是检查每个对象的`null`。然而，这导致了许多多余的`null` 检查，使我们的代码可读性更差。**

在接下来的几节中，我们将介绍 Java 中避免这种冗余的一些替代方法。

## 3.通过 API 合同处理`null`

正如上一节所讨论的，访问`null` 对象的方法或变量会导致一个`NullPointerException`。我们还讨论了在访问对象之前对其进行`null` 检查消除了`NullPointerException`的可能性。

然而，通常有一些 API 可以处理`null` 值:

```java
public void print(Object param) {
    System.out.println("Printing " + param);
}

public Object process() throws Exception {
    Object result = doSomething();
    if (result == null) {
        throw new Exception("Processing fail. Got a null response");
    } else {
        return result;
    }
}
```

方法调用只会输出“null ”,但不会抛出异常。同样，`process()` 也不会在它的响应中返回`null` 。而是抛出一个`Exception`。

因此，对于访问上述 API 的客户机代码，不需要进行`null` 检查。

然而，这样的 API 需要在它们的契约中明确这一点。API 发布这种契约的常见地方是 Javadoc。

但是这没有给出 API 契约的明确指示，因此依赖于客户代码开发者来确保它的符合性。

在下一节中，我们将看到一些 ide 和其他开发工具如何帮助开发人员做到这一点。

## 4.自动化 API 合同

### 4.1.使用静态代码分析

[静态代码分析](/web/20220812145536/https://www.baeldung.com/java-static-analysis-tools)工具极大地帮助提高了代码质量。一些这样的工具也允许开发人员维护`null` 契约。一个例子是 [FindBugs](/web/20220812145536/https://www.baeldung.com/intro-to-findbugs) 。

**FindBugs 通过`@Nullable` 和`@NonNull` 注释帮助管理`null` 合同。我们可以在任何方法、字段、局部变量或参数上使用这些注释。这使得客户机代码清楚地知道注释的类型是否可以是`null` 。**

让我们看一个例子:

```java
public void accept(@NonNull Object param) {
    System.out.println(param.toString());
}
```

这里，`@NonNull` 明确了论点不能是`null`。**如果客户端代码调用这个方法而没有检查`null,` 的参数，FindBugs 会在编译时生成一个警告。**

### 4.2.使用 IDE 支持

开发人员通常依靠 ide 来编写 Java 代码。智能代码完成和有用的警告(例如，当变量可能没有被赋值时)等功能无疑非常有用。

一些 ide 还允许开发人员管理 API 契约，从而消除了对静态代码分析工具的需求。 **IntelliJ IDEA 提供了`@NonNull` 和`@Nullable` 注解。**

要在 IntelliJ 中添加对这些注释的支持，我们需要添加以下 Maven 依赖项:

```java
<dependency>
    <groupId>org.jetbrains</groupId>
    <artifactId>annotations</artifactId>
    <version>16.0.2</version>
</dependency>
```

现在，如果`null` 检查丢失，IntelliJ 将生成一个警告，就像我们上一个例子一样。

IntelliJ 还提供了一个用于处理复杂 API 契约的`[Contract](https://web.archive.org/web/20220812145536/https://www.jetbrains.com/help/idea/contract-annotations.html)`注释。

## 5.断言

到目前为止，我们只讨论了从客户端代码中移除对`null` 检查的需求。但是这在现实世界的应用中很少适用。

现在让我们**假设我们正在使用一个 API，它不能接受`null` 参数或者可以返回一个必须由客户端处理的`null` 响应。**这需要我们检查参数或对 `null`值的响应。

在这里，我们可以使用 [Java 断言](/web/20220812145536/https://www.baeldung.com/java-assert)代替传统的`null`检查条件语句:

```java
public void accept(Object param){
    assert param != null;
    doSomething(param);
}
```

在第 2 行，我们检查一个`null`参数。**如果断言被启用，这将导致一个** `**AssertionError**` **。**

尽管这是一种断言前提条件(如非`null`参数、**的好方法，但这种方法有两个主要问题**:

1.  JVM 中通常禁用断言。
2.  一个`false` 断言导致一个不可恢复的未检查的错误。

**因此，不建议程序员使用断言来检查条件。**在接下来的部分中，我们将讨论处理`null` 验证的其他方式。

## 6.通过编码实践避免`Null` 检查

### 6.1.前提

编写早期失败的代码通常是一个好的实践。因此，如果一个 API 接受多个不允许为`null`、**的参数，最好检查每个非`null`参数，作为 API 的前提条件。**

让我们来看看两种方法——一种在早期会失败，另一种不会:

```java
public void goodAccept(String one, String two, String three) {
    if (one == null || two == null || three == null) {
        throw new IllegalArgumentException();
    }

    process(one);
    process(two);
    process(three);
}

public void badAccept(String one, String two, String three) {
    if (one == null) {
        throw new IllegalArgumentException();
    } else {
        process(one);
    }

    if (two == null) {
        throw new IllegalArgumentException();
    } else {
        process(two);
    }

    if (three == null) {
        throw new IllegalArgumentException();
    } else {
        process(three);
    }
}
```

显然，我们应该更喜欢`goodAccept()`而不是`badAccept()`。

作为替代，我们也可以使用 [Guava 的前提条件](/web/20220812145536/https://www.baeldung.com/guava-preconditions)来验证 API 参数。

### 6.2.使用原语而不是包装类

因为对于像`int`这样的原语来说`null` 不是一个可接受的值，所以我们应该尽可能地选择它们，而不是像`Integer` 这样的包装器。

考虑对两个整数求和的方法的两种实现:

```java
public static int primitiveSum(int a, int b) {
    return a + b;
}

public static Integer wrapperSum(Integer a, Integer b) {
    return a + b;
}
```

现在让我们在客户端代码中调用这些 API:

```java
int sum = primitiveSum(null, 2);
```

**这将导致编译时错误，因为`null` 不是`int`的有效值。**

当使用带有包装类的 API 时，我们得到一个`NullPointerException`:

```java
assertThrows(NullPointerException.class, () -> wrapperSum(null, 2));
```

正如我们在另一篇教程 [Java 原语与对象](/web/20220812145536/https://www.baeldung.com/java-primitives-vs-objects)中所介绍的，在包装器上使用原语还有其他因素。

### 6.3.空集合

有时候，我们需要返回一个集合作为方法的响应。对于这样的方法，我们应该总是试图用**返回一个空集合，而不是用`null`** :

```java
public List<String> names() {
    if (userExists()) {
        return Stream.of(readName()).collect(Collectors.toList());
    } else {
        return Collections.emptyList();
    }
}
```

这样，我们就避免了客户端在调用这个方法时执行`null`检查的需要。

## 7.使用`Objects`

Java 7 引入了新的`Objects` API。这个 API 有几个`static` 实用方法，可以去掉很多冗余代码。

让我们来看一个这样的方法， [`requireNonNull()`](https://web.archive.org/web/20220812145536/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Objects.html#requireNonNull(T)) :

```java
public void accept(Object param) {
    Objects.requireNonNull(param);
    // doSomething()
}
```

现在让我们测试一下`accept()` 方法:

```java
assertThrows(NullPointerException.class, () -> accept(null));
```

所以，如果`null` 作为参数传递，`accept()`抛出一个`NullPointerException`。

这个类还有 **`isNull()` 和`nonNull()` 方法，可以用作谓词来检查对象的`null`。**

## 8.使用`Optional`

### 8.1.使用`orElseThrow`

Java 8 在语言中引入了一个新的`[Optional](/web/20220812145536/https://www.baeldung.com/java-optional)` API。与`null`相比，这为处理可选值提供了更好的契约。

让我们看看`Optional` 如何消除对`null` 检查的需求:

```java
public Optional<Object> process(boolean processed) {
    String response = doSomething(processed);

    if (response == null) {
        return Optional.empty();
    }

    return Optional.of(response);
}

private String doSomething(boolean processed) {
    if (processed) {
        return "passed";
    } else {
        return null;
    }
}
```

通过如上所示返回一个`Optional,` ，**`process`方法向调用者表明响应可以是空的，需要在编译时处理。**

这明显地消除了客户端代码中任何`null` 检查的需要。使用`Optional` API 的声明式风格可以不同地处理空响应:

```java
assertThrows(Exception.class, () -> process(false).orElseThrow(() -> new Exception()));
```

此外，它还为 API 开发者提供了一个更好的契约，向客户表明 API 可以返回空响应。

虽然我们消除了对这个 API 的调用者进行`null` 检查的需要，但是我们使用它来返回一个空响应。

为了避免这种情况， **`Optional`提供了一个`ofNullable` 方法，该方法返回一个具有指定值的`Optional` ，或者如果值为`null`** ，则返回`empty`:

```java
public Optional<Object> process(boolean processed) {
    String response = doSomething(processed);
    return Optional.ofNullable(response);
}
```

### 8.2.对集合使用`Optional`

在处理空集合时，`Optional` 派上了用场:

```java
public String findFirst() {
    return getList().stream()
      .findFirst()
      .orElse(DEFAULT_VALUE);
}
```

这个函数应该返回列表的第一项。当没有数据时，`Stream` API 的`findFirst`函数将返回一个空的`Optional` 。这里，我们使用`orElse`来提供一个默认值。

这允许我们处理空列表，或者在我们使用了`Stream`库的`filter`方法之后，没有条目可提供的列表。

或者，我们也可以允许客户端通过从该方法返回`Optional` 来决定如何处理`empty`:

```java
public Optional<String> findOptionalFirst() {
    return getList().stream()
      .findFirst();
}
```

**因此，如果`getList` 的结果为空，这个方法将返回一个空的`Optional` 给客户端。**

对集合使用`Optional` 允许我们设计一定会返回非空值的 API，从而避免在客户端进行显式的`null` 检查。

这里需要注意的是，这个实现依赖于`getList` 不返回`null.` 然而，正如我们在上一节中讨论的，返回一个空列表通常比返回一个`null`更好。

### 8.3.组合选项

当我们开始让函数返回`Optional`时，我们需要一种方法将它们的结果合并成一个值。

让我们以之前的`getList`为例。如果它要返回一个`Optional`列表，或者要用一个使用`ofNullable`包装`Optional`和`null`的方法来包装，那会怎么样呢？

我们的`findFirst`方法想要返回一个`Optional`列表的第一个`Optional`元素:

```java
public Optional<String> optionalListFirst() {
   return getOptionalList()
      .flatMap(list -> list.stream().findFirst());
}
```

通过对从`getOptional`返回的`Optional`使用`flatMap`函数，我们可以解包返回`Optional`的内部表达式的结果。没有`flatMap`，结果会是`Optional<Optional<String>>`。只有当`Optional`不为空时，才会执行`flatMap`操作。

## 9.图书馆

### 9.1.使用 Lombok

[Lombok](/web/20220812145536/https://www.baeldung.com/intro-to-project-lombok) 是一个很棒的库，它减少了我们项目中样板代码的数量。它附带了一组注释，取代了我们在 Java 应用程序中经常自己编写的代码的公共部分，例如 getters、setters 和`toString()`等等。

它的另一个注解是`@NonNull`。所以，如果一个项目已经使用 Lombok 来消除样板代码， **`@NonNull` 可以取代`null` 检查的需要。**

在我们继续讨论一些例子之前，让我们为 Lombok 添加一个 [Maven](https://web.archive.org/web/20220812145536/https://search.maven.org/search?q=g:org.projectlombok%20AND%20a:lombok&core=gav) 依赖项:

```java
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.20</version>
</dependency>
```

现在我们可以在任何需要`null` 检查的地方使用`@NonNull` :

```java
public void accept(@NonNull Object param){
    System.out.println(param);
}
```

因此，我们简单地注释了需要进行`null` 检查的对象，Lombok 生成了编译后的类:

```java
public void accept(@NonNull Object param) {
    if (param == null) {
        throw new NullPointerException("param");
    } else {
        System.out.println(param);
    }
}
```

如果`param` 是`null`，这个方法抛出一个`NullPointerException`。**方法必须在其契约中明确这一点，并且客户端代码必须处理异常。**

### 9.2.使用`StringUtils`

一般来说，`String`验证除了检查`null` 值之外，还包括检查空值。

因此，这将是一个常见的验证语句:

```java
public void accept(String param){
    if (null != param && !param.isEmpty())
        System.out.println(param);
}
```

**如果我们必须处理大量的`String` 类型，这很快就变得多余了。**这就是`StringUtils` 派上用场的地方。

在我们看到这一点之前，让我们为 [commons-lang3](https://web.archive.org/web/20220812145536/https://search.maven.org/search?q=g:org.apache.commons%20AND%20a:commons-lang3&core=gav) 添加一个 Maven 依赖项:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

现在让我们用`StringUtils`重构上面的代码:

```java
public void accept(String param) {
    if (StringUtils.isNotEmpty(param))
        System.out.println(param);
}
```

因此，我们用一个`static` 实用方法`isNotEmpty()`替换了我们的`null` 或空支票。这个 API 提供了其他[强大的实用方法](/web/20220812145536/https://www.baeldung.com/string-processing-commons-lang)来处理常见的`String`函数。

## 10.结论

在本文中，我们研究了`NullPointerException` 的各种原因以及为什么它很难识别。

然后我们看到了各种方法来避免用参数、返回类型和其他变量检查`null`时代码中的冗余。

所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220812145536/https://github.com/eugenp/tutorials/tree/master/patterns/design-patterns-behavioral)