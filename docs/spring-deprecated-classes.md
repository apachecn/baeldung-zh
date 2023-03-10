# Spring 中弃用的类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-deprecated-classes>

 ![](img/d4cb3fc947cf3ce99113e22e8adead76.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220524010452/https://www.baeldung.com/lightrun-n-security)

## 1。简介

在本教程中，我们将看看 Spring 和 Spring Boot 中被弃用的类，并解释它们被什么所取代。

我们将探索从春天 4 和 Spring Boot 1.4 开始的课程。

## 2.Spring 中弃用的类

为了便于阅读，我们列出了基于 Spring 版本的类及其替换。并且，在每一组类中，我们已经按照类名对它们进行了排序，而不考虑包。

### 2.1.Spring 4.0.x

*   `**org.springframework.cache.interceptor.DefaultKeyGenerator **` `–`替换为基于哈希码的`SimpleKeyGenerator`或自定义`KeyGenerator`实现
*   `**org.springframework.jdbc.support.lob.OracleLobHandler**` `–` `DefaultLobHandler`为 Oracle 10g 及更高版本驱动；即使与 Oracle 9i 数据库相比，我们也应该考虑它
*   我们应该利用 JUnit 4 的`@Test(expected=…)`支持
*   `**org.springframework.http.converter.xml.XmlAwareFormHttpMessageConverter**``–`

从 Spring 4.0.2 开始，以下类被弃用，取而代之的是 CGLIB 3.1 的默认策略，并且在 Spring 4.1 中被删除:

*   `**org.springframework.cglib.transform.impl.MemorySafeUndeclaredThrowableStrategy**`

这个 Spring 版本的所有不推荐使用的类，以及不推荐使用的接口、字段、方法、构造函数和枚举常量都可以在官方文档页面的[中找到。](https://web.archive.org/web/20220524010452/https://docs.spring.io/spring-framework/docs/4.0.x/javadoc-api/)

### 2.2.Spring 4.1.x

*   `**org.springframework.jdbc.core.simple.ParameterizedBeanPropertyRowMapper**``–`
*   `**org.springframework.jdbc.core.simple.ParameterizedSingleColumnRowMapper**``–`

我们可以在 Spring 4.1.x JavaDoc 中找到[完整列表。](https://web.archive.org/web/20220524010452/https://docs.spring.io/spring-framework/docs/4.1.x/javadoc-api/deprecated-list.html)

### 2.3.Spring 4.2.x

*   `**org.springframework.web.servlet.view.document.AbstractExcelView**` `–` `AbstractXlsView`及其`AbstractXlsxView`和`AbstractXlsxStreamingView`变体
*   `**org.springframework.format.number.CurrencyFormatter**``–`
*   `**org.springframework.messaging.simp.user.DefaultUserSessionRegistry**` `–`我们应该使用`SimpUserRegistry`结合`ApplicationListener`监听`AbstractSubProtocolEvent`事件
*   `**org.springframework.messaging.handler.HandlerMethodSelector**` `–`一般化和精细化`MethodIntrospector`
*   我们应该通过反射对期望的 JDK API 变体进行直接检查
*   `**org.springframework.format.number.NumberFormatter**``–`
*   `**org.springframework.format.number.PercentFormatter**``–`
*   `**org.springframework.test.context.transaction.TransactionConfigurationAttributes **` `–`这个类随着`@TransactionConfiguration `在第五春被移除
*   `**org.springframework.oxm.xmlbeans.XmlBeansMarshaller **` `–`在阿帕奇`XMLBeans`退役后

为了支持 Apache Log4j 2，下列类已被弃用:

*   `**org.springframework.web.util.Log4jConfigListener**`
*   `**org.springframework.util.Log4jConfigurer**`
*   `**org.springframework.web.filter.Log4jNestedDiagnosticContextFilter**`
*   `**org.springframework.web.context.request.Log4jNestedDiagnosticContextInterceptor**`
*   `**org.springframework.web.util.Log4jWebConfigurer**`

更多细节可在 [Spring 4.2.x JavaDoc](https://web.archive.org/web/20220524010452/https://docs.spring.io/spring-framework/docs/4.2.x/javadoc-api/deprecated-list.html) 中获得。

### 2.4.Spring 4.3.x

这个版本的 Spring 带来了许多不推荐使用的类:

*   `**org.springframework.web.servlet.mvc.method.annotation.AbstractJsonpResponseBodyAdvice**` `–`该类在 Spring Framework 5.1 中被移除；我们应该用 CORS 来代替
*   `**org.springframework.oxm.castor.CastorMarshaller**` `–`因 Castor 项目缺乏活动而被弃用
*   `**org.springframework.web.servlet.mvc.method.annotation.CompletionStageReturnValueHandler**` `–` `DeferredResultMethodReturnValueHandler`，现在通过适配器机制支持`CompletionStage`返回值
*   `**org.springframework.jdbc.support.incrementer.DB2MainframeSequenceMaxValueIncrementer**` `–`改名为`Db2MainframeMaxValueIncrementer`
*   `**org.springframework.jdbc.support.incrementer.DB2SequenceMaxValueIncrementer **` `–`改名为`Db2LuwMaxValueIncrementer`
*   `**org.springframework.core.GenericCollectionTypeResolver**` `–`已弃用，支持直接使用`ResolvableType`
*   `**org.springframework.web.servlet.mvc.method.annotation.ListenableFutureReturnValueHandler**` `–` `DeferredResultMethodReturnValueHandler`，现在通过适配器机制支持`ListenableFuture`返回值
*   `**org.springframework.jdbc.support.incrementer.PostgreSQLSequenceMaxValueIncrementer**` `–`我们应该用`PostgresSequenceMaxValueIncrementer`来代替
*   `**org.springframework.web.servlet.ResourceServlet**``–`

这些类被弃用，取而代之的是基于`HandlerMethod`的 MVC 基础设施:

*   `**org.springframework.web.servlet.mvc.support.ControllerClassNameHandlerMapping**`
*   `**org.springframework.web.bind.annotation.support.HandlerMethodInvoker**`
*   `**org.springframework.web.bind.annotation.support.HandlerMethodResolver**`

有几个类被弃用，取而代之的是批注驱动的处理程序方法:

*   `**org.springframework.web.servlet.mvc.support.AbstractControllerUrlHandlerMapping**`
*   `**org.springframework.web.servlet.mvc.multiaction.AbstractUrlMethodNameResolver**`
*   `**org.springframework.web.servlet.mvc.support.ControllerBeanNameHandlerMapping**`
*   `**org.springframework.web.servlet.mvc.multiaction.InternalPathMethodNameResolver**`
*   `**org.springframework.web.servlet.mvc.multiaction.ParameterMethodNameResolver**`
*   `**org.springframework.web.servlet.mvc.multiaction.PropertiesMethodNameResolver**`

Spring 中也有很多类，我们应该用它们的 Hibernate 4.x/5.x 等价物来替换:

*   `**org.springframework.orm.hibernate3.support.AbstractLobType**`
*   `**org.springframework.orm.hibernate3.AbstractSessionFactoryBean**`
*   `**org.springframework.orm.hibernate3.annotation.AnnotationSessionFactoryBean**`
*   `**org.springframework.orm.hibernate3.support.BlobByteArrayType**`
*   `**org.springframework.orm.hibernate3.support.BlobSerializableType**`
*   `**org.springframework.orm.hibernate3.support.BlobStringType**`
*   `**org.springframework.orm.hibernate3.support.ClobStringType**`
*   `**org.springframework.orm.hibernate3.FilterDefinitionFactoryBean**`
*   `**org.springframework.orm.hibernate3.HibernateAccessor**`
*   `**org.springframework.orm.hibernate3.support.HibernateDaoSupport**`
*   `**org.springframework.orm.hibernate3.HibernateExceptionTranslator**`
*   `**org.springframework.orm.jpa.vendor.HibernateJpaSessionFactoryBean**`
*   `**org.springframework.orm.hibernate3.HibernateTemplate**`
*   `**org.springframework.orm.hibernate3.HibernateTransactionManager**`
*   `**org.springframework.orm.hibernate3.support.IdTransferringMergeEventListener**`
*   `**org.springframework.orm.hibernate3.LocalDataSourceConnectionProvider**`
*   `**org.springframework.orm.hibernate3.LocalJtaDataSourceConnectionProvider**`
*   `**org.springframework.orm.hibernate3.LocalRegionFactoryProxy**`
*   `**org.springframework.orm.hibernate3.LocalSessionFactoryBean**`
*   `**org.springframework.orm.hibernate3.LocalTransactionManagerLookup**`
*   `**org.springframework.orm.hibernate3.support.OpenSessionInterceptor**`
*   `**org.springframework.orm.hibernate3.support.OpenSessionInViewFilter**`
*   `**org.springframework.orm.hibernate3.support.OpenSessionInViewInterceptor**`
*   `**org.springframework.orm.hibernate3.support.ScopedBeanInterceptor**`
*   `**org.springframework.orm.hibernate3.SessionFactoryUtils**`
*   `**org.springframework.orm.hibernate3.SessionHolder**`
*   `**org.springframework.orm.hibernate3.SpringSessionContext**`
*   `**org.springframework.orm.hibernate3.SpringTransactionFactory**`
*   `**org.springframework.orm.hibernate3.TransactionAwareDataSourceConnectionProvider**`
*   `**org.springframework.orm.hibernate3.TypeDefinitionBean**`

有几个职业被弃用，取而代之的是 [FreeMarker](/web/20220524010452/https://www.baeldung.com/freemarker-in-spring-mvc-tutorial) :

*   `**org.springframework.web.servlet.view.velocity.VelocityConfigurer**`
*   `**org.springframework.ui.velocity.VelocityEngineFactory**`
*   `**org.springframework.ui.velocity.VelocityEngineFactoryBean**`
*   `**org.springframework.ui.velocity.VelocityEngineUtils**`
*   `**org.springframework.web.servlet.view.velocity.VelocityLayoutView**`
*   `**org.springframework.web.servlet.view.velocity.VelocityLayoutViewResolver**`
*   `**org.springframework.web.servlet.view.velocity.VelocityToolboxView**`
*   `**org.springframework.web.servlet.view.velocity.VelocityView**`
*   `**org.springframework.web.servlet.view.velocity.VelocityViewResolver**`

这些类在 Spring Framework 5.1 中被删除了，我们应该使用其他传输方式来代替:

*   `**org.springframework.web.socket.sockjs.transport.handler.JsonpPollingTransportHandler**`
*   `**org.springframework.web.socket.sockjs.transport.handler.JsonpReceivingTransportHandler**`

最后，还有几个类没有适当的替换:

*   `**org.springframework.core.ControlFlowFactory**`
*   `**org.springframework.util.WeakReferenceMonitor**`

和往常一样， [Spring 4.3.x JavaDoc](https://web.archive.org/web/20220524010452/https://docs.spring.io/spring-framework/docs/4.3.x/javadoc-api/deprecated-list.html) 包含了完整的列表。

### 2.5.Spring 5.0.x

*   `**org.springframework.web.reactive.support.AbstractAnnotationConfigDispatcherHandlerInitializer**` `–`弃用`AbstractReactiveWebInitializer`
*   `**org.springframework.web.util.AbstractUriTemplateHandler**` `–`
*   `**org.springframework.web.socket.config.annotation.AbstractWebSocketMessageBrokerConfigurer **` `–`已被弃用，取而代之的是简单地使用`WebSocketMessageBrokerConfigurer`，它有默认的方法，Java 8 基线使之成为可能
*   `**org.springframework.web.client.AsyncRestTemplate **``–`
*   `**org.springframework.web.context.request.async.CallableProcessingInterceptorAdapter **` `–`已弃用，因为`CallableProcessingInterceptor`有默认方法
*   `**org.springframework.messaging.support.ChannelInterceptorAdapter **` `–`已弃用，因为`ChannelInterceptor`有默认方法(通过 Java 8 基线成为可能),并且可以直接实现，不需要这个无操作适配器
*   `**org.springframework.util.comparator.CompoundComparator**` `–`弃用标准 JDK 8 `Comparator.thenComparing(Comparator)`
*   `**org.springframework.web.util.DefaultUriTemplateHandler **``–``DefaultUriBuilderFactory`；我们应该注意到`DefaultUriBuilderFactory`对于`parsePath`属性有不同的默认值(从`false`更改为`true`
*   `**org.springframework.web.context.request.async.DeferredResultProcessingInterceptorAdapter **` `–`因为`DeferredResultProcessingInterceptor`已经默认了方法
*   `**org.springframework.util.comparator.InvertibleComparator **` `–`弃用标准 JDK 8 `Comparator.reversed()`
*   `**org.springframework.http.client.Netty4ClientHttpRequestFactory **` `–`弃用`ReactorClientHttpConnector`
*   `**org.apache.commons.logging.impl.SimpleLog **` `–`移至`spring-jcl`(有效等同于`NoOpLog`)
*   `**org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter **` `–` `WebMvcConfigurer`有默认方法(通过 Java 8 基线成为可能),可以直接实现，不需要这个适配器
*   `**org.springframework.beans.factory.config.YamlProcessor.StrictMapAppenderConstructor **` `–`被 SnakeYAML 自己的复制键处理取代

我们有两个类被弃用，取而代之的是`AbstractReactiveWebInitializer`:

*   `**org.springframework.web.reactive.support.AbstractDispatcherHandlerInitializer**`
*   `**org.springframework.web.reactive.support.AbstractServletHttpHandlerAdapterInitializer**`

并且，以下类没有替换:

*   `**org.springframework.http.client.support.AsyncHttpAccessor**`
*   `**org.springframework.http.client.HttpComponentsAsyncClientHttpRequestFactory**`
*   `**org.springframework.http.client.InterceptingAsyncClientHttpRequestFactory**`
*   `**org.springframework.http.client.support.InterceptingAsyncHttpAccessor**`
*   `**org.springframework.mock.http.client.MockAsyncClientHttpRequest**`

完整的列表可以在 [Spring 5.0.x JavaDoc](https://web.archive.org/web/20220524010452/https://docs.spring.io/spring-framework/docs/5.0.x/javadoc-api/deprecated-list.html) 中找到。

### 2.6.Spring 5.1.x

*   `**org.springframework.http.client.support.BasicAuthorizationInterceptor **` `–`已被弃用，取而代之的是`BasicAuthenticationInterceptor`，它重用了`HttpHeaders.setBasicAuth(java.lang.String, java.lang.String)`，现在共享其默认字符集 ISO-8859-1，而不是像以前那样使用 UTF-8
*   `**org.springframework.jdbc.core.BatchUpdateUtils **``–``JdbcTemplate`不再使用
*   `**org.springframework.web.reactive.function.client.ExchangeFilterFunctions.Credentials **` `–`我们应该在构建请求时使用`HttpHeaders.setBasicAuth(String, String)`方法
*   `**org.springframework.web.filter.reactive.ForwardedHeaderFilter **` `–`这个过滤器被弃用，取而代之的是使用`ForwardedHeaderTransformer`，它可以被声明为一个名为`forwardedHeaderTransformer`的 bean，或者在`WebHttpHandlerBuilder`中显式注册
*   `**org.springframework.jdbc.core.namedparam.NamedParameterBatchUpdateUtils **``–``NamedParameterJdbcTemplate`不再使用
*   `**org.springframework.core.io.PathResource **``–`
*   `**org.springframework.beans.factory.annotation.RequiredAnnotationBeanPostProcessor **` `–`我们应该使用构造函数注入进行所需的设置(或者自定义`InitializingBean`实现)
*   `**org.springframework.remoting.caucho.SimpleHessianServiceExporter **``–`
*   `**org.springframework.remoting.httpinvoker.SimpleHttpInvokerServiceExporter**` `–`
*   `**org.springframework.remoting.support.SimpleHttpServerFactoryBean **` `–`嵌入式雄猫/突堤/回流
*   `**org.springframework.remoting.jaxws.SimpleHttpServerJaxWsServiceExporter **``–`

这些都被弃用，取而代之的是`EncodedResourceResolver`:

*   `**org.springframework.web.reactive.resource.GzipResourceResolver**`
*   `**org.springframework.web.servlet.resource.GzipResourceResolver**`

有几个类被弃用，取而代之的是 Java EE 7 的`DefaultManagedTaskScheduler`:

*   `**org.springframework.scheduling.commonj.DelegatingTimerListener**`
*   `**org.springframework.scheduling.commonj.ScheduledTimerListener**`
*   `**org.springframework.scheduling.commonj.TimerManagerAccessor**`
*   `**org.springframework.scheduling.commonj.TimerManagerFactoryBean**`
*   `**org.springframework.scheduling.commonj.TimerManagerTaskScheduler**`

并且，有一些被弃用，而支持 Java EE 7 的`DefaultManagedTaskExecutor`:

*   `**org.springframework.scheduling.commonj.DelegatingWork**`
*   `**org.springframework.scheduling.commonj.WorkManagerTaskExecutor**`

最后，有一个类被弃用，没有替代品:

*   `**org.apache.commons.logging.LogFactoryService**`

更多详情请见官方 [Spring 5.1.x JavaDoc 关于弃用类](https://web.archive.org/web/20220524010452/https://docs.spring.io/spring-framework/docs/5.1.x/javadoc-api/deprecated-list.html)。

## 3.Spring Boot 的废弃类

现在，让我们来看看 Spring Boot 1.4 版本中不赞成使用的类。

这里我们应该注意到，对于 Spring Boot 1.4 和 1.5，**大多数替换类保留了它们原来的名字，但是已经被移动到不同的包中**。因此，在接下来的两个小节中，我们对不推荐使用的类和替换类都使用完全限定的类名。

### 3.1.Spring Boot 1.4.x

*   `**org.springframework.boot.actuate.system.ApplicationPidFileWriter **` `–`弃用`org.springframework.boot.system.ApplicationPidFileWriter`
*   `**org.springframework.boot.yaml.ArrayDocumentMatcher **` `–`已弃用，支持基于精确`String`的匹配
*   `**org.springframework.boot.test.ConfigFileApplicationContextInitializer **``–`
*   `**org.springframework.boot.yaml.DefaultProfileDocumentMatcher **` `–`它不再被使用
*   `**org.springframework.boot.context.embedded.DelegatingFilterProxyRegistrationBean **``–`
*   `**org.springframework.boot.actuate.system.EmbeddedServerPortFileWriter **``–`
*   `**org.springframework.boot.test.EnvironmentTestUtils **``–`
*   `**org.springframework.boot.context.embedded.ErrorPage **``–`
*   `**org.springframework.boot.context.web.ErrorPageFilter **``–`
*   `**org.springframework.boot.context.embedded.FilterRegistrationBean **``–`
*   `**org.springframework.boot.test.IntegrationTestPropertiesListener **``–``@IntegrationTest`不再使用
*   `**org.springframework.boot.context.embedded.MultipartConfigFactory **``–`
*   `**org.springframework.boot.context.web.OrderedCharacterEncodingFilter **``–`
*   `**org.springframework.boot.context.web.OrderedHiddenHttpMethodFilter **``–`
*   `**org.springframework.boot.context.web.OrderedHttpPutFormContentFilter **``–`
*   `**org.springframework.boot.context.web.OrderedRequestContextFilter **``–`
*   `**org.springframework.boot.test.OutputCapture **``–`
*   **`org.springframework.boot.context.web.ServerPortInfoApplicationContextInitializer`**`–`
*   `**org.springframework.boot.context.web.ServletContextApplicationContextInitializer**` `–`
*   `**org.springframework.boot.context.embedded.ServletListenerRegistrationBean **``–`
*   `**org.springframework.boot.context.embedded.ServletRegistrationBean **``–`
*   `**org.springframework.boot.test.SpringApplicationContextLoader **` `–`弃用赞成`@SpringBootTest`；如有必要，我们也可以使用`org.springframework.boot.test.context.SpringBootContextLoader`
*   `**org.springframework.boot.test.SpringBootMockServletContext **``–`
*   `**org.springframework.boot.context.web.SpringBootServletInitializer **``–`
*   `**org.springframework.boot.test.TestRestTemplate **``–`

由于在 Spring Framework 4.3 中不支持 Velocity，因此在 Spring Boot 也不支持以下类:

*   `**org.springframework.boot.web.servlet.view.velocity.EmbeddedVelocityViewResolver**`
*   `**org.springframework.boot.autoconfigure.velocity.VelocityAutoConfiguration**`
*   `**org.springframework.boot.autoconfigure.velocity.VelocityAutoConfiguration.VelocityConfiguration**`
*   `**org.springframework.boot.autoconfigure.velocity.VelocityAutoConfiguration.VelocityNonWebConfiguration**`
*   `**org.springframework.boot.autoconfigure.velocity.VelocityAutoConfiguration.VelocityWebConfiguration**`
*   `**org.springframework.boot.autoconfigure.velocity.VelocityProperties**`
*   `**org.springframework.boot.autoconfigure.velocity.VelocityTemplateAvailabilityProvider**`

Spring Boot 1.4.x JavaDoc 有完整的列表。

### 3.2.Spring Boot 1.5.x

*   `**org.springframework.boot.context.event.ApplicationStartedEvent **` `–`弃用`org.springframework.boot.context.event.ApplicationStartingEvent`
*   `**org.springframework.boot.autoconfigure.EnableAutoConfigurationImportSelector **` `–`弃用`org.springframework.boot.autoconfigure.AutoConfigurationImportSelector`
*   `**org.springframework.boot.actuate.cache.GuavaCacheStatisticsProvider **` `–`在弹簧框架 5 中移除番石榴支架后
*   `**org.springframework.boot.loader.tools.Layouts.Module **` `–`弃用风俗`LayoutFactory`
*   `**org.springframework.boot.autoconfigure.MessageSourceAutoConfiguration **` `–`弃用`org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration`
*   `**org.springframework.boot.autoconfigure.PropertyPlaceholderAutoConfiguration **` `–`弃用`org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration`
*   `**org.springframework.boot.actuate.autoconfigure.ShellProperties **` `–`已弃用，因为崩溃未被主动维护

不推荐使用这两个类，因为没有主动维护崩溃:

*   `**org.springframework.boot.actuate.autoconfigure.CrshAutoConfiguration**`
*   `**org.springframework.boot.actuate.autoconfigure.CrshAutoConfiguration.AuthenticationManagerAdapterConfiguration**`

还有几个班没有替补:

*   `**org.springframework.boot.autoconfigure.cache.CacheProperties.Hazelcast**`
*   `**org.springframework.boot.autoconfigure.jdbc.metadata.CommonsDbcpDataSourcePoolMetadata**`
*   `**org.springframework.boot.autoconfigure.mustache.MustacheCompilerFactoryBean**`

要查看被否决内容的完整列表，我们可以参考 Spring Boot 官方网站。

### 3.3.Spring Boot 2.0.x

*   `**org.springframework.boot.test.util.EnvironmentTestUtils **` `–`弃用`TestPropertyValues`
*   `**org.springframework.boot.actuate.metrics.web.reactive.server.RouterFunctionMetrics **` `–`已弃用，支持自动配置的`MetricsWebFilter`

有一门课没有替代品:

*   `**org.springframework.boot.actuate.autoconfigure.couchbase.CouchbaseHealthIndicatorProperties**`

请查看 Spring Boot 2.0.x 的[弃用列表了解更多详情。](https://web.archive.org/web/20220524010452/https://docs.spring.io/spring-boot/docs/2.0.x/api/deprecated-list.html)

### 3.4.Spring Boot 2.1.x

*   `**org.springframework.boot.actuate.health.CompositeHealthIndicatorFactory **` `–`弃用`CompositeHealthIndicator.CompositeHealthIndicator(HealthAggregator, HealthIndicatorRegistry)`
*   `**org.springframework.boot.actuate.health.CompositeReactiveHealthIndicatorFactory **` `–`弃用`CompositeReactiveHealthIndicator.CompositeReactiveHealthIndicator(HealthAggregator, ReactiveHealthIndicatorRegistry)`

最后，我们可以参考 Spring Boot 2.1.x 中的[弃用类和接口的完整列表。](https://web.archive.org/web/20220524010452/https://docs.spring.io/spring-boot/docs/2.1.x/api/deprecated-list.html)

## 4。结论

在本教程中，我们探讨了 Spring 版和 Spring since 版中不推荐使用的类，以及它们相应的替换。