# 网络安全

1. 检查 Request Headers 中的 Refer 信息：
   通过`request.getHeader("referer")`获取 Refer 的值，检查是来自本域的请求。
2. 在 Chrome DevTools 的 Application 中：
   1. Secure：
      当设置为`true`时，表示创建的 Cookie 会被以安全的形式向服务器传输，也就是只能在 HTTPS 连接中被浏览器传递到服务器端进行会话验证，如果是 HTTP 连接则不会传递该信息，所以不会被窃取到 Cookie 的具体内容。
   2. HttpOnly：
      如果在Cookie中设置了`HttpOnly`属性，那么通过程序（JavaScript、Applet等）将无法读取到 Cookie 信息，这样能有效的防止 XSS 攻击。

# 会话

## 辨析 SessionCreationPolicy 和 SessionAuthenticationStrategy

在 Spring Security 的 `HttpSecurity` 中，`sessionCreationPolicy` 和 `sessionAuthenticationStrategy` 是用于管理会话的两个不同方面的配置。它们有着不同的目的，分别用于控制会话的创建策略和处理身份验证时如何使用会话。

### 1. `SessionCreationPolicy` （会话创建策略）

`SessionCreationPolicy` 是用于控制 Spring Security 如何创建和使用 HTTP 会话的策略。它决定了应用程序中何时创建会话，以及是否使用现有的会话。

#### 主要策略类型

Spring Security 提供了几种 `SessionCreationPolicy` 策略：

- **`ALWAYS`**：
  - 每次请求都会创建新的会话，或者如果已有会话，则使用当前会话。
  - 这种策略会确保无论是否需要，Spring Security 都会始终创建会话。
  
- **`IF_REQUIRED`**（默认值）：
  - 如果需要时才创建会话（例如，进行表单登录时会自动创建会话）。
  - 如果已经存在会话，Spring Security 会复用已有的会话；如果没有，会在需要时（如用户登录时）创建一个。
  
- **`NEVER`**：
  - Spring Security 不会主动创建会话，但如果存在会话，它可以使用这个会话。
  - 适用于那些不希望通过 Spring Security 创建会话，但仍然允许应用程序其他部分创建的场景。
  
- **`STATELESS`**：
  - 完全不创建会话，也不使用会话。
  - 适用于无状态的 RESTful API，通常与基于 JWT 的认证一起使用。每个请求都是独立的，不保留任何服务器端的会话状态。

#### `SessionCreationPolicy` 使用示例

