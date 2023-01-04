# Spring @Value 快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-value-annotation>

## 1.概观

在这个快速教程中，我们将**看一下`@Value` Spring 注释。**

该注释可用于将值注入 Spring 管理的 beans 中的字段，并且可以在字段或构造函数/方法参数级别应用。

## 延伸阅读:

## [什么是春豆？](/web/20220625164441/https://www.baeldung.com/spring-bean)

A quick and practical explanation of what a Spring Bean is.[Read more](/web/20220625164441/https://www.baeldung.com/spring-bean) →

## [使用带有默认值的 Spring @ Value】](/web/20220625164441/https://www.baeldung.com/spring-value-defaults)

A quick and practical guide to setting default values when using the @Value annotation in Spring.[Read more](/web/20220625164441/https://www.baeldung.com/spring-value-defaults) →

## 2.设置应用程序

为了描述这个注释的不同用法，我们需要配置一个简单的 Spring 应用程序配置类。

自然地，**我们将需要一个属性文件**来定义我们想要用`@Value`注释注入的值。因此，我们首先需要在我们的配置类中定义一个`@PropertySource` ——用属性文件名。

让我们定义属性文件:

```
value.from.file=Value got from the file
priority=high
listOfValues=A,B,C
```

## 3.用法示例

作为一个基本的、几乎没有用的例子，我们只能将注释中的“字符串值”注入到字段中:

```
@Value("string value")
private String stringValue;
```

使用`@PropertySource` 注释允许我们使用来自带有`@Value` 注释的属性文件的值。

在下面的例子中，我们将`Value got from the file`分配给该字段:

```
@Value("${value.from.file}")
private String valueFromFile;
```

我们也可以使用相同的语法从系统属性中设置该值。

假设我们已经定义了一个名为`systemValue`的系统属性:

```
@Value("${systemValue}")
private String systemValue;
```

可以为可能未定义的属性提供默认值。这里，值`some default`将被注入:

```
@Value("${unknown.param:some default}")
private String someDefault;
```

如果在属性文件中将同一属性定义为系统属性，则系统属性将被应用。

假设我们有一个属性`priority`被定义为一个值为`System property`的系统属性，并被定义为属性文件中的其他内容。该值将是`System property`:

```
@Value("${priority}")
private String prioritySystemProperty;
```

有时候，我们需要注入一堆价值观。将它们定义为属性文件中的单个属性的逗号分隔值，或者定义为系统属性并注入到数组中会很方便。

在第一部分中，我们在属性文件`,`的`listOfValues`中定义了逗号分隔的值，因此数组值将是`[“A”, “B”, “C”]:`

```
@Value("${listOfValues}")
private String[] valuesArray;
```

## 4.SpEL 的高级示例

我们还可以使用 SpEL 表达式来获取值。

如果我们有一个名为`priority,`的系统属性，那么它的值将应用于该字段:

```
@Value("#{systemProperties['priority']}")
private String spelValue;
```

如果我们没有定义系统属性，那么`null` 值将被赋值。

为了防止这种情况，我们可以在 SpEL 表达式中提供一个默认值。如果未定义系统属性，我们将获得字段的`some default` 值:

```
@Value("#{systemProperties['unknown'] ?: 'some default'}")
private String spelSomeDefault;
```

此外，我们可以使用来自其他 beans 的字段值。假设我们有一个名为`someBean`的 bean，其字段`someValue`等于`10`。然后，`10`将被分配到字段:

```
@Value("#{someBean.someValue}")
private Integer someBeanValue;
```

我们可以操纵属性来获得一个`List`值，这里是字符串值 A、B 和 C 的列表:

```
@Value("#{'${listOfValues}'.split(',')}")
private List<String> valuesList;
```

## 5.使用`@Value`和`Maps`

我们还可以使用`@Value`注释来注入一个`Map`属性。

首先，我们需要在属性文件的`{key: ‘value' }` 表单中定义属性:

```
valuesMap={key1: '1', key2: '2', key3: '3'}
```

**注意，`Map`中的值必须用单引号括起来。**

现在我们可以从属性文件中注入这个值作为`Map`:

```
@Value("#{${valuesMap}}")
private Map<String, Integer> valuesMap;
```

如果我们需要**来获取`Map`中特定键**的值，我们所要做的就是**在表达式**中添加键的名称:

```
@Value("#{${valuesMap}.key1}")
private Integer valuesMapKey1;
```

如果我们不确定`Map`是否包含某个键，我们应该选择**一个更安全的表达式，它不会抛出异常，而是在找不到键时将值设置为`null`** :

```
@Value("#{${valuesMap}['unknownKey']}")
private Integer unknownMapKey;
```

我们还可以**为可能不存在的属性或键设置默认值**:

```
@Value("#{${unknownMap : {key1: '1', key2: '2'}}}")
private Map<String, Integer> unknownMap;

@Value("#{${valuesMap}['unknownKey'] ?: 5}")
private Integer unknownMapKeyWithDefaultValue;
```

**`Map`条目也可以在注入前过滤**。

假设我们只需要获取那些值大于 1 的条目:

```
@Value("#{${valuesMap}.?[value>'1']}")
private Map<String, Integer> valuesMapFiltered;
```

我们也可以使用`@Value`注释来**注入所有当前系统属性**:

```
@Value("#{systemProperties}")
private Map<String, String> systemPropertiesMap;
```

## 6.通过构造函数注入使用`@Value`

当我们使用`@Value`注释时，我们并不局限于字段注入。**我们也可以和构造函数注入一起使用。**

让我们看看实际情况:

```
@Component
@PropertySource("classpath:values.properties")
public class PriorityProvider {

    private String priority;

    @Autowired
    public PriorityProvider(@Value("${priority:normal}") String priority) {
        this.priority = priority;
    }

    // standard getter
}
```

在上面的例子中，我们将一个`priority`直接注入到我们的`PriorityProvider`的构造函数中。

请注意，我们还提供了一个默认值，以防找不到该属性。

## 7.使用`@Value`和 Setter 注入

类似于构造函数注入，**我们也可以将`@Value`用于 setter 注入。**

让我们来看看:

```
@Component
@PropertySource("classpath:values.properties")
public class CollectionProvider {

    private List<String> values = new ArrayList<>();

    @Autowired
    public void setValues(@Value("#{'${listOfValues}'.split(',')}") List<String> values) {
        this.values.addAll(values);
    }

    // standard getter
}
```

我们使用 SpEL 表达式将一系列值注入到`setValues`方法中。

## 8.结论

在本文中，我们研究了将`@Value`注释用于文件中定义的简单属性、系统属性以及用 SpEL 表达式计算的属性的各种可能性。

与往常一样，这个示例应用程序可以在 [GitHub 项目](https://web.archive.org/web/20220625164441/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-properties-2)上获得。