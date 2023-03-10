# 使用 Spring Security 5.1 客户端定制授权和令牌请求

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-custom-oauth-requests>

## 1。概述

有时，OAuth2 APIs 可能与标准略有不同，在这种情况下，我们需要对标准的 OAuth2 请求进行一些定制。

**Spring Security 5.1 提供了对定制 OAuth2 授权和令牌请求的支持。**

在本教程中，我们将了解如何定制请求参数和响应处理。

## 2。定制授权请求

首先，我们将定制 OAuth2 授权请求。我们可以修改标准参数，并根据需要向授权请求添加额外的参数。

为此，**我们需要实现我们自己的`OAuth2AuthorizationRequestResolver` :**

```java
public class CustomAuthorizationRequestResolver 
  implements OAuth2AuthorizationRequestResolver {

    private OAuth2AuthorizationRequestResolver defaultResolver;

    public CustomAuthorizationRequestResolver(
      ClientRegistrationRepository repo, String authorizationRequestBaseUri) {
        defaultResolver = new DefaultOAuth2AuthorizationRequestResolver(repo, authorizationRequestBaseUri);
    }

    // ...
}
```

注意，我们使用了`DefaultOAuth2AuthorizationRequestResolver`来提供基本功能。

我们还将覆盖`resolve()`方法来添加我们的定制逻辑:

```java
public class CustomAuthorizationRequestResolver 
  implements OAuth2AuthorizationRequestResolver {

    //...

    @Override
    public OAuth2AuthorizationRequest resolve(HttpServletRequest request) {
        OAuth2AuthorizationRequest req = defaultResolver.resolve(request);
        if(req != null) {
            req = customizeAuthorizationRequest(req);
        }
        return req;
    }

    @Override
    public OAuth2AuthorizationRequest resolve(HttpServletRequest request, String clientRegistrationId) {
        OAuth2AuthorizationRequest req = defaultResolver.resolve(request, clientRegistrationId);
        if(req != null) {
            req = customizeAuthorizationRequest(req);
        }
        return req;
    }

    private OAuth2AuthorizationRequest customizeAuthorizationRequest(
      OAuth2AuthorizationRequest req) {
        // ...
    }

}
```

稍后我们将使用我们的方法`customizeAuthorizationRequest()`添加我们的定制，我们将在接下来的部分中讨论。

