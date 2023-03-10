# Java 周刊，第 466 期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-weekly-466>

## 1. **Spring 和 Java**

**[>>Spring Boot 3.0 Goes GA](https://web.archive.org/web/20221204204959/https://spring.io/blog/2022/11/24/spring-boot-3-0-goes-ga)**spring . io

**期待已久的 Spring Boot 版来了**！哦——这么多好东西，都基于 Java 17，更好的原生支持，还有很多值得探索的地方！

**[>>Spring Integration 6.0 goes GA](https://web.archive.org/web/20221204204959/https://spring.io/blog/2022/11/29/spring-integration-6-0-goes-ga)**Spring . io

还有，认识一下 **Spring Integration 6** :基于 Java 17，支持 AOT 和 GraalVM，增强的插装，以及 Jakarta EE 10。

**[> >反应性积垢性能:案例分析](https://web.archive.org/web/20221204204959/https://quarkus.io/blog/reactive-crud-performance-case-study/)** [ quarkus.io

关于基准测试的棘手本质:如何一步一步地**将基准测试从 1 RPS 提高到 26K RPS！**

#### 同样值得一读:

*   **[> > Spring Boot 3 和 Spring Framework 6 使用 Java 17 和 Jakarta EE 9，用 GraalVM](https://web.archive.org/web/20221204204959/https://www.infoq.com/news/2022/11/spring-6-spring-boot-3-launch/?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=Java)T3【infoq.com】支持原生 Java**
*   **[> >异常 Java: StackTrace 扩展 Throwable](https://web.archive.org/web/20221204204959/https://foojay.io/today/unusual-java-stacktrace-extends-throwable/)**[foojay . io]
*   **[> >贡献 BigInteger.parallelMultiply()给 open JDK](https://web.archive.org/web/20221204204959/https://www.javaspecialists.eu/archive/Issue305-Contributing-BigInteger.parallelMultiply-to-OpenJDK.html)**[javaspecialists . eu]
*   **[> >文章:Java 冠军乔希·朗论 Spring 框架 6 与 Spring Boot 3](https://web.archive.org/web/20221204204959/https://www.infoq.com/articles/josh-long-spring-6/?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=Java)**【infoq.com】
*   **[> >流言蜚语:闲聊街区的活动巴士](https://web.archive.org/web/20221204204959/https://foojay.io/today/metaphorical-programming-gossips-event-bus/)**foojay . io
*   **[> >雅加达休息 3.1 雅加达 EE 10 有什么新鲜事？](https://web.archive.org/web/20221204204959/https://blog.payara.fish/whats-new-in-jakarta-rest-3.1-in-jakarta-ee-10)T3blog . payara . fish**

**网络研讨会和演示:**

*   **[**>>**](https://web.archive.org/web/20221204204959/https://blog.sebastian-daschner.com/entries/capitalize-titles)[Spring Tips:通往 Spring 框架之路 6:全新提前编译引擎和 GraalVM](https://web.archive.org/web/20221204204959/https://spring.io/blog/2022/11/23/spring-tips-the-road-to-spring-framework-6-the-new-ahead-of-time-compilation-engine-and-graalvm)**Spring . io
*   **[> >如何使用 Java 记录与夸尔库斯](https://web.archive.org/web/20221204204959/https://blog.sebastian-daschner.com/entries/java-records-quarkus-enterprise)**【blog.sebastian-daschner.com
*   **[> >演示:面向 Java 开发者的 devo PS](https://web.archive.org/web/20221204204959/https://www.infoq.com/presentations/devops-java-devs/?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=Java)**【infoq.com】
*   **[> >展示:在 Java 中使用共享内存映射文件](https://web.archive.org/web/20221204204959/https://www.infoq.com/presentations/java-shared-memory-files/?utm_campaign=infoq_content&utm_source=infoq&utm_medium=feed&utm_term=Java)**【infoq.com】
*   **[> > #92: Clojure:一种将改变你思考编程方式的语言](https://web.archive.org/web/20221204204959/https://nurkiewicz.com/92)**【nurkiewicz.com

**升级时间:**

*   **[>>Spring Boot 2 . 6 . 14 和](https://web.archive.org/web/20221204204959/https://spring.io/blog/2022/11/24/spring-boot-2-6-14-available-now) [2.7.6 现已上市](https://web.archive.org/web/20221204204959/https://spring.io/blog/2022/11/24/spring-boot-2-7-6-available-now)**spring . io
*   **[> > Spring 框架 6.0.2 现已可用](https://web.archive.org/web/20221204204959/https://spring.io/blog/2022/11/24/spring-framework-6-0-2-available-now)**Spring . io
*   **[>>Spring Modulith 0.1 发布](https://web.archive.org/web/20221204204959/https://spring.io/blog/2022/11/24/spring-modulith-0-1-released)**Spring . io
*   **[> >春批 5.0 谬嘎！](https://web.archive.org/web/20221204204959/https://spring.io/blog/2022/11/24/spring-batch-5-0-goes-ga)**spring . io
*   [**> >弹簧跳马 3.0 谬 GA**](https://web.archive.org/web/20221204204959/https://spring.io/blog/2022/11/28/spring-vault-3-0-goes-ga)Spring . io
*   **[> >夸尔库斯 2.14.2 .最终发布](https://web.archive.org/web/20221204204959/https://github.com/quarkusio/quarkus/releases/tag/2.14.2.Final)** [ 夸尔库斯. io
*   **[> >骆驼-3.18.4 上映](https://web.archive.org/web/20221204204959/https://github.com/apache/camel/releases/tag/camel-3.18.4)**【apache.org】

## 2。技术&思考

**[> >网页缓存](https://web.archive.org/web/20221204204959/https://blog.frankel.ch/web-caching/)**[blog . frankel . ch]

通过缓存提高性能的各种方法的详尽目录:客户端、浏览器和服务器端。有趣的读物。

**同样值得一读:**

*   **[> > registry.k8s.io:更快、更便宜、普遍可用(GA)](https://web.archive.org/web/20221204204959/https://kubernetes.io/blog/2022/11/28/registry-k8s-io-faster-cheaper-ga/)**[kubernetes . io]
*   **[> >嘘……Kubernetes 的秘密其实不是秘密！](https://web.archive.org/web/20221204204959/https://auth0.com/blog/kubernetes-secrets-management/)**【auth0.com】
*   **[>>Kubernetes](https://web.archive.org/web/20221204204959/https://reflectoring.io/blog/2022/2022-11-24-6-cloud-cost-management-practices/)**[reflector ing . io]的 6 个行之有效的云成本管理实践
*   **[> >你做错了——招一个 DevRel](https://web.archive.org/web/20221204204959/https://foojay.io/today/youre-doing-it-wrong-recruiting-a-devrel/)**[foojay . io]
*   **[> >按照惯例映射事件类型](https://web.archive.org/web/20221204204959/https://event-driven.io/en/hot_to_map_event_type_by_convention/)**事件驱动. io
*   **[> > Web 资源缓存:客户端](https://web.archive.org/web/20221204204959/https://blog.frankel.ch/web-caching/client/)** [ blog.frankel.ch
*   **[> >使用 UNION、INTERSECT 的最佳方式，除了与冬眠的](https://web.archive.org/web/20221204204959/https://vladmihalcea.com/hibernate-union-intersect-except/)**【vladmihalcea.com】
*   **[> >事业画布](https://web.archive.org/web/20221204204959/https://blog.scottlogic.com/2022/11/29/The-career-canvas.html)**【blog.scottlogic.com】
*   **[> >规则助你走得更快](https://web.archive.org/web/20221204204959/https://blog.scottlogic.com/2022/11/30/rules-help-you-go-faster.html)**【blog.scottlogic.com】

## 4。漫画

**[> >幸存者的罪恶感](https://web.archive.org/web/20221204204959/https://dilbert.com/strip/2022-12-03)**【dilbert.com】

**[> >一切照旧](https://web.archive.org/web/20221204204959/https://dilbert.com/strip/2022-12-02)**【dilbert.com】

**[> >道格伯特帮忙裁员](https://web.archive.org/web/20221204204959/https://dilbert.com/strip/2022-12-01)**【dilbert.com】

## 5。本周精选

**[> >我是一个有着果蝇般记忆力的多产程序员](https://web.archive.org/web/20221204204959/https://hynek.me/articles/productive-fruit-fly-programmer/)**hynek . me