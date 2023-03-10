# 如何替换 Java 中的许多 if 语句

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-replace-if-statements>

****

## 1。概述

决策结构是任何编程语言的重要组成部分。但是我们最终编码了大量嵌套的 if 语句，这使得我们的代码更加复杂和难以维护。

在本教程中，我们将介绍替换嵌套 if 语句的各种方法。

让我们探讨如何简化代码的不同选择。

## 2。案例研究

我们经常会遇到一个包含很多条件的业务逻辑，每个条件都需要不同的处理。为了便于演示，让我们以一个`Calculator`类为例。我们将有一个方法，它接受两个数字和一个运算符作为输入，并根据运算返回结果:

```java
public int calculate(int a, int b, String operator) {
    int result = Integer.MIN_VALUE;

    if ("add".equals(operator)) {
        result = a + b;
    } else if ("multiply".equals(operator)) {
        result = a * b;
    } else if ("divide".equals(operator)) {
        result = a / b;
    } else if ("subtract".equals(operator)) {
        result = a - b;
    }
    return result;
}
```

我们也可以使用`switch`语句`:`来实现这一点

```java
public int calculateUsingSwitch(int a, int b, String operator) {
    switch (operator) {
    case "add":
        result = a + b;
        break;
    // other cases    
    }
    return result;
}
```

在典型的开发中，**if 语句可能会变得更大、更复杂**。另外，**开关语句在有复杂条件的时候不太合适**。

嵌套决策结构的另一个副作用是它们变得难以管理。例如，如果我们需要添加一个新的操作符，我们必须添加一个新的 if 语句并实现该操作。

## 3。重构

让我们探索替代选项，将上面复杂的 if 语句替换成更简单、更易于管理的代码。

### 3.1。工厂类别

很多时候，我们会遇到最终在每个分支中执行相似操作的决策结构。这为**提取工厂方法提供了机会，该方法返回给定类型的对象，并基于具体的对象行为**执行操作。

对于我们的例子，让我们定义一个具有单个`apply`方法的`Operation`接口:

```java
public interface Operation {
    int apply(int a, int b);
}
```

该方法接受两个数字作为输入，并返回结果。让我们定义一个执行加法的类:

```java
public class Addition implements Operation {
    @Override
    public int apply(int a, int b) {
        return a + b;
    }
}
```

我们现在将实现一个工厂类，它基于给定的操作符返回`Operation` 的实例:

```java
public class OperatorFactory {
    static Map<String, Operation> operationMap = new HashMap<>();
    static {
        operationMap.put("add", new Addition());
        operationMap.put("divide", new Division());
        // more operators
    }

    public static Optional<Operation> getOperation(String operator) {
        return Optional.ofNullable(operationMap.get(operator));
    }
}
```

现在，在`Calculator`类中，我们可以查询工厂以获得相关的操作并应用于源编号:

```java
public int calculateUsingFactory(int a, int b, String operator) {
    Operation targetOperation = OperatorFactory
      .getOperation(operator)
      .orElseThrow(() -> new IllegalArgumentException("Invalid Operator"));
    return targetOperation.apply(a, b);
}
```

在这个例子中，我们看到了责任是如何委托给由工厂类服务的松散耦合对象的。但是有可能嵌套的 if 语句被简单地转移到工厂类，这违背了我们的目的。

或者，**我们可以在`Map` 中维护一个对象库，可以通过快速查找**对其进行查询。正如我们已经看到的`OperatorFactory#operationMap` 服务于我们的目的。我们还可以在运行时初始化`Map`,并配置它们用于查找。

### 3.2。枚举的使用

除了使用`Map,` **之外，我们还可以使用`Enum`来标记特定的业务逻辑**。之后，我们可以在嵌套的`if statements`或`switch case` `statements`中使用它们。或者，我们也可以将它们用作对象工厂，并对它们进行策略设计以执行相关的业务逻辑。

这也将减少嵌套 if 语句的数量，并将责任委托给各个`Enum`值。

让我们看看如何实现它。首先，我们需要定义我们的`Enum`:

```java
public enum Operator {
    ADD, MULTIPLY, SUBTRACT, DIVIDE
}
```

正如我们所观察到的，这些值是不同运算符的标签，将进一步用于计算。我们总是可以选择在嵌套的 if 语句或 switch cases 中使用值作为不同的条件，但是让我们设计一种替代的方式将逻辑委托给`Enum`本身。

我们将为每个`Enum`值定义方法并进行计算。例如:

