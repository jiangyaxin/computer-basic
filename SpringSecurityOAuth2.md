# OAuth2协议

## 简介

在传统的客户端-服务器身份验证模式中，客户端请求服务器上限制访问的资源（受保护资源）时，需要使用资源所有者的凭据在服务器上进行身份验证。

资源所有者为了给第三方应用提供受限资源的访问，需要与第三方共享它的凭据。 这造成一些问题和局限：

- 需要第三方应用存储资源所有者的凭据，以供将来使用，通常是明文密码。
- 需要服务器支持密码身份认证，尽管密码认证天生就有安全缺陷。
- 第三方应用获得的资源所有者的受保护资源的访问权限过于宽泛，从而导致资源所有者失去对资源使用时限或使用范围的控制。
- 资源所有者不能仅撤销某个第三方的访问权限而不影响其它，并且，资源所有者只有通过改变第三方的密码，才能单独撤销这第三方的访问权限。
- 与任何第三方应用的让步导致对终端用户的密码及该密码所保护的所有数据的让步。

OAuth2通过引入授权层以及分离客户端角色和资源所有者角色来解决这些问题：

- 客户端在请求受资源所有者控制并托管在资源服务器上的资源的访问权限时，将被授权层颁发一组不同于资源所有者所拥有凭据的令牌。
- 客户端获得一个访问令牌（一个代表特定作用域、生命期以及其他访问属性的字符串），用以代替使用资源所有者的凭据来访问受保护资源。
- 访问令牌由授权服务器在资源所有者认可的情况下颁发给第三方客户端。
- 客户端使用访问令牌访问托管在资源服务器的受保护资源。

## 角色

OAuth定义了四种角色：

- 资源所有者

  能够许可受保护资源访问权限的实体。当资源所有者是个人时，它作为最终用户被提及。
- 资源服务器

  托管受保护资源的服务器，能够接收和响应使用访问令牌对受保护资源的请求。
- 客户端

  使用资源所有者的授权代表资源所有者发起对受保护资源的请求的应用程序。
- 授权服务器

  在成功验证资源所有者且获得授权后颁发访问令牌给客户端的服务器。

## 协议流程

```java
 +--------+                               +---------------+
 |        |--(A)- Authorization Request ->|   Resource    |
 |        |                               |     Owner     |
 |        |<-(B)-- Authorization Grant ---|               |
 |        |                               +---------------+
 |        |
 |        |                               +---------------+
 |        |--(C)-- Authorization Grant -->| Authorization |
 | Client |                               |     Server    |
 |        |<-(D)----- Access Token -------|               |
 |        |                               +---------------+
 |        |
 |        |                               +---------------+
 |        |--(E)----- Access Token ------>|    Resource   |
 |        |                               |     Server    |
 |        |<-(F)--- Protected Resource ---|               |
 +--------+                               +---------------+
```

如上所示的抽象OAuth 2.0流程描述了四个角色之间的交互，包括以下步骤：

- （A）客户端向从资源所有者请求授权；授权请求可以直接向资源所有者发起；或者更可取的是通过作为中介的授权服务器间接发起。
- （B）客户端收到授权许可，这是一个代表资源所有者的授权的凭据。
- （C）客户端与授权服务器进行身份认证并出示授权许可请求访问令牌。
- （D）授权服务器验证客户端身份并验证授权许可，若有效则颁发访问令牌。
- （E）客户端从资源服务器请求受保护资源并出示访问令牌进行身份验证。
- （F）资源服务器验证访问令牌，若有效则满足该请求。

## 授权模式

OAuth2.0有4种授权模式：

- 授权码模式（authorization code）
- 简化模式（implicit）
- 密码模式（resource owner password credentials）
- 客户端模式（client credentials）

### 授权码模式（authorization code）

![authorization-code](assets/authorization-code.png)

> 举例说明：
> Resource-Owner => 用户
> User-Agent => 浏览器
> Client => 网站A
> Authorization-Server => QQ认证服务器
>
> 用户(Resource-Owner)在浏览器(User-Agent)上访问网站A(Client)，并使用QQ账号登陆(Authorization-Server)。

第一步（A）：

首先网站A登陆页面提供QQ登陆按钮，再由用户点击跳转到QQ登陆页面。下面就是一个跳转的示意链接：

```http
https://QQ.com/authorize?response_type=code&client_id=CLIENT_ID&redirect_uri=CALLBACK_URL&scope=read
```

上述的URL中有四个参数：

response_type：响应结果类型，code表示要求返回授权码。

client_id：客户端ID，值是网站A在授权服务器注册时的备案ID，由授权服务器提供，用来让授权服务器知道是谁在请求。

redirect_uri：授权服务器接受或者拒绝请求后跳转的网址，需要在授权服务器备案。

scope：表示要求授权的范围。

第二步（B-C）：

跳转到QQ登录页面后，QQ要求用户登录，并询问是否同意给网站A授权；如果登录成功并成功授权，则由登录响应重定向回第一步中设定的`CALLBACK_URL`，并携带一个授权码带回。如下所示：

```http
https://A.com/callback?code=AUTHORIZATION_CODE
```

其中，AUTHORIZATION_CODE就是授权码。

第三步（C-D-E）：

`CALLBACK_URL`页面拿到授权码后，请求网站A后端，并传递授权码，再由网站A后端发送链接给认证服务器，最后网站A后端拿到认证服务器返回的令牌，并返回给用户（浏览器）,然后跳转到用户需要访问的页面`redirect_uri`。发送链接如下所示:

```http
https://b.com/token?client_id=CLIENT_ID&client_secret=CLIENT_SECRET&grant_type=authorization_code&code=AUTHORIZATION_CODE&redirect_uri=CALLBACK_URL
```

上述的URL中有五个参数：

client_id：客户端ID

client_secret：客户端ID对应的密码，参数是保密的，因此只能在后端发请求。

grant_type：参数是AUTHORIZATION_CODE，表示采用的授权方式是授权码模式。

code：授权码，参数为第二步拿到的授权码。

redirect_uri: 拿到令牌后的回调地址，即为用户最开始需要访问的地址。

### 隐藏模式（implicit grant type）

![implicit-grant-type](assets/implicit-grant-type.png)

有些 Web 应用是纯前端应用，没有后端。这时就不能用上面的方式了，必须将令牌储存在前端。**RFC 6749 就规定了第二种方式，允许直接向前端颁发令牌。这种方式没有授权码这个中间步骤，所以称为（授权码）"隐藏式"（implicit）。**

第一步（A）：

网站A提供一个链接，要求用户跳转到认证服务器登录页面，链接如下：

```http
https://b.com/authorize?response_type=token&client_id=CLIENT_ID&redirect_uri=CALLBACK_URL&scope=read
```

上述的URL中有四个参数：

response_type：响应结果类型，token表示要求直接返回令牌。

client_id：客户端ID，值是网站A在授权服务器注册时的备案ID，由授权服务器提供，用来让授权服务器知道是谁在请求。

redirect_uri：授权服务器接受或者拒绝请求后跳转的网址，需要在授权服务器备案。

scope：表示要求授权的范围。

第二步（B-C）：

用户登录，同意给网站A授权，认证服务器登录响应返回，重定向到`CALLBACK_URL`并携带令牌返回。重定向链接如下：

```http
https://a.com/callback#token=ACCESS_TOKEN
```

值得注意的是`#`，这表示`token` 是以 URL 锚点（fragment）返回的，而不是参数，这是因为 OAuth 2.0 允许跳转网址是 HTTP 协议，因此存在"中间人攻击"的风险，而浏览器跳转时，锚点不会发到服务器，就减少了泄漏令牌的风险。

这种方式把令牌直接传给前端，是很不安全的。因此，只能用于一些安全要求不高的场景，并且令牌的有效期必须非常短，通常就是会话期间（session）有效，浏览器关掉，令牌就失效了

### 密码模式（resource owner password credentials）

![resource-owner-password-credentials](assets/resource-owner-password-credentials.png)

**如果你高度信任某个应用，RFC 6749 也允许用户把用户名和密码，直接告诉该应用。该应用就使用你的密码，申请令牌，这种方式称为"密码式"（password）。**

第一步（A-B）：

网站A要求用户提供 认证服务器的用户名和密码。拿到以后，网站A 就直接向认证服务器请求令牌。

```http
https://b.com/token?grant_type=password&username=USERNAME&password=PASSWORD&client_id=CLIENT_ID
```

上述的URL中有四个参数：

grant_type：授权类型，password表示密码模式。

username：用户名。

password：用户名对应的密码。

client_id：客户端ID，值是网站A在授权服务器注册时的备案ID，由授权服务器提供，用来让授权服务器知道是谁在请求。

第二步（C）:
认证服务器验证身份通过后，直接给出令牌。注意，这时不需要跳转，而是把令牌放在 JSON 数据里面，作为 HTTP 回应，网站A 因此拿到令牌。

这种方式需要用户给出自己的用户名/密码，显然风险很大，因此只适用于其他授权方式都无法采用的情况，而且必须是用户高度信任的应用。

### 客户端模式（client credentials）

![client-credentials](assets/client-credentials.png)

客户端模式适用于没有前端的命令行应用。

第一步（A）：

网站A后台直接向认证服务器发送请求。链接如下：

```http
https://b.com/token?grant_type=client_credentials&client_id=CLIENT_ID&client_secret=CLIENT_SECRET
```

grant_type：授权类型，client_credentials表示客户端模式。

client_id：客户端ID，值是网站A在授权服务器注册时的备案ID，由授权服务器提供，用来让授权服务器知道是谁在请求。

client_secret：客户端ID对应的密码。

第一步（B）：

认证服务器通过后直接返回令牌。

## 令牌使用

网站A拿到令牌之后，就可以向资源服务器请求数据，此时，每个发送的请求都必须带有令牌。

具体的做法是在请求头里加上`Authorization`，令牌就放在这个头里面。

```shell
curl -H "Authorization: Bearer ACCESS_TOKEN" "https://api.a.com"
```

`ACCESS_TOKEN`就是拿到的令牌。

## 令牌更新

令牌的有效期到了，如果让用户重新走一遍上面的流程，再申请一个新的令牌，很可能体验不好，而且也没有必要。OAuth 2.0 允许用户自动更新令牌。

