# 使用 Java 通配符导入的优点和缺点

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-wildcard-imports>

## 1.概观

在本教程中，我们将讨论在 Java 中使用通配符导入的优点和缺点。

## 2.Java 中的导入

**Java `import`语句声明代码中使用的名字(类名、静态变量和方法名)的来源。**

举个例子，我们来看一个`Book`类:

```java
import java.util.Date;
import java.util.List;
import java.util.UUID;

public class Book {

    private UUID id;

    private String name;

    private Date datePublished;

    private List<String> authors;
}
```

这里我们需要导入`Date`和`UUID`两种数据类型以及`List`接口，因为默认情况下它们是不可用的。因此，我们编写了三个 import 语句，使这些数据类型可用于我们的类。让我们将这些类型的导入称为特定导入。

## 3.Java 通配符导入

**通配符导入指的是导入一个[包](https://web.archive.org/web/20220815140637/http://baeldung.com/java-packages)，而不是声明一个包中正在使用的特定类名。**

使用通配符，我们可以用一条语句来替换前面示例中的三条导入语句:

```java
import java.util.*;

public class Book {

    private UUID id;

    private String name;

    private Date datePublished;

    private List<String> authors;
}
```

这个通配符`import`语句将整个`java.util`包添加到搜索路径中，其中可以找到所需的名称`UUID`、`Date,`和`List`。

## 4.通配符导入的优势

自然，与 Java 中的特定导入相比，通配符导入有一些优势。让我们在下面的小节中讨论通配符导入的主要优点。

### 4.1.干净的代码

通配符导入帮助我们在代码中避免一长串的导入。因此，这会影响代码的可读性，因为读者可能不得不在到达显示逻辑的代码之前在每个源代码文件中滚动很多次。无疑，可读性更强的代码也是[干净的代码](https://web.archive.org/web/20220815140637/http://baeldung.com/java-clean-code)。

罗伯特·c·马丁的《T2》一书中也支持这个观点。事实上，这本书推荐在使用来自同一来源的多个类时使用通配符导入。**换句话说，当我们从一个包中导入两个或更多的类时，最好导入整个包。**

### 4.2.重构简易性

有了通配符导入，重构变得更加容易。例如，在重命名一个类时，我们不需要删除它所有特定的导入声明。

另外，如果我们将一个类从我们的一个包移动到我们自己的另一个包，如果通配符导入已经存在于两个包的文件中，我们不需要重构任何代码。

### 4.3.松耦合

通配符导入加强了现代软件开发中的[松耦合](https://web.archive.org/web/20220815140637/http://baeldung.com/cs/cohesion-vs-coupling)方法。

根据 Robert C. Martin 的说法，通配符导入的想法加强了松耦合。对于特定的导入，该类必须存在于包中。然而，使用通配符导入，特定的类不需要存在于包中。事实上，通配符导入将指定的包添加到搜索路径中，在这里可以搜索所需的类名。

因此，通配符风格的导入没有给包添加真正的依赖。

## 5.通配符导入的缺点

通配符导入也有其缺点。接下来，让我们看看通配符导入是如何导致一些问题的。

### 5.1.类名冲突

不幸的是，当一个类名出现在通过通配符导入的多个包中时，就会发生冲突。

在这种情况下，编译器注意到有两个`Date`类，并给出一个错误，因为在`java.sql`和`java.util`包中都找到了`Date`类:

```java
import java.util.*;
import java.sql.*;

public class Library {

    private UUID id;

    private String name;

    private Time openingTime;

    private Time closingTime;

    private List<Date> datesClosed;
}
```

为了防止这样的错误，我们可以指定冲突类的源。

为了防止上面例子中的错误，我们可以添加第三行来指定冲突的`Date`类的源到两个现有的导入中:

```java
import java.util.*;
import java.sql.*;
import java.sql.Date; 
```

### 5.2。不可预见的类名冲突

有趣的是，冲突也会随着时间的推移而出现，比如当一个类被添加到我们正在使用的另一个包的新版本中时。

例如，在 Java 1.1 中，`List`类只能在 [`java.awt`](/web/20220815140637/https://www.baeldung.com/java-images) 包中找到。然而，在 Java 1.2 中，名为`List`的接口被添加到了`java.util`包中。

我们来看一个例子:  

```java
import java.awt.*;
import java.util.*;

public class BookView extends Frame {

    private UUID id;

    private String name;

    private Date datePublished;

    private List<String> authors;
} 
```

最后，当`java.awt`和`java.util`包都作为通配符导入时，这种情况可能会导致冲突。**因此，当** **将代码迁移到更新的** **Java 版本时，我们可能会面临潜在的问题。**

## 6.结论

In this article, we discussed `import` statements in Java and what wildcard imports are. We learned the advantages and disadvantages of using wildcard imports in our programs.Usage of wildcard imports vs. specific imports remains a popular debate in the Java community. **In brief,** **we can say that the wildcard import approach has advantages, but its usage can cause problems in certain situations.**As always, the source code for the examples is available [over on GitHub](https://web.archive.org/web/20220815140637/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-5/).