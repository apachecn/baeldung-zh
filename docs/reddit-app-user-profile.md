# Reddit 应用程序中的用户资料

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reddit-app-user-profile>

## 1。概述

在这篇文章中，我们将为我们的 Reddit 应用程序[的用户建立**档案，允许他们配置用户特定的偏好。**](/web/20221208143841/https://www.baeldung.com/case-study-a-reddit-app-with-spring)

目标很简单——不用让用户每次安排新帖子时都填写相同的数据，**他们可以设置一次——在他们的个人资料**的首选项中。当然，用户可以随时为每个帖子调整这些设置，但他们不必这样做。

## 2。`Preference`实体

总的来说，现在可以在应用程序中配置的大多数东西都将成为可以在用户配置文件中全局配置的**。**

首先，让我们从一个`Preference`实体开始:

```java
@Entity
public class Preference {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String email;

    private String subreddit;

    private boolean sendReplies;

    // for post re-submission
    private int noOfAttempts;
    private int timeInterval;
    private int minScoreRequired;
    private int minUpvoteRatio;
    private boolean keepIfHasComments;
    private boolean deleteAfterLastAttempt;
}
```

那么，我们现在可以配置什么呢？简而言之，应用程序中的几乎所有设置都默认使用—**。**

我们还存储了用户的电子邮件，让他们能够收到关于他们的帖子发生了什么的通知。

现在，让我们**将偏好链接到用户**:

```java
@Entity
public class User {
    ...

    @OneToOne
    @JoinColumn(name = "preference_id")
    private Preference preference;
}
```

如您所见，我们在`User`和`Preference.` 之间有一个简单的一对一关系

## 3。简单个人资料页面

首先，让我们创建一个简单的个人资料页面:

```java
<form >
    <input type="hidden" name="id" />
    <input name="email" type="email"/>
    <input name="subreddit"/>
    ...
   <button onclick="editPref()" >Save Changes</button>
</form>
<script>
$(function() {
    $.get("user/preference", function (data){
        $.each(data, function(key, value) {
            $('*[name="'+key+'"]').val(value);
        });
    });
});
function editPref(){
    var data = {};
	$('form').serializeArray().map(function(x){data[x.name] = x.value;});
	$.ajax({
        url: "user/preference/"+$('input[name="id"]').val(),
        data: JSON.stringify(data),
        type: 'PUT',
        contentType:'application/json'
    }).done(function() { window.location.href = "./"; })
      .fail(function(error) { alert(error.responseText); }); 
}
</script>
```

这里没有什么新奇的东西，只有一些普通的 HTML 和 JavaScript。

让我们也添加一个快速链接到新的配置文件:

```java
<h1>Welcome, <a href="profile" sec:authentication="principal.username">username</a></h1>
```

## 4。API

这里是控制器，用于创建和编辑用户的偏好:

```java
@Controller
@RequestMapping(value = "/user/preference")
public class UserPreferenceController {

    @Autowired
    private PreferenceRepository preferenceReopsitory;

    @RequestMapping(method = RequestMethod.GET)
    @ResponseBody
    public Preference getCurrentUserPreference() {
        return getCurrentUser().getPreference();
    }

    @RequestMapping(value = "/{id}", method = RequestMethod.PUT)
    @ResponseStatus(HttpStatus.OK)
    public void updateUserPreference(@RequestBody Preference pref) {
        preferenceReopsitory.save(pref);
        getCurrentUser().setPreference(pref);
    }
}
```

最后，我们需要确保在创建用户时，其首选项也是未初始化的:

```java
public void loadAuthentication(String name, OAuth2AccessToken token) {
    ...
    Preference pref = new Preference();
    preferenceReopsitory.save(pref);
    user.setPreference(pref);
    userReopsitory.save(user);
    ...   
}
```

## 5。加载/使用首选项

现在，让我们看看如何使用这些首选项，并在需要时填充它们。

我们将从主`Post Schedule`页面开始，在这里我们将加载用户的偏好:

```java
$(function() {
    $.get("user/preference", function (data){
        $.each(data, function(key, value) {
            $('*[name="'+key+'"]').val(value);
        });
    });
});
```

## 6。测试和结论

我们差不多完成了——我们只需要为我们刚刚介绍的新概要文件实体实现一些基本的集成测试。

在大多数情况下，我们只是简单地扩展现有的基础持久性测试，并从那里继承一组测试。

最后，我们可以总结一下新的用户配置文件功能，用户现在可以设置自己的首选项。