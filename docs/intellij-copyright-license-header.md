# 在 IntelliJ IDEA 中为 Java 源文件添加版权许可头

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/intellij-copyright-license-header>

## 1。概述

在本教程中，我们将了解如何向 IntelliJ IDEA 项目文件添加许可证头。许可证头通常描述项目文件的允许用途和所有权。

我们假设您对 IntelliJ 有一个[的基础知识，因此，我们将直奔主题。](/web/20220812130612/https://www.baeldung.com/intellij-basics)

## 2。配置许可证标题

让我们打开任何现有的项目来配置许可证头。我们首先需要通过点击 IntelliJ IDEA 菜单项或按 Command +逗号键来访问`Preference`窗口。

新弹出的窗口会在左侧显示一个菜单列表，我们可以在其中点击`Editor` > `Copyright` > `Copyright Profiles`。为了更快地访问，我们可以在首选项窗口的搜索栏中输入`Copyright`。

在窗口的右侧窗格中，让我们单击顶部的“+”按钮来创建一个新的版权配置文件。

我们首先给它起一个名字— `MIT License`，例如`—`，然后点击“OK”:

[![Copyright Profile](img/30618bd29d1ddb8eadd8df0136705b3b.png)](/web/20220812130612/https://www.baeldung.com/wp-content/uploads/2019/03/Screenshot-2019-03-03-at-6.43.52-PM.png)

之后，我们将许可证文本添加到文本区域。

我们使用了一个样本文本，最终应该替换为我们自己的文本:

[![Copyright Profiles](img/e5b95538537bf5667a5bd3127189fed6.png)](/web/20220812130612/https://www.baeldung.com/wp-content/uploads/2019/03/Screenshot-2019-03-03-at-7.00.58-PM.png)

**我们用占位符–`$today.year.`**替换了`<YEAR>`,每次我们使用概要文件时，IntelliJ 都会自动将其翻译为当前年份。

IntelliJ 定义了我们可以在版权配置文件中使用的附加变量。详情请查看[帮助页面](https://web.archive.org/web/20220812130612/https://www.jetbrains.com/help/idea/copyright-profiles.html)。

最后，让我们单击许可证文本正下方的“验证”按钮，验证我们的文本并确保一切正常，然后单击“应用”保存配置文件。

接下来要做的是将概要文件设置为默认概要文件。我们可以通过点击`Editor` > `Copyright`并选择麻省理工学院许可证作为默认项目版权来实现:

[![MIT License](img/76ebfb4cd9b10384a6fffd77d153b525.png)](/web/20220812130612/https://www.baeldung.com/wp-content/uploads/2019/03/Screenshot-2019-03-03-at-7.12.23-PM.png)

## 3。在项目文件中使用创建的版权配置文件

现在，是时候在我们的项目文件中使用版权配置文件了。我们可以通过按下 Control + Return 键并选择版权菜单来实现这一点:

[![package](img/fcc860ccd8676d86adf769886d03b974.png)](/web/20220812130612/https://www.baeldung.com/wp-content/uploads/2019/03/Screenshot-2019-03-04-at-7.28.52-AM.png)

这将把我们先前设置为默认的版权文本应用到我们项目文件的最上面部分。

如果我们对结果文本的格式不满意，我们可以进入`Preferences` > `Editor` > `Copyright` > `Copyright Profiles`进行编辑，然后点击“应用”。之后，我们必须清除现有的文本，并按 Control + Return 键来应用更新的文本。

随后，我们创建的任何新项目文件都将自动具有许可证头，无需手动添加。

## 4。结论

在这篇简短的文章中，我们学习了如何配置 IntelliJ IDEA 来自动添加许可证头到我们所有的项目文件中——这对于较大的代码库来说非常方便。