# IntelliJ IDEA 中的常见快捷方式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/intellij-idea-shortcuts>

## 1.概观

本文着眼于在 JetBrains 的 Java IDE IntelliJ IDEA 中编辑、构建和运行 Java 应用程序所需的**键盘快捷键。**键盘快捷键节省了我们的时间**，因为我们可以将手放在键盘上，更快地完成工作。**

我们在上一篇文章中讨论了 IntelliJ IDEA [的重构，所以我们在这里不讨论这些快捷方式。](/web/20220526035714/https://www.baeldung.com/intellij-refactoring)

## 2.唯一的捷径

如果我们只记得一个 IntelliJ IDEA 快捷方式，那么它一定是 Windows 中的 `is` `Ctrl + Shift + A`和 macOS 中的`Shift + Cmd + A`中的 **`Help – Find Action,` 。此快捷键会打开一个搜索窗口，其中包含所有菜单项和其他 IDE 操作，无论它们是否有快捷键。我们可以立即输入以缩小搜索范围，使用光标键选择一个功能，并使用`Enter`来执行它。**

从现在开始，我们将在菜单项名称后面的括号中列出键盘快捷键。如果 Windows 和 macOS 之间的快捷方式不同，正如他们通常所做的那样，那么我们将 Windows 快捷方式放在第一位，macOS 放在第二位。

在 macOS 电脑上，`Alt`键通常被称为`Option`。为了使我们的快捷方式简洁，在本文中我们仍然称它为`Alt`。

## 3.设置

让我们从配置 IntelliJ IDEA 和我们的项目开始。

**我们在 Windows 中用`File – Settings` ( `Ctrl + Alt + S`)达到 IntelliJ 的设置，在 macOS 中用`IntelliJ IDEA – Preferences` ( `Cmd + ,` )** 。为了配置我们当前的项目，我们选择了`Project`视图中的顶级元素。它有项目名称。然后我们可以用`File – Project Structure` ( `Ctrl + Alt + Shift + S` / `Cmd + ;`)打开它的配置。

## 4.导航到文件

配置完成后，我们就可以开始编码了。首先，我们需要找到我们想要处理的文件。

我们通过浏览左边的`Project`视图来选择文件。我们也可以用`File – New` ( `Alt + Insert` / `Cmd + N`)在当前选择的位置新建文件。要删除当前选中的文件/文件夹，我们触发`Edit – Delete` ( `Delete` / `⌫`)。**我们可以用 Windows 上的`Esc`和 macOS 上的`⎋`从`Project`视图切换回编辑器**。这没有菜单项。

**直接打开一个类，我们用`Navigate – Class` ( `Ctrl + N` / `Cmd + O`)。**这适用于 Java 类和其他语言的类，比如 TypeScript 或 Dart。如果我们想打开任何文件，比如 HTML 或文本文件，我们使用`Navigate – File` ( `Ctrl + Shift + N` / `Shift + Cmd + O`)。

所谓切换器就是当前打开文件的列表。**我们只能通过快捷键`Ctrl + Tab`** 看到切换器，因为它没有菜单项。**最近打开的文件列表有`View – Recent` ( `Ctrl + E` / `Cmd + E` )** 。如果我们再次按下该快捷键，那么我们只能看到最近更改的文件。

**我们用`Navigate – Last Edit Location` ( `Ctrl + Shift + Backspace` / `Shift + Cmd + ⌫` )** 去我们最后代码变化的地方。IntelliJ 还跟踪我们的编辑器文件位置。我们可以用`Navigate – Back` ( `Ctrl + Alt + Left` / `Cmd + [`)和`Navigate – Forward` ( `Ctrl + Alt + Right` / `Cmd + ]`)来浏览那段历史。

## 5.在文件中导航

我们找到了想要处理的文件。现在我们需要导航到正确的地方。

**我们直接用`Navigate – File Structure` ( `Ctrl + F12` / `Cmd + F12` )** 跳转到一个类的一个字段或者方法。与`Help – Find Action`一样，我们可以立即输入以缩小显示的成员范围，使用光标键选择一个成员，并使用`Enter`跳转到该成员。如果我们想在当前文件中突出显示一个成员的用法，我们使用`Edit – Find Usages – Find Usages in File` ( `Ctrl + F7` / `Cmd` ` + F7`)。

**我们用`Navigate – Declaration or Usages` ( `Ctrl + B` / `Cmd + B` )** 得出基类或方法的定义。顾名思义，调用基类或方法本身的功能显示了它的用法。因为这是一个非常常用的功能，所以它有一个鼠标快捷键:Windows 上的`Ctrl + Click`和 macOS 上的`Cmd + Click`。如果我们需要查看项目中某个类或方法的所有用法，我们调用`Edit – Find Usages – Find Usages` ( `Alt + F7`)。

我们的代码经常调用其他方法。如果我们将光标放在方法调用括号内，那么 **`View – Parameter Info` ( `Ctrl + P` / `Cmd + P`)会显示关于方法参数**的信息。在默认的 IntelliJ IDEA 配置中，此参数信息会在短暂的延迟后自动出现。

要查看类型或方法的快速文档窗口，我们需要`View – Quick Documentation` ( `Ctrl + Q` / `F1`)。在默认的 IntelliJ IDEA 配置中，如果我们将鼠标光标移到类型或方法上并稍作等待，快速文档会自动出现。

## 6.**编辑文件**

### 6.1.更改代码

一旦我们到达正确的文件和正确的位置，我们就可以开始编辑我们的代码。

当我们开始键入变量、方法或类型的名称时，IntelliJ IDEA 会帮助我们用`Code – Code Completion – Basic` ( `Ctrl + Space`)来完成这些名称。在默认的 IntelliJ IDEA 配置中，该函数也会在短暂延迟后自动启动。我们可能需要键入一个右括号，并在末尾加上一个分号。 **`Code – Code Completion – Complete Current Statement` ( `Ctrl + Shift + Enter` / `Shift + Cmd + Enter`)结束我们当前的线路**。

**`Code – Override Methods` ( `Ctrl + O`)让我们选择继承的方法来覆盖**。使用`Code – Generate` ( `Alt + Insert` / `Cmd + N`，我们可以创建像 getters、setters 或`toString()`这样的通用方法。

我们可以使用`Code`–`Surround with`(`Ctrl + Alt + T`/`Alt + Cmd +T`)在我们的代码周围放置控制结构，比如一个 `if`语句。我们甚至可以用`Code – Comment with Block Comment`注释掉一整块代码。也就是 Windows 中的`Ctrl + Shift + /`和 macOS 中的`Alt + Cmd + /`。

例如，IntelliJ IDEA 会在运行之前自动保存我们的代码。我们仍然可以用`File – Save all` ( `Ctrl + S` / `Cmd + S`)手动保存所有文件。

### 6.2.浏览代码

有时，我们需要在文件中移动代码。`Code – Move Statement Up` ( `Ctrl + Shift + Up` / `Alt + Shift +Up`)和`Code – Move Statement Down` ( `Ctrl + Shift + Down` / `Alt + Shift +Down`)对当前选中的代码进行此操作。如果我们没有选择任何东西，那么当前行被移动。同样， **`Edit – Duplicate Line or Selection` ( `Ctrl + D` / `Cmd + D`)复制选中的代码或当前行**。

我们可以用`Navigate – Next Highlighted Error` ( `F2`)和`Navigate – Previous Highlighted Error` ( `Shift + F2`)循环遍历当前文件中的错误。如果**我们将光标放在不正确的代码上并点击`Alt + Enter`，IntelliJ IDEA 将建议修复**。这个快捷方式没有菜单项。如果我们的代码没有错误，这个快捷方式也可能会建议修改我们的代码。

## 7.查找和替换

我们经常需要查找和替换代码。下面是我们如何在当前文件或所有文件中实现这一点。

**为了在当前文件中查找文本，我们使用`Edit – Find – Find` ( `Ctrl + F` / `Cmd + F` )** 。为了替换当前文件中的文本，我们使用了`Edit – Find – Replace` ( `Ctrl + R` / `Cmd + R`)。在这两种情况下，我们使用`Edit – Find –` `Find Next Occurrence` ( `F3 / Cmd + G`)和`Edit – Find –` `Find Previous` `Occurrence` ( `Shift + F3` / `Shift + Cmd + G`)在搜索结果中移动。

我们也可以在我们所有的文件中找到带有`Edit – Find – Find in Files` ( `Ctrl + Shift + F` / `Shift + Cmd + F` ) 的文本。同样，`Edit – Find – Replace in Files (Ctrl + Shift + R / Shift + Cmd +R)` 替换我们所有文件中的文本。我们仍然可以使用`F3 / Cmd + G`和`Shift + ` `F3 / Shift + Cmd + G` 来浏览我们的搜索结果。

## 8.**构建并运行**

我们想在完成编码后运行我们的项目。

当我们运行项目时，IntelliJ IDEA 通常会自动构建我们的项目。**使用`Build – Build Project` ( `Ctrl + F9` / `Cmd + F9`)，我们手动验证我们最近的代码更改是否仍然编译**。我们可以用`Build – Rebuild Project` ( `Ctrl + Shift + F9` ` / Shift + Cmd + F9`)从头开始重建我们的整个项目。

为了用当前的运行配置运行我们的项目，我们使用了`Run – Run ‘(configuration name)'` ( `Shift + F10` / `Ctrl + R`)。**我们用`Run – Run…` ( `Alt + Shift + F10` / `Ctrl + Alt + R` )** 执行一个特定的运行配置。同样，我们可以用`Run – Debug ‘(configuration name)'` ( `Shift + F9` / `Ctrl + D`)调试当前运行配置，用`Run – Debug` ( `Alt + Shift + F9` / `Ctrl` `+ Alt + D`)调试任何其他运行配置。

## 9.排除故障

我们的项目会有 bug。调试帮助我们找到并修复这些错误。

调试器在断点处停止。**我们用`Run – View Breakpoints` ( `Ctrl + Shift + F8` / `Shift + Cmd + F8` )** 查看当前断点。我们可以用`Run – Toggle Breakpoint – Line Breakpoint` ( `Ctrl + F8` / `Cmd + F8`)在当前行切换一个断点。

当我们的代码在调试过程中遇到断点时，我们可以用`Run – Debugging Actions – Step Over` ( `F8`)单步执行当前行。所以如果这一行是一个方法，我们将一次性执行整个方法。或者，**我们可以用`Run – Debugging Actions – Step Into` ( `F7` )** 深入当前行的方法。

调试时，我们可能希望运行我们的代码，直到当前方法完成。那就是`Run – Debugging Actions – Step Out` ( `Shift + F8`)做的事情。如果**我们希望我们的程序运行到光标所在的行，那么`Run – Debugging Actions – Run to Cursor` ( `Alt + F9`)完成这个**。如果我们想让程序一直运行到遇到下一个断点，那么`Run – Debugging Actions – Resume Program` ( `F9`)就是这么做的。

## 10.饭桶

我们的程序通常驻留在 Git 存储库中。IntelliJ IDEA 对 Git 有极好的支持。

**我们有一个快捷键来提供所有可能的 Git 操作:`Git – VCS Operations` ( `Alt + `` / `Ctrl + V` )** 。正如所料，我们可以用光标选择项目并点击`Enter`来执行它们。这也是一个很好的方法来使用默认没有快捷键的常用功能，比如`Show History`或者`Show Diff`。

如果我们想从一个远程 Git 存储库更新我们的项目，那么我们选择`Git – Update Project` ( `Ctrl + T` / `Cmd + T`)。当我们需要在 Git 中提交我们的更改时，那么`Git – Commit` ( `Ctrl + K` ` / Cmd + K`)是可用的。**为了将我们的更改恢复到 Git 中的内容，我们使用`Git – Uncommitted Changes – Rollback` ( `Ctrl + Alt + Z` / `Alt + Cmd + Z` )** 。并且`Git – Push` ( `Ctrl + Shift + K` / `Shift + Cmd + K`)将我们的更改推送到远程 Git 存储库。

## 11.结论

键盘快捷键节省了我们的时间，因为我们可以把手放在键盘上，更快地完成工作。本文介绍了在 IntelliJ IDEA 中配置、导航、编辑、查找和替换、运行和调试程序的快捷方式。

我们还查看了使用 Git 的快捷方式。