具体方法是，认证服务器颁发令牌的时候，一次性颁发两个令牌，一个用于获取数据`token`，另一个用于获取新的令牌`refresh token`。

令牌到期前，用户使用 `refresh token` 发一个请求，去更新令牌。请求如下：

```http
https://b.com/token?grant_type=refresh_token&client_id=CLIENT_ID&client_secret=CLIENT_SECRET&refresh_token=REFRESH_TOKEN
```

grant_type：授权类型，refresh_token表示要求更新令牌。

client_id：客户端ID，值是网站A在授权服务器注册时的备案ID，由授权服务器提供，用来让授权服务器知道是谁在请求。

client_secret：客户端ID对应的密码。

refresh_token：参数就是用于更新令牌的令牌。

## JWT令牌

JWT由三部分构成，结构为：

```properties
header.payload.signature
```

### 头（header）

`header`是JWT的元数据，主要描述它的基本信息，例如：

```json
{
  "typ": "JWT",
  "alg": "HS256"
}
```

typ：声明类型。

alg：声明加密的算法 。

然后使用Base64URL进行编码得到类似下面的字符串：

```properties
yJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

### 载荷（payload）

`payload`是关于实体(通常是用户)和其他数据的声明，声明有三种类型: registered, public 和 private。

- Registered claims : 这里有一组预定义的声明，它们不是强制的，但是推荐。比如：

  > iss (issuer)：签发人
  >
  > exp (expiration time)：过期时间
  >
  > sub (subject)：主题
  >
  > aud (audience)：受众
  >
  > nbf (Not Before)：生效时间
  >
  > iat (Issued At)：签发时间
  >
  > jti (JWT ID)：编号
  >
- Public claims : 可以随意定义。
- Private claims : 用于在同意使用它们的各方之间共享信息，并且不是注册的或公开的声明。

例如：

```json
 {
   "sub": "1234567890",
   "name": "chongchong",
   "admin": true
 }
```

然后对其使用Base64URL编码得到类似下面的字符串：

```properties
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ
```

### 签名（signature）

`signature` 部分是对前两部分的签名，防止数据篡改。

首先，需要指定一个密钥`secret`。这个密钥只有服务器才知道，不能泄露给用户。然后，使用 `header `里面指定的签名算法（默认是 `HMAC SHA256`），按照下面的公式产生签名:

```java
HMACSHA256(base64UrlEncode(header) + "." +base64UrlEncode(payload),secret)
```

最后签名以后，把`header`、`payload`、`signature` 三个部分拼成一个字符串，每个部分之间用"点"（.）分隔，就构成整个令牌。

# 登录

## 使用Google举例

1. 首先登陆`Google API Console`，获取` OAuth 2.0 credentials`,即`Client ID`和` Client Secret`。
2. 设置登陆重定向地址为`http://localhost:8080/login/oauth2/code/google`，默认`URI Template`为`{baseUrl}/login/oauth2/code/{registrationId}`.
3. 引入依赖。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

4. 配置 Security。

```java
@Configuration
public class Oauth2LoginConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .sessionManagement(sessionManagement -> sessionManagement.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                .csrf(AbstractHttpConfigurer::disable)
                .antMatcher("/**").authorizeRequests()
                .antMatchers("/index.html").permitAll()
                .anyRequest().authenticated()
                .and()
                .oauth2Login();
    }
}
```

5. 设置`application.yml`：

```yaml
spring:
 security:
   oauth2:
     client:
       registration:
         google:
           client-id: google-client-id
           client-secret: google-client-secret
```

该配置会注入到 OAuth2ClientProperties：

```java
@ConfigurationProperties(prefix = "spring.security.oauth2.client")
public class OAuth2ClientProperties implements InitializingBean {

	private final Map<String, Provider> provider = new HashMap<>();

	private final Map<String, Registration> registration = new HashMap<>();
```

## 核心接口

### ClientRegistration

`ClientRegistration`表示注册在`OAuth 2.0`或`OpenID Connect 1.0`服务器的客户端信息 。

```java
public final class ClientRegistration {
	private String registrationId;
	private String clientId;
	private String clientSecret;
	private ClientAuthenticationMethod clientAuthenticationMethod;
	private AuthorizationGrantType authorizationGrantType;
	private String redirectUriTemplate;
	private Set<String> scopes;
	private ProviderDetails providerDetails;
	private String clientName;

	public class ProviderDetails {
		private String authorizationUri;
		private String tokenUri;
		private UserInfoEndpoint userInfoEndpoint;
		private String jwkSetUri;
		private String issuerUri;
        private Map<String, Object> configurationMetadata;

		public class UserInfoEndpoint {
			private String uri;
            private AuthenticationMethod authenticationMethod;
			private String userNameAttributeName;

		}
	}
}
```

1. `registrationId`： `ClientRegistration` 的唯一`ID`。
2. `clientId`：该`Client`在`OAuth 2.0`或`OpenID Connect 1.0`申请备案的`ID`,比如在`Google API Console`申请的`ID`。
3. `clientSecret`：`clientId`对应的密码。
4. `clientAuthenticationMethod`：使用`Provider`对客户端进行身份验证的方法。支持`basic`、`post`、none。
5. `authorizationGrantType`：授权类型。支持`authorization_code`, `implicit`，`refresh_token`，`client_credentials`，`password`。
6. `redirectUriTemplate`：用户 在`Authorization Server`完成认证授权后被重定向的地址。默认的重定向URI模板是`{baseUrl}/login/oauth2/code/{registrationId}`。
7. `scopes`：授权请求流程中客户端请求的范围。
8. `clientName`：`Client`名称。
9. `authorizationUri`：`Authorization Server`的授权地址（`Authorization Endpoint`）。
10. `tokenUri`：`Authorization Server`的令牌地址（`Token Endpoint`）。
11. `jwkSetUri`：从`Authorization Server`检索`JWK`的地址，从而获得`JWS`和`UserInfo`。
12. `(userInfoEndpoint)uri`：用来获取已认证用户的属性。
13. `(userInfoEndpoint)authenticationMethod`：将`access token `发到`UserInfo Endpoint`使用的方法，支持`header`、`form`、`query`。
14. `userNameAttributeName`：`UserInfo Response`中返回的用户`ID`的`key`，例如`sub`、`id`。

`ClientRegistration`除了手动配置外还可以使用`Authorization Server`的`Metadata endpoint`进行自动配置。

例如:

```java
ClientRegistration clientRegistration =
    ClientRegistrations.fromIssuerLocation("https://idp.example.com/issuer").build();
```

#### CommonOAuth2Provider

`CommonOAuth2Provider`预定义 `Google`、`GitHub`、`Facebook`、 `Okta`的默认配置。

* 当配置  OAuth2ClientProperties 后，引入 spring-boot-starter-oauth2-client 会进行自动装配。
* OAuth2ClientRegistrationRepositoryConfiguration 利用 OAuth2ClientPropertiesRegistrationAdapter 通过 CommonOAuth2Provider 获取对应的配置。
* 然后保存在 InMemoryClientRegistrationRepository(继承ClientRegistrationRepository)。

### ClientRegistrationRepository

`ClientRegistrationRepository`是`ClientRegistration`储存库，默认是`InMemoryClientRegistrationRepository`，并且已经注册为`Bean`。

`spring.security.oauth2.client.registration.[registrationId]`下的属性将会自动绑定到`ClientRegistration`。

### OAuth2AuthorizedClient

描述一个用户的已经授权的客户端，可以理解为授权用户，包含用户的principalName、 accessToken、refreshToken、clientRegistration。

`OAuth2AuthorizedClient`是` an Authorized Client`。当用户给客户端授权时，客户端被认为已经被授权。

`OAuth2AuthorizedClient`将` OAuth2AccessToken`(或`OAuth2RefreshToken`)与`ClientRegistration`、用户相关联。

```java
public class OAuth2AuthorizedClient {
	private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;
	private final ClientRegistration clientRegistration;
	private final String principalName;
	private final OAuth2AccessToken accessToken;
	private final OAuth2RefreshToken refreshToken;
}
```

clientRegistrationId、principalName 可以唯一确定一个用户 OAuth2AuthorizedClient。

### OAuth2AuthorizedClientRepository / OAuth2AuthorizedClientService

用来储存已经授权的用户，即OAuth2AuthorizedClient。

`OAuth2AuthorizedClientService` 默认使用 `InMemoryOAuth2AuthorizedClientService`，也可以使用 `JdbcOAuth2AuthorizedClientService` ，需要创建表：

```sql
CREATE TABLE oauth2_authorized_client (
  client_registration_id varchar(100) NOT NULL,
  principal_name varchar(200) NOT NULL,
  access_token_type varchar(100) NOT NULL,
  access_token_value blob NOT NULL,
  access_token_issued_at timestamp NOT NULL,
  access_token_expires_at timestamp NOT NULL,
  access_token_scopes varchar(1000) DEFAULT NULL,
  refresh_token_value blob DEFAULT NULL,
  refresh_token_issued_at timestamp DEFAULT NULL,
  created_at timestamp DEFAULT CURRENT_TIMESTAMP NOT NULL,
  PRIMARY KEY (client_registration_id, principal_name)
);
```

`OAuth2AuthorizedClientRepository` 默认使用 `AuthenticatedPrincipalOAuth2AuthorizedClientRepository`。
`AuthenticatedPrincipalOAuth2AuthorizedClientRepository` 包含 `OAuth2AuthorizedClientService`，除此之外还有一个 `HttpSessionOAuth2AuthorizedClientRepository` ，用来保存匿名用户。

`OAuth2AuthorizedClientRepository`用来保存 `OAuth2AuthorizedClient`，本质上还是调用`OAuth2AuthorizedClientService`。

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnBean(ClientRegistrationRepository.class)
class OAuth2WebSecurityConfiguration {

	@Bean
	@ConditionalOnMissingBean
	OAuth2AuthorizedClientService authorizedClientService(ClientRegistrationRepository clientRegistrationRepository) {
		return new InMemoryOAuth2AuthorizedClientService(clientRegistrationRepository);
	}

