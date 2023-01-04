# Java 中的方法参数反射

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-parameter-reflection>

## 1。概述

Java 8 中增加了方法参数反射支持。简单地说，它提供了在运行时获取参数名称的支持。

在这个快速教程中，我们将看看如何在运行时使用反射来访问构造函数和方法的参数名。

## 2。编译器参数

为了能够访问方法名信息，我们必须显式地选择加入。

为此，我们在编译期间**指定了`parameters`选项。**

对于一个 Maven 项目，我们可以在`pom.xml`中声明这个选项:

```java
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <version>3.1</version>
  <configuration>
    <source>1.8</source>
    <target>1.8</target>
    <compilerArgument>-parameters</compilerArgument>
  </configuration>
</plugin> 
```

## 3。示例类

我们将使用一个虚构的具有名为`fullName`的属性的`Person`类来演示:

```java
public class Person {

    private String fullName;

    public Person(String fullName) {
        this.fullName = fullName;
    }

    public void setFullName(String fullName) {
        this.fullName = fullName;
    }

    // other methods
}
```

## 4。用途

`Parameter`类是 Java 8 中的新特性，有各种有趣的方法。如果提供了`-parameters`编译器选项，`isNamePresent()`方法将返回 true。

要访问参数的名称，我们可以简单地调用`getName()` `:`

```java
@Test
public void whenGetConstructorParams_thenOk() 
  throws NoSuchMethodException, SecurityException {

    List<Parameter> parameters 
        = Arrays.asList(Person.class.getConstructor(String.class).getParameters());
    Optional<Parameter> parameter 
        = parameters.stream().filter(Parameter::isNamePresent).findFirst();
    assertThat(parameter.get().getName()).isEqualTo("fullName");
}

@Test
public void whenGetMethodParams_thenOk() 
  throws NoSuchMethodException, SecurityException {

    List<Parameter> parameters = Arrays.asList(
      Person.class.getMethod("setFullName", String.class).getParameters());
    Optional<Parameter> parameter= parameters.stream()
      .filter(Parameter::isNamePresent)
      .findFirst();

    assertThat(parameter.get().getName()).isEqualTo("fullName");
}
```

## 5。结论

在这篇简短的文章中，我们研究了 Java 8 中对参数名的新反射支持。

这些信息最明显的用例是帮助实现编辑器中的自动完成支持。

和往常一样，源代码可以在 Github 上找到[。](https://web.archive.org/web/20220926200947/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-reflection)