以下是配置不同会话策略的示例：

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/public/**").permitAll()
                .anyRequest().authenticated()
            .and()
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS) // 无状态
            .and()
            .formLogin(); // 使用表单登录
    }
}
```

#### `SessionCreationPolicy` 的典型使用场景

- **无状态应用（STATELESS）**：在 RESTful API 中很常见，通常用于微服务架构，结合 JWT 认证机制，不使用服务器端会话。
- **有状态应用（ALWAYS/IF_REQUIRED）**：适用于传统的 Web 应用程序（如表单登录），在用户登录时创建会话以保存用户状态。
- **限制会话创建（NEVER）**：应用程序可能依赖其他组件（如应用服务器）创建的会话，但不希望 Spring Security 主动创建。

### 2. `SessionAuthenticationStrategy` （会话身份验证策略）

`SessionAuthenticationStrategy` 控制的是**身份验证成功时**，如何处理会话。它的重点在于处理用户成功登录后的会话管理行为，比如当用户登录时，是否应该重新创建会话、使旧会话失效或执行其他操作。

#### 主要策略类型

Spring Security 提供了几种常见的 `SessionAuthenticationStrategy` 实现：

- **`ChangeSessionIdAuthenticationStrategy`**：
  - 当用户登录成功时，会更改现有会话的 ID，而不丢失会话中的内容。这是防止会话固定攻击的常用策略。
  - 适合大多数应用场景，能够保证会话安全性而不会丢失用户之前的会话信息。
  
- **`ConcurrentSessionControlAuthenticationStrategy`**：
  - 处理并发会话的策略，限制同一用户的会话数量。
  - 通常用于阻止一个用户在多个设备上同时登录（防止并发登录）。
  
- **`RegisterSessionAuthenticationStrategy`**：
  - 用于记录会话信息，通常与并发会话控制结合使用，管理当前用户会话的生命周期。
  
- **`NullAuthenticatedSessionStrategy`**：
  - 什么都不做。适合无状态的应用场景，如使用 JWT 的 RESTful API，不需要使用会话。当配置了`SessionCreationPolicy.STATELESS` 时，通常**不需要显式配置 `NullAuthenticatedSessionStrategy`**。这是因为在 `SessionCreationPolicy.STATELESS` 模式下，Spring Security 会自动避免使用或创建会话，因此不会触发与会话相关的操作，像是 `SessionAuthenticationStrategy` 这样的配置实际上不会生效。

#### `SessionAuthenticationStrategy` 使用示例

以下是配置不同会话身份验证策略的示例：

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/public/**").permitAll()
                .anyRequest().authenticated()
            .and()
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.ALWAYS) // 始终创建会话
            .and()
            .formLogin()
            .and()
            .sessionManagement()
                .sessionAuthenticationStrategy(new ChangeSessionIdAuthenticationStrategy()); // 认证成功后更改会话ID
    }
}
```

#### `SessionAuthenticationStrategy` 的典型使用场景

- **防止会话固定攻击**：`ChangeSessionIdAuthenticationStrategy` 是常见的防御措施，防止攻击者通过盗用会话 ID 来冒充用户。
- **并发会话控制**：通过 `ConcurrentSessionControlAuthenticationStrategy` 来限制每个用户的最大会话数量。
- **无状态应用**：在无状态应用中，通常会使用 `NullAuthenticatedSessionStrategy`，因为不需要任何会话管理。

### 区别总结

- **`SessionCreationPolicy`**：
  - 关注的是**何时创建会话**以及是否使用会话。它控制会话的创建与管理逻辑，决定是否在请求过程中创建会话，适用于会话的整体策略管理。
  - 典型场景：是否为每个用户请求创建新会话（`ALWAYS`）、只在需要时创建会话（`IF_REQUIRED`）、或完全不创建会话（`STATELESS`）。

- **`SessionAuthenticationStrategy`**：
  - 关注的是**用户成功认证后如何处理会话**。主要用于处理身份验证成功时的会话行为，例如是否重新生成会话 ID、限制并发会话等。
  - 典型场景：防止会话固定攻击（`ChangeSessionIdAuthenticationStrategy`）、控制并发会话（`ConcurrentSessionControlAuthenticationStrategy`）等。

### 使用关系
- **`SessionCreationPolicy`** 决定了**是否以及何时创建会话**，主要控制会话的生命周期管理。
- **`SessionAuthenticationStrategy`** 决定了**用户认证成功后如何处理会话**，例如是否重新生成会话 ID、防止固定会话攻击等。

这两者是独立但互补的配置项，共同决定了应用中会话的创建、使用和管理策略。

# Filter

# WebSecurity

# WebFluxSecurity

## Spring Security 配置

Spring Security 设置要采用响应式配置，基于 WebFlux 中 WebFilter 实现，与 Spring MVC 的 Security 是通过 Servlet 的 Filter 实现类似，也是一系列 Filter 组成的过滤链。
 Reactive 与传统 MVC 配置对应：

|                       WebFlux                        |             MVC              |       作用       |
| :--------------------------------------------------: | :--------------------------: | :--------------: |
|                @EnableWebFluxSecurity                |      @EnableWebSecurity      | 开启security配置 |
|          ServerAuthenticationSuccessHandler          | AuthenticationSuccessHandler | 登录成功Handler  |
|          ServerAuthenticationFailureHandler          | AuthenticationFailureHandler | 登陆失败Handler  |
| ReactiveAuthorizationManager\<AuthorizationContext\> |     AuthorizationManager     |     认证管理     |
|           ServerSecurityContextRepository            |    SecurityContextHolder     | 认证信息存储管理 |
|              ReactiveUserDetailsService              |      UserDetailsService      |     用户登录     |
|             ReactiveAuthorizationManager             |    AccessDecisionManager     |     鉴权管理     |
|            ServerAuthenticationEntryPoint            |   AuthenticationEntryPoint   |  未认证Handler   |
|              ServerAccessDeniedHandler               |     AccessDeniedHandler      | 鉴权失败Handler  |

建立步骤：

配置类：

```java
@EnableWebFluxSecurity
public class MyWebFluxSecurityConfiguration {
    /**
     * 获取用户详情服务
     */
    @Autowired
    ReactiveUserDetailsService userDetailsService;

    @Autowired
    PermissionMapper permissionMapper;

    /**
     * 密码编码器
     *
     * @return 密码编码器
     */
    @Bean
    PasswordEncoder passwordEncoder() {
        return new SCryptPasswordEncoder();
    }

    /**
     * 配置获取用户详情服务，配置密码编码器，由 AuthenticationWebFilter 持有
     *
     * @return 鉴权管理器
     */
    @Bean
    ReactiveAuthenticationManager reactiveAuthenticationManager() {
        UserDetailsRepositoryReactiveAuthenticationManager manager = new UserDetailsRepositoryReactiveAuthenticationManager(userDetailsService);
        manager.setPasswordEncoder(passwordEncoder());
        return manager;
    }

    @Bean
    <T> ReactiveAuthorizationManager<T> reactiveAuthorizationManager() {
        return new MyReactiveAuthorizationManager<>(permissionMapper);
    }

    @Bean
    public SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity security) {
        return security
                .authorizeExchange() // 设置 AuthorizationWebFilter
                .pathMatchers(HttpMethod.OPTIONS).permitAll()
                .pathMatchers("/login", "/logout").permitAll()
                .pathMatchers("/user").access(reactiveAuthorizationManager())
                .anyExchange().authenticated().and()
                .formLogin().and()
                // .loginPage("/login").and()
                .logout().logoutUrl("/logout").and()
                .csrf().csrfTokenRepository(CookieServerCsrfTokenRepository.withHttpOnlyFalse()).and()
                .cors().disable()
                .httpBasic().disable()
                .build();
    }
}
```

授权管理：

```java
public class MyReactiveAuthorizationManager<T> implements ReactiveAuthorizationManager<T> {
    private PermissionMapper mapper;