	@Bean
	@ConditionalOnMissingBean
	OAuth2AuthorizedClientRepository authorizedClientRepository(OAuth2AuthorizedClientService authorizedClientService) {
		return new AuthenticatedPrincipalOAuth2AuthorizedClientRepository(authorizedClientService);
	}
}
```

`OAuth2AuthorizedClientService`在程序中调用示例:

```java
@Controller
public class OAuth2ClientController {

    @Autowired
    private OAuth2AuthorizedClientService authorizedClientService;

    @GetMapping("/")
    public String index(Authentication authentication) {
        OAuth2AuthorizedClient authorizedClient =
            this.authorizedClientService.loadAuthorizedClient("okta", authentication.getName());

        OAuth2AccessToken accessToken = authorizedClient.getAccessToken();

        ...

        return "index";
    }
}
```

### OAuth2AuthorizedClientManager / OAuth2AuthorizedClientProvider

OAuth2AuthorizedClientProvider 负责使用 OAuth2AuthorizationContext 请求认证服务器，并且对认证成功的用户创建 OAuth2AuthorizedClient 。
OAuth2AuthorizedClientProvider 有多种实现：AuthorizationCodeOAuth2AuthorizedClientProvider 、 ClientCredentialsOAuth2AuthorizedClientProvider 、PasswordOAuth2AuthorizedClientProvider、RefreshTokenOAuth2AuthorizedClientProvider 对应 OAuth2 不同的认证模式。
除此之外还有 DelegatingOAuth2AuthorizedClientProvider，它可以完成对多种 OAuth2AuthorizedClientProvider 的代理。

`OAuth2AuthorizedClientProvider`实现` OAuth 2.0 Client`，有不同的种类，例如`authorization_code`, `client_credentials`等。可以使用`OAuth2AuthorizedClientProviderBuilder`来构建`DelegatingOAuth2AuthorizedClientProvider`。

例如：

```java
@Bean
public OAuth2AuthorizedClientManager authorizedClientManager(
		ClientRegistrationRepository clientRegistrationRepository,
		OAuth2AuthorizedClientRepository authorizedClientRepository) {

	OAuth2AuthorizedClientProvider authorizedClientProvider =
			OAuth2AuthorizedClientProviderBuilder.builder()
					.authorizationCode()
					.refreshToken()
					.clientCredentials()
					.password()
					.build();

	DefaultOAuth2AuthorizedClientManager authorizedClientManager =
			new DefaultOAuth2AuthorizedClientManager(
					clientRegistrationRepository, authorizedClientRepository);
	authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);

	return authorizedClientManager;
}
```

`OAuth2AuthorizedClientManager`：`DefaultOAuth2AuthorizedClientManager`是它的默认实现，负责利用 OAuth2AuthorizeRequest 构建 OAuth2AuthorizationContext，并使用`DelegatingOAuth2AuthorizedClientProvider` 完成认证，并将 OAuth2AuthorizedClient 保存在 OAuth2AuthorizedClientService 中 。

主要功能包括：

- 使用`OAuth2AuthorizedClientProvider`授权客户端。
- 使用`OAuth2AuthorizedClientService`持久化 `OAuth2AuthorizedClient`。
- 授权成功后调用` OAuth2AuthorizationSuccessHandler`。
- 授权失败后调用`OAuth2AuthorizationFailureHandler`。

当授权成功后，`DefaultOAuth2AuthorizedClientManager `调用`OAuth2AuthorizationSuccessHandler`，`OAuth2AuthorizationSuccessHandler`使用`OAuth2AuthorizedClientRepository`保存`OAuth2AuthorizedClient`，当重新授权失败(例如`refresh token`失效)后，`RemoveAuthorizedClientOAuth2AuthorizationFailureHandler`会把`OAuth2AuthorizedClient`从`OAuth2AuthorizedClientRepository`删除。

`DefaultOAuth2AuthorizedClientManager `另外还包含`Function<OAuth2AuthorizeRequest, Map<String, Object>>`类型的`contextAttributesMapper `，它负责将属性从`OAuth2AuthorizeRequest `映射到`OAuth2AuthorizationContext `。当使用`OAuth2AuthorizedClientProvider`时`OAuth2AuthorizationContext `会很有用，例如`PasswordOAuth2AuthorizedClientProvider`通过

`OAuth2AuthorizationContext.getAttributes()`获取`username` 、`password` ：

```java
public OAuth2AuthorizedClientManager authorizedClientManager(
		ClientRegistrationRepository clientRegistrationRepository,
		OAuth2AuthorizedClientRepository authorizedClientRepository) {

	OAuth2AuthorizedClientProvider authorizedClientProvider =
			OAuth2AuthorizedClientProviderBuilder.builder()
					.password()
					.refreshToken()
					.build();

	DefaultOAuth2AuthorizedClientManager authorizedClientManager =
			new DefaultOAuth2AuthorizedClientManager(
					clientRegistrationRepository, authorizedClientRepository);
	authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);

	// Assuming the `username` and `password` are supplied as `HttpServletRequest` parameters,
	// map the `HttpServletRequest` parameters to `OAuth2AuthorizationContext.getAttributes()`
	authorizedClientManager.setContextAttributesMapper(contextAttributesMapper());

	return authorizedClientManager;
}