```java
ADD {
    @Override
    public int apply(int a, int b) {
        return a + b;
    }
},
// other operators

public abstract int apply(int a, int b);
```

然后在`Calculator`类中，我们可以定义一个方法来执行操作:

```java
public int calculate(int a, int b, Operator operator) {
    return operator.apply(a, b);
}
```

现在，我们可以通过使用`Operator#valueOf()`方法将`String` 值转换为`Operator` 来调用方法:

```java
@Test
public void whenCalculateUsingEnumOperator_thenReturnCorrectResult() {
    Calculator calculator = new Calculator();
    int result = calculator.calculate(3, 4, Operator.valueOf("ADD"));
    assertEquals(7, result);
}
```

### 3.3。命令模式

在前面的讨论中，我们已经看到了使用工厂类为给定的操作符返回正确的业务对象的实例。稍后，业务对象用于执行`Calculator`中的计算。

**我们还可以设计`a Calculator#calculate`方法来接受一个可以在输入**上执行的命令。这将是替代嵌套`if statements`的另一种方式。

我们将首先定义我们的`Command`接口:

```java
public interface Command {
    Integer execute();
}
```

接下来，让我们实现一个`AddCommand:`

```java
public class AddCommand implements Command {
    // Instance variables

    public AddCommand(int a, int b) {
        this.a = a;
        this.b = b;
    }

    @Override
    public Integer execute() {
        return a + b;
    }
}
```

最后，让我们在`Calculator` 中引入一个新方法，它接受并执行`Command`:

```java
public int calculate(Command command) {
    return command.execute();
}
```

接下来，我们可以通过实例化一个`AddCommand`来调用计算，并将其发送给`Calculator#calculate`方法:

```java
@Test
public void whenCalculateUsingCommand_thenReturnCorrectResult() {
    Calculator calculator = new Calculator();
    int result = calculator.calculate(new AddCommand(3, 7));
    assertEquals(10, result);
}
```

### 3.4。规则引擎

当我们最终编写大量嵌套的 if 语句时，每一个条件都描述了一个业务规则，必须对其进行评估才能得到正确的逻辑处理。规则引擎将这种复杂性从主代码中去除。 **A `RuleEngine`对`Rules`求值并根据输入返回结果。**

让我们通过设计一个简单的`RuleEngine`来完成一个例子，这个简单的`RuleEngine`通过一组`Rules` 来处理一个`Expression`，并从选择的`Rule`返回结果。首先，我们将定义一个`Rule`接口:

```java
public interface Rule {
    boolean evaluate(Expression expression);
    Result getResult();
}
```

其次，让我们实现一个`RuleEngine`:

```java
public class RuleEngine {
    private static List<Rule> rules = new ArrayList<>();

    static {
        rules.add(new AddRule());
    }

    public Result process(Expression expression) {
        Rule rule = rules
          .stream()
          .filter(r -> r.evaluate(expression))
          .findFirst()
          .orElseThrow(() -> new IllegalArgumentException("Expression does not matches any Rule"));
        return rule.getResult();
    }
}
```

`RuleEngine` 接受一个`Expression`对象并返回`Result`。现在`,` 让我们将`Expression`类设计成一组两个`Integer`对象，并应用`Operator`:

```java
public class Expression {
    private Integer x;
    private Integer y;
    private Operator operator;        
}
```

最后，让我们定义一个自定义的`AddRule`类，它仅在指定了`ADD Operation`时才进行计算:

```java
public class AddRule implements Rule {
    @Override
    public boolean evaluate(Expression expression) {
        boolean evalResult = false;
        if (expression.getOperator() == Operator.ADD) {
            this.result = expression.getX() + expression.getY();
            evalResult = true;
        }
        return evalResult;
    }    
}
```

我们现在用一个`Expression`调用`RuleEngine`:

```java
@Test
public void whenNumbersGivenToRuleEngine_thenReturnCorrectResult() {
    Expression expression = new Expression(5, 5, Operator.ADD);
    RuleEngine engine = new RuleEngine();
    Result result = engine.process(expression);

    assertNotNull(result);
    assertEquals(10, result.getValue());
}
```

## 4。结论

在本教程中，我们探索了许多不同的选项来简化复杂的代码。我们还学习了如何通过使用有效的设计模式来替换嵌套的 if 语句。

和往常一样，我们可以在 [GitHub 库](https://web.archive.org/web/20220724011535/https://github.com/eugenp/tutorials/tree/master/patterns/design-patterns-creational)上找到完整的源代码。

****