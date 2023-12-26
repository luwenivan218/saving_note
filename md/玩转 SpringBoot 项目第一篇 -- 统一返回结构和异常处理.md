> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7316115845095915555)

技术栈
---

> SpringBoot 3.0.2
> 
> SpringCloudAlibaba 2022.0.0.0-RC2
> 
> JDK 17

项目结构
----

### 父项目 --learning

依赖版本相关的`pom`

```
<modules>  
    <module>user-center</module>  
    <module>common</module>  
    <module>spring-boot-starter-security</module>  
</modules>  
<properties>  
    <java.version>17</java.version>  
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>  
    <spring-boot.version>3.0.2</spring-boot.version>  
    <lombok.version>1.18.24</lombok.version>  
    <spring-cloud-alibaba.version>2022.0.0.0-RC2</spring-cloud-alibaba.version>  
    <mysql-connection-version>8.0.30</mysql-connection-version>  
    <mybatis-plus.version>3.4.1</mybatis-plus.version>  
    <fastjson.version>2.0.43</fastjson.version>  
</properties>


```

大家可以自行调整

### 子项目 --common

这里面将会存储整个项目的所有公共的配置，拦截器，数据模型等。所以这里也将统一返回结构和异常处理的部分放了进去。主要依赖如下，版本跟随父级`pom`

```
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter</artifactId>  
</dependency>  
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-web</artifactId>  
</dependency>  
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-web</artifactId>  
</dependency>  
<dependency>  
    <groupId>com.alibaba</groupId>  
    <artifactId>fastjson</artifactId>  
</dependency>  
<dependency>  
    <groupId>org.projectlombok</groupId>  
    <artifactId>lombok</artifactId>  
</dependency>


```

核心代码
----

### 返回结构 --ResponseResult

```
/**  
* 请求统一返回结果  
* @author yulbo  
* @date 2023/12/24 15:07  
*/  
@Data  
@ToString  
@AllArgsConstructor  
@NoArgsConstructor  
public class ResponseResult<T> implements Serializable {  
  
    /**  
    * 状态码  
    */  
    private Integer code;  

    /**  
    * 返回成功标识  
    */  
    private Boolean success;  

    /**  
    * 返回信息（异常信息）  
    */  
    private String message;  

    /**  
    * 返回结果  
    */  
    private T data;  

    /**  
    * 全参数方法  
    *  
    * @param code 状态码  
    * @param status 状态  
    * @param message 返回信息  
    * @param data 返回数据  
    * @param  
    * @return {@link ResponseResult}  
    */  
    private static ResponseResult response(Integer code, Boolean status, String message, Object data) {  
        ResponseResult responseResult = new ResponseResult();  
        responseResult.setCode(code);  
        responseResult.setSuccess(status);  
        responseResult.setMessage(message);  
        responseResult.setData(data);  
        return responseResult;  
    }  

    /**  
    * 全参数方法  
    *  
    * @param code 状态码  
    * @param status 状态  
    * @param message 返回信息  
    * @param  
    * @return {@link ResponseResult}  
    */  
    private static ResponseResult response(Integer code, Boolean status, String message) {  
        ResponseResult responseResult = new ResponseResult();  
        responseResult.setCode(code);  
        responseResult.setSuccess(status);  
        responseResult.setMessage(message);  
        return responseResult;  
    }  

    /**  
    * 成功返回（数据）  
    *  
    * @param data 数据  
    * @param  
    * @return {@link ResponseResult}  
    */  
    public static ResponseResult success(Object data) {  
        return response(HttpStatusEnum.SUCCESS.getCode(), true, null, data);  
    }  


    /**  
    * 失败返回（状态码+返回信息）  
    *  
    * @param code 状态码  
    * @param message 返回信息  
    * @param  
    * @return {@link ResponseResult}  
    */  
    public static ResponseResult fail(Integer code, String message) {  
        return response(code, false, message);  
    }  


    /**  
    * 失败返回（返回信息）  
    *  
    * @param message 返回信息  
    * @param  
    * @return {@link ResponseResult}  
    */  
    public static ResponseResult fail(String message) {  
        return response(HttpStatusEnum.ERROR.getCode(), false, message, null);    
    }  
}


```

这个类就是返回结构，同时对外提供了成功和返回的方法。对应的状态码枚举如下

```
@Getter  
public enum HttpStatusEnum {  
    /**  
    * 操作成功  
    */  
    SUCCESS(200, "操作成功"),  
    /**  
    * 对象创建成功  
    */  
    CREATED(201, "对象创建成功"),  
    /**  
    * 请求已经被接受  
    */  
    ACCEPTED(202, "请求已经被接受"),  
    /**  
    * 操作已经执行成功，但是没有返回数据  
    */  
    NO_CONTENT(204, "操作已经执行成功，但是没有返回数据"),  
    /**  
    * 资源已被移除  
    */  
    MOVED_PERM(301, "资源已被移除"),  
    /**  
    * 重定向  
    */  
    SEE_OTHER(303, "重定向"),  
    /**  
    * 资源没有被修改  
    */  
    NOT_MODIFIED(304, "资源没有被修改"),  
    /**  
    * 参数列表错误（缺少，格式不匹配）  
    */  
    BAD_REQUEST(400, "参数列表错误（缺少，格式不匹配）"),  
    /**  
    * 未授权  
    */  
    UNAUTHORIZED(401, "未授权"),  
    /**  
    * 访问受限，授权过期  
    */  
    FORBIDDEN(403, "访问受限，授权过期"),  
    /**  
    * 资源，服务未找到  
    */  
    NOT_FOUND(404, "资源，服务未找！"),  
    /**  
    * 不允许的http方法  
    */  
    BAD_METHOD(405, "不允许的http方法"),  
    /**  
    * 资源冲突，或者资源被锁  
    */  
    CONFLICT(409, "资源冲突，或者资源被锁"),  
    /**  
    * 不支持的数据，媒体类型  
    */  
    UNSUPPORTED_TYPE(415, "不支持的数据，媒体类型"),  
    /**  
    * 系统内部错误  
    */  
    ERROR(500, "系统内部错误"),  
    /**  
    * 接口未实现  
    */  
    NOT_IMPLEMENTED(501, "接口未实现"),  
    /**  
    * 系统警告消息  
    */  
    WARN(601,"系统警告消息");  

    private final Integer code;  
    private final String message;  

    HttpStatusEnum(Integer code, String message) {  
        this.code = code;  
        this.message = message;  
    }  
}


```

