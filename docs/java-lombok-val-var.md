# 在 Lombok 中声明 Val 和 Var 变量

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-lombok-val-var>

## 1.介绍

[Project Lombok](/web/20220524021036/https://www.baeldung.com/intro-to-project-lombok) 有助于减少 Java 对我们源代码中重复任务的冗长。在本教程中，我们将解释如何通过在 Lombok 中声明局部`val`和`var`变量来推断类型。

## 2.在龙目岛上声明`val`和`var`变量

**Lombok 提供智能功能来避免样板代码**。例如，它对域模型对象隐藏了[getter 和 setter](/web/20220524021036/https://www.baeldung.com/intro-to-project-lombok#constructors)。[构建器](/web/20220524021036/https://www.baeldung.com/lombok-builder)注释是另一个有趣的特性，有助于正确实现构建器[模式](/web/20220524021036/https://www.baeldung.com/creational-design-patterns#builder)。

在接下来的章节中，我们将重点关注**Lombok 特性来定义局部变量，而无需指定类型**。我们将使用 Lombok `val`和`var`类型来声明变量，并避免在源代码中出现多余的行。

[`val`](https://web.archive.org/web/20220524021036/https://projectlombok.org/features/val) 在 0.10 版本中推出。使用`val`时，Lombok 将变量声明为`final`，初始化后自动推断类型。因此，初始化表达式是强制性的。

`[var](https://web.archive.org/web/20220524021036/https://projectlombok.org/features/var)`在 1.16.20 版本中推出。和`val`一样，它也从初始化表达式中推断类型，最大的不同是变量没有声明为`final`。因此，允许进一步赋值，但它们应该符合声明变量时指定的类型。

## 3.在龙目岛上实现`val`和`var` 示例

### 3.1.属国

为了实现这些示例，我们只需将 [Lombok](https://web.archive.org/web/20220524021036/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.projectlombok%22) 依赖项添加到我们的`pom.xml`:

```
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.20</version>
    <scope>provided</scope>
</dependency>
```

我们可以在这里查看最新的可用版本[。](https://web.archive.org/web/20220524021036/https://projectlombok.org/changelog)

### 3.2.`val` 变量声明

首先，我们将从 Lombok 导入`val`类型:

```
import lombok.val;
```

其次，我们将使用`val`声明不同的局部变量。例如，我们可以从一个简单的`String`开始:

```
public Class name() {
    val name = "name";
    System.out.println("Name: " + name);
    return name.getClass();
}
```

Lombok 自动生成以下标准 Java:

```
final java.lang.String name = "name";
```

然后，让我们创建一个`Integer`:

```
public Class age() {
    val age = Integer.valueOf(30);
    System.out.println("Age: " + age);
    return age.getClass();
}
```

正如我们所看到的，Lombok 生成了正确的类型:

```
final java.lang.Integer age = Integer.valueOf(30);
```

我们还可以声明一个`List`:

```
public Class listOf() {
    val agenda = new ArrayList<String>();
    agenda.add("Day 1");
    System.out.println("Agenda: " + agenda);
    return agenda.getClass();
}
```

Lombok 不仅推断出了`List`，还推断出了其中的类型:

```
final java.util.ArrayList<java.lang.String> agenda = new ArrayList<String>();
```

现在，让我们创建一个`Map`:

```
public Class mapOf() {
    val books = new HashMap<Integer, String>();
    books.put(1, "Book 1");
    books.put(2, "Book 2");
    System.out.println("Books:");
    for (val entry : books.entrySet()) {
        System.out.printf("- %d. %s\n", entry.getKey(), entry.getValue());
    }
    return books.getClass();
}
```

同样，推断出正确的类型:

```
final java.util.HashMap<java.lang.Integer, java.lang.String> books = new HashMap<Integer, String>();
// ...
for (final java.util.Map.Entry<java.lang.Integer, java.lang.String> entry : books.entrySet()) {
   // ...
}
```

**我们可以看到 Lombok 将正确的类型声明为`final`** 。因此，如果我们试图修改名称，构建将会由于`val`的最终性质而失败:

```
name = "newName";

[12,9] cannot assign a value to final variable name
```

接下来，我们将运行一些测试来验证 Lombok 生成了正确的类型:

```
ValExample val = new ValExample();
assertThat(val.name()).isEqualTo(String.class);
assertThat(val.age()).isEqualTo(Integer.class);
assertThat(val.listOf()).isEqualTo(ArrayList.class);
assertThat(val.mapOf()).isEqualTo(HashMap.class);
```

最后，我们可以在控制台输出中看到特定类型的对象:

```
Name: name
Age: 30
Agenda: [Day 1]
Books:
- 1\. Book 1
- 2\. Book 2
```

### 3.3.`var `变量声明

`var`声明与`val`非常相似，区别在于变量不是`final`:

```
import lombok.var;

var name = "name";
name = "newName";

var age = Integer.valueOf(30);
age = 35;

var agenda = new ArrayList<String>();
agenda.add("Day 1");
agenda = new ArrayList<String>(Arrays.asList("Day 2"));

var books = new HashMap<Integer, String>();
books.put(1, "Book 1");
books.put(2, "Book 2");
books = new HashMap<Integer, String>();
books.put(3, "Book 3");
books.put(4, "Book 4");
```

让我们看看生成的普通 Java:

```
var name = "name";

var age = Integer.valueOf(30);

var agenda = new ArrayList<String>();

var books = new HashMap<Integer, String>();
```

这是因为 Java 10 支持`var` [声明](/web/20220524021036/https://www.baeldung.com/java-10-local-variable-type-inference)使用初始化式表达式来推断局部变量的类型。然而，在使用它时，我们需要考虑一些[约束](/web/20220524021036/https://www.baeldung.com/java-10-local-variable-type-inference#illegal-use-of-var)。

由于声明的变量不是`final`，我们可以做进一步的赋值。然而，**对象必须符合从初始化表达式**推断出的适当类型。

如果我们试图分配一个不同的类型，我们将在编译期间得到一个错误:

```
books = new ArrayList<String>();

[37,17] incompatible types: java.util.ArrayList<java.lang.String> cannot be converted to java.util.HashMap<java.lang.Integer,java.lang.String>
```

让我们稍微改变一下测试，检查一下新的作业:

```
VarExample varExample = new VarExample();
assertThat(varExample.name()).isEqualTo("newName");
assertThat(varExample.age()).isEqualTo(35);
assertThat("Day 2").isIn(varExample.listOf());
assertThat(varExample.mapOf()).containsValue("Book 3");
```

最后，控制台输出也不同于上一节:

```
Name: newName
Age: 35
Agenda: [Day 2]
Books:
- 3\. Book 3
- 4\. Book 4
```

## 4.复合类型

有些情况下，我们需要使用复合类型作为初始化表达式:

```
val compound = isArray ? new ArrayList<String>() : new HashSet<String>();
```

在上面的代码片段中，赋值依赖于布尔值，并推断出最常见的超类。

Lombok 将`AbstractCollection`指定为普通代码所示的类型:

```
final java.util.AbstractCollection<java.lang.String> compound = isArray ? new ArrayList<String>() : new HashSet<String>();
```

在不明确的情况下，例如使用`null`值，则推断出类别`Object`。

## 5.配置密钥

Lombok 允许[在整个项目中配置](https://web.archive.org/web/20220524021036/https://projectlombok.org/features/configuration)一个文件中的特性。因此，可以将项目的指令和设置放在一个地方。

有时，作为在我们的项目中执行开发标准的一部分，我们可能想要限制 Lombok 的`var`和`val`的使用。而且，如果有人无意中使用了它们，我们可能希望在编译期间生成一个警告。

对于这些情况，**我们可以通过在`lombok.config`文件**中包含以下内容，将`var`或`val`的任何用法标记为警告或错误:

```
lombok.var.flagUsage = error
lombok.val.flagUsage = warning
```

我们将收到一个关于在整个项目中非法使用`var`的错误:

```
[12,13] Use of var is flagged according to lombok configuration.
```

同样，我们会收到一条关于使用`val`的警告消息:

```
ValExample.java:18: warning: Use of val is flagged according to lombok configuration.
val age = Integer.valueOf(30); 
```

## 6.结论

在本文中，我们展示了如何使用 Lombok 在不指定类型的情况下定义局部变量。此外，我们学习了声明`val`和`var`变量的复杂性。

我们还演示了局部变量的泛型声明如何处理复合类型。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220524021036/https://github.com/eugenp/tutorials/tree/master/lombok-2)