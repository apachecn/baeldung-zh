# Spring Expression 语言指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-expression-language>

## 1。概述

Spring 表达式语言(SpEL)是一种强大的表达式语言，支持在运行时查询和操作对象图。我们可以将它用于 XML 或基于注释的 Spring 配置。

语言中有几个可用的运算符:

| **类型** | **操作员** |
| --- | --- |
| 算术 | +，-，*，/，%, ^，div，mod |
| 有关系的 | 、==、！=<=, >=【lt，gt，eq，ne，le，ge】 |
| 逻辑学的 | and，or，not，&&，&#124;&#124;，！ |
| 有条件的 | ？： |
| 正则表达式 | 比赛 |

## 2。操作员

对于这些例子，我们将使用基于注释的配置。在本文后面的小节中可以找到关于 XML 配置的更多细节。

SpEL 表达式以符号`#`开始，用大括号括起来:`#{expression}`。

属性可以以类似的方式引用，以符号`$`开始，用大括号括起来:`${property.name}`。

属性占位符不能包含 SpEL 表达式，但表达式可以包含属性引用:

```
#{${someProperty} + 2}
```

在上面的例子中，假设`someProperty`的值为 2，那么得到的表达式将是 2 + 2，其值将为 4。

### 2.1。算术运算符

SpEL 支持所有基本算术运算符:

```
@Value("#{19 + 1}") // 20
private double add; 

@Value("#{'String1 ' + 'string2'}") // "String1 string2"
private String addString; 

@Value("#{20 - 1}") // 19
private double subtract;

@Value("#{10 * 2}") // 20
private double multiply;

@Value("#{36 / 2}") // 19
private double divide;

@Value("#{36 div 2}") // 18, the same as for / operator
private double divideAlphabetic; 

@Value("#{37 % 10}") // 7
private double modulo;

@Value("#{37 mod 10}") // 7, the same as for % operator
private double moduloAlphabetic; 

@Value("#{2 ^ 9}") // 512
private double powerOf;

@Value("#{(2 + 2) * 2 + 9}") // 17
private double brackets; 
```

除法和模运算都有字母别名，`/`用`div`，`%`用`mod`。`+`操作符也可以用来连接字符串。

### 2.2。关系和逻辑运算符

SpEL 还支持所有基本的关系和逻辑操作:

```
@Value("#{1 == 1}") // true
private boolean equal;

@Value("#{1 eq 1}") // true
private boolean equalAlphabetic;

@Value("#{1 != 1}") // false
private boolean notEqual;

@Value("#{1 ne 1}") // false
private boolean notEqualAlphabetic;

@Value("#{1 < 1}") // false
private boolean lessThan;

@Value("#{1 lt 1}") // false
private boolean lessThanAlphabetic;

@Value("#{1 <= 1}") // true
private boolean lessThanOrEqual;

@Value("#{1 le 1}") // true
private boolean lessThanOrEqualAlphabetic;

@Value("#{1 > 1}") // false
private boolean greaterThan;

@Value("#{1 gt 1}") // false
private boolean greaterThanAlphabetic;

@Value("#{1 >= 1}") // true
private boolean greaterThanOrEqual;

@Value("#{1 ge 1}") // true
private boolean greaterThanOrEqualAlphabetic; 
```

所有关系运算符都有字母别名。例如，在基于 XML 的配置中，我们不能使用包含尖括号的操作符(`<`、`<=`、`>`、`>=`)。而是可以用`lt`(小于)、`le`(小于等于)、`gt`(大于)或者`ge`(大于等于)。

### 2.3。逻辑运算符

SpEL 还支持所有基本的逻辑操作:

```
@Value("#{250 > 200 && 200 < 4000}") // true
private boolean and; 

@Value("#{250 > 200 and 200 < 4000}") // true
private boolean andAlphabetic;

@Value("#{400 > 300 || 150 < 100}") // true
private boolean or;

@Value("#{400 > 300 or 150 < 100}") // true
private boolean orAlphabetic;

@Value("#{!true}") // false
private boolean not;

@Value("#{not true}") // false
private boolean notAlphabetic;
```

与算术和关系运算符一样，所有逻辑运算符也有字母克隆。

