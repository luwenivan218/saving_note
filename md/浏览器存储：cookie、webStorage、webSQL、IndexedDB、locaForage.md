> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7316102254728134675)

随着前端业务的不断扩展，逻辑也越来越复杂，就连服务端存储都逐渐渗透到前端来。几种常见的浏览器存储，了解一下。

一、Cookie
--------

存储 cookie 是浏览器的功能，浏览器下有一个 cookie 文件夹专门存放各个域下设置的 cookie。Cookie 都是`name=value`的结构，name 和 value 都为字符串。另外，Cookie 是有生命周期的，在设置 Cookie 值时，可以同时设置有效期。当超过了这个有效期之后，Cookie 便会失效，前端请求时，不会携带过期的 Cookie。

特点：

1.  在首次访问网站时，浏览器发送请求中并未携带 Cookie。
2.  浏览器看到请求中未携带 Cookie，在 HTTP 的响应头中加入`Set-Cookie`。
3.  浏览器收到`Set-Cookie`后，会将 Cookie 保存下来
4.  下次再访问该网站时，HTTP 请求头就会携带 Cookie。

缺点：

1.  存储空间有限: 约 4K
2.  每次向服务器发请求都会被携带，增加 Web 请求大小
3.  只能存储字符串

使用：

```
// 设置
document.cookie = "a=1;";
document.cookie = "a=1; doamin=jzplp.com";
document.cookie = "a=1; doamin=jzplp.com；path=/abc";
//读取
document.cookie
// 删除，将它的有效期直接设置为过期，即可实现删除功能。
document.cookie = "a=1; max-age=-1";


```

二、webStorage
------------

包含两种存储方式，存储的值可以是字符串、数字、布尔、数组和对象。对象和数组必须转换为 string 进行存储。JSON.parse() 和 JSON.stringify() 方法可以将数组、对象等值类型转换为字符串类型，从而存储到 Storage 中。

特点：

1.  **localStorage** - 用于长久保存整个网站的数据，保存的数据没有过期时间，直到手动去除。
2.  **sessionStorage** - 用于临时保存同一窗口 (或标签页) 的数据，在关闭窗口或标签页之后将会删除这些数据。

缺点：

1.  操作过程是同步的会阻塞主线程
2.  存储大小约 5MB(不同浏览器，限制值不一样)
3.  只能存储字符串

使用：

```
// 存储
localStorage.setItem("username", "name"); // "name"
localStorage.setItem("count", 1); // "1"
localStorage.setItem("isOnline", true); // "true"
sessionStorage.setItem("username", "name");
// user 存储时，先使用 JSON 序列化，否则保存的是[object Object]
const user = { "username": "name" };
localStorage.setItem("user", JSON.stringify(user));
sessionStorage.setItem("user", JSON.stringify(user));
// 获取
localStorage.getItem("username");
sessionStorage.getItem("username");
// 删除
localStorage.removeItem("username");
sessionStorage.removeItem("username");
// 清空
localStorage.clear();
sessionStorage.clear();
// 检查是否存在
localStorage.hasOwnProperty("userName"); // true
sessionStorage.hasOwnProperty("userName"); // false


```

三、webSQL
--------

HTML5 引入了一种新的 Web SQL 数据库：关系数据库管理系统（RDBMS），它允许我们在浏览器中使用 SQL 语言进行数据存储和管理，而无需依赖于传统的服务器端数据库。

特点：

当用户在 Web 浏览器中使用 Web SQL 数据库时，数据实际上是存储在一个特定的文件中。这个文件通常位于用户的计算机上的浏览器的专属文件夹中。在不同的操作系统上，该文件夹的存储位置可能会有所不同。

缺点：

如今已经废弃，只有部分浏览器支持，推荐使用的是 webStorage 或者 indexedDB。具体用法就不再赘述了，跟后面 indexedDB 类似。

四、**IndexedDB**
---------------

浏览器提供的本地 nosql 数据库，它可以被网页脚本创建和操作。IndexedDB 允许储存大量数据，提供查找接口，还能建立索引。WebSQL 也是一种在浏览器里存储数据的技术, 跟 IndexedDB 不同的是, IndexedDB 更像是一个 NoSQL 数据库, 而 WebSQL 更像是关系型数据库, 使用 SQL 查询数据

特点：