private Function<OAuth2AuthorizeRequest, Map<String, Object>> contextAttributesMapper() {
	return authorizeRequest -> {
		Map<String, Object> contextAttributes = Collections.emptyMap();
		HttpServletRequest servletRequest = authorizeRequest.getAttribute(HttpServletRequest.class.getName());
		String username = servletRequest.getParameter(OAuth2ParameterNames.USERNAME);
		String password = servletRequest.getParameter(OAuth2ParameterNames.PASSWORD);
		if (StringUtils.hasText(username) && StringUtils.hasText(password)) {
			contextAttributes = new HashMap<>();

			// `PasswordOAuth2AuthorizedClientProvider` requires both attributes
			contextAttributes.put(OAuth2AuthorizationContext.USERNAME_ATTRIBUTE_NAME, username);
			contextAttributes.put(OAuth2AuthorizationContext.PASSWORD_ATTRIBUTE_NAME, password);
		}
		return contextAttributes;
	};
}
```

`DefaultOAuth2AuthorizedClientManager `主要用于操作`web HttpServletRequest`，如果是`HttpServletRequest`之外，改用`AuthorizedClientServiceOAuth2AuthorizedClientManager`，例如，程序内部一般在后台运行，没有和用户交互，`Client`使用`client_credentials`方式访问：

```java
@Bean
public OAuth2AuthorizedClientManager authorizedClientManager(
		ClientRegistrationRepository clientRegistrationRepository,
		OAuth2AuthorizedClientService authorizedClientService) {

	OAuth2AuthorizedClientProvider authorizedClientProvider =
			OAuth2AuthorizedClientProviderBuilder.builder()
					.clientCredentials()
					.build();

	AuthorizedClientServiceOAuth2AuthorizedClientManager authorizedClientManager =
			new AuthorizedClientServiceOAuth2AuthorizedClientManager(
					clientRegistrationRepository, authorizedClientService);
	authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);

	return authorizedClientManager;
}
```

DefaultOAuth2AuthorizedClientManager 用于 OAuth2AuthorizedClientArgumentResolver 处理 @RegisteredOAuth2AuthorizedClient 注解获取 OAuth2AuthorizedClient。

### DefaultAuthorizationCodeTokenResponseClient / DefaultRefreshTokenTokenResponseClient

`DefaultAuthorizationCodeTokenResponseClient`、`DefaultRefreshTokenTokenResponseClient`是授权码模式下 `OAuth2AccessTokenResponseClient` 的默认实现，使用 `RestOperations` 请求`Authorization Server Authorization Endpoint`（`authorization-uri`）获取`access token`。

`DefaultAuthorizationCodeTokenResponseClient`、`DefaultRefreshTokenTokenResponseClient`允许操作`the pre-processing of the Token Request `或`post-handling of the Token Response`。

### DefaultClientCredentialsTokenResponseClient

`DefaultClientCredentialsTokenResponseClient`是客户端模式下 `OAuth2AccessTokenResponseClient` 的默认实现。

### DefaultJwtBearerTokenResponseClient

`DefaultJwtBearerTokenResponseClient`是`JWT Bearer`模式下 `OAuth2AccessTokenResponseClient` 的默认实现。

## 启动

1. 通过使用 http.oauth2Login 创建 OAuth2LoginConfigurer 并配置 http 。
2. 自动装配。
   * SpringSecurity 主流程。
   * OAuth2ClientConfiguration：添加 OAuth2AuthorizedClientArgumentResolver 处理 @RegisteredOAuth2AuthorizedClient。
   * OAuth2ClientAutoConfiguration：添加 InMemoryClientRegistrationRepository 、OAuth2AuthorizedClientService、OAuth2AuthorizedClientRepository。

### OAuth2LoginConfigurer配置内容

继承 `AbstractAuthenticationFilterConfigurer<B, OAuth2LoginConfigurer<B>, OAuth2LoginAuthenticationFilter>`, 表明 它负责创建 OAuth2AuthorizationRequestRedirectFilter、OAuth2LoginAuthenticationFilter ，最后会将它添加到过滤器链，OAuth2LoginAuthenticationFilter 使用 LoginUrlAuthenticationEntryPoint 作为 LoginUrlAuthenticationEntryPoint。

可以通过 http.oauth2Login 配置 ClientRegistrationRepository、OAuth2AuthorizedClientRepository、OAuth2AuthorizedClientService(最后也是配置一个AuthenticatedPrincipalOAuth2AuthorizedClientRepository)。

### OAuth2AuthorizationRequestRedirectFilter

```java
public class OAuth2AuthorizationRequestRedirectFilter extends OncePerRequestFilter {
	public static final String DEFAULT_AUTHORIZATION_REQUEST_BASE_URI = "/oauth2/authorization";
	private final ThrowableAnalyzer throwableAnalyzer = new DefaultThrowableAnalyzer();
	private final RedirectStrategy authorizationRedirectStrategy = new DefaultRedirectStrategy();
	private OAuth2AuthorizationRequestResolver authorizationRequestResolver;
  // HttpSessionOAuth2AuthorizationRequestRepository 最好自己实现，不用session。
	private AuthorizationRequestRepository<OAuth2AuthorizationRequest> authorizationRequestRepository = new HttpSessionOAuth2AuthorizationRequestRepository();
	private RequestCache requestCache = new HttpSessionRequestCache();
}
```

主要功能：

负责完成 OAuth2 的第一步。

1. 负责拦截 authorizationRequestBaseUri(默认为 OAuth2AuthorizationRequestRedirectFilter.DEFAULT_AUTHORIZATION_REQUEST_BASE_URI,即 `/oauth2/authorization`) 前缀的请求,所以当使用第三方登陆按钮关联的连接应该是 /oauth2/authorization/{registrationId}
2. 使用 OAuth2AuthorizationRequestResolver(默认为 DefaultOAuth2AuthorizationRequestResolver) 从 request 中按照 `/oauth2/authorization/{registrationId}` 解析出 registrationId，再通过 registrationId 从  ClientRegistrationRepository 获取 ClientRegistration，最后使用 ClientRegistration 构建 OAuth2AuthorizationRequest，OAuth2AuthorizationRequest 包含重定向地址，例如：

```http
https://accounts.google.com/o/oauth2/v2/auth?response_type=code&client_id=137558705174-hgc0dv0rsd2mdu68c95jq98oialasv32.apps.googleusercontent.com&scope=openid%20profile%20email&state=nwFMHMpyu-4K5l6I4oHk5py7HGlmnfidi-s4PcPeWVk%3D&redirect_uri=http://localhost:30011/login/oauth2/code/google&nonce=OZ5cbAKOiOScnzEKb7km3od3rTrGBTCnCG7eIaZjpFI
```

即授权码模式的第一步。
3. OAuth2AuthorizationRequest 包含一个 32 位的随机state值， 将 OAuth2AuthorizationRequest 缓存在 HttpSessionOAuth2AuthorizationRequestRepository的map中，key 为 state，value 为 OAuth2AuthorizationRequest，state 会发送给 认证服务器，认证服务器认证完成之后会发送回来。
4. 使用 DefaultRedirectStrategy 将 OAuth2AuthorizationRequest 中重定向地址设置到 HttpServletResponse 中，然后返回给浏览器，将浏览器重定向到认证中心登陆页面。

**authorizationRequestBaseUri 通过 `http.oauth2Login(loginConfigurer -> loginConfigurer.authorizationEndpoint(authorizationEndpointConfig -> authorizationEndpointConfig.baseUri("xxxxx")))` 设置授权端点。**

### OAuth2AuthorizationRequestRedirectFilter

```java
public class OAuth2LoginAuthenticationFilter extends AbstractAuthenticationProcessingFilter {
	/**
	 * The default {@code URI} where this {@code Filter} processes authentication requests.
	 */
	public static final String DEFAULT_FILTER_PROCESSES_URI = "/login/oauth2/code/*";
	private static final String AUTHORIZATION_REQUEST_NOT_FOUND_ERROR_CODE = "authorization_request_not_found";
	private static final String CLIENT_REGISTRATION_NOT_FOUND_ERROR_CODE = "client_registration_not_found";
	private ClientRegistrationRepository clientRegistrationRepository;
	private OAuth2AuthorizedClientRepository authorizedClientRepository;
	private AuthorizationRequestRepository<OAuth2AuthorizationRequest> authorizationRequestRepository = new HttpSessionOAuth2AuthorizationRequestRepository();
}
```

主要功能：

负责完成 OAuth2 的第二步。

1. 负责拦截 filterProcessesUrl(默认为 OAuth2LoginAuthenticationFilter.DEFAULT_FILTER_PROCESSES_URI,即 `/login/oauth2/code/` ) 前缀的请求。

   * 所以 认证服务器注册的回调地址应该为 http://localhost:8080/login/oauth2/code/google ，这样认证服务器认证完成之后会回调该地址，然后被 OAuth2LoginAuthenticationFilter 拦截。
   * 可通过 `http..oauth2Login(httpSecurityOAuth2LoginConfigurer -> httpSecurityOAuth2LoginConfigurer.loginProcessingUrl("xxxxx"))` 修改。
2. 获取认证服务器回调地址中的参数，即 state、code(AUTHORIZATION_CODE 即授权码)、scope等。
3. 根据state从HttpSessionOAuth2AuthorizationRequestRepository 取 OAuth2AuthorizationRequest，这是在OAuth2AuthorizationRequestRedirectFilter 保存进去的。
4. 使用 clientRegistration 、authorizationRequest、 authorizationResponse 生成 OAuth2LoginAuthenticationToken,这时还没有 acessToken 和 refreshToken。

```java
public class OAuth2LoginAuthenticationToken extends AbstractAuthenticationToken {
	private OAuth2User principal;
	private ClientRegistration clientRegistration;
	private OAuth2AuthorizationExchange authorizationExchange;
	private OAuth2AccessToken accessToken;
	private OAuth2RefreshToken refreshToken;
}
```

4. 使用 AuthenticationManager 对 OAuth2LoginAuthenticationToken 进行认证，有两个AuthenticationProvider 支持OAuth2LoginAuthenticationToken：

* OAuth2LoginAuthenticationProvider 负责 scope 不包含 openid 的 request，即负责OAuth2的认证。

  1. 重新构建 OAuth2AuthorizationCodeAuthenticationToken 将其委托给 OAuth2AuthorizationCodeAuthenticationProvider 认证，最后由 DefaultAuthorizationCodeTokenResponseClient 向 tokenUri 发送请求，获取 accessToken。
  2. DefaultOAuth2UserService 使用 accessToken 向 userInfoUri 获取 DefaultOAuth2User。
  3. 构建 OAuth2LoginAuthenticationToken。
* OidcAuthorizationCodeAuthenticationProvider 负责 scope 包含 openid 的 request，即负责OCID的认证。

  1. 使用 DefaultAuthorizationCodeTokenResponseClient 发送请求，获取 Respose，请求实例：

  ```http
  # Post 请求
  https://www.googleapis.com/oauth2/v4/token
  # 表单参数
  grant_type -> authorization_code
  code -> 4/0AX4XfWhfwu31rRMjyTpqxDn7SeI9CiZ8HbL_WK2E0Edj7Eqt5l2a1b5Co7_Dw5sO97Zk8Q(收钱码)
  redirect_uri -> http://localhost:30011/login/oauth2/code/google
  ```

  2. 使用 JWT 解析Respose 中额外参数 id_token 构建 OidcIdToken。
  3. OidcUserService 加载用户信息、权限信息，构建OidcUser。
  4. 构建 OAuth2LoginAuthenticationToken。

4. 使用  OAuth2AuthorizedClientRepository 保存 OAuth2LoginAuthenticationToken 和  OAuth2AuthorizedClient。
5. 认证成功后 由 SavedRequestAwareAuthenticationSuccessHandler 重定向到 defaultTargetUrl，可通过 `http.oauth2Login(httpSecurityOAuth2LoginConfigurer -> httpSecurityOAuth2LoginConfigurer.defaultSuccessUrl("/user"))`设置。

流程中token区别：
* OAuth2LoginAuthenticationToken：用以给Provider认证过渡用，最初仅含code，最终包含access_token、user等。
* OAuth2AuthorizationCodeAuthenticationToken：用以给Provider认证过渡用，未填充时仅含code，经填充后包含access_token等。
* OAuth2AuthenticationToken：authenticated=true 认证后安全上下文实际保存的OAuth2用户认证，由convert将填充后的OAuth2LoginAuthenticationToken转换而来。

## Login授权流程

![106](assets/106.png)

1. 当 FilterSecurityInterceptor 鉴权失败时，由 ExceptionTranslationFilter 使用 LoginUrlAuthenticationEntryPoint  重定向到 loginFormUrl 默认为 /login,可以通过 `http.oauth2Login(httpSecurityOAuth2LoginConfigurer -> httpSecurityOAuth2LoginConfigurer.loginPage("xxxx"))` 或者 `http.formLogin(httpSecurityFormLoginConfigurer -> httpSecurityFormLoginConfigurer.loginPage("/2"))`更改。
2. 用户点击 google 登陆，发送请求`/oauth2/authorization/{registrationId}`。
3. 当 requet 可以匹配到`DEFAULT_AUTHORIZATION_REQUEST_BASE_URI = "/oauth2/authorization/{registrationId}"`时， OAuth2AuthorizationRequestRedirectFilter 会根据 registrationId 获得对应的 ClientRegistration ，然后构造出 OAuth2AuthorizationRequest 重定向到 authorizationUri 。
4. 填写用户名密码，认证通过`Authorization Server`回调备案地址(如上面例子中`http://localhost:8080/login/oauth2/code/google`)并携带`code`参数，**备案地址需要匹配 AbstractAuthenticationProcessingFilter 中`requiresAuthenticationRequestMatcher`，可以通过`defaultFilterProcessesUrl`修改，默认值为`DEFAULT_FILTER_PROCESSES_URI = "/login/oauth2/code/*"`。**
5. 由`OAuth2LoginAuthenticationFilter`拦截到`http://localhost:8080/login/oauth2/code/google`开始`POST`调用`tokenUri`并携带上一步得到的`code`参数，获取到`access_token`后重定向回最开始用户想访问的`URL`并设置`cookie`。
6. 然后带着`cookie`访问。

可使用 `http.sessionManagement(sessionManagement -> sessionManagement.sessionCreationPolicy(SessionCreationPolicy.STATELESS))`不使用session，这时携带cookie无法认证。

## ClientRegistration 属性配置


