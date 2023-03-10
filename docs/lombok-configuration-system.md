# 龙目岛配置系统

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/lombok-configuration-system>

## 1.介绍

在本教程中，我们将讨论 Lombok 的配置参数。我们将讨论许多不同的选项，以及如何正确设置我们的配置。

## 2.配置概述

Lombok 是一个帮助我们消除 Java 应用程序中几乎所有标准样板文件的库。我们将测试许多属性和配置。第一件事是添加[龙目岛](https://web.archive.org/web/20220627165851/https://search.maven.org/search?q=a:lombok%20AND%20g:%20org.projectlombok)属地:

```java
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.20</version>
    <scope>provided</scope>
</dependency>
```

Lombok 的配置系统为我们提供了许多有价值的设置，这些设置在我们项目的所有组件中都是相同的。然而，它也让我们改变或定制 Lombok 的行为，有时甚至定义所有可用功能中哪些可以使用，哪些不可以使用。例如，如果使用了任何实验特性，我们可以告诉 Lombok 显示警告或错误。

**要开始定义或定制 Lombok 的行为，我们必须创建一个名为** `**lombok.config.**`的文件。这个文件可以放在我们的项目、源代码或任何包的根目录下。一旦创建，子目录中的所有源文件都将继承在这样的文件中定义的配置。有可能有多个配置文件。例如，我们可以在根目录中定义一个带有通用属性的配置文件，并在给定的包中创建另一个定义其他属性的文件。

新配置将影响给定包的所有类和所有子包。此外，在同一属性有多个定义的情况下，更接近类或成员的定义优先。

## 3.基本配置

首先要提到的一件事是，要讨论的特性属性太多了。出于这个原因，我们将只看到最常见的。要检查可用选项，让我们转到 [Lombok 的页面](https://web.archive.org/web/20220627165851/https://projectlombok.org/download)，下载 jar，并在终端中运行以下命令:

```java
java -jar lombok.jar config -g --verbose 
```

因此，我们将看到所有属性及其可能值的完整列表，以及解释其目标的简短描述。

现在，让我们来看一个典型的`lombok.config`文件:

```java
config.stopBubbling = true
lombok.anyconstructor.addconstructorproperties = false
lombok.addLombokGeneratedAnnotation = true
lombok.experimental.flagUsage = WARNING

# ... more properties 
```

文件中使用的属性仅用于说明目的。我们稍后将讨论它们。但是在这里，我们可以观察到 Lombok 属性的格式及其定义。

**让我们从`config.stopBubbling `属性开始——这个选项告诉配置系统不要在父目录**中搜索配置文件。将该属性添加到您的工作区或项目的根目录是一个很好的做法。默认情况下，其值为`false`。

## 4.主要属性

### 4.1.全局配置键

**全局配置键是可能影响许多配置系统本身的配置**。接下来，我们将看到一些这样的键的例子。

我们要讨论的第一个关键是 `lombok.anyConstructor.addConstructorProperties.`，它将`@java.beans.ConstructorProperties`注释添加到所有带参数的构造函数中。通常，在构造函数上使用反射的框架需要这个注释来映射属性，并知道构造函数中参数的正确顺序。下面是 Lombok 版本的代码:

```java
@AllArgsConstructor
@NoArgsConstructor
@Getter
@Setter
public class Account {
    private double balance;
    private String accountHolder;
}
```

下面是生成的代码:

```java
public class Account {
    private double balance;
    private String accountHolder;

    @ConstructorProperties({"balance", "accountHolder"})
    @Generated
    public Account(double balance, String accountHolder) {
        this.balance = balance;
        this.accountHolder = accountHolder;
    }

    @Generated
    public Account() {}

    // default generated getter and setters
}
```

在上面的代码片段中，我们可以看到生成的包含`@ConstructorProperties`注释的类。

接下来，我们有了**` lombok.addLombokGeneratedAnnotation.`If`true`，Lombok 会用`@lombok.Generated.` 标记所有生成的方法，这在从包扫描或代码覆盖工具**中移除 Lombok 生成的方法时会派上用场。

另一个有用的关键是 `lombok.addNullAnnotations.`这个属性支持许多内置选项，比如 javax (JSR305)、eclipse、JetBrains、NetBeans、Android 等等。也可以使用我们自己定义的注释，比如`CUSTOM:com.example.NonNull:example.Nullable`。只要有意义，Lombok 就会添加`nullable`或`NotNull`注释。

最后，我们有`lombok.addSuppressWarnings,` ，如果`false`，Lombok 停止添加注释`@SuppressWarnings(“all”),`，这是当前的默认行为。如果我们对生成的代码使用静态分析器，这是很有用的。

### 4.2.其他配置键

作为第一个特性特定的键 `lombok.accessors.chain,`，如果`true`，改变 setter 方法的行为。这些方法将返回`this`，而不是`void` 返回`,`。允许链接呼叫，如下所示:

```java
@Test
void should_initialize_account() {
    Account myAccount = new Account()
      .setBalance(2000.00)
      .setAccountHolder("John Snow");

    assertEquals(2000.00, myAccount.getBalance());
    assertEquals("John Snow", myAccount.getAccountHolder());
}
```

与前一个类似， `lombok.accessors.fluent`让 Lombok 从访问器方法中删除前缀`set`和`get`，只使用属性名来命名它们。

用户配置时，`lombok.log.fieldName`键改变生成日志字段的名称。默认情况下，`lombok.log.fieldName`键使用`log`来命名字段，但是在我们的例子中，我们将其改为`domainLog`:

```java
#Log name customization
lombok.log.fieldName = domainLog
```

然后我们可以看到它的运行:

```java
@AllArgsConstructor
@NoArgsConstructor
@Getter
@Setter
@Log
public class Account {

    // same as defined previously

   public Account withdraw(double amount) {
        if (this.balance - abs(amount) < 0) {
            domainLog.log(Level.INFO, "Transaction denied for account holder: %s", this.accountHolder);
            throw new IllegalArgumentException(String.format("Not enough balance, you have %.2f", this.balance));
        }

        this.balance -= abs(amount);
        return this;
    }
}
```

接下来是`lombok.(featureName).flagUsage.` 这组属性有`warning`、`error,`和`allow` 作为可能的值。我们可以使用它们来控制在我们的项目中允许哪些 Lombok 特性。例如，如果使用了任何实验特征，可以使用单词`experimental`和值`warning`在日志中输出一条消息:

```java
/home/dev/repository/git/tutorials/lombok/src/main/java/com/baeldung/lombok/configexamples/TransactionLog.java:9:
 warning: Use of any lombok.experimental feature is flagged according to lombok configuration.
@Accessors(prefix = {"op"})
```

### 4.3.特殊配置键

有些键不是公共的键值属性，比如 `lombok.copyableAnnotations.` 这个属性是不同的，因为它代表了完全限定的注释类型的列表。当添加到一个字段中时，Lombok 会将这些注释复制到与该字段相关的构造函数、getters 和 setters 中。**该特性的一个典型用例是 Spring 的 bean 定义，其中注释`@Qualifier`和`@Value`经常需要被复制到构造函数参数**中。其他框架也利用了这一特性。

要给列表添加注释，用户必须使用下面的表达式:`lombok.copyableAnnotations` `+= com.test.MyAnnotation`。本库使用这种机制来传播前面提到的可空注释:

```java
@NoArgsConstructor
@AllArgsConstructor
@Getter
@Setter
@Log
public class Account {

    @NonNull
    private Double balance = 0.;
    @NonNull
    private String accountHolder = "";

    // other methods
}
```

现在，由 Lombok 生成的代码:

```java
public class Account {

    @Generated
    private static final Logger domainLog = Logger.getLogger(Account.class.getName());
    @NonNull
    private Double balance = 0.0D;
    @NonNull
    private String accountHolder = "";

    @ConstructorProperties({"balance", "accountHolder"})
    @Generated
    public Account(@NonNull Double balance, @NonNull String accountHolder) {
        if (balance == null) {
            throw new NullPointerException("balance is marked non-null but is null");
        } else if (accountHolder == null) {
            throw new NullPointerException("accountHolder is marked non-null but is null");
        } else {
            this.balance = balance;
            this.accountHolder = accountHolder;
        }
    }

    @NonNull
    @Generated
    public Double getBalance() {
        return this.balance;
    }

    @NonNull
    @Generated
    public String getAccountHolder() {
        return this.accountHolder;
    }

    // Rest of the class members...

}
```

最后，我们有一个`clear lombok.(anyConfigKey)` 指令。将任何配置密钥恢复为默认值。如果有人更改了任何父配置文件中给定键的值，它现在将被忽略。我们可以使用指令`clear,`，后跟任何 Lombok 配置键:

```java
clear lombok.addNullAnnotations
```

### 4.4.文件指令

现在，我们对 Lombok 的配置系统是如何工作的以及它的一些特性有了一个很好的了解。当然，这并不是所有可用特性的详尽列表，但是从这里开始，我们必须清楚地了解如何使用它。最后但同样重要的是，让我们看看如何将另一个文件中定义的配置导入到我们当前的配置文件中。

**将一个配置文件导入另一个文件时，指令必须放在文件的顶部，路径可以是相对的，也可以是绝对的**:

```java
##     relative or absolute path  
import lombok_feature.config

config.stopBubbling = true
lombok.anyconstructor.addconstructorproperties=false
lombok.addLombokGeneratedAnnotation = true
lombok.addSuppressWarnings = false 
```

为了便于说明，导入的文件:

```java
# lombok_feature.config file

lombok.experimental.flagUsage = warning
```

## 5.结论

在本文中，我们研究了 Lombok 的配置系统，它的不同属性，以及它们如何影响它的功能。尽管如前所述，还有很多选项可用，但我们只研究了最常见的选项。请随意在 [Lombok 页面](https://web.archive.org/web/20220627165851/https://projectlombok.org/features/configuration)查看更多信息。

像往常一样，本文中使用的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220627165851/https://github.com/eugenp/tutorials/tree/master/lombok-modules/lombok)