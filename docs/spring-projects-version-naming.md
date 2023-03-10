# Spring Projects 版本命名方案

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-projects-version-naming>

## 1.概观

在命名发布版本时，通常使用[语义版本化](/web/20221208143841/https://www.baeldung.com/cs/semantic-versioning)。例如，这些规则适用于版本格式，如`MAJOR.MINOR.REVISION`:

*   主要:主要特性和潜在的突破性变化
*   次要:向后兼容的功能
*   版本 **:** 向后兼容的修正和改进

与语义版本化一起，项目经常使用标签来进一步阐明特定版本的状态。事实上，通过使用这些标签，我们给出了关于构建生命周期或者工件发布位置的提示。

在这篇简短的文章中，我们将研究主要 Spring 项目采用的版本命名方案。

## 2.Spring 框架和 Spring Boot

除了语义版本控制，我们可以看到 Spring Framework 和 Spring Boot 使用了这些标签:

*   构建快照
*   M[ `number` ]
*   RC[ `number` ]
*   释放；排放；发布

构建快照是当前的开发版本。Spring 团队每天都会制作这个神器，并部署到 https://repo.spring.io/ui/native/snapshot。

里程碑发布(M1、M2、M3……)标志着发布过程中的一个重要阶段。当开发迭代完成时，团队构建这个工件，并将其部署到 https://repo.spring.io/ui/native/milestone 的。

候选版本(RC1、RC2、RC3……)是构建最终版本之前的最后一步。为了最大限度地减少代码变更，在这个阶段应该只进行错误修复。它也被部署到 https://repo.spring.io/ui/native/milestone 的。

在发布过程的最后，Spring 团队发布了一个版本。因此，这通常是唯一的生产就绪工件。我们也可以将此版本称为 **GA，用于正式发布**。

**这些标签按字母顺序排列**以确保构建和依赖管理器正确地确定一个版本是否比另一个版本更新。例如，Maven 2 错误地认为 1.0 版快照比 1.0 版更新。Maven 3 修复了这种行为。因此，当我们的命名方案不是最佳的时候，我们会经历奇怪的行为。

## 3.总括项目

伞式项目，如 Spring Cloud 和 Spring Data，是独立但相关的子项目。为了避免与这些子项目冲突，一个伞状项目采用不同的命名方案。每个版本都有一个特殊的名字，而不是一个编号的版本。

按照字母顺序，伦敦地铁站是春季云发布名称的灵感来源——首先是安吉尔、布里克斯顿、芬奇利、格林威治和霍克斯顿。

除了上面显示的 Spring 标签，它**还定义了一个服务发布标签(SR1，SR2…)** 。如果我们发现一个严重的错误，可以发布一个服务版本。

重要的是要认识到 Spring Cloud 版本只与特定的 Spring Boot 版本兼容。作为参考， [Spring Cloud 项目页面](https://web.archive.org/web/20221208143841/https://spring.io/projects/spring-cloud)包含兼容性表。

## 4.结论

如上所示，有一个清晰的版本命名方案是很重要的。虽然一些版本，如里程碑或候选版本可能是稳定的，但我们应该始终使用生产就绪的工件。你的命名方案是什么？它比这个有什么优势？