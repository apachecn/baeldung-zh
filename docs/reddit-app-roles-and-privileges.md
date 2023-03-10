# 向 Reddit 应用程序添加角色和权限

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reddit-app-roles-and-privileges>

## 1。概述

在这一期中，我们将在我们的 Reddit 应用程序中引入简单的角色和权限，然后能够做一些有趣的事情，例如——限制普通用户每天可以在 Reddit 上发布多少帖子。

由于我们将有一个管理员角色，也就是管理员用户，我们还将添加一个管理员管理区域。

## 2。`User`、`Role`和`Privilege`实体

首先，我们将修改`User`实体——我们通过我们的 [Reddit 应用程序系列](/web/20210917093839/https://www.baeldung.com/case-study-a-reddit-app-with-spring)使用它——来添加角色:

```java
@Entity
public class User {
    ...

    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(name = "users_roles", 
      joinColumns = @JoinColumn(name = "user_id", referencedColumnName = "id"), 
      inverseJoinColumns = @JoinColumn(name = "role_id", referencedColumnName = "id"))
    private Collection<Role> roles;

    ...
}
```

请注意用户-角色关系是一种灵活的多对多关系。

接下来，我们将定义`Role`和`Privilege`实体。有关该实现的全部细节，请查看[关于 Baeldung](/web/20210917093839/https://www.baeldung.com/role-and-privilege-for-spring-security-registration) 的这篇文章。

## 3。设置

接下来，我们将在项目引导上运行一些基本设置，以创建这些角色和权限:

```java
private void createRoles() {
    Privilege adminReadPrivilege = createPrivilegeIfNotFound("ADMIN_READ_PRIVILEGE");
    Privilege adminWritePrivilege = createPrivilegeIfNotFound("ADMIN_WRITE_PRIVILEGE");
    Privilege postLimitedPrivilege = createPrivilegeIfNotFound("POST_LIMITED_PRIVILEGE");
    Privilege postUnlimitedPrivilege = createPrivilegeIfNotFound("POST_UNLIMITED_PRIVILEGE");

    createRoleIfNotFound("ROLE_ADMIN", Arrays.asList(adminReadPrivilege, adminWritePrivilege));
    createRoleIfNotFound("ROLE_SUPER_USER", Arrays.asList(postUnlimitedPrivilege));
    createRoleIfNotFound("ROLE_USER", Arrays.asList(postLimitedPrivilege));
}
```

并让我们的测试用户成为管理员:

```java
private void createTestUser() {
    Role adminRole = roleRepository.findByName("ROLE_ADMIN");
    Role superUserRole = roleRepository.findByName("ROLE_SUPER_USER");
    ...
    userJohn.setRoles(Arrays.asList(adminRole, superUserRole));
}
```

## 4。注册标准用户

我们还需要确保我们通过`registerNewUser()`实现注册了标准用户:

```java
@Override
public void registerNewUser(String username, String email, String password) {
    ...
    Role role = roleRepository.findByName("ROLE_USER");
    user.setRoles(Arrays.asList(role));
}
```

请注意，系统中的角色包括:

1.  对于普通用户(默认角色)——他们有一天可以安排多少帖子的限制
2.  `ROLE_SUPER_USER`:无调度限制
3.  `ROLE_ADMIN`:附加管理选项

## 5。本金

接下来，让我们将这些新特权集成到我们的主要实现中:

```java
public class UserPrincipal implements UserDetails {
    ...

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        List<GrantedAuthority> authorities = new ArrayList<GrantedAuthority>();
        for (Role role : user.getRoles()) {
            for (Privilege privilege : role.getPrivileges()) {
                authorities.add(new SimpleGrantedAuthority(privilege.getName()));
            }
        }
        return authorities;
    }
}
```

## 6。限制标准用户的预定帖子

现在，让我们利用新的角色和权限，**限制标准用户每天安排的新文章不超过 3 篇**，以避免向 Reddit 发送垃圾邮件。

### 6.1。发布存储库

首先，我们将在我们的`PostRepository`实现中添加一个新操作——统计特定用户在特定时间段内的预定帖子:

```java
public interface PostRepository extends JpaRepository<Post, Long> {
    ...

    Long countByUserAndSubmissionDateBetween(User user, Date start, Date end);

}
```

### 5.2。预定后控制器

然后，我们将向`schedule()`和`updatePost()`方法添加一个简单的检查:

```java
public class ScheduledPostRestController {
    private static final int LIMIT_SCHEDULED_POSTS_PER_DAY = 3;

    public Post schedule(HttpServletRequest request,...) throws ParseException {
        ...
        if (!checkIfCanSchedule(submissionDate, request)) {
            throw new InvalidDateException("Scheduling Date exceeds daily limit");
        }
        ...
    }

    private boolean checkIfCanSchedule(Date date, HttpServletRequest request) {
        if (request.isUserInRole("POST_UNLIMITED_PRIVILEGE")) {
            return true;
        }
        Date start = DateUtils.truncate(date, Calendar.DATE);
        Date end = DateUtils.addDays(start, 1);
        long count = postReopsitory.
          countByUserAndSubmissionDateBetween(getCurrentUser(), start, end);
        return count < LIMIT_SCHEDULED_POSTS_PER_DAY;
    }
}
```

这里发生了一些有趣的事情。首先——注意我们如何手动与 Spring Security 交互，并检查当前登录的用户是否有特权。这不是你每天都要做的事情，但是当你不得不做的时候，这个 API 非常有用。

按照目前的逻辑——如果用户有`POST_UNLIMITED_PRIVILEGE`——他们能够——惊喜地——安排他们想安排的时间。

然而，如果他们没有这种特权，他们每天最多只能排 3 个帖子。

## 7。管理员用户页面

接下来——现在我们已经根据用户的角色对他们进行了明确的划分——让我们为小型 Reddit 应用的管理员实现一些非常简单的用户管理。

### 7.1。显示所有用户

首先，让我们创建一个列出系统中所有用户的基本页面:

这里列出所有用户的 API:

```java
@PreAuthorize("hasRole('ADMIN_READ_PRIVILEGE')")
@RequestMapping(value="/admin/users", method = RequestMethod.GET)
@ResponseBody
public List<User> getUsersList() {
    return service.getUsersList();
}
```

和服务层实现:

```java
@Transactional
public List<User> getUsersList() {
    return userRepository.findAll();
}
```

然后，简单的前端:

```java
<table>
    <thead>
        <tr>
            <th>Username</th>
            <th>Roles</th>
            <th>Actions</th></tr>
    </thead>
</table>

<script>
$(function(){
    var userRoles="";
    $.get("admin/users", function(data){
        $.each(data, function( index, user ) {
            userRoles = extractRolesName(user.roles);
            $('.table').append('<tr><td>'+user.username+'</td><td>'+
              userRoles+'</td><td><a href="#" onclick="showEditModal('+
              user.id+',\''+userRoles+'\')">Modify User Roles</a></td></tr>');
        });
    });
});

function extractRolesName(roles){ 
    var result =""; 
    $.each(roles, function( index, role ) { 
        result+= role.name+" "; 
    }); 
    return result; 
}
</script>
```

### 7.2。修改用户的角色

接下来，一些简单的逻辑来管理这些用户的角色；让我们从控制器开始:

```java
@PreAuthorize("hasRole('USER_WRITE_PRIVILEGE')")
@RequestMapping(value = "/user/{id}", method = RequestMethod.PUT)
@ResponseStatus(HttpStatus.OK)
public void modifyUserRoles(
  @PathVariable("id") Long id, 
  @RequestParam(value = "roleIds") String roleIds) {
    service.modifyUserRoles(id, roleIds);
}

@PreAuthorize("hasRole('USER_READ_PRIVILEGE')")
@RequestMapping(value = "/admin/roles", method = RequestMethod.GET)
@ResponseBody
public List<Role> getRolesList() {
    return service.getRolesList();
}
```

服务层:

```java
@Transactional
public List<Role> getRolesList() {
    return roleRepository.findAll();
}
@Transactional
public void modifyUserRoles(Long userId, String ids) {
    List<Long> roleIds = new ArrayList<Long>();
    String[] arr = ids.split(",");
    for (String str : arr) {
        roleIds.add(Long.parseLong(str));
    }
    List<Role> roles = roleRepository.findAll(roleIds);
    User user = userRepository.findOne(userId);
    user.setRoles(roles);
    userRepository.save(user);
}
```

最后，简单的前端:

```java
<div id="myModal">
    <h4 class="modal-title">Modify User Roles</h4>
    <input type="hidden" name="id" id="userId"/>
    <div id="allRoles"></div>
    <button onclick="modifyUserRoles()">Save changes</button>
</div>

<script>
function showEditModal(userId, roleNames){
    $("#userId").val(userId);
    $.get("admin/roles", function(data){
        $.each(data, function( index, role ) {
            if(roleNames.indexOf(role.name) != -1){
                $('#allRoles').append(
                  '<input type="checkbox" name="roleIds" value="'+role.id+'" checked/> '+role.name+'<br/>')
            } else{
                $('#allRoles').append(
                  '<input type="checkbox" name="roleIds" value="'+role.id+'" /> '+role.name+'<br/>')
            }
       });
       $("#myModal").modal();
    });
}

function modifyUserRoles(){
    var roles = [];
    $.each($("input[name='roleIds']:checked"), function(){ 
        roles.push($(this).val());
    }); 
    if(roles.length == 0){
        alert("Error, at least select one role");
        return;
    }

    $.ajax({
        url: "user/"+$("#userId").val()+"?roleIds="+roles.join(","),
        type: 'PUT',
        contentType:'application/json'
        }).done(function() { window.location.href="users";
        }).fail(function(error) { alert(error.responseText); 
    }); 
}
</script>
```

## 8。安全配置

最后，我们需要修改安全配置，将管理员用户重定向到系统中这个新的独立页面:

```java
@Autowired 
private AuthenticationSuccessHandler successHandler;

@Override
protected void configure(HttpSecurity http) throws Exception {
    http.
    ...
    .authorizeRequests()
    .antMatchers("/adminHome","/users").hasAuthority("ADMIN_READ_PRIVILEGE")    
    ...
    .formLogin().successHandler(successHandler)
}
```

我们使用一个定制的身份验证成功处理程序来**决定用户登录**后的位置:

```java
@Component
public class MyAuthenticationSuccessHandler implements AuthenticationSuccessHandler {

    @Override
    public void onAuthenticationSuccess(
      HttpServletRequest request, HttpServletResponse response, Authentication auth) 
      throws IOException, ServletException {
        Set<String> privieleges = AuthorityUtils.authorityListToSet(auth.getAuthorities());
        if (privieleges.contains("ADMIN_READ_PRIVILEGE")) {
            response.sendRedirect("adminHome");
        } else {
            response.sendRedirect("home");
        }
    }
}
```

以及极其简单的管理主页`adminHome.html`:

```java
<html>
<body>
    <h1>Welcome, <small><span sec:authentication="principal.username">Bob</span></small></h1>
    <br/>
    <a href="users">Display Users List</a>
</body>
</html>
```

## 9。结论

在案例研究的这个新部分中，我们在应用程序中添加了一些简单的安全构件——角色和权限。在这种支持下，**我们构建了两个简单的特性**——标准用户的调度限制和管理用户的基本管理。