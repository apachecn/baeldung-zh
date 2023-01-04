# Spring Security OAuth 2——简单的令牌撤销(使用 Spring Security OAuth 遗留堆栈)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-oauth-revoke-tokens>

## 1。概述

在这个快速教程中，我们将说明如何撤销用`Spring Security`实现的`OAuth Authorization Server`授予的令牌。

当用户注销时，他们的令牌不会立即从令牌存储中删除；相反，它保持有效，直到它自己到期。

因此，令牌的撤销将意味着从令牌库中移除该令牌。我们将讨论框架中的标准令牌实现，而不是 JWT 令牌。

**注**:本文使用的是 [Spring OAuth 遗留项目](https://web.archive.org/web/20220703142311/https://spring.io/projects/spring-security-oauth)。

## 2。`TokenStore`

首先，让我们设置令牌存储；我们将使用一个`JdbcTokenStore`，以及附带的数据源:

```
@Bean 
public TokenStore tokenStore() { 
    return new JdbcTokenStore(dataSource()); 
}

@Bean 
public DataSource dataSource() { 
    DriverManagerDataSource dataSource =  new DriverManagerDataSource();
    dataSource.setDriverClassName(env.getProperty("jdbc.driverClassName"));
    dataSource.setUrl(env.getProperty("jdbc.url"));
    dataSource.setUsername(env.getProperty("jdbc.user"));
    dataSource.setPassword(env.getProperty("jdbc.pass")); 
    return dataSource;
}
```

## 3。`DefaultTokenServices`比恩

处理所有令牌的类是`DefaultTokenServices`–在我们的配置中必须定义为 bean:

```
@Bean
@Primary
public DefaultTokenServices tokenServices() {
    DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
    defaultTokenServices.setTokenStore(tokenStore());
    defaultTokenServices.setSupportRefreshToken(true);
    return defaultTokenServices;
}
```

## 4。显示令牌列表

出于管理目的，让我们也设置一种方法来查看当前有效的令牌。

我们将访问控制器中的`TokenStore`,并为指定的客户端 id 检索当前存储的令牌:

```
@Resource(name="tokenStore")
TokenStore tokenStore;

@RequestMapping(method = RequestMethod.GET, value = "/tokens")
@ResponseBody
public List<String> getTokens() {
    List<String> tokenValues = new ArrayList<String>();
    Collection<OAuth2AccessToken> tokens = tokenStore.findTokensByClientId("sampleClientId"); 
    if (tokens!=null){
        for (OAuth2AccessToken token:tokens){
            tokenValues.add(token.getValue());
        }
    }
    return tokenValues;
}
```

## 5。撤销访问令牌

为了使令牌无效，我们将利用来自`ConsumerTokenServices`接口的`revokeToken()` API:

```
@Resource(name="tokenServices")
ConsumerTokenServices tokenServices;

@RequestMapping(method = RequestMethod.POST, value = "/tokens/revoke/{tokenId:.*}")
@ResponseBody
public String revokeToken(@PathVariable String tokenId) {
    tokenServices.revokeToken(tokenId);
    return tokenId;
}
```

当然，这是一个非常敏感的操作，所以我们应该只在内部使用它，或者我们应该非常小心地公开它，并采取适当的安全措施。

## 6。前端

对于我们示例的前端，我们将显示有效令牌的列表、发出撤销请求的登录用户当前使用的令牌，以及用户可以输入他们希望撤销的令牌的字段:

```
$scope.revokeToken = 
  $resource("http://localhost:8082/spring-security-oauth-resource/tokens/revoke/:tokenId",
  {tokenId:'@tokenId'});
$scope.tokens = $resource("http://localhost:8082/spring-security-oauth-resource/tokens");

$scope.getTokens = function(){
    $scope.tokenList = $scope.tokens.query();	
}

$scope.revokeAccessToken = function(){
    if ($scope.tokenToRevoke && $scope.tokenToRevoke.length !=0){
        $scope.revokeToken.save({tokenId:$scope.tokenToRevoke});
        $rootScope.message="Token:"+$scope.tokenToRevoke+" was revoked!";
        $scope.tokenToRevoke="";
    }
}
```

如果用户试图再次使用已吊销的令牌，他们将收到状态代码为 401 的“无效令牌”错误。

## 7。撤销刷新令牌

刷新令牌可用于获得新的访问令牌。每当访问令牌被撤销时，与其一起接收的刷新令牌被无效。

如果我们也想使刷新令牌本身无效，我们可以使用类`JdbcTokenStore`的方法`removeRefreshToken()`，它将从存储中删除刷新令牌:

```
@RequestMapping(method = RequestMethod.POST, value = "/tokens/revokeRefreshToken/{tokenId:.*}")
@ResponseBody
public String revokeRefreshToken(@PathVariable String tokenId) {
    if (tokenStore instanceof JdbcTokenStore){
        ((JdbcTokenStore) tokenStore).removeRefreshToken(tokenId);
    }
    return tokenId;
}
```

为了测试刷新令牌在被撤销后是否不再有效，我们将编写以下测试，其中我们获取一个访问令牌，刷新它，然后删除刷新令牌，并尝试再次刷新它。

我们将看到，在撤销之后，我们将收到响应错误:“无效的刷新令牌”:

```
public class TokenRevocationLiveTest {
    private String refreshToken;

    private String obtainAccessToken(String clientId, String username, String password) {
        Map<String, String> params = new HashMap<String, String>();
        params.put("grant_type", "password");
        params.put("client_id", clientId);
        params.put("username", username);
        params.put("password", password);

        Response response = RestAssured.given().auth().
          preemptive().basic(clientId,"secret").and().with().params(params).
          when().post("http://localhost:8081/spring-security-oauth-server/oauth/token");
        refreshToken = response.jsonPath().getString("refresh_token");

        return response.jsonPath().getString("access_token");
    }

    private String obtainRefreshToken(String clientId) {
        Map<String, String> params = new HashMap<String, String>();
        params.put("grant_type", "refresh_token");
        params.put("client_id", clientId);
        params.put("refresh_token", refreshToken);

        Response response = RestAssured.given().auth()
          .preemptive().basic(clientId,"secret").and().with().params(params)
          .when().post("http://localhost:8081/spring-security-oauth-server/oauth/token");

        return response.jsonPath().getString("access_token");
    }

    private void authorizeClient(String clientId) {
        Map<String, String> params = new HashMap<String, String>();
        params.put("response_type", "code");
        params.put("client_id", clientId);
        params.put("scope", "read,write");

        Response response = RestAssured.given().auth().preemptive()
          .basic(clientId,"secret").and().with().params(params).
          when().post("http://localhost:8081/spring-security-oauth-server/oauth/authorize");
    }

    @Test
    public void givenUser_whenRevokeRefreshToken_thenRefreshTokenInvalidError() {
        String accessToken1 = obtainAccessToken("fooClientIdPassword", "john", "123");
        String accessToken2 = obtainAccessToken("fooClientIdPassword", "tom", "111");
        authorizeClient("fooClientIdPassword");

        String accessToken3 = obtainRefreshToken("fooClientIdPassword");
        authorizeClient("fooClientIdPassword");
        Response refreshTokenResponse = RestAssured.given().
          header("Authorization", "Bearer " + accessToken3)
          .get("http://localhost:8082/spring-security-oauth-resource/tokens");
        assertEquals(200, refreshTokenResponse.getStatusCode());

        Response revokeRefreshTokenResponse = RestAssured.given()
          .header("Authorization", "Bearer " + accessToken1)
          .post("http://localhost:8082/spring-security-oauth-resource/tokens/revokeRefreshToken/"+refreshToken);
        assertEquals(200, revokeRefreshTokenResponse.getStatusCode());

        String accessToken4 = obtainRefreshToken("fooClientIdPassword");
        authorizeClient("fooClientIdPassword");
        Response refreshTokenResponse2 = RestAssured.given()
          .header("Authorization", "Bearer " + accessToken4)
          .get("http://localhost:8082/spring-security-oauth-resource/tokens");
        assertEquals(401, refreshTokenResponse2.getStatusCode());
    }
}
```

## 8。结论

在本教程中，我们演示了如何撤销 Oauth 访问令牌和 OAuth 刷新令牌。

本教程的实现可以在[GitHub 项目](https://web.archive.org/web/20220703142311/https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-legacy "The Full Registration Example Project on Github ")中找到。