可以根据异常指定不同的异常类型指定不同的状态码，比如如果是权限问题，涉及到前端页面的跳转，那么就需要和其他的异常`code`区分出来。

### 正常结果返回拦截器 --ResponseBodyHandler

```
@RestControllerAdvice  
public class ResponseBodyHandler implements ResponseBodyAdvice<Object> {  
    @Override  
    public boolean supports(MethodParameter returnType
    , Class<? extends HttpMessageConverter<?>> converterType) {  
        return true;  
    }  

    @Override  
    public Object beforeBodyWrite(Object body, MethodParameter returnType
    , MediaType selectedContentType
    , Class<? extends HttpMessageConverter<?>> selectedConverterType
    , ServerHttpRequest request, ServerHttpResponse response) {  
        if(body instanceof ResponseResult){  
            return body;  
        }  
        if(body instanceof String){  
            return JSON.toJSONString(ResponseResult.success(body));  
        }  
        return ResponseResult.success(body);  
    }  
}


```

主要方法就是`beforeBodyWrite`，这里面有个特殊的就是需要对`String`类型的特殊处理，大家可以试一下去掉的效果，有一个类型转换的异常。

### 异常拦截器 --

```
@RestControllerAdvice  
@Slf4j  
public class GlobalExceptionHandler {  
  
    @ExceptionHandler(value = BusinessException.class)  
    public ResponseResult handleBusinessException(BusinessException bx) {  
        log.info("接口调用业务异常信息{}", bx.getMessage());  
        // 返回统一处理类  
        return ResponseResult.fail( bx.getMessage());  
    }  
    /**  
    * 捕获 {@code ParamException} 异常  
    */  
    @ExceptionHandler(value = ParamException.class)  
        public ResponseResult paramExceptionHandler(ParamException ex) {  
        log.info("接口调用异常：{},异常信息{}", ex.getMessage(), ex.getMessage());  
        // 返回统一处理类  
        return ResponseResult.fail(ex.getMessage());  
    }  

    /**  
    * 顶级异常捕获并统一处理，当其他异常无法处理时候选择使用  
    */  
    @ExceptionHandler(value = Exception.class)  
        public ResponseResult handle(Exception ex) {  
        log.info("接口调用异常：{},异常信息{}", ex.getMessage(), ex.getMessage());  
        // 返回统一处理类  
        return ResponseResult.fail(ex.getMessage());  
    }  
}


```

根据异常类型选择不同的返回信息或者处理情况等等。如果引入了`Swagger`等接口文档工具，需要在注解`@RestControllerAdvice`指定扫描包的路径，也就是`Controller`的路径。异常类示例如下

```
public class AccessDenialException extends RuntimeException{  
  
    private Integer code;  

    private String message;  

    /**  
    * 默认信息：未授权，状态码：401  
    * @author yulbo  
    * @date 2023/12/24 21:34  
    */  
    public AccessDenialException(){  
        super("未授权");  
        this.code = HttpStatusEnum.UNAUTHORIZED.getCode();  
    }  

    /**  
    * 默认状态码：401  
    * @author yulbo  
    * @date 2023/12/24 21:35  
    */  
    public AccessDenialException(String message) {  
        super(message);  
        this.message = message;  
        this.code = HttpStatusEnum.UNAUTHORIZED.getCode();  
    }  

    public AccessDenialException(Integer code,String message){  
        super("未授权");  
        this.code = code;  
        this.message = message;  
    }  
  
}


```

### 配置类 --CommonConfiguration

对外提供拦截器的`Bean`。

```
@Configuration  
public class CommonConfiguration {  
  
    @ConditionalOnMissingBean  
    @Bean  
    public ResponseBodyHandler responseBodyHandler() {  
        return new ResponseBodyHandler();  
    }  

    @ConditionalOnMissingBean  
    @Bean  
    public GlobalExceptionHandler globalExceptionHandler() {  
        return new GlobalExceptionHandler();  
    }  
}


```

可以选择两种方式导入配置类，一种是在启动类上面`@Import`导入配置，一种是依赖于`SpringBoot`的扫描配置类的机制，在`SpringBoot3`的版本改变了扫描的结构，扫描的是`resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`这个文件里面的配置，在`SpringBoot2`的版本中是`resources/META-INF/spirng.factories，`这是个变化。我选择的是扫描注入的方式。

测试
--

新包`userCenter`下测试，创建一个`Controller`，然后发起调用，正常结果如下

```
@RestController  
@RequestMapping("userCenter/rest/v1/user")  
public class UserController {  
  
    @GetMapping("/testSuccess")  
    public Object testSuccess(){  
        return "123213";  
    }  
  
    @GetMapping("/testFail")  
    public Object testFail(){  
        throw new AccessDenialException();  
    } 
}


```

返回结果

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/62210d89a2394255b11f416793bc3aab~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1167&h=147&s=19956&e=png&b=1e1e1e) 异常情况返回结果

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a129c09618d3450b9c857be111a5751d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1240&h=189&s=15312&e=png&b=1e1e1e)