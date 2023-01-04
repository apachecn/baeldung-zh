# 无冲突复制数据类型简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-conflict-free-replicated-data-types>

## 1。概述

在本文中，我们将研究无冲突复制数据类型(CRDT)以及如何在 Java 中使用它们。对于我们的例子，我们将使用来自 [`wurmloch-crdt`](https://web.archive.org/web/20221116082707/https://github.com/netopyr/wurmloch-crdt) 库的实现。

当我们在分布式系统中拥有一个由`N`个副本节点组成的集群时，我们可能会遇到一个**网络分区——一些节点暂时无法相互通信**。这种情况被称为裂脑。

当我们的系统中有裂脑时，**一些写请求——即使是对同一个用户——可以发送到彼此不相连的不同副本上**。当这种情况发生时，我们的**系统仍然可用，但不一致**。

当两个分离集群之间的网络重新开始工作时，我们需要决定如何处理不一致的写入和数据。

## 2。拯救无冲突复制数据类型

让我们考虑两个节点，`A`和`B`，它们由于大脑分裂而断开连接。

假设一个用户更改了他的登录，一个请求被发送到节点`A`。然后，他/她决定再次更改它，但是这次请求发送到节点`B`。

因为裂脑，两个节点没有连接。我们需要决定当网络再次工作时，这个用户的登录应该是什么样子。

我们可以利用几个策略:我们可以给用户解决冲突的机会(就像在 Google Docs 中所做的那样)，或者**我们可以** **使用 CRDT 为我们合并来自不同副本**的数据。

## 3。Maven 依赖关系

首先，让我们向库中添加一个提供一组有用的 CRDTs 的依赖项:

```
<dependency>
    <groupId>com.netopyr.wurmloch</groupId>
    <artifactId>wurmloch-crdt</artifactId>
    <version>0.1.0</version>
</dependency>
```

最新版本可以在 [Maven Central](https://web.archive.org/web/20221116082707/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.netopyr.wurmloch%22%20AND%20a%3A%22wurmloch-crdt%22) 上找到。

## 4。仅增长集

最基本的 CRDT 是一个只生长的集合。**元素只能添加到`GSet` 中，不能删除。**当`GSet`发散时，通过计算两个集合的并集，可以**轻松合并。**

首先，让我们创建两个副本来模拟分布式数据结构，并使用`connect()` 方法连接这两个副本:

```
LocalCrdtStore crdtStore1 = new LocalCrdtStore();
LocalCrdtStore crdtStore2 = new LocalCrdtStore();
crdtStore1.connect(crdtStore2);
```

一旦我们在集群中获得了两个副本，我们可以在第一个副本上创建一个`GSet` ,并在第二个副本上引用它:

```
GSet<String> replica1 = crdtStore1.createGSet("ID_1");
GSet<String> replica2 = crdtStore2.<String>findGSet("ID_1").get();
```

此时，我们的集群工作正常，两个副本之间有一个活动连接。我们可以从两个不同的副本向集合添加两个元素，并断言集合在两个副本上包含相同的元素:

```
replica1.add("apple");
replica2.add("banana");

assertThat(replica1).contains("apple", "banana");
assertThat(replica2).contains("apple", "banana");
```

假设我们突然有了一个网络分区，并且第一个和第二个副本之间没有连接。我们可以使用`disconnect()`方法模拟网络分区:

```
crdtStore1.disconnect(crdtStore2);
```

接下来，当我们从两个副本向数据集添加元素时，这些更改并不是全局可见的，因为它们之间没有联系:

```
replica1.add("strawberry");
replica2.add("pear");

assertThat(replica1).contains("apple", "banana", "strawberry");
assertThat(replica2).contains("apple", "banana", "pear");
```

**一旦两个集群成员之间的连接再次建立，`GSet` 在两个集合上使用联合在内部合并**，并且两个副本再次一致:

```
crdtStore1.connect(crdtStore2);

assertThat(replica1)
  .contains("apple", "banana", "strawberry", "pear");
assertThat(replica2)
  .contains("apple", "banana", "strawberry", "pear");
```

## 5。仅递增计数器

Increment-Only 计数器是一个 CRDT，它在每个节点上本地聚合所有增量。

**当副本同步时，在网络分区之后，通过对所有节点**上的所有增量求和来计算结果值——这类似于来自`java.concurrent` 的`LongAdder` ，但是在更高的抽象级别上。

让我们使用`GCounter` 创建一个只递增的计数器，并从两个副本中递增它。我们可以看到，总和计算得当:

```
LocalCrdtStore crdtStore1 = new LocalCrdtStore();
LocalCrdtStore crdtStore2 = new LocalCrdtStore();
crdtStore1.connect(crdtStore2);

GCounter replica1 = crdtStore1.createGCounter("ID_1");
GCounter replica2 = crdtStore2.findGCounter("ID_1").get();

replica1.increment();
replica2.increment(2L);

assertThat(replica1.get()).isEqualTo(3L);
assertThat(replica2.get()).isEqualTo(3L); 
```

当我们断开两个集群成员的连接并执行本地增量操作时，我们可以看到值不一致:

```
crdtStore1.disconnect(crdtStore2);

replica1.increment(3L);
replica2.increment(5L);

assertThat(replica1.get()).isEqualTo(6L);
assertThat(replica2.get()).isEqualTo(8L);
```

但是，一旦集群恢复正常，增量将被合并，产生正确的值:

```
crdtStore1.connect(crdtStore2);

assertThat(replica1.get())
  .isEqualTo(11L);
assertThat(replica2.get())
  .isEqualTo(11L);
```

## 6。PN 计数器

对只递增计数器使用类似的规则，我们可以创建一个既可以递增也可以递减的计数器。`PNCounter`分别存储所有增量和减量。

**当副本同步时，结果值将等于** **所有增量的总和减去所有减量的总和**:

```
@Test
public void givenPNCounter_whenReplicasDiverge_thenMergesWithoutConflict() {
    LocalCrdtStore crdtStore1 = new LocalCrdtStore();
    LocalCrdtStore crdtStore2 = new LocalCrdtStore();
    crdtStore1.connect(crdtStore2);

    PNCounter replica1 = crdtStore1.createPNCounter("ID_1");
    PNCounter replica2 = crdtStore2.findPNCounter("ID_1").get();

    replica1.increment();
    replica2.decrement(2L);

    assertThat(replica1.get()).isEqualTo(-1L);
    assertThat(replica2.get()).isEqualTo(-1L);

    crdtStore1.disconnect(crdtStore2);

    replica1.decrement(3L);
    replica2.increment(5L);

    assertThat(replica1.get()).isEqualTo(-4L);
    assertThat(replica2.get()).isEqualTo(4L);

    crdtStore1.connect(crdtStore2);

    assertThat(replica1.get()).isEqualTo(1L);
    assertThat(replica2.get()).isEqualTo(1L);
}
```

## 7。最后写入者获胜寄存器

有时，我们有更复杂的业务规则，在器械包或柜台上操作是不够的。我们可以使用 Last-Writer-Wins 寄存器，当合并分歧的数据集时，**只保存最后更新的值。卡桑德拉用这种策略来解决冲突。**

我们需要**在使用这种策略时非常小心，因为它会丢弃同时发生的变化**。

让我们创建一个包含两个副本和`LWWRegister` 类实例的集群:

```
LocalCrdtStore crdtStore1 = new LocalCrdtStore("N_1");
LocalCrdtStore crdtStore2 = new LocalCrdtStore("N_2");
crdtStore1.connect(crdtStore2);

LWWRegister<String> replica1 = crdtStore1.createLWWRegister("ID_1");
LWWRegister<String> replica2 = crdtStore2.<String>findLWWRegister("ID_1").get();

replica1.set("apple");
replica2.set("banana");

assertThat(replica1.get()).isEqualTo("banana");
assertThat(replica2.get()).isEqualTo("banana"); 
```

当第一个副本将值设置为`apple` 并且第二个副本将其更改为`banana,` 时，`LWWRegister` 仅保留最后一个值。

让我们看看如果集群断开连接会发生什么:

```
crdtStore1.disconnect(crdtStore2);

replica1.set("strawberry");
replica2.set("pear");

assertThat(replica1.get()).isEqualTo("strawberry");
assertThat(replica2.get()).isEqualTo("pear");
```

每个副本都保留其不一致数据的本地副本。当我们调用`set()` 方法时，`LWWRegister` 在内部分配一个特殊的版本值，该值使用一个`VectorClock`算法来标识每个更新。

集群同步时，**取最高版本** **的值，** **丢弃之前的每次更新**:

```
crdtStore1.connect(crdtStore2);

assertThat(replica1.get()).isEqualTo("pear");
assertThat(replica2.get()).isEqualTo("pear");
```

## 8。结论

在本文中，我们展示了分布式系统在维护可用性时的一致性问题。

在网络分区的情况下，我们需要在集群同步时合并分散的数据。我们看到了如何使用 CRDTs 来执行不同数据的合并。

所有这些例子和代码片段都可以在 [GitHub 项目](https://web.archive.org/web/20221116082707/https://github.com/eugenp/tutorials/tree/master/libraries-data-2)中找到——这是一个 Maven 项目，所以它应该很容易导入和运行。