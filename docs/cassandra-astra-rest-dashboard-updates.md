# 用 Cassandra、Astra、REST 和 GraphQL 构建一个仪表板，记录状态更新

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/cassandra-astra-rest-dashboard-updates>

## 1。简介

在我们的[上一篇文章](/web/20221127034840/https://www.baeldung.com/cassandra-astra-stargate-dashboard)中，我们看到了使用 [DataStax Astra](/web/20221127034840/https://www.baeldung.com/datastax-signup) 构建一个仪表板来查看复仇者联盟的当前状态，一个由 [Apache Cassandra](https://web.archive.org/web/20221127034840/https://cassandra.apache.org/) 驱动的 DBaaS 使用 [Stargate](/web/20221127034840/https://www.baeldung.com/stargate) 来提供额外的 API 来使用它。

[![cassandra avengers status dashboard 1](img/e911e2ea7b9c99e88fb8d4d2052921af.png)](/web/20221127034840/https://www.baeldung.com/wp-content/uploads/2021/06/cassandra-avengers-status-dashboard-1.png)

用卡珊德拉和星际之门建造的复仇者联盟状态面板

在本文中，我们将扩展它来存储离散的事件，而不是汇总的摘要。这将允许我们在 UI 中查看这些事件。我们将允许用户点击一张卡片，并获得一个将我们带到这一点的事件表。与概要不同，这些事件将分别代表一个复仇者和一个离散的时间点。每次接收到一个新事件，它就会和所有其他事件一起被添加到表中。

我们之所以使用 Cassandra，是因为它提供了一种非常有效的方式来存储时间序列数据，我们写的时间比读的时间要多得多。这里的目标是一个可以频繁更新的系统——例如，每 30 秒更新一次——然后可以让用户很容易地看到最近记录的事件。

## 2。构建数据库模式

与我们在上一篇文章中使用的文档 API 不同，这将使用 [REST](/web/20221127034840/https://www.baeldung.com/datastax-docs-astra) 和[graph QL](/web/20221127034840/https://www.baeldung.com/datastax-docs-GraphQL)API 来构建。**这些工作在一个 Cassandra 表之上，这些 API 完全可以相互协作以及与 CQL API 协作。**

为了处理这些，我们需要已经为我们存储数据的表定义了一个模式。我们使用的表格是为特定的模式设计的——按照事件发生的时间顺序为给定的复仇者查找事件。

该模式将如下所示:

```java
CREATE TABLE events (
    avenger text,
    timestamp timestamp,
    latitude decimal,
    longitude decimal,
    status decimal,
    PRIMARY KEY (avenger, timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC);
```

数据看起来像这样:

| 复仇者 | 时间戳 | 纬度 | 经度 | 状态 |
| --- | --- | --- | --- | --- |
| 猎鹰 | 2021-05-16 09:00:30.000000+0000 |  40.715255 |  -73.975353 | 0.999954 |
| 用潜望镜侦察潜水艇的装置 |  2021-05-16 09:00:30.000000+0000 |  40.714602 |  -73.975238 |  0.99986 |
| 用潜望镜侦察潜水艇的装置 |  2021-05-16 09:01:00.000000+0000 |  40.713572 |  -73.975289 | 0.999804 |

这将我们的表定义为具有[多行分区](/web/20221127034840/https://www.baeldung.com/datastax-cassandra-tables)，分区键为“avenger”，聚类键为“timestamp”。Cassandra 使用分区键来确定数据存储在哪个节点上。聚集键用于确定数据在分区中的存储顺序。

通过指明“复仇者”是我们的分区键，它将确保同一复仇者的所有数据都保存在一起。通过指出“时间戳”是我们的聚集键，它将把数据以最有效的顺序存储在这个分区中，以便我们检索。鉴于我们对这些数据的核心查询是为单个复仇者选择每个事件，即我们的分区键，按事件的时间戳排序，即我们的聚类键，Cassandra 可以让我们非常高效地访问这些数据。

此外，应用程序的设计使用方式意味着我们在近乎连续的基础上编写事件数据。例如，我们可能每 30 秒从每个复仇者那里得到一个新事件。以这种方式构造我们的表可以非常有效地将新事件插入到正确分区的正确位置。

为了方便起见，我们用于预填充数据库的[脚本](https://web.archive.org/web/20221127034840/https://github.com/Baeldung/datastax-cassandra/blob/main/avengers-dashboard/data.sh)也将创建并填充这个模式。

## 3。使用 Astra、REST、&graph QL API构建客户端层

为了不同的目的，我们将同时使用 REST 和 GraphQL APIs 与 Astra 进行交互。REST API 将用于向表中插入新事件。GraphQL API 将用于再次检索它们。

为了最好地做到这一点，我们需要一个客户端层来执行与 Astra 的交互。对于另外两个 API，它们相当于我们在上一篇文章中构建的`DocumentClient`类。

### 3.1。休息客户端

首先，我们的 REST 客户端。**我们将使用它来插入新的完整记录**，因此只需要一个方法来插入数据:

```java
@Repository
public class RestClient {
  @Value("https://${ASTRA_DB_ID}-${ASTRA_DB_REGION}.apps.astra.datastax.com/api/rest/v2/keyspaces/${ASTRA_DB_KEYSPACE}")
  private String baseUrl;

  @Value("${ASTRA_DB_APPLICATION_TOKEN}")
  private String token;

  private RestTemplate restTemplate;

  public RestClient() {
    this.restTemplate = new RestTemplate();
    this.restTemplate.setRequestFactory(new HttpComponentsClientHttpRequestFactory());
  }

  public <T> void createRecord(String table, T record) {
    var uri = UriComponentsBuilder.fromHttpUrl(baseUrl)
      .pathSegment(table)
      .build()
      .toUri();
    var request = RequestEntity.post(uri)
      .header("X-Cassandra-Token", token)
      .body(record);
    restTemplate.exchange(request, Map.class);
  }
}
```

### 3.2。GraphQL 客户端

然后，我们的 GraphQL 客户端。**这一次我们进行了一次完整的 GraphQL 查询，并返回它获取的数据**:

```java
@Repository
public class GraphqlClient {
  @Value("https://${ASTRA_DB_ID}-${ASTRA_DB_REGION}.apps.astra.datastax.com/api/graphql/${ASTRA_DB_KEYSPACE}")
  private String baseUrl;

  @Value("${ASTRA_DB_APPLICATION_TOKEN}")
  private String token;

  private RestTemplate restTemplate;

  public GraphqlClient() {
    this.restTemplate = new RestTemplate();
    this.restTemplate.setRequestFactory(new HttpComponentsClientHttpRequestFactory());
  }

  public <T> T query(String query, Class<T> cls) {
    var request = RequestEntity.post(baseUrl)
      .header("X-Cassandra-Token", token)
      .body(Map.of("query", query));
    var response = restTemplate.exchange(request, cls);

    return response.getBody();
  }
}
```

和以前一样，我们的`baseUrl`和`token`字段是根据定义如何与 Astra 对话的属性配置的。这些客户端类都知道如何构建与数据库交互所需的完整 URL。我们可以使用它们发出正确的 HTTP 请求来执行所需的操作。

这就是与 Astra 交互所需要的一切，因为这些 API 只是通过 HTTP 交换 JSON 文档来工作。

## 4。记录单个事件

为了显示事件，我们需要能够记录它们。这将建立在我们之前更新`statuses`表的功能之上，并将额外的新记录插入到`events`表中。

### 4.1。插入事件

我们首先需要的是这个表中数据的表示。这将表示为一个 Java 记录:

```java
public record Event(String avenger, 
  String timestamp,
  Double latitude,
  Double longitude,
  Double status) {}
```

这与我们之前定义的模式直接相关。当我们实际进行 API 调用时，Jackson 会将其转换成 REST API 的正确 JSON。

接下来，我们需要我们的服务层实际记录这些。这将从外部获取适当的细节，用时间戳增加它们，并调用我们的 REST 客户机来创建新记录:

```java
@Service
public class EventsService {
  @Autowired
  private RestClient restClient;

  public void createEvent(String avenger, Double latitude, Double longitude, Double status) {
    var event = new Event(avenger, Instant.now().toString(), latitude, longitude, status);

    restClient.createRecord("events", event);
  }
}
```

### 4.2。更新 API

最后，我们需要一个控制器来接收事件。**这是对我们在上一篇文章中写的`UpdateController`的扩展，以连接新的`EventsService`，然后从我们的`update`方法**中调用它。

```java
@RestController
public class UpdateController {
  ......
  @Autowired
  private EventsService eventsService;

  @PostMapping("/update/{avenger}")
  public void update(@PathVariable String avenger, @RequestBody UpdateBody body) throws Exception {
    eventsService.createEvent(avenger, body.lat(), body.lng(), body.status());
    statusesService.updateStatus(avenger, lookupLocation(body.lat(), body.lng()), getStatus(body.status()));
  }
  ......
}
```

此时，调用我们的 API 来记录复仇者的状态既会更新 status 文档，又会在 events 表中插入一条新记录。这将允许我们记录发生的每一个更新事件。

这意味着，每当我们接到一个更新复仇者状态的电话时，我们都会向该表添加一条新记录。实际上，我们需要通过修剪或添加额外的分区来支持数据存储的规模，但这超出了本文的范围。【T2

## 5。通过 GraphQL API 向用户提供事件

一旦我们的表中有了事件，下一步就是让它们对用户可用。**我们将使用 GraphQL API 来实现这一点，为给定的复仇者一次检索一页事件，这些事件总是有序的，因此最近的事件排在最前面**。

使用 GraphQL，我们还能够只检索我们真正感兴趣的字段的子集，而不是全部。如果我们获取大量的记录，那么这有助于降低负载大小，从而提高性能。

### 5.1。检索事件

我们首先需要的是我们正在检索的数据的表示。这是表中存储的实际数据的子集。因此，我们需要一个不同的类来表示它:

```java
public record EventSummary(String timestamp,
  Double latitude,
  Double longitude,
  Double status) {}
```

我们还需要一个类来表示这些列表的 GraphQL 响应。这将包括事件摘要列表和用于光标到下一页的页面状态:

```java
public record Events(List<EventSummary> values, String pageState) {}
```

我们现在可以在事件服务中创建一个新方法来实际执行搜索。

```java
public class EventsService {
  ......
  @Autowired
  private GraphqlClient graphqlClient;

  public Events getEvents(String avenger, String offset) {
    var query = "query {" + 
      "  events(filter:{avenger:{eq:\"%s\"}}, orderBy:[timestamp_DESC], options:{pageSize:5, pageState:%s}) {" +
      "    pageState " +
      "    values {" +
      "     timestamp " +
      "     latitude " +
      "     longitude " +
      "     status" +
      "   }" +
      "  }" +
      "}";

    var fullQuery = String.format(query, avenger, offset == null ? "null" : "\"" + offset + "\"");

    return graphqlClient.query(fullQuery, EventsGraphqlResponse.class).data().events();
  }

  private static record EventsResponse(Events events) {}
  private static record EventsGraphqlResponse(EventsResponse data) {}
}
```

这里我们有两个内部类，它们纯粹是为了表示 GraphQL API 返回的 JSON 结构，直到我们感兴趣的部分——这些完全是 GraphQL API 的人工制品。

然后我们有了一个方法，为我们想要的细节构建一个 GraphQL 查询，通过`avenger`字段过滤并通过`timestamp`字段降序排序。在传递给我们的 GraphQL 客户端以获取实际数据之前，我们将使用实际的 Avenger ID 和页面状态。

### 5.2。在 UI 中显示事件

既然我们可以从数据库中获取事件，那么我们可以将它连接到我们的用户界面。

首先，我们将更新我们在上一篇文章中编写的`StatusesController`,以支持获取事件的 UI 端点:

```java
public class StatusesController {
  ......

  @Autowired
  private EventsService eventsService;

  @GetMapping("/avenger/{avenger}")
  public Object getAvengerStatus(@PathVariable String avenger, @RequestParam(required = false) String page) {
    var result = new ModelAndView("dashboard");
    result.addObject("avenger", avenger);
    result.addObject("statuses", statusesService.getStatuses());
    result.addObject("events", eventsService.getEvents(avenger, page));

    return result;
  }
} 
```

然后我们需要更新模板来呈现事件表。我们将向`dashboard.html`文件添加一个新表，只有在从控制器接收到的模型中存在`events`对象时，才会呈现该表:

```java
......
    <div th:if="${events}">
      <div class="row">
        <table class="table">
          <thead>
            <tr>
              <th scope="col">Timestamp</th>
              <th scope="col">Latitude</th>
              <th scope="col">Longitude</th>
              <th scope="col">Status</th>
            </tr>
          </thead>
          <tbody>
            <tr th:each="data, iterstat: ${events.values}">
              <th scope="row" th:text="${data.timestamp}">
                </td>
              <td th:text="${data.latitude}"></td>
              <td th:text="${data.longitude}"></td>
              <td th:text="${(data.status * 100) + '%'}"></td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="row" th:if="${events.pageState}">
        <div class="col position-relative">
          <a th:href="@{/avenger/{id}(id = ${avenger}, page = ${events.pageState})}"
            class="position-absolute top-50 start-50 translate-middle">Next
            Page</a>
        </div>
      </div>
    </div>
  </div>
......
```

这包括底部的一个链接，可以转到下一页，它通过我们的事件数据和我们正在查看的复仇者的 ID 来传递页面状态。

最后，我们需要更新状态卡，以允许我们链接到这个条目的 events 表。这只是每张卡片标题周围的一个超链接，呈现在`status.html:`中

```java
......
  <a th:href="@{/avenger/{id}(id = ${data.avenger})}">
    <h5 class="card-title" th:text="${data.name}"></h5>
  </a>
......
```

**我们现在可以启动应用程序，点击卡片查看导致此状态的最近事件:**

[![cassandra avengers status dashboard events](img/b6316b86d1ba586bf0a0cf6c7be89c84.png)](/web/20221127034840/https://www.baeldung.com/wp-content/uploads/2021/06/cassandra-avengers-status-dashboard-events.png)

复仇者联盟状态仪表板使用 GraphQL 扩展了状态更新

## 6。总结

**在这里，我们看到了 Astra REST 和 GraphQL APIs 如何用于处理基于行的数据，以及它们如何协同工作**。我们也开始看到 Cassandra 和这些 API 可以很好地用于大规模数据集。

本文中的所有代码都可以在 GitHub 上找到。