    public void setMapper(PermissionMapper mapper) {
        this.mapper = mapper;
    }

    public MyReactiveAuthorizationManager(PermissionMapper mapper) {
        this.mapper = mapper;
    }

    @Override
    public Mono<AuthorizationDecision> check(Mono<Authentication> authentication, T object) {
        ServerWebExchange exchange = ((AuthorizationContext) object).getExchange();
        String url = exchange.getRequest().getPath().value();
        List<String> permissions = mapper.findPermissionByUrl(url);
        return authentication
                .filter(a -> a.isAuthenticated())
                .flatMapIterable(a -> a.getAuthorities())
                .map(g-> g.getAuthority())
                .any(a -> permissions.contains(a))
                .map( hasAuthority -> new AuthorizationDecision(hasAuthority))
                .defaultIfEmpty(new AuthorizationDecision(false));
    }
}
```

数据库用户信息服务类：

```java
@Service
public class MyUserDetailsService implements ReactiveUserDetailsService {
    @Autowired
    private UserMapper userMapper;
    
    @Autowired
    private PermissionMapper permissionMapper;

    @Override
    public Mono<UserDetails> findByUsername(String username) {
        return Mono.just(userService.loadUserByUsername(username));
    }
    
    public UserDetails loadUserByUsername(String username) {
        List<SysUser> users = userMapper.findByUserName(username);
        if (users.isEmpty()) {
            throw new UsernameNotFoundException("user: " + username + " do not exist!");
        }
        SysUser user = users.get(0);
        // 获取该用户所有角色的所有权限 Spring Security 中角色等于权限加 ROLE_ 前缀
        List<Permission> permissions = permissionMapper.findByAdminUserId(user.getId());
        List<GrantedAuthority> grantedAuthorities = new ArrayList<>();
        for (Permission permission : permissions) {
            if (permission != null && permission.getName() != null) {
                // 此处将权限信息添加到 GrantedAuthority 对象中，在后面进行全权限验证时会使用 GrantedAuthority 对象。
                GrantedAuthority ga = new SimpleGrantedAuthority(permission.getName());
                grantedAuthorities.add(ga);
            }
        }
        return new User(user.getUsername(), user.getPassword(), grantedAuthorities);
    }
}
```

---

## 初始化顺序

注解 EnableWebFluxSecurity.java 向容器中注入配置类 Bean：

```java
@Import({ServerHttpSecurityConfiguration.class, WebFluxSecurityConfiguration.class, ReactiveOAuth2ClientImportSelector.class})
@Configuration
```

自定义的配置类 MyWebFluxSecurityConfig 通过 ServerHttpSecurity.java 的`build()`方法自行向容器中注入 SecurityWebFilterChain 的 Bean：

```java
@EnableWebFluxSecurity
public class MyWebFluxSecurityConfig {
    @Bean
    public SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity security) {
        return security.build();
    }
}
```

上述 SecurityWebFilterChain 的实现类为 MatcherSecurityWebFilterChain，向其内部填充过滤器：

```java
public SecurityWebFilterChain build() {
    // 省略异常
    // 省略 HttpHeaderWriterWebFilter
    // 省略 securityContextRepository ReactorContextWebFilter
    // 省略 HttpsRedirectWebFilter
    // 省略 CsrfWebFilter
    // 省略 CorsWebFilter
    // 省略 httpBasic AuthenticationWebFilter
    // 省略 formLogin AuthenticationWebFilter
    // 省略 oauth2Login OAuth2AuthorizationRequestRedirectWebFilter 和 AuthenticationWebFilter
    // 省略 resourceServer AuthenticationWebFilter
    // 省略 client OAuth2AuthorizationCodeGrantWebFilter 和 OAuth2AuthorizationRequestRedirectWebFilter
    // 省略 loginPage LoginPageGeneratingWebFilter 和 LogoutPageGeneratingWebFilter
    // 省略 logout LogoutWebFilter
    // 省略 ServerRequestCacheWebFilter
    // 省略 SecurityContextServerWebExchangeWebFilter
    // 省略 authorizeExchange AuthorizationWebFilter 和 ExceptionTranslationWebFilter
    
    // 将 webFilters 中已有的过滤器先排序一次
    AnnotationAwareOrderComparator.sort(this.webFilters);
    // 过滤器列表
    List<WebFilter> sortedWebFilters = new ArrayList<>();
    // 将可排序且已排序的过滤器放入 sortedWebFilters
    this.webFilters.forEach( f -> {
        // 判断是否是已排序
        if (f instanceof OrderedWebFilter) {
            f = ((OrderedWebFilter) f).webFilter;
        }
        sortedWebFilters.add(f);
    });
    // ServerWebExchangeReactorContextWebFilter 没有实现 Ordered 不能排序，直接放入列表头
    sortedWebFilters.add(0, new ServerWebExchangeReactorContextWebFilter());
    return new MatcherSecurityWebFilterChain(getSecurityMatcher(), sortedWebFilters);
}
```

![](./assets/MatcherSecurityWebFilterChain.filters.png)

|                          过滤器（生成顺序） |   序号    | 功能                                                 |
| ------------------------------------------: | :-------: | ---------------------------------------------------- |
|                   HttpHeaderWriterWebFilter |    100    |                                                      |
|                     ReactorContextWebFilter |    500    |                                                      |
|                      HttpsRedirectWebFilter |    200    |                                                      |
|                               CsrfWebFilter |    400    | 检查、验证、生成 Token                               |
|                               CorsWebFilter |    300    |                                                      |
|                4 个 AuthenticationWebFilter | 600 ~ 800 | HttpBasic 登录、Form 登录、OAuth 登录和资源 JWT 验证 |
| OAuth2AuthorizationRequestRedirectWebFilter |    600    |                                                      |
|       OAuth2AuthorizationCodeGrantWebFilter |    900    |                                                      |
|                LoginPageGeneratingWebFilter |   1000    |                                                      |
|               LogoutPageGeneratingWebFilter |   1100    |                                                      |
|                             LogoutWebFilter |   1400    |                                                      |
|                 ServerRequestCacheWebFilter |   1300    |                                                      |
|   SecurityContextServerWebExchangeWebFilter |   1200    |                                                      |
|               ExceptionTranslationWebFilter |   1500    |                                                      |
|                      AuthorizationWebFilter |   1600    |                                                      |
|    ServerWebExchangeReactorContextWebFilter |    无     |                                                      |

WebFluxSecurityConfiguration.java 自动装配的过滤器链列表`securityWebFilterChains`，列表内有上述`MatcherSecurityWebFilterChain`的 Bean：

```java
@Autowired(required = false)
private List<SecurityWebFilterChain> securityWebFilterChains;
```

向容器中注入 WebFilterChainProxy 的 Bean，并让其持有`securityWebFilterChains`：

```java
public static final int WEB_FILTER_CHAIN_FILTER_ORDER = 0 - 100;

