# OptaPlanner 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/opta-planner>

## 1.OptaPlanner 简介

在本教程中，我们来看一个名为 [OptaPlanner](https://web.archive.org/web/20220912123853/https://www.optaplanner.org/) 的 Java 约束满足求解器。

OptaPlanner 使用一套设置最少的算法来解决规划问题。

虽然对算法的理解可能会提供有用的细节，但框架为我们完成了艰巨的工作。

## 2.Maven 依赖性

首先，我们将为 OptaPlanner 添加一个 Maven 依赖项:

```
<dependency>
    <groupId>org.optaplanner</groupId>
    <artifactId>optaplanner-core</artifactId>
    <version>8.24.0.Final</version>
</dependency> 
```

我们从 [Maven 中央存储库](https://web.archive.org/web/20220912123853/https://search.maven.org/search?q=g:org.optaplanner%20AND%20a:optaplanner-core&core=gav)中找到 OptaPlanner 的最新版本。

## 3.问题/解决方案类

为了解决一个问题，我们当然需要一个具体的例子。

由于难以平衡教室、时间和教师等资源，讲座时间表就是一个合适的例子。

### 3.1.`CourseSchedule`

包含我们的问题变量和规划实体的组合，因此它是解决方案类。因此，我们使用多个注释来配置它。

让我们分别仔细看看:

```
@PlanningSolution
public class CourseSchedule {

    @ValueRangeProvider(id = "availableRooms")
    @ProblemFactCollectionProperty
    private List<Integer> roomList;
    @ValueRangeProvider(id = "availablePeriods")
    @ProblemFactCollectionProperty
    private List<Integer> periodList;
    @ProblemFactCollectionProperty
    private List<Lecture> lectureList;
    @PlanningScore
    private HardSoftScore score;
```

`PlanningSolution `注释告诉 OptaPlanner 这个类包含包含解决方案的数据。

OptaPlanner 期望这些最基本的组成部分:规划实体、问题事实和分数。

### 3.2.`Lecture`

`Lecture,`一个 POJO，看起来像:

```
@PlanningEntity
public class Lecture {

    @PlaningId
    private Long id;
    public Integer roomNumber;
    public Integer period;
    public String teacher;

    @PlanningVariable(
      valueRangeProviderRefs = {"availablePeriods"})
    public Integer getPeriod() {
        return period;
    }

    @PlanningVariable(
      valueRangeProviderRefs = {"availableRooms"})
    public Integer getRoomNumber() {
        return roomNumber;
    }
}
```

我们使用 `Lecture`类作为规划实体，所以我们在`CourseSchedule`中的 getter 上添加了另一个注释:

```
@PlanningEntityCollectionProperty
public List<Lecture> getLectureList() {
    return lectureList;
}
```

我们的规划实体包含正在设置的约束。

`PlanningVariable`注释和`valueRangeProviderRef`注释将约束链接到问题事实。

稍后将在所有规划实体中对这些约束值进行评分。

### 3.3.问题事实

`roomNumber `和`period`变量作为约束彼此相似。

OptaPlanner 使用这些变量将解决方案作为逻辑结果进行评分。我们给两个`getter`方法都添加了注释:

```
@ValueRangeProvider(id = "availableRooms")
@ProblemFactCollectionProperty
public List<Integer> getRoomList() {
    return roomList;
}

@ValueRangeProvider(id = "availablePeriods")
@ProblemFactCollectionProperty
public List<Integer> getPeriodList() {
    return periodList;
} 
```

这些列表都是在`Lecture `字段中使用的可能值。

OptaPlanner 在搜索空间的所有解决方案中填充它们。

最后，它为每个解决方案设置一个分数，因此我们需要一个字段来存储分数:

```
@PlanningScore
public HardSoftScore getScore() {
    return score;
}
```

没有分数，OptaPlanner 无法找到最优解，因此之前强调了重要性。

## 4.得分

与我们到目前为止所看到的相反，scoring 类需要更多的定制代码。

这是因为分数计算器是特定于问题和领域模型的。

### 4.1.自定义 Java

我们用一个简单的分数计算来解决这个问题(虽然看起来可能不是这样):

```
public class ScoreCalculator 
  implements EasyScoreCalculator<CourseSchedule, HardSoftScore> {

    @Override
    public HardSoftScore calculateScore(CourseSchedule courseSchedule) {
        int hardScore = 0;
        int softScore = 0;

        Set<String> occupiedRooms = new HashSet<>();
        for(Lecture lecture : courseSchedule.getLectureList()) {
            String roomInUse = lecture.getPeriod()
              .toString() + ":" + lecture.getRoomNumber().toString();
            if(occupiedRooms.contains(roomInUse)){
                hardScore += -1;
            } else {
                occupiedRooms.add(roomInUse);
            }
        }

        return HardSoftScore.Of(hardScore, softScore);
    }
}
```

如果我们仔细看看上面的代码，重要的部分会变得更加清楚。**我们在循环中计算分数，因为`List<Lecture> `包含房间和时段的特定非唯一组合。**

`HashSet `用于保存一个唯一的键(字符串),这样我们就可以惩罚在同一教室和同一时间段的重复讲座。

因此，我们会收到不同的房间和时间段。

## 5.测试

我们配置了解决方案、规划求解和问题类。来测试一下吧！

### 5.1.设置我们的测试

首先，我们做一些设置:

```
SolverFactory<CourseSchedule> solverFactory = SolverFactory.create(new SolverConfig() 
                                                      .withSolutionClass(CourseSchedule.class)
                                                      .withEntityClasses(Lecture.class)
                                                      .withEasyScoreCalculatorClass(ScoreCalculator.class)
                                                      .withTerminationSpentLimit(Duration.ofSeconds(1))); 
solver = solverFactory.buildSolver();
```

```
unsolvedCourseSchedule = new CourseSchedule();
```

其次，我们将数据填充到规划实体集合和问题事实`List`对象中。

### 5.2.测试执行和验证

最后，我们通过调用`solve`来测试它。

```
CourseSchedule solvedCourseSchedule = solver.solve(unsolvedCourseSchedule);

assertNotNull(solvedCourseSchedule.getScore());
assertEquals(-4, solvedCourseSchedule.getScore().getHardScore());
```

我们检查`solvedCourseSchedule`是否有一个分数，这个分数告诉我们有“最优”的解决方案。

作为奖励，我们创建了一个打印方法来显示我们的优化解决方案:

```
public void printCourseSchedule() {
    lectureList.stream()
      .map(c -> "Lecture in Room "
        + c.getRoomNumber().toString() 
        + " during Period " + c.getPeriod().toString())
      .forEach(k -> logger.info(k));
}
```

该方法显示:

```
Lecture in Room 1 during Period 1
Lecture in Room 2 during Period 1
Lecture in Room 1 during Period 2
Lecture in Room 2 during Period 2
Lecture in Room 1 during Period 3
Lecture in Room 2 during Period 3
Lecture in Room 1 during Period 1
Lecture in Room 1 during Period 1
Lecture in Room 1 during Period 1
Lecture in Room 1 during Period 1
```

注意最后三个条目是如何重复的。这是因为我们的问题没有最优解。我们选了三节课，两个教室，十节课。

由于这些固定的资源，只有六个可能的讲座。至少这个答案向用户表明没有足够的房间或时间来容纳所有的讲座。

## 6.额外功能

我们为 OptaPlanner 创建的例子是一个简单的例子，但是，该框架为更多样化的用例添加了特性。我们可能想要实现或改变我们的优化算法，然后指定使用它的框架。

由于最近对 Java 多线程功能的改进，OptaPlanner 还使开发人员能够使用多线程的多种实现，如 fork 和 join、增量求解和多租户。

更多信息请参考[文档](https://web.archive.org/web/20220912123853/https://docs.optaplanner.org/7.9.0.Final/optaplanner-docs/html_single/index.html#multithreadedSolving)。

## 7.结论

OptaPlanner 框架为开发人员提供了一个强大的工具来解决约束满足问题，如调度和资源分配。

OptaPlanner 提供了最少的 JVM 资源使用以及与 Jakarta EE 的集成。作者继续支持该框架，Red Hat 已经将它添加为其业务规则管理套件的一部分。

和往常一样，代码可以在 Github 上找到[。](https://web.archive.org/web/20220912123853/https://github.com/eugenp/tutorials/tree/master/optaplanner)