# Spring 安全 ACL 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-acl>

## 1。简介

`Access Control List`(*)ACL)*是附加到对象的权限列表。一个`ACL`指定哪些身份被授予对给定对象的哪些操作。

*Spring Security `Access Control List`* 是**一个支持`Domain Object Security.`** 的`Spring`组件简单来说，Spring ACL 有助于在单个域对象上为特定用户/角色定义权限——而不是在典型的每个操作级别上全面定义。

例如，角色为`Admin`的用户可以查看(`READ)`)和编辑(`WRITE)`)一个`Central Notice Box`上的所有消息，但是普通用户只能查看消息，与消息相关，不能编辑。同时，其他角色为`Editor`的用户可以查看和编辑一些特定的消息。

因此，不同的用户/角色对每个特定对象具有不同的权限。在这种情况下，`Spring ACL` 有能力完成任务。在本文中，我们将探索如何用`Spring ACL` 设置基本的权限检查。

## 2。配置

### 2.1。ACL 数据库

为了使用`Spring Security ACL`，我们需要在数据库中创建四个强制表。

第一个表是`ACL_CLASS`，其中存储了域对象的类名，列包括:

*   `ID`
*   *类:*安全域对象的类名，例如: *`com.baeldung.acl.persistence.entity.NoticeMessage`*

其次，我们需要`ACL_SID`表，它允许我们普遍地识别系统中的任何原则或权威。桌子需要:

*   `ID`
*   `SID:` 用户名或角色名。`SID` 代表`Security Identity`
*   `PRINCIPAL:` `0`或`1`，表示对应的`SID`是委托人(用户，如`mary, mike, jack…`)或权限(角色，如`ROLE_ADMIN, ROLE_USER, ROLE_EDITOR…`)

下一个表是`ACL_OBJECT_IDENTITY,` ，它存储每个唯一域对象的信息:

