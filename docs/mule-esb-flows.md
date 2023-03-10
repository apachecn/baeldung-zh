# Mule 中不同类型的流程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mule-esb-flows>

## 1。简介

Mule 是一个基于 Java 的产品，它提供了企业服务总线(ESB)解决方案。我们可以使用 Eclipse 插件 [Anypoint Studio](https://web.archive.org/web/20220616162341/https://www.mulesoft.com/platform/studio) 开发 Mule 应用程序。

在简要介绍了 ESB 和流之后，**我们将讨论 Mule 中不同类型的流，以及我们在哪里使用每种类型的流。**

## 2。企业服务总线

ESB 是连接独立软件的中介层。这些应用通常使用不同的协议。因此，ESB 将负责数据的转换和路由。这允许创建分离的服务。因此，每个服务不需要担心另一个服务将如何使用它的输出。

ESB 的工作是确保数据格式正确。我们可以在之前的教程中找到更多关于 Mule ESB 的细节。

是 Mule 应用程序的构建模块。因此，正是这些组件负责转换和路由数据。我们从 Anypoint Studio 右侧的`Mule Palette`向 Mule 应用程序添加组件:

[![m1](img/bd2c7d98d5531e3f79f7099b5f31779d.png)](/web/20220616162341/https://www.baeldung.com/wp-content/uploads/2018/08/m1.png)

组件随后被分组到`flows`中。Mule 应用程序由一个或多个流组成。

## 3。什么是流量？

流程是 Mule 组件的连接集合。

它通常由一个入站端点组件(消息的来源)和一个出站端点组件组成。**因此，流负责消息可能经历的各种处理阶段。**

每个流可以有一个[处理策略](https://web.archive.org/web/20220616162341/https://docs.mulesoft.com/mule-user-guide/v/3.5/flow-processing-strategies)以及一个与之相关的[异常处理策略](https://web.archive.org/web/20220616162341/https://docs.mulesoft.com/mule-user-guide/v/3.5/catch-exception-strategy)。一个流也可以使用[流引用组件](https://web.archive.org/web/20220616162341/https://docs.mulesoft.com/mule-user-guide/v/3.9/flow-reference-component-reference)来引用另一个流。

Mule 中有三种不同类型的流程:

*   子流–继承父流的处理和异常处理策略的同步流
*   同步流——带有处理和异常处理策略的同步流
*   异步流——带有处理和异常处理策略的异步流

## 4。子流程

我们使用子流来对公共逻辑进行分组。**子流程同步处理**；也就是说，调用流的执行停止，直到子流完成。

具体来说，我们可以从 Mule 调色板添加一个子流:

[![m2](img/205d24bece7e1fd280b928e35ce5581c.png)](/web/20220616162341/https://www.baeldung.com/wp-content/uploads/2018/08/m2.png)

使用`flow reference component`调用子流程:

[![m3](img/dcdfa428f3e7e1fcae0f77062208e5d1.png)](/web/20220616162341/https://www.baeldung.com/wp-content/uploads/2018/08/m3.png)

另外，**子流继承了调用流的处理策略和异常处理策略。**我们可以从多个不同的流中调用一个子流。如果我们不希望继承这些策略，我们可以称之为同步流。

## 5。同步流

像子流一样，**同步流也是同步处理的。**这意味着当我们调用同步流时，它必须在父流恢复执行之前完成。

我们通过添加常规流将同步流添加到我们的 Mule 应用程序中；没有“同步流”组件。我们添加一个常规的`flow component`:

[![m4](img/7076bdd0e6abde28e273e1e348960e56.png)](/web/20220616162341/https://www.baeldung.com/wp-content/uploads/2018/08/m4.png)

为了调用同步流，我们再次使用了一个`flow reference component`:

[![m5](img/96aa8701d427906f6aef9b429b5438c9.png)](/web/20220616162341/https://www.baeldung.com/wp-content/uploads/2018/08/m5.png)

与子流不同，它不继承调用流的处理和异常处理策略。因此，调用流的处理和异常处理策略不会影响这种类型的流的行为。

由于这些原因，这种类型的流非常适合事务性处理，因为使用同步流处理的消息在单个线程上执行。

## 6。异步流

异步流与调用流并行执行；即**它们被异步处理。**

我们向 Mule 应用程序添加异步流的方式与添加同步流的方式相同。所以，让流异步的是，我们在异步范围内调用它。我们可以通过将 `flow reference component`包装在 [`async component`](https://web.archive.org/web/20220616162341/https://docs.mulesoft.com/mule-user-guide/v/3.6/async-scope-reference) 中来做到这一点:

[![m6](img/48222b92e83240bcc0500b105fbf66c0.png)](/web/20220616162341/https://www.baeldung.com/wp-content/uploads/2018/08/m6.png)

类似于同步流——它们不继承调用流的处理和异常处理策略。

此外，使用异步流处理的消息在多个线程上执行，这使得这种类型的流非常适合于发送电子邮件或执行 I/O 操作等耗时的任务。

## 7.结论

在这个简短的教程中，我们讨论了 Mule 中不同类型的流的特征。

欲了解更多信息，请访问 Mule 官方网站。