| Spring Boot 2.x                                                                              | ClientRegistration                                       |
| :--------------------------------------------------------------------------------------------- | :--------------------------------------------------------- |
| `spring.security.oauth2.client.registration.*[registrationId]*`                              | `registrationId`                                         |
| `spring.security.oauth2.client.registration.*[registrationId]*.client-id`                    | `clientId`                                               |
| `spring.security.oauth2.client.registration.*[registrationId]*.client-secret`                | `clientSecret`                                           |
| `spring.security.oauth2.client.registration.*[registrationId]*.client-authentication-method` | `clientAuthenticationMethod`                             |
| `spring.security.oauth2.client.registration.*[registrationId]*.authorization-grant-type`     | `authorizationGrantType`                                 |
| `spring.security.oauth2.client.registration.*[registrationId]*.redirect-uri`                 | `redirectUri`                                            |
| `spring.security.oauth2.client.registration.*[registrationId]*.scope`                        | `scopes`                                                 |
| `spring.security.oauth2.client.registration.*[registrationId]*.client-name`                  | `clientName`                                             |
| `spring.security.oauth2.client.provider.*[providerId]*.authorization-uri`                    | `providerDetails.authorizationUri`                       |
| `spring.security.oauth2.client.provider.*[providerId]*.token-uri`                            | `providerDetails.tokenUri`                               |
| `spring.security.oauth2.client.provider.*[providerId]*.jwk-set-uri`                          | `providerDetails.jwkSetUri`                              |
| `spring.security.oauth2.client.provider.*[providerId]*.issuer-uri`                           | `providerDetails.issuerUri`                              |
| `spring.security.oauth2.client.provider.*[providerId]*.user-info-uri`                        | `providerDetails.userInfoEndpoint.uri`                   |
| `spring.security.oauth2.client.provider.*[providerId]*.user-info-authentication-method`      | `providerDetails.userInfoEndpoint.authenticationMethod`  |
| `spring.security.oauth2.client.provider.*[providerId]*.user-name-attribute`                  | `providerDetails.userInfoEndpoint.userNameAttributeName` |

## 自定义 ClientRegistration 实例

```yaml
spring:
 security:
   oauth2:
     client:
       registration:
         okta:
           client-id: okta-client-id
           client-secret: okta-client-secret
       provider:
         okta:
           authorization-uri: https://your-subdomain.oktapreview.com/oauth2/v1/authorize
           token-uri: https://your-subdomain.oktapreview.com/oauth2/v1/token
           user-info-uri: https://your-subdomain.oktapreview.com/oauth2/v1/userinfo
           user-name-attribute: sub
           jwk-set-uri: https://your-subdomain.oktapreview.com/oauth2/v1/keys
```

## 覆盖自动装配

如果使用`CommonOAuth2Provider`，需要配置`ClientRegistrationRepository`，`httpSecurity.oauth2Login()`。

```java
@Configuration
public class OAuth2LoginConfig {

	@EnableWebSecurity
	public static class OAuth2LoginSecurityConfig extends WebSecurityConfigurerAdapter {

		@Override
		protected void configure(HttpSecurity http) throws Exception {
			http
				.authorizeHttpRequests(authorize -> authorize
					.anyRequest().authenticated()
				)
				.oauth2Login(withDefaults());
		}
	}

	@Bean
	public ClientRegistrationRepository clientRegistrationRepository() {
		return new InMemoryClientRegistrationRepository(this.googleClientRegistration());
	}

	private ClientRegistration googleClientRegistration() {
		return ClientRegistration.withRegistrationId("google")
			.clientId("google-client-id")
			.clientSecret("google-client-secret")
			.clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
			.authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
			.redirectUri("{baseUrl}/login/oauth2/code/{registrationId}")
			.scope("openid", "profile", "email", "address", "phone")
			.authorizationUri("https://accounts.google.com/o/oauth2/v2/auth")
			.tokenUri("https://www.googleapis.com/oauth2/v4/token")
			.userInfoUri("https://www.googleapis.com/oauth2/v3/userinfo")
			.userNameAttributeName(IdTokenClaimNames.SUB)
			.jwkSetUri("https://www.googleapis.com/oauth2/v3/certs")
			.clientName("Google")
			.build();
	}
}
```

如果无法使用`CommonOAuth2Provider`，配置:

```java
@Configuration
public class OAuth2LoginConfig {

	@EnableWebSecurity
	public static class OAuth2LoginSecurityConfig extends WebSecurityConfigurerAdapter {

		@Override
		protected void configure(HttpSecurity http) throws Exception {
			http
				.authorizeHttpRequests(authorize -> authorize
					.anyRequest().authenticated()
				)
				.oauth2Login(withDefaults());
		}
	}

	@Bean
	public ClientRegistrationRepository clientRegistrationRepository() {
		return new InMemoryClientRegistrationRepository(this.googleClientRegistration());
	}

	@Bean
	public OAuth2AuthorizedClientService authorizedClientService(
			ClientRegistrationRepository clientRegistrationRepository) {
		return new InMemoryOAuth2AuthorizedClientService(clientRegistrationRepository);
	}

	@Bean
	public OAuth2AuthorizedClientRepository authorizedClientRepository(
			OAuth2AuthorizedClientService authorizedClientService) {
		return new AuthenticatedPrincipalOAuth2AuthorizedClientRepository(authorizedClientService);
	}

	private ClientRegistration googleClientRegistration() {
		return CommonOAuth2Provider.GOOGLE.getBuilder("google")
			.clientId("google-client-id")
			.clientSecret("google-client-secret")
			.build();
	}
}
```

## 高级配置

`HttpSecurity.oauth2Login()`提供一些自定义配置，例如：

```java
@EnableWebSecurity
public class OAuth2LoginSecurityConfig extends WebSecurityConfigurerAdapter {

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			.oauth2Login(oauth2 -> oauth2
			    .clientRegistrationRepository(this.clientRegistrationRepository())
			    .authorizedClientRepository(this.authorizedClientRepository())
			    .authorizedClientService(this.authorizedClientService())
			    .loginPage("/login")
			    .authorizationEndpoint(authorization -> authorization
			        .baseUri(this.authorizationRequestBaseUri())
			        .authorizationRequestRepository(this.authorizationRequestRepository())
			        .authorizationRequestResolver(this.authorizationRequestResolver())
			    )
			    .redirectionEndpoint(redirection -> redirection
			        .baseUri(this.authorizationResponseBaseUri())
			    )
			    .tokenEndpoint(token -> token
			        .accessTokenResponseClient(this.accessTokenResponseClient())
			    )
			    .userInfoEndpoint(userInfo -> userInfo
			        .userAuthoritiesMapper(this.userAuthoritiesMapper())
			        .userService(this.oauth2UserService())
			        .oidcUserService(this.oidcUserService())
			    )
			);
	}
}
```

# 客户端

## 使用

1. 首先登陆`Google API Console`，获取` OAuth 2.0 credentials`,即`Client ID`和` Client Secret`。
2. 设置重定向地址 不能为`{baseUrl}/login/oauth2/code/{registrationId}`，否则会优先被 http.oauth2Login() 拦截，被 http.oauth2Client 该重定向地址会在认证成功后直接重定向到该地址
3. 引入依赖。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

4. 配置 Security。

```java
@Configuration
public class Oauth2ClientConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .csrf(csrf -> csrf.disable())
                .antMatcher("/**").authorizeRequests()
                .antMatchers("/index.html").permitAll()
                .anyRequest().authenticated()
                .and()
                .oauth2Client();
    }
}
```

5. 设置`application.yml`：

```yaml
spring:
 security:
   oauth2:
     client:
       registration:
         google:
           client-id: google-client-id
           client-secret: google-client-secret
           # 设置重定向地址，需要和备案重定向地址一样
           redirect-Uri: http://localhost:30010/index.html
```

## 启动

1. 通过使用 http.oauth2Client 创建 OAuth2ClientConfigurer 并配置 http 。
2. 自动装配。
   * SpringSecurity 主流程。
   * OAuth2ClientConfiguration：添加 OAuth2AuthorizedClientArgumentResolver 处理 @RegisteredOAuth2AuthorizedClient。
   * OAuth2ClientAutoConfiguration：添加 InMemoryClientRegistrationRepository 、OAuth2AuthorizedClientService、OAuth2AuthorizedClientRepository。

![107](assets/107.png)

### OAuth2ClientConfigurer配置内容

实际委托给 AuthorizationCodeGrantConfigurer 进行 init() 和 configure(), 负责创建 OAuth2AuthorizationRequestRedirectFilter、OAuth2AuthorizationCodeGrantFilter ，最后会将它添加到过滤器链，由 OAuth2AuthorizationCodeAuthenticationProvider 负责认证操作。

可以通过 http.oauth2Login 配置 ClientRegistrationRepository、OAuth2AuthorizedClientRepository、OAuth2AuthorizedClientService(最后也是配置一个AuthenticatedPrincipalOAuth2AuthorizedClientRepository)。

### OAuth2AuthorizationRequestRedirectFilter

详情见登录流程

### OAuth2AuthorizationCodeGrantFilter

1. 通过 state 从 HttpSessionOAuth2AuthorizationRequestRepository 获取 OAuth2AuthorizationRequest。
2. 负责拦截 OAuth2AuthorizationRequest#getRedirectUri() 的请求，即 spring.security.oauth2.client.registration.google.redirect-Uri: http://localhost:30010/index.html
3. 通过授权码获取请求认证服务器获取 accessToken，然后重定向到 redirectUri。

## 配置示例

```java
@EnableWebSecurity
public class OAuth2ClientSecurityConfig extends WebSecurityConfigurerAdapter {

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			.oauth2Client(oauth2 -> oauth2
				.clientRegistrationRepository(this.clientRegistrationRepository())
				.authorizedClientRepository(this.authorizedClientRepository())
				.authorizedClientService(this.authorizedClientService())
				.authorizationCodeGrant(codeGrant -> codeGrant
					.authorizationRequestRepository(this.authorizationRequestRepository())
					.authorizationRequestResolver(this.authorizationRequestResolver())
					.accessTokenResponseClient(this.accessTokenResponseClient())
				)
			);
	}
}
```

`OAuth2AuthorizedClientManager`负责协调一个或多个`OAuth2AuthorizedClientProvider`。

例如：

```java
@Bean
public OAuth2AuthorizedClientManager authorizedClientManager(
		ClientRegistrationRepository clientRegistrationRepository,
		OAuth2AuthorizedClientRepository authorizedClientRepository) {
	// 配置多个OAuth2AuthorizedClientProvider
	OAuth2AuthorizedClientProvider authorizedClientProvider =
			OAuth2AuthorizedClientProviderBuilder.builder()
        			 //authorization_code
					.authorizationCode()
        			 //refresh_token
					.refreshToken()
                	 //client_credentials
					.clientCredentials()
                	 //password
					.password()
					.build();

	DefaultOAuth2AuthorizedClientManager authorizedClientManager =
			new DefaultOAuth2AuthorizedClientManager(
					clientRegistrationRepository, authorizedClientRepository);
	authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);

	return authorizedClientManager;
}
```

