# 清除注册生成的过期令牌

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/registration-token-cleanup>

## 1。概述

在本教程中，我们将继续[正在进行的](/web/20220813051842/https://www.baeldung.com/spring-security-registration) **`Registration with Spring Security`系列**来设置一个计划任务，以清除过期的`VerificationToken`。在注册过程中，`VerificationToken`将持续存在。在本文中，我们将展示如何删除这些实体。

## 2。移除过期令牌

回想一下系列的前一篇文章中的[，一个验证令牌有成员 **`expiryDate`** ，代表令牌的到期时间戳:](/web/20220813051842/https://www.baeldung.com/registration-verify-user-by-email)

```
@Entity
public class VerificationToken {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String token;

    @OneToOne(targetEntity = User.class, fetch = FetchType.EAGER)
    @JoinColumn(nullable = false, name = "user_id", 
      foreignKey = @ForeignKey(name="FK_VERIFY_USER"))
    private User user;

    private Date expiryDate;
    ...
}
```

我们将使用这个`expiryDate`属性来生成一个包含 Spring 数据的查询。

如果你想了解更多关于 Spring Data JPA 的信息，请看这篇文章。

## 2.1。删除操作

为了便于删除令牌，我们将在`VerificationTokenRepository`中添加一个新方法来删除过期的令牌:

```
public interface VerificationTokenRepository
  extends JpaRepository<VerificationToken, Long> {

    void deleteByExpiryDateLessThan(Date now);
}
```

使用[查询关键字](https://web.archive.org/web/20220813051842/https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repository-query-keywords) `LessThan`向 Spring Data 的[查询创建](https://web.archive.org/web/20220813051842/https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)机制表明，我们只对 **`expiryDate`** 属性小于指定时间的令牌感兴趣。

请注意，因为`VerificationToken`与标记为`FetchType.EAGER`的`User`有一个`@OneToOne`关联，所以**也发出一个 select 来填充`User`实体**——即使`deleteByExpiryDateLessThan`的签名具有返回类型`void`:

```
select 
    *
from 
    VerificationToken verification 
where 
    verification.expiryDate < ?

select 
    * 
from
    user_account user 
where
    user.id=?

delete from 
    VerificationToken
where
    id=? 
```

## 2.2。用 JPQL 删除

或者，如果不需要将实体加载到持久性上下文中，我们可以创建一个 JPQL 查询:

```
public interface VerificationTokenRepository
  extends JpaRepository<VerificationToken, Long> {

    @Modifying
    @Query("delete from VerificationToken t where t.expiryDate <= ?1")
    void deleteAllExpiredSince(Date now);
}
```

Hibernate 不会将实体加载到持久性上下文中:

```
delete from
    VerificationToken
where
    expiryDate <= ? 
```

## 3。安排令牌清除任务

我们现在有一个希望定期执行的查询；我们将[使用 Spring](https://web.archive.org/web/20220813051842/https://docs.spring.io/spring/docs/current/spring-framework-reference/html/scheduling.html) 中的调度支持，并创建一个方法来运行我们的删除逻辑。

如果你正在寻找关于 Spring 作业调度框架的更多信息，请看这篇文章。

## 3.1。启用调度

为了能够调度任务，我们创建了一个新的配置类`SpringTaskConfig`，用`@EnableScheduling`进行了注释:

```
@Configuration
@EnableScheduling
public class SpringTaskConfig {
    //
} 
```

## 3.2。清除令牌任务

在服务层，我们用当前时间调用我们的存储库。

然后我们用`@Scheduled`注释该方法，表示 Spring 应该定期执行它:

```
@Service
@Transactional
public class TokensPurgeTask {

    @Autowired
    private VerificationTokenRepository tokenRepository;

    @Scheduled(cron = "${purge.cron.expression}")
    public void purgeExpired() {
        Date now = Date.from(Instant.now());
        tokenRepository.deleteAllExpiredSince(now);
    }
}
```

## 3.3。时间表

我们使用一个属性来保存 crontab 设置的值，以避免在更改时重新编译。在`application.properties`中，我们赋值:

```
#    5am every day
purge.cron.expression=0 0 5 * * ?
```

## 4。结论

在本文中，我们使用 **Spring 数据 JPA** 解决了`VerificationToken` s 的移除问题。

我们演示了如何使用属性表达式来创建查询，以查找到期日期小于指定时间的所有令牌。我们创建了一个任务来在运行时调用这个干净的逻辑——并将它注册到 Spring Job Scheduling 框架中，以便定期执行。

使用 Spring 安全教程注册的实现可以在 github 项目中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。