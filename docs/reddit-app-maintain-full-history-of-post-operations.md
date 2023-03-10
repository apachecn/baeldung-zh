# 保留 Reddit 帖子提交的历史

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reddit-app-maintain-full-history-of-post-operations>

## 1。概述

在本期的[Reddit 应用案例研究](/web/20220813070425/https://www.baeldung.com/case-study-a-reddit-app-with-spring)中，我们将开始跟踪**提交帖子**的历史，并使状态更具描述性和易于理解。

## 2。改善`Post`实体

首先，让我们开始用一个更完整的提交响应列表替换`Post`实体中的旧字符串状态，跟踪更多的信息:

```java
public class Post {
    ...
    @OneToMany(fetch = FetchType.EAGER, mappedBy = "post")
    private List<SubmissionResponse> submissionsResponse;
}
```

接下来，让我们看看我们在这个新的提交响应实体中实际跟踪了什么:

```java
@Entity
public class SubmissionResponse implements IEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private int attemptNumber;

    private String content;

    private Date submissionDate;

    private Date scoreCheckDate;

    @JsonIgnore
    @ManyToOne
    @JoinColumn(name = "post_id", nullable = false)
    private Post post;

    public SubmissionResponse(int attemptNumber, String content, Post post) {
        super();
        this.attemptNumber = attemptNumber;
        this.content = content;
        this.submissionDate = new Date();
        this.post = post;
    }

    @Override
    public String toString() {
        StringBuilder builder = new StringBuilder();
        builder.append("Attempt No ").append(attemptNumber).append(" : ").append(content);
        return builder.toString();
    }
}
```

注意，每个**消耗的提交尝试**都有一个`SubmissionResponse`，因此:

*   `attemptNumber`:本次尝试的次数
*   `content`:此次尝试的详细回应
*   `submissionDate`:本次尝试的提交日期
*   `scoreCheckDate`:我们在这次尝试中检查 Reddit `Post`分数的日期

下面是简单的 Spring 数据 JPA 存储库:

```java
public interface SubmissionResponseRepository extends JpaRepository<SubmissionResponse, Long> {

    SubmissionResponse findOneByPostAndAttemptNumber(Post post, int attemptNumber);
}
```

## 3。调度服务

我们现在需要开始修改服务层来跟踪这些额外的信息。

我们将首先确保我们有一个格式良好的成功或失败原因，来说明帖子被认为是成功还是失败:

```java
private final static String SCORE_TEMPLATE = "score %d %s minimum score %d";
private final static String TOTAL_VOTES_TEMPLATE = "total votes %d %s minimum total votes %d";

protected String getFailReason(Post post, PostScores postScores) { 
    StringBuilder builder = new StringBuilder(); 
    builder.append("Failed because "); 
    builder.append(String.format(
      SCORE_TEMPLATE, postScores.getScore(), "<", post.getMinScoreRequired())); 

    if (post.getMinTotalVotes() > 0) { 
        builder.append(" and "); 
        builder.append(String.format(TOTAL_VOTES_TEMPLATE, 
          postScores.getTotalVotes(), "<", post.getMinTotalVotes()));
    } 
    if (post.isKeepIfHasComments()) { 
        builder.append(" and has no comments"); 
    } 
    return builder.toString(); 
}

protected String getSuccessReason(Post post, PostScores postScores) {
    StringBuilder builder = new StringBuilder(); 
    if (postScores.getScore() >= post.getMinScoreRequired()) { 
        builder.append("Succeed because "); 
        builder.append(String.format(SCORE_TEMPLATE, 
          postScores.getScore(), ">=", post.getMinScoreRequired())); 
        return builder.toString(); 
    } 
    if (
      (post.getMinTotalVotes() > 0) && 
      (postScores.getTotalVotes() >= post.getMinTotalVotes())
    ) { 
        builder.append("Succeed because "); 
        builder.append(String.format(TOTAL_VOTES_TEMPLATE, 
          postScores.getTotalVotes(), ">=", post.getMinTotalVotes()));
        return builder.toString(); 
    } 
    return "Succeed because has comments"; 
} 
```

现在，我们将改进旧的逻辑并**跟踪这些额外的历史信息**:

```java
private void submitPost(...) {
    ...
    if (errorNode == null) {
        post.setSubmissionsResponse(addAttemptResponse(post, "Submitted to Reddit"));
        ...
    } else {
        post.setSubmissionsResponse(addAttemptResponse(post, errorNode.toString()));
        ...
    }
}
private void checkAndReSubmit(Post post) {
    if (didIntervalPass(...)) {
        PostScores postScores = getPostScores(post);
        if (didPostGoalFail(post, postScores)) {
            ...
            resetPost(post, getFailReason(post, postScores));
        } else {
            ...
            updateLastAttemptResponse(
              post, "Post reached target score successfully " + 
                getSuccessReason(post, postScores));
        }
    }
}
private void checkAndDeleteInternal(Post post) {
    if (didIntervalPass(...)) {
        PostScores postScores = getPostScores(post);
        if (didPostGoalFail(post, postScores)) {
            updateLastAttemptResponse(post, 
              "Deleted from reddit, consumed all attempts without reaching score " + 
                getFailReason(post, postScores));
            ...
        } else {
            updateLastAttemptResponse(post, 
              "Post reached target score successfully " + 
                getSuccessReason(post, postScores));
            ...
        }
    }
}
private void resetPost(Post post, String failReason) {
    ...
    updateLastAttemptResponse(post, "Deleted from Reddit, to be resubmitted " + failReason);
    ...
}
```

请注意较低级别的方法实际上在做什么:

*   `addAttemptResponse()`:创建一个新的`SubmissionResponse`记录，并将其添加到帖子中(在每次提交尝试时调用)
*   `updateLastAttemptResponse()`:更新最后一次尝试响应(检查帖子得分时调用)

## 4。预定后 DTO

接下来，我们将修改 DTO，以确保这些新信息能够公开给客户端:

```java
public class ScheduledPostDto {
    ...

    private String status;

    private List<SubmissionResponseDto> detailedStatus;
}
```

这里有一个简单的`SubmissionResponseDto`:

```java
public class SubmissionResponseDto {

    private int attemptNumber;

    private String content;

    private String localSubmissionDate;

    private String localScoreCheckDate;
}
```

我们还将在`ScheduledPostRestController`中修改转换方法:

```java
private ScheduledPostDto convertToDto(Post post) {
    ...
    List<SubmissionResponse> response = post.getSubmissionsResponse();
    if ((response != null) && (response.size() > 0)) {
        postDto.setStatus(response.get(response.size() - 1).toString().substring(0, 30));
        List<SubmissionResponseDto> responsedto = 
          post.getSubmissionsResponse().stream().
            map(res -> generateResponseDto(res)).collect(Collectors.toList());
        postDto.setDetailedStatus(responsedto);
    } else {
        postDto.setStatus("Not sent yet");
        postDto.setDetailedStatus(Collections.emptyList());
    }
    return postDto;
}

private SubmissionResponseDto generateResponseDto(SubmissionResponse responseEntity) {
    SubmissionResponseDto dto = modelMapper.map(responseEntity, SubmissionResponseDto.class);
    String timezone = userService.getCurrentUser().getPreference().getTimezone();
    dto.setLocalSubmissionDate(responseEntity.getSubmissionDate(), timezone);
    if (responseEntity.getScoreCheckDate() != null) {
        dto.setLocalScoreCheckDate(responseEntity.getScoreCheckDate(), timezone);
    }
    return dto;
}
```

## 5。前端

接下来，我们将修改我们的前端`scheduledPosts.jsp`来处理我们的新响应:

```java
<div class="modal">
    <h4 class="modal-title">Detailed Status</h4>
    <table id="res"></table>
</div>

<script >
var loadedData = [];
var detailedResTable = $('#res').DataTable( {
    "searching":false,
    "paging": false,
    columns: [
        { title: "Attempt Number", data: "attemptNumber" },
        { title: "Detailed Status", data: "content" },
        { title: "Attempt Submitted At", data: "localSubmissionDate" },
        { title: "Attempt Score Checked At", data: "localScoreCheckDate" }
 ]
} );

$(document).ready(function() {
    $('#myposts').dataTable( {
        ...
        "columnDefs": [
            { "targets": 2, "data": "status",
              "render": function ( data, type, full, meta ) {
                  return data + 
                    ' <a href="#" onclick="showDetailedStatus('+meta.row+' )">More Details</a>';
              }
            },
            ....
        ],
        ...
    });
});

function showDetailedStatus(row){
    detailedResTable.clear().rows.add(loadedData[row].detailedStatus).draw();
    $('.modal').modal();
}

</script>
```

## 6。测试

最后，我们将对我们的新方法执行一个简单的单元测试:

首先，我们将测试`getSuccessReason()` 的实现:

```java
@Test
public void whenHasEnoughScore_thenSucceed() {
    Post post = new Post();
    post.setMinScoreRequired(5);
    PostScores postScores = new PostScores(6, 10, 1);

    assertTrue(getSuccessReason(post, postScores).contains("Succeed because score"));
}

@Test
public void whenHasEnoughTotalVotes_thenSucceed() {
    Post post = new Post();
    post.setMinScoreRequired(5);
    post.setMinTotalVotes(8);
    PostScores postScores = new PostScores(2, 10, 1);

    assertTrue(getSuccessReason(post, postScores).contains("Succeed because total votes"));
}

@Test
public void givenKeepPostIfHasComments_whenHasComments_thenSucceed() {
    Post post = new Post();
    post.setMinScoreRequired(5);
    post.setKeepIfHasComments(true);
    final PostScores postScores = new PostScores(2, 10, 1);

    assertTrue(getSuccessReason(post, postScores).contains("Succeed because has comments"));
}
```

接下来，我们将测试`getFailReason()`的实现:

```java
@Test
public void whenNotEnoughScore_thenFail() {
    Post post = new Post();
    post.setMinScoreRequired(5);
    PostScores postScores = new PostScores(2, 10, 1);

    assertTrue(getFailReason(post, postScores).contains("Failed because score"));
}

@Test
public void whenNotEnoughTotalVotes_thenFail() {
    Post post = new Post();
    post.setMinScoreRequired(5);
    post.setMinTotalVotes(15);
    PostScores postScores = new PostScores(2, 10, 1);

    String reason = getFailReason(post, postScores);
    assertTrue(reason.contains("Failed because score"));
    assertTrue(reason.contains("and total votes"));
}

@Test
public void givenKeepPostIfHasComments_whenNotHasComments_thenFail() {
    Post post = new Post();
    post.setMinScoreRequired(5);
    post.setKeepIfHasComments(true);
    final PostScores postScores = new PostScores(2, 10, 0);

    String reason = getFailReason(post, postScores);
    assertTrue(reason.contains("Failed because score"));
    assertTrue(reason.contains("and has no comments"));
}
```

## 7。结论

在这一期中，我们介绍了 Reddit 帖子生命周期中一些非常有用的可见性。我们现在可以看到每次提交和删除帖子的确切时间，以及每次操作的确切原因。