# 如何在 Java 中制作对象的深层副本

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-deep-copy>

## 1。简介

当我们想在 Java 中复制一个对象时，有两种可能性需要考虑，[浅层复制和深层复制](/web/20220926184512/https://www.baeldung.com/cs/deep-vs-shallow-copy)。

对于浅层复制方法，我们只复制字段值，因此复制可能依赖于原始对象。在深度复制方法中，我们确保树中的所有对象都被深度复制，因此副本不依赖于任何可能改变的先前存在的对象。

在本教程中，我们将比较这两种方法，并学习实现深度复制的四种方法。

## 延伸阅读:

## [Java 复制构造器](/web/20220926184512/https://www.baeldung.com/java-copy-constructor)

Here's how to create copy constructors in Java and why to implementing Cloneable isn't such a great idea.[Read more](/web/20220926184512/https://www.baeldung.com/java-copy-constructor) →

## [如何在 Java 中复制数组](/web/20220926184512/https://www.baeldung.com/java-array-copy)

Learn how to copy an array in Java, with examples of various methods.[Read more](/web/20220926184512/https://www.baeldung.com/java-array-copy) →

## [在 Java 中复制集合](/web/20220926184512/https://www.baeldung.com/java-copy-sets)

Learn several different ways how to copy a Set in Java.[Read more](/web/20220926184512/https://www.baeldung.com/java-copy-sets) →

## 2。Maven 设置

我们将使用三个 Maven 依赖项 Gson、Jackson 和 Apache Commons Lang 来测试执行深度复制的不同方式。

让我们将这些依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.2</version>
</dependency>
<dependency>
    <groupId>commons-lang</groupId>
    <artifactId>commons-lang</artifactId>
    <version>2.6</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.0</version>
</dependency>
```

最新版本的 [Gson](https://web.archive.org/web/20220926184512/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.code.gson%22%20AND%20a%3A%22gson%22) 、 [Jackson](https://web.archive.org/web/20220926184512/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.fasterxml.jackson.core%22%20AND%20a%3A%22jackson-databind%22) 和 [Apache Commons Lang](https://web.archive.org/web/20220926184512/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22commons-lang%22%20AND%20a%3A%22commons-lang%22) 可以在 Maven Central 上找到。

## 3。型号

为了比较复制 Java 对象的不同方法，我们需要两个类:

```java
class Address {

    private String street;
    private String city;
    private String country;

    // standard constructors, getters and setters
}
```

```java
class User {

    private String firstName;
    private String lastName;
    private Address address;

    // standard constructors, getters and setters
}
```

## 4。浅层复制

浅层拷贝是这样一种拷贝:我们只将字段的值从一个对象拷贝到另一个对象:

```java
@Test
public void whenShallowCopying_thenObjectsShouldNotBeSame() {

    Address address = new Address("Downing St 10", "London", "England");
    User pm = new User("Prime", "Minister", address);

    User shallowCopy = new User(
      pm.getFirstName(), pm.getLastName(), pm.getAddress());

    assertThat(shallowCopy)
      .isNotSameAs(pm);
}
```

在这种情况下，`pm != shallowCopy`，这意味着**它们是不同的对象；然而，问题是当我们改变任何一个原始的`address'`属性时，这也会影响`shallowCopy`的地址**。

如果`Address`是不可变的，我们也不会为此烦恼，但它不是:

```java
@Test
public void whenModifyingOriginalObject_ThenCopyShouldChange() {

    Address address = new Address("Downing St 10", "London", "England");
    User pm = new User("Prime", "Minister", address);
    User shallowCopy = new User(
      pm.getFirstName(), pm.getLastName(), pm.getAddress());

    address.setCountry("Great Britain");
    assertThat(shallowCopy.getAddress().getCountry())
      .isEqualTo(pm.getAddress().getCountry());
}
```

## 5。深层复制

深层拷贝是解决这一问题的替代方案。它的优点是**对象图中的每个可变对象都被递归复制**。

由于副本不依赖于之前创建的任何可变对象，所以它不会像我们在浅层副本中看到的那样被意外修改。

在接下来的几节中，我们将讨论几个深度拷贝实现，并展示这一优势。

### 5.1。复制构造函数

我们将考察的第一个实现基于复制构造函数:

```java
public Address(Address that) {
    this(that.getStreet(), that.getCity(), that.getCountry());
}
```

```java
public User(User that) {
    this(that.getFirstName(), that.getLastName(), new Address(that.getAddress()));
}
```

在上面深度复制的实现中，我们没有在复制构造函数中创建新的`Strings` ,因为`String`是一个不可变的类。

因此，它们不可能被意外修改。让我们看看这是否可行:

```java
@Test
public void whenModifyingOriginalObject_thenCopyShouldNotChange() {
    Address address = new Address("Downing St 10", "London", "England");
    User pm = new User("Prime", "Minister", address);
    User deepCopy = new User(pm);

    address.setCountry("Great Britain");
    assertNotEquals(
      pm.getAddress().getCountry(), 
      deepCopy.getAddress().getCountry());
}
```

### 5.2。可克隆界面

第二个实现基于从`Object`继承的克隆方法。它是受保护的，但是我们需要将其重写为`public`。

我们还将添加一个标记接口，`Cloneable,`到类中，以表明这些类实际上是可克隆的。

让我们将`clone()`方法添加到`Address` 类中:

```java
@Override
public Object clone() {
    try {
        return (Address) super.clone();
    } catch (CloneNotSupportedException e) {
        return new Address(this.street, this.getCity(), this.getCountry());
    }
}
```

现在让我们为`User`类实现`clone()`:

```java
@Override
public Object clone() {
    User user = null;
    try {
        user = (User) super.clone();
    } catch (CloneNotSupportedException e) {
        user = new User(
          this.getFirstName(), this.getLastName(), this.getAddress());
    }
    user.address = (Address) this.address.clone();
    return user;
}
```

**注意，`super.clone()`调用返回一个对象的浅层副本，但是我们手动设置可变字段的深层副本，所以结果是正确的:**

```java
@Test
public void whenModifyingOriginalObject_thenCloneCopyShouldNotChange() {
    Address address = new Address("Downing St 10", "London", "England");
    User pm = new User("Prime", "Minister", address);
    User deepCopy = (User) pm.clone();

    address.setCountry("Great Britain");

    assertThat(deepCopy.getAddress().getCountry())
      .isNotEqualTo(pm.getAddress().getCountry());
}
```

## 6。外部库

上面的例子看起来很简单，但是当我们不能添加一个额外的构造函数或者覆盖克隆方法的时候，有时它们并不能作为一个解决方案**工作。**

当我们不拥有代码时，或者当对象图如此复杂，如果我们专注于编写额外的构造函数或在对象图中的所有类上实现`clone`方法，我们将无法按时完成项目时，这可能会发生。

那么我们能做什么呢？在这种情况下，我们可以使用外部库。为了实现深度复制，**我们可以序列化一个对象，然后将其反序列化为一个新的对象**。

我们来看几个例子。

### 6.1 .Apache common lang〔t1〕

Apache Commons Lang 有`SerializationUtils#clone,`，当对象图中的所有类都实现了`Serializable`接口时，它执行深度复制。

**如果方法遇到一个不可序列化的类，它将失败并抛出一个未检查的`SerializationException`。**

因此，我们需要在我们的类中添加 `Serializable`接口:

```java
@Test
public void whenModifyingOriginalObject_thenCommonsCloneShouldNotChange() {
    Address address = new Address("Downing St 10", "London", "England");
    User pm = new User("Prime", "Minister", address);
    User deepCopy = (User) SerializationUtils.clone(pm);

    address.setCountry("Great Britain");

    assertThat(deepCopy.getAddress().getCountry())
      .isNotEqualTo(pm.getAddress().getCountry());
}
```

### 6.2。JSON 序列化与 Gson

另一种序列化方式是使用 JSON 序列化。Gson 是一个用于将对象转换成 JSON 的库，反之亦然。

与 Apache Commons Lang 不同， **GSON 不需要`Serializable`接口来进行转换**。

让我们快速看一个例子:

```java
@Test
public void whenModifyingOriginalObject_thenGsonCloneShouldNotChange() {
    Address address = new Address("Downing St 10", "London", "England");
    User pm = new User("Prime", "Minister", address);
    Gson gson = new Gson();
    User deepCopy = gson.fromJson(gson.toJson(pm), User.class);

    address.setCountry("Great Britain");

    assertThat(deepCopy.getAddress().getCountry())
      .isNotEqualTo(pm.getAddress().getCountry());
}
```

### 6.3。与 Jackson 的 JSON 序列化

Jackson 是另一个支持 JSON 序列化的库。这个实现将非常类似于使用 Gson 的实现，但是**我们需要将默认的构造函数添加到我们的类**中。

让我们看一个例子:

```java
@Test
public void whenModifyingOriginalObject_thenJacksonCopyShouldNotChange() 
  throws IOException {
    Address address = new Address("Downing St 10", "London", "England");
    User pm = new User("Prime", "Minister", address);
    ObjectMapper objectMapper = new ObjectMapper();

    User deepCopy = objectMapper
      .readValue(objectMapper.writeValueAsString(pm), User.class);

    address.setCountry("Great Britain");

    assertThat(deepCopy.getAddress().getCountry())
      .isNotEqualTo(pm.getAddress().getCountry());
}
```

## 7。结论

制作深层副本时，我们应该使用哪种实现？最终的决定通常取决于我们要复制的类，以及我们是否拥有对象图中的类。

和往常一样，本文的完整代码示例可以在 GitHub 的[中找到。](https://web.archive.org/web/20220926184512/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-patterns)