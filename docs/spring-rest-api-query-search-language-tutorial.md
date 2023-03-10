# 构建 REST 查询语言

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-rest-api-query-search-language-tutorial>

一个成熟的 REST API 可能需要大量的工作，以灵活的方式发布资源通常是一种平衡行为。

一方面，你希望**允许客户以多种灵活的方式搜索信息**。另一方面，你不想实现太多的操作。

用于 API 的搜索/查询语言是最有意义的——它允许一个单一的、干净的操作，同时仍然为强大的搜索开放 API。

![Query Basics - icon](img/170748f1f9aa34938952c58bc25ae992.png)

## REST 查询语言基础

*   ***[其余查询语言同 Spring 和 JPA 标准](/web/20220627151417/https://www.baeldung.com/rest-search-language-spring-jpa-criteria)***
**   ***[REST 查询语言同 Spring 数据 JPA 规范](/web/20220627151417/https://www.baeldung.com/rest-api-search-language-spring-data-specifications)*****   ***[REST 查询语言有 Spring 数据 JPA 和 QueryDSL](/web/20220627151417/https://www.baeldung.com/rest-api-search-language-spring-data-querydsl)*****