# Java SE/EE/ME 的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-se-ee-me>

## 1。概述

在这个简短的教程中，我们将比较三个不同的 Java 版本。我们将了解它们提供了哪些功能以及它们的典型使用案例。

## 2。Java 标准版

让我们从 Java 标准版开始，简称 Java SE。这个版本提供了 Java 语言的核心功能。

Java SE 为 Java 应用提供了必不可少的组件: [Java 虚拟机、Java 运行时环境、Java 开发工具包](/web/20221128044152/https://www.baeldung.com/jvm-vs-jre-vs-jdk)。截至撰写本文时，最新版本是 Java 18。

让我们描述一个 Java SE 应用程序的简单用例。我们可以使用 [OOP 概念](/web/20221128044152/https://www.baeldung.com/java-oop)实现业务逻辑，使用 [`java.net`包](/web/20221128044152/https://www.baeldung.com/java-9-http-client)发出 HTTP 请求，使用 [JDBC](/web/20221128044152/https://www.baeldung.com/java-jdbc) 连接数据库。我们甚至可以使用 Swing 或 AWT 显示用户界面。

## 3。Java 企业版

Java EE 基于标准版，提供了更多的 API。缩写代表 Java 企业版，但也可以叫雅加达 EE。[它们都指同一个东西](/web/20221128044152/https://www.baeldung.com/java-enterprise-evolution)。

新的 Java EE APIs 允许我们创建更大的、可伸缩的应用程序。

通常，Java EE 应用程序被部署到应用服务器上。**提供了许多与 web 相关的 APIs】来促进这一点: [WebSocket](/web/20221128044152/https://www.baeldung.com/java-websockets) 、 [JavaServer Pages](/web/20221128044152/https://www.baeldung.com/jsp) 、 [JAX-RS](/web/20221128044152/https://www.baeldung.com/jax-rs-spec-and-implementations#inclusion-in-java-ee) 等。企业特性还包括与 JSON 处理、安全、Java 消息服务、 [JavaMail](/web/20221128044152/https://www.baeldung.com/java-email) 等相关的 API。**

在 Java EE 应用程序中，我们可以使用标准 API 中的任何东西。最重要的是，我们可以使用更先进的技术。

现在让我们看一个 Java EE 的用例。例如，我们可以创建[servlet](/web/20221128044152/https://www.baeldung.com/intro-to-servlets)来处理来自用户的 HTTP 请求，并使用 [JavaServer Pages](/web/20221128044152/https://www.baeldung.com/jsp) 创建动态 UI。我们可以使用 JMS 生成和消费消息，[发送电子邮件](/web/20221128044152/https://www.baeldung.com/java-email)和[认证用户](/web/20221128044152/https://www.baeldung.com/java-ee-8-security)以确保我们的应用程序安全。此外，我们可以使用[依赖注入机制](/web/20221128044152/https://www.baeldung.com/java-ee-cdi)来使我们的代码更易于维护。

## 4。Java 微型版

Java Micro Edition 或 Java ME 为面向嵌入式和移动设备的应用程序提供 API。这些可以是手机、机顶盒、传感器、打印机等。

Java ME 包括一些 Java SE 功能，同时提供了针对这些设备的新 API。比如蓝牙、位置、传感器 API 等。

大多数时候，这些小型设备在 CPU 或内存方面都有资源限制。使用 Java ME 时我们必须考虑这些约束。

有时，目标设备甚至可能无法用于测试我们的代码。SDK 可以在这方面提供帮助，因为它提供了模拟器、应用程序分析和监控。

例如，一个简单的 Java ME 应用程序可以读取温度传感器的值，并在 HTTP 请求中将其与位置一起发送。

## 5。结论

在这篇短文中，我们了解了三个 Java 版本，并比较了它们各自提供的功能。

Java SE 可以用于简单的应用程序。是学习 Java 最好的起点。我们可以使用 Java EE 来创建更复杂、更健壮的应用程序。最后，如果我们想针对嵌入式和移动设备，我们可以使用 Java ME。