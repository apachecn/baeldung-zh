# 使用 AssertJ 自定义断言

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/assertj-custom-assertion>

[This article is part of a series:](javascript:void(0);)[• Introduction to AssertJ](/web/20221101120817/https://www.baeldung.com/introduction-to-assertj)
[• AssertJ for Guava](/web/20221101120817/https://www.baeldung.com/assertJ-for-guava)
[• AssertJ’s Java 8 Features](/web/20221101120817/https://www.baeldung.com/assertJ-java-8-features)
• Custom Assertions with AssertJ (current article)

## 1。概述

在本教程中，我们将逐步创建定制的 [AssertJ](https://web.archive.org/web/20221101120817/https://joel-costigliola.github.io/assertj/) 断言；AssertJ 的基本知识[可以在这里找到。](/web/20221101120817/https://www.baeldung.com/introduction-to-assertj)

简单地说，定制断言允许创建特定于我们自己的类的断言，允许我们的测试更好地反映领域模型。

## 2。测试中的类别

本教程中的测试用例将围绕`Person`类构建:

```java
public class Person {
    private String fullName;
    private int age;
    private List<String> nicknames;

    public Person(String fullName, int age) {
        this.fullName = fullName;
        this.age = age;
        this.nicknames = new ArrayList<>();
    }

    public void addNickname(String nickname) {
        nicknames.add(nickname);
    }

    // getters
} 
```

## 3。自定义断言类

编写定制的 AssertJ 断言类非常简单。**我们需要做的就是声明一个扩展`AbstractAssert`的类，添加一个需要的构造函数，并提供自定义断言方法。**

断言类必须扩展`AbstractAssert`类，让我们能够访问 API 的基本断言方法，比如`isNotNull` 和`isEqualTo`。

下面是`Person`的自定义断言类的框架:

```java
public class PersonAssert extends AbstractAssert<PersonAssert, Person> {

    public PersonAssert(Person actual) {
        super(actual, PersonAssert.class);
    }

    // assertion methods described later
}
```

我们必须在扩展`AbstractAssert`类时指定两个类型参数:第一个是自定义断言类本身，这是方法链接所需要的，第二个是测试中的类。

为了给我们的断言类提供一个入口点，我们可以定义一个静态方法来启动断言链:

```java
public static PersonAssert assertThat(Person actual) {
    return new PersonAssert(actual);
}
```

接下来，我们将检查包含在`PersonAssert`类中的几个自定义断言。

第一种方法验证一个`Person`的全名是否与一个`String`参数匹配:

```java
public PersonAssert hasFullName(String fullName) {
    isNotNull();
    if (!actual.getFullName().equals(fullName)) {
        failWithMessage("Expected person to have full name %s but was %s", 
          fullName, actual.getFullName());
    }
    return this;
}
```

以下方法根据`age`测试`Person`是否为成年人:

```java
public PersonAssert isAdult() {
    isNotNull();
    if (actual.getAge() < 18) {
        failWithMessage("Expected person to be adult");
    }
    return this;
}
```

最后检查一个`nickname`的存在:

```java
public PersonAssert hasNickName(String nickName) {
    isNotNull();
    if (!actual.getNickNames().contains(nickName)) {
        failWithMessage("Expected person to have nickname %s", 
          nickName);
    }
    return this;
}
```

当有多个自定义断言类时，我们可以将所有的`assertThat`方法包装在一个类中，为每个断言类提供一个静态工厂方法:

```java
public class Assertions {
    public static PersonAssert assertThat(Person actual) {
        return new PersonAssert(actual);
    }

    // static factory methods of other assertion classes
}
```

上面显示的`Assertions`类是所有定制断言类的一个方便的入口点。

该类的静态方法具有相同的名称，并且通过它们的参数类型相互区别。

## 4。行动中

下面的测试用例将说明我们在上一节中创建的定制断言方法。注意，`assertThat`方法是从我们的自定义`Assertions`类导入的，而不是从核心 AssertJ API 导入的。

下面是如何使用`hasFullName`方法:

```java
@Test
public void whenPersonNameMatches_thenCorrect() {
    Person person = new Person("John Doe", 20);
    assertThat(person)
      .hasFullName("John Doe");
}
```

这是一个说明`isAdult`方法的负面测试案例:

```java
@Test
public void whenPersonAgeLessThanEighteen_thenNotAdult() {
    Person person = new Person("Jane Roe", 16);

    // assertion fails
    assertThat(person).isAdult();
}
```

另一个测试演示了`hasNickname`方法:

```java
@Test
public void whenPersonDoesNotHaveAMatchingNickname_thenIncorrect() {
    Person person = new Person("John Doe", 20);
    person.addNickname("Nick");

    // assertion will fail
    assertThat(person)
      .hasNickname("John");
}
```

## 5。断言生成器

编写对应于对象模型的定制断言类为可读性很强的测试用例铺平了道路。

然而，如果我们有很多类，为它们手动创建自定义断言类会很痛苦。这就是 AssertJ 断言生成器发挥作用的地方。

要在 Maven 中使用断言生成器，我们需要在`pom.xml`文件中添加一个插件:

```java
<plugin>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-assertions-generator-maven-plugin</artifactId>
    <version>2.1.0</version>
    <configuration>
        <classes>
            <param>com.baeldung.testing.assertj.custom.Person</param>
        </classes>
    </configuration>
</plugin>
```

最新版本的`assertj-assertions-generator-maven-plugin`可以在[这里](https://web.archive.org/web/20221101120817/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.assertj%22%20AND%20a%3A%22assertj-assertions-generator-maven-plugin%22)找到。

上面插件中的`classes`元素标记了我们想要生成断言的类。插件的其他配置请看[这篇帖子](https://web.archive.org/web/20221101120817/https://joel-costigliola.github.io/assertj/assertj-assertions-generator-maven-plugin.html#configuration)。

AssertJ 断言生成器为目标类的每个公共属性创建断言。每个断言方法的具体名称取决于字段或属性的类型。对于断言生成器的完整描述，请查看[参考文献](https://web.archive.org/web/20221101120817/https://joel-costigliola.github.io/assertj/assertj-assertions-generator.html)。

在项目基本目录中执行以下 Maven 命令:

```java
mvn assertj:generate-assertions
```

我们应该看到在`target` `/generated-test-sources/assertj-assertions`文件夹中生成的断言类。例如，为生成的断言生成的入口点类如下所示:

```java
// generated comments are stripped off for brevity

package com.baeldung.testing.assertj.custom;

@javax.annotation.Generated(value="assertj-assertions-generator")
public class Assertions {

    @org.assertj.core.util.CheckReturnValue
    public static com.baeldung.testing.assertj.custom.PersonAssert
      assertThat(com.baeldung.testing.assertj.custom.Person actual) {
        return new com.baeldung.testing.assertj.custom.PersonAssert(actual);
    }

    protected Assertions() {
        // empty
    }
}
```

现在，我们可以将生成的源文件复制到测试目录，然后添加定制的断言方法来满足我们的测试需求。

需要注意的一件重要事情是，生成的代码不能保证完全正确。在这一点上，发电机不是一个成品，社区正在努力。

因此，我们应该使用发电机作为辅助工具，使我们的生活更容易，而不是想当然。

## 6。结论

在本教程中，我们展示了如何手动和自动地创建自定义断言，以便用 AssertJ 库创建可读的测试代码。

如果我们只有少量的测试类，手工解决方案就足够了；否则，应使用发电机。

和往常一样，所有示例和代码片段的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20221101120817/https://github.com/eugenp/tutorials/tree/master/testing-modules/assertion-libraries)

上一篇[AssertJ 的 Java 8 特性](/web/20221101120817/https://www.baeldung.com/assertJ-java-8-features)