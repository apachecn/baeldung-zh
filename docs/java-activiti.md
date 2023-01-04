# Java 活动指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-activiti>

## 1。概述

Activiti API 是一个工作流和业务流程管理系统。我们可以在其中定义一个流程，执行它，并使用 API 提供的服务以不同的方式操作它。它需要 JDK 7+。

使用 API 的开发可以在任何 IDE 中完成，但是要使用 [Activiti Designer](https://web.archive.org/web/20220627092156/https://www.activiti.org/userguide/index.html?_ga=2.55182893.76071610.1499064413-1368418377.1499064413#eclipseDesignerInstallation) ，我们需要 Eclipse。

我们可以使用 BPMN 2.0 标准在 it 中定义一个流程。还有一种不太流行的方法——使用 Java 类，如`StartEvent`、`EndEvent`、`UserTask`、`SequenceFlow`等。

如果我们想要运行一个流程或访问任何服务，我们需要创建一个`ProcessEngineConfiguration`。

我们可以通过某些方式使用`ProcessEngineConfiguration,` 得到`ProcessEngine`，这我们将在本文中进一步讨论`.`通过*T5`ProcessEngine`我们可以执行工作流和 BPMN 操作。*

## 2。Maven 依赖关系

要使用这个 API，我们需要包含 Activiti 依赖项:

```java
<dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-engine</artifactId>
</dependency>
```

## 3。创造一个`ProcessEngine`

在 Activiti 中，`ProcessEngine`通常使用 XML 文件`activiti.cfg.xml`进行配置。此配置文件的一个示例是:

```java
<beans >
    <bean id="processEngineConfiguration" class=
      "org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
        <property name="jdbcUrl" 
          value="jdbc:h2:mem:activiti;DB_CLOSE_DELAY=1000" />
        <property name="jdbcDriver" value="org.h2.Driver" />
        <property name="jdbcUsername" value="root" />
        <property name="jdbcPassword" value="" />
        <property name="databaseSchemaUpdate" value="true" />
    </bean>
</beans> 
```

现在我们可以使用`ProcessEngines`类获得`ProcessEngine`:

```java
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
```

该语句将在类路径中查找一个`activiti.cfg.` xml 文件，并根据文件中的配置构造一个`ProcessEngine` 。

配置文件的示例代码显示它只是一个基于 Spring 的配置。但是，这并不意味着我们只能在 Spring 环境中使用 Activiti。Spring 的功能只是在内部用来创建`ProcessEngine`。

让我们编写一个 JUnit 测试用例，它将使用上面显示的配置文件创建`ProcessEngine`:

```java
@Test
public void givenXMLConfig_whenGetDefault_thenGotProcessEngine() {
    ProcessEngine processEngine 
      = ProcessEngines.getDefaultProcessEngine();
    assertNotNull(processEngine);
    assertEquals("root", processEngine.getProcessEngineConfiguration()
      .getJdbcUsername());
} 
```

## 4。Activiti 流程引擎 API 和服务

与 API 交互的入口点是`ProcessEngine`。通过`ProcessEngine,`，我们可以访问各种提供工作流/BPMN 方法的服务。`ProcessEngine`和所有的服务对象都是线程安全的。

[![activiti](img/e590f64c4d597f4cdf3a96e234301bfa.png)](/web/20220627092156/https://www.baeldung.com/wp-content/uploads/2017/07/activiti.png)

取自 https://www.activiti.org/userguiimg/api.services.png

`ProcessEngines`类将扫描`activiti.cfg.xml`和`activiti-context.xml`文件。如前所述，对于所有的`activiti.cfg.xml`文件，`ProcessEngine`将以一种典型的方式创建。

然而，对于所有的`activiti-context.xml`文件，它将以 Spring 方式创建——我将创建 Spring 应用程序上下文，并从中获得`ProcessEngine`。在流程执行过程中，将按照 BPMN 文件中定义的顺序访问所有步骤。

在流程执行过程中，将按照 BPMN 文件中定义的顺序访问所有步骤。

### 4.1。过程定义和相关术语

**一个`ProcessDefinition`代表一个业务流程。**它用于定义流程中不同步骤的结构和行为。部署流程定义意味着将流程定义加载到 Activiti 数据库中。

过程定义主要是由 BPMN 2.0 标准定义的。也可以使用 Java 代码来定义它们。本节中定义的所有术语也可以作为 Java 类使用。

一旦我们开始运行一个过程定义，它可以被称为一个过程

**一个`ProcessInstance`是一个`ProcessDefinition.`T3 的一次执行**

**一个`StartEvent`与每一个业务流程相关联。它指示流程的入口点。**同样，有一个`EndEvent`表示过程结束。我们可以定义这些事件的条件。

开始和结束之间的所有步骤(或元素)称为`Tasks`。`Tasks`可以是各种类型。最常用的任务是`UserTasks`和`ServiceTasks`。

顾名思义，它们需要由用户手动执行。

另一方面，`ServiceTasks`是用一段代码配置的。每当执行到达他们时，他们的代码块将被执行。

`SequenceFlows`连接`Tasks`。我们可以通过它们将要连接的源元素和目标元素来定义`SequenceFlows`。同样，我们还可以在`SequenceFlows`上定义条件，以在流程中创建条件路径。

### 4.2。服务

我们将简要讨论 Activiti 提供的服务:

*   帮助我们操纵过程定义的部署。该服务处理与过程定义相关的静态数据
*   `**RuntimeService**`管理`ProcessInstances`(当前运行的进程)以及进程变量
*   `**TaskService**`跟踪`UserTasks`。需要用户手动执行的`Tasks`是 Activiti API 的核心。我们可以创建一个任务，声明并完成一个任务，操纵任务的受托人，等等。使用此服务
*   `**FormService**`是一项可选服务。该 API 可以在没有它的情况下使用，并且不会牺牲它的任何特性。它用于定义流程中的开始表单和任务表单。
*   `**IdentityService**`管理着`Users`和`Groups`
*   跟踪 Activiti 引擎的历史。我们也可以设置不同的历史水平。
*   `**ManagementService**`与元数据相关，在创建应用程序时通常不需要
*   帮助我们在不重新部署的情况下更改流程中的任何内容

## 5。使用 Activiti 服务

为了了解我们如何处理不同的服务并运行流程，让我们以“员工休假请求”的流程为例:

[![vacation request process](img/f7075f4a9a1f83c5e3bf3231d19c45e2.png)](/web/20220627092156/https://www.baeldung.com/wp-content/uploads/2017/07/vacation-request-process.png)

此流程的 BPMN 2.0 文件`VacationRequest.bpmn20.xml`将启动事件定义为:

```java
<startEvent id="startEvent" name="request" 
  activiti:initiator="employeeName">
    <extensionElements>
        <activiti:formProperty id="numberOfDays" 
          name="Number of days" type="long" required="true"/>
        <activiti:formProperty id="startDate" 
          name="Vacation start date (MM-dd-yyyy)" type="date" 
          datePattern="MM-dd-yyyy hh:mm" required="true"/>
        <activiti:formProperty id="reason" name="Reason for leave" 
          type="string"/>
     </extensionElements>
</startEvent> 
```

类似地，分配给用户组“管理”的第一个用户任务将如下所示:

```java
<userTask id="handle_vacation_request" name=
  "Handle Request for Vacation">
    <documentation>${employeeName} would like to take ${numberOfDays} day(s)
      of vacation (Motivation: ${reason}).</documentation>
    <extensionElements>
        <activiti:formProperty id="vacationApproved" name="Do you approve
          this vacation request?" type="enum" required="true"/>
        <activiti:formProperty id="comments" name="Comments from Manager"
          type="string"/>
    </extensionElements>
    <potentialOwner>
      <resourceAssignmentExpression>
        <formalExpression>management</formalExpression>
      </resourceAssignmentExpression>
    </potentialOwner>
</userTask>
```

使用`ServiceTask,`,我们需要定义要执行的代码。我们有这段 Java 类代码:

```java
<serviceTask id="send-email-confirmation" name="Send email confirmation" 
  activiti:class=
  "com.example.activiti.servicetasks.SendEmailServiceTask.java">
</serviceTask>
```

条件流将通过在`“sequenceFlow”:`中添加`“conditionExpression”`标签来显示

```java
<sequenceFlow id="flow3" name="approved" 
  sourceRef="sid-12A577AE-5227-4918-8DE1-DC077D70967C" 
  targetRef="send-email-confirmation">
    <conditionExpression xsi:type="tFormalExpression">
      <![CDATA[${vacationApproved == 'true'}]]>
    </conditionExpression>
</sequenceFlow>
```

这里的`vacationApproved`就是上面显示的`UserTask` 的`formProperty`。

正如我们在图中看到的，这是一个非常简单的过程。员工提出休假请求，提供休假天数和开始日期。这个请求被提交给经理。他们可以批准/不批准该请求。

如果获得批准，将定义一个服务任务来发送确认电子邮件。如果未被批准，员工可以选择修改并重新发送请求，或者什么也不做。

服务任务提供了一些要执行的代码(这里是一个 Java 类)。我们已经上了`SendEmailServiceTask.java.` 课

这些类型的类应该扩展`JavaDelegate.` 并且，我们需要覆盖它的`execute()`方法，这将在流程执行到这一步时执行。

### 5.1。部署流程

为了让 Activiti 引擎知道我们的流程，我们需要部署流程。我们可以使用`RepositoryService.` 以编程的方式实现它。让我们编写一个 JUnit 测试来展示这一点:

```java
@Test 
public void givenBPMN_whenDeployProcess_thenDeployed() {
    ProcessEngine processEngine 
      = ProcessEngines.getDefaultProcessEngine();
    RepositoryService repositoryService 
      = processEngine.getRepositoryService();
    repositoryService.createDeployment()
      .addClasspathResource(
      "org/activiti/test/vacationRequest.bpmn20.xml")
      .deploy();
    Long count=repositoryService.createProcessDefinitionQuery().count();
    assertEquals("1", count.toString());
}
```

部署意味着引擎将解析 BPMN 文件并将其转换成可执行文件。此外，将为每个部署向存储库表添加一条记录。

因此，之后，我们可以查询`Repository`服务来获取部署的流程；`ProcessDefinitions`。

### 5.2。`ProcessInstance`首发

在将`ProcessDefinition`部署到 Activiti 引擎之后，我们可以通过创建`ProcessInstances`来执行流程。`ProcessDefinition`是一个蓝图，`ProcessInstance`是它的运行时执行。

对于单个`ProcessDefinition`，可以有多个`ProcessInstances`。

与`ProcessInstances`相关的所有细节都可以通过`RuntimeService`访问。

在我们的例子中，在开始事件中，我们需要传递假期天数、开始日期和原因。我们将使用流程变量，并在创建`ProcessInstance.` 时传递它们

让我们编写一个 JUnit 测试用例来获得更好的想法:

```java
@Test
public void givenDeployedProcess_whenStartProcessInstance_thenRunning() {
    //deploy the process definition    
    Map<String, Object> variables = new HashMap>();
    variables.put("employeeName", "John");
    variables.put("numberOfDays", 4);
    variables.put("vacationMotivation", "I need a break!");

    RuntimeService runtimeService = processEngine.getRuntimeService();
    ProcessInstance processInstance = runtimeService
      .startProcessInstanceByKey("vacationRequest", variables);
    Long count=runtimeService.createProcessInstanceQuery().count();

    assertEquals("1", count.toString());
}
```

单个过程定义的多个实例将因过程变量而不同。

启动流程实例有多种方式。在这里，我们使用了过程的关键。在启动流程实例之后，我们可以通过查询`RuntimeService`来获得关于它的信息。

### 5.3。完成任务

当我们的流程实例开始运行时，第一步是一个用户任务，分配给用户组`“management”.`

用户可能有一个收件箱，其中有他们要完成的任务列表。现在，如果我们想继续流程执行，用户需要完成这个任务。对于 Activiti Engine 来说，叫做“完成任务”。

我们可以查询`TaskService,` 来获取任务对象，然后完成它。

我们需要为此编写的代码如下所示:

```java
@Test 
public void givenProcessInstance_whenCompleteTask_thenGotNextTask() {
    // deploy process and start process instance   
    TaskService taskService = processEngine.getTaskService();
    List<Task> tasks = taskService.createTaskQuery()
      .taskCandidateGroup("management").list();
    Task task = tasks.get(0);

    Map<String, Object> taskVariables = new HashMap<>();
    taskVariables.put("vacationApproved", "false");
    taskVariables.put("comments", "We have a tight deadline!");
    taskService.complete(task.getId(), taskVariables);

    Task currentTask = taskService.createTaskQuery()
      .taskName("Modify vacation request").singleResult();
    assertNotNull(currentTask);
}
```

注意，`TaskService`的`complete()`方法也接受所需的过程变量。我们递交了经理的答复。

在此之后，流程引擎将继续下一步。这里，下一步询问员工是否要重新发送休假请求。

所以，我们的`ProcessInstance`现在正在等待这个`UserTask,` ，它的名字是【T2 请求】。

### 5.4。暂停和激活进程

我们可以暂停一个`ProcessDefinition`和一个`ProcessInstance`。如果我们挂起了一个`ProcessDefinition,` ，我们就不能在它挂起的时候创建它的实例。我们可以使用`RepositoryService:`来完成这项工作

```java
@Test(expected = ActivitiException.class)
public void givenDeployedProcess_whenSuspend_thenNoProcessInstance() {
    // deploy the process definition
    repositoryService.suspendProcessDefinitionByKey("vacationRequest");
    runtimeService.startProcessInstanceByKey("vacationRequest");
} 
```

要再次激活它，我们只需要调用其中一个`repositoryService.activateProcessDefinitionXXX`方法。

类似地，我们可以使用`RuntimeService.` 来暂停`ProcessInstance,`

## 6。结论

在本文中，我们看到了如何在 Java 中使用 Activiti。我们创建了一个示例文件`ProcessEngineCofiguration` ，它帮助我们创建`ProcessEngine`。

使用它，我们可以访问 API 提供的各种服务。这些服务帮助我们管理和跟踪`ProcessDefinitions`、`ProcessInstances`、`UserTasks`等。

和往常一样，我们在文章中看到的示例代码位于 GitHub 上的[。](https://web.archive.org/web/20220627092156/https://github.com/eugenp/tutorials/tree/master/spring-activiti)