# Spring REST API 的指标

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-rest-api-metrics>

## 1。概述

在本教程中，我们将把**基本指标集成到 Spring REST API** 中。

我们将首先使用简单的 Servlet 过滤器构建度量功能，然后使用 Spring Boot 执行器模块。

## 2。`web.xml`

让我们首先在应用程序的`web.xml`中注册一个过滤器——“`MetricFilter`”:

```java
<filter>
    <filter-name>metricFilter</filter-name>
    <filter-class>org.baeldung.metrics.filter.MetricFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>metricFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

请注意我们是如何映射过滤器以覆盖所有进入的请求–`“/*”`–这当然是完全可配置的。

## 3。Servlet 过滤器

现在，让我们创建自定义过滤器:

```java
public class MetricFilter implements Filter {

    private MetricService metricService;

    @Override
    public void init(FilterConfig config) throws ServletException {
        metricService = (MetricService) WebApplicationContextUtils
         .getRequiredWebApplicationContext(config.getServletContext())
         .getBean("metricService");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
      throws java.io.IOException, ServletException {
        HttpServletRequest httpRequest = ((HttpServletRequest) request);
        String req = httpRequest.getMethod() + " " + httpRequest.getRequestURI();

        chain.doFilter(request, response);

        int status = ((HttpServletResponse) response).getStatus();
        metricService.increaseCount(req, status);
    }
}
```

因为过滤器不是标准的 bean，所以我们不打算注入`metricService`，而是通过`ServletContext`手动检索它。

还要注意，我们在这里通过调用`doFilter` API 继续执行过滤器链。

## 4。度量-状态代码计数

接下来，让我们来看看简单的`InMemoryMetricService`:

```java
@Service
public class MetricService {

    private Map<Integer, Integer> statusMetric;

    public MetricService() {
        statusMetric = new ConcurrentHashMap<>();
    }

    public void increaseCount(String request, int status) {
        Integer statusCount = statusMetric.get(status);
        if (statusCount == null) {
            statusMetric.put(status, 1);
        } else {
            statusMetric.put(status, statusCount + 1);
        }
    }

    public Map getStatusMetric() {
        return statusMetric;
    }
}
```

我们使用内存中的`ConcurrentMap`来保存每种 HTTP 状态代码的计数。

现在，为了显示这个基本指标，我们将把它映射到一个`Controller`方法:

```java
@GetMapping(value = "/status-metric")
@ResponseBody
public Map getStatusMetric() {
    return metricService.getStatusMetric();
}
```

下面是一个回答示例:

```java
{  
    "404":1,
    "200":6,
    "409":1
}
```

## 5。指标-按请求的状态代码

接下来—**让我们按请求记录计数指标**:

```java
@Service
public class MetricService {

    private Map<String, Map<Integer, Integer>> metricMap;

    public void increaseCount(String request, int status) {
        Map<Integer, Integer> statusMap = metricMap.get(request);
        if (statusMap == null) {
            statusMap = new ConcurrentHashMap<>();
        }

        Integer count = statusMap.get(status);
        if (count == null) {
            count = 1;
        } else {
            count++;
        }
        statusMap.put(status, count);
        metricMap.put(request, statusMap);
    }

    public Map getFullMetric() {
        return metricMap;
    }
}
```

我们将通过 API 显示指标结果:

```java
@GetMapping(value = "/metric")
@ResponseBody
public Map getMetric() {
    return metricService.getFullMetric();
}
```

这些指标如下所示:

```java
{
    "GET /users":
    {
        "200":6,
        "409":1
    },
    "GET /users/1":
    {
        "404":1
    }
}
```

根据上面的示例，API 具有以下活动:

*   “7”请求“获取`/users`
*   其中“6 个”导致“200”状态代码响应，只有一个导致“409”

## 6。度量–时间序列数据

总计数在应用程序中有些用处，但是如果系统已经运行了相当长的时间—**,就很难判断这些指标的实际意义了**。

为了使数据有意义并易于解释，你需要当时的背景。

现在，让我们构建一个简单的基于时间的指标；我们将记录每分钟的状态代码计数，如下所示:

```java
@Service
public class MetricService {

    private static final SimpleDateFormat DATE_FORMAT = 
      new SimpleDateFormat("yyyy-MM-dd HH:mm");
    private Map<String, Map<Integer, Integer>> timeMap;

    public void increaseCount(String request, int status) {
        String time = DATE_FORMAT.format(new Date());
        Map<Integer, Integer> statusMap = timeMap.get(time);
        if (statusMap == null) {
            statusMap = new ConcurrentHashMap<>();
        }

        Integer count = statusMap.get(status);
        if (count == null) {
            count = 1;
        } else {
            count++;
        }
        statusMap.put(status, count);
        timeMap.put(time, statusMap);
    }
}
```

而`getGraphData()`:

```java
public Object[][] getGraphData() {
    int colCount = statusMetric.keySet().size() + 1;
    Set<Integer> allStatus = statusMetric.keySet();
    int rowCount = timeMap.keySet().size() + 1;

    Object[][] result = new Object[rowCount][colCount];
    result[0][0] = "Time";

    int j = 1;
    for (int status : allStatus) {
        result[0][j] = status;
        j++;
    }
    int i = 1;
    Map<Integer, Integer> tempMap;
    for (Entry<String, Map<Integer, Integer>> entry : timeMap.entrySet()) {
        result[i][0] = entry.getKey();
        tempMap = entry.getValue();
        for (j = 1; j < colCount; j++) {
            result[i][j] = tempMap.get(result[0][j]);
            if (result[i][j] == null) {
                result[i][j] = 0;
            }
        }
        i++;
    } 
```

```java
    for (int k = 1; k < result[0].length; k++) {
        result[0][k] = result[0][k].toString();
    }
```

```java
 return result; 
}
```

我们现在将它映射到 API:

```java
@GetMapping(value = "/metric-graph-data")
@ResponseBody
public Object[][] getMetricData() {
    return metricService.getGraphData();
}
```

最后，我们将使用谷歌图表来呈现它:

```java
<html>
<head>
<title>Metric Graph</title>
<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.11.2/jquery.min.js"></script>
<script type="text/javascript" src="https://www.google.com/jsapi"></script>
<script type="text/javascript">
google.load("visualization", "1", {packages : [ "corechart" ]});

function drawChart() {
$.get("/metric-graph-data",function(mydata) {
    var data = google.visualization.arrayToDataTable(mydata);
    var options = {title : 'Website Metric',
                   hAxis : {title : 'Time',titleTextStyle : {color : '#333'}},
                   vAxis : {minValue : 0}};

    var chart = new google.visualization.AreaChart(document.getElementById('chart_div'));
    chart.draw(data, options);

});

}
</script>
</head>
<body onload="drawChart()">
    <div id="chart_div" style="width: 900px; height: 500px;"></div>
</body>
</html>
```

## 7。使用 Spring Boot 1.x 致动器

在接下来的几节中，我们将结合 Spring Boot 的执行器功能来展示我们的指标。

首先，我们需要将致动器依赖性添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 7.1。`MetricFilter`

接下来——我们可以把`MetricFilter`—变成一个真正的弹簧豆:

```java
@Component
public class MetricFilter implements Filter {

    @Autowired
    private MetricService metricService;

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
      throws java.io.IOException, ServletException {
        chain.doFilter(request, response);

        int status = ((HttpServletResponse) response).getStatus();
        metricService.increaseCount(status);
    }
}
```

当然，这是一个微小的简化——但是为了摆脱以前手工连接的依赖关系，这是值得做的。

### 7.2。使用`CounterService`

现在让我们使用`CounterService`来统计每个状态代码的出现次数:

```java
@Service
public class MetricService {

    @Autowired
    private CounterService counter;

    private List<String> statusList;

    public void increaseCount(int status) {
        counter.increment("status." + status);
        if (!statusList.contains("counter.status." + status)) {
            statusList.add("counter.status." + status);
        }
    }
}
```

### 7.3。使用`MetricRepository` 导出指标

接下来，我们需要使用`MetricRepository`导出指标:

```java
@Service
public class MetricService {

    @Autowired
    private MetricRepository repo;

    private List<List<Integer>> statusMetric;
    private List<String> statusList;

    @Scheduled(fixedDelay = 60000)
    private void exportMetrics() {
        Metric<?> metric;
        List<Integer> statusCount = new ArrayList<>();
        for (String status : statusList) {
            metric = repo.findOne(status);
            if (metric != null) {
                statusCount.add(metric.getValue().intValue());
                repo.reset(status);
            } else {
                statusCount.add(0);
            }
        }
        statusMetric.add(statusCount);
    }
}
```

请注意，我们正在存储每分钟的**状态代码的计数。**

### 7.4。Spring Boot `PublicMetrics`

我们还可以使用 Spring Boot `PublicMetrics`来导出指标，而不是使用我们自己的过滤器，如下所示:

首先，我们的预定任务是**导出每分钟指标**:

```java
@Autowired
private MetricReaderPublicMetrics publicMetrics;

private List<List<Integer>> statusMetricsByMinute;
private List<String> statusList;
private static final SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm");

@Scheduled(fixedDelay = 60000)
private void exportMetrics() {
    List<Integer> lastMinuteStatuses = initializeStatuses(statusList.size());
    for (Metric<?> counterMetric : publicMetrics.metrics()) {
        updateMetrics(counterMetric, lastMinuteStatuses);
    }
    statusMetricsByMinute.add(lastMinuteStatuses);
}
```

当然，我们需要初始化 HTTP 状态代码列表:

```java
private List<Integer> initializeStatuses(int size) {
    List<Integer> counterList = new ArrayList<>();
    for (int i = 0; i < size; i++) {
        counterList.add(0);
    }
    return counterList;
}
```

然后，我们将使用**状态代码计数**实际更新指标:

```java
private void updateMetrics(Metric<?> counterMetric, List<Integer> statusCount) {

    if (counterMetric.getName().contains("counter.status.")) {
        String status = counterMetric.getName().substring(15, 18); // example 404, 200
        appendStatusIfNotExist(status, statusCount);
        int index = statusList.indexOf(status);
        int oldCount = statusCount.get(index) == null ? 0 : statusCount.get(index);
        statusCount.set(index, counterMetric.getValue().intValue() + oldCount);
    }
}

private void appendStatusIfNotExist(String status, List<Integer> statusCount) {
    if (!statusList.contains(status)) {
        statusList.add(status);
        statusCount.add(0);
    }
}
```

请注意:

*   `PublicMetics`状态计数器名称以`counter.status`开头，例如`counter.status.200.root`
*   我们在列表中记录每分钟的状态计数`statusMetricsByMinute`

**我们可以导出收集到的数据，并将其绘制成图表**，如下所示:

```java
public Object[][] getGraphData() {
    Date current = new Date();
    int colCount = statusList.size() + 1;
    int rowCount = statusMetricsByMinute.size() + 1;
    Object[][] result = new Object[rowCount][colCount];
    result[0][0] = "Time";
    int j = 1;

    for (String status : statusList) {
        result[0][j] = status;
        j++;
    }

    for (int i = 1; i < rowCount; i++) {
        result[i][0] = dateFormat.format(
          new Date(current.getTime() - (60000L * (rowCount - i))));
    }

    List<Integer> minuteOfStatuses;
    List<Integer> last = new ArrayList<Integer>();

    for (int i = 1; i < rowCount; i++) {
        minuteOfStatuses = statusMetricsByMinute.get(i - 1);
        for (j = 1; j <= minuteOfStatuses.size(); j++) {
            result[i][j] = 
              minuteOfStatuses.get(j - 1) - (last.size() >= j ? last.get(j - 1) : 0);
        }
        while (j < colCount) {
            result[i][j] = 0;
            j++;
        }
        last = minuteOfStatuses;
    }
    return result;
}
```

### 7.5。使用指标绘制图表

最后，让我们通过一个二维数组来表示这些指标，这样我们就可以用图表来表示它们:

```java
public Object[][] getGraphData() {
    Date current = new Date();
    int colCount = statusList.size() + 1;
    int rowCount = statusMetric.size() + 1;
    Object[][] result = new Object[rowCount][colCount];
    result[0][0] = "Time";

    int j = 1;
    for (String status : statusList) {
        result[0][j] = status;
        j++;
    }

    ArrayList<Integer> temp;
    for (int i = 1; i < rowCount; i++) {
        temp = statusMetric.get(i - 1);
        result[i][0] = dateFormat.format
          (new Date(current.getTime() - (60000L * (rowCount - i))));
        for (j = 1; j <= temp.size(); j++) {
            result[i][j] = temp.get(j - 1);
        }
        while (j < colCount) {
            result[i][j] = 0;
            j++;
        }
    }

    return result;
}
```

这是我们的控制器方法`getMetricData()`:

```java
@GetMapping(value = "/metric-graph-data")
@ResponseBody
public Object[][] getMetricData() {
    return metricService.getGraphData();
}
```

下面是一个回答示例:

```java
[
    ["Time","counter.status.302","counter.status.200","counter.status.304"],
    ["2015-03-26 19:59",3,12,7],
    ["2015-03-26 20:00",0,4,1]
]
```

## 8。使用 Spring Boot 2.x 致动器

在 Spring Boot 2 中，Spring Actuator 的 API 见证了一个重大的变化。 **Spring 自己的度量已经被替换为 [`Micrometer`](/web/20220625235338/https://www.baeldung.com/micrometer) 。**因此，让我们用`Micrometer`编写上面相同的指标示例。

### 8.1。将`CounterService`替换为`MeterRegistry`

由于我们的 Spring Boot 应用已经依赖于启动器，千分尺已经自动配置。我们可以注射`MeterRegistry`而不是`CounterService`。我们可以使用不同类型的`Meter`来获取指标。`Counter`是米中的一种:

```java
@Autowired
private MeterRegistry registry;

private List<String> statusList;

@Override
public void increaseCount(int status) {
    String counterName = "counter.status." + status;
    registry.counter(counterName).increment(1);
    if (!statusList.contains(counterName)) {
        statusList.add(counterName);
    }
}
```

### 8.2.查看自定义指标

由于我们的指标现在已经注册到 Micrometer，首先，让我们在应用程序配置中[启用它们。现在我们可以通过导航到`/actuator/metrics`处的致动器端点来查看它们:](/web/20220625235338/https://www.baeldung.com/spring-boot-actuator-enable-endpoints#3-enabling-specific-endpoints)

```java
{
  "names": [
    "application.ready.time",
    "application.started.time",
    "counter.status.200",
    "disk.free",
    "disk.total",
    .....
  ]
}
```

在这里，我们可以看到我们的`counter.status.200`指标列在标准执行器指标中。此外，我们还可以通过将 URI 中的选择器设置为`/actuator/metrics/counter.status.200`来获得该指标的最新值:

```java
{
  "name": "counter.status.200",
  "description": null,
  "baseUnit": null,
  "measurements": [
    {
      "statistic": "COUNT",
      "value": 2
    }
  ],
  "availableTags": []
}
```

### 8.3。使用`MeterRegistry` 导出计数

以微米为单位，我们可以使用`MeterRegistry:`导出`Counter`值

```java
@Scheduled(fixedDelay = 60000)
private void exportMetrics() {
    List<Integer> statusCount = new ArrayList<>();
    for (String status : statusList) {
        Search search = registry.find(status);
        Counter counter = search.counter();
         if (counter == null) {
             statusCount.add(0);
         } else {
             statusCount.add(counter != null ? ((int) counter.count()) : 0);
             registry.remove(counter);
         }
    }
    statusMetricsByMinute.add(statusCount);
}
```

### 8.3。发布指标使用`Meters`

现在我们也可以使用`MeterRegistry's Meters:`来发布指标

```java
@Scheduled(fixedDelay = 60000)
private void exportMetrics() {
    List<Integer> lastMinuteStatuses = initializeStatuses(statusList.size());

    for (Meter counterMetric : publicMetrics.getMeters()) {
        updateMetrics(counterMetric, lastMinuteStatuses);
    }
    statusMetricsByMinute.add(lastMinuteStatuses);
}

private void updateMetrics(Meter counterMetric, List<Integer> statusCount) {
    String metricName = counterMetric.getId().getName();
    if (metricName.contains("counter.status.")) {
        String status = metricName.substring(15, 18); // example 404, 200
        appendStatusIfNotExist(status, statusCount);
        int index = statusList.indexOf(status);
        int oldCount = statusCount.get(index) == null ? 0 : statusCount.get(index);
        statusCount.set(index, (int)((Counter) counterMetric).count() + oldCount);
    }
}
```

## 9。结论

在本文中，我们探索了一些简单的方法来将一些基本的度量功能构建到 Spring web 应用程序中。

注意，计数器**不是线程安全的**——所以如果不使用原子序数之类的东西，它们可能不精确。这是故意的，只是因为 delta 应该很小，100%的准确性不是目标——相反，及早发现趋势才是。

当然，在应用程序中记录 HTTP 指标有更成熟的方法，但这是一种简单、轻量级、超级有用的方法，不需要成熟工具的额外复杂性。

本文的完整实现可以在 GitHub 项目中找到。