1.  键值对存储： 内部采用对象仓库（Object store）存放数据。所有类型数据都可以直接存入。对象仓库中，数据以” 键值对 “形式保存，每个数据记录都有对应的主键
2.  异步：防止大量数据的读写，拖慢网页的展现
3.  支持事务：意味着一系列操作步骤中，只要有异步失败，整个事务就都取消，数据库回滚到事务发生之前的记录，不存在只改写部分数据的情况
4.  同源限制：每个数据库对应创建他的域名，网页只能访问自身域名下的数据库，而不能访问跨域数据库。
5.  支持二进制存储
6.  存储空间大： 取决于硬件。

*   Chrome 允许浏览器使用多达 80% 的总磁盘空间。一个源最多可以使用总磁盘空间的 60%。
*   IE 10 及以上最多可存储 250MB
*   Firefox 允许浏览器使用多达 50% 的可用磁盘空间
*   Safari 允许大约 1GB

缺点：

与大多数现代基于 Promise 的 API 不同，它是基于事件的，在使用前需要学习相关 API 进行设置，增加学习成本。

使用：

```
// 创建并打开数据库
let dbName = 'hello IndexedDB', version = 1, storeName = 'helloStore'；
 
let indexedDB = window.indexedDB
let db
const request = indexedDB.open(dbName, version)
request.onsuccess = function(event) {
    db = event.target.result // 数据库对象
    console.log('数据库打开成功')
}
// 添加数据
function addData(db, storeName, data) {
    // 事务对象 指定表格名称和操作模式（"只读"或"读写"）
    let request = db.transaction([storeName], 'readwrite') 
        .objectStore(storeName) // 仓库对象
        .add(data)
    request.onsuccess = function (event) {
        console.log('数据写入成功')
    }
    request.onerror = function (event) {
        console.log('数据写入失败')
        throw new Error(event.target.error)
    }
}
// 根据id获取数据
function getDataByKey(db, storeName, key) {
    let transaction = db.transaction([storeName]) // 事务
    let objectStore = transaction.objectStore(storeName) // 仓库对象
    let request = objectStore.get(key);
    // 同样支持onerror、onsuccess监听
}
// 根据id删除数据
function deleteDB(db, storeName, id) {
    let request = db.transaction([storeName], 'readwrite').objectStore(storeName).delete(id)
     // 同样支持onerror、onsuccess监听
}


```

效果如下：

*   ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d8acbd181e34c13a95ce629e831d1d4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=737&h=392&s=37246&e=png&b=fefefe)

五、**locaForage**
----------------

localForage 是一个 JavaScript 库，是对 IndexedDB 的封装，通过简单类似 localStorage API 的异步存储来改进你的 Web 应用程序的离线体验。它能存储多种类型的数据，而不仅仅是字符串。

特点：

1.  异步离线存储，以免阻塞应用程序
2.  用法上靠近 Promise, 方便执行回调
3.  写法简单，类似 Web Storage API
4.  支持存储多种类型数据：js 常见数据类型外，还有 Float32Array、Float64Array、Int8Array、Int16Array、Int32Array、Uint8Array、Uint8ClampedArray、Uint16Array、Uint32Array

6.  支持强制设置特定的驱动：IndexedDB、Web SQL、localStorage
7.  存在降级策略：若不支持 IndexedDB 或 Web SQL, 则使用 localStorage
8.  适合存储大量数据

使用：

localForage 提供回调 API 同时也支持 ES6 Promises API

```
//回调API形式
localforage.setItem('key', 'value', function (err) {
  // if err is non-null, we got an error
  localforage.getItem('key', function (err, value) {
    // if err is non-null, we got an error. otherwise, value is the value
  });
});
// ES6 Promise形式
localforage.setItem('key', 'value').then(function () {
  return localforage.getItem('key');
  }).then(function (value) {
    // we got our value
  }).catch(function (err) {
    console.log(err);
});
// async/await
try {
    const value = await localforage.getItem('somekey');
    console.log(value);
} catch (err) {
    console.log(err);
}


```

createInstance 创建并返回一个 localForage 的新实例。每个实例对象都有独立的数据库，而不会影响到其他实例

```
var store = localforage.createInstance({
  name: "nameHere"
});
var otherStore = localforage.createInstance({
  name: "otherName"
});

// 设置某个数据仓库 key 的值不会影响到另一个数据仓库
store.setItem("key1", "value1");
otherStore.setItem("key2", "value2");


```

效果如下

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d2897c69197147729dddb389cc9cffed~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1024&h=524&s=174618&e=png&b=fefefe)

六、对比
----

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/199e2d1b6cc044cfa982e8058c316461~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1024&h=216&s=78259&e=png&b=f9f9f9)

这里只是把几种存储方式拿出来对比了解，具体使用方法和场景可以再针对性去学习。