在实现我们的自定义`OAuth2AuthorizationRequestResolver`之后，我们需要将它添加到我们的安全配置中:

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.oauth2Login()
          .authorizationEndpoint()
          .authorizationRequestResolver(
            new CustomAuthorizationRequestResolver(
              clientRegistrationRepository(), "/oauth2/authorize-client"))
        //...
    }
}
```

**这里我们用`oauth2Login().authorizationEndpoint().authorizationRequestResolver()`来注入我们的自定义`OAuth2AuthorizationRequestResolver.`**

## 3。定制 **授权申请标准参数**

现在，我们来讨论一下实际的定制。我们可以随心所欲地修改`OAuth2AuthorizationRequest`。

首先，我们可以为每个授权请求修改一个标准参数。

例如，我们可以生成自己的`“state”`参数:

```java
private OAuth2AuthorizationRequest customizeAuthorizationRequest(
  OAuth2AuthorizationRequest req) {
    return OAuth2AuthorizationRequest
      .from(req).state("xyz").build();
}
```

## 4。授权请求 **额外参数**

**我们还可以使用`OAuth2AuthorizationRequest`的`additionalParameters()`方法并传入一个`Map:`来给我们的`OAuth2AuthorizationRequest`** 添加额外的参数

```java
private OAuth2AuthorizationRequest customizeAuthorizationRequest(
  OAuth2AuthorizationRequest req) {
    Map<String,Object> extraParams = new HashMap<String,Object>();
    extraParams.putAll(req.getAdditionalParameters()); 
    extraParams.put("test", "extra");

    return OAuth2AuthorizationRequest
      .from(req)
      .additionalParameters(extraParams)
      .build();
}
```

我们还必须确保在添加新内容之前包含旧内容。

让我们通过定制 Okta 授权服务器使用的授权请求来看一个更实际的例子。

### 4.1。定制 Okta 授权请求

[Okta](https://web.archive.org/web/20220801021059/https://developer.okta.com/docs/api/resources/oidc#authorize) 为授权请求提供额外的可选参数，为用户提供更多功能。例如，`idp`表示身份提供者。

默认情况下，身份提供者是 Okta，但是我们可以使用`idp`参数对其进行定制:

```java
private OAuth2AuthorizationRequest customizeOktaReq(OAuth2AuthorizationRequest req) {
    Map<String,Object> extraParams = new HashMap<String,Object>();
    extraParams.putAll(req.getAdditionalParameters()); 
    extraParams.put("idp", "https://idprovider.com");
    return OAuth2AuthorizationRequest
      .from(req)
      .additionalParameters(extraParams)
      .build();
}
```

## 5。自定义令牌请求

现在，我们将了解如何定制 OAuth2 令牌请求。

**我们可以通过定制`OAuth2AccessTokenResponseClient`来定制令牌请求。**

`OAuth2AccessTokenResponseClient`的默认实现是`DefaultAuthorizationCodeTokenResponseClient`。

我们可以通过提供定制的`RequestEntityConverter`来定制令牌请求本身，我们甚至可以通过定制`DefaultAuthorizationCodeTokenResponseClient` `RestOperations`来定制令牌响应处理:

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.tokenEndpoint()
          .accessTokenResponseClient(accessTokenResponseClient())
            //...
    }

    @Bean
    public OAuth2AccessTokenResponseClient<OAuth2AuthorizationCodeGrantRequest> accessTokenResponseClient(){
        DefaultAuthorizationCodeTokenResponseClient accessTokenResponseClient = 
          new DefaultAuthorizationCodeTokenResponseClient(); 
        accessTokenResponseClient.setRequestEntityConverter(new CustomRequestEntityConverter()); 

        OAuth2AccessTokenResponseHttpMessageConverter tokenResponseHttpMessageConverter = 
          new OAuth2AccessTokenResponseHttpMessageConverter(); 
        tokenResponseHttpMessageConverter.setTokenResponseConverter(new CustomTokenResponseConverter()); 
        RestTemplate restTemplate = new RestTemplate(Arrays.asList(
          new FormHttpMessageConverter(), tokenResponseHttpMessageConverter)); 
        restTemplate.setErrorHandler(new OAuth2ErrorResponseErrorHandler()); 

        accessTokenResponseClient.setRestOperations(restTemplate); 
        return accessTokenResponseClient;
    }
}
```

**我们可以使用`tokenEndpoint().accessTokenResponseClient().`** 注入我们自己的`OAuth2AccessTokenResponseClient `

为了定制令牌请求参数，我们将实现`CustomRequestEntityConverter.` 。类似地，为了定制处理令牌响应，我们将实现`CustomTokenResponseConverter.`

我们将在接下来的章节中讨论`CustomRequestEntityConverter`和`CustomTokenResponseConverter`。

## 6。令牌请求额外参数

现在，我们将看到如何通过构建一个定制的`Converter`来向令牌请求添加额外的参数:

```java
public class CustomRequestEntityConverter implements 
  Converter<OAuth2AuthorizationCodeGrantRequest, RequestEntity<?>> {

    private OAuth2AuthorizationCodeGrantRequestEntityConverter defaultConverter;

    public CustomRequestEntityConverter() {
        defaultConverter = new OAuth2AuthorizationCodeGrantRequestEntityConverter();
    }

    @Override
    public RequestEntity<?> convert(OAuth2AuthorizationCodeGrantRequest req) {
        RequestEntity<?> entity = defaultConverter.convert(req);
        MultiValueMap<String, String> params = (MultiValueMap<String,String>) entity.getBody();
        params.add("test2", "extra2");
        return new RequestEntity<>(params, entity.getHeaders(), 
          entity.getMethod(), entity.getUrl());
    }

}
```