private static final String BEAN_NAME_PREFIX = "org.springframework.security.config.annotation.web.reactive.WebFluxSecurityConfiguration.";

private static final String SPRING_SECURITY_WEBFILTERCHAINFILTER_BEAN_NAME = BEAN_NAME_PREFIX + "WebFilterChainFilter";

@Bean(SPRING_SECURITY_WEBFILTERCHAINFILTER_BEAN_NAME)
@Order(value = WEB_FILTER_CHAIN_FILTER_ORDER)
public WebFilterChainProxy springSecurityWebFilterChainFilter() {
    return new WebFilterChainProxy(getSecurityWebFilterChains());
}

private List<SecurityWebFilterChain> getSecurityWebFilterChains() {
    List<SecurityWebFilterChain> result = this.securityWebFilterChains;
    if (ObjectUtils.isEmpty(result)) {
        return Arrays.asList(springSecurityFilterChain());
    }
    return result;
}
```

在 WebFluxSecurityConfiguration 之后自动装配的 HttpHandlerAutoConfiguration 中的 AnnotationConfig 向容器中注入 HttpHandler 的 Bean：

```java
@Bean
public HttpHandler httpHandler() {
    return WebHttpHandlerBuilder.applicationContext(this.applicationContext).build();
}
```

其中`builder()`方法会生成 HttpHandler：

```java
public HttpHandler build() {
    WebHandler decorated = new FilteringWebHandler(this.webHandler, this.filters);
    decorated = new ExceptionHandlingWebHandler(decorated,  this.exceptionHandlers);

    HttpWebHandlerAdapter adapted = new HttpWebHandlerAdapter(decorated);
    // 省略部分配置代码......
    adapted.afterPropertiesSet();

    return adapted;
}
```

其中`decorated`的构造器`FilteringWebHandler`会将`filters`中的过滤器包装成 DefaultWebFilterChain 并赋给成员变量`chain`：

```java
private final DefaultWebFilterChain chain;

