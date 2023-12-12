> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/huangxuanheng/article/details/119052844)

单点登录是我们在分布式系统中很常见的一个需求。

> 分布式系统由多个不同的子系统组成，而我们在使用系统的时候，只需要登录一次即可，这样其他系统都认为用户已经登录了，不用再次去登录。大家可以登录淘宝天猫、支付宝、阿里云等网址体验一下单点登录

熟悉了 oauth2 单点登录功能，去面试时将是刘备遇到诸葛亮，如鱼得水

今天亨哥想和大家说一说 Spring Boot+OAuth2 做单点登录，利用 @EnableOAuth2Sso 注解快速实现单点登录功能。  
[源码下载地址](https://github.com/huangxuanheng/oauth2)

话不多说，开搞步骤

### 项目创建

把授权服务器和资源服务器搭建在一起（当然最好是分开，但是也有很多企业的产品也是放在一起的，这样做的目的就是方便）。

今天我们一共需要三个服务：

项目 端口 描述  
auth-server 9090 授权服务器 + 资源服务器  
client1 9091 子系统 1  
client2 9092 子系统 2  
auth-server 用来扮演授权服务器 + 资源服务器的角色，client1 和 client2 则分别扮演子系统的角色，将来等 client1 登录成功之后，我们也就能访问 client2 了，这样就能看出来单点登录的效果。

我们创建一个名为 oauth2-sso 的 Maven 项目作为父工程即可。

### 统一认证中心

接下来我们来搭建统一认证中心。

*   首先我们创建一个名为 auth-server 的 module，创建时添加如下依赖：

```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-security</artifactId>
        </dependency>

```

*   项目创建成功之后，这个模块由于要扮演授权服务器 + 资源服务器的角色，所以我们先在这个项目的启动类上添加 @EnableResourceServer 注解，表示这是一个资源服务器：

```
@SpringBootApplication
@EnableResourceServer
public class AuthServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(AuthServerApplication.class, args);
    }

}

```

*   接下来我们进行授权服务器的配置，由于资源服务器和授权服务器合并在一起，这样授权服务器的配置要省事很多：

```
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {
    @Autowired
    PasswordEncoder passwordEncoder;

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security.checkTokenAccess("permitAll()");
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                .withClient("api")
                .secret(passwordEncoder.encode("123456"))
                .autoApprove(true)
                .redirectUris("http://127.0.0.1:9091/login", "http://127.0.0.1:9092/login")
                .scopes("user")
                .accessTokenValiditySeconds(7200)
                .authorizedGrantTypes("authorization_code");

    }
}

```

这里只需要简单配置一下客户端的信息即可，配置很简单

为了简便，客户端的信息配置是基于内存的

*   接下来我们再来配置 Spring Security：

```
@Configuration
@Order(1)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/login.html", "/css/**", "/js/**", "/images/**");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.requestMatchers()
                .antMatchers("/login")
                .antMatchers("/oauth/authorize")
                .and()
                .authorizeRequests().anyRequest().authenticated()
                .and()
                .formLogin()
                .loginPage("/login.html")
                .loginProcessingUrl("/login")
                .permitAll()
                .and()
                .csrf().disable();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("harry")
                .password(passwordEncoder().encode("admin123"))
                .roles("admin");
    }
}

```

这里来大致捋一下：

1. 首先提供一个 BCryptPasswordEncoder 的实例，用来做密码加解密用。  
由于我自定义了登录页面，所以在 WebSecurity 中对这些静态资源放行。  
HttpSecurity 中，我们对认证相关的端点放行，同时配置一下登录页面和登录接口。  
2.AuthenticationManagerBuilder 中提供一个基于内存的用户（可以在这里修改为从数据库加载）。  
3. 还有一个比较关键的地方，因为资源服务器和授权服务器在一起，所以我们需要一个 @Order 注解来提升 Spring Security 配置的优先级。

*   SecurityConfig 和 AuthServerConfig 都是授权服务器需要提供的东西, 如果想将授权服务器和资源服务器拆分，我们还需要提供一个暴露用户信息的接口（如果将授权服务器和资源服务器分开，这个接口将由资源服务器提供）：

```
@RestController
public class UserController {
    @GetMapping("/user")
    public Principal getCurrentUser(Principal principal) {
        return principal;
    }
}

```

*   最后，我们在 application.properties 中配置一下项目端口：

server.port=9091  
另外，亨哥自己提前准备了一个登录页面，如下：

![](https://img-blog.csdnimg.cn/5dd4332ce69f4d55813eaf46e7fff015.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YW5neHVhbmhlbmc=,size_16,color_FFFFFF,t_70)

*   将登录页面相关的 html、css、js 等拷贝到 resources/static 目录下：  
    ![](https://img-blog.csdnimg.cn/a02c1846d064499193035f32864d6f9a.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YW5neHVhbmhlbmc=,size_16,color_FFFFFF,t_70)
    
*   这个页面很简单，就是一个登录表单而已，我把核心部分列出来：
    

```
<form action="/login" method="post">
    <div class="input">
        <label for="name">用户名</label>
        <input type="text" >
        <span class="spin"></span>
    </div>
    <div class="input">
        <label for="pass">密码</label>
        <input type="password" >
        <span class="spin"></span>
    </div>
    <div class="button login">
        <button type="submit">
            <span>登录</span>
            <i class="fa fa-check"></i>
        </button>
    </div>
</form>

```

注意一下 action 提交地址不要写错即可。

如此之后，我们的统一认证登录平台就算是 OK 了。

### 客户端创建

接下来我们来创建一个客户端项目，创建一个名为 client1 的 Spring Boot 项目，

*   添加如下依赖：

```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-security</artifactId>
        </dependency>


```

*   项目创建成功之后，我们来配置一下 Spring Security：

```
@Configuration
@EnableOAuth2Sso
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().authenticated().and().csrf().disable();
    }
}

```

这段配置很简单，就是说我们 client1 中所有的接口都需要认证之后才能访问，另外添加一个 @EnableOAuth2Sso 注解来开启单点登录功能。

*   接下来我们在 client1 中再来提供一个测试接口：

```
@RestController
public class UserController {
    @Value("${server.port}")
    private String port;
    @GetMapping("/getLogin")
    public String getLogin() {

        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        return "当前登录的用户是"+authentication.getName() + Arrays.toString(authentication.getAuthorities().toArray())+",登录端口："+port;
    }
}

```

这个测试接口返回当前登录用户的姓名和角色信息，服务端口号。

*   接下来我们需要在 client1 的 application.properties 中配置 oauth2 的相关信息：

```
security.oauth2.client.client-secret=123456
security.oauth2.client.client-id=api
security.oauth2.client.user-authorization-uri=http://127.0.0.1:9090/oauth/authorize
security.oauth2.client.access-token-uri=http://127.0.0.1:9090/oauth/token
security.oauth2.resource.user-info-uri=http://127.0.0.1:9090/user

server.port=9091

server.servlet.session.cookie.name=client1

```

这里的配置也比较熟悉，我们来看一下：

client-secret 是客户端密码。  
client-id 是客户端 id。  
user-authorization-uri 是用户授权的端点。  
access-token-uri 是获取令牌的端点。  
user-info-uri 是获取用户信息的接口（从资源服务器上获取）。  
最后再配置一下端口，然后给 cookie 取一个名字。  
如此之后，我们的 client1 就算是配置完成了。

按照相同的方式，我们再来配置一个 client2，client2 和 client1 一模一样，就是 cookie 的名字不同（随意取，不相同即可）。

### 测试

接下来，我们分别启动 auth-server、client1 和 client2，首先我们尝试去方式 client1 中的 getLogin 接口，这个时候会自动跳转到统一认证中心  
然后输入用户名：harry，密码：admin123

登录成功之后，会自动跳转回 client1 的 hello 接口，如下：

![](https://img-blog.csdnimg.cn/ce983f4038f54033908a2db6e30e9b0a.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YW5neHVhbmhlbmc=,size_16,color_FFFFFF,t_70)

client1 登录后，我们再去访问 client2 ，发现也不用登录了，直接就可以访问：  
![](https://img-blog.csdnimg.cn/f3dd6a29e3ab41d8a98bd5f55094d94a.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YW5neHVhbmhlbmc=,size_16,color_FFFFFF,t_70)

OK，如此之后，我们的单点登录就成功了。

### 流程解析

最后，我再来和小伙伴们把上面代码的一个执行流程捋一捋：

1. 首先我们去访问 client1 的 /getLogin 接口，这个接口是需要登录才能访问的，因此我们的请求被拦截下来，拦截下来之后，系统会给我们重定向到 client1 的 /login 接口，这是让我们去登录。

![](https://img-blog.csdnimg.cn/0f7abbcc7dd14cc4a999452733e33d2c.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YW5neHVhbmhlbmc=,size_16,color_FFFFFF,t_70)

2. 当我们去访问 client1 的登录接口时，由于我们配置了 @EnableOAuth2Sso 注解，这个操作会再次被拦截下来，单点登录拦截器会根据我们在 application.properties 中的配置，自动发起请求去获取授权码：

![](https://img-blog.csdnimg.cn/adcb7be9084a472cb71bbc49fed19efd.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YW5neHVhbmhlbmc=,size_16,color_FFFFFF,t_70)

3. 在第二步发送的请求是请求 auth-server 服务上的东西，这次请求当然也避免不了要先登录，所以再次重定向到 auth-server 的登录页面，也就是大家看到的统一认证中心。  
![](https://img-blog.csdnimg.cn/14222213f41e44b88832fe409a3ca45c.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YW5neHVhbmhlbmc=,size_16,color_FFFFFF,t_70)

4. 在统一认证中心完成登录功能之后，会继续执行第二步的请求，这个时候就可以成功获取到授权码了。  
![](https://img-blog.csdnimg.cn/3609b3068f61415a9f7855dd13cb07d7.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YW5neHVhbmhlbmc=,size_16,color_FFFFFF,t_70)  
![](https://img-blog.csdnimg.cn/5898569ba6d04b349ffa74a0524d4ee5.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YW5neHVhbmhlbmc=,size_16,color_FFFFFF,t_70)

5. 获取到授权码之后，这个时候会重定向到我们 client1 的 login 页面，但是实际上我们的 client1 其实是没有登录页面的，所以这个操作依然会被拦截，此时拦截到的地址包含有授权码，拿着授权码，在 OAuth2ClientAuthenticationProcessingFilter 类中向 auth-server 发起请求，就能拿到 access_token 了。  
![](https://img-blog.csdnimg.cn/079ec42f3fe44de7a8074f26739a26e6.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h1YW5neHVhbmhlbmc=,size_16,color_FFFFFF,t_70)

6. 在第五步拿到 access_token 之后，接下来在向我们配置的 user-info-uri 地址发送请求，获取登录用户信息，拿到用户信息之后，在 client1 上自己再走一遍 Spring Security 登录流程，这就 OK 了。

需要项目学习的可以到我的 github 上去 clone 下来  
[GitHub 源码地址](https://github.com/huangxuanheng/oauth2)