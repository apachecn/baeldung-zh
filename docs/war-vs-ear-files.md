# WAR 和 EAR 文件的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/war-vs-ear-files>

## 1.概观

在本教程中，我们将看看 Java JAR (Java 归档)、WAR (Web 应用程序归档)和 EAR(企业应用程序归档)工件文件之间的区别。

我们将讨论每种文件格式的功能以及如何使用它。

## 2.Java 工件文件格式

Java 规范包含多种类型的工件文件。最突出的是罐子、战争和耳朵，但也有其他的。从 JAR 开始，每种类型分别建立在另一种之上。

尽管 JAR 文件格式是 JavaSE 规范的一部分，但 WAR 和 EAR 文件本身也是 JavaEE(现在称为 JakartaEE)的一部分。

## 3.JAR 文件

众所周知，**JAR 文件是 Java 应用程序**的基本构件。

本质上，它是一个扩展名为`.jar`的`.zip`文件。它包括类、资源和元信息文件等内容。

**我们可以用`jar`程序[创建 JAR 文件](/web/20220820180507/https://www.baeldung.com/java-create-jar)，并使用`java -jar`命令**在任何只有 Java 运行时环境(JRE)的系统上[运行](/web/20220820180507/https://www.baeldung.com/java-create-jar#running)它们。

## 4.战争档案

**WAR 文件用于** **捆绑 Java web 应用**。

它们也是幕后的`.zip`文件，是 JAR 文件格式的扩展。

**除了类和资源，WAR 文件还可以包含库形式的其他 jar、 [JSP 文件](/web/20220820180507/https://www.baeldung.com/jsp)、 [Java Servlets](/web/20220820180507/https://www.baeldung.com/intro-to-servlets) ，以及各种类型的静态 web 文件** (HTML、CSS、Javascript 等。).

JakartaEE 指定格式正确的 WAR 文件必须包含一个`WEB-INF`目录，该目录本身包含一个`web.xml`文件。这是我们声明 web 应用程序的结构和配置的地方，比如每个 servlet 的 web 路径或者会话超时时间。

与 jar 相比， **WAR 文件不能作为独立的应用程序运行，我们只能将它们作为另一个应用程序的组件，比如 servlet 容器或应用服务器**。

将 WAR 文件放入服务器的过程称为应用程序部署。典型的应用服务器包括 Tomcat、Jetty 和 Wildfly。

## 5.EAR 文件

**EAR 文件用于由多个模块组成的企业应用。**

有多种类型的模块，但最常见的是 web 模块，本质上是 WAR 文件，以及 [EJB 模块](/web/20220820180507/https://www.baeldung.com/ejb-intro)，它们是包含企业 Java Bean 类的特殊类型的 JAR 文件。

与 WAR 类似，EAR 是一个 JAR 扩展，必须包含一个名为`application.xml`的特殊 XML 文件，位于根目录`META-INF`下。在这个文件中，我们描述了企业应用程序并列出其模块。此外，我们可以为整个应用程序添加安全角色。

像 WAR 一样，EAR 文件也不能作为独立的应用程序运行。我们必须将它部署在应用服务器上。

但是，在这种情况下，并非所有类型都兼容。我们必须使用支持 JakartaEE 技术如 Wildfly 的服务器，但我们不能使用 Jetty。至于 Tomcat，也是不兼容的，但是有一种不同的味道，叫 TomEE，是兼容的。

EAR 格式的优点是它允许我们在单个文件中指定一个复杂的多组件应用程序。此外，我们可以通过共享公共资源来避免重复。最后，单一文件格式非常有用，因为它消除了部署错误。

## 6.结论

在本文中，我们研究了 WAR 和 EAR 文件之间的差异。我们学习了每个文件的结构，并讨论了它们的典型用例。最后，我们看到了如何使用每种类型的工件。