public FilteringWebHandler(WebHandler handler, List<WebFilter> filters) {
    super(handler);
    this.chain = new DefaultWebFilterChain(handler, filters);
}
```

**不同于 Servlet 模式的 Security 的 Cookie 键为 JSESSION，Reactive 模式的 WebFlux Security 的默认 Cookie 键为 SESSION**

## 辨析 ServerHttpSecurity 和 HttpSecurity

`ServerHttpSecurity` 和 `HttpSecurity` 是 Spring Security 框架中用于配置安全性的两个类，但它们适用于不同的编程模型和应用架构。它们的区别主要在于所应用的上下文（reactive 与 servlet）和设计目的：

### 1. `HttpSecurity`
`HttpSecurity` 是 Spring Security 中用于传统的 Servlet-based 应用的安全配置类，适用于基于 Spring MVC 或基于 servlet 的 Web 应用。它提供了配置 HTTP 请求安全性的机制，通常用于同步编程模型。

#### 典型的使用场景：
- 用于基于 Spring MVC 框架的应用程序（例如使用 `@Controller` 和 `DispatcherServlet` 的应用）。
- 配置方式依赖于 servlet 容器（如 Tomcat、Jetty）。
- 支持传统的 servlet filters。

#### 代码示例：
```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/admin/**").hasRole("ADMIN")
                .antMatchers("/user/**").hasRole("USER")
                .anyRequest().authenticated()
            .and()
            .formLogin();
    }
}
```

### 2. `ServerHttpSecurity`
`ServerHttpSecurity` 是 Spring Security 提供的用于基于 Spring WebFlux（Reactive Programming 模型）的应用的安全配置类。它适用于响应式编程模型，通常用于构建响应式 Web 应用和微服务架构。

#### 典型的使用场景：
- 用于基于 Spring WebFlux 的应用程序（基于响应式流的非阻塞框架）。
- 适用于响应式服务器，如 Netty 或 Undertow。
- 支持响应式的过滤器链和非阻塞 IO。
  
#### 代码示例：
```java
@EnableWebFluxSecurity
public class SecurityConfig {
    @Bean
    public SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity http) {
        return http
            .authorizeExchange()
                .pathMatchers("/admin/**").hasRole("ADMIN")
                .pathMatchers("/user/**").hasRole("USER")
                .anyExchange().authenticated()
            .and()
            .formLogin()
            .and()
            .build();
    }
}
```

### 主要区别
1. **编程模型**：
   - `HttpSecurity`：用于同步的 servlet-based 应用。
   - `ServerHttpSecurity`：用于响应式的 WebFlux 应用。

2. **底层框架**：
   - `HttpSecurity`：基于 Servlet API 和 Spring MVC。
   - `ServerHttpSecurity`：基于 WebFlux 响应式框架，使用非阻塞的 IO 模型。

3. **服务器容器**：
   - `HttpSecurity`：通常与 Tomcat、Jetty 等 servlet 容器一起使用。
   - `ServerHttpSecurity`：与 Netty、Undertow 等非阻塞响应式服务器一起使用。

4. **过滤器机制**：
   - `HttpSecurity`：使用 servlet filter 机制。
   - `ServerHttpSecurity`：使用 WebFlux 的 `WebFilter` 机制。

### 总结
- 如果你的项目是基于传统的 servlet 和 Spring MVC 架构的 Web 应用，应该使用 `HttpSecurity`。
- 如果你的项目是基于 Spring WebFlux 响应式框架的应用，应该使用 `ServerHttpSecurity`。

# HTTPS

![在这里插入图片描述](./assets/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LqR54Of5oiQ6ZuoY3Nkbg==,size_14,color_FFFFFF,t_70,g_se,x_16.png)

Https 请求相当于两次 Http 请求，第一次获取公钥，第二次发送对称密钥并获取内容。

**HTTPS协议 = HTTP协议 + SSL/TLS协议**

https://www.cnblogs.com/muffe/p/11635124.html

https://blog.csdn.net/qq_43437874/article/details/121631195

![img](./assets/5142903-d457e7d6ac496ca3..webp)

![在这里插入图片描述](./assets/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LqR54Of5oiQ6ZuoY3Nkbg==,size_17,color_FFFFFF,t_70,g_se,x_16.png)

![img](./assets/https_rsa.png)

CA：Certificate Authority

CSR：Certificate Signing Request

CRT：Certificate 公钥

CER：Certificate 公钥 + 私钥

PEM：Privacy Enhanced Mail

预主密钥：pre-master secret， 在 TLS 握手过程中,客户端生成一个随机的预主密钥

客户端随机数 + 服务端随机数 + 预主密钥 = 会话密钥

## 密钥和证书

![在这里插入图片描述](./assets/6423d93363b8479d973d64c99c04eb30.png)

在 SSL/TLS 证书体系中，根证书（Root Certificate）、中间证书（Intermediate Certificate）、证书链（Certificate Chain）都是至关重要的概念，它们构成了整个证书验证的体系结构，确保了 SSL 连接的安全性。下面是对这些概念的详细解释：

### 根证书（Root Certificate）

- 根证书是由受信任的证书颁发机构（CA, Certificate Authority）签发的自签名证书。它是整个证书链的起点，具有最高级别的信任。

- 根证书由全球受信任的CA（如 DigiCert、GlobalSign、Let’s Encrypt）发布，并已经内置于常见的操作系统和浏览器中。这意

  着，当用户访问使用 SSL 的网站时，浏览器会自动信任这些根证书。

- 根证书自签名，这意味着它是由CA自己签署的，而不是由其他任何CA颁发。

特点：

- 权威性：根证书位于证书链的最顶端，具有最高的信任级别。
- 自签名：根证书不是由上级签发，而是自签名的。
- 存储于操作系统和浏览器中：根证书是操作系统、浏览器的信任列表的一部分。

### 中间证书（Intermediate Certificate）

- 中间证书是由根证书或其他中间证书颁发的，用来为最终的服务器证书提供一个中间的信任层。中间证书的主要目的是在根证书和服务器证书之间建立一个链式结构，使得根证书不直接参与每个网站的签发。
- CA使用中间证书对服务器证书进行签名，而不是使用根证书直接签名。这是为了安全性考虑，防止根证书泄露或受到攻击，因为一旦根证书失效，整个 CA 的信任链都会崩溃。

特点：

- 由根证书或其他中间证书签发。
- 用于签发服务器证书。
- 具有更短的有效期，以便于安全性管理。

### 服务器证书（Server Certificate）

- 服务器证书是网站的SSL证书，通常由中间证书签发。它是网站服务器提供给客户端（如浏览器）的证书，客户端使用它来建立SSL连接。
- 服务器证书是实际被使用来保护网站通信的证书，包含了网站的域名、证书的有效期、CA的签名等信息。

### 证书链（Certificate Chain）

- 证书链是由多个证书按层级结构组成的一条信任链，从服务器证书到根证书。它包含服务器证书、中间证书，以及根证书的顺序关系。客户端通过验证整个证书链，逐层确认每一个证书的真实性，最终信任根证书。
- 证书链中的每个证书都由上一级的证书进行签名，最终根证书是链条的顶端。

证书链验证流程：

1. 浏览器或客户端获取服务器的证书。
2. 浏览器检查该服务器证书的签发者（CA），并通过中间证书找到这个CA的证书。
3. 如果中间证书也是由更上一级的中间证书签发，继续往上查找，直到找到根证书。
4. 浏览器信任已知的根证书，因此证书链验证通过，浏览器认为该网站安全。

### 举例说明

假设你访问一个网站时，浏览器接收到以下证书链：

- 服务器证书：由 Intermediate CA 签发，用于特定网站（如 www.example.com）。
- 中间证书：由 Root CA 签发，用于 Intermediate CA 颁发服务器证书。
- 根证书：由 Root CA 自己签发，是操作系统或浏览器中已经内置的信任证书。

证书链的顺序：服务器证书 ← 中间证书 ← 根证书

### 总结

- 根证书是整个信任链的顶点，由 CA 自签名并存储在操作系统和浏览器中。
- 中间证书由根证书或其他中间证书签发，用于签发服务器证书，增强安全性。
- 服务器证书是最终用来保护网站的SSL证书。
- 证书链是由服务器证书、中间证书和根证书组成的层级信任链，保证浏览器信任网站的安全性。


仅导入自签名的中间证书无法使浏览器信任这个证书链，也无法消除“不安全”的提示。原因是：

1. 根证书的信任是基础
浏览器信任链的基础在于根证书。只有浏览器信任的根证书才能验证并信任后续的中间证书和服务器证书。如果只导入了中间证书，浏览器无法验证它的签发者，因此无法信任它。

2. 中间证书需要上级签发
中间证书的可信赖性依赖于它的上级证书（通常是根证书）。如果浏览器没有信任根证书，那么中间证书也不会被信任。即使你导入了中间证书，浏览器仍然会尝试向上查找根证书。如果根证书不在浏览器的信任列表中，浏览器就会认为证书链是不完整的，从而发出“不安全”的警告。

3. 自签名证书链的验证过程
自签名证书是由自己签发的根证书，通常不被浏览器默认信任。对于自签名证书链，整个信任链是由自签名的根证书开始的。因此，如果你使用自签名的证书，而不导入根证书，浏览器无法通过根证书来验证中间证书，也无法信任服务器的证书。

如何解决：
导入自签名的根证书：要让浏览器信任自签名的证书链，必须将自签名的根证书导入到浏览器或操作系统的信任证书列表中。只有这样，浏览器才能信任证书链中的中间证书和服务器证书，从而避免“不安全”的警告。

使用受信任的CA颁发证书：另一种选择是使用全球受信任的证书颁发机构（CA）签发的证书，这些机构的根证书已经被浏览器信任，避免了手动导入证书的麻烦。

因此，导入自签名的根证书是必要的，否则浏览器无法完全信任证书链，即使你导入了中间证书。

### 通过 Keytool 生成密钥证书

```shell
❯ keytool -?
密钥和证书管理工具

命令:

 -certreq            生成证书请求
 -changealias        更改条目的别名
 -delete             删除条目
 -exportcert         导出证书
 -genkeypair         生成密钥对
 -genseckey          生成密钥
 -gencert            根据证书请求生成证书
 -importcert         导入证书或证书链
 -importpass         导入口令
 -importkeystore     从其他密钥库导入一个或所有条目
 -keypasswd          更改条目的密钥口令
 -list               列出密钥库中的条目
 -printcert          打印证书内容
 -printcertreq       打印证书请求的内容
 -printcrl           打印 CRL 文件的内容
 -storepasswd        更改密钥库的存储口令
```

https://docs.oracle.com/en/java/javase/11/tools/keytool.html

```shell
# 官网示例
keytool -genkeypair -keystore root.jks -alias root -ext bc:c
keytool -genkeypair -keystore ca.jks -alias ca -ext bc:c
keytool -genkeypair -keystore server.jks -alias server

# A生成证书
keytool -keystore root.jks -alias root -exportcert -rfc > root.pem

# B证书请求 ==> A生成证书
keytool -storepass <passwd> -keystore ca.jks -certreq -alias ca |
    keytool -storepass <passwd> -keystore root.jks
    -gencert -alias root -ext BC=0 -rfc > ca.pem
# B导入证书
keytool -keystore ca.jks -importcert -alias ca -file ca.pem

# C证书请求 ==> B生成证书
keytool -storepass <passwd> -keystore server.jks -certreq -alias server |
    keytool -storepass <passwd> -keystore ca.jks -gencert -alias ca
    -ext ku:c=dig,kE -rfc > server.pem
# C导入证书
cat root.pem ca.pem server.pem |
    keytool -keystore server.jks -importcert -alias server
```

生成密钥对和证书：

```shell
# 生成根密钥对 条目别名为root 存入root.jks密钥库 采用RSA算法 密钥库口令为123456 密钥口令为123456 有效期365天
keytool -genkeypair -alias root -keystore root.jks -keyalg RSA -storepass 123456 -keypass 123456 -validity 365 -dname 'CN=LuJin, OU=CESI, O=CESI, L=Beijing, ST=Beijing, C=CN' -ext bc:c
# 生成中间密钥对
keytool -genkeypair -alias ca -keystore ca.jks -keyalg RSA -storepass 123456 -keypass 123456 -validity 365 -dname 'CN=LuJin, OU=CESI, O=CESI, L=Beijing, ST=Beijing, C=CN' -ext bc:c
# 生成服务密钥对
keytool -genkeypair -alias server -keystore server.jks -keyalg RSA -storepass 123456 -keypass 123456 -validity 365 -dname 'CN=LuJin, OU=CESI, O=CESI, L=Beijing, ST=Beijing, C=CN'

# 导出根证书 <== 根密钥条目
keytool -exportcert -alias root -keystore root.jks -storepass 123456 -rfc > root.pem

# 生成中间证书 <== 中间密钥条目 + 根密钥条目
keytool -certreq -alias ca -keystore ca.jks -storepass 123456 | keytool -gencert -alias root -keystore root.jks -storepass 123456 -validity 365 -ext bc=0 -rfc > ca.pem
# 拼接证书链 = 根证书 + 中间证书
cat root.pem >> ca_chain.pem
cat ca.pem >> ca_chain.pem
# 中间密钥库导入证书链
keytool -importcert -alias ca -keystore ca.jks -storepass 123456 -file ca_chain.pem

# 生成服务证书 <== 服务密钥条目 + 中间密钥条目 jdk15之前dns首字必须为字母
keytool -certreq -alias server -keystore server.jks -storepass 123456 | keytool -gencert -alias ca -keystore ca.jks -storepass 123456 -validity 365 -ext san=dns:localhost,dns:127.0.0.1,ip:127.0.0.1 -ext ku:c=dig,keyEncipherment -rfc > server.pem
# 拼接证书链 = 根证书 + 中间证书 + 服务证书
cat ca_chain.pem >> server_chain.pem
cat server.pem >> server_chain.pem
# 服务密钥库导入证书链
keytool -importcert -alias server -keystore server.jks -storepass 123456 -file server_chain.pem
```

导入到 jdk，其中`changeit`为 jdk 默认存储密码：

```cmd
keytool -import -trustcacerts -alias server -keystore "%JAVA_HOME%/lib/security/cacerts" -keypass changeit -file server_chain.pem
```

```powershell
keytool -import -trustcacerts -alias server -keystore "$Env:JAVA_HOME/lib/security/cacerts" -keypass changeit -file server_chain.pem
```

提取公钥：

```shell
keytool -list -rfc --keystore server.jks | openssl x509 -inform pem -pubkey
```

将密钥对转换为 PKCS12 格式：

```shell
keytool -importkeystore -srckeystore server.jks -destkeystore server.p12 -deststoretype PKCS12 -srcalias server -deststorepass 123456 -destkeypass 123456
```

提取私钥：

```shell
openssl pkcs12 -nodes -in server.p12 -out private.key
```

配置文件 application.yml：

```yaml
server:
  ssl:
    enabled: true
    key-alias: server
    key-store: classpath:server.jks
    key-store-type: jks
    key-password: 123456
    key-store-password: 123456
```

将密钥库 server.jks 放入`server.ssl.key-store`对应路径下，导入证书到“浏览器 - 设置 - 隐私和安全 - 安全 - 管理证书 - 颁发机构”。

[JDK 的证书库安装证书](https://blog.csdn.net/oscar999/article/details/127991038)
