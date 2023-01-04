# JIRA REST API 集成

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jira-rest-api>

## 1。简介

在本文中，我们将快速了解如何使用 REST API 与 JIRA 集成。

## 2。Maven 依赖关系

所需的工件可以在 Atlassian 的公共 Maven 资源库中找到:

```
<repository>
    <id>atlassian-public</id>
    <url>https://packages.atlassian.com/maven/repository/public</url>
</repository>
```

一旦存储库被添加到`pom.xml`，我们需要添加下面的依赖项:

```
<dependency>
    <groupId>com.atlassian.jira</groupId>
    <artifactId>jira-rest-java-client-core</artifactId>
    <version>4.0.0</version>
</dependency>
<dependency>
    <groupId>com.atlassian.fugue</groupId>
    <artifactId>fugue</artifactId>
    <version>2.6.1</version>
</dependency>
```

关于`[core](https://web.archive.org/web/20220626194457/https://mvnrepository.com/artifact/com.atlassian.jira/jira-rest-java-client-core)`和`[fugue](https://web.archive.org/web/20220626194457/https://mvnrepository.com/artifact/com.atlassian.fugue/fugue)`依赖项的最新版本，可以参考 Maven Central。

## 3。创建吉拉客户端

首先，让我们看一下连接到吉拉实例所需的一些基本信息:

*   `username`–任何有效吉拉用户的用户名
*   `password`–是该用户的密码
*   `jiraUrl`–是托管吉拉实例的 URL

一旦我们有了这些细节，我们就可以实例化我们的吉拉客户端:

```
MyJiraClient myJiraClient = new MyJiraClient(
  "user.name", 
  "password", 
  "http://jira.company.com");
```

该类的构造函数:

```
public MyJiraClient(String username, String password, String jiraUrl) {
    this.username = username;
    this.password = password;
    this.jiraUrl = jiraUrl;
    this.restClient = getJiraRestClient();
}
```

`getJiraRestClient()`利用所有提供的信息并返回一个`JiraRestClient`的实例。这是我们与吉拉 REST API 通信的主要接口:

```
private JiraRestClient getJiraRestClient() {
    return new AsynchronousJiraRestClientFactory()
      .createWithBasicHttpAuthentication(getJiraUri(), this.username, this.password);
}
```

这里，我们使用基本身份验证与 API 通信。但是，也支持更复杂的身份验证机制，如 OAuth。

`getUri()`方法只是将`jiraUrl`转换成`java.net.URI`的一个实例:

```
private URI getJiraUri() {
    return URI.create(this.jiraUrl);
}
```

这就结束了我们创建定制吉拉客户端的基础结构。我们现在可以看看与 API 交互的各种方式。

### 3.1。创建新问题

让我们从创建一个新问题开始。我们将在本文的所有其他示例中使用这个新创建的问题:

```
public String createIssue(String projectKey, Long issueType, String issueSummary) {
    IssueRestClient issueClient = restClient.getIssueClient();
    IssueInput newIssue = new IssueInputBuilder(
      projectKey, issueType, issueSummary).build();
    return issueClient.createIssue(newIssue).claim().getKey();
}
```

`projectKey`是定义您的项目的惟一元素。这只是附加在我们所有问题上的前缀。下一个参数`issueType`也是项目相关的，它标识了你的问题的类型，比如“任务”或“故事”。《T2》是我们这一期的标题。

这个问题作为`IssueInput`的一个实例传递给 rest API。除了我们描述的输入之外，诸如受托人、报告者、受影响的版本和其他元数据之类的东西可以作为一个`IssueInput`出现。

### 3.2。更新问题描述

吉拉的每一期都有一个独特的`String`，就像`MYKEY-123`一样。我们需要这个问题密钥来与 rest API 交互，并更新问题的描述:

```
public void updateIssueDescription(String issueKey, String newDescription) {
    IssueInput input = new IssueInputBuilder()
      .setDescription(newDescription)
      .build();
    restClient.getIssueClient()
      .updateIssue(issueKey, input)
      .claim();
}
```

描述更新后，我们不要回读更新后的描述:

```
public Issue getIssue(String issueKey) {
    return restClient.getIssueClient()
      .getIssue(issueKey) 
      .claim();
}
```

`Issue`实例表示由`issueKey`识别的问题。我们可以用这个实例来阅读这个问题的描述:

```
Issue issue = myJiraClient.getIssue(issueKey);
System.out.println(issue.getDescription());
```

这将把问题的描述打印到控制台。

### 3.3。为一个问题投票

一旦我们获得了问题的实例，我们也可以使用它来执行更新/编辑操作。让我们为这个问题投票:

```
public void voteForAnIssue(Issue issue) {
    restClient.getIssueClient()
      .vote(issue.getVotesUri())
      .claim();
}
```

这将代表其凭证被使用的用户向`issue`添加投票。这可以通过检查计票来验证:

```
public int getTotalVotesCount(String issueKey) {
    BasicVotes votes = getIssue(issueKey).getVotes();
    return votes == null ? 0 : votes.getVotes();
}
```

这里需要注意的一点是，我们再次在这里获取一个新的实例`Issue`,因为我们希望得到更新后的投票数。

### 3.4。添加评论

我们可以使用同一个`Issue`实例来代表用户添加评论。像添加投票一样，添加评论也非常简单:

```
public void addComment(Issue issue, String commentBody) {
    restClient.getIssueClient()
      .addComment(issue.getCommentsUri(), Comment.valueOf(commentBody));
}
```

我们使用由`Comment`类提供的工厂方法`valueOf()`来创建一个`Comment`的实例。对于高级用例，还有各种其他的工厂方法，比如控制一个`Comment`的可见性。

让我们获取一个新的`Issue`实例，并读取所有的`Comment`:

```
public List<Comment> getAllComments(String issueKey) {
    return StreamSupport.stream(getIssue(issueKey).getComments().spliterator(), false)
      .collect(Collectors.toList());
}
```

### 3.5。删除一个问题

删除问题也相当简单。我们只需要识别问题的问题密钥:

```
public void deleteIssue(String issueKey, boolean deleteSubtasks) {
    restClient.getIssueClient()
      .deleteIssue(issueKey, deleteSubtasks)
      .claim();
}
```

## 4。结论

在这篇简短的文章中，我们创建了一个简单的 Java 客户端，它集成了吉拉 REST API 并执行一些基本操作。

这篇文章的完整来源可以在 GitHub 上找到[。](https://web.archive.org/web/20220626194457/https://github.com/eugenp/tutorials/tree/master/saas)