# 资源服务器

## Nimbus JOSE + JWT

nimbus-jos-jwt 用来操作 JWT 的库,官网 https://connect2id.com/products/nimbus-jose-jwt

```xml
<dependency>
    <groupId>com.nimbusds</groupId>
    <artifactId>nimbus-jose-jwt</artifactId>
    <version>9.11.1</version>
</dependency>
```

JWT、JWS、JWE关系：
* JWT（JSON Web Token）指的是一种规范，这种规范允许我们使用 JWT 在两个组织之间传递安全可靠的信息。
* JWS（JSON Web Signature）和 JWE（JSON Web Encryption）是 JWT 规范的两种不同实现，我们平时最常使用的实现就是 JWS 。

对称加密和非对称加密：
* 『对称加密』指的是使用相同的秘钥来进行加密和解密，如果你的秘钥不想暴露给解密方，考虑使用非对称加密。在加密方和解密方是同一个人（或利益关系紧密）的情况下可以使用它。
* 『非对称加密』指的是使用公钥和私钥来进行加密解密操作。对于加密操作，公钥负责加密，私钥负责解密，对于签名操作，私钥负责签名，公钥负责验证。非对称加密在 JWT 中的使用显然属于签名操作。在加密方和解密方是不同人（或不同利益方）的情况下可以使用它。

nimbus-jose-jwt 支持的算法都在它的 JWSAlgorithm 和 JWEAlgorithm 类中有定义。例如：JWSAlgorithm algorithm = JWSAlgorithm.HS256

### API

JWSHeader：对应JWT中的header部分。

```java
JWSHeader header = new JWSHeader.Builder(algorithm).type(JOSEObjectType.JWT).build();
// 获得头部信息的 Base64 形式（ JWT 中的实际头部信息）
header.getParsedBase64URL();
```

Payload：对应JWT中的payload部分。

```java
// 这里还可以传 JSON 串，或 Map
Payload payload = new Payload("hello world");
// 获得荷载部信息的 Base64 形式（ JWT 中的实际荷载部信息）
header.toBase64URL();
```

JWSSigner：签名器，负责生成 signature 部分。

```java
// secret 为秘钥
JWSSigner jwsSigner = new MACSigner(secret);
// 获得荷载部信息的 Base64 形式（ JWT 中的实际荷载部信息）

JWSObject jwsObject = new JWSObject(jwsHeader, payload);
jwsObject.sign(jwsSigner);

// jwsObject 序列化为 token
String token = jwsObject.serialize();
```

JWSVerifier：验证器，负责对token进行验证。

```java
// 将 token 反序列化为 jwsObject
JWSObject jwsObject = JWSObject.parse(token);
JWSVerifier jwsVerifier = new MACVerifier(secret);
if (!jwsObject.verify(jwsVerifier)) {
    throw new RuntimeException("token 签名不合法！");
}
```

使用公私钥生成JWT：

1. 使用 keytool 生成秘钥对:

```bash
# 语法
keytool -genkey -alias <证书别名> -keyalg <密钥算法> -keystore <证书库的位置和名称> -keysize <密钥长度> -validity <证书有效期（天数）> -storepass <储存库密码>
# 示例
keytool -genkey -alias jwt -keyalg RSA -keystore jwt.jks -storepass 123456
# 导出公钥证书
keytool -export -alias jwt -keystore jwt.jks  -file jwt-pub.cer
# 查看公钥,openssl 可在 http://slproweb.com/products/Win32OpenSSL.html 下载，或者使用下面的办法获得
keytool -list -rfc --keystore jwt.jks | openssl x509 -inform pem -pubkey
```

2. 读取公私钥：

```xml
<dependency>
   <groupId>org.springframework.security</groupId>
   <artifactId>spring-security-rsa</artifactId>
   <!-- spring-cloud-dependencies 已含有版本信息 -->
</dependency>
```

```java
public RSAKey generateRsaKey() {
    // 123456 储存库密码，即 storepass
    String storePass = "123456";
    KeyStoreKeyFactory keyStoreKeyFactory = new KeyStoreKeyFactory(new ClassPathResource("jwt.jks"), storePass.toCharArray());
    KeyPair keyPair = keyStoreKeyFactory.getKeyPair("jwt", storePass.toCharArray());

    RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
    // 打印公钥
    System.out.println(org.apache.tomcat.util.codec.binary.Base64.encodeBase64String(publicKey.getEncoded()));
    RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();
    RSAKey rsaKey = new RSAKey.Builder(publicKey).privateKey(privateKey).build();
    return rsaKey;
}
```

3. 生成token：

```java
RSAKey rsaKey = generateRsaKey();

JWSHeader jwsHeader = new JWSHeader
              .Builder(JWSAlgorithm.RS256)
              .type(JOSEObjectType.JWT)
              .build();

Payload payload = new Payload("hello world");

JWSObject jwsObject = new JWSObject(jwsHeader, payload);
// rsaKey 生成签名器
JWSSigner jwsSigner = new RSASSASigner(rsaKey, true);
jwsObject.sign(jwsSigner);

// JWT/JWS 字符串
String token = jwsObject.serialize();
```

4. 解析token：

```java
JWSObject jwsObject = JWSObject.parse(token);

RSAKey rsaKey = generateRsaKey();
RSAKey publicRsaKey = rsaKey.toPublicJWK();
JWSVerifier jwsVerifier = new RSASSAVerifier(publicRsaKey);

if (!jwsObject.verify(jwsVerifier)) {
    throw new RuntimeException("token签名不合法！");
}

String payload = jwsObject.getPayload().toString();
```

JWKSource<SecurityContext> : 用来获取公钥集合。

```
public JWKSource<SecurityContext> jwkSource() {
    RSAKey rsaKey = generateRsaKey();

    JWKSet jwkSet = new JWKSet(rsaKey);
    return (jwkSelector, securityContext) -> jwkSelector.select(jwkSet);
}
```

## 使用

1. 引入依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

2.  配置 JwtDecoder ：有4种方式

```yaml
# 使用认证服务器公钥端点
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          jwk-set-uri: https://www.googleapis.com/oauth2/v3/certs
#或者使用认证服务器端点，获取公钥端点
# 会通过 https://xxx.com/aaa/.well-known/openid-configuration、https://xxx.com/.well-known/openid-configuration/aaa、https://xxx.com/.well-known/oauth-authorization-server/aaa 端点获取 jwks_uri，然后流程和上面一样。
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://xxx.com/aaa

# 或者使用本地公钥配置
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          public-key-location: public.key
```

或者自定义 JwtDecoder：NimbusJwtDecoder是它的实现类

```java
// 使用jwkSetUri
NimbusJwtDecoder nimbusJwtDecoder = NimbusJwtDecoder.withJwkSetUri("xxxx").jwsAlgorithm(SignatureAlgorithm.from("xxxx")).build();
// 直接使用公钥字符串
RSAPublicKey publicKey = (RSAPublicKey) KeyFactory.getInstance("RSA").generatePublic(new X509EncodedKeySpec(java.util.Base64.getMimeDecoder().decode("publickey")));
NimbusJwtDecoder.withPublicKey(publicKey).signatureAlgorithm(SignatureAlgorithm.from("xxxx")).build();
// 获取 RSAPublicKey 可以使用 RsaKeyConversionServicePostProcessor 来获取
@Bean
BeanFactoryPostProcessor conversionServiceCustomizer() {
    return beanFactory -> beanFactory.getBean(RsaKeyConversionServicePostProcessor.class).setResourceLoader(new CustomResourceLoader());
}

@Value("${key.location}")
RSAPublicKey key;

// 使用IssuerUri
JwtDecoders.fromIssuerLocation("xxxx");

// 对称秘钥
NimbusJwtDecoder.withSecretKey(this.key).build();

// NimbusJwtDecoder 设置缓存
NimbusJwtDecoder.withJwkSetUri(jwkSetUri).cache(cacheManager.getCache("jwks")).build();

// NimbusJwtDecoder设置请求超时时间
RestOperations rest = builder
        .setConnectTimeout(Duration.ofSeconds(60))
        .setReadTimeout(Duration.ofSeconds(60))
        .build();
NimbusJwtDecoder jwtDecoder = NimbusJwtDecoder.withJwkSetUri(jwkSetUri).restOperations(rest).build();
```

3. 配置 HttpSecurity：springboot 已在 OAuth2ResourceServerJwtConfiguration 默认配置，自定义WebSecurityConfigurerAdapter时需要配置。

```java
		http.authorizeRequests((requests) -> requests.anyRequest().authenticated()).oauth2ResourceServer(OAuth2ResourceServerConfigurer::jwt);
```

## 启动

1. 通过使用 http.oauth2ResourceServer(OAuth2ResourceServerConfigurer::jwt) 创建 OAuth2ResourceServerConfigurer并配置 http ，OAuth2ResourceServerConfigurer::jwt 负责初始化 OAuth2ResourceServerConfigurer 的 JwtConfigurer。
2. 自动装配。
   * SpringSecurity 主流程。
   * Oauth2ResourceServerConfiguration.JwtConfiguration：根据配置初始化 JwtDecoder。

## OAuth2ResourceServerConfigurer 配置内容

* 向 exceptionTranslationFilter 中添加 BearerTokenRequestMatcher 对应的 BearerTokenAccessDeniedHandler、BearerTokenAuthenticationEntryPoint，当请求匹配BearerTokenRequestMatcher时，执行该 accessDeniedHandler、authenticationEntryPoint。
* 通过 JwtConfigurer 获取 AuthenticationProvider，添加到 HttpSecurity 中。
* 添加 BearerTokenAuthenticationFilter。

### BearerTokenRequestMatcher

委托给 DefaultBearerTokenResolver 解析，两种方式：

* 优先取http包含 Authorization 头，内容为 Bearer <TOKEN内容>
* 如果不满足上面条件，对应 POST 和 GET 请求，会取 url 后面的 access_token 参数。

