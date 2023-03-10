# Java 报告工具:比较

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-reporting-tools-comparison>

## 1.概观

当我们谈到`Reporting tools`的时候，很多软件都涵盖了这个领域。然而，大多数都是羽翼丰满的`Business Intelligence platforms`或`Cloud services`。

但是，如果我们只是想在作为库的应用程序中添加一些报告特性，会发生什么呢？我们将在这里回顾一些非常适合这一目的的`Java reporting tools` 。

我们将主要关注这些开源工具:

*   `[BIRT](https://web.archive.org/web/20220627180516/https://www.eclipse.org/birt/)`
*   `[Jasper Reports](https://web.archive.org/web/20220627180516/https://community.jaspersoft.com/project/jasperreports-library)`
*   `[Pentaho](https://web.archive.org/web/20220627180516/https://github.com/pentaho/pentaho-reporting)`

此外，我们将简要分析以下商业工具:

*   `[FineReport](https://web.archive.org/web/20220627180516/https://www.finereport.com/en/)`
*   `[Logi Report](https://web.archive.org/web/20220627180516/https://www.logianalytics.com/jreport/)`(原`JReport`)
*   `[Report Mill](https://web.archive.org/web/20220627180516/http://www.reportmill.com/product/)`

## 2.设计报表

通过这一部分，我们将回顾如何可视化地设计报表和处理数据。注意，在这一部分中，我们将只提到开源工具。

### 2.1.可视化编辑器

所有这三个工具都包括一个带有报告预览功能的 WYSIWIG 编辑器。

