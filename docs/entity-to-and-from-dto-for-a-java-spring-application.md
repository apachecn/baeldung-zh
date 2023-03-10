# Spring REST API 的实体到 DTO 的转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/entity-to-and-from-dto-for-a-java-spring-application>

## 1。概述

在本教程中，我们将处理 Spring 应用程序的内部实体和发布回客户端的外部[dto](/web/20220925070143/https://www.baeldung.com/java-dto-pattern)(数据传输对象)之间需要发生的**转换。**

## 延伸阅读:

## [Spring 的 RequestBody 和 ResponseBody 注释](/web/20220925070143/https://www.baeldung.com/spring-request-response-body)

Learn about the Spring @RequestBody and @ResponseBody annotations.[Read more](/web/20220925070143/https://www.baeldung.com/spring-request-response-body) →

## [地图结构快速指南](/web/20220925070143/https://www.baeldung.com/mapstruct)

A quick and practical guide to using MapStruct[Read more](/web/20220925070143/https://www.baeldung.com/mapstruct) →

## 2。模型映射器

让我们首先介绍我们将要用来执行这个实体-DTO 转换的主库，`[ModelMapper](https://web.archive.org/web/20220925070143/http://modelmapper.org/getting-started/)`。

我们将在`pom.xml`中需要这个依赖关系:

```java
<dependency>
    <groupId>org.modelmapper</groupId>
    <artifactId>modelmapper</artifactId>
    <version>2.4.5</version>
</dependency>
```

要检查这个库是否有新版本，[请点击这里](https://web.archive.org/web/20220925070143/https://search.maven.org/classic/#search|gav|1|g%3A%22org.modelmapper%22%20AND%20a%3A%22modelmapper%22)。

然后我们将在 Spring 配置中定义`ModelMapper` bean:

```java
@Bean
public ModelMapper modelMapper() {
    return new ModelMapper();
}
```

## 3。DTO

接下来我们来介绍一下这个双面问题的 DTO 一面，`Post` DTO:

```java
public class PostDto {
    private static final SimpleDateFormat dateFormat
      = new SimpleDateFormat("yyyy-MM-dd HH:mm");

    private Long id;

    private String title;

    private String url;

    private String date;

    private UserDto user;

    public Date getSubmissionDateConverted(String timezone) throws ParseException {
        dateFormat.setTimeZone(TimeZone.getTimeZone(timezone));
        return dateFormat.parse(this.date);
    }

    public void setSubmissionDate(Date date, String timezone) {
        dateFormat.setTimeZone(TimeZone.getTimeZone(timezone));
        this.date = dateFormat.format(date);
    }

    // standard getters and setters
} 
```

请注意，这两个与自定义日期相关的方法处理客户端和服务器之间的日期转换:

*   `getSubmissionDateConverted()`方法将日期`String`转换成服务器时区的`Date`,以便在持久化的`Post`实体中使用
*   `setSubmissionDate()`方法是将 DTO 的日期设置为当前用户时区中`Post`的`Date`

## 4。服务层

现在让我们来看一个服务级别的操作，它显然将与实体(而不是 DTO)一起工作:

```java
public List<Post> getPostsList(
  int page, int size, String sortDir, String sort) {

    PageRequest pageReq
     = PageRequest.of(page, size, Sort.Direction.fromString(sortDir), sort);

    Page<Post> posts = postRepository
      .findByUser(userService.getCurrentUser(), pageReq);
    return posts.getContent();
}
```

接下来，我们将看看服务之上的层，即控制器层。这是转换实际发生的地方。

## 5。控制器层

接下来让我们检查一个标准的控制器实现，为`Post`资源公开简单的 REST API。

我们将在这里展示一些简单的 CRUD 操作:创建、更新、获取一个和获取全部。鉴于操作非常简单，**我们对实体-DTO 转换方面特别感兴趣**:

```java
@Controller
class PostRestController {

    @Autowired
    private IPostService postService;

    @Autowired
    private IUserService userService;

    @Autowired
    private ModelMapper modelMapper;

    @GetMapping
    @ResponseBody
    public List<PostDto> getPosts(...) {
        //...
        List<Post> posts = postService.getPostsList(page, size, sortDir, sort);
        return posts.stream()
          .map(this::convertToDto)
          .collect(Collectors.toList());
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    @ResponseBody
    public PostDto createPost(@RequestBody PostDto postDto) {
        Post post = convertToEntity(postDto);
        Post postCreated = postService.createPost(post));
        return convertToDto(postCreated);
    }

    @GetMapping(value = "/{id}")
    @ResponseBody
    public PostDto getPost(@PathVariable("id") Long id) {
        return convertToDto(postService.getPostById(id));
    }

    @PutMapping(value = "/{id}")
    @ResponseStatus(HttpStatus.OK)
    public void updatePost(@PathVariable("id") Long id, @RequestBody PostDto postDto) {
        if(!Objects.equals(id, postDto.getId())){
            throw new IllegalArgumentException("IDs don't match");
        }
        Post post = convertToEntity(postDto);
        postService.updatePost(post);
    }
}
```

这里是**我们从`Post`实体到`PostDto` :** 的转换

```java
private PostDto convertToDto(Post post) {
    PostDto postDto = modelMapper.map(post, PostDto.class);
    postDto.setSubmissionDate(post.getSubmissionDate(), 
        userService.getCurrentUser().getPreference().getTimezone());
    return postDto;
}
```

下面是从 DTO 到实体的转换:

```java
private Post convertToEntity(PostDto postDto) throws ParseException {
    Post post = modelMapper.map(postDto, Post.class);
    post.setSubmissionDate(postDto.getSubmissionDateConverted(
      userService.getCurrentUser().getPreference().getTimezone()));

    if (postDto.getId() != null) {
        Post oldPost = postService.getPostById(postDto.getId());
        post.setRedditID(oldPost.getRedditID());
        post.setSent(oldPost.isSent());
    }
    return post;
}
```

所以我们可以看到，在模型映射器的帮助下，**转换逻辑快速简单。**我们正在使用映射器的`map` API，无需编写一行转换逻辑就能转换数据。

## 6。单元测试

最后，让我们做一个非常简单的测试来确保实体和 DTO 之间的转换正常进行:

```java
public class PostDtoUnitTest {

    private ModelMapper modelMapper = new ModelMapper();

    @Test
    public void whenConvertPostEntityToPostDto_thenCorrect() {
        Post post = new Post();
        post.setId(1L);
        post.setTitle(randomAlphabetic(6));
        post.setUrl("www.test.com");

        PostDto postDto = modelMapper.map(post, PostDto.class);
        assertEquals(post.getId(), postDto.getId());
        assertEquals(post.getTitle(), postDto.getTitle());
        assertEquals(post.getUrl(), postDto.getUrl());
    }

    @Test
    public void whenConvertPostDtoToPostEntity_thenCorrect() {
        PostDto postDto = new PostDto();
        postDto.setId(1L);
        postDto.setTitle(randomAlphabetic(6));
        postDto.setUrl("www.test.com");

        Post post = modelMapper.map(postDto, Post.class);
        assertEquals(postDto.getId(), post.getId());
        assertEquals(postDto.getTitle(), post.getTitle());
        assertEquals(postDto.getUrl(), post.getUrl());
    }
}
```

## 7 .**。结论**

在本文中，我们详细描述了在 Spring REST API 中简化从实体到 DTO 以及从 DTO 到实体的转换，使用模型映射器库而不是手工编写这些转换。

示例的完整源代码可以在 [GitHub 项目](https://web.archive.org/web/20220925070143/https://github.com/eugenp/tutorials/tree/master/spring-boot-rest)中找到。