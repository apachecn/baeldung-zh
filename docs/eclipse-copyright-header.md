# 在 Eclipse 中为 Java 源文件添加版权许可证头

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/eclipse-copyright-header>

## 1。概述

众所周知，在 Eclipse IDE 中为源文件添加版权许可证头是一项困难且容易出错的任务。

在本教程中，我们将学习两种方法来使这项任务变得简单和没有错误。第一种使用 Eclipse IDE 的代码模板特性。第二种使用版权生成器插件。

## 2。使用代码模板

首先，让我们看看如何使用代码模板添加版权许可证头。让我们从 Eclipse 设置开始:

### 2.1.Eclipse 设置

*   从主菜单栏中，导航至**首选项**
*   然后，导航到 **Java - >代码风格- >代码模板**
*   从窗口的右侧，展开**代码**部分，并选择**新 Java 文件**

[![eclipsecopy1](img/d0166f46de563550b0e05ecdb5af9e30.png)](/web/20220628113315/https://www.baeldung.com/wp-content/uploads/2019/07/eclipsecopy1.png)

*   然后，我们通过点击**编辑**按钮进入**编辑模板**
*   在**编辑模板**窗口，我们在**图案**文本区添加我们的版权许可头
*   然后，点击**确定**按钮更新模板
*   最后，让我们点击**应用**按钮来完成设置

[![eclipsecopy2](img/3135264d89ac97db280b94543843c14a.png)](/web/20220628113315/https://www.baeldung.com/wp-content/uploads/2019/07/eclipsecopy2.png)

### 2.2.应用版权

我们现在得到一个自动应用于每个新 Java 源文件的版权头**:**

[![eclipsecopy3](img/9bed0522ee9024718da34d760d1a9414.png)](/web/20220628113315/https://www.baeldung.com/wp-content/uploads/2019/07/eclipsecopy3.png)

然而，**这种方法有一些缺点**:

*   我们不能用它来给现有的源文件添加版权头
*   我们不能在许可证文本中包含自定义变量，如我们的公司名称
*   它不够灵活，不能让我们选择不同的版权许可证头
*   我们只能在 Java 和 Javascript 源文件上使用它

幸运的是，有一个以版权生成器插件形式出现的替代方案。

## 3。使用版权生成器插件

让我们从设置版权生成器插件开始。

### 3.1.安装插件

*   我们通过导航到**帮助->Eclipse market place**T3 来安装插件
*   然后我们在**查找**文本框中搜索 **Eclipse 版权生成器**
*   最后，让我们点击**安装**按钮，并按照说明进行操作

[![eclipsecopy4](img/b72f5036c55c78a3152f3cecacac45ac.png)](/web/20220628113315/https://www.baeldung.com/wp-content/uploads/2019/07/eclipsecopy4.png)

### 3.2.自定义版权许可证标题

所有标准的版权许可证头都是随插件一起预装的。

但是，如果我们想添加一个自定义标题，或者编辑一个呢？让我们看看如何在**版权偏好**中实现这一点:

*   我们先导航到**偏好设置- >常规- >版权**
*   然后，要添加新的版权标题，我们单击**添加**按钮
*   要修改现有许可证，我们从**许可证**中选择一个许可证，然后点击**修改**按钮

[![eclipsecopy5](img/fed46050f5ddbfe32ad393f45821ea22.png)](/web/20220628113315/https://www.baeldung.com/wp-content/uploads/2019/07/eclipsecopy5.png)

### 3.3。申请版权

安装插件后，将版权许可头应用到一个或多个源文件是非常容易的。

让我们看看如何将它应用于所选的源文件:

*   从**项目浏览器**面板中，我们选择源文件
*   然后，**在选中的源文件**上右键
*   从上下文菜单中，我们选择**应用版权**选项

同样，将此应用于一个或多个项目:

*   从主菜单栏中选择**项目- >申请版权**T3

[![eclipsecopy6](img/f82f11772150efb6659129952ff5690e.png)](/web/20220628113315/https://www.baeldung.com/wp-content/uploads/2019/07/eclipsecopy6.png)

然后我们按照**应用版权**对话框中的指示给文件添加版权头:

*   我们点击**下一个**按钮，这将我们带到**版权设置**

[![eclipsecopy7](img/b86795f35d6d8e0f87ad6bc38ae69d21.png)](/web/20220628113315/https://www.baeldung.com/wp-content/uploads/2019/07/eclipsecopy7.png)

*   然后，从**版权类型**选择框中选择一个版权许可证

[![eclipsecopy8](img/4f7d78877bd147b2d0c082eaad570d29.png)](/web/20220628113315/https://www.baeldung.com/wp-content/uploads/2019/07/eclipsecopy8.png)

*   然后，点击**下一个**按钮，这将把我们带到文件窗口
*   最后，我们点击**完成**按钮，将版权头应用到选定的源文件

与代码模板不同，插件**不会在创建新文件**时自动添加版权头。

然而，插件与代码模板相比有几个优势:

*   很容易为现有的源文件添加版权许可证头
*   我们可以在许可证文本中包含自定义变量，如公司名称
*   该插件支持许多不同的版权许可证头
*   我们可以为所有类型的源文件添加版权头，而不仅限于 Java 源文件

## 4。结论

在这篇文章中，我们学习了两种不同的方法来给我们的源文件添加版权许可头。

[**Eclipse Copyright Generator**](https://web.archive.org/web/20220628113315/https://jmini.github.io/Eclipse-Copyright-Generator)插件是添加版权头最简单、最灵活的方式。

唯一的缺点是，它需要在文件创建后应用，其中代码模板会在每个文件创建时为我们添加一个版权头。