# 字符串格式中的命名占位符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-formatting-named-placeholders>

## 1.概观

Java 标准库提供了 [`String.format()`](/web/20220831130238/https://www.baeldung.com/string/format) 方法来格式化一个基于模板的字符串，比如:`String.format(“%s is awesome”, “Java”)`。

在本教程中，我们将探索如何使字符串格式支持命名参数。

## 2.问题简介

`String.format()`方法使用起来非常简单。然而，当`format()`调用有许多参数时，很难理解哪个值将属于哪个格式说明符，例如:

```java
Employee e = ...; // get an employee instance
String template = "Firstname: %s, Lastname: %s, Id: %s, Company: %s, Role: %s, Department: %s, Address: %s ...";
String.format(template, e.firstName, e.lastName, e.Id, e.company, e.department, e.role ... ) 
```

此外，当我们将这些参数传递给方法时，很容易出错。例如，在上面的例子中，我们错误地将`e.department`放在了`e.role`的前面。

因此，如果我们可以在模板中使用命名参数之类的东西，然后通过保存所有参数映射的`Map`应用格式，那就太好了:

```java
String template = "Firstname: ${firstname}, Lastname: ${lastname}, Id: ${id} ...";
ourFormatMethod.format(template, parameterMap);
```

在本教程中，我们将首先看一个使用流行的外部库的解决方案，它可以解决这个问题的大多数情况。然后，我们将讨论一个打破解决方案的边缘案例。

最后，我们将创建自己的`format()`方法来涵盖所有情况。

为了简单起见，我们将使用单元测试断言来验证一个方法是否返回预期的字符串。

同样值得一提的是**在本教程**中，我们将只关注简单的字符串格式(`%s`)。不支持其他格式类型，如日期、数字或具有定义的宽度和精度的格式。

## 3.使用 Apache Commons 文本中的`StrSubstitutor`

[Apache Commons Text](/web/20220831130238/https://www.baeldung.com/java-apache-commons-text) 库包含许多使用字符串的实用工具。它附带了`StrSubstitutor`，允许我们基于命名参数进行字符串替换。

首先，让我们将这个库作为新的依赖项添加到我们的 Maven 配置文件中:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-text</artifactId>
    <version>1.9</version>
</dependency>
```

当然，我们总能在 Maven 中央存储库中找到最新版本的。

在我们看到如何使用`StrSubstitutor`类之前，让我们创建一个模板作为例子:

```java
String TEMPLATE = "Text: [${text}] Number: [${number}] Text again: [${text}]";
```

接下来，让我们创建一个测试，使用`StrSubstitutor`基于上面的模板构建一个字符串:

```java
Map<String, Object> params = new HashMap<>();
params.put("text", "It's awesome!");
params.put("number", 42);
String result = StrSubstitutor.replace(TEMPLATE, params, "${", "}");
assertThat(result).isEqualTo("Text: [It's awesome!] Number: [42] Text again: [It's awesome!]"); 
```

如测试代码所示，我们让`params`保存所有的`name -> value`映射。当我们调用`StrSubstitutor.replace()`方法时，**除了`template`和`params,` 之外，我们还传递前缀和后缀来通知`StrSubstitutor`模板**中的参数由什么组成。`StrSubstitutor`将在`prefix + map.entry.key + suffix`中搜索参数名称。

当我们运行测试时，它通过了。于是，似乎`StrSubstitutor`解决了问题。

## 4.边缘情况:当替换包含占位符时

我们已经看到`StrSubstitutor.replace()`测试通过了我们的基本用例。但是，一些特殊情况不在测试范围内。例如，参数值可能包含参数名称模式“`${ … }`”。

现在，让我们测试这个案例:

```java
Map<String, Object> params = new HashMap<>();
params.put("text", "'${number}' is a placeholder.");
params.put("number", 42);
String result = StrSubstitutor.replace(TEMPLATE, params, "${", "}");

assertThat(result).isEqualTo("Text: ['${number}' is a placeholder.] Number: [42] Text again: ['${number}' is a placeholder.]"); 
```

在上面的测试中，参数“`${text}`”的值包含文本“`${number}`”。因此，我们期望"`${text}`"被文本"`${number}`"逐字替换。

但是，如果我们执行它，测试就会失败:

```java
org.opentest4j.AssertionFailedError: 
expected: "Text: ['${number}' is a placeholder.] Number: [42] Text again: ['${number}' is a placeholder.]"
 but was: "Text: ['42' is a placeholder.] Number: [42] Text again: ['42' is a placeholder.]"
```

因此，`StrSubstitutor`也将文字`${number}`视为参数占位符。

事实上，`StrSubstitutor`的 Javadoc 已经陈述了这个案例:

> 变量替换以递归方式工作。因此，如果变量值包含一个变量，那么该变量也将被替换。

这是因为，**在每个递归步骤中，`StrSubstitutor`将最后一个替换结果作为新的`template`继续进行进一步的替换**。

为了绕过这个问题，我们可以选择不同的前缀和后缀，这样它们就不会受到干扰:

```java
String TEMPLATE = "Text: [%{text}] Number: [%{number}] Text again: [%{text}]";
Map<String, Object> params = new HashMap<>();
params.put("text", "'${number}' is a placeholder.");
params.put("number", 42);
String result = StrSubstitutor.replace(TEMPLATE, params, "%{", "}");

assertThat(result).isEqualTo("Text: ['${number}' is a placeholder.] Number: [42] Text again: ['${number}' is a placeholder.]"); 
```

然而，从理论上讲，由于我们不能预测值，所以值总是可能包含参数名模式并干扰替换。

接下来，让我们创建自己的`format()`方法来解决问题。

## 5.构建我们自己的格式化程序

我们已经讨论了为什么`StrSubstitutor`不能很好地处理边缘情况。所以，如果我们创建一个方法，困难在于**我们不应该使用循环或递归来将上一步的结果作为当前步骤**的新输入。

### 5.1.解决问题的想法

想法是我们在模板中搜索参数名模式。然而，当我们找到一个时，我们不会立即用地图上的值替换它。相反，我们构建了一个新的模板，可以用于标准的`String.format()`方法。以我们的例子为例，我们将尝试转换:

```java
String TEMPLATE = "Text: [${text}] Number: [${number}] Text again: [${text}]";
Map<String, Object> params ...
```

变成:

```java
String NEW_TEMPLATE = "Text: [%s] Number: [%s] Text again: [%s]";
List<Object> valueList = List.of("'${number}' is a placeholder.", 42, "'${number}' is a placeholder.");
```

然后，我们可以调用`String.format(NEW_TEMPLATE, valueList.toArray());`来完成任务。

### 5.2.创建方法

接下来，让我们创建一个方法来实现这个想法:

```java
public static String format(String template, Map<String, Object> parameters) {
    StringBuilder newTemplate = new StringBuilder(template);
    List<Object> valueList = new ArrayList<>();

    Matcher matcher = Pattern.compile("[$][{](\\w+)}").matcher(template);

    while (matcher.find()) {
        String key = matcher.group(1);

        String paramName = "${" + key + "}";
        int index = newTemplate.indexOf(paramName);
        if (index != -1) {
            newTemplate.replace(index, index + paramName.length(), "%s");
            valueList.add(parameters.get(key));
        }
    }

    return String.format(newTemplate.toString(), valueList.toArray());
} 
```

上面的代码非常简单。让我们快速浏览一下，了解它是如何工作的。

首先，我们声明了两个新变量来保存新模板(`newTemplate`)和值列表(`valueList`)。我们以后给`String.format()` 打电话时会用到它们。

我们使用`[Regex](/web/20220831130238/https://www.baeldung.com/regular-expressions-java)`来定位模板中的参数名模式。然后，我们用`“%s”`替换参数名模式，并将相应的值添加到`valueList`变量中。

最后，我们用新转换的模板和来自`valueList.`的值调用`String.format()`

为了简单起见，我们在方法中硬编码了前缀“`${`”和后缀“`}`”。另外，**如果没有提供参数“`${unknown}`”的值，我们将简单地用“`null`”**替换“`${unknown}`”参数。

### 5.3.测试我们的`format()`方法

接下来，让我们测试该方法是否适用于常规情况:

```java
Map<String, Object> params = new HashMap<>();
params.put("text", "It's awesome!");
params.put("number", 42);
String result = NamedFormatter.format(TEMPLATE, params);
assertThat(result).isEqualTo("Text: [It's awesome!] Number: [42] Text again: [It's awesome!]"); 
```

同样，如果我们试一试，测试就会通过。

当然，我们也想看看它是否适用于边缘情况:

```java
params.put("text", "'${number}' is a placeholder.");
result = NamedFormatter.format(TEMPLATE, params);
assertThat(result).isEqualTo("Text: ['${number}' is a placeholder.] Number: [42] Text again: ['${number}' is a placeholder.]"); 
```

如果我们执行这个测试，它也会通过！我们已经解决了这个问题。

## 6.结论

在本文中，我们探讨了如何从一组值中替换基于模板的字符串中的参数。基本上，Apache Commons Text 的`StrSubstitutor.replace()`方法使用起来非常简单，可以解决大多数情况。但是，当值包含参数名模式时，`StrSubstitutor`可能会产生意外的结果。

因此，我们实现了一个`format()`方法来解决这种边缘情况。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220831130238/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-4)