DefaultBearerTokenResolver 可以配置成 Bean 替换掉默认 DefaultBearerTokenResolver。

### BearerTokenAccessDeniedHandler

* 向 Response 中添加 WWW-Authenticate 头，The request requires higher privileges than provided by the access token.
* 状态码403

### BearerTokenAuthenticationEntryPoint

* 处理 AuthenticationException 异常。
* 状态码401

### BearerTokenAuthenticationFilter

1. 使用 DefaultBearerTokenResolver 匹配 request，当失败时 使用 BearerTokenAuthenticationEntryPoint 处理。
2. 使用 AuthenticationManagerResolver 获取 AuthenticationManager 进行认证，这样可以对不同的请求使用不同的 AuthenticationManager，使用 http.oauth2ResourceServer(resourceServer -> resourceServer.jwt().and().authenticationManagerResolver(new xxxAuthenticationManagerResolver)) 配置。
3. 认证失败时，使用 AuthenticationFailureHandler 处理，默认也是使用 BearerTokenAuthenticationEntryPoint 处理，使用 http.oauth2ResourceServer(resourceServer -> resourceServer.jwt().and().authenticationEntryPoint(new xxxAuthenticationEntryPoint()))。

### JwtConfigurer

* 负责生成 JwtAuthenticationProvider，并设置  JwtDecoder 和 Converter<Jwt, ? extends AbstractAuthenticationToken> ，默认为 JwtDecoder 通过 Bean 获取，Converter<Jwt, ? extends AbstractAuthenticationToken> 默认为 JwtAuthenticationConverter

#### JwtAuthenticationProvider

![111](assets/111.png)

* 支持认证 BearerTokenAuthenticationToken
* 使用 JwtDecoder 将 token 解析为 JWT 对象。
* 使用 JwtAuthenticationConverter 将 JWT 对象转换成 AbstractAuthenticationToken

当解析失败时会抛出 InvalidBearerTokenException，可使用 AuthenticationFailureBadCredentialsEvent 监听：

```java
@Component
public class FailureEvents {
	@EventListener
    public void onFailure(AuthenticationFailureBadCredentialsEvent badCredentials) {
		if (badCredentials.getAuthentication() instanceof BearerTokenAuthenticationToken) {
		    // ... handle
        }
    }
}
```

#### JwtAuthenticationConverter

负责从 JWT 对象解析 Collection<GrantedAuthority> 生成 JwtAuthenticationToken

自定义：

```java
public JwtAuthenticationConverter jwtAuthenticationConverter() {
    JwtGrantedAuthoritiesConverter grantedAuthoritiesConverter = new JwtGrantedAuthoritiesConverter();
    // 将读取 Claim 中 authorities 属性转换为权限，默认读取 "scope", "scp"，生成的权限字符串默认添加 SCOPE_ 前缀。
    grantedAuthoritiesConverter.setAuthoritiesClaimName("authorities");

    JwtAuthenticationConverter jwtAuthenticationConverter = new JwtAuthenticationConverter();
    jwtAuthenticationConverter.setJwtGrantedAuthoritiesConverter(grantedAuthoritiesConverter);
    return jwtAuthenticationConverter;
}
```

#### MappedJwtClaimSetConverter

负责对JWT具体字段进行转换，例如 claims

#### JwtDecoder

```java
// 负责解析token为 JWT 对象
public interface JwtDecoder {
	Jwt decode(String token) throws JwtException;
}
```

#### NimbusJwtDecoder

实现JwtDecoder：

1. 负责解析token为 JWT 对象
2. 使用 JWTProcessor<SecurityContext> 验证签名。
3. 使用 OAuth2TokenValidator<Jwt> 进行验证，验证失败返回 JwtValidationException，默认为 JwtValidators.createDefault() 即 new DelegatingOAuth2TokenValidator<>(Arrays.asList(new JwtTimestampValidator()))，JwtTimestampValidator 会验证 token 过期时间，默认使用utc时间，验证完成后返回 OAuth2TokenValidatorResult。

## 请求流程

* 添加 Authorization ：Bearer <TOKEN内容>  头，发起请求：

![110](assets/110.png)

  详情见 BearerTokenAuthenticationFilter

* token 已认证，但是权限不够：

![109](assets/109.png)

  1. FilterSecurityInterceptor 抛出 AccessDeniedException。
  2. ExceptionTranslationFilter 使用 BearerTokenAuthenticationEntryPoint 添加  WWW-Authenticate 头信息返回

认证成功后 Authentication#getPrincipal 获取的是 Jwt 对象， Authentication#getPrincipal 获取的是 JWT 的 sub 属性。

# 认证服务器

## 使用

```xml
<!-- 需要同时引入这两个包 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<!-- 0.2.3 对应springBootVersion 2.5.10  -->
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-oauth2-authorization-server</artifactId>
    <version>${spring-security-oauth2-authorization-server.version}</version>
</dependency>
```

目前还没有 authorization-server 的自动配置，需要手动配置。

## 核心API

### ProviderContextFilter

负责配置ProviderContextHolder，当 issuer 端点不存在是，默认使用 http://ip:端口/<server.servlet.context-path> 作为 issuer 端点。

#### ProviderSettings

负责配置端点映射，可使用 ProviderSettings.builder().build() 创建，启动时必须配置该Bean,以下为默认配置：

```java
public static Builder builder() {
  return new Builder()
      .authorizationEndpoint("/oauth2/authorize")
      .tokenEndpoint("/oauth2/token")
      .jwkSetEndpoint("/oauth2/jwks")
      .tokenRevocationEndpoint("/oauth2/revoke")
      .tokenIntrospectionEndpoint("/oauth2/introspect")
      .oidcClientRegistrationEndpoint("/connect/register")
      .oidcUserInfoEndpoint("/userinfo");
}
```

### OAuth2AuthorizationEndpointFilter

负责处理 authorizationEndpoint 端点，即 OAuth2 协议的第一步，获取 AUTHORIZATION_CODE,也是用户授权接口 /oauth2/authorize

```java
public final class OAuth2AuthorizationEndpointFilter extends OncePerRequestFilter {
	private final AuthenticationManager authenticationManager;
	private final RequestMatcher authorizationEndpointMatcher;
	private final RedirectStrategy redirectStrategy = new DefaultRedirectStrategy();
	private AuthenticationConverter authenticationConverter;
	private AuthenticationSuccessHandler authenticationSuccessHandler = this::sendAuthorizationResponse;
	private AuthenticationFailureHandler authenticationFailureHandler = this::sendErrorResponse;
	private String consentPage;
}
```

处理 3 类请求：

* authorizationEndpoint、GET 类型：GET类型请求授权码
* authorizationEndpoint、POST类型、参数包含 response_type、scope、scope中包含openid：POST类型请求授权码
* authorizationEndpoint、POST类型、参数不包含 response_type：授权接口：例如：

```http
Request URL: http://localhost:30009/oauth2/authorize
Request Method: POST
Content-Type: application/x-www-form-urlencoded

form-data：
client_id: login-client
state: mWW7YPLra-rBmw-tBJ8AGdjMkrWh-e4pJeNwR-04df4=
scope: message.read
scope: message.write
```

流程：
1. 使用 OAuth2AuthorizationCodeRequestAuthenticationConverter 解析请求参数，构建 OAuth2AuthorizationCodeRequestAuthenticationToken 。
2.

#### RegisteredClient

已经注册的客户端对象。

```java
public class RegisteredClient implements Serializable {
	private String id;
  // 客户端ID，提供给客户端使用
	private String clientId;
  // 客户端注册时间
	private Instant clientIdIssuedAt;
  // 客户端密码，提供给客户端使用
	private String clientSecret;
  // 客户端密码过期时间，程序中未使用,可以自己实现
	private Instant clientSecretExpiresAt;
  // 客户端名称
	private String clientName;
  // 认证客户端身份的方法
	private Set<ClientAuthenticationMethod> clientAuthenticationMethods;
  // 授权模式
	private Set<AuthorizationGrantType> authorizationGrantTypes;
  // 重定向地址
	private Set<String> redirectUris;
  // 授权域
	private Set<String> scopes;
  // 配置Client属性，例如 是否需要授权 等
	private ClientSettings clientSettings;
  // 配置 Token 属性，例如 token 过期时间（默认5分钟）、refresh token 过期时间（默认 60 分钟）等
	private TokenSettings tokenSettings;
}
```

#### RegisteredClientRepository

对 RegisteredClient 进行持久化操作，必须要配置。


# RBAC

## 权限系统与RBAC模型概述

RBAC（Role-Based Access Control ）基于角色的访问控制。

在20世纪90年代期间，大量的专家学者和专门研究单位对RBAC的概念进行了深入研究，先后提出了许多类型的RBAC模型，其中以美国George Mason大学信息安全技术实验室（LIST）提出的RBAC96模型最具有系统性，得到普遍的公认。

RBAC认为权限的过程可以抽象概括为：判断【Who是否可以对What进行How的访问操作（Operator）】这个逻辑表达式的值是否为True的求解过程。

即将权限问题转换为Who、What、How的问题。who、what、how构成了访问权限三元组。

## RBAC组成

在RBAC模型里面，有3个基础组成部分，分别是：用户、角色和权限。

RBAC通过定义角色的权限，并对用户授予某个角色从而来控制用户的权限，实现了用户和权限的逻辑分离（区别于ACL模型），极大地方便了权限的管理

下面在讲解之前，先介绍一些名词：

- User（用户）：每个用户都有唯一的UID识别，并被授予不同的角色
- Role（角色）：不同角色具有不同的权限
- Permission（权限）：访问权限
- 用户-角色映射：用户和角色之间的映射关系
- 角色-权限映射：角色和权限之间的映射

它们之间的关系如下图所示：

![1-1](assets/1-1.png)

例如：

管理员和普通用户被授予不同的权限，普通用户只能去修改和查看个人信息，而不能创建创建用户和冻结用户，而管理员由于被授 予所有权限，所以可以做所有操作。

![1-2](assets/1-2.png)

## 安全原则

RBAC支持三个著名的安全原则：最小权限原则、责任分离原则和数据抽象原则

