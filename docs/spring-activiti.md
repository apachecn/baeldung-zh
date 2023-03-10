# Spring 活动简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-activiti>

## 1。概述

简单来说， **Activiti 是一个工作流和业务流程管理平台。**

我们可以通过创建一个`ProcessEngineConfiguration`(通常基于配置文件)来快速开始。由此，我们可以获得一个`ProcessEngine`——并且通过`ProcessEngine,`我们可以执行工作流& BPM 操作。

API 提供了各种可用于访问和管理流程的服务。这些服务可以为我们提供有关流程历史、当前正在运行的内容以及已部署但尚未运行的流程的信息。

这些服务还可用于定义流程结构和操作流程状态，即运行、暂停、取消等。

如果你是 API 新手，可以看看我们的**[Java 的 Activiti API 简介](/web/20220628100146/https://www.baeldung.com/java-activiti)** 。在本文中，我们将讨论如何在 Spring Boot 应用程序中设置 Activiti API。

## 2。Spring Boot 的设置

让我们看看如何将 Activiti 设置为 Spring Boot Maven 应用程序并开始使用它。

### 2.1。初始设置

像往常一样，我们需要添加 maven 依赖项:

```java
<dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-spring-boot-starter-basic</artifactId>
</dependency>
```

API 的最新稳定版本可以在[这里](https://web.archive.org/web/20220628100146/https://mvnrepository.com/artifact/org.activiti/activiti-engine)找到。它适用于 Spring Boot 1 . 5 . 4 版，但不适用于 2.0.0.M1 版。

我们还可以使用 [https://start.spring.io](https://web.archive.org/web/20220628100146/https://start.spring.io/) 生成一个 Spring Boot 项目，并选择 Activiti 作为依赖项。

只要将这个依赖项和`@EnableAutoConfiguration`注释添加到 Spring Boot 应用程序中，它就会完成初始设置:

*   创建数据源(API 需要一个数据库来创建`ProcessEngine`)
*   创建并公开`ProcessEngine` bean
*   创建并公开活动服务 beans
*   创建 Spring 作业执行器

### 2.2。创建和运行流程

让我们构建一个创建和运行业务流程的例子。为了定义一个过程，我们需要创建一个 BPMN 文件。

然后，只需下载 BPMN 文件。我们需要将这个文件放在`src/main/resources/processes`文件夹中。默认情况下，Spring Boot 将在该文件夹中查找以部署过程定义。

我们将创建一个包含一个用户任务的演示流程:

[![startEvent endEvent](img/094e40ef931a9d5967b2ddc47f0e9e8c.png)](/web/20220628100146/https://www.baeldung.com/wp-content/uploads/2017/08/image-9.jpg)

用户任务的受理人被设置为流程的发起者。这个过程定义的 BPMN 文件如下所示:

```java
 <process id="my-process" name="say-hello-process" isExecutable="true">
     <startEvent id="startEvent" name="startEvent">
     </startEvent>
     <sequenceFlow id="sequence-flow-1" sourceRef="startEvent" targetRef="A">
     </sequenceFlow>     
     <userTask id="A" name="A" activiti:assignee="$INITIATOR">
     </userTask>
     <sequenceFlow id="sequence-flow-2" sourceRef="A" targetRef="endEvent">
     </sequenceFlow>
     <endEvent id="endEvent" name="endEvent">
     </endEvent>
</process>
```

现在，我们将创建一个 REST 控制器来处理启动该流程的请求:

```java
@Autowired
private RuntimeService runtimeService;

@GetMapping("/start-process")
public String startProcess() {

    runtimeService.startProcessInstanceByKey("my-process");
    return "Process started. Number of currently running"
      + "process instances = "
      + runtimeService.createProcessInstanceQuery().count();
}
```

这里，`runtimeService.startProcessInstanceByKey(“my-process”)` 开始执行键为`“my-process”`的进程。`runtimeService.createProcessInstanceQuery().count()`将获取流程实例的数量。

每次我们点击路径`“/start-process”`，一个新的`ProcessInstance`将被创建，我们将看到当前正在运行的进程的计数增加。

JUnit 测试案例向我们展示了这种行为:

```java
@Test
public void givenProcess_whenStartProcess_thenIncreaseInProcessInstanceCount() 
  throws Exception {

    String responseBody = this.mockMvc
      .perform(MockMvcRequestBuilders.get("/start-process"))
      .andReturn().getResponse().getContentAsString();

    assertEquals("Process started. Number of currently running"
      + " process instances = 1", responseBody);

    responseBody = this.mockMvc
      .perform(MockMvcRequestBuilders.get("/start-process"))
      .andReturn().getResponse().getContentAsString();

    assertEquals("Process started. Number of currently running"
      + " process instances = 2", responseBody);

    responseBody = this.mockMvc
      .perform(MockMvcRequestBuilders.get("/start-process"))
      .andReturn().getResponse().getContentAsString();

    assertEquals("Process started. Number of currently running"
      + " process instances = 3", responseBody);
} 
```

## 3。玩进程

现在我们已经在 Activiti 中使用 Spring Boot 运行了一个流程，让我们扩展上面的例子来演示我们如何访问和操作这个流程。

### 3.1。获取给定`ProcessInstance` 的`Tasks`列表

我们有两个用户任务`A`和`B`。当我们启动一个进程时，它会等待第一个任务`A`完成，然后执行任务`B`。让我们创建一个处理程序方法，该方法接受查看与给定的`processInstance`相关的任务的请求。

像`Task`这样的对象不能直接作为响应发送，因此我们需要创建一个自定义对象，并将`Task`转换为我们的自定义对象。我们称这个类为`TaskRepresentation`:

```java
class TaskRepresentation {
    private String id;
    private String name;
    private String processInstanceId;

    // standard constructors
}
```

处理程序方法将类似于:

```java
@GetMapping("/get-tasks/{processInstanceId}")
public List<TaskRepresentation> getTasks(
  @PathVariable String processInstanceId) {

    List<Task> usertasks = taskService.createTaskQuery()
      .processInstanceId(processInstanceId)
      .list();

    return usertasks.stream()
      .map(task -> new TaskRepresentation(
        task.getId(), task.getName(), task.getProcessInstanceId()))
      .collect(Collectors.toList());
} 
```

在这里，`taskService.createTaskQuery().processInstanceId(processInstanceId).list()` 使用`TaskService`并获得与给定的`processInstanceId`相关的任务列表。我们可以看到，当我们开始运行我们创建的流程时，我们将通过向我们刚刚定义的方法发出请求来获得任务`A`:

```java
@Test
public void givenProcess_whenProcessInstance_thenReceivedRunningTask() 
  throws Exception {

    this.mockMvc.perform(MockMvcRequestBuilders.get("/start-process"))
      .andReturn()
      .getResponse();
    ProcessInstance pi = runtimeService.createProcessInstanceQuery()
      .orderByProcessInstanceId()
      .desc()
      .list()
      .get(0);
    String responseBody = this.mockMvc
      .perform(MockMvcRequestBuilders.get("/get-tasks/" + pi.getId()))
      .andReturn()
      .getResponse()
      .getContentAsString();

    ObjectMapper mapper = new ObjectMapper();
    List<TaskRepresentation> tasks = Arrays.asList(mapper
      .readValue(responseBody, TaskRepresentation[].class));

    assertEquals(1, tasks.size());
    assertEquals("A", tasks.get(0).getName());
}
```

### 3.2。完成一个`Task`

现在，我们来看看完成任务`A`后会发生什么。我们创建一个 handler 方法，它将处理为给定的`processInstance`完成任务`A`的请求:

```java
@GetMapping("/complete-task-A/{processInstanceId}")
public void completeTaskA(@PathVariable String processInstanceId) {
    Task task = taskService.createTaskQuery()
      .processInstanceId(processInstanceId)
      .singleResult();
    taskService.complete(task.getId());
}
```

`taskService.createTaskQuery().processInstanceId(processInstanceId).singleResult()`在任务服务上创建一个查询，并给出给定`processInstance`的任务。这就是`UserTask A`。下一行`taskService.complete(task.getId)`完成这个任务。
因此，现在该过程已经结束，`RuntimeService`不包含任何`ProcessInstances`。我们可以使用 JUnit 测试用例来看到这一点:

```java
@Test
public void givenProcess_whenCompleteTaskA_thenNoProcessInstance() 
  throws Exception {

    this.mockMvc.perform(MockMvcRequestBuilders.get("/start-process"))
      .andReturn()
      .getResponse();
    ProcessInstance pi = runtimeService.createProcessInstanceQuery()
      .orderByProcessInstanceId()
      .desc()
      .list()
      .get(0);
    this.mockMvc.perform(MockMvcRequestBuilders.get("/complete-task-A/" + pi.getId()))
      .andReturn()
      .getResponse()
      .getContentAsString();
    List<ProcessInstance> list = runtimeService.createProcessInstanceQuery().list();
    assertEquals(0, list.size());
}
```

这就是我们如何使用 Activiti 服务处理流程。

## 4。结论

在本文中，我们浏览了在 Spring Boot `.`中使用 Activiti API 的概述。关于 API 的更多信息可以在[用户指南](https://web.archive.org/web/20220628100146/https://www.activiti.org/userguide/)中找到。我们还看到了如何使用 Activiti 服务创建一个流程并在其上执行各种操作。

Spring Boot 使它易于使用，因为我们不需要担心创建数据库、部署流程或创建`ProcessEngine`。

请记住，Activiti 与 Spring Boot 的集成仍处于试验阶段，它还不被新 Spring Boot 协议所支持。

和往常一样，我们看到的所有例子的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20220628100146/https://github.com/eugenp/tutorials/tree/master/spring-activiti)