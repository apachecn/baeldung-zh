# Apache Maven 标准目录布局

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-directory-structure>

## 1。简介

Apache Maven 是 Java 项目最流行的构建工具之一。除了分散依赖项和存储库之外，促进跨项目的统一目录结构也是它的一个重要方面。

在这篇简短的文章中，我们将探索典型 Maven 项目的标准目录布局。

## 2。目录布局

典型的 Maven 项目有一个`pom.xml`文件和一个基于已定义约定的目录结构:

```java
└───maven-project
    ├───pom.xml
    ├───README.txt
    ├───NOTICE.txt
    ├───LICENSE.txt
    └───src
        ├───main
        │   ├───java
        │   ├───resources
        │   ├───filters
        │   └───webapp
        ├───test
        │   ├───java
        │   ├───resources
        │   └───filters
        ├───it
        ├───site
        └───assembly
```

可以使用项目描述符覆盖默认的目录布局，但这种做法并不常见，也不鼓励使用。

在本文中，我们将揭示每个标准文件和子目录的更多细节。

## 3。根目录

这个目录是每个 Maven 项目的根目录。

让我们仔细看看通常位于根目录下的标准文件和子目录:

*   `maven-project/pom.xml`–定义 Maven 项目构建生命周期中所需的依赖项和模块
*   `maven-project/LICENSE.txt`–项目许可信息
*   `maven-project/README.txt`–项目总结
*   `maven-project/NOTICE.txt`–关于项目中使用的第三方库的信息
*   `maven-project/src/main`–包含成为工件一部分的源代码和资源
*   `maven-project/src/test`–保存所有的测试代码和资源
*   `maven-project/src/it`–通常保留给`Maven Failsafe Plugin`使用的集成测试
*   `maven-project/src/site`–使用`Maven Site Plugin`创建的现场文件
*   `maven-project/src/assembly`–打包二进制文件的汇编配置

## 4。`src/main`目录

顾名思义，`src/main`是一个 Maven 项目最重要的目录。任何被认为是神器一部分的东西，不管是`jar`还是`war`，都应该出现在这里。

它的子目录是:

*   `src/main/java`–工件的 Java 源代码
*   `src/main/resources`–配置文件和其他文件，如 `i18n`文件、每个环境的配置文件和 XML 配置
*   `src/main/webapp`–对于 web 应用程序，包含 JavaScript、CSS、HTML 文件、视图模板和图像等资源
*   `src/main/filters`–包含在构建阶段将值注入资源文件夹中的配置属性的文件

## 5。`src/test`目录

目录`src/test`是应用程序中每个组件的测试所在的地方。

注意，这些目录或文件都不会成为工件的一部分。让我们看看它的子目录:

*   `src/test/java`–用于测试的 Java 源代码
*   `src/test/resources`–测试使用的配置文件和其他文件
*   `src/test/filters`–包含在测试阶段将值注入资源文件夹中的配置属性的文件

## 6。结论

在本文中，我们研究了 Apache Maven 项目的标准目录布局。

Maven 项目结构的多个例子可以在 [GitHub 项目](https://web.archive.org/web/20220628152849/https://github.com/eugenp/tutorials/tree/master/maven-modules)中找到。