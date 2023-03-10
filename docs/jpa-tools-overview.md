# JPA 支持—2021 年工具生态系统的状态

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-tools-overview>

 ![announcement-icon.png](img/aba27cb66ef71752bfbae1d47789cc0a.png)

JPA can behave very differently depending on the exact circumstances under which it is used. Code that works in our local environment or in staging performs very poorly (or even flat out fails) when thrown against real-scale databases in production environments.

Debugging these JPA issues in production is pretty difficult - existing APMs don’t provide enough granular insights at the code level, and tracking every single place someone queried entities one by one instead of in bulk can be a grueling, time-consuming task.

**Lightrun** is a new approach to debugging in production. Using Lightrun’s Logs and Snapshots, you can now get debugger-level granularity in production without opening inbound ports, redeploying, restarting, or even stropping the running application.

In addition, instrumenting Lightrun Metrics at runtime allows you to track down persistence issues securely and in real-time. Want to see it in action? **Check out our 2-minute tutorial** for debugging JPA performance issues in production using Lightrun:

[>> Debugging Spring Persistence and JPA Issues Using Lightrun](/web/20220524235121/https://www.baeldung.com/lightrun-n-jpa)

## 1.概观

在本教程中，我们将看看 JPA 的一些支持工具。我们将关注两个最流行的 ide 可用的插件:IntelliJ IDEA 和 Eclipse 。

## 2.IntelliJ IDEA 和 Eclipse 中的 JPA 支持

JPA 是在 Java 应用程序中使用关系数据库最广泛的规范。事实上， **JPA 定义了实现的所有方面，从注释到数据处理规则。**

通常，我们并不仅仅在 JPA 实体上工作。除了纯粹的 ORM 相关代码，我们可能还需要数据库版本控制系统、SQL/JPQL/HQL 查询优化、与 IoC 容器的集成等等。 插件在这里大有用武之地。它们可以支持数据库逆向工程、模式生成、迁移脚本生成或 Spring Data JPA 存储库搭建。

当谈到在应用程序开发中使用 JPA 框架时，通常是在 IDE 的帮助下完成的。这是因为他们为我们提供了**一套强大的工具来提高开发人员的生产力:**

*   样板代码生成
*   数据库逆向工程
*   数据对象定义生成
*   【Java 和 JPQL 的高级代码自动完成功能
*   特定于 JPA 的代码建议
*   等。

让我们看看两个最常见的 ide——IntelliJ IDEA 和 Eclipse——看看它们在 JPA 应用程序开发支持方面提供了什么。

## 3.智能理念

IntelliJ IDEA 有两个版本:社区版(免费)和旗舰版(付费)。终极版有一个捆绑的支持 JPA 的[插件](https://web.archive.org/web/20220524235121/https://www.jetbrains.com/help/idea/jakarta-persistence-jpa.html)。

另一方面，Community Edition 不提供对 JPA、Hibernate、Spring Data 等的专门支持。

然而，IntelliJ 有着广阔的市场和不同的插件。因此，我们几乎可以找到任何技术的支持，JPA 也不例外。

**对于 IntelliJ IDEA 社区用户，我们将覆盖 [JPA 好友](/web/20220524235121/https://www.baeldung.com/jpa-buddy-post)插件**来支持 JPA 特性。然而，它也是 IntelliJ IDEA Ultimate 的一个很好的补充。

### 3.1.IntelliJ Ultimate 的 JPA 插件

该插件提供了一系列高级功能，包括:

*   JPA 实体的 ER 图
*   持久性工具窗口
*   数据库逆向工程
*   JPA 控制台测试 JPQL 语句并生成 DDL 脚本
*   JPA 特定代码检查和完成

在 IntelliJ IDEA Ultimate 中，我们可以使用一个专用的工具窗口——“持久性”。这将显示项目中 JPA 实体的结构。

在该视图中，我们可以使用可视化工具创建实体属性和关系:

[![](img/3bfe43375a805a204d010bb4451aef4b.png)](/web/20220524235121/https://www.baeldung.com/wp-content/uploads/2021/06/persistence.png)

“持久性”窗口还允许使用代码编辑器中的装订线图标在代码及其相关实体层次结构之间快速导航。

我们还可以利用逆向工程流程。它将为我们生成 JPA 实体、关联和适当的注释。此外，它会创建一个`pesistence.xml`文件，或者更新它(如果存在的话)。

除了一般的 JPA 支持，如果我们选择 Hibernate 作为 JPA 实现，它将提供额外的帮助。我们可以执行特定于 Hibernate 的操作，例如(但不限于):

*   管理配置和映射文件
*   在 HQL 控制台执行 HQL 查询

在 [Youtube](https://web.archive.org/web/20220524235121/https://www.youtube.com/watch?v=QJddHc41xrM) 上有一篇详细的评论，展示了该插件的运行情况。

### 3.2.JPA 好友

**[JPA Buddy](/web/20220524235121/https://www.baeldung.com/jpa-buddy-post) 提供了一组可视化编辑器来处理 JPA 实体和 Spring 数据 JPA** 存储库。此外，它同时支持 Java 和 Kotlin 语言。

让我们看看它提供的主要功能:

*   可视化工具和代码导航
*   Hibernate bean 验证支持
*   Hibernate 类型和 JPA 转换器支持
*   数据库版本工具支持(Liquibase 和 Flyway)
*   特定于 JPA 的代码检查
*   科特林支持
*   Spring 数据仓库可视化设计器

我们不需要记住所有的 JPA 注释、Spring 数据仓库方法命名规则或 Liquibase 标签。该插件允许我们从调色板中选择所需的项目。

因此，**它使我们能够在可视化编辑器**中更新它们的属性，而不是编写代码:

[![](img/9a811089ad3fa0169f211059e8f73d84.png)](/web/20220524235121/https://www.baeldung.com/wp-content/uploads/2021/06/buddy3.png)

JPA Buddy 解决的另一个挑战是数据库版本控制。该插件支持 Liquibase 和 Flyway 工具，并可以相应地生成 XML 或 SQL 迁移脚本。

JPA Buddy 通过将项目数据库与代码库中定义的实际 JPA 实体进行比较来生成脚本。在生成之后，我们可以检查生成的脚本，并在将它们保存到代码库之前更新它们。

除此之外，**插件还引入了几个智能检查和快速修复**，比如:

*   ID 字段在实体代码中缺失
*   无参数构造函数缺失
*   Lombok 用法的潜在问题(例如不正确的 equals() & hashCode()定义)

除了 JPA 支持，插件**可以帮助创建 Spring 数据 JPA 库**。它提供了可视化编辑器，可以帮助我们正确命名存储库方法。

## 4.黯然失色

众所周知，Eclipse 有一个很大的插件生态系统。然而， **Dali 工具插件是 Eclipse IDE** 中 JPA 支持的事实上的标准。

让我们来探索一下这个插件。然后，如果我们在项目中使用 Hibernate，我们将看到如何获得一些额外的支持。

### 4.1.达利工具

**大理几乎涵盖了 JPA 发展的所有领域**。其功能包括(但不限于):

*   JPA 实体的 ER 图
*   可视化编辑器和导航器
*   基于 JPA 模型的 DDL (SQL)生成
*   数据库逆向工程

插件提供了**两个主要的可视化工具**。上下文相关的“JPA 细节”编辑器允许我们使用不同的 JPA 对象，我们可以在“JPA 结构”导航器中选择这些对象:

[![](img/9b3ebc4c39af4ceb4c9bb329a7e293f6.png)](/web/20220524235121/https://www.baeldung.com/wp-content/uploads/2021/06/dali.png)

我们可以选择和更改:

*   实体定义，包括命名查询(借助 JPQL 代码)
*   实体属性和关联:一对多、多对一等。
*   `orm.xml`文件。我们可以编辑项目的默认映射和持久性信息

此外，一个专用的`persistence.xml`编辑器允许我们根据 JPA 规范避免记忆所有可以在这个配置文件中使用的参数。我们可以编辑从连接选项到锁定超时或验证模式的所有内容。

Dali Tools 插件**可以基于 JPA 模型为特定数据库生成并执行 DDL SQL** 。此外，执行阶段与生成阶段是分开的，因此我们可以在将生成的 SQL 应用到应用程序的数据库之前对其进行检查。

数据库逆向工程过程设置允许我们更改实体代码中的以下选项:

*   PK 生成策略
*   字段访问策略
*   关联提取策略
*   集合属性类(列表，集合，…)

### 4.2.JBoss Hibernate 工具

如果我们需要专门针对 Hibernate 的支持，让我们来看一个选项。在 Eclipse 插件库中，我们可以找到 Jboss Hibernate 工具插件。

**该插件可以单独使用，也可以与 JPA 工具**结合使用，并提供:

*   代码生成工具，包括逆向工程和重构
*   高级 Hibernate 配置和映射文件显示和编辑
*   HQL 执行控制台和 SQL 预览
*   数据模型导出

逆向工程工具允许配置许多选项，包括表名掩码、类型映射，你甚至可以从生成的实体类中排除一些列:

[![](img/976548faff9ad60e9ed42fc7ca240f17.png)](/web/20220524235121/https://www.baeldung.com/wp-content/uploads/2021/06/features-hibernate-launchconfig-reveng.png)

生成的 JPA 实体可以表示为带有 `*.hbm.xml`映射文件的 POJOs，或者表示为带注释的类。此外，该插件支持使用 CRUD 方法为每个实体生成 DAO 对象。DAO 利用特定于 Hibernate 的类进行数据库访问。

HQL 执行控制台和标准编辑器允许我们生成和执行 HQL 查询。除了执行任意的 HQL，控制台还为 HQL 语句提供 SQL 预览。这对于创建优化的 SQL 查询以避免“n+1 选择”和其他 ORM 边缘情况可能是有用的。

## 5.结论

在本文中，我们介绍了在最流行的 Java IDEs 中支持 JPA 的几种工具:IntelliJ IDEA 和 Eclipse。

我们看到了这些如何在开发过程中帮助编写更好的应用程序并提高生产率。因此，在为工作选择最佳工具时，我们不仅要注意“纯 JPA”特性，还要注意与 IDE 和所选开发堆栈的集成。