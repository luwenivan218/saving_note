> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1131244380)

> Java9 提供了新的 HTTP 客户端（HttpClient）来处理 HTTP 调用，但是那时还处于孵化器阶段，经历了 Java9、Java10，终于在 Java11 版本正式 “转正”。为什么要提供全新的 HTTP 客

原

 2023-11-28  阅读 (95)  点赞 (0)

Java 9 提供了新的 HTTP 客户端（`HttpClient`）来处理 HTTP 调用，但是那时还处于孵化器阶段，经历了 Java 9 、Java 10 ，终于在 Java11 版本正式 “转正”。

![](https://sike.skjava.com/java-features/202311081000002.png)

为什么要提供全新的 HTTP 客户端
------------------

Java 11 引入新的 HTTP 客户端的原因是为了解决旧的`HttpURLConnection`类存在的多个问题和局限性。`HttpURLConnection` 存在如下几个局限性：

1.  **不直观的 API 设计**：`HttpURLConnection` 的 API 设计过于复杂，对于新手比较难上手，例如我们必须手动处理输入和输出流，以及错误处理。
2.  **不支持 HTTP/2**：`HttpURLConnection`仅支持 HTTP/1.1，不支持 HTTP/2。HTTP/2 支持多路复用、服务器推送和头部压缩等多项特性，这些都能提高网络通信的效率。
3.  **性能问题**：`HttpURLConnection` 在高并发场景下性能表现不是很好，尤其是同步阻塞的特性更是性能的瓶颈。
4.  **缺乏现代 Web 特性的支持**：随着 Web 技术的发展，如 WebSocket 等技术变得越来越重要，而`HttpURLConnection`并不支持这些。
5.  **缺乏异步处理能力**：在处理 HTTP 请求时，`HttpURLConnection`不支持异步模式。
6.  **错误处理繁琐**：`HttpURLConnection` 的错误处理比较麻烦。我们需要检查特定的 HTTP 错误代码，并对不同的响应代码进行不同的处理。
7.  **连接管理不足**：`HttpURLConnection` 不支持连接池、不支持自动重试。
8.  **不可变对象和线程安全**：`HttpURLConnection`不是不可变的，也不是线程安全的。

基于上述几点，Java 11 引入 `HttpClient` 来替代`HttpURLConnection`。相比`HttpURLConnection` ，`HttpClient` 具有如下优势：

1.  **支持 HTTP/2**：`HttpClient` 支持 HTTP/2 协议的所有特性，包括同步和异步编程模型。
2.  **支持** ** WebSocket**：支持 WebSocket，允许建立持久的连接，并进行全双工通信。
3.  **更简洁的 API**：`HttpClient`提供了更简洁、更易于使用的 API，我们能够非常方便地发送 HTTP 请求。
4.  **支持同步和异步**：提供了同步和异步的两种模式。这意味着我们既可以等待 HTTP 响应，也可以使用回调机制处理 HTTP 响应。
5.  **链式调用**：新的`HttpClient` 允许链式调用，使得构建和发送请求变得更简单。
6.  **更好的错误处理**机制：新的`HttpClient` 提供了更好的错误处理机制，当 HTTP 请求失败时，可以通过异常机制更清晰地了解到发生了什么。

核心类介绍
-----

HttpClient 的主要类有下面三个：

*   `java.net.http.HttpClient`
*   `java.net.http.HttpRequest`
*   `java.net.http.HttpResponse`

### HttpClient

`HttpClient`是用于发送请求和获取响应的客户端。它可以发送同步和异步的 HTTP 请求，并支持 HTTP/1.1、HTTP/2 以及 WebSocket。

由于`HttpClient`是不可变的，所以它是线程安全的，可以重用以发送多个请求。

利用`HttpClient.newBuilder()` 可以创建`HttpClient`的实例，并可以配置各种属性，如：

<table><thead><tr><th>priority</th><th>配置属性</th></tr></thead><tbody><tr><td><code>version()</code></td><td>指定 HTTP 协议的版本（HTTP/1.1 或 HTTP/2）</td></tr><tr><td><code>followRedirects()</code></td><td>设置重定向策略</td></tr><tr><td><code>proxy()</code></td><td>设置代理服务器</td></tr><tr><td><code>authenticator()</code></td><td>设置 HTTP 身份验证</td></tr><tr><td><code>connectTimeout()</code></td><td>置建立 HTTP 连接的超时时间</td></tr><tr><td><code>executor()</code></td><td>设置自定义的 Executor，该 Executor 用于异步任务</td></tr><tr><td><code>cookieHandler()</code></td><td>设置 Cookie 处理器</td></tr><tr><td><code>sslParameters()</code></td><td>设置 SSL/TLS 的配置</td></tr><tr><td><code>priority()</code></td><td>设置请求的优先级</td></tr></tbody></table>

这些方法可以使用链式的方式配置在一起，一旦配置完成，调用`build()`方法就会创建一个新的`HttpClient`实例，该实例包含所有配置的设置。例如：

```
HttpClient client = HttpClient.newBuilder()
                .version(HttpClient.Version.HTTP_2)
                .followRedirects(HttpClient.Redirect.NEVER)
                .proxy(ProxySelector.of(new InetSocketAddress("127.0.0.1", 8808)))
                .connectTimeout(Duration.ofMinutes(1))
                .build();


```

`HttpClient`

### HttpRequest

`HttpRequest` 表示一个 HTTP 请求，它也是不可变的，所以一旦创建，就不能更改它的配置和数据。使用 `HttpRequest.newBuilder()` 来创建 `HttpRequest` 实例，然后链式调用设置方法来配置请求，配置项如下：

<table><thead><tr><th>方法名</th><th>配置项</th></tr></thead><tbody><tr><td><code>uri()</code></td><td>设置请求的统一资源标识符 (URI)</td></tr><tr><td><code>timeout()</code></td><td>设置请求的超时时间</td></tr><tr><td><code>header()</code></td><td>添加单个请求头</td></tr><tr><td><code>headers()</code></td><td>批量添加请求头</td></tr><tr><td><code>version()</code></td><td>指定 HTTP 协议的版本</td></tr><tr><td><code>expectContinue()</code></td><td>设置 <code>Expect: 100-Continue</code> 请求头</td></tr><tr><td><code>setHeader()</code></td><td>设置特定的请求头，如认证头</td></tr></tbody></table>

`HttpRequest` 支持 GET、POST、DELETE 等方法，对应的方法如下：

<table><thead><tr><th>方法名</th><th>描述</th></tr></thead><tbody><tr><td><code>GET()</code></td><td>创建一个 GET 请求</td></tr><tr><td><code>POST(HttpRequest.BodyPublisher body)</code></td><td>创建一个带有请求体的 POST 请求</td></tr><tr><td><code>PUT(HttpRequest.BodyPublisher body)</code></td><td>创建一个带有请求体的 PUT 请求</td></tr><tr><td><code>DELETE()</code></td><td>创建一个 DELETE 请求</td></tr><tr><td><code>method(String method, HttpRequest.BodyPublisher body)</code></td><td>创建一个自定义方法的请求</td></tr></tbody></table>

下面代码是创建一个 GET 请求：

```
HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://www.skjava.com"))
                .version(HttpClient.Version.HTTP_2)
                .timeout(Duration.ofSeconds(10))
                .GET()
                .build();


```

对于需要请求体的请求，例如 POST、PUT，我们可以使用`HttpRequest.BodyPublishers`来提供请求体的内容。

### HttpResponse

`HttpResponse` 表示一个 HTTP 响应，它包含 HTTP 响应的所有信息，包括状态码、响应头和响应体。与`HttpClient`和`HttpRequest`一样，`HttpResponse`也是不可变的。

当我们利用 `HttpClient` 发送 HTTP 请求时，需要指定一个`HttpResponse.BodyHandler`来处理响应体。这个处理器决定了如何处理响应数据，比如将其作为 String，如下：

```
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());



```

`HttpResponse` 方法如下：

<table><thead><tr><th>方法名</th><th>描述</th></tr></thead><tbody><tr><td><code>statusCode()</code></td><td>获取 HTTP 响应的状态码</td></tr><tr><td><code>body()</code></td><td>获取 HTTP 响应体</td></tr><tr><td><code>headers()</code></td><td>获取 HTTP 响应头</td></tr><tr><td><code>uri()</code></td><td>获取请求的 URI</td></tr><tr><td><code>version()</code></td><td>获取响应的 HTTP 协议版本</td></tr><tr><td><code>request()</code></td><td>获取生成此响应的<code>HttpRequest</code>对象</td></tr><tr><td><code>previousResponse()</code></td><td>获取重定向之前的响应，如果有的话</td></tr><tr><td><code>sslSession()</code></td><td>获取 SSL 会话信息，如果在 SSL 连接上获得响应，则为<code>Optional&lt;SSLSession&gt;</code></td></tr><tr><td><code>trailers()</code></td><td>异步获取 HTTP 尾部头，返回<code>CompletableFuture&lt;HttpHeaders&gt;</code></td></tr></tbody></table>

示例
--

`HttpClient` 可以使用同步发送和异步发送。

### 同步发送

`HttpClient` 提供了 `send()` 方法，发送请求后会一直阻塞到收到 response 为止。

```
    @Test
    public void sendTest() {
        
        HttpClient httpClient = HttpClient.newBuilder()
                .version(HttpClient.Version.HTTP_2)     
                .connectTimeout(Duration.ofSeconds(10))  
                .build();

        
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://skjava.com/article/list?page=3")) 
                .timeout(Duration.ofSeconds(5))     
                .GET().build();

        HttpResponse<String> response = null;
        try {
            response = httpClient.send(request, HttpResponse.BodyHandlers.ofString());

            if (response.statusCode() == 200) {
                
                System.out.println("Response Body: " +  response.body());
            } else {
                
                System.err.println("Response Status Code: " + response.statusCode());
                System.err.println("Response Body: " + response.body());
            }
        } catch (IOException e) {
            
            System.err.println("IOException occurred: " + e.getMessage());
        } catch (InterruptedException e) {
            
            System.err.println("InterruptedException occurred: " + e.getMessage());
            
            Thread.currentThread().interrupt();
        }
    }


```

首先构建 `HttpClient` 和 `HttpRequest` 实例对象，调用 HttpClient 的 `send()` 方法发送 HTTP 同步请求，结果如下：

![](https://sike.skjava.com/java-features/202311081000001.jpg)

发送同步请求处理比较简单，我们只需要注意处理 `IOException` 和 `InterruptedException` 两个异常。

### 异步发送

`HttpClient` 提供了 `sendAsync()` 方法用于发送异步请求，发送请求后会立刻返回 `CompletableFuture`，然后我们可以使用`CompletableFuture`中的方法来设置异步处理器。

```
    @Test
    public void sendAsyncTest() {
        
        HttpClient httpClient = HttpClient.newBuilder()
                .version(HttpClient.Version.HTTP_2)     
                .connectTimeout(Duration.ofSeconds(10))  
                .build();

        
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("https://skjava.com/article/list?page=1")) 
                .timeout(Duration.ofSeconds(5))     
                .GET().build();

        
        CompletableFuture<HttpResponse<String>> futureResponse = httpClient.sendAsync(request, HttpResponse.BodyHandlers.ofString());

        
        futureResponse.thenApply(HttpResponse::body)    
                .thenAccept(System.out::println)        
                .join();                
    }


```

注意，异步请求 `sendAsync()` 返回的是一个 `CompletableFuture`，关于 可以看这篇文章：[Java 8 新特性—深入理解 CompletableFuture](https://www.skjava.com/series/article/5020516690)