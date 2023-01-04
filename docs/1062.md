# 罗马 RSS 快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rome-rss>

## 1。概述

RSS (Rich Site Summary 或 Really Simple Syndication)是一种 web 提要标准，为读者提供来自不同位置的聚合内容。用户可以在一个地方看到他最喜欢的博客、新闻网站等最近发布的内容。

应用程序还可以使用 RSS 通过 RSS 提要来读取、操作或发布信息。

本文概述了如何使用 Rome API 在 Java 中处理 RSS 提要。

## 2。Maven 依赖关系

我们需要将 Rome API 的依赖项添加到我们的项目中:

```
<dependency>			
    <groupId>rome</groupId>			
    <artifactId>rome</artifactId>			
    <version>1.0</version>
</dependency>
```

我们可以在 [Maven Central](https://web.archive.org/web/20220627075338/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22rome%22%20AND%20a%3A%22rome%22) 上找到最新版本。

## 3。创建新的 RSS 源

首先，让我们用罗马 API **创建一个新的 RSS 提要，它使用了`SyndFeed`接口**的默认实现`SyndFeedImpl`。这个接口能够处理所有的 RSS 风格，所以我们可以放心使用它:

```
SyndFeed feed = new SyndFeedImpl();
feed.setFeedType("rss_1.0");
feed.setTitle("Test title");
feed.setLink("http://www.somelink.com");
feed.setDescription("Basic description");
```

在这个代码片段中，我们创建了一个带有标准 RSS 字段(如标题、链接和描述)的 RSS 提要。 **`SyndFeed`提供了添加更多字段**的机会，包括作者、贡献者、版权、模块、发布日期、图像、外国标记和语言。

## 4。添加条目

由于我们已经创建了 RSS 提要，现在我们可以向它添加一个条目。在下面的例子中，我们**使用`SyndEntry`接口**的默认实现`SyndEntryImpl`来创建一个新条目:

```
SyndEntry entry = new SyndEntryImpl();
entry.setTitle("Entry title");        
entry.setLink("http://www.somelink.com/entry1");

feed.setEntries(Arrays.asList(entry));
```

## 5。添加描述

到目前为止，我们的条目相当空，让我们为它添加一个描述。我们可以通过**使用`SyndContent`接口**的默认实现`SyndContentImpl`来做到这一点:

```
SyndContent description = new SyndContentImpl();
description.setType("text/html");
description.setValue("First entry");

entry.setDescription(description);
```

使用`setType` 方法，我们已经指定描述的内容将是文本或 HTML。

## 6。添加类别

RSS 条目通常被分类，以简化寻找我们感兴趣的条目的任务。让我们看看如何使用`SyndCategory`接口的默认实现`SyndCategoryImpl`向条目**添加一个类别**

```
List<SyndCategory> categories = new ArrayList<>();
SyndCategory category = new SyndCategoryImpl();
category.setName("Sophisticated category");
categories.add(category);

entry.setCategories(categories);
```

## 7 .**。发布订阅源**

我们已经有了一个带有条目的 RSS 提要。现在我们想出版它。就本文而言，发布意味着将提要写入流:

```
Writer writer = new FileWriter("xyz.txt");
SyndFeedOutput syndFeedOutput = new SyndFeedOutput();
syndFeedOutput.output(feed, writer);
writer.close();
```

## 8。读取外部进纸

我们已经知道如何创建新的提要，但是有时我们只需要连接到一个现有的提要。

让我们看看如何在给定 URL 的情况下读取/加载提要:

```
URL feedSource = new URL("http://rssblog.whatisrss.com/feed/");
SyndFeedInput input = new SyndFeedInput();
SyndFeed feed = input.build(new XmlReader(feedSource));
```

## 9。结论

在本文中，我们展示了如何创建包含一些条目的 RSS 提要，如何发布提要，以及如何阅读外部提要。

和往常一样，你可以在 GitHub 上查看本文[中提供的例子。](https://web.archive.org/web/20220627075338/https://github.com/eugenp/tutorials/tree/master/libraries-4)