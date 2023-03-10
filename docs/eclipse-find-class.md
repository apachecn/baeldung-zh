# 如何用 Eclipse 找到并打开一个类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/eclipse-find-class>

## 1。简介

在本文中，我们将研究在 Eclipse 中查找类的多种方法。所有的例子都是基于`Eclipse Oxygen`。

## 2。概述

在 Eclipse 中，我们经常需要寻找一个类或一个接口。我们有很多方法可以做到:

*   “打开类型”对话框
*   “打开资源”对话框
*   包资源管理器视图
*   开放声明函数
*   类型层次结构视图

## 3。开放式

做到这一点的最有效的方法之一是使用开放类型对话框。

### 3.1。使用工具

我们可以通过三种方式访问它:

1.  使用键盘快捷键，在 PC 上是`Ctrl` `+ Shift +` `T`，在 Mac 上是`Cmd + Shift + T`。
2.  打开`Navigate > Open Type` 下的菜单
3.  单击主工具栏中的图标:

[![Senzanome-1](img/b91637751ad38bcb09043383d070f493.png)](/web/20221129004330/https://www.baeldung.com/wp-content/uploads/2018/03/Senzanome-1.png)

### 3.2。用它来找一个类

一旦我们启动了`Open Type`,我们只需要开始输入，我们就会看到结果:

[![Senzanome-2](img/82d7abe5a47b82f4bee523e6389ab53e.png)](/web/20221129004330/https://www.baeldung.com/wp-content/uploads/2018/03/Senzanome-2.png)

结果将包含我们的开放项目的构建路径中的类，包括项目类、库和 JRE 本身。

此外，它还显示了包及其在我们环境中的位置。

正如我们在图像中看到的，结果是名称以我们键入的内容开头的任何类。这种搜索不区分大小写。

**我们也可以在骆驼箱里搜索**。例如，要查找类别`ArraysParallelSortHelpers`，我们只需键入`APSH`或`ArrayPSH.` **即可。这种类型的搜索区分大小写。**

此外，**也可以使用通配符**“*”或“？”在搜索文本中。“*”代表任何字符串，包括空字符串和“？”对于任何字符，不包括空字符串。

例如，我们想找到一个我们记得包含`Linked,`和其他内容的类，然后`Multi.` "* "就派上用场了:

[![Senzanome-3](img/362e29880403e3b4dac1afb8900eee44.png)](/web/20221129004330/https://www.baeldung.com/wp-content/uploads/2018/03/Senzanome-3.png)

或者如果我们加上一个“？”：

[![Senzanome-4](img/f81e6cf293e2af290db8810dccfa4241.png)](/web/20221129004330/https://www.baeldung.com/wp-content/uploads/2018/03/Senzanome-4.png)

那个“？”这里不包括空字符串，所以从结果中删除了`LinkedMultiValueMap`。

还要注意，在每个输入的结尾都有一个隐含的“*”，但在开头没有。

## 4。打开资源

在 Eclipse 中查找和打开一个类的另一个简单方法是`Open Resource`。

### 4.1。使用工具

我们可以通过两种方式访问它:

*   使用键盘快捷键，在 PC 上是`Ctrl` `+ Shift +` `R`，在 Mac 上是`Cmd + Shift + R`。
*   打开`Navigate > Open Resource`下的菜单

### 4.2。用它来找一个类

一旦对话框打开，我们只需开始输入，我们将看到结果:

[![Senzanome-9](img/15ecf5739fd3f8310360eeeb38b6e179.png)](/web/20221129004330/https://www.baeldung.com/wp-content/uploads/2018/03/Senzanome-9.png)

结果将包含类以及我们打开的项目的构建路径中的所有其他文件。

有关通配符和 camel case 搜索的使用细节，请查看上面的`Open Type` 部分。

## 5。包浏览器

当我们知道了我们类所属的包，就可以用`Package Explorer`。

### 5.1。使用工具

如果它还不可见，那么我们可以通过`Window > Show View > Package Explorer`下的菜单打开这个 Eclipse 视图。

### 5.2。使用工具查找课程

这里的类别按字母顺序显示:

[![Senzanome-5](img/7f397ca045246708db5f4147a0c8f850.png)](/web/20221129004330/https://www.baeldung.com/wp-content/uploads/2018/03/Senzanome-5.png)

如果列表很长，我们可以使用一个技巧:我们点击包树上的任何地方，然后我们开始输入类名。我们将看到选择自动在类之间滚动，直到它匹配我们的类。

还有`Navigator`视图，其工作方式几乎相同。

主要区别在于，`Package Explorer` 显示相对于包的类，而`Navigator`显示相对于底层文件系统的类。

要打开这个视图，我们可以在`Window > Show View > Navigator`下的菜单中找到它。

## 6。公开声明

在我们查看引用我们的类的代码的情况下，`Open Declaration`是一种非常快速的跳转方式。

### 6.1。使用工具

有三种方法可以使用该功能:

1.  单击我们想要打开的类名的任意位置，然后按 F3
2.  点击类名的任意位置，进入`Navigate > Open Declaration`下的菜单
3.  按住`Ctrl`按钮，将鼠标悬停在类名上，然后点击它

### 6.2。用它来找一个类

想想下面的截图，如果我们按下`Ctrl` 并将鼠标悬停在`ModelMap`上，就会出现一个链接:

[![Senzanome-6](img/a215119a78e53322e7c1814b08f4dbe1.png)](/web/20221129004330/https://www.baeldung.com/wp-content/uploads/2018/03/Senzanome-6.png)

请注意，颜色变为浅蓝色，并带有下划线。这表明它现在可以直接链接到该类。如果我们单击链接，Eclipse 将在编辑器中打开`ModelMap` 。

## 7。类型层次结构

在像 Java 这样的面向对象语言中，我们也可以考虑与超类和子类的层次结构相关的类型。

`Type Hierarchy`是一个类似于`Package Explorer`和`Navigator`的视图，这次关注的是层次结构。

### 7.1。使用工具

我们可以通过三种方式访问该视图:

1.  单击类名中的任意位置，然后按 F4
2.  点击类名中的任意位置，进入`Navigate > Open Type Hierarchy`下的菜单
3.  使用`Open Type in Hierarchy` 对话框

`Open Type in Hierarchy` 对话框的行为就像我们在第 3 节看到的`Open Type`。

要到达那里，我们进入`Navigate > Open Type in Hierarchy` 下的菜单，或者使用快捷方式:PC 上的`Ctrl` `+ Shift + H`或 Mac 上的`Cmd + Shift + H` 。

[![Senzanome](img/1b62da5aec8c5291f7e3da91a3330497.png)](/web/20221129004330/https://www.baeldung.com/wp-content/uploads/2018/03/Senzanome.png)

该对话框类似于`Open Type` 对话框。除了这次我们点击一个类，然后我们得到了`Type Hierarchy`视图。

### 7.2。使用工具查找课程

一旦我们知道了想要打开的类的超类或子类，我们就可以在层次结构树中导航，并在那里寻找该类:

[![Senzanome-7](img/7e97536277ec973bf488ec599423b06f.png)](/web/20221129004330/https://www.baeldung.com/wp-content/uploads/2018/03/Senzanome-7.png)

如果列表很长，我们可以使用与`Package Explorer`相同的技巧:我们点击树上的任何地方，然后开始输入类名。我们将看到选择自动在类之间滚动，直到它匹配我们的类。

## 8。结论

在本文中，我们研究了用 Eclipse IDE 查找和打开 Java 类的最常见方法，包括`Open Type,` `Open Resource, Package Explorer, Open Declaration,` 和`Type Hierarchy`。