### 2.4。条件运算符

我们使用条件运算符根据某些条件注入不同的值:

```
@Value("#{2 > 1 ? 'a' : 'b'}") // "a"
private String ternary;
```

我们使用三元运算符在表达式中执行紧凑的 if-then-else 条件逻辑。在这个例子中，我们试图检查是否有`true`。

三元运算符的另一个常见用途是检查某个变量是否为`null`，然后返回变量值或默认值:

```
@Value("#{someBean.someProperty != null ? someBean.someProperty : 'default'}")
private String ternary;
```

Elvis 操作符是 Groovy 语言中使用的三进制操作符语法的一种简化形式。它也有 SpEL 版本。

这段代码相当于上面的代码:

```
@Value("#{someBean.someProperty ?: 'default'}") // Will inject provided string if someProperty is null
private String elvis;
```

### 2.5。在 SpEL 中使用正则表达式

我们可以使用 `matches`操作符来检查一个字符串是否匹配给定的正则表达式:

```
@Value("#{'100' matches '\\d+' }") // true
private boolean validNumericStringResult;

@Value("#{'100fghdjf' matches '\\d+' }") // false
private boolean invalidNumericStringResult;

@Value("#{'valid alphabetic string' matches '[a-zA-Z\\s]+' }") // true
private boolean validAlphabeticStringResult;

@Value("#{'invalid alphabetic string #$1' matches '[a-zA-Z\\s]+' }") // false
private boolean invalidAlphabeticStringResult;

@Value("#{someBean.someValue matches '\d+'}") // true if someValue contains only digits
private boolean validNumericValue;
```

### 2.6。访问`List`和`Map`对象

在 SpEL 的帮助下，我们可以访问上下文中任何`Map`或`List`的内容。

我们将创建新的 bean `workersHolder`，它将在`List`和`Map`中存储一些工人及其工资的信息:

```
@Component("workersHolder")
public class WorkersHolder {
    private List<String> workers = new LinkedList<>();
    private Map<String, Integer> salaryByWorkers = new HashMap<>();

    public WorkersHolder() {
        workers.add("John");
        workers.add("Susie");
        workers.add("Alex");
        workers.add("George");

        salaryByWorkers.put("John", 35000);
        salaryByWorkers.put("Susie", 47000);
        salaryByWorkers.put("Alex", 12000);
        salaryByWorkers.put("George", 14000);
    }

    //Getters and setters
}
```

现在我们可以使用 SpEL 来访问集合的值:

```
@Value("#{workersHolder.salaryByWorkers['John']}") // 35000
private Integer johnSalary;

@Value("#{workersHolder.salaryByWorkers['George']}") // 14000
private Integer georgeSalary;

@Value("#{workersHolder.salaryByWorkers['Susie']}") // 47000
private Integer susieSalary;

@Value("#{workersHolder.workers[0]}") // John
private String firstWorker;

@Value("#{workersHolder.workers[3]}") // George
private String lastWorker;

@Value("#{workersHolder.workers.size()}") // 4
private Integer numberOfWorkers;
```

## 3。在弹簧配置中使用

### 3.1。引用一个 Bean

在这个例子中，我们将看看如何在基于 XML 的配置中使用 SpEL。我们可以使用表达式来引用 bean 或 bean 字段/方法。

例如，假设我们有以下类:

```
public class Engine {
    private int capacity;
    private int horsePower;
    private int numberOfCylinders;

   // Getters and setters
}

public class Car {
    private String make;
    private int model;
    private Engine engine;
    private int horsePower;

   // Getters and setters
}
```

现在，我们创建一个应用程序上下文，其中表达式用于注入值:

```
<bean id="engine" class="com.baeldung.spring.spel.Engine">
   <property name="capacity" value="3200"/>
   <property name="horsePower" value="250"/>
   <property name="numberOfCylinders" value="6"/>
</bean>
<bean id="someCar" class="com.baeldung.spring.spel.Car">
   <property name="make" value="Some make"/>
   <property name="model" value="Some model"/>
   <property name="engine" value="#{engine}"/>
   <property name="horsePower" value="#{engine.horsePower}"/>
</bean>
```

