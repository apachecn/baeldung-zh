# SQL 注入以及如何防范？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/sql-injection>

 ![announcement-icon.png](img/6945c4034b299267bc6d862f961d9fcf.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220524121614/https://www.baeldung.com/lightrun-n-security)

## 1.介绍

尽管是最著名的漏洞之一， [SQL 注入](/web/20220524121614/https://www.baeldung.com/cs/sql-injection)仍然排在臭名昭著的 [OWASP 十大漏洞列表](https://web.archive.org/web/20220524121614/https://owasp.org/www-project-top-ten/)的首位——现在是更一般的`Injection`类的一部分。

在本教程中，我们将探索 Java 中导致易受攻击的常见编码错误，以及如何使用 JVM 的标准运行时库中可用的 API 来避免这些错误。我们还将讨论我们可以从 JPA、Hibernate 和其他 ORM 中得到什么保护，以及我们仍然需要担心哪些盲点。

## 2.应用程序如何变得容易受到 SQL 注入的攻击？

注入攻击之所以有效，是因为对于许多应用程序来说，执行给定计算的唯一方法是动态生成代码，然后由另一个系统或组件运行。如果在生成这些代码的过程中，我们使用了不可信的数据而没有进行适当的净化，我们就为黑客的利用留下了方便之门。

这种说法可能听起来有点抽象，所以让我们用一个教科书示例来看看这在实践中是如何发生的:

```
public List<AccountDTO>
  unsafeFindAccountsByCustomerId(String customerId)
  throws SQLException {
    // UNSAFE !!! DON'T DO THIS !!!
    String sql = "select "
      + "customer_id,acc_number,branch_id,balance "
      + "from Accounts where customer_id = '"
      + customerId 
      + "'";
    Connection c = dataSource.getConnection();
    ResultSet rs = c.createStatement().executeQuery(sql);
    // ...
}
```

这段代码的问题是显而易见的:**我们已经将****`customerId`的值放入查询中，根本没有验证**。如果我们确信这个值只来自可信的来源，就不会有什么不好的事情发生，但是我们能做到吗？

让我们假设这个函数在 REST API 实现中用于一个`account `资源。利用这段代码很简单:我们所要做的就是发送一个值，当它与查询的固定部分连接时，会改变它的预期行为:

```
curl -X GET \
  'http://localhost:8080/accounts?customerId=abc%27%20or%20%271%27=%271' \
```

假设`customerId`参数值在到达我们的函数之前没有被检查，下面是我们将收到的结果:

```
abc' or '1' = '1
```

当我们将这个值与固定部分连接起来时，我们得到将被执行的最终 SQL 语句:

```
select customer_id, acc_number,branch_id, balance
  from Accounts where customerId = 'abc' or '1' = '1'
```

可能不是我们想要的…

一个聪明的开发者(我们不都是吗？)现在可能会想:“这太愚蠢了！我会使用字符串连接来构建一个这样的查询”。

没那么快…这个典型的例子确实很傻，但是**有些情况下我们可能仍然需要这么做**:

*   具有动态搜索标准的复杂查询:根据用户提供的标准添加 UNION 子句
*   动态分组或排序:REST APIs 用作 GUI 数据表的后端

### 2.1.我用的是 JPA。我很安全，对吗？

这是一个常见的误解。JPA 和其他 ORM 让我们不用创建手工编码的 SQL 语句，但是它们**不会阻止我们编写易受攻击的代码**。

让我们看看前面例子的 JPA 版本是什么样子的:

```
public List<AccountDTO> unsafeJpaFindAccountsByCustomerId(String customerId) {    
    String jql = "from Account where customerId = '" + customerId + "'";        
    TypedQuery<Account> q = em.createQuery(jql, Account.class);        
    return q.getResultList()
      .stream()
      .map(this::toAccountDTO)
      .collect(Collectors.toList());        
} 
```

我们之前指出的相同问题在这里也存在:**我们使用未验证的输入来创建 JPA 查询**，所以我们在这里暴露于相同类型的利用。

## 3.预防技术

现在我们知道了什么是 SQL 注入，让我们看看如何保护我们的代码免受这种攻击。这里我们主要关注 Java 和其他 JVM 语言中的一些非常有效的技术，但是类似的概念也适用于其他环境，比如 PHP。Net，Ruby 等等。

对于那些寻找可用技术的完整列表的人来说，包括特定于数据库的技术， [OWASP 项目](https://web.archive.org/web/20220524121614/https://www.owasp.org/)维护了一个 [SQL 注入预防备忘单](https://web.archive.org/web/20220524121614/https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)，这是一个学习更多关于这个主题的好地方。

### 3.1.参数化查询

这种技术包括使用带有问号占位符("？"的准备好的语句)中，每当我们需要插入用户提供的值时。这非常有效，除非 JDBC 驱动程序的实现中有错误，否则不会受到攻击。

让我们重写示例函数来使用这种技术:

```
public List<AccountDTO> safeFindAccountsByCustomerId(String customerId)
  throws Exception {

    String sql = "select "
      + "customer_id, acc_number, branch_id, balance from Accounts"
      + "where customer_id = ?";

    Connection c = dataSource.getConnection();
    PreparedStatement p = c.prepareStatement(sql);
    p.setString(1, customerId);
    ResultSet rs = p.executeQuery(sql)); 
    // omitted - process rows and return an account list
}
```

这里我们使用了在`Connection`实例中可用的`prepareStatement()`方法来获得一个`PreparedStatement`。这个接口用几个方法扩展了常规的`Statement `接口，这些方法允许我们在执行查询之前安全地在查询中插入用户提供的值。

对于 JPA，我们有一个类似的特性:

```
String jql = "from Account where customerId = :customerId";
TypedQuery<Account> q = em.createQuery(jql, Account.class)
  .setParameter("customerId", customerId);
// Execute query and return mapped results (omitted)
```

当在 Spring Boot 下运行这段代码时，我们可以将属性`logging.level.sql`设置为 DEBUG，并查看为了执行该操作实际构建了什么查询:

```
// Note: Output formatted to fit screen
[DEBUG][SQL] select
  account0_.id as id1_0_,
  account0_.acc_number as acc_numb2_0_,
  account0_.balance as balance3_0_,
  account0_.branch_id as branch_i4_0_,
  account0_.customer_id as customer5_0_ 
from accounts account0_ 
where account0_.customer_id=?
```

正如所料，ORM 层使用一个占位符为`customerId`参数创建了一个准备好的语句。这与我们在普通的 JDBC 案例中所做的一样——但是少了一些语句，这很好。

另外，这种方法通常会带来更好的查询性能，因为大多数数据库可以缓存与准备好的语句相关联的查询计划。

请注意**这种方法只对用作** **值**的占位符有效。例如，我们不能使用占位符来动态改变表的名称:

```
// This WILL NOT WORK !!!
PreparedStatement p = c.prepareStatement("select count(*) from ?");
p.setString(1, tableName);
```

在这里，JPA 也不会有所帮助:

```
// This WILL NOT WORK EITHER !!!
String jql = "select count(*) from :tableName";
TypedQuery q = em.createQuery(jql,Long.class)
  .setParameter("tableName", tableName);
return q.getSingleResult(); 
```

在这两种情况下，我们都会得到一个运行时错误。

这背后的主要原因是预准备语句的本质:数据库服务器使用它们来缓存提取结果集所需的查询计划，对于任何可能的值，这通常是相同的。对于表名和 SQL 语言中可用的其他结构(比如在`order by`子句中使用的列)来说，情况并非如此。

### 3.2.JPA 标准 API

因为显式 JQL 查询构建是 SQL 注入的主要来源，所以我们应该尽可能地支持使用 JPA 的查询 API。

关于这个 API 的快速入门，请参考关于 Hibernate 标准查询的文章。同样值得一读的是我们的[关于 JPA 元模型](/web/20220524121614/https://www.baeldung.com/hibernate-criteria-queries-metamodel)的文章，它展示了如何生成元模型类，这将帮助我们摆脱用于列名的字符串常量——以及当它们改变时出现的运行时错误。

让我们重写 JPA 查询方法，以使用标准 API:

```
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Account> cq = cb.createQuery(Account.class);
Root<Account> root = cq.from(Account.class);
cq.select(root).where(cb.equal(root.get(Account_.customerId), customerId));

TypedQuery<Account> q = em.createQuery(cq);
// Execute query and return mapped results (omitted)
```

在这里，我们使用了更多的代码行来获得相同的结果，但好处是现在**我们不必担心 JQL 语法**。

另一个要点:尽管冗长，Criteria API 使得创建复杂的查询服务更加简单和安全。要获得展示如何在实践中做到这一点的完整示例，请查看由 [JHipster](https://web.archive.org/web/20220524121614/https://www.jhipster.tech/) 生成的应用程序所使用的方法。

### 3.3.用户数据净化

数据净化是一种对用户提供的数据进行过滤的技术，因此它可以被我们应用程序的其他部分安全地使用。过滤器的实现可能有很大不同，但我们通常可以将它们分为两种类型:白名单和黑名单。

由试图识别无效模式的过滤器组成的`Blacklists`，通常在 SQL 注入预防的上下文中没有什么价值——但对于检测来说却没有！稍后将详细介绍。

另一方面，当**我们可以准确定义什么是有效输入时，`Whitelists`工作得特别好。**

让我们增强我们的`safeFindAccountsByCustomerId `方法，这样现在调用者也可以指定用于对结果集进行排序的列。因为我们知道可能的列的集合，所以我们可以使用一个简单的集合实现一个白名单，并使用它来整理接收到的参数:

```
private static final Set<String> VALID_COLUMNS_FOR_ORDER_BY
  = Collections.unmodifiableSet(Stream
      .of("acc_number","branch_id","balance")
      .collect(Collectors.toCollection(HashSet::new)));

public List<AccountDTO> safeFindAccountsByCustomerId(
  String customerId,
  String orderBy) throws Exception { 
    String sql = "select "
      + "customer_id,acc_number,branch_id,balance from Accounts"
      + "where customer_id = ? ";
    if (VALID_COLUMNS_FOR_ORDER_BY.contains(orderBy)) {
        sql = sql + " order by " + orderBy;
    } else {
        throw new IllegalArgumentException("Nice try!");
    }
    Connection c = dataSource.getConnection();
    PreparedStatement p = c.prepareStatement(sql);
    p.setString(1,customerId);
    // ... result set processing omitted
}
```

在这里，**我们结合了准备好的语句方法和一个用于净化`orderBy`参数**的白名单。最终结果是一个包含最终 SQL 语句的安全字符串。在这个简单的例子中，我们使用了一个静态集合，但是我们也可以使用数据库元数据函数来创建它。

我们可以对 JPA 使用相同的方法，同样利用标准 API 和元数据来避免在代码中使用`String`常量:

```
// Map of valid JPA columns for sorting
final Map<String,SingularAttribute<Account,?>> VALID_JPA_COLUMNS_FOR_ORDER_BY = Stream.of(
  new AbstractMap.SimpleEntry<>(Account_.ACC_NUMBER, Account_.accNumber),
  new AbstractMap.SimpleEntry<>(Account_.BRANCH_ID, Account_.branchId),
  new AbstractMap.SimpleEntry<>(Account_.BALANCE, Account_.balance))
  .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));

SingularAttribute<Account,?> orderByAttribute = VALID_JPA_COLUMNS_FOR_ORDER_BY.get(orderBy);
if (orderByAttribute == null) {
    throw new IllegalArgumentException("Nice try!");
}

CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Account> cq = cb.createQuery(Account.class);
Root<Account> root = cq.from(Account.class);
cq.select(root)
  .where(cb.equal(root.get(Account_.customerId), customerId))
  .orderBy(cb.asc(root.get(orderByAttribute)));

TypedQuery<Account> q = em.createQuery(cq);
// Execute query and return mapped results (omitted)
```

这种代码的基本结构与普通的 JDBC 代码相同。首先，我们使用一个白名单来净化列名，然后我们继续创建一个`CriteriaQuery`来从数据库中获取记录。

### 3.4.我们现在安全了吗？

让我们假设我们在任何地方都使用了参数化查询和/或白名单。我们现在可以去找我们的经理并保证我们是安全的吗？

嗯…没那么快。即使不考虑[图灵的停机问题](https://web.archive.org/web/20220524121614/https://en.wikipedia.org/wiki/Halting_problem)，我们还必须考虑其他方面:

1.  `Stored Procedures` : **这些也容易出现 SQL 注入问题**；请尽可能对通过准备好的语句发送到数据库的值进行清理
2.  与过程调用的问题相同，但更加隐蔽，因为有时我们不知道它们在那里…
3.  `Insecure Direct Object References`:即使我们的应用程序没有 SQL 注入，仍然存在与该漏洞类别相关的风险——这里的要点与攻击者欺骗应用程序的不同方式有关，因此它会返回他或她不应该访问的记录——在 OWASP 的 GitHub 知识库中有一个关于该主题的很好的备忘单

简而言之，我们最好的选择是谨慎。如今，许多组织正是为此而使用“红队”。让他们做他们的工作，这就是找到任何剩余的漏洞。

## 4.损害控制技术

**作为一种良好的安全实践，我们应该始终实施多重防御层**，这一概念被称为`defense in depth`。主要的想法是，即使我们不能在代码中找到所有可能的漏洞——这是处理遗留系统时的常见场景——我们至少应该尝试限制攻击可能造成的损害。

当然，这可能是一整篇文章甚至一本书的主题，但让我们列举一些措施:

1.  应用最小特权原则:**尽可能地限制用于访问数据库的帐户的特权**
2.  使用特定于数据库的可用方法来添加额外的保护层；例如，H2 数据库有一个会话级选项，禁用 SQL 查询中的所有文字值
3.  使用短期凭证:**让应用程序经常轮换数据库凭证；**实现这一点的一个好方法是使用 [Spring Cloud Vault](/web/20220524121614/https://www.baeldung.com/spring-cloud-vault)
4.  记录一切:**如果应用程序存储客户数据，这是必须的；**有许多解决方案可以直接集成到数据库中，或者作为代理，因此在发生攻击时，我们至少可以评估损失
5.  使用 [WAFs](https://web.archive.org/web/20220524121614/https://owasp.org/www-community/Web_Application_Firewall) 或类似的入侵检测解决方案:这些是典型的`blacklist`例子——通常，它们带有一个相当大的已知攻击特征数据库，并会在检测到时触发可编程的操作。有些还包括可以通过应用一些工具来检测入侵的内置 JVM 代理——这种方法的主要优点是最终的漏洞变得更容易修复，因为我们将有一个完整的堆栈跟踪可用。

## 5.结论

在本文中，我们讨论了 Java 应用程序中的 SQL 注入漏洞——对于任何依赖数据开展业务的组织来说，这都是一个非常严重的威胁——以及如何使用简单的技术来防止这些漏洞。

和往常一样，这篇文章的完整代码可以在 Github 上找到。