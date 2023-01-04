# Lombok 的@ToString 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/lombok-tostring>

## 1.概观

我们知道， [`toString()`](/web/20220925203315/https://www.baeldung.com/java-tostring) 方法用于获取 Java 对象的字符串表示。

Project Lombok 可以帮助我们生成一致的字符串表示，而无需样板文件和混乱的源代码。它还可以提高可维护性，尤其是在类可能包含大量字段的情况下。

在本教程中，我们将看到如何自动生成该方法以及各种可用的配置选项，以进一步微调结果输出。

## 2.设置

让我们首先在我们的示例项目中包含[项目 Lombok](https://web.archive.org/web/20220925203315/https://search.maven.org/artifact/org.projectlombok/lombok) 依赖项:

```
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.22</version>
    <scope>provided</scope>
</dependency>
```

在我们的示例中，我们将使用一个简单的`Account` POJO 类和几个字段来演示功能和各种配置选项。

## 3.基本用法

我们可以用 Lombok `@ToString`注释来注释任何类。这将修改生成的字节码，并创建一个`toString()`方法的实现。

让我们将这个注释应用到简单的 POJO:

```
@ToString
public class Account {

    private String id;

    private String name;

    // standard getters and setters
}
```

默认情况下，**`@ToString`注释打印类名，以及每个非静态字段名及其通过调用 getter(如果声明的话)**获得的值。**这些字段也根据源类**中的声明顺序出现。逗号分隔不同的字段值对。

现在，对该类的一个实例调用`toString()`方法会生成以下输出:

```
Account(id=12345, name=An account) 
```

在大多数情况下，这足以生成 Java 对象的标准且有用的字符串表示。

## 4.配置选项

有几个配置选项允许我们修改和调整生成的`toString()`方法。这些在某些用例中会有所帮助。让我们更详细地看看这些。

### 4.1.超类`toString()`

默认情况下，输出不包含来自`toString()`方法的超类实现的数据。然而，**我们可以通过将`callSuper`属性值设置为** `**true**:`来修改它

```
@ToString(callSuper = true)
public class SavingAccount extends Account {

    private String savingAccountId;

    // standard getters and setters
}
```

这将产生以下输出，超类信息后跟子类字段和值:

```
SavingAccount(super=Account(id=12345, name=An account), savingAccountId=6789)
```

**重要的是，只有当我们扩展一个类而不是`java.lang.Object`** 时，这才是真正有益的。`toString()`的`Object`实现没有提供太多有用的信息。换句话说，包含这些数据只会增加冗余信息，并增加输出的详细程度。

### 4.2.省略字段名称

正如我们前面看到的，默认输出包含字段名称，后跟值。然而，**我们可以通过在`@ToString`注释**中将`includeFieldNames`属性设置为`false`来从输出中省略字段名称:

```
@ToString(includeFieldNames = false)
public class Account {

    private String id;

    private String name;

    // standard getters and setters
}
```

因此，输出现在显示了所有字段值的逗号分隔列表，但没有字段名:

```
Account(12345, An account)
```

### 4.3.使用字段而不是 Getters

正如我们已经看到的，getter 方法为打印提供了字段值。此外，如果该类不包含特定字段的 getter 方法，那么 Lombok 将直接访问该字段并获取其值。

然而，**我们可以通过将`doNotUseGetters`属性设置为`true`** 来配置 Lombok 总是使用直接字段值而不是 getters:

```
@ToString(doNotUseGetters = true)
public class Account {

    private String id;

    private String name;

    // ignored getter
    public String getId() {
        return "this is the id:" + id;
    }

    // standard getters and setters
}
```

如果没有这个属性，我们将通过调用 getters 得到输出:

```
Account(id=this is the id:12345, name=An account)
```

相反，**具有`doNotUseGetters`属性，输出实际上显示了`id`字段的值，而没有调用 getter** :

```
Account(id=12345, name=An account)
```

### 4.4.字段包含和排除

假设我们想从字符串表示中排除某些字段，例如密码、其他敏感信息或大型 JSON 结构。**我们可以简单地通过用`@ToString.Exclude `注释**来注释这些字段来省略它们。

让我们从表示中排除`name`字段:

```
@ToString
public class Account {

    private String id;

    @ToString.Exclude
    private String name;

    // standard getters and setters
}
```

**或者，我们可以只指定输出**中需要的字段。让我们通过在类级别使用`@ToString(onlyExplicitlyIncluded = true)`来实现这一点，然后用`@ToString.Include`注释每个必填字段:

```
@ToString(onlyExplicitlyIncluded = true)
public class Account {

    @ToString.Include
    private String id;

    private String name;

    // standard getters and setters
}
```

上述两种方法仅使用`id`字段产生以下输出:

```
Account(id=12345)
```

此外， **Lombok 输出自动排除任何以`$`符号**开头的变量。但是，我们可以覆盖这种行为，通过在字段级别添加`@ToString.Include`注释来包含它们。

### 4.5.订购输出

默认情况下，输出根据类中的声明顺序包含字段。然而，**我们可以简单地通过给`@ToString.Include`注释**添加 rank 属性来调整排序。

让我们修改我们的`Account`类，使`id`字段呈现在任何其他字段之前，而不管声明在类定义中的位置。我们可以通过向`id`字段添加`@ToString.Include(rank = 1)`注释来实现这一点:

```
@ToString
public class Account {

    private String name;

    @ToString.Include(rank = 1)
    private String id;

    // standard getters and setters
}
```

现在，`id`字段在输出中首先呈现，尽管它在`name`字段之后声明:

```
Account(id=12345, name=An account)
```

输出首先包含较高等级的成员，随后是较低等级的成员。没有等级属性的成员的默认等级值为 0。具有相同等级的成员根据其申报顺序打印。

### 4.6.方法输出

除了字段之外，还可以包含不带参数的实例方法的输出。**我们可以通过用`@ToString.Include`** 标记无参数实例方法来做到这一点:

```
@ToString
public class Account {

    private String id;

    private String name;

    @ToString.Include
    String description() {
        return "Account description";
    }

    // standard getters and setters
}
```

这将把`description`作为键，并将其输出作为值添加到`Account`表示中:

```
Account(id=12345, name=An account, description=Account description)
```

**如果指定的方法名与字段名匹配，则该方法优先于字段**。换句话说，输出包含方法调用的结果，而不是匹配的字段值。

### 4.7.修改字段名称

我们可以通过在`@ToString.Include`注释的`name`属性中指定不同的值来更改任何字段名:

```
@ToString
public class Account {

    @ToString.Include(name = "identification")
    private String id;

    private String name;

    // standard getters and setters
}
```

现在，输出包含来自注释属性的替代字段名称，而不是实际的字段名称:

```
Account(identification=12345, name=An account)
```

## 5.打印阵列

使用 [`Arrays.deepToString()`](/web/20220925203315/https://www.baeldung.com/java-util-arrays#1tostring-anddeeptostring) 方法打印数组。**这将数组元素转换成它们相应的字符串表示**。但是，数组可能包含直接引用或间接循环引用。

为了避免无限递归及其相关的运行时错误，该方法将数组内部的任何循环引用都呈现为`“[[…]]”.`

让我们通过向我们的`Account`类添加一个`Object`数组字段来看看这一点:

```
@ToString
public class Account {

    private String id;

    private Object[] relatedAccounts;

    // standard getters and setters
}
```

`relatedAccounts`数组现在包含在输出中:

```
Account(id=12345, relatedAccounts=[54321, [...]])
```

重要的是，循环引用由`deepToString()`方法检测，并由 Lombok 适当渲染，不会引起任何 [`StackOverflowError`](/web/20220925203315/https://www.baeldung.com/java-stack-overflow-error) 。

## 6.要记住的要点

有几个细节值得一提，它们对于避免意外结果非常重要。

**在类中存在任何名为`toString()`的方法时(无论返回类型如何)，Lombok 不生成其`toString()`方法**。

**不同版本的 Lombok 可能会改变生成方法**的输出格式。在任何情况下，我们都应该避免依赖解析`toString()`方法输出的代码。所以这应该不是什么问题。

最后，**我们还可以在`enum` s** 上添加这个注释。这产生了一个表示，其中`enum`值跟在`enum`类名之后，例如`AccounType.SAVING`。

## 7.结论

在本文中，我们看到了如何使用 Lombok 注释以最少的工作量和样板文件生成 Java 对象的`String`表示。

最初，我们查看了基本用法，这通常足以满足大多数情况。然后，我们讨论了可以用来调整和优化生成的输出的各种选项。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220925203315/https://github.com/eugenp/tutorials/tree/master/lombok-modules/lombok-2)