看一看`someCar`豆。`someCar`的`engine`和`horsePower` 字段使用的表达式分别是对`engine` bean 和`horsePower`字段的 bean 引用。

要对基于注释的配置进行同样的操作，请使用`@Value(“#{expression}”)` 注释。

### 3.2。在配置中使用运算符

本文第一部分中的每个操作符都可以在基于 XML 和注释的配置中使用。

但是，请记住，在基于 XML 的配置中，我们不能使用尖括号运算符“lt(小于)或`le`(小于或等于)。

对于基于注释的配置，没有这样的限制:

```
public class SpelOperators {
    private boolean equal;
    private boolean notEqual;
    private boolean greaterThanOrEqual;
    private boolean and;
    private boolean or;
    private String addString;

    // Getters and setters
```

```
 @Override
    public String toString() {
        // toString which include all fields
    }
```

现在我们将向应用程序上下文添加一个`spelOperators` bean:

```
<bean id="spelOperators" class="com.baeldung.spring.spel.SpelOperators">
   <property name="equal" value="#{1 == 1}"/>
   <property name="notEqual" value="#{1 lt 1}"/>
   <property name="greaterThanOrEqual" value="#{someCar.engine.numberOfCylinders >= 6}"/>
   <property name="and" value="#{someCar.horsePower == 250 and someCar.engine.capacity lt 4000}"/>
   <property name="or" value="#{someCar.horsePower > 300 or someCar.engine.capacity > 3000}"/>
   <property name="addString" value="#{someCar.model + ' manufactured by ' + someCar.make}"/>
</bean>
```

从上下文中检索该 bean，然后我们可以验证值是否被正确注入:

```
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
SpelOperators spelOperators = (SpelOperators) context.getBean("spelOperators"); 
```

这里我们可以看到`spelOperators` bean 的`toString`方法的输出:

```
[equal=true, notEqual=false, greaterThanOrEqual=true, and=true, 
or=true, addString=Some model manufactured by Some make] 
```

## 4。以编程方式解析表达式

有时，我们可能希望在配置上下文之外解析表达式。幸运的是，使用`SpelExpressionParser`这是可能的。

我们可以使用我们在前面的例子中看到的所有操作符，但是应该使用不带括号和散列符号的操作符。也就是说，如果我们想在 Spring 配置中使用带有`+` 操作符的表达式，语法是`#{1 + 1}`；当在配置之外使用时，语法就是简单的`1 + 1`。

在下面的例子中，我们将使用上一节中定义的`Car`和`Engine`bean。

### 4.1。使用`ExpressionParser`

让我们看一个简单的例子:

```
ExpressionParser expressionParser = new SpelExpressionParser();
Expression expression = expressionParser.parseExpression("'Any string'");
String result = (String) expression.getValue(); 
```

`ExpressionParser` 负责解析表达式字符串。在这个例子中，SpEL 解析器将简单地把字符串`‘Any String'`作为一个表达式来计算。不出所料，结果将是`‘Any String'`。

如同在配置中使用 SpEL 一样，我们可以用它来调用方法、访问属性或调用构造函数:

```
Expression expression = expressionParser.parseExpression("'Any string'.length()");
Integer result = (Integer) expression.getValue();
```

此外，我们可以调用构造函数，而不是直接对文本进行操作:

```
Expression expression = expressionParser.parseExpression("new String('Any string').length()");
```

我们也可以用同样的方式访问`String`类的`bytes`属性，得到字符串的 byte[]表示:

```
Expression expression = expressionParser.parseExpression("'Any string'.bytes");
byte[] result = (byte[]) expression.getValue();
```

我们可以链接方法调用，就像在普通 Java 代码中一样:

```
Expression expression = expressionParser.parseExpression("'Any string'.replace(\" \", \"\").length()");
Integer result = (Integer) expression.getValue();
```

在这种情况下，结果将是 9，因为我们已经用空字符串替换了空白。

如果我们不想强制转换表达式结果，我们可以使用泛型方法`T getValue(Class<T> desiredResultType)`，在这个方法中我们可以提供我们想要返回的类的类型。

请注意，如果返回值不能强制转换为`desiredResultType`，则会抛出`EvaluationException`:

