# 在 Kubernetes 运行 Cron 作业

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/kubernetes-run-cron-jobs>

## 1.介绍

在本教程中，我们将看看如何在 [Kubernetes](/web/20220815225448/https://www.baeldung.com/ops/kubernetes) 中运行 cron 作业。

## 2.什么是 Cron 工作？

作为背景，**cron 作业指的是在时间表**上重复的任何任务。Unix 和大多数相关的操作系统都提供一些 cron 作业功能。

cron 作业的一个典型用例是定期自动执行重要任务。例如:

*   清理磁盘空间
*   备份文件或目录
*   生成指标或报告

Cron 作业按计划运行。使用[标准符号](/web/20220815225448/https://www.baeldung.com/cron-expressions)，我们可以定义各种各样的时间表来执行作业:

*   每晚 10 点
*   每月的第一天早上 6:00
*   每天早上 8 点和下午 6 点
*   每周二早上 7 点

## 3.在 Kubernetes 中定义 Cron 作业

**从 1.21 版本开始，Kubernetes 为 cron 作业提供一流的支持**。首先，让我们看看如何定义 cron 作业以及如何设置它的时间表。

### 3.1.定义 Cron 作业

Kubernetes 中的 Cron 作业类似于其他工作负载，比如部署或守护进程集。事实上，定义 cron 工作的 YAML 看起来非常相似:

```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cleanup-job
spec:
  schedule: "0 2 * * *"
  concurrencyPolicy: Allow
  suspend: false
  successfulJobsHistoryLimit: 10
  failedJobsHistoryLimit: 3
  startingDeadlineSeconds: 60
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cleanup-job
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/rm
            - -f
            - /tmp/*
          restartPolicy: OnFailure
```

上面的 YAML 定义了一个 cron 作业，它将在每天凌晨 2:00 运行，并从 temp 目录中清除文件。

如前所述，cron 作业的 YAML 与 Kubernetes 中的其他工作负载几乎相同。事实上，**配置的`jobTemplate`部分与部署、副本集和其他类型的工作负载**完全相同。

主要区别在于 cron 作业规范包含用于定义 cron 作业执行行为的附加字段:

*   `schedule`:使用标准 cron 语法指定 cron 作业计划的必需字段。
*   `concurrencyPolicy`:指定如何处理并发作业的可选字段。默认值是`Allow`，意味着可以同时运行多个作业。其他可能的值有`Forbid`(不允许)或`Replace`(新任务将替换任何正在运行的任务)。
*   `suspend`:可选字段，指定是否应跳过作业的未来执行。默认为`false`。
*   `successfulJobsHistoryLimit`:可选字段，指定历史中要跟踪的成功执行次数。默认值为 3。
*   `failedJobsHistoryLimit`:可选字段，指定历史中要跟踪的失败执行次数。默认值为 1。
*   `startingDeadlineSeconds`:可选字段，指定在作业被视为失败之前，允许其错过预定开始时间的秒数。默认情况下，不强制执行任何此类截止日期。

请注意，只有一个字段`schedule`是必需的。稍后我们将更仔细地研究这个领域。

### 3.2.管理 Cron 作业

现在我们已经看到了如何定义 cron 作业，让我们看看如何在 Kubernetes 中管理它们。

首先，假设我们将把 cron 作业 YAML 定义放到一个名为`cronjob.yaml`的文件中。然后，我们可以使用`kubelet`命令创建 cron 作业:

```
kubelet create -f /path/to/cronjob.yaml
```

此外，我们可以使用以下命令列出所有 cron 作业:

```
kubectl get cronjob
NAME          SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cleanup-job   * 2 * * *   False     0        15s             32s
```

我们还可以使用`describe`命令查看特定 cron 作业的详细信息，包括运行历史:

```
kubectl describe cronjob hello
Name:                          cleanup-job
Namespace:                     default
Labels:                        <none>
Annotations:                   <none>
Schedule:                      * 2 * * *
Concurrency Policy:            Allow
Suspend:                       False
Successful Job History Limit:  3
Failed Job History Limit:      1
Starting Deadline Seconds:     <unset>
Selector:                      <unset>
Parallelism:                   <unset>
Completions:                   <unset>
Pod Template:
  Labels:  <none>
  Containers:
   hello:
    Image:      busybox:1.28
    Port:       <none>
    Host Port:  <none>
    Command:
      /bin/rm
      -f
      /tmp/*
    Environment:     <none>
    Mounts:          <none>
  Volumes:           <none>
Last Schedule Time:  Mon, 30 May 2022 02:00:00 -0600
Active Jobs:         <none>
Events:
  Type    Reason            Age                   From                Message
  ----    ------            ----                  ----                -------
  Normal  SuccessfulCreate  16m                   cronjob-controller  Created job cleanup-job-27565242
  Normal  SawCompletedJob   16m                   cronjob-controller  Saw completed job: cleanup-job-27565242, status: Complete
```

最后，当我们不再需要 cron 作业时，我们可以使用以下命令删除它:

```
kubectl delete cronjob cleanup-job
```

### 3.3.Cron 计划语法

cron 作业计划语法包含由空格分隔的五个参数。每个参数可以是星号或数字。

参数的顺序与传统的 Unix cron 语法相同。从左到右，这些字段具有以下含义和可能的值:

*   分钟(0–59)
*   小时(0–23)
*   一个月中的第几天(1–31)
*   月份(1–12)
*   星期几(0–6)

**请注意，对于一个月中的某一天参数，一些系统将 0 视为星期日，而另一些系统将其视为星期一。**

除了上面确定的可能值，任何参数也可以是星号，这意味着它适用于该字段的所有可能值。

让我们看一些例子。首先，我们可以安排一个作业在每天上午 8:00 运行:

```
0 8 * * *
```

让我们看看每周二下午 5:00 运行作业的计划参数:

```
0 17 * * 2
```

最后，让我们看看如何在每月 15 日每隔一小时的 30 分钟运行一个作业:

```
30 0,2,4,6,8,10,12,14,16,18,20,22 15 * *
```

**注意，上述时间表也可以使用 skip 语法**来简化:

```
30 0-23/2 15 * *
```

### 3.4.特殊 Cron 作业条目

除了标准计划语法之外，cron 作业还可以使用许多特殊标识符来指定它们的计划:

*   @ annually/@ annually–在每年 1 月 1 日午夜运行作业
*   @ monthly–在每月第一天的午夜运行作业
*   @ weekly–在每周星期日午夜运行作业
*   @ daily/@ midnight–每天午夜运行作业
*   @ hourly–在每天每小时开始时运行作业

### 3.5.Cron 作业时区

**默认情况下，所有 Kubernetes cron 作业都在控制管理中心**的时区内运行。在某些情况下，可以使用变量`CRON_TZ`或`TZ`来指定特定的时区。然而，这并没有得到官方的支持。这些变量被视为内部实施细节，因此可能会在没有警告的情况下发生更改。

从 Kubernetes 版本 1.24 开始，可以将时区指定为 cron 作业规范的一部分:

```
spec:
  schedule: "0 2 * * *"
  timeZone: "GMT"
```

`timeZone`字段可以是任何有效的[时区标识符](https://web.archive.org/web/20220815225448/https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)。

由于该功能仍处于试验阶段，在使用之前，我们必须首先启用`CronJobTimeZone`功能门。

## 4.结论

Cron 作业是重复执行重要系统任务的好方法。

在本文中，我们研究了如何在 Kubernetes 集群中利用 cron 作业。首先，我们看到了定义 cron 作业所需的 YAML，以及如何使用`kubectl`命令管理其生命周期。最后，我们研究了定义他们的时间表以及如何处理时区的各种方法。