*   `ID`
*   `OBJECT_ID_CLASS:`定义域对象类，链接到 `ACL_CLASS` 表
*   根据类别的不同，域对象可以存储在许多表中。因此，该字段存储目标对象的主键
*   `PARENT_OBJECT:` 在此表中指定此`Object Identity`的父级
*   `OWNER_SID:` `ID`的对象所有者，链接到`ACL_SID` 表
*   `ENTRIES_INHERITING:` 该对象的`ACL Entries`是否继承自父对象(`ACL_ENTRY`表中定义了`ACL Entries`

最后，`ACL_ENTRY` 商店将个人权限分配给`Object Identity`上的每个`SID`:

*   `ID`
*   `ACL_OBJECT_IDENTITY:`指定对象标识，链接到`ACL_OBJECT_IDENTITY`表
*   `ACE_ORDER:`对应`Object Identity`的`ACL entries`列表中当前条目的顺序
*   `SID:`被授予或拒绝权限的目标`SID`链接到`ACL_SID` 表
*   `MASK:`表示被授予或拒绝的实际权限的整数位掩码
*   `GRANTING:` 值 1 表示同意，值`0`表示拒绝
*   `AUDIT_SUCCESS` 和`AUDIT_FAILURE`:审计用

### 2.2。依赖性

为了能够在我们的项目中使用`Spring ACL` ,让我们首先定义我们的依赖关系:

```java
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-acl</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
</dependency>
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache-core</artifactId>
    <version>2.6.11</version>
</dependency>
```

`Spring ACL`需要一个缓存来存储`Object Identity`和`ACL entries`，所以我们将在这里使用`Ehcache` 。为了支持`Spring,` 中的`Ehcache`，我们还需要`spring-context-support.`

当不使用 Spring Boot 时，我们需要显式地添加版本。那些可以在 Maven Central 上查: [spring-security-acl](https://web.archive.org/web/20221208143837/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.security%22%20AND%20a%3A%22spring-security-acl%22) ， [spring-security-config](https://web.archive.org/web/20221208143837/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.security%22%20AND%20a%3A%22spring-security-config%22) ， [spring-context-support](https://web.archive.org/web/20221208143837/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-context-support%22) ， [ehcache-core](https://web.archive.org/web/20221208143837/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22net.sf.ehcache.internal%22%20AND%20a%3A%22ehcache-core%22) 。

### 2.3。ACL 相关配置

我们需要通过启用`Global Method Security:`来保护所有返回安全域对象或对对象进行更改的方法

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class AclMethodSecurityConfiguration 
  extends GlobalMethodSecurityConfiguration {

    @Autowired
    MethodSecurityExpressionHandler 
      defaultMethodSecurityExpressionHandler;

    @Override
    protected MethodSecurityExpressionHandler createExpressionHandler() {
        return defaultMethodSecurityExpressionHandler;
    }
}
```

让我们通过将 `prePostEnabled` 设置为 `true`来启用`Expression-Based Access Control` 以使用`Spring Expression Language (SpEL)` `.` 此外`,`我们需要一个支持`ACL` 的表达式处理程序:

```java
@Bean
public MethodSecurityExpressionHandler 
  defaultMethodSecurityExpressionHandler() {
    DefaultMethodSecurityExpressionHandler expressionHandler
      = new DefaultMethodSecurityExpressionHandler();
    AclPermissionEvaluator permissionEvaluator 
      = new AclPermissionEvaluator(aclService());
    expressionHandler.setPermissionEvaluator(permissionEvaluator);
    return expressionHandler;
}
```

因此`,` 我们将`AclPermissionEvaluator`分配给`DefaultMethodSecurityExpressionHandler`。评估器需要一个`MutableAclService`从数据库中加载权限设置和域对象的定义。

为简单起见，我们使用提供的`JdbcMutableAclService`:

```java
@Bean 
public JdbcMutableAclService aclService() { 
    return new JdbcMutableAclService(
      dataSource, lookupStrategy(), aclCache()); 
}
```

顾名思义，`JdbcMutableAclService` 使用`JDBCTemplate` 来简化数据库访问。它需要一个*数据源(*用于`JDBCTemplate)`，`LookupStrategy` (在查询数据库时提供优化的查找)，以及一个`AclCache (`缓存`ACL` `Entries`和`Object Identity)`。

同样，为了简单起见，我们使用提供的`BasicLookupStrategy`和`EhCacheBasedAclCache`。

```java
@Autowired
DataSource dataSource;

@Bean
public AclAuthorizationStrategy aclAuthorizationStrategy() {
    return new AclAuthorizationStrategyImpl(
      new SimpleGrantedAuthority("ROLE_ADMIN"));
}

@Bean
public PermissionGrantingStrategy permissionGrantingStrategy() {
    return new DefaultPermissionGrantingStrategy(
      new ConsoleAuditLogger());
}

@Bean
public EhCacheBasedAclCache aclCache() {
    return new EhCacheBasedAclCache(
      aclEhCacheFactoryBean().getObject(), 
      permissionGrantingStrategy(), 
      aclAuthorizationStrategy()
    );
}

@Bean
public EhCacheFactoryBean aclEhCacheFactoryBean() {
    EhCacheFactoryBean ehCacheFactoryBean = new EhCacheFactoryBean();
    ehCacheFactoryBean.setCacheManager(aclCacheManager().getObject());
    ehCacheFactoryBean.setCacheName("aclCache");
    return ehCacheFactoryBean;
}

@Bean
public EhCacheManagerFactoryBean aclCacheManager() {
    return new EhCacheManagerFactoryBean();
}

@Bean 
public LookupStrategy lookupStrategy() { 
    return new BasicLookupStrategy(
      dataSource, 
      aclCache(), 
      aclAuthorizationStrategy(), 
      new ConsoleAuditLogger()
    ); 
} 
```

这里，`AclAuthorizationStrategy` 负责判断当前用户是否拥有对某些对象的所有权限。

它需要`PermissionGrantingStrategy,` 的支持，T0 定义了确定是否授予特定`SID`权限的逻辑。

## 3。使用 Spring ACL 的方法安全性

到目前为止，我们已经完成了所有必要的配置`.` ,现在我们可以在我们的安全方法上放置所需的检查规则。

默认情况下，`Spring ACL` 引用所有可用权限的`BasePermission`类。基本上，我们有`READ, WRITE, CREATE, DELETE` 和`ADMINISTRATION` 权限。

让我们尝试定义一些安全规则:

```java
@PostFilter("hasPermission(filterObject, 'READ')")
List<NoticeMessage> findAll();

@PostAuthorize("hasPermission(returnObject, 'READ')")
NoticeMessage findById(Integer id);

@PreAuthorize("hasPermission(#noticeMessage, 'WRITE')")
NoticeMessage save(@Param("noticeMessage")NoticeMessage noticeMessage);
```

执行`findAll()`方法后，会触发`@PostFilter`。所需规则`hasPermission(filterObject, ‘READ'),` 意味着只返回当前用户拥有`READ` 权限的那些`NoticeMessage` 。

同样，`@PostAuthorize` 在`findById()`方法执行后被触发，确保只有当前用户拥有`READ`权限时才返回`NoticeMessage`对象。如果没有，系统会抛出一个`AccessDeniedException`。

另一方面，系统在调用`save()`方法之前触发`@PreAuthorize`注释。它将决定相应的方法是否允许在哪里执行。如果没有，`AccessDeniedException` 就会被抛出。

## 4。行动中

现在我们将使用`JUnit`测试所有这些配置。我们将使用`H2` 数据库来尽可能简化配置。

我们需要添加:

```java
<dependency>
  <groupId>com.h2database</groupId>
  <artifactId>h2</artifactId>
</dependency>

<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-test</artifactId>
  <scope>test</scope>
</dependency>

<dependency>
  <groupId>org.springframework.security</groupId>
  <artifactId>spring-security-test</artifactId>
  <scope>test</scope>
</dependency>
```

### 4.1。场景

在这个场景中，我们将有两个用户(`manager, hr)`和一个用户角色(`ROLE_EDITOR),` )，因此我们的`acl_sid`将是:

```java
INSERT INTO acl_sid (id, principal, sid) VALUES
  (1, 1, 'manager'),
  (2, 1, 'hr'),
  (3, 0, 'ROLE_EDITOR');
```

然后，我们需要在`acl_class`中声明`NoticeMessage`类。并且三个`NoticeMessage`类的实例将被插入到`system_message.` 中

此外，这 3 个实例的相应记录必须在`acl_object_identity`中声明:

```java
INSERT INTO acl_class (id, class) VALUES
  (1, 'com.baeldung.acl.persistence.entity.NoticeMessage');

INSERT INTO system_message(id,content) VALUES 
  (1,'First Level Message'),
  (2,'Second Level Message'),
  (3,'Third Level Message');

INSERT INTO acl_object_identity 
  (id, object_id_class, object_id_identity, 
  parent_object, owner_sid, entries_inheriting) 
  VALUES
  (1, 1, 1, NULL, 3, 0),
  (2, 1, 2, NULL, 3, 0),
  (3, 1, 3, NULL, 3, 0);
```

最初，我们将第一个对象(`id =1`)的`READ` 和`WRITE` 权限授予用户`manager`。同时，任何拥有`ROLE_EDITOR`的用户将拥有所有三个对象的`READ`权限，但只拥有第三个对象(`id=3`)的`WRITE`权限。此外，用户`hr`对第二个对象只有`READ`权限。

这里，因为我们使用默认的`Spring ACL` `BasePermission` 类进行权限检查， `READ` 权限的掩码值将为 1， `WRITE` 权限的掩码值将为 2。我们在`acl_entry`的数据将是:

```java
INSERT INTO acl_entry 
  (id, acl_object_identity, ace_order, 
  sid, mask, granting, audit_success, audit_failure) 
  VALUES
  (1, 1, 1, 1, 1, 1, 1, 1),
  (2, 1, 2, 1, 2, 1, 1, 1),
  (3, 1, 3, 3, 1, 1, 1, 1),
  (4, 2, 1, 2, 1, 1, 1, 1),
  (5, 2, 2, 3, 1, 1, 1, 1),
  (6, 3, 1, 3, 1, 1, 1, 1),
  (7, 3, 2, 3, 2, 1, 1, 1);
```

### 4.2。测试用例

首先，我们尝试调用`findAll`方法。

按照我们的配置，该方法只返回用户拥有`READ`权限的那些`NoticeMessage`。

因此，我们希望结果列表只包含第一条消息:

```java
@Test
@WithMockUser(username = "manager")
public void 
  givenUserManager_whenFindAllMessage_thenReturnFirstMessage(){
    List<NoticeMessage> details = repo.findAll();

    assertNotNull(details);
    assertEquals(1,details.size());
    assertEquals(FIRST_MESSAGE_ID,details.get(0).getId());
}
```

然后，我们尝试对角色为`ROLE_EDITOR`的任何用户调用相同的方法。注意，在这种情况下，这些用户对所有三个对象都有`READ`权限。

因此，我们期望结果列表将包含所有三条消息:

```java
@Test
@WithMockUser(roles = {"EDITOR"})
public void 
  givenRoleEditor_whenFindAllMessage_thenReturn3Message(){
    List<NoticeMessage> details = repo.findAll();

    assertNotNull(details);
    assertEquals(3,details.size());
}
```

接下来，使用`manager`用户，我们将尝试通过 id 获取第一条消息，并更新其内容——这一切都应该工作正常:

```java
@Test
@WithMockUser(username = "manager")
public void 
  givenUserManager_whenFind1stMessageByIdAndUpdateItsContent_thenOK(){
    NoticeMessage firstMessage = repo.findById(FIRST_MESSAGE_ID);
    assertNotNull(firstMessage);
    assertEquals(FIRST_MESSAGE_ID,firstMessage.getId());

    firstMessage.setContent(EDITTED_CONTENT);
    repo.save(firstMessage);

    NoticeMessage editedFirstMessage = repo.findById(FIRST_MESSAGE_ID);

    assertNotNull(editedFirstMessage);
    assertEquals(FIRST_MESSAGE_ID,editedFirstMessage.getId());
    assertEquals(EDITTED_CONTENT,editedFirstMessage.getContent());
}
```

但是，如果任何具有`ROLE_EDITOR`角色的用户更新了第一条消息的内容，我们的系统将抛出一个`AccessDeniedException`:

```java
@Test(expected = AccessDeniedException.class)
@WithMockUser(roles = {"EDITOR"})
public void 
  givenRoleEditor_whenFind1stMessageByIdAndUpdateContent_thenFail(){
    NoticeMessage firstMessage = repo.findById(FIRST_MESSAGE_ID);

    assertNotNull(firstMessage);
    assertEquals(FIRST_MESSAGE_ID,firstMessage.getId());

    firstMessage.setContent(EDITTED_CONTENT);
    repo.save(firstMessage);
}
```

类似地，`hr`用户可以通过 id 找到第二条消息，但是无法更新它:

```java
@Test
@WithMockUser(username = "hr")
public void givenUsernameHr_whenFindMessageById2_thenOK(){
    NoticeMessage secondMessage = repo.findById(SECOND_MESSAGE_ID);
    assertNotNull(secondMessage);
    assertEquals(SECOND_MESSAGE_ID,secondMessage.getId());
}

@Test(expected = AccessDeniedException.class)
@WithMockUser(username = "hr")
public void givenUsernameHr_whenUpdateMessageWithId2_thenFail(){
    NoticeMessage secondMessage = new NoticeMessage();
    secondMessage.setId(SECOND_MESSAGE_ID);
    secondMessage.setContent(EDITTED_CONTENT);
    repo.save(secondMessage);
}
```

## 5。结论

在本文中，我们已经介绍了`Spring ACL`的基本配置和用法。

我们知道，`Spring ACL`需要具体的表格来管理对象、原则/权限和权限设置。所有与这些表的交互，尤其是更新操作，都必须通过`AclService.`完成。我们将在以后的文章中探讨这个服务的基本`CRUD` 操作。

默认情况下，我们被限制在`BasePermissio` n 类中的预定义权限。

最后，本教程的实现可以在 Github 上找到[。](https://web.archive.org/web/20221208143837/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-acl)