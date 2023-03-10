# IntelliJ IDEA 重构简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/intellij-refactoring>

## 1.概观

保持代码整洁并不总是容易的。幸运的是，如今我们的 IDEs 相当智能，可以帮助我们实现这一目标。在本教程中，我们将重点介绍 IntelliJ IDEA，即 JetBrains Java 代码编辑器。

我们将看到编辑器为[重构代码](/web/20221205113308/https://www.baeldung.com/cs/refactoring)提供的一些特性，从重命名变量到改变方法签名。

## 2.重新命名

### 2.1.基本重命名

先从基础说起:[改名](https://web.archive.org/web/20221205113308/https://www.jetbrains.com/help/idea/rename-dialogs.html)。 **IntelliJ 为我们提供了重命名代码中不同元素的可能性:类型、变量、方法，甚至是包。**

要重命名元素，我们需要遵循以下步骤:

*   右键单击该元素
*   **触发`Refactor > Rename`选项**
*   键入新元素名称
*   按下`Enter`

顺便说一下，**我们可以通过选择元素并按下`Shift + F6`来替换前两步。**

当被触发时，重命名动作将**在代码中搜索元素的每次使用，然后用提供的值**改变它们。

让我们想象一个`SimpleClass`类，它有一个命名不当的加法方法`someAdditionMethod`，在`main`方法中被调用:

```java
public class SimpleClass {
    public static void main(String[] args) {
        new SimpleClass().someAdditionMethod(1, 2);
    }

    public int someAdditionMethod(int a, int b) {
        return a + b;
    }
}
```

因此，如果我们选择将此方法重命名为`add`，IntelliJ 将生成以下代码:

```java
public class SimpleClass() {
    public static void main(String[] args) {
        new SimpleClass().add(1, 2);
    }

    public int add(int a, int b) {
        return a + b;
    }
}
```

### 2.2.高级重命名

然而，IntelliJ 做的不仅仅是搜索元素的代码用法并重命名它们。事实上，还有更多的选择。IntelliJ **还可以搜索注释和字符串中的出现，甚至是不包含源代码**的文件中的出现。至于参数，在方法被覆盖的情况下，它可以在类层次结构中重命名它们。

在重命名我们的元素之前，再次点击`Shift + F6`可以使用这些选项，将会出现一个弹出窗口:

[![renaming](img/aefc478e91b91ca95ed291eb73846d65.png)](/web/20221205113308/https://www.baeldung.com/wp-content/uploads/2019/04/renaming.png)

`Search in comments and strings`选项可用于任何重命名。至于`Search for text occurrences`选项，它不适用于方法参数和局部变量。最后，`Rename parameters in hierarchy`选项仅适用于方法参数。

因此，**如果发现与这两个选项中的一个匹配，IntelliJ 将显示它们，并为我们提供退出某些更改的可能性**(比如，如果它匹配与我们的重命名无关的内容)。

让我们向我们的方法添加一些 Javadoc，然后重命名它的第一个参数`a`:

```java
/**
  * Adds a and b
  * @param a the first number
  * @param b the second number
  */
public int add(int a, int b) {...}
```

通过检查确认弹出窗口中的第一个选项，IntelliJ 匹配方法的 Javadoc 注释中提到的任何参数，并提供对它们的重命名:

```java
/**
  * Adds firstNumber and b
  * @param firstNumber the first number
  * @param b the second number
  */
public int add(int firstNumber, int b) {...}
```

最后，我们必须注意到 **IntelliJ 是智能的，它主要在重命名元素的范围内搜索用法。**在我们的例子中，这意味着位于方法之外(Javadoc 除外)并且包含对`a`的引用的注释不会被考虑重命名。

## 3.提取

现在，我们来谈谈[提取](https://web.archive.org/web/20221205113308/https://www.jetbrains.com/help/idea/extract-dialogs.html)。**提取使我们能够抓取一段代码，并将其放入一个变量、一个方法甚至一个类中。** IntelliJ 非常聪明地处理了这一点，它搜索相似的代码片段，并提供以同样的方式提取它们。

因此，在这一节中，我们将学习如何利用 IntelliJ 提供的提取特性。

### 3.1.变量

首先，让我们从变量提取开始。这意味着局部变量、参数、字段和常量。要提取一个变量，我们必须遵循以下步骤:

*   选择适合变量的表达式
*   右键单击选定的区域
*   **触发`Refactor > Extract > Variable/Parameter/Field/Constant`选项**
*   如有建议，在`Replace this occurrence only`或`Replace all x occurrences`选项之间选择
*   为提取的表达式输入一个名称(如果选择的名称不适合我们)
*   按下`Enter`

至于重命名，可以使用键盘快捷键来代替菜单。**默认的快捷键分别是`Ctrl + Alt + V, Ctrl + Alt + P, Ctrl + Alt + F`和`Ctrl + Alt + C`。**

IntelliJ 将根据表达式返回的内容，尝试猜测我们提取的表达式的名称。如果它不符合我们的需求，我们可以在确认提取之前自由更改它。

让我们用一个例子来说明。我们可以想象向我们的`SimpleClass`类添加一个方法，告诉我们当前日期是否在两个给定日期之间:

```java
public static boolean isNowBetween(LocalDate startingDate, LocalDate endingDate) {
    return LocalDate.now().isAfter(startingDate) && LocalDate.now().isBefore(endingDate);
}
```

假设我们想要改变我们的实现，因为我们使用了`LocalDate.now()`两次，我们想要确保它在两次评估中具有完全相同的值。让我们选择表达式，并将其提取到一个局部变量`now:`

[![extract local variable](img/6e43ad089f9c223baa5036763f22c109.png)](/web/20221205113308/https://www.baeldung.com/wp-content/uploads/2019/04/extract_local_variable.png)

然后，我们的`LocalDate.now()`调用在一个局部变量中被捕获:

```java
public static boolean isNowBetween(LocalDate startingDate, LocalDate endingDate) {
    LocalDate now = LocalDate.now();
    return now.isAfter(startingDate) && now.isBefore(endingDate);
}
```

通过检查`Replace all`选项，我们确保两个表达式都被立刻替换。

### 3.2.方法

现在让我们看看如何使用 IntelliJ 提取方法:

*   选择适合我们想要创建的方法的表达式或代码行
*   右键单击选定的区域
*   **触发`Refactor > Extract > Method`选项**
*   键入方法信息:名称、可见性和参数
*   按下`Enter`

**选择方法体后点击`Ctrl + Alt + M`同样有效。**

让我们重复前面的例子，假设我们想要一个方法来检查任何日期是否在其他日期之间。然后，我们只需选择`isNowBetween`方法中的最后一行，并触发方法提取特性。

在打开的对话框中，我们可以看到 IntelliJ 已经找到了三个需要的参数:`startingDate` `, endingDate`和`now`。因为我们希望这个方法尽可能的通用，我们将`now`参数重命名为`date`。出于内聚的目的，我们将它作为第一个参数。

最后，我们给我们的方法命名为`isDateBetween`，并完成提取过程:

[![extract method](img/17709b7b8aecf7308161640c5958336d.png)](/web/20221205113308/https://www.baeldung.com/wp-content/uploads/2019/04/extract_method.png)

然后，我们将获得以下代码:

```java
public static boolean isNowBetween(LocalDate startingDate, LocalDate endingDate) {
    LocalDate now = LocalDate.now();
    return isDateBetween(now, startingDate, endingDate);
}

private static boolean isDateBetween(LocalDate date, LocalDate startingDate, LocalDate endingDate) {
    return date.isBefore(endingDate) && date.isAfter(startingDate);
}
```

正如我们所看到的，这个动作触发了新的`isDateBetween`方法的创建，这个方法也在`isNowBetween`方法中被调用。默认情况下，该方法是私有的。当然，这可以使用可见性选项进行更改。

### 3.3.班级

在所有这些之后，我们可能想要在一个特定的类中得到我们的日期相关的方法，集中在日期管理上，比方说:`DateUtils`。同样，这很简单:

*   右键单击包含我们想要移动的元素的类
*   **触发`Refactor > Extract > Delegate`选项**
*   键入类信息:它的名称、它的包、要委托的元素、这些元素的可见性
*   按下`Enter`

默认情况下，此功能没有可用的键盘快捷键。

比方说，在触发特性之前，我们在`main`方法中调用与日期相关的方法:

```java
isNowBetween(LocalDate.MIN, LocalDate.MAX);
isDateBetween(LocalDate.of(2019, 1, 1), LocalDate.MIN, LocalDate.MAX);
```

然后我们使用委托选项将这两个方法委托给一个`DateUtils`类:

[![delegate](img/be72e68164bca4fca0c3eed87c6bff9e.png)](/web/20221205113308/https://www.baeldung.com/wp-content/uploads/2019/04/delegate.png)

触发该特性会产生以下代码:

```java
public class DateUtils {
    public static boolean isNowBetween(LocalDate startingDate, LocalDate endingDate) {
        LocalDate now = LocalDate.now();
        return isDateBetween(now, startingDate, endingDate);
    }

    public static boolean isDateBetween(LocalDate date, LocalDate startingDate, LocalDate endingDate) {
        return date.isBefore(endingDate) && date.isAfter(startingDate);
    }
}
```

我们可以看到`isDateBetween`方法已经做成了`public`。这是可见性选项的结果，默认设置为`escalate`。 **`Escalate`意味着可见性将被改变，以便确保对委托元素的当前调用仍在编译。**

在我们的例子中，`isDateBetween`用于`SimpleClass:`的`main `方法中

```java
DateUtils.isNowBetween(LocalDate.MIN, LocalDate.MAX);
DateUtils.isDateBetween(LocalDate.of(2019, 1, 1), LocalDate.MIN, LocalDate.MAX);
```

因此，当移动方法时，有必要使它不是私有的。

但是，可以通过选择其他选项来为我们的元素提供特定的可见性。

## 4.内嵌

既然我们已经讨论了提取，那么让我们来讨论它的对应部分:[内联](https://web.archive.org/web/20221205113308/https://www.jetbrains.com/help/idea/inline-dialogs.html)。**内联就是获取一个代码元素并用它的构成来替换它。**对于变量，这将是它被赋值的表达式。对于一个方法来说，它就是它的主体。前面，我们看到了如何创建一个新的类，并将一些代码元素委托给它。但是有时候**我们可能想要[将一个方法委托给一个现有的类](https://web.archive.org/web/20221205113308/https://www.jetbrains.com/help/idea/move-dialogs.html)** 。这就是本节的内容。

为了内联一个元素，我们必须右键单击这个元素——它的定义或者对它的引用——并触发 **`Refactor >` `Inline`** 选项。我们也可以通过选择元素并点击 **`Ctrl + Alt + N`** 键来实现。

此时，IntelliJ 将为我们提供多种选择，无论我们希望内联一个变量还是一个方法，无论我们选择一个定义还是一个引用。这些选项是:

*   内联所有引用并移除元素
*   内联所有引用，但保留元素
*   仅内嵌选定的引用并保留元素

让我们用我们的`isNowBetween`方法，去掉`now`变量，现在看来有点多余:

[![inline local variable](img/78187a5048f13f8a0561f2d2fc736e5e.png)](/web/20221205113308/https://www.baeldung.com/wp-content/uploads/2019/04/inline_local_variable.png)

通过内联该变量，我们将获得以下结果:

```java
public static boolean isNowBetween(LocalDate startingDate, LocalDate endingDate) {
    return isDateBetween(LocalDate.now(), startingDate, endingDate);
}
```

在我们的例子中，唯一的选择是删除所有的引用和元素。但是让我们假设我们也想去掉`isDateBetween`调用，选择内联它。然后，IntelliJ 将为我们提供我们之前讨论过的三种可能性:

[![extract method 1](img/47e103e274e6310971d4cc3972cd4712.png)](/web/20221205113308/https://www.baeldung.com/wp-content/uploads/2019/04/extract_method-1.png)

选择第一个会用方法体替换所有调用，并删除该方法。至于第二个，它会用方法体替换所有调用，但保留方法。最后，最后一个只会用方法体替换当前调用:

```java
public class DateUtils {
    public static boolean isNowBetween(LocalDate startingDate, LocalDate endingDate) {
        LocalDate date = LocalDate.now();
        return date.isBefore(endingDate) && date.isAfter(startingDate);
    }

    public static boolean isDateBetween(LocalDate date, LocalDate startingDate, LocalDate endingDate) {
        return date.isBefore(endingDate) && date.isAfter(startingDate);
    }
}
```

我们在`SimpleClass`中的`main`方法也保持不变。

## 5.移动的

前面，我们看到了如何创建一个新的类，并将一些代码元素委托给它。但是有时候**我们可能希望[将一个方法委托给一个现有的类](https://web.archive.org/web/20221205113308/https://www.jetbrains.com/help/idea/move-dialogs.html)** 。这就是本节的内容。

为了移动元素，我们必须遵循以下步骤:

*   选择要移动的元素
*   右键单击该元素
*   **触发`Refactor > Move`选项**
*   选择接收方类和方法可见性
*   按下`Enter`

**我们也可以通过选择元素后按`F6`来实现。**

假设我们向我们的`SimpleClass`，`isDateOutstide()`添加了一个新方法，它将告诉我们一个日期是否位于日期间隔之外:

```java
public static boolean isDateOutside(LocalDate date, LocalDate startingDate, LocalDate endingDate) {
    return !DateUtils.isDateBetween(date, startingDate, endingDate);
}
```

然后我们意识到它应该在我们的`DateUtils`类中。所以我们决定移动它:

[![move method](img/c20a59bc935e3b214e2da9817ea2ce8b.png)](/web/20221205113308/https://www.baeldung.com/wp-content/uploads/2019/04/move_method.png)

我们的方法现在在`DateUtils`类中。我们可以看到方法中对`DateUtils`的引用已经消失，因为不再需要它了:

```java
public static boolean isDateOutside(LocalDate date, LocalDate startingDate, LocalDate endingDate) {
    return !isDateBetween(date, startingDate, endingDate);
}
```

我们刚刚做的例子工作得很好，因为它涉及一个静态方法。然而，在实例方法的情况下，事情就不那么简单了。

如果我们想要[移动一个实例方法](https://web.archive.org/web/20221205113308/https://medium.com/@pelensky/refactoring-in-intellij-move-method-d9f43e108e9a)， **IntelliJ 将搜索当前类的字段中引用的类，并提议将该方法移动到其中一个类**(假设它们是我们要修改的)。

如果在字段中没有引用可修改的类，那么 IntelliJ 建议在移动它之前创建方法`static`。

## 6.更改方法签名

最后，我们将讨论一个允许我们[改变方法签名](https://web.archive.org/web/20221205113308/https://www.jetbrains.com/help/idea/change-signature-dialog.html)的特性。**这个特性的目的是操纵方法签名的每个方面。**

像往常一样，我们必须通过几个步骤来触发该功能:

*   选择要更改的方法
*   右键单击该方法
*   **触发`Refactor > Change signature`选项**
*   对方法签名进行更改
*   按下`Enter`

如果我们喜欢使用键盘快捷键，**也可以使用`Ctrl + F6`。**

该功能将打开一个与方法提取功能非常相似的对话框。所以**我们有同样的可能性，当我们提取一个方法**时:改变它的名字，它的可见性，以及添加/删除参数和微调它们。

假设我们想要改变我们的`isDateBetween`的实现来考虑日期界限是包含性的还是排他性的。为了做到这一点，我们想给我们的方法添加一个`boolean`参数:

[![change signature](img/332da19602a9852578826bcdc07a5c37.png)](/web/20221205113308/https://www.baeldung.com/wp-content/uploads/2019/04/change_signature.png)

通过更改方法签名，我们可以添加这个参数，给它命名并给它一个默认值:

```java
public static boolean isDateBetween(LocalDate date, LocalDate startingDate,
   LocalDate endingDate, boolean inclusive) {
    return date.isBefore(endingDate) && date.isAfter(startingDate);
}
```

之后，我们只需要根据我们的需要修改方法体。

如果我们愿意，我们可以选择`Delegate via overloading method`选项来创建另一个带参数的方法，而不是修改当前的方法。

## 7.向上拉和向下推

我们的 Java 代码通常有类层次结构——**派生类扩展了基类**。

有时我们想在这些类之间移动成员(方法、字段和常量)。这就是最后一个重构派上用场的地方:它允许我们**将成员从一个派生类提升到基类中，或者将它们从一个基类下推到每个派生类中**。

### 7.1.停下

首先，让我们[为基类拉一个方法](https://web.archive.org/web/20221205113308/https://www.jetbrains.com/help/idea/pull-members-up-dialog.html):

*   选择要提取的派生类成员
*   右键单击成员
*   **触发`Refactor > Pull Members Up…`选项**
*   按下`Refactor`按钮

默认情况下，此功能没有可用的键盘快捷键。

假设我们有一个名为`Derived.` 的派生类，它使用一个私有的`doubleValue()`方法:

```java
public class Derived extends Base {

    public static void main(String[] args) {
        Derived subject = new Derived();
        System.out.println("Doubling 21\. Result: " + subject.doubleValue(21));
    }

    private int doubleValue(int number) {
        return number + number;
    }
}
```

基类`Base`为空。

那么，当我们把`doubleValue()`拉高到`Base`时会发生什么呢？

[![Refactoring with IntelliJ IDEA Pull Up](img/e71b9beb7edd905065fd9f1f931d9b1e.png)](/web/20221205113308/https://www.baeldung.com/wp-content/uploads/2019/04/Refactoring-with-IntelliJ-IDEA-Pull-Up.png)

当我们在上面的对话框中按下“重构”时,`doubleValue()` 会发生两件事:

*   它移动到`Base` 类
*   它的可见性从`private`变为`protected`，这样`Derived`类仍然可以使用它

之后的`Base`类现在有了方法:

```java
public class Base {
    protected int doubleValue(int number) {
        return number + number;
    }
}
```

用于调出成员的对话框(如上图)为我们提供了更多的选项:

*   我们可以选择其他成员，然后一下子把他们拉上来
*   我们可以使用“预览”按钮预览我们的更改
*   只有方法在“成为抽象”列中有一个复选框。如果选中，此选项将在拉起过程中给基类一个抽象方法定义。实际的方法将保留在派生类中，但会获得一个`@Override` 注释。**因此，其他的派生类将不再编译**，因为它们缺少了新的抽象基类方法的实现

### 7.2.下推

最后，让我们[将成员](https://web.archive.org/web/20221205113308/https://www.jetbrains.com/help/idea/push-members-down-dialog.html)下推到派生类。这与我们刚刚执行的引体向上相反:

*   选择要下推的基类成员
*   右键单击成员
*   **触发`Refactor > Push Members Down…`选项**
*   按下`Refactor`按钮

与向上拉成员一样，默认情况下，此功能没有可用的键盘快捷键。

让我们把刚刚拉上来的方法再往下推一次。上一节末尾的`Base`类是这样的:

```java
public class Base {
    protected int doubleValue(int number) {
        return number + number;
    }
}
```

现在，让我们将`doubleValue()`下推到`Derived`类:

[![Refactoring with IntelliJ IDEA Push Down](img/3cb6e7e079f822c2ea56f2adc1d5d51c.png)](/web/20221205113308/https://www.baeldung.com/wp-content/uploads/2019/04/Refactoring-with-IntelliJ-IDEA-Push-Down.png)

这是上图对话框中按下“Refactor”后的`Derived`类。`doubleValue()`方法回来了:

```java
public class Derived extends Base {
    private int theField = 5;

    public static void main(String[] args) {
        Derived subject = new Derived();
        System.out.println( "Doubling 21\. Result: " + subject.doubleValue(21));
    }

    protected int doubleValue(int number) {
        return number + number;
    }
}
```

现在`Base`类和`Derived`类都回到了它们在之前的“拉起”部分开始的地方。差不多，也就是——`doubleValue()`保持了在`Base`时的`protected`(原来是`private`)。

**IntelliJ 2019.3.4 在下推`doubleValue()`的时候居然带来了一个警告**:“被推送的成员在某些调用站点将不可见”。但是正如我们在上面的`Derived`类中看到的，`doubleValue()`对于`main()`方法确实是可见的。

用于下推成员的对话框(如上图)也为我们提供了更多的选项:

*   如果我们有多个派生类，那么 IntelliJ 会将成员推入每个派生类
*   我们可以将多个成员向下推
*   我们可以使用“预览”按钮预览我们的更改
*   只有方法在“保持抽象”列中有一个复选框——这类似于提取成员:如果选中，这个选项将在基类中留下一个抽象方法。与提取成员不同，该选项将把方法实现放入所有派生类中。这些方法也将获得一个`@Override`注释

## 8.结论

在本文中，我们有机会深入了解 IntelliJ 提供的一些重构特性。当然，我们没有涵盖所有的可能性，因为 IntelliJ 是一个非常强大的工具。要了解这个编辑器的更多信息，我们可以随时参考[的文档](https://web.archive.org/web/20221205113308/https://www.jetbrains.com/help/idea/2018.3/meet-intellij-idea.html)。

我们看到了一些事情，比如如何重命名我们的代码元素，以及如何将一些行为提取到变量、方法或类中。我们还学习了如何内联一些不需要的独立元素，将一些代码移到别处，甚至完全改变现有的方法签名。