# Gradle 中的自定义任务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gradle-custom-task>

## 1。概述

在本文中，我们将介绍如何在 Gradle 中**创建一个定制任务。我们将使用构建脚本或定制任务类型来展示一个新的任务定义。**

有关 Gradle 的介绍，请参见本文。它包含了 Gradle 的基础知识，以及对 Gradle 任务的介绍，这也是本文最重要的内容。

## 2。`build.gradle`内自定义任务定义

为了创建一个简单的 Gradle 任务，我们需要将其定义添加到我们的`build.gradle`文件中:

```
task welcome {
    doLast {
        println 'Welcome in the Baeldung!'
    }
} 
```

上述任务的主要目标只是打印文本“欢迎来到 Baeldung！”。我们可以通过运行`**gradle tasks –all**`命令来检查**任务是否可用**:

```
gradle tasks --all 
```

该任务在组**其他任务**下的列表中:

```
Other tasks
-----------
welcome
```

它可以像其他 Gradle 任务一样执行:

```
gradle welcome 
```

输出和预期的一样——“欢迎来到 Baeldung！”消息。

备注:如果选项`–all`未设置，则属于“其他”类别的任务不可见。自定义梯度任务可以属于不同于“其他”的组，并且可以包含描述。

## 3。设置组和描述

有时按功能对任务进行分组很方便，因此它们在一个类别下是可见的。我们可以快速**为我们的自定义任务设置** **组，只需定义一个组属性**:

```
task welcome {
    group 'Sample category'
    doLast {
        println 'Welcome on the Baeldung!'
    }
}
```

现在，当我们运行 Gradle 命令列出所有可用任务时(`–all`选项不再需要)，我们将在新组下看到我们的任务:

```
Sample category tasks
---------------------
welcome 
```

然而，让其他人看到任务负责什么也是有益的。我们可以**创建包含简短信息的描述**:

```
task welcome {
    group 'Sample category'
    description 'Tasks which shows a welcome message'
    doLast {
        println 'Welcome in the Baeldung!'
    }
} 
```

当我们打印可用任务的列表时，输出如下:

```
Sample category tasks
---------------------
welcome - Tasks which shows a welcome message 
```

这种任务定义称为**临时定义**。

进一步来说，创建一个定义可以重用的可定制任务是有益的。我们将讨论如何从一个类型中创建一个任务，以及如何对这个任务的用户进行一些定制。

## 4。在 `build.gradle` 中定义梯度任务类型

上面的“欢迎”任务不能定制，因此，在大多数情况下，它不是很有用。我们可以运行它，但是如果我们在不同的项目(或者子项目)中需要它，那么我们需要复制并粘贴它的定义。

我们可以通过创建任务类型来快速**实现任务的定制。仅仅是，任务类型是在构建脚本中定义的:**

```
class PrintToolVersionTask extends DefaultTask {
    String tool

    @TaskAction
    void printToolVersion() {
        switch (tool) {
            case 'java':
                println System.getProperty("java.version")
                break
            case 'groovy':
                println GroovySystem.version
                break
            default:
                throw new IllegalArgumentException("Unknown tool")
        }
    }
}
```

一个**定制任务类型是一个简单的 Groovy 类，它扩展了`DefaultTask`**——定义标准任务实现的类。我们可以扩展其他任务类型，但是在大多数情况下，`DefaultTask`类是合适的选择。

`PrintToolVersionTask` **任务包含工具属性，可由该任务的**实例定制；

```
String tool 
```

我们可以添加任意多的属性——记住这只是一个简单的 Groovy 类字段。

此外，它还包含用`@TaskAction`注释的**方法。它定义了这个任务正在做什么**。在这个简单的例子中，它打印已安装的 Java 或 Groovy 的版本——取决于给定的参数值。

为了基于创建的任务类型运行定制任务，我们需要**创建一个这种类型的新任务实例**:

```
task printJavaVersion(type : PrintToolVersionTask) {
    tool 'java'
} 
```

最重要的部分是:

*   我们的任务是一个`PrintToolVersionTask`类型，所以当**被执行时，它将触发用`@TaskAction`** 标注的方法中定义的动作
*   我们添加了一个定制的工具属性值(`java`)，将由`PrintToolVersionTask`使用

当我们运行上述任务时，输出与预期的一样(取决于安装的 Java 版本):

```
> Task :printJavaVersion 
9.0.1 
```

现在让我们创建一个打印 Groovy 安装版本的任务:

```
task printGroovyVersion(type : PrintToolVersionTask) {
    tool 'groovy'
} 
```

它使用与我们之前定义的相同的任务类型，但是它具有不同的工具属性值。当我们执行这个任务时，它打印 Groovy 版本:

```
> Task :printGroovyVersion 
2.4.12 
```

**如果我们没有太多的自定义任务，那么我们可以直接在`build.gradle`文件**中定义它们(就像我们上面做的那样)。然而，如果有很多，那么我们的`build.` gradle 文件将变得难以阅读和理解。

幸运的是，Gradle 提供了一些解决方案。

## 5。在`buildSrc`文件夹中定义任务类型

我们可以在位于根项目级别的`buildSrc`文件夹中**定义任务类型。Gradle 编译里面的所有内容，并将类型添加到类路径中，以便我们的构建脚本可以使用它。**

我们之前定义的任务类型(`PrintToolVersionTask`)可以移到`buildSrc/src/main/groovy/com/baeldung/PrintToolVersionTask.groovy`。我们只需将一些从 Gradle API 导入的**添加到一个移动的类**中。****

我们可以在`buildSrc`文件夹中定义无限数量的任务类型。它更容易维护、阅读，并且任务类型声明不在与任务实例化相同的位置。

我们可以像使用直接在构建脚本中定义的类型一样使用这些类型。我们必须记住只添加适当的导入。

## 6。在插件中定义任务类型

我们可以在一个自定义的 Gradle 插件中定义一个自定义的任务类型。请参考[这篇文章](/web/20221129015002/https://www.baeldung.com/gradle-create-plugin)，它描述了如何定义一个自定义的 Gradle 插件，定义在:

*   `build.gradle`文件
*   `buildSrc`文件夹作为其他 Groovy 类

当我们定义了对这个插件的依赖时，这些自定义任务将可用于我们的构建。请注意，临时任务也是可用的，不仅仅是自定义任务类型。

## 7。结论

在本教程中，我们介绍了如何在 Gradle 中创建自定义任务。你可以在你的`build.gradle` 文件中使用许多插件，这些插件将提供你需要的许多自定义任务类型。

和往常一样，代码片段可以在 Github 上找到[。](https://web.archive.org/web/20221129015002/https://github.com/eugenp/tutorials/tree/master/gradle-modules/gradle)