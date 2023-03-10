# 弹性搜索全文搜索快速入门

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/elasticsearch-full-text-search-rest-api>

## 1。概述

全文搜索对文档进行查询和语言搜索。它包括单个或多个单词或短语，并返回符合搜索条件的文档。

ElasticSearch 是一个基于 [Apache Lucene](https://web.archive.org/web/20221012183903/https://lucene.apache.org/core/) 的搜索引擎，Apache Lucene 是一个免费开源的信息检索软件库。它提供了一个分布式的全文搜索引擎，带有 HTTP web 接口和无模式的 JSON 文档。

本文研究了 ElasticSearch REST API，并演示了仅使用 HTTP 请求的基本操作。

## 2。设置

为了在您的机器上安装 ElasticSearch，请参考[官方安装指南](https://web.archive.org/web/20221012183903/https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html)。

RESTfull API 运行在端口 9200 上。让我们使用下面的 curl 命令来测试它是否正常运行:

```java
curl -XGET 'http://localhost:9200/'
```

如果您观察到以下响应，则该实例正在正常运行:

```java
{
  "name": "NaIlQWU",
  "cluster_name": "elasticsearch",
  "cluster_uuid": "enkBkWqqQrS0vp_NXmjQMQ",
  "version": {
    "number": "5.1.2",
    "build_hash": "c8c4c16",
    "build_date": "2017-01-11T20:18:39.146Z",
    "build_snapshot": false,
    "lucene_version": "6.3.0"
  },
  "tagline": "You Know, for Search"
}
```

## 3。索引文件

ElasticSearch 面向文档。它存储和索引文档。索引创建或更新文档。建立索引后，您可以搜索、排序和筛选整个文档，而不是列数据行。这是一种根本不同的思考数据的方式，也是 ElasticSearch 能够执行复杂的全文搜索的原因之一。

文档被表示为 JSON 对象。JSON 序列化得到大多数编程语言的支持，并且已经成为 NoSQL 运动使用的标准格式。它简单、简洁、易读。

我们将使用以下随机条目来执行全文搜索:

```java
{
  "title": "He went",
  "random_text": "He went such dare good fact. The small own seven saved man age."
}

{
  "title": "He oppose",
  "random_text": 
    "He oppose at thrown desire of no. \
      Announcing impression unaffected day his are unreserved indulgence."
}

{
  "title": "Repulsive questions",
  "random_text": "Repulsive questions contented him few extensive supported."
}

{
  "title": "Old education",
  "random_text": "Old education him departure any arranging one prevailed."
}
```

在我们能够索引一个文档之前，我们需要决定在哪里存储它。有可能有多个索引，而索引又包含多种类型。这些类型包含多个文档，每个文档有多个字段。

我们将使用以下方案存储我们的文档:

`text`:索引名称。
`article`:类型名称。
`id`:这个特殊例子的 ID 文本条目。

要添加文档，我们将运行以下命令:

```java
curl -XPUT 'localhost:9200/text/article/1?pretty'
  -H 'Content-Type: application/json' -d '
{
  "title": "He went",
  "random_text": 
    "He went such dare good fact. The small own seven saved man age."
}'
```

这里我们使用`id=1`，我们可以使用相同的命令和递增的 id 添加其他条目。

## 4。正在检索文档

添加完所有文档后，我们可以使用以下命令检查集群中有多少文档:

```java
curl -XGET 'http://localhost:9200/_count?pretty' -d '
{
  "query": {
    "match_all": {}
  }
}' 
```

此外，我们可以通过以下命令使用文档的 id 来获取文档:

```java
curl -XGET 'localhost:9200/text/article/1?pretty' 
```

而我们应该从弹性搜索得到以下答案:

```java
{
  "_index": "text",
  "_type": "article",
  "_id": "1",
  "_version": 1,
  "found": true,
  "_source": {
    "title": "He went",
    "random_text": 
      "He went such dare good fact. The small own seven saved man age."
  }
}
```

正如我们所看到的，这个答案对应于使用 id 1 添加的条目。

## 5。查询单据

好的，让我们使用以下命令执行全文搜索:

```java
curl -XGET 'localhost:9200/text/article/_search?pretty' 
  -H 'Content-Type: application/json' -d '
{
  "query": {
    "match": {
      "random_text": "him departure"
    }
  }
}'
```

我们得到以下结果:

```java
{
  "took": 32,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 1.4513469,
    "hits": [
      {
        "_index": "text",
        "_type": "article",
        "_id": "4",
        "_score": 1.4513469,
        "_source": {
          "title": "Old education",
          "random_text": "Old education him departure any arranging one prevailed."
        }
      },
      {
        "_index": "text",
        "_type": "article",
        "_id": "3",
        "_score": 0.28582606,
        "_source": {
          "title": "Repulsive questions",
          "random_text": "Repulsive questions contented him few extensive supported."
        }
      }
    ]
  }
}
```

正如我们所看到的，我们正在寻找`“him departure”`，我们得到了两个分数不同的结果。第一个结果是显而易见的，因为文本中包含已执行的搜索，正如我们可以看到的，我们有得分`1.4513469`。

检索第二个结果是因为目标文档包含单词“him”。

默认情况下，ElasticSearch 按照相关性分数对匹配结果进行排序，也就是说，按照每个文档与查询的匹配程度进行排序。请注意，第二个结果的分数相对于第一个结果较小，表明相关性较低。

## 6。模糊搜索

模糊匹配将“模糊”相似的两个单词视为同一个单词。首先，我们需要定义什么是模糊性。

Elasticsearch 支持最大编辑距离，由模糊度参数指定，为 2。“模糊度”参数可以设定为“自动”,这会产生以下最大编辑距离:

*   `0`对于一个或两个字符的字符串
*   `1`对于三个、四个或五个字符的字符串
*   `2`对于超过五个字符的字符串

您可能会发现编辑距离`2`会返回看起来不相关的结果。

最大模糊度为 1 时，您可能会获得更好的结果和更好的性能。距离指的是 Levenshtein 距离，它是用于测量两个序列之间差异的字符串度量。非正式地说，两个单词之间的 [Levenshtein 距离](https://web.archive.org/web/20221012183903/https://en.wikipedia.org/wiki/Levenshtein_distance)是单个字符编辑的最小数量。

好了，让我们用模糊性进行搜索:

```java
curl -XGET 'localhost:9200/text/article/_search?pretty' -H 'Content-Type: application/json' -d' 
{ 
  "query": 
  { 
    "match": 
    { 
      "random_text": 
      {
        "query": "him departure",
        "fuzziness": "2"
      }
    } 
  } 
}'
```

这是结果:

```java
{
  "took": 88,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 4,
    "max_score": 1.5834423,
    "hits": [
      {
        "_index": "text",
        "_type": "article",
        "_id": "4",
        "_score": 1.4513469,
        "_source": {
          "title": "Old education",
          "random_text": "Old education him departure any arranging one prevailed."
        }
      },
      {
        "_index": "text",
        "_type": "article",
        "_id": "2",
        "_score": 0.41093433,
        "_source": {
          "title": "He oppose",
          "random_text":
            "He oppose at thrown desire of no. 
              \ Announcing impression unaffected day his are unreserved indulgence."
        }
      },
      {
        "_index": "text",
        "_type": "article",
        "_id": "3",
        "_score": 0.2876821,
        "_source": {
          "title": "Repulsive questions",
          "random_text": "Repulsive questions contented him few extensive supported."
        }
      },
      {
        "_index": "text",
        "_type": "article",
        "_id": "1",
        "_score": 0.0,
        "_source": {
          "title": "He went",
          "random_text": "He went such dare good fact. The small own seven saved man age."
        }
      }
    ]
  }
}'
```

正如我们所看到的，模糊性给了我们更多的结果。

我们需要小心使用模糊性，因为它倾向于检索看起来不相关的结果。

## 7。结论

在这个快速教程中，我们关注了直接通过 REST API 索引文档和查询全文搜索的 Elasticsearch。

当然，当我们需要时，我们有可用于多种编程语言的 API——但是 API 仍然非常方便，并且是语言不可知的。