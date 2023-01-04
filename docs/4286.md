# 以用户的时区显示日期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reddit-app-show-date-in-the-users-timezone>

## 1。概述

在本期的[Reddit 应用案例研究](/web/20210918113153/https://www.baeldung.com/case-study-a-reddit-app-with-spring)中，我们将根据用户的时区添加 be **调度帖子。**

众所周知，处理时区问题非常困难，而且技术选项也非常开放。我们首先关注的是，我们需要根据用户自己的(可配置的)时区向用户显示日期。我们还需要决定**在数据库**中，日期将以什么格式保存。

## 2。一个新的用户偏好——`timezone`

首先，我们将在现有的首选项中添加一个新字段–`timezone`:

```
@Entity
public class Preference {
    ...
    private String timezone;
}
```

然后我们简单地**在用户首选项页面**中配置时区——利用一个简单但非常有用的 JQuery [插件](https://web.archive.org/web/20210918113153/http://www.jqueryscript.net/time-clock/Easy-Timezone-Picker-with-jQuery-Moment-js-Timezones.html):

```
<select id="timezone" name="timezone"></select>
<script>
    $(function() {
        $('#timezone').timezones();
    });
</script>
```

请注意，默认时区是服务器时区-**，它在`UTC`运行。**

## 3。控制器

现在，有趣的是。我们需要将日期从**用户的时区**转换为**服务器的时区**:

```
@Controller
@RequestMapping(value = "/api/scheduledPosts")
public class ScheduledPostRestController {
    private static final SimpleDateFormat dateFormat = 
      new SimpleDateFormat("yyyy-MM-dd HH:mm");

    @RequestMapping(method = RequestMethod.POST)
    @ResponseStatus(HttpStatus.OK)
    public void schedule(
      @RequestBody Post post, 
      @RequestParam(value = "date") String date) throws ParseException 
    {
        post.setSubmissionDate(
          calculateSubmissionDate(date, getCurrentUser().getPreference().getTimezone()));
        ...
    }

    @RequestMapping(value = "/{id}", method = RequestMethod.PUT)
    @ResponseStatus(HttpStatus.OK)
    public void updatePost(
      @RequestBody Post post, 
      @RequestParam(value = "date") String date) throws ParseException 
    {
        post.setSubmissionDate(
          calculateSubmissionDate(date, getCurrentUser().getPreference().getTimezone()));
        ...
    }

    private synchronized Date calculateSubmissionDate(String dateString, String userTimeZone) 
      throws ParseException {
        dateFormat.setTimeZone(TimeZone.getTimeZone(userTimeZone));
        return dateFormat.parse(dateString);
    }
}
```

这种转换非常简单，但是请注意，它只发生在写操作上——对于读操作，服务器仍然返回 UTC。

这对我们的客户端来说完全没问题，因为我们将在 JS 中进行转换——但是值得理解的是，对于读取操作，**服务器仍然返回 UTC 日期**。

## 4。前端

现在，让我们看看如何在前端使用用户的时区:

### 4.1。显示帖子

我们需要使用用户的时区显示文章的`submissionDate` :

```
<table><thead><tr>
<th>Post title</th>
<th>Submission Date 
  (<span id="timezone" sec:authentication="principal.preference.timezone">UTC</span>)</th>
</tr></thead></table>
```

这是我们的函数:

```
function loadPage(page){
    ...
    $('.table').append('<tr><td>'+post.title+'</td><td>'+
      convertDate(post.submissionDate)+'</td></tr>');
    ...
}
function convertDate(date){
    var serverTimezone = [[${#dates.format(#calendars.createToday(), 'z')}]];
    var serverDate = moment.tz(date, serverTimezone);
    var clientDate = serverDate.clone().tz($("#timezone").html());
    var myformat = "YYYY-MM-DD HH:mm";
    return clientDate.format(myformat);
}
```

在这里，Moment.js 帮助进行时区转换。

### 4.2。安排一个新帖子

我们还需要修改我们的`schedulePostForm.html`:

```
Submission Date (<span sec:authentication="principal.preference.timezone">UTC</span>)
<input id="date" name="date" />

<script type="text/javascript">
function schedulePost(){
    var data = {};
    $('form').serializeArray().map(function(x){data[x.name] = x.value;});
    $.ajax({
        url: 'api/scheduledPosts?date='+$("#date").val(),
        data: JSON.stringify(data),
        type: 'POST',
        contentType:'application/json',
        success: function(result) {
            window.location.href="scheduledPosts";
        },
        error: function(error) {
            alert(error.responseText);
        }   
    }); 
}
</script>
```

最后，我们还需要修改我们的`editPostForm.html`来本地化旧的`submissonDate`值:

```
$(function() {
    var serverTimezone = [[${#dates.format(#calendars.createToday(), 'z')}]];
    var serverDate = moment.tz($("#date").val(), serverTimezone);
    var clientDate = serverDate.clone().tz($("#timezone").html());
    var myformat = "YYYY-MM-DD HH:mm";
    $("#date").val(clientDate.format(myformat));
});
```

## 5。结论

在这篇简单的文章中，我们向 Reddit 应用程序引入了一个简单但非常有用的功能——根据您自己的时区查看所有内容的能力。

这是我在使用应用程序时的主要痛点之一——一切都在 UTC。now–所有日期都按照用户的时区正确显示，就像它们应该的那样。