`BIRT Report Designer`和`Jaspersoft Studio`是构建在 [Eclipse RCP](https://web.archive.org/web/20220627180516/https://wiki.eclipse.org/Rich_Client_Platform) 上的工具。对于我们大多数 Java 开发人员来说，这是一个很好的观点，因为我们可能熟悉 Eclipse 环境。与那些不同的是，`Pentaho Report Designer`T5 已经老成视觉不佳。

另外，关于`Jaspersoft Studio`还有一个有趣的特性:我们可以直接在他们的`Jasper Reports Server` (报告管理系统)上发布我们的报告。

### 2.2.数据集

与所有报告工具一样，我们可以通过查询一个`datasource`来检索数据集(见下文)。然后，我们可以将它们转换成报表字段，创建计算字段，或者使用聚合公式。

除此之外，比较一下**我们如何管理多个数据集**也很有趣，因为如果我们的数据来自不同的查询，甚至不同的`datasources`，我们可能需要几个数据集:

*   **`BIRT`提供了最简单的解决方案，因为我们可以在同一个报告中拥有多个数据集**
*   使用`Jasper Reports`和`Pentaho`，我们每次都需要创建一个单独的子报表，这可能会非常棘手

### 2.3.图表和视觉元素

所有工具都提供了简单的元素，如形状和图像，以及每种图表风格:`lines`、`areas`、`pies`、`radar`、`ring`等。它们都支持交叉标签。

然而， **`Jasper Reports`提供了最丰富的视觉元素集合**。它增加了上述列表中的`maps`、`sparklines`、`pyramids`和`Gantt diagrams`。

### 2.4.造型报告

现在，让我们比较页面中元素的位置和大小:

*   所有的工具都提供像素定位
*   **`BIRT`和`Pentaho`还提供了类似 HTML 的定位**(表格、块、内联)
*   它们都不支持类似 CSS 的 flexbox 或网格系统来控制元素大小

此外，当我们必须管理多个报告时，我们可能希望共享相同的视觉主题:

*   `Jasper Reports`提供具有 XML-CSS 语法的主题文件
*   `BIRT`可以将 CSS 样式表导入到设计系统中
*   使用`Pentaho`，我们只能在页眉中添加 CSS 样式表。所以很难将它们与内部设计系统混合在一起

## 3.呈现报表

现在，我们已经看到了如何设计报告，让我们比较如何以编程方式呈现它们。

### 3.1.装置

首先，让我们注意到**所有的工具都被设计成容易嵌入 Java 项目**中。

首先，你可以看看我们关于 [BIRT](https://web.archive.org/web/20220627180516/https://baeldung.com/birt-reports-spring-boot#dependencies) 和[贾斯珀报道](https://web.archive.org/web/20220627180516/https://baeldung.com/spring-jasper#maven-dependency)的专题文章。对于 Pentaho，有一个[帮助页面](https://web.archive.org/web/20220627180516/https://help.pentaho.com/Documentation/9.0/Developer_center/Embed_reporting_functionality)和免费的[代码样本](https://web.archive.org/web/20220627180516/https://github.com/fcorti/pentaho-8-reporting-for-java-developers)。

接下来，对于每个工具，我们将把报告引擎连接到我们的应用程序数据。

### 3.2.数据源

我们应该问的第一个问题是:我们如何将报告引擎连接到我们的项目数据源？

*   `Jasper Reports`:我们只是把它作为 [`fillReport`方法](https://web.archive.org/web/20220627180516/https://baeldung.com/spring-jasper#populating-reports)的一个参数添加进去
*   这个问题的解决方案有点复杂:我们应该修改我们的报告，将数据源属性设置为参数
*   **Pentaho 有一个很大的缺点**这里:除非我们买他们的`PDI`商业软件，**我们必须使用一个 JNDI 数据源**，这更难设置

说到数据源，支持哪些类型？

*   这三个工具都支持最常见的类型:`JDBC`、`JNDI`、`POJOs`、`CSV`、`XML`和`MongoDB`
*   **`REST API`是现代项目的一个需求，但是没有一个项目本身支持它**
    *   有了`BIRT`，我们应该编码一个`Groovy script`
    *   `Jasper Reports`需要一个额外的[免费插件](https://web.archive.org/web/20220627180516/https://community.jaspersoft.com/project/web-service-data-source)
    *   有了`Pentaho`，我们应该编码一个`Groovy script`或者收购`PDI`商业软件
*   **JSON 文件由 `Jasper Reports`和`Pentaho`** 原生支持，但是`BIRT`将需要一个外部 Java 解析器库
*   我们可以在这个[矩阵](https://web.archive.org/web/20220627180516/http://www.innoventsolutions.com/comparison-matrix.html)中找到完整的比较列表

### 3.3.参数和运行时定制

因为我们已经将报告连接到数据源，所以让我们呈现一些数据！

现在重要的是如何检索我们的最终用户数据。为此，我们可以向呈现方法传递参数。这些参数应该在我们设计报告时定义，而不是在运行时定义。但是，如果我们的数据集基于不同的查询(取决于最终用户的上下文),我们该怎么办呢？

使用 **`Pentaho`和`Jasper Reports`，根本不可能做到这一点**，因为报告文件是二进制的，没有 Java SDK 来修改它们。相比之下， **`BIRT`报表是纯 XML 文件**。此外，我们可以使用 Java API 来修改它们，所以在运行时定制一切非常容易。

### 3.4.输出格式和 Javascript 客户端

值得庆幸的是，所有工具都支持大多数常见格式:`**HTML, PDF, Excel, CSV, plain text,**` **和** `**RTF**`。现在，我们可能还会问如何将报告结果直接集成到我们的 web 页面中。不过我们不会提到 PDF 可视化工具的粗略包含。

*   最好的解决方案是使用`Javascript` 客户端将报告直接呈现到 HTML 元素中。对于 **BIRT，Javascript 客户端是****`Actuate JSAPI`** **`and`** **对于`Jasper Reports`，我们应该用** `**JRIO.js**`
*   除了 iFrame 集成之外，不提供任何东西。这个解决方案可行，但是可能有严重的缺点

### 3.5.独立渲染工具

除了将我们的报告集成到 web 页面中，我们还可能对现成的呈现服务器感兴趣。每个工具都提供了自己的解决方案:

*   **`BIRT Viewer`** **是一个轻量级的 web 应用程序**示例，用于按需执行`BIRT`报表。**它是开源的，但不包括报告管理功能**
*   **对于`Pentaho`和`Jasper Report`，只有商业软件包**

## 4.项目状态和活动

首先，谈谈许可证。`BIRT`在`EPL`下，`Jasper Reports`在`LGPLv3`下，`Pentaho`在`LGPLv2.1`下。因此，我们可以将所有这些库嵌入到我们自己的产品中，即使它们是商业产品。

然后，我们可以问自己这些开源项目是如何维护的，社区是否仍然活跃:

*   **`Jasper Reports`有一个维护良好的[资源库，](https://web.archive.org/web/20220627180516/https://github.com/TIBCOSoftware/jasperreports)有一个稳定的介质[活动](https://web.archive.org/web/20220627180516/https://www.openhub.net/p/jasperreports)由其编辑 TIBCO 软件**
*   **`BIRT` [知识库](https://web.archive.org/web/20220627180516/https://github.com/eclipse/birt)** 保持不变，但其 **[活跃度](https://web.archive.org/web/20220627180516/https://www.openhub.net/p/birt/commits/summary)自 2015 年 OpenText 收购其编辑器安讯后**就很低了
*   同样，自 2015 年 Hitachi-Vantara 收购以来， **`Pentaho`存储库[活动](https://web.archive.org/web/20220627180516/https://www.openhub.net/p/pentaho-reporting/commits/summary)非常少**

我们可以使用 Stackoverflow 趋势来确认这一点。`[BIRT](https://web.archive.org/web/20220627180516/https://insights.stackoverflow.com/trends?tags=birt)`和`[Pentaho](https://web.archive.org/web/20220627180516/https://insights.stackoverflow.com/trends?tags=pentaho),`的受欢迎程度最低，但`[Jasper Reports](https://web.archive.org/web/20220627180516/https://insights.stackoverflow.com/trends?tags=jasper-reports)`的受欢迎程度适中。

所有这三种 Java 报告工具在过去 5 年中的受欢迎程度都有所下降，尽管目前保持稳定。**我们可以通过云计算和 Javascript 的出现来解释这一点。**

## 5.商业 Java 报告工具

除了开源解决方案，还有一些值得一提的商业选项。

### 5.1.好报告

`Fine Report`最初被设计为作为独立服务器执行。幸运的是，如果我们想使用它，我们可以将它作为项目的一部分。我们必须手动将所有 jar 和资源复制到我们的 WAR 中，如[他们的过程](https://web.archive.org/web/20220627180516/http://doc.fanruan.com/display/VHD/Embedded+Deployment)中所述。

这样做之后，我们可以在我们的项目中看到作为 URL 可用的`Decision-making Platform`工具。从这个 URL，我们可以直接在提供的 web 视图中执行报告，或者使用他们的 Javascript 客户端。**然而，我们不能以编程方式生成报告。**

另一个巨大的限制是目标运行时。版本 10 只支持 Java 8 和 Tomcat 8.x。

### 5.2.Logi 报告(以前的 JReport)

像 Fine Report 一样，Logi Report 被设计成作为一个独立的服务器来执行，但是我们可以将它集成到我们现有的 WAR 项目中。因此，我们将面临与`Fine Report` : **相同的限制:我们不能以编程方式生成报告**。

与罚款报告不同。但是，Logi Report 支持几乎所有的 servlet 容器和 Java 8 到 13。

### 5.3.ReportMill 报告

最后`,` ReportMill 值得一提，因为**我们可以将它平滑地嵌入到每个 Java 应用程序中**。此外，像 BIRT `,` 一样，它非常灵活:**我们可以在运行时定制报告，因为它们是普通的 XML 文件**。

然而，我们可以马上看到 ReportMill 已经老化，与其他解决方案相比，它的功能也很差。

## 6.结论

在本文中，我们介绍了一些最著名的 Java 报告工具，并比较了它们的特性。

最后，我们可以根据自己的需求选择一个 Java 报告工具:

我们会选择 BIRT。

*   对于一个简单的库来说**取代一个现有的自制解决方案**
*   凭借其**最大的灵活性和高度的定制潜力**

**我们将选择 Jasper Reports** :

*   如果我们需要一个与**成熟的报告管理系统**兼容的报告库
*   如果我们想赌上**最好的长期进化并支持**