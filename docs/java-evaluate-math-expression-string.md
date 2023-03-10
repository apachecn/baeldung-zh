# 在 Java 中计算数学表达式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-evaluate-math-expression-string>

## 1.概观

在本教程中，我们将讨论使用 Java 计算数学表达式的各种方法。在我们希望计算字符串格式的数学表达式的项目中，这个特性会派上用场。

首先，我们将讨论一些第三方库及其用法。接下来，我们将看到如何使用内置的 Java 脚本 API 来完成这项任务。

## 2.exp4j

[exp4j](https://web.archive.org/web/20220630132713/https://www.objecthunter.net/exp4j/) 是一个开源库，可以用来评估数学表达式和函数。该库实现了 [Dijkstra 的调车场算法，](https://web.archive.org/web/20220630132713/https://en.wikipedia.org/wiki/Shunting-yard_algorithm)一种解析用[中缀符号](https://web.archive.org/web/20220630132713/https://en.wikipedia.org/wiki/Infix_notation "Infix notation")指定的数学表达式的方法。

除了使用标准的操作符和函数，exp4j 还允许我们创建定制的操作符和函数。

### 2.1.添加 Maven 依赖项

要使用 exp4j，我们需要在项目中添加 [Maven 依赖项](https://web.archive.org/web/20220630132713/https://search.maven.org/search?q=g:net.objecthunter%20a:exp4j):

```java
<dependency>
    <groupId>net.objecthunter</groupId>
    <artifactId>exp4j</artifactId>
    <version>0.4.8</version>
</dependency>
```

### 2.2.评估简单表达式

我们可以评估一个以`String`格式提供的简单数学表达式:

```java
@Test
public void givenSimpleExpression_whenCallEvaluateMethod_thenSuccess() {
    Expression expression = new ExpressionBuilder("3+2").build();
    double result = expression.evaluate();
    Assertions.assertEquals(5, result);
}
```

在上面的代码片段中，我们首先创建了一个`ExpressionBuilder`的实例。然后我们将它赋给一个`Expression`引用，我们用它来评估我们的表达式。

### 2.3.在表达式中使用变量

现在我们知道了如何计算简单的表达式，让我们给表达式添加一些变量:

```java
@Test
public void givenTwoVariables_whenCallEvaluateMethod_thenSuccess() {
    Expression expression = new ExpressionBuilder("3x+2y")
      .variables("x", "y")
      .build()
      .setVariable("x", 2)
      .setVariable("y", 3);

    double result = expression.evaluate();

    Assertions.assertEquals(12, result);
}
```

在上面的例子中，我们使用`variables`方法引入了两个变量`x,`和`y,`。**使用这种方法，我们可以在表达式中添加任意多的变量**。一旦我们声明了变量，我们就可以使用`setVariable`方法给它们赋值。

### 2.4.评估包含数学函数的表达式

现在让我们来看一个简短的例子，看看如何计算一些标准的数学函数:

```java
@Test
public void givenMathFunctions_whenCallEvaluateMethod_thenSuccess() {
    Expression expression = new ExpressionBuilder("sin(x)*sin(x)+cos(x)*cos(x)")
      .variables("x")
      .build()
      .setVariable("x", 0.5);

    double result = expression.evaluate();

    Assertions.assertEquals(1, result);
}
```

## 3.Java 求值程序

Java evaluator 是另一个独立的轻量级库，免费提供。像 exp4j 一样，Javaluator 也是**用于计算中缀表达式**。

### 3.1.添加 Maven 依赖项

我们可以使用下面的 [Maven 依赖关系](https://web.archive.org/web/20220630132713/https://search.maven.org/search?q=g:com.fathzer%20a:javaluator)在我们的项目中使用 Javaluator:

```java
<dependency>
    <groupId>com.fathzer</groupId>
    <artifactId>javaluator</artifactId>
    <version>3.0.3</version>
</dependency>
```

### 3.2.评估简单表达式

要使用 Javaluator 计算表达式，我们首先需要创建一个`DoubleEvaluator`的实例:

```java
@Test
public void givenExpression_whenCallEvaluateMethod_thenSuccess() {
    String expression = "3+2";
    DoubleEvaluator eval = new DoubleEvaluator();

    Double result = eval.evaluate(expression);

    Assertions.assertEquals(5, result);
}
```

### 3.3.评估包含变量的表达式

为了计算包含变量的表达式，我们使用`StaticVariableSet`:

```java
@Test
public void givenVariables_whenCallEvaluateMethod_thenSuccess() {
    String expression = "3*x+2*y";
    DoubleEvaluator eval = new DoubleEvaluator();
    StaticVariableSet<Double> variables = new StaticVariableSet<Double>();
    variables.set("x", 2.0);
    variables.set("y", 3.0);

    Double result = eval.evaluate(expression, variables);

    Assertions.assertEquals(12, result);
}
```

然后我们使用`StaticVariableSet#set`方法给变量赋值。

### 3.4.评估包含数学函数的表达式

我们还可以使用 Javaluator 库求解包含数学函数的表达式:

```java
@Test
public void givenMathFunction_whenCallEvaluateMethod_thenSuccess() {
    String expression = "sin(x)*sin(x)+cos(x)*cos(x)";
    DoubleEvaluator eval = new DoubleEvaluator();
    StaticVariableSet<Double> variables = new StaticVariableSet<Double>();
    variables.set("x", 0.5);

    Double result = eval.evaluate(expression, variables);

    Assertions.assertEquals(1, result);
}
```

## 4.Java 脚本 API

既然我们已经讨论了第三方库，现在让我们讨论如何使用内置 API 来实现这一点。Java 已经提供了一个小而强大的脚本 API。这个 API 的所有类和接口都在`javax.script`包里。

它包含了允许我们评估 JavaScript 的`ScriptEngineManager` 和`ScriptEngine`接口。在 Java 8 之前，Java 带有`Rhino`引擎。然而，从 Java 8 开始，Java 就有了更新更强大的 [Nashorn](/web/20220630132713/https://www.baeldung.com/java-nashorn) 引擎。

### 4.1.正在获取`ScriptEngine`实例

要创建一个`ScriptEngine`，我们首先要创建一个`ScriptEngineManager`的实例。一旦我们有了实例，我们需要调用`ScriptEngineManager#getEngineByName`方法来获得`ScriptEngine`:

```java
ScriptEngineManager scriptEngineManager = new ScriptEngineManager();
ScriptEngine scriptEngine = scriptEngineManager.getEngineByName("JavaScript");
```

**注意，`Nashorn`是 JDK 附带的默认 JavaScript 引擎。**

### 4.2.评估简单表达式

我们现在可以使用上面的`scriptEngine`实例来调用`ScriptEngine#eval`方法:

```java
String expression = "3+2";
Integer result = (Integer) scriptEngine.eval(expression);
Assertions.assertEquals(5, result);
```

### 4.3.评估包含变量的表达式

为了计算包含变量的表达式，我们需要声明和初始化变量:

```java
String expression = "x=2; y=3; 3*x+2*y;";
Double result = (Double) scriptEngine.eval(expression);
Assertions.assertEquals(12, result);
```

由于我们使用的是 JavaScript 引擎，**我们可以像在 JavaScript** 中一样直接向表达式添加变量。

**注意–JavaScript 没有执行数学运算的直接方法，需要访问`Math`对象。因此，我们不能使用 Java 脚本 API 来求解数学表达式。
**

## 5.结论

在本文中，我们看到了使用 Java 计算数学表达式的各种技术。

和往常一样，这个例子的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220630132713/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-math-3)