- 最小权限原则：RBAC可以将角色配置成其完成任务所需的最小权限集合
- 责任分离原则：可以通过调用相互独立互斥的角色来共同完成敏感的任务，例如要求一个计账员和财务管理员共同参与统一过账操作
- 数据抽象原则：可以通过权限的抽象来体现，例如财务操作用借款、存款等抽象权限，而不是使用典型的读、写、执行权限

## RBAC96模型族

RBAC96是一个模型族，其中包括RBAC0~RBAC3四个概念性模型。

1、基本模型RBAC0定义了完全支持RBAC概念的任何系统的最低需求。

2、RBAC1和RBAC2两者都包含RBAC0，但各自都增加了独立的特点，它们被称为高级模型。

RBAC1中增加了角色分级的概念，一个角色可以从另一个角色继承许可权。

RBAC2中增加了一些限制，强调在RBAC的不同组件中在配置方面的一些限制。

3、RBAC3称为统一模型，它包含了RBAC1和RBAC2，利用传递性，也把RBAC0包括在内。这些模型构成了RBAC96模型族。

## RBAC0

RBAC0，是最简单、最原始的实现方式，也是其他RBAC模型的基础。

![1-3](assets/1-3.png)

在RBAC之中,包含用户users(USERS)、角色roles(ROLES)、目标objects(OBS)、操作operations(OPS)、许可权permissions(PRMS)五个基本数据元素，此模型指明用户、角色、访问权限和会话之间的关系。

> 会话可以理解为某次使用某种角色的用户访问站点或者是同一个用户从一种角色切换到另外一种角色。

每个角色至少具备一个权限，每个用户至少扮演一个角色；可以对两个完全不同的角色分配完全相同的访问权限；会话由用户控制，一个用户可以创建会话并激活多个用户角色，从而获取相应的访问权限，用户可以在会话中更改激活角色，并且用户可以主动结束一个会话。

在该模型中，用户和角色之间可以是多对多的关系，即一个用户在不同场景下是可以有不同的角色，例如：项目经理也可能是组长也可能是架构师。同时每个角色都至少有一个权限。这种模型下，用户和权限被分离独立开来，使得权限的授权认证更加灵活。

## RBAC1

基于RBAC0模型，引入了角色间的继承关系，即角色上有了上下级的区别。

![1-4](assets/1-4.png)

角色间的继承关系可分为一般继承关系和受限继承关系。一般继承关系仅要求角色继承关系是一个绝对偏序关系，允许角色间的多继承。而受限继承关系则进一步要求角色继承关系是一个树结构，实现角色间的单继承。

> 若角色R1继承了角色R2，则称R1是R2的父角色，R2是R1的子角色，记为R1≥R2(偏序关系)。

（受限继承关系）举个实例就是：在OA系统内，集团旗下拥有多个部门，而总经理则拥有最全权限，部门经理则是从总经理这继承部分权限，而部门员工则从经理这继承部分权限。形成一个比较有序的树结构，实现了单继承关系。

（一般继承关系）举个实例就是：同样是OA系统，是现有各个操作员工角色的，后慢慢产生部门经理，则部门经理从多个部门员工里继承权限。这其中差异较大就是此时部门经理是子角色，同时其拥有多个父角色，并从父角色里继承权限，若后面又产生了部门副经理，是同样从多个部门员工里继承，形成多对多的关系，即为多继承关系。

这种模型适合于角色之间层次分明，可以给角色分组分层。

## RBAC2

RBAC2，基于RBAC0模型的基础上，进行了角色的访问控制。

RBAC2模型中添加了责任分离关系。RBAC2的约束规定了权限被赋予角色时，或角色被赋予用户时，以及当用户在某一时刻激活一个角色时所应遵循的强制性规则。责任分离包括静态责任分离和动态责任分离。约束与用户-角色-权限关系一起决定了RBAC2模型中用户的访问许可，此约束有多种。

- 互斥角色 ：同一用户只能分配到一组互斥角色集合中至多一个角色，支持责任分离的原则。互斥角色是指各自权限互相制约的两个角色。对于这类角色一个用户在某一次活动中只能被分配其中的一个角色，不能同时获得两个角色的使用权。常举的例子：在审计活动中，一个角色不能同时被指派给会计角色和审计员角色。
- 基数约束 ：一个角色被分配的用户数量受限；一个用户可拥有的角色数目受限；同样一个角色对应的访问权限数目也应受限，以控制高级权限在系统中的分配。例如公司的领导人有限的；
- 先决条件角色 ：可以分配角色给用户仅当该用户已经是另一角色的成员；对应的可以分配访问权限给角色，仅当该角色已经拥有另一种访问权限。指要想获得较高的权限，要首先拥有低一级的权限。就像我们生活中，国家主席是从副主席中选举的一样。
- 运行时互斥 ：例如，允许一个用户具有两个角色的成员资格，但在运行中不可同时激活这两个角色。

## RBAC3

RBAC3可以被称为最复杂，最全面的权限管理。基于RBAC0的，同时将RBAC1与RBAC2结合起来。

## 优缺点

优点：

- 简化了用户和权限的关系
- 易扩展、易维护

缺点：

- RBAC模型没有提供操作顺序的控制机制，这一缺陷使得RBAC模型很难适应哪些对操作次序有严格要求的系统

## 使用场景

实际的使用场景中看，RBAC0是应用的最广，最多的。RBAC1~3应用程度几乎为0，并不是因为没场景，其中最重要的一个原因是投入产出比不符，除了复杂的OA系统环境等外，几乎很少系统需要系统性的设计到RBAC1/RBAC2。

此外，两项基本原则在权限设计过程中也是被广泛应用：职责分离与最小特权。

职责分离：不同于约束，只是强调不同角色完成不同业务的分工，通过不同角色的分配实现对业务与数据的安全性管理；

最小特权：系统分配给角色的权限需要受限，权限满足完成任务的基本需要即可；

## 数据库建模

我们比较常见的就是基于角色的访问控制，用户通过角色与权限进行关联。简单地说，一个用户拥有多个角色，一个角色拥有多个权限。这样，就构造成“用户-角色-权限”的授权模型。在这种模型中，用户与角色之间、角色与权限之间，通常都是多对多的关系。如下图：

![1-5](assets/1-5.png)

基于这个，得先了解角色到底是什么？我们可以理解它为一定数量的权限的集合，是一个权限的载体。例如：一个论坛的“管理员”、“版主”，它们都是角色。但是所能做的事情是不完全一样的，版主只能管理版内的贴子，用户等，而这些都是属于权限，如果想要给某个用户授予这些权限，不用直接将权限授予用户，只需将“版主”这个角色赋予该用户即可。

但是通过上面我们也发现问题了，如果用户的数量非常大的时候，就需要给系统的每一个用户逐一授权(分配角色)，这是件非常繁琐的事情，这时就可以增加一个用户组，每个用户组内有多个用户，除了给单个用户授权外，还可以给用户组授权，这样一来，通过一次授权，就可以同时给多个用户授予相同的权限，而这时用户的所有权限就是用户个人拥有的权限与该用户所在组所拥有的权限之和。用户组、用户与角色三者的关联关系如下图：

![1-6](assets/1-6.png)

通常在应用系统里面的权限我们把它表现为菜单的访问(页面级)、功能模块的操作(功能级)、文件上传的删改，甚至页面上某个按钮、图片是否可见等等都属于权限的范畴。有些权限设计，会把功能操作作为一类，而把文件、菜单、页面元素等作为另一类，这样构成“用户-角色-权限-资源”的授权模型。而在做数据表建模时，可把功能操作和资源统一管理，也就是都直接与权限表进行关联，这样可能更具便捷性和易扩展性。如下图：

![1-7](assets/1-7.png)

这里特别需要注意以下权限表中有一列“PowerType(权限类型)”，我们根据它的取值来区分是哪一类权限，可以把它理解为一个枚举，如“MENU”表示菜单的访问权限、“OPERATION”表示功能模块的操作权限、“FILE”表示文件的修改权限、“ELEMENT”表示页面元素的可见性控制等。

这样设计的好处有两个。一、不需要区分哪些是权限操作，哪些是资源，（实际上，有时候也不好区分，如菜单，把它理解为资源呢还是功能模块权限呢？）；二、方便扩展，当系统要对新的东西进行权限控制时，我只需要建立一个新的关联表“权限XX关联表”，并确定这类权限的权限类型字符串即可。

需要注意的是，权限表与权限菜单关联表、权限菜单关联表与菜单表都是一对一的关系。（文件、页面权限点、功能操作等同理）。也就是每添加一个菜单，就得同时往这三个表中各插入一条记录。这样，可以不需要权限菜单关联表，让权限表与菜单表直接关联，此时，须在权限表中新增一列用来保存菜单的ID，权限表通过“权限类型”和这个ID来区分是种类型下的哪条记录。最后扩展出来的模型完整设计如下图：

![1-8](assets/1-8.png)

注意上面额外增加了一个操作日志表；

随着系统的日益庞大，为了方便管理，如果有需要可引入角色组对角色进行分类管理，跟用户组不同，角色组不参与授权。例如：当遇到有多个子公司，每个子公司下有多个部门，这是我们就可以把部门理解为角色，子公司理解为角色组，角色组不参于权限分配。另外，为方便上面各主表自身的管理与查找，可采用树型结构，如菜单树、功能树等，当然这些可不需要参于权限分配。

除了用户管理，角色权限管理，还有组织管理，我们可以把组织与角色进行关联,用户加入组织后,就会自动获得该组织的全部角色,无须管理员手动授予,大大减少工作量,同时用户在调岗时,只需调整组织,角色即可批量调整。组织的另外一个作用是控制数据权限,把角色关联到组织,那么该角色只能看到该组织下的数据权限。

在组织中，不用的岗位拥有不同的权限，比如财务部有总监,会计,出纳等职位,虽然都在同一部门,但是每个职位的权限是不同的,职位高的拥有更多的权限。总监拥有所有权限,会计和出纳拥有部分权限。特殊情况下,一个人可能身兼多职。

所以组织和岗位还需要和用户角色关联，如下：

![1-9](assets/1-9.png)

这时用户的所有权限就是用户个人拥有的权限、该用户所在组所拥有的权限、岗位所拥有的权限以及组织所拥有的权限之和。