```
Integer result = expression.getValue(Integer.class);
```

最常见的用法是提供根据特定对象实例计算的表达式字符串:

```
Car car = new Car();
car.setMake("Good manufacturer");
car.setModel("Model 3");
car.setYearOfProduction(2014);

ExpressionParser expressionParser = new SpelExpressionParser();
Expression expression = expressionParser.parseExpression("model");

EvaluationContext context = new StandardEvaluationContext(car);
String result = (String) expression.getValue(context);
```

在这种情况下，结果将等于`car`对象的`model`字段的值，“`Model 3`”。`StandardEvaluationContext`类指定表达式将针对哪个对象进行计算。

创建上下文对象后，不能对其进行更改。`StandardEvaluationContext` 的构建成本很高，在重复使用期间，它会建立缓存状态，使后续的表达式求值能够更快地执行。由于缓存的原因，如果根对象没有改变，在可能的情况下重用`StandardEvaluationContext` 是一个好的做法。

但是，如果根对象被重复更改，我们可以使用下面示例中所示的机制:

```
Expression expression = expressionParser.parseExpression("model");
String result = (String) expression.getValue(car);
```

这里我们用一个参数调用`getValue` 方法，这个参数代表我们想要应用 SpEL 表达式的对象。

我们也可以使用通用的`getValue`方法，就像之前一样:

```
Expression expression = expressionParser.parseExpression("yearOfProduction > 2005");
boolean result = expression.getValue(car, Boolean.class);
```

### 4.2。使用`ExpressionParser`设定一个值

对解析表达式返回的`Expression`对象使用`setValue`方法，我们可以设置对象的值。SpEL 将负责类型转换。默认情况下，SpEL 使用`org.springframework.core.convert.ConversionService`。我们可以在类型之间创建自己的自定义转换器。`ConversionService`是泛型感知的，所以我们可以和泛型一起使用。

让我们看看我们在实践中是如何做到的:

```
Car car = new Car();
car.setMake("Good manufacturer");
car.setModel("Model 3");
car.setYearOfProduction(2014);

CarPark carPark = new CarPark();
carPark.getCars().add(car);

StandardEvaluationContext context = new StandardEvaluationContext(carPark);

ExpressionParser expressionParser = new SpelExpressionParser();
expressionParser.parseExpression("cars[0].model").setValue(context, "Other model");
```

产生的汽车对象将有`model``Other model`，它是由`Model 3`改变而来的。

### 4.3。解析器配置

在下面的例子中，我们将使用这个类:

```
public class CarPark {
    private List<Car> cars = new ArrayList<>();

    // Getter and setter
}
```

可以通过调用带有`SpelParserConfiguration`对象`.` 的构造函数来配置`ExpressionParser`

例如，如果我们试图在没有配置解析器的情况下将`car`对象添加到`CarPark`类的`cars`数组中，我们将得到如下错误:

```
EL1025E:(pos 4): The collection has '0' elements, index '0' is invalid
```

我们可以改变解析器的行为，允许它在指定的索引为空时自动创建元素(`autoGrowNullReferences`，构造函数的第一个参数)，或者自动增长数组或列表以容纳超出其初始大小的元素(`autoGrowCollections`，第二个参数):

```
SpelParserConfiguration config = new SpelParserConfiguration(true, true);
StandardEvaluationContext context = new StandardEvaluationContext(carPark);

ExpressionParser expressionParser = new SpelExpressionParser(config);
expressionParser.parseExpression("cars[0]").setValue(context, car);

Car result = carPark.getCars().get(0);
```

产生的`car`对象将等于`car`对象，该对象被设置为前面示例中`carPark`对象的`cars`数组的第一个元素。

## 5。结论

SpEL 是一种强大的、得到良好支持的表达式语言，我们可以在 Spring 产品组合中的所有产品上使用它。我们可以用它来配置 Spring 应用程序，或者编写解析器来在任何应用程序中执行更一般的任务。

本文中的代码示例可以在[链接的 GitHub 存储库](https://web.archive.org/web/20220706111627/https://github.com/eugenp/tutorials/tree/master/spring-spel)中找到。