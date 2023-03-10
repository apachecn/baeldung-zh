# IntelliJ 中的自动导入类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/intellij-auto-import-class>

## 1.概观

这个简短的教程将描述 IntelliJ IDEA 的“自动导入”功能的每个选项。

## 2.自动导入

IntelliJ IDEA 中有几个选项我们可以在`Settings > Editor > Auto Import` : [![import1](img/b2265fe3e1bb569b7f5c7d6c3fa87616.png)](/web/20221208143845/https://www.baeldung.com/wp-content/uploads/2018/07/import1.png) 中配置

让我们回顾一下这些选项。

### 2.1.显示导入弹出窗口

启用后，IDEA 将在我们的代码中为类引用加下划线，并建议添加一个导入:

[![import2](img/36edf3fb14473050153a80ecbff20268.png)](/web/20221208143845/https://www.baeldung.com/wp-content/uploads/2018/07/import2.png)

如果有几个选项可供选择，Idea 会让我们从选项列表中选择一个导入: [![import3](img/497816b16f669b5fdd401e270b2c21fa.png)](/web/20221208143845/https://www.baeldung.com/wp-content/uploads/2018/07/import3.png)

### 2.2.动态优化导入

这将使 IDEA 自动移除未使用的导入，并根据“代码风格”偏好重新排列其他导入。

### 2.3.动态添加明确的导入

此外，当我们向需要导入的类添加引用时，有一种方法可以自动添加导入。

### 2.4.显示静态方法和字段的导入建议

我们的最后一个选项将启用静态的导入弹出特性。

但是，请注意，仅打开此选项(没有`‘Show import popup'`)将不会为类 [![import4](img/11c70bc4b00ca420fd9c23036094b923.png)](/web/20221208143845/https://www.baeldung.com/wp-content/uploads/2018/07/import4.png) 启用导入建议

## 3.结论

一些开发人员喜欢完全控制类中的导入，其他人则依赖 IDE 来处理这项技术任务。

这两者都可以从 IntelliJ IDEA IDE 的各种配置选项中受益，包括那些用于导入行为的选项。