**我们的`Converter`将`OAuth2AuthorizationCodeGrantRequest`转换成一个`RequestEntity. `**

我们使用默认转换器`OAuth2AuthorizationCodeGrantRequestEntityConverter`来提供基本功能，并向`RequestEntity`主体添加额外的参数。

## 7。自定义令牌响应处理

现在，我们将定制处理令牌响应。

我们可以使用默认的令牌响应转换器 [`OAuth2AccessTokenResponseHttpMessageConverter`](https://web.archive.org/web/20220801021059/https://github.com/spring-projects/spring-security/blob/master/oauth2/oauth2-core/src/main/java/org/springframework/security/oauth2/core/http/converter/OAuth2AccessTokenResponseHttpMessageConverter.java#L134) 作为起点。

**我们将实现`CustomTokenResponseConverter`来不同地处理`“scope”`参数:**

```java
public class CustomTokenResponseConverter implements 
  Converter<Map<String, String>, OAuth2AccessTokenResponse> {
    private static final Set<String> TOKEN_RESPONSE_PARAMETER_NAMES = Stream.of(
        OAuth2ParameterNames.ACCESS_TOKEN, 
        OAuth2ParameterNames.TOKEN_TYPE, 
        OAuth2ParameterNames.EXPIRES_IN, 
        OAuth2ParameterNames.REFRESH_TOKEN, 
        OAuth2ParameterNames.SCOPE).collect(Collectors.toSet());

    @Override
    public OAuth2AccessTokenResponse convert(Map<String, String> tokenResponseParameters) {
        String accessToken = tokenResponseParameters.get(OAuth2ParameterNames.ACCESS_TOKEN);

        Set<String> scopes = Collections.emptySet();
        if (tokenResponseParameters.containsKey(OAuth2ParameterNames.SCOPE)) {
            String scope = tokenResponseParameters.get(OAuth2ParameterNames.SCOPE);
            scopes = Arrays.stream(StringUtils.delimitedListToStringArray(scope, ","))
                .collect(Collectors.toSet());
        }

        //...
        return OAuth2AccessTokenResponse.withToken(accessToken)
          .tokenType(accessTokenType)
          .expiresIn(expiresIn)
          .scopes(scopes)
          .refreshToken(refreshToken)
          .additionalParameters(additionalParameters)
          .build();
    }

}
```

令牌响应转换器将`Map`转换为`OAuth2AccessTokenResponse.`

在这个例子中，我们将`“scope”`参数解析为逗号分隔的参数，而不是空格分隔的参数`String.`

让我们通过使用 LinkedIn 作为授权服务器定制令牌响应来看另一个实际例子。

### 7.1。LinkedIn 令牌响应处理

最后，让我们看看如何处理 [LinkedIn](https://web.archive.org/web/20220801021059/https://docs.microsoft.com/en-us/linkedin/shared/authentication/token-introspection?tabs=http) 令牌响应。这仅包含`access_token`和`expires_in,`，但我们还需要`token_type.`

我们可以简单地实现我们自己的令牌响应转换器，并手动设置`token_type`:

```java
public class LinkedinTokenResponseConverter 
  implements Converter<Map<String, String>, OAuth2AccessTokenResponse> {

    @Override
    public OAuth2AccessTokenResponse convert(Map<String, String> tokenResponseParameters) {
        String accessToken = tokenResponseParameters.get(OAuth2ParameterNames.ACCESS_TOKEN);
        long expiresIn = Long.valueOf(tokenResponseParameters.get(OAuth2ParameterNames.EXPIRES_IN));

        OAuth2AccessToken.TokenType accessTokenType = OAuth2AccessToken.TokenType.BEARER;

        return OAuth2AccessTokenResponse.withToken(accessToken)
          .tokenType(accessTokenType)
          .expiresIn(expiresIn)
          .build();
    }
}
```

## 8。结论

在本文中，我们学习了如何通过添加或修改请求参数来定制 OAuth2 授权和令牌请求。

GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220801021059/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-oauth2)