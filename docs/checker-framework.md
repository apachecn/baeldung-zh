# Checker 框架——面向 Java 的可插拔类型系统

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/checker-framework>

## 1。概述

从`Java 8`版本开始，有可能使用所谓的 [`Pluggable Type Systems`](https://web.archive.org/web/20220628084955/https://docs.oracle.com/javase/tutorial/java/annotations/type_annotations.html) 来编译程序——这可以应用比编译器应用的检查更严格的检查。

我们只需要使用几个可用的`Pluggable Type Systems`提供的注释。

在这篇简短的文章中，我们将探索由华盛顿大学提供的`[the Checker Framework](https://web.archive.org/web/20220628084955/https://checkerframework.org/)`。

## 2。肚子

为了开始使用 Checker 框架，我们需要首先将它添加到我们的`pom.xml:`

```
<dependency>
    <groupId>org.checkerframework</groupId>
    <artifactId>checker-qual</artifactId>
    <version>2.3.2</version>
</dependency>
<dependency>
    <groupId>org.checkerframework</groupId>
    <artifactId>checker</artifactId>
    <version>2.3.2</version>
</dependency>
<dependency>
    <groupId>org.checkerframework</groupId>
    <artifactId>jdk8</artifactId>
    <version>2.3.2</version>
</dependency>
```

可以在 [Maven Central](https://web.archive.org/web/20220628084955/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.checkerframework%22%20AND%20a%3A%22checker%22) 上查看最新版本的库。

前两个依赖项包含了`The Checker Framework`的代码，而后者是`Java 8` 类的定制版本，其中的所有类型都已经被`The Checker Framework`的开发者正确地注释了。

然后我们必须适当地调整`maven-compiler-plugin`来使用`The Checker Framework`作为可插拔的`Type System`:

```
<plugin>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.6.1</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <compilerArguments>
            <Xmaxerrs>10000</Xmaxerrs>
            <Xmaxwarns>10000</Xmaxwarns>
        </compilerArguments>
        <annotationProcessors>
            <annotationProcessor>
                org.checkerframework.checker.nullness.NullnessChecker
            </annotationProcessor>
            <annotationProcessor>
                org.checkerframework.checker.interning.InterningChecker
            </annotationProcessor>
            <annotationProcessor>
                org.checkerframework.checker.fenum.FenumChecker
            </annotationProcessor>
            <annotationProcessor>
                org.checkerframework.checker.formatter.FormatterChecker
            </annotationProcessor>
        </annotationProcessors>
        <compilerArgs>
            <arg>-AprintErrorStack</arg>
            <arg>-Awarns</arg>
        </compilerArgs>
    </configuration>
</plugin>
```

这里的要点是`<annotationProcessors>`标签的内容。在这里，我们列出了我们想要对我们的源运行的所有检查器。

## 3。避免 nullpointerexception

`The Checker Framework`可以帮助我们的第一个场景是识别`NullPoinerException`可能源自的那段代码:

```
private static int countArgs(@NonNull String[] args) {
    return args.length;
}

public static void main(@Nullable String[] args) {
    System.out.println(countArgs(args));
}
```

在上面的例子中，我们用`@NonNull`注释声明了`countArgs()`的`args`参数不能为空。

不考虑这个约束，在`main()`中，我们调用传递一个参数的方法，这个参数实际上可以是 null，因为它已经用`@Nullable`进行了注释。

当我们编译代码时，`The Checker Framework`适时地警告我们代码中可能有错误:

```
[WARNING] /checker-plugin/.../NonNullExample.java:[12,38] [argument.type.incompatible]
 incompatible types in argument.
  found   : null
  required: @Initialized @NonNull String @Initialized @NonNull []
```

## 4。常量作为枚举的正确使用

有时我们使用一系列常数，因为它们是枚举的项目。

假设我们需要一系列的国家和星球。然后，我们可以用`@Fenum`注释对这些项进行注释，将属于同一个“假”枚举的所有常量分组:

```
static final @Fenum("country") String ITALY = "IT";
static final @Fenum("country") String US = "US";
static final @Fenum("country") String UNITED_KINGDOM = "UK";

static final @Fenum("planet") String MARS = "Mars";
static final @Fenum("planet") String EARTH = "Earth";
static final @Fenum("planet") String VENUS = "Venus";
```

在此之后，当我们编写一个应该接受一个“planet”字符串的方法时，我们可以适当地注释这个参数:

```
void greetPlanet(@Fenum("planet") String planet){
    System.out.println("Hello " + planet);
}
```

错误的是，我们可以用一个没有被定义为行星的可能值的字符串调用`greetPlanet()`,比如:

```
public static void main(String[] args) {
    obj.greetPlanets(US);
}
```

`The Checker Framework`能发现错误:

```
[WARNING] /checker-plugin/.../FakeNumExample.java:[29,26] [argument.type.incompatible]
 incompatible types in argument.
  found   : @Fenum("country") String
  required: @Fenum("planet") String
```

## 5。正则表达式

假设我们知道一个`String`变量必须存储一个至少有一个匹配组的正则表达式。

我们可以利用`the Checker Framework`并声明这样的变量:

```
@Regex(1) private static String FIND_NUMBERS = "\\d*";
```

这显然是一个潜在的错误，因为我们分配给`FIND_NUMBERS`的正则表达式没有任何匹配组。

的确，`the Checker Framework`会在编译时努力通知我们我们的错误:

```
[WARNING] /checker-plugin/.../RegexExample.java:[7,51] [assignment.type.incompatible]
incompatible types in assignment.
  found   : @Regex String
  required: @Regex(1) String
```

## 6。结论

对于希望超越标准编译器并提高代码正确性的开发人员来说,`The Checker Framework`是一个有用的工具。

它能够在编译时检测到几个通常只能在运行时检测到的典型错误，甚至通过引发编译错误来停止编译。

除了本文中介绍的内容之外，还有很多标准检查；点击查看`The Checker Framework`官方手册[中可用的检查，甚至编写自己的检查。](https://web.archive.org/web/20220628084955/https://checkerframework.org/manual/)

和往常一样，本教程的源代码，以及更多的例子，可以在 GitHub 上找到[。](https://web.archive.org/web/20220628084955/https://github.com/eugenp/tutorials/tree/master/checker-plugin/)