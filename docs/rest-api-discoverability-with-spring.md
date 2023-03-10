# 春假服务的帽子

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-api-discoverability-with-spring>

## 1。概述

本文将关注 Spring REST 服务中可发现性的**实现，以及满足 HATEOAS 约束。**

本文主要关注 Spring MVC。我们的文章[介绍春季帽子](/web/20211028210020/https://www.baeldung.com/spring-hateoas-tutorial)描述了如何在 Spring Boot 使用帽子。

## 2。通过事件解耦可发现性

作为 web 层的一个独立方面或关注点，可发现性应该与处理 HTTP 请求的控制器分离。为此，控制器将为所有需要额外操作响应的操作触发事件。

首先，让我们创建事件:

```java
public class SingleResourceRetrieved extends ApplicationEvent {
    private HttpServletResponse response;

    public SingleResourceRetrieved(Object source, HttpServletResponse response) {
        super(source);

        this.response = response;
    }

    public HttpServletResponse getResponse() {
        return response;
    }
}
public class ResourceCreated extends ApplicationEvent {
    private HttpServletResponse response;
    private long idOfNewResource;

    public ResourceCreated(Object source, 
      HttpServletResponse response, long idOfNewResource) {
        super(source);

        this.response = response;
        this.idOfNewResource = idOfNewResource;
    }

    public HttpServletResponse getResponse() {
        return response;
    }
    public long getIdOfNewResource() {
        return idOfNewResource;
    }
}
```

然后，**控制器，有两个简单的操作——`find by id`和`create` :**

```java
@RestController
@RequestMapping(value = "/foos")
public class FooController {

    @Autowired
    private ApplicationEventPublisher eventPublisher;

    @Autowired
    private IFooService service;

    @GetMapping(value = "foos/{id}")
    public Foo findById(@PathVariable("id") Long id, HttpServletResponse response) {
        Foo resourceById = Preconditions.checkNotNull(service.findOne(id));

        eventPublisher.publishEvent(new SingleResourceRetrieved(this, response));
        return resourceById;
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public void create(@RequestBody Foo resource, HttpServletResponse response) {
        Preconditions.checkNotNull(resource);
        Long newId = service.create(resource).getId();

        eventPublisher.publishEvent(new ResourceCreated(this, response, newId));
    }
}
```

然后，我们可以用任意数量的解耦监听器来处理这些事件。其中每一项都可以专注于自己的特定情况，并有助于满足总体 HATEOAS 约束。

侦听器应该是调用堆栈中的最后一个对象，没有必要直接访问它们；因此，它们不是公开的。

## 3。使新创建的资源的 URI 可被发现

正如在 HATEOAS 、**的[上一篇文章中所讨论的，创建新资源的操作应该在响应的`Location` HTTP 头](/web/20211028210020/https://www.baeldung.com/restful-web-service-discoverability "REST API Discoverability")**中返回该资源的 URI。

我们将使用一个监听器来处理这个问题:

```java
@Component
class ResourceCreatedDiscoverabilityListener
  implements ApplicationListener<ResourceCreated>{

    @Override
    public void onApplicationEvent(ResourceCreated resourceCreatedEvent){
       Preconditions.checkNotNull(resourceCreatedEvent);

       HttpServletResponse response = resourceCreatedEvent.getResponse();
       long idOfNewResource = resourceCreatedEvent.getIdOfNewResource();

       addLinkHeaderOnResourceCreation(response, idOfNewResource);
   }
   void addLinkHeaderOnResourceCreation
     (HttpServletResponse response, long idOfNewResource){
       URI uri = ServletUriComponentsBuilder.fromCurrentRequestUri().
         path("/{idOfNewResource}").buildAndExpand(idOfNewResource).toUri();
       response.setHeader("Location", uri.toASCIIString());
    }
}
```

在本例中，**我们使用了`ServletUriComponentsBuilder`**——这有助于使用当前请求。这样，我们不需要传递任何东西，我们可以简单地静态访问它。

如果 API 将返回`ResponseEntity`——我们也可以使用 [`Location`支持](https://web.archive.org/web/20211028210020/https://github.com/spring-projects/spring-framework/issues/12675)。

## 4。获取单个资源

在检索单个资源时，**客户端应该能够发现 URI 以获取该类型的所有资源**:

```java
@Component
class SingleResourceRetrievedDiscoverabilityListener
 implements ApplicationListener<SingleResourceRetrieved>{

    @Override
    public void onApplicationEvent(SingleResourceRetrieved resourceRetrievedEvent){
        Preconditions.checkNotNull(resourceRetrievedEvent);

        HttpServletResponse response = resourceRetrievedEvent.getResponse();
        addLinkHeaderOnSingleResourceRetrieval(request, response);
    }
    void addLinkHeaderOnSingleResourceRetrieval(HttpServletResponse response){
        String requestURL = ServletUriComponentsBuilder.fromCurrentRequestUri().
          build().toUri().toASCIIString();
        int positionOfLastSlash = requestURL.lastIndexOf("/");
        String uriForResourceCreation = requestURL.substring(0, positionOfLastSlash);

        String linkHeaderValue = LinkUtil
          .createLinkHeader(uriForResourceCreation, "collection");
        response.addHeader(LINK_HEADER, linkHeaderValue);
    }
}
```

注意，链接关系的语义利用了`“collection”`关系类型，在[几个微格式](https://web.archive.org/web/20211028210020/http://microformats.org/wiki/existing-rel-values#non_HTML_rel_values "Microformats - link relations")中指定和使用，但是还没有标准化。

**`Link`报头是最常用的 HTTP 报头之一** **用于发现目的。**创建这个标题的实用程序非常简单:

```java
public class LinkUtil {
    public static String createLinkHeader(String uri, String rel) {
        return "<" + uri + ">; rel=\"" + rel + "\"";
    }
}
```

## 5。根的可发现性

根是整个服务的入口点——它是客户端第一次使用 API 时接触到的东西。

如果从头到尾都要考虑和实现 HATEOAS 约束，那么这就是开始的地方。因此，系统的所有主 URIs 都必须能够从根处被发现。

现在让我们来看看控制器:

```java
@GetMapping("/")
@ResponseStatus(value = HttpStatus.NO_CONTENT)
public void adminRoot(final HttpServletRequest request, final HttpServletResponse response) {
    String rootUri = request.getRequestURL().toString();

    URI fooUri = new UriTemplate("{rootUri}{resource}").expand(rootUri, "foos");
    String linkToFoos = LinkUtil.createLinkHeader(fooUri.toASCIIString(), "collection");
    response.addHeader("Link", linkToFoos);
}
```

当然，这是一个概念的说明，针对`Foo`资源，集中于单个样本 URI。类似地，真正的实现应该为发布给客户端的所有资源添加 URIs。

### 5.1。可发现性与改变 URIs 无关

这可能是一个有争议的点——一方面，HATEOAS 的目的是让客户发现 API 的 URIs，而不是依赖硬编码的值。另一方面——这不是网络的工作方式:是的，URIs 被发现了，但是它们也被加入了书签。

一个微妙但重要的区别是 API 的演变——旧的 URIs 应该仍然工作，但任何将发现 API 的客户端都应该发现新的 URIs——它允许 API 动态变化，即使 API 发生变化，好的客户端也能很好地工作。

总之——仅仅因为 RESTful web 服务的所有 URIs 应该被认为是 [c](https://web.archive.org/web/20211028210020/https://www.w3.org/TR/cooluris/ "Cool URI specification") [ool](https://web.archive.org/web/20211028210020/https://www.w3.org/TR/cooluris/ "Cool URI specification") [URIs](https://web.archive.org/web/20211028210020/https://www.w3.org/TR/cooluris/ "Cool URI specification") (以及酷 URIs [不改变](https://web.archive.org/web/20211028210020/https://www.w3.org/Provider/Style/URI.html "Cool URIs don't change"))—这并不意味着在发展 API 时遵守 HATEOAS 约束不是非常有用的。

## 6。可发现性的警告

正如围绕前几篇文章的一些讨论所述，**可发现性的第一个目标是最少或不使用文档**,让客户通过它得到的响应学习和理解如何使用 API。

事实上，这不应该被认为是一个遥不可及的理想——我们就是这样消费每一个新网页的——没有任何文档。因此，如果这个概念在 REST 的上下文中更成问题，那么它一定是一个技术实现的问题，而不是它是否可能的问题。

也就是说，从技术上来说，我们离一个完全可行的解决方案还很远——规范和框架支持仍在发展，正因为如此，我们必须做出一些妥协。

## 7。结论

本文介绍了在使用 Spring MVC 的 RESTful 服务的上下文中实现可发现性的一些特征，并从根本上触及了可发现性的概念。

所有这些例子和代码片段的实现都可以在 GitHub 上找到[——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。](https://web.archive.org/web/20211028210020/https://github.com/eugenp/tutorials/tree/master/spring-boot-rest "Github Project exemplifying how to implement Discoverability")