# 将改造与 RxJava 集成

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/retrofit-rxjava>

## 1。概述

本文主要关注如何使用[改型](https://web.archive.org/web/20220627093309/https://square.github.io/retrofit/)实现一个简单的 [RxJava-ready](https://web.archive.org/web/20220627093309/https://github.com/ReactiveX/RxJava) REST 客户端。

我们将构建一个与 GitHub API 交互的示例应用程序——使用标准的改进方法，然后我们将使用 RxJava 增强它，以利用反应式编程的优势。

## 2。普通改装

让我们首先构建一个带有改型的例子。我们将使用 GitHub APIs 获得所有贡献者的排序列表，这些贡献者在任何存储库中都有超过 100 个贡献。

### 2.1。Maven 依赖关系

为了开始一个带有改造的项目，让我们包括这些 Maven 工件:

```java
<dependency>
    <groupId>com.squareup.retrofit2</groupId>
    <artifactId>retrofit</artifactId>
    <version>2.3.0</version>
</dependency>

<dependency>
    <groupId>com.squareup.retrofit2</groupId>
    <artifactId>converter-gson</artifactId>
    <version>2.3.0</version>
</dependency>
```

对于最新版本，看看 Maven 中央存储库上的[改型](https://web.archive.org/web/20220627093309/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.squareup.retrofit2%22%20AND%20a%3A%22retrofit%22)和[转换器-gson](https://web.archive.org/web/20220627093309/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.squareup.retrofit2%22%20AND%20a%3A%22converter-gson%22) 。

### 2.2。API 接口

让我们创建一个简单的界面:

```java
public interface GitHubBasicApi {

    @GET("users/{user}/repos")
    Call<List> listRepos(@Path("user") String user);

    @GET("repos/{user}/{repo}/contributors")
    Call<List> listRepoContributors(
      @Path("user") String user,
      @Path("repo") String repo);   
}
```

`listRepos()`方法检索作为路径参数传递的给定用户的存储库列表。

`listRepoContributers()`方法检索给定用户和存储库的贡献者列表，两者都作为路径参数传递。

### 2.3。逻辑

让我们使用改进的`Call`对象和普通的 Java 代码来实现所需的逻辑:

```java
class GitHubBasicService {

    private GitHubBasicApi gitHubApi;

    GitHubBasicService() {
        Retrofit retrofit = new Retrofit.Builder()
          .baseUrl("https://api.github.com/")
          .addConverterFactory(GsonConverterFactory.create())
          .build();

        gitHubApi = retrofit.create(GitHubBasicApi.class);
    }

    List<String> getTopContributors(String userName) throws IOException {
        List<Repository> repos = gitHubApi
          .listRepos(userName)
          .execute()
          .body();

        repos = repos != null ? repos : Collections.emptyList();

        return repos.stream()
          .flatMap(repo -> getContributors(userName, repo))
          .sorted((a, b) -> b.getContributions() - a.getContributions())
          .map(Contributor::getName)
          .distinct()
          .sorted()
          .collect(Collectors.toList());
    }

    private Stream<Contributor> getContributors(String userName, Repository repo) {
        List<Contributor> contributors = null;
        try {
            contributors = gitHubApi
              .listRepoContributors(userName, repo.getName())
              .execute()
              .body();
        } catch (IOException e) {
            e.printStackTrace();
        }

        contributors = contributors != null ? contributors : Collections.emptyList();

        return contributors.stream()
          .filter(c -> c.getContributions() > 100);
    }
}
```

## 3。与 RxJava 集成

翻新让我们通过使用翻新的`Call`适配器，用定制的处理程序而不是普通的`Call`对象来接收调用结果。这使得在这里使用 RxJava `Observables`和`Flowables`成为可能。

### 3.1。Maven 依赖关系

要使用 RxJava 适配器，我们需要包含这个 Maven 工件:

```java
<dependency>
    <groupId>com.squareup.retrofit2</groupId>
    <artifactId>adapter-rxjava</artifactId>
    <version>2.3.0</version>
</dependency>
```

最新版本请查看 Maven 中央存储库中的 [adapter-rxjava](https://web.archive.org/web/20220627093309/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.squareup.retrofit2%22%20AND%20a%3A%22adapter-rxjava%22) 。

### 3.2。注册 RxJava 调用适配器

让我们将`RxJavaCallAdapter`添加到构建器中:

```java
Retrofit retrofit = new Retrofit.Builder()
  .baseUrl("https://api.github.com/")
  .addConverterFactory(GsonConverterFactory.create())
  .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
  .build();
```

### 3.3。API 接口

此时，我们可以将接口方法的返回类型改为使用`Observable<…>`而不是`Call<…>`。我们可以使用其他 Rx 类型，如`Observable`、`Flowable`、`Single`、`Maybe`、`Completable`。

让我们修改我们的 API 接口以使用`Observable`:

```java
public interface GitHubRxApi {

    @GET("users/{user}/repos")
    Observable<List<Repository>> listRepos(@Path("user") String user);

    @GET("repos/{user}/{repo}/contributors")
    Observable<List<Contributer>> listRepoContributors(
      @Path("user") String user,
      @Path("repo") String repo);   
}
```

### 3.4。逻辑

让我们使用 RxJava 来实现它:

```java
class GitHubRxService {

    private GitHubRxApi gitHubApi;

    GitHubRxService() {
        Retrofit retrofit = new Retrofit.Builder()
          .baseUrl("https://api.github.com/")
          .addConverterFactory(GsonConverterFactory.create())
          .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
          .build();

        gitHubApi = retrofit.create(GitHubRxApi.class);
    }

    Observable<String> getTopContributors(String userName) {
        return gitHubApi.listRepos(userName)
          .flatMapIterable(x -> x)
          .flatMap(repo -> gitHubApi.listRepoContributors(userName, repo.getName()))
          .flatMapIterable(x -> x)
          .filter(c -> c.getContributions() > 100)
          .sorted((a, b) -> b.getContributions() - a.getContributions())
          .map(Contributor::getName)
          .distinct();
    }
}
```

## 4。结论

比较使用 RxJava 前后的代码，我们发现它在以下方面得到了改进:

*   反应式——由于我们的数据现在以流的形式流动，它使我们能够在无阻塞背压的情况下进行异步流处理
*   清晰——由于其声明性
*   简明——整个操作可以表示为一个操作链

本文中的所有代码都可以在 GitHub 上获得。

包`com.baeldung.retrofit.basic`包含基本改装示例，而包`com.baeldung.retrofit.` rx 包含带有 RxJava 集成的改装示例。