

# OAuth2中的token处理源码解析

### TokenEndPoint

## 1.获取token

### 1.1请求/oauth/token

![image-20210526165251667](https://gitee.com/SaulZ/img/raw/master/img/image-20210526165251667.png)

### 1.2过滤器AbstractAuthenticationProcessingFilter

发出请求后会经过这个过滤器`AbstractAuthenticationProcessingFilter`，主要作用是**验证当前请求是不是请求token以及client_id和client_secret是否正确，最后生成Authentication对象**

```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
    throws IOException, ServletException {

    HttpServletRequest request = (HttpServletRequest) req;
    HttpServletResponse response = (HttpServletResponse) res;

    // 判断该请求是不是身份验证的请求，oauth/token
    if (!requiresAuthentication(request, response)) {
        chain.doFilter(request, response);
        return;
    }

    Authentication authResult;
    try {
        // 获取认证结果对象Authentication。
        //主要是根据client_id从oauth_client_detils读取记录,验证client_secret。Authentication保存了这条记录的部分信息
        authResult = attemptAuthentication(request, response);
        if (authResult == null) {
            // return immediately as subclass has indicated that it hasn't completed authentication
            return;
        }
        sessionStrategy.onAuthentication(authResult, request, response);
    }
    catch (InternalAuthenticationServiceException failed) {
        logger.error(
            "An internal error occurred while trying to authenticate the user.",
            failed);
        unsuccessfulAuthentication(request, response, failed);

        return;
    }
    catch (AuthenticationException failed) {
        // Authentication failed
        unsuccessfulAuthentication(request, response, failed);
        return;
    }

    // Authentication success
    if (continueChainBeforeSuccessfulAuthentication) {
        chain.doFilter(request, response);
    }
    successfulAuthentication(request, response, chain, authResult);
}
```

### 1.3处理请求

通过所有过滤器后

```java
@RequestMapping(value = "/oauth/token", method=RequestMethod.POST)
public ResponseEntity<OAuth2AccessToken> postAccessToken(Principal principal, @RequestParam
                                                         Map<String, String> parameters) throws HttpRequestMethodNotSupportedException {
    if (!(principal instanceof Authentication)) {
        throw new InsufficientAuthenticationException(
            "There is no client authentication. Try adding an appropriate authentication filter.");
    }
    // 获取到参数中的client_id
    String clientId = getClientId(principal);
    ClientDetails authenticatedClient = getClientDetailsService().loadClientByClientId(clientId);
    // tokenRequest封装了当前请求的所有参数
    TokenRequest tokenRequest = getOAuth2RequestFactory().createTokenRequest(parameters, authenticatedClient);

    if (clientId != null && !clientId.equals("")) {
        // Only validate the client details if a client authenticated during this request.
        if (!clientId.equals(tokenRequest.getClientId())) {
            // double check to make sure that the client ID in the token request is the same as that in the
            // authenticated client
            throw new InvalidClientException("Given client ID does not match authenticated client");
        }
    }
    if (authenticatedClient != null) {
        oAuth2RequestValidator.validateScope(tokenRequest, authenticatedClient);
    }
    if (!StringUtils.hasText(tokenRequest.getGrantType())) {
        throw new InvalidRequestException("Missing grant type");
    }
    if (tokenRequest.getGrantType().equals("implicit")) {
        throw new InvalidGrantException("Implicit grant type not supported from token endpoint");
    }

    if (isAuthCodeRequest(parameters)) {
        // The scope was requested or determined during the authorization step
        if (!tokenRequest.getScope().isEmpty()) {
            logger.debug("Clearing scope of incoming token request");
            tokenRequest.setScope(Collections.<String> emptySet());
        }
    }

    if (isRefreshTokenRequest(parameters)) {
        // A refresh token has its own default scopes, so we should ignore any added by the factory here.
        tokenRequest.setScope(OAuth2Utils.parseParameterList(parameters.get(OAuth2Utils.SCOPE)));
    }
    //getTokenGranter()返回TokenGranter，是一个接口，有一个grant()方法
    // grant()验证身份发放token，这里的逻辑见下边1.4 TokenGranter
    OAuth2AccessToken token = getTokenGranter().grant(tokenRequest.getGrantType(), tokenRequest);
    if (token == null) {
        throw new UnsupportedGrantTypeException("Unsupported grant type: " + tokenRequest.getGrantType());
    }

    return getResponse(token);
}
```

### 1.4TokenGranter

```java
public interface TokenGranter {
  OAuth2AccessToken grant(String var1, TokenRequest var2);
}
```

TokenGranter, 字面上的理解: 令牌授予者。 以下是各授权模式对应的 TokenGranter，它们都实现了TokenGranter

| 实现类                            | 对应的授权模式          |
| :-------------------------------- | :---------------------- |
| AuthorizationCodeTokenGranter     | 授权码模式              |
| ClientCredentialsTokenGranter     | 客户端模式              |
| ImplicitTokenGranter              | implicit 模式           |
| RefreshTokenGranter               | 刷新 token 模式         |
| ResourceOwnerPasswordTokenGranter | 密码模式                |
| CompositeTokenGranter             | 包括以上5种基本授权模式 |

#### 1.4.1 CompositeTokenGranter

```java
// 只保留了核心代码
public class CompositeTokenGranter implements TokenGranter {
	private final List<TokenGranter> tokenGranters;

    // tokenGranters中包含5种基本授权模式，会根据我们请求中的grant_type来判断使用哪种授权模式，本例中使用的grant_type是password，所以会使用ResourceOwnerPasswordTokenGranter
	public OAuth2AccessToken grant(String grantType, TokenRequest tokenRequest) {
		for (TokenGranter granter : tokenGranters) {
			OAuth2AccessToken grant = granter.grant(grantType, tokenRequest);
			if (grant!=null) {
				return grant;
			}
		}
		return null;
	}
}
```

#### 1.4.2 AbstractTokenGranter

ResourceOwnerPasswordTokenGranter继承了AbstractTokenGranter，而AbstractTokenGranter实现了TokenGranter

ResourceOwnerPasswordTokenGranter是没有重写grant()方法的，AbstractTokenGranter实现了grant()方法，所以上边1.3中调用的grant()是调用AbstractTokenGranter中的grant()方法

```java
// AbstractTokenGranter实现的grant()方法
public OAuth2AccessToken grant(String grantType, TokenRequest tokenRequest) {
		if (!this.grantType.equals(grantType)) {
			return null;
		}
		String clientId = tokenRequest.getClientId();
		ClientDetails client = clientDetailsService.loadClientByClientId(clientId);
		validateGrantType(grantType, client);

		if (logger.isDebugEnabled()) {
			logger.debug("Getting access token for: " + clientId);
		}
		// 调用了getAccessToken()方法
		return getAccessToken(client, tokenRequest);

	}

	protected OAuth2AccessToken getAccessToken(ClientDetails client, TokenRequest tokenRequest) {
        // 调用了getOAuth2Authentication()方法，因为ResourceOwnerPasswordTokenGranter重写了这个方法，所以会调用ResourceOwnerPasswordTokenGranter中的getOAuth2Authentication()方法
		return tokenServices.createAccessToken(getOAuth2Authentication(client, tokenRequest));
	}
	//	这个方法被ResourceOwnerPasswordTokenGranter重写了
    protected OAuth2Authentication getOAuth2Authentication(ClientDetails client, TokenRequest tokenRequest) {
            OAuth2Request storedOAuth2Request = requestFactory.createOAuth2Request(client, tokenRequest);
            return new OAuth2Authentication(storedOAuth2Request, null);
     }
```



#### 1.4.2 ResourceOwnerPasswordTokenGranter

```java
//ResourceOwnerPasswordTokenGranter继承了AbstractTokenGranter
public class ResourceOwnerPasswordTokenGranter extends AbstractTokenGranter {

	private static final String GRANT_TYPE = "password";

	private final AuthenticationManager authenticationManager;

	public ResourceOwnerPasswordTokenGranter(AuthenticationManager authenticationManager,
			AuthorizationServerTokenServices tokenServices, ClientDetailsService clientDetailsService, OAuth2RequestFactory requestFactory) {
		this(authenticationManager, tokenServices, clientDetailsService, requestFactory, GRANT_TYPE);
	}

	protected ResourceOwnerPasswordTokenGranter(AuthenticationManager authenticationManager, AuthorizationServerTokenServices tokenServices,
			ClientDetailsService clientDetailsService, OAuth2RequestFactory requestFactory, String grantType) {
		super(tokenServices, clientDetailsService, requestFactory, grantType);
		this.authenticationManager = authenticationManager;
	}

    // 这个是主要方法，重写了AbstractTokenGranter中的getOAuth2Authentication()
	@Override
	protected OAuth2Authentication getOAuth2Authentication(ClientDetails client, TokenRequest tokenRequest) {

		Map<String, String> parameters = new LinkedHashMap<String, String>(tokenRequest.getRequestParameters());
        //username和password就是参数中传的admin@test.com和123456
		String username = parameters.get("username");
		String password = parameters.get("password");
		// Protect from downstream leaks of password
		parameters.remove("password");

		Authentication userAuth = new UsernamePasswordAuthenticationToken(username, password);
		((AbstractAuthenticationToken) userAuth).setDetails(parameters);
		try {
            // 判断用户账户密码是否正确，并给Authentication添加其他属性
			userAuth = authenticationManager.authenticate(userAuth);
		}
		catch (AccountStatusException ase) {
			//covers expired, locked, disabled cases (mentioned in section 5.2, draft 31)
			throw new InvalidGrantException(ase.getMessage());
		}
		catch (BadCredentialsException e) {
			// If the username/password are wrong the spec says we should send 400/invalid grant
			throw new InvalidGrantException(e.getMessage());
		}
		if (userAuth == null || !userAuth.isAuthenticated()) {
			throw new InvalidGrantException("Could not authenticate user: " + username);
		}
		
		OAuth2Request storedOAuth2Request = getRequestFactory().createOAuth2Request(client, tokenRequest);		
		return new OAuth2Authentication(storedOAuth2Request, userAuth);
	}
}
```



AuthorizationServerEndpointsConfigurer

### 1.5 保存Token

上边该验证已经验证了，该生成的也生成了，接下来主要任务就是生成token并把token保存下来了，可以保存到数据库(JdbcTokenSotre)，redis(RedisTokenStore)，内存中(InMemoryTokenStore)或者jwt(JwkTokenStore)

#### 1.5.1DefaultTokenServices

```java
@Transactional
	public OAuth2AccessToken createAccessToken(OAuth2Authentication authentication) throws AuthenticationException {

		OAuth2AccessToken existingAccessToken = tokenStore.getAccessToken(authentication);
		OAuth2RefreshToken refreshToken = null;
		if (existingAccessToken != null) {
			if (existingAccessToken.isExpired()) {
				if (existingAccessToken.getRefreshToken() != null) {
					refreshToken = existingAccessToken.getRefreshToken();
					// The token store could remove the refresh token when the
					// access token is removed, but we want to
					// be sure...
					tokenStore.removeRefreshToken(refreshToken);
				}
				tokenStore.removeAccessToken(existingAccessToken);
			}
			else {
				// Re-store the access token in case the authentication has changed
				tokenStore.storeAccessToken(existingAccessToken, authentication);
				return existingAccessToken;
			}
		}

		// Only create a new refresh token if there wasn't an existing one
		// associated with an expired access token.
		// Clients might be holding existing refresh tokens, so we re-use it in
		// the case that the old access token
		// expired.
		if (refreshToken == null) {
			refreshToken = createRefreshToken(authentication);
		}
		// But the refresh token itself might need to be re-issued if it has
		// expired.
		else if (refreshToken instanceof ExpiringOAuth2RefreshToken) {
			ExpiringOAuth2RefreshToken expiring = (ExpiringOAuth2RefreshToken) refreshToken;
			if (System.currentTimeMillis() > expiring.getExpiration().getTime()) {
				refreshToken = createRefreshToken(authentication);
			}
		}

		OAuth2AccessToken accessToken = createAccessToken(authentication, refreshToken);
        // 这里是保存的token，我们配置了使用redis保存token，所以会调用RedisTokenStore的storeAccessToken()方法
		tokenStore.storeAccessToken(accessToken, authentication);
		// In case it was modified
		refreshToken = accessToken.getRefreshToken();
		if (refreshToken != null) {
			tokenStore.storeRefreshToken(refreshToken, authentication);
		}
		return accessToken;

	}
```

```java
//OAuth2AccessToken实际包含的字段
public class DefaultOAuth2AccessToken implements Serializable, OAuth2AccessToken {

	private static final long serialVersionUID = 914967629530462926L;

	private String value;

	private Date expiration;

	private String tokenType = BEARER_TYPE.toLowerCase();

	private OAuth2RefreshToken refreshToken;

	private Set<String> scope;

	private Map<String, Object> additionalInformation = Collections.emptyMap();
}
```

redis保存token的代码就不贴了，无非就是往redis中添加几条记录，redis中保存的数据还是挺多的，具体见下表

| 前缀                 | 拼接部分                            | vlue                         | 备注                               |
| -------------------- | ----------------------------------- | ---------------------------- | ---------------------------------- |
| access:              | token值                             | OAuth2AccessToken的序列化    |                                    |
| auth:                | token值                             | OAuth2Authentication的序列化 |                                    |
| auth_to_access:      | (username，client_id和scope)的md5值 | OAuth2AccessToken的序列化    |                                    |
| uname_to_access:     | clientId:username                   | OAuth2AccessToken的序列化    | username为空时，拼接部分是clientId |
| client_id_to_access: | clientId                            | OAuth2AccessToken的序列化    |                                    |
| refresh_to_access:   | refreshToken值                      | token序列化                  |                                    |
| access_to_refresh:   | token值                             | refreshToken序列化           |                                    |





