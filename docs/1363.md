# 使用 Apache Spark 的 Spring 云数据流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-data-flow-spark>

## 1.介绍

**Spring Cloud Data Flow 是一个构建数据集成和实时数据处理管道的工具包。**

在这种情况下，管道是使用 [Spring Cloud Stream](https://web.archive.org/web/20220625085016/https://cloud.spring.io/spring-cloud-stream/) 或 [Spring Cloud Task](https://web.archive.org/web/20220625085016/https://spring.io/projects/spring-cloud-task) 框架构建的 Spring Boot 应用程序。

在本教程中，我们将展示如何通过 [Apache Spark](/web/20220625085016/https://www.baeldung.com/apache-spark) 使用 Spring Cloud 数据流。

## 2.数据流本地服务器

首先，我们需要运行[数据流服务器](/web/20220625085016/https://www.baeldung.com/spring-cloud-data-flow-stream-processing)来部署我们的作业。

为了在本地运行数据流服务器，我们需要用[和`spring-cloud-starter-dataflow-server-local`依赖项](https://web.archive.org/web/20220625085016/https://search.maven.org/search?q=spring-cloud-starter-dataflow-server-local)创建一个新项目:

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-dataflow-server-local</artifactId>
    <version>1.7.4.RELEASE</version>
</dependency>
```

之后，我们需要用`@EnableDataFlowServer`注释服务器中的主类:

```
@EnableDataFlowServer
@SpringBootApplication
public class SpringDataFlowServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(
          SpringDataFlowServerApplication.class, args);
    }
}
```

一旦我们运行这个应用程序，我们将在端口 9393 上拥有一个本地数据流服务器。

## 3.创建项目

我们将[创建一个 Spark 作业](/web/20220625085016/https://www.baeldung.com/apache-spark)作为独立的本地应用程序，这样我们就不需要任何集群来运行它。

### 3.1.属国

首先，我们将添加[火花相关性](https://web.archive.org/web/20220625085016/https://search.maven.org/artifact/org.apache.spark/spark-core_2.10/2.2.3/jar):

```
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.10</artifactId>
    <version>2.4.0</version>
</dependency> 
```

### 3.2.创建工作

对于我们的工作，让我们近似圆周率:

```
public class PiApproximation {
    public static void main(String[] args) {
        SparkConf conf = new SparkConf().setAppName("BaeldungPIApproximation");
        JavaSparkContext context = new JavaSparkContext(conf);
        int slices = args.length >= 1 ? Integer.valueOf(args[0]) : 2;
        int n = (100000L * slices) > Integer.MAX_VALUE ? Integer.MAX_VALUE : 100000 * slices;

        List<Integer> xs = IntStream.rangeClosed(0, n)
          .mapToObj(element -> Integer.valueOf(element))
          .collect(Collectors.toList());

        JavaRDD<Integer> dataSet = context.parallelize(xs, slices);

        JavaRDD<Integer> pointsInsideTheCircle = dataSet.map(integer -> {
           double x = Math.random() * 2 - 1;
           double y = Math.random() * 2 - 1;
           return (x * x + y * y ) < 1 ? 1: 0;
        });

        int count = pointsInsideTheCircle.reduce((integer, integer2) -> integer + integer2);

        System.out.println("The pi was estimated as:" + count / n);

        context.stop();
    }
}
```

## 4.数据流外壳

数据流外壳是一个应用程序，它将使我们能够与服务器进行交互。Shell 使用 DSL 命令来描述数据流。

为了使用数据流外壳,我们需要创建一个允许我们运行它的项目。首先，我们需要[`spring-cloud-dataflow-shell`依赖](https://web.archive.org/web/20220625085016/https://search.maven.org/search?q=spring-cloud-dataflow-shell):

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dataflow-shell</artifactId>
    <version>1.7.4.RELEASE</version>
</dependency>
```

添加依赖项后，我们可以创建运行数据流外壳的类:

```
@EnableDataFlowShell
@SpringBootApplication
public class SpringDataFlowShellApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringDataFlowShellApplication.class, args);
    }
}
```

## 5.部署项目

为了部署我们的项目，我们将使用 Apache Spark 的三个版本的任务运行器:`cluster`、`yarn`和`client`。我们将继续使用本地`client`版本。

任务执行人是我们 Spark 工作的执行者。

为此，我们首先需要**使用数据流外壳**注册我们的任务:

```
app register --type task --name spark-client --uri maven://org.springframework.cloud.task.app:spark-client-task:1.0.0.BUILD-SNAPSHOT 
```

该任务允许我们指定多个不同的参数，其中一些是可选的，但一些参数是正确部署 Spark 作业所必需的:

*   `spark.app-class`，我们提交的作业的主类
*   `spark.app-jar`，一条通往装着我们工作的肥罐子的路
*   `spark.app-` `name`，这个名字将用于我们的工作
*   `spark.app-args`，将要传递给作业的参数

我们可以使用注册的任务`spark-client`来提交我们的作业，记住要提供所需的参数:

```
task create spark1 --definition "spark-client \
  --spark.app-name=my-test-pi --spark.app-class=com.baeldung.spring.cloud.PiApproximation \
  --spark.app-jar=/apache-spark-job-0.0.1-SNAPSHOT.jar --spark.app-args=10"
```

请注意，`spark.app-jar`是我们的工作通向脂肪罐的路径。

成功创建任务后，我们可以使用以下命令继续运行它:

```
task launch spark1
```

这将调用我们任务的执行。

## 6.摘要

在本教程中，我们展示了如何使用 Spring Cloud 数据流框架通过 Apache Spark 处理数据。关于 Spring Cloud 数据流框架的更多信息可以在[文档](https://web.archive.org/web/20220625085016/https://cloud.spring.io/spring-cloud-dataflow/)中找到。

所有代码示例[都可以在 GitHub 上找到。](https://web.archive.org/web/20220625085016/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-data-flow/apache-spark-job)