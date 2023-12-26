> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7315730050577006592)

> 大家好，我是大明哥，一个专注「[死磕 Java](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fsike-java "https://www.skjava.com/sike-java")」系列创作的硬核程序员。 本文已收录到我的小站：[skjava.com](https://link.juejin.cn?target=https%3A%2F%2Fskjava.com%2F "https://skjava.com/")。

> Java 现在发布的版本很快，每年两个，但是真正会被大规模使用的是 3 年一个的 LTS 版本。

*   每 3 年发布一个 LTS（Long-Term Support），长期维护版本。意味着只有 **Java 8 ，Java 11， Java 17，Java 21 才可能被大规模使用**。
*   每年发布两个正式版本，分别是 3 月份和 9 月份。

在 Java 版本中，一个特性的发布都会经历孵化阶段、预览阶段和正式版本。其中孵化和预览可能会跨越多个 Java 版本。所以大明哥在介绍 Java 新特性时采用如下这种策略：

1.  每个版本的新特性，大明哥都会做一个简单的概述。
    
2.  单独出文介绍跟编码相关的新特性，一些如 JVM、性能优化的新特性不单独出文介绍。
    
3.  孵化阶段的新特性不出文介绍。
    
4.  首次引入为预览特性、新特性增强、首次引入的正式特性，单独出文做详细介绍。
    
5.  影响比较大的新特性如果在现阶段没有转正的新特性不单独出文介绍，单独出文的重大特性一般都在 Java 21 版本之前已转为正式特性，例如：
    
    1.  虚拟线程，Java 19 引入的，在 Java 21 转正，所以大明哥会在 Java 19 单独出文做详细介绍
    2.  作用域值，Java 20 引入的，但是在 Java 21 还处于预览阶段，所以不做介绍，等将来转正后会详细介绍

Java 8 新特性
----------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de7595f1d8174e589a5953ea5877330f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2062&h=1252&s=244686&e=png&b=ffffff)

*   JEP 126：Lambda 表达式：[Java 8 新特性—Lambda 表达式](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1747461365 "https://www.skjava.com/series/article/1747461365")
*   JEP 126：函数式接口：[Java 8 新特性—函数式接口](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1841959591 "https://www.skjava.com/series/article/1841959591")
*   JEP 179：方法引用：[Java 8 新特性—方法引用和构造器引用](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1019158498 "https://www.skjava.com/series/article/1019158498")
*   JEP 150：接口的默认方法：[Java 8 新特性—接口默认方法和静态方法](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F9298099860 "https://www.skjava.com/series/article/9298099860")
*   JEP 107：Stream API：[Java 8 新特性—Stream API 对元素流进行函数式操作](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F6972779809 "https://www.skjava.com/series/article/6972779809")
*   Optional 类：[Java 8 新特性— 利用 Optional 解决 NullPointerException](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1810194665 "https://www.skjava.com/series/article/1810194665")
*   JEP 170：新的日期时间 API：[Java 8 新特性—日期时间 API](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1658730277 "https://www.skjava.com/series/article/1658730277") && [Java 8 新特性—日期时间格式化](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1765935055 "https://www.skjava.com/series/article/1765935055")
*   JEP 120：重复注解：[Java 8 新特性—重复注解 @Repeatable](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F8453320489 "https://www.skjava.com/series/article/8453320489")
*   Base64 编码解码：[Java 8 新特性—全新的、标准的 Base64 API](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1066055319 "https://www.skjava.com/series/article/1066055319")
*   JEP 104：类型注解：[Java 8 新特性—类型注解](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F2019475189 "https://www.skjava.com/series/article/2019475189")
*   JEP 101：类型推断优化：[Java 8 新特性—类型推断优化](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F6176475946 "https://www.skjava.com/series/article/6176475946")
*   JEP 174：Nashorn JavaScript 引擎
*   JEP 122：移除 Permgen

Java 9 新特性
----------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff3a1e7913b9419ab6b4a6eb546149ec~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2082&h=1420&s=325083&e=png&b=ffffff)

*   JEP 261: 模块系统：[Java 9 新特性—模块化](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F2044128092 "https://www.skjava.com/series/article/2044128092")
*   JEP 269: 集合工厂方法：[Java 9 新特性—新增只读集合和工厂方法](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F6358413511 "https://www.skjava.com/series/article/6358413511")
*   JEP 222：Jshell：[Java 9 新特性—REPL 工具：JSheel 命令](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F2132633297 "https://www.skjava.com/series/article/2132633297")
*   JEP 213：接口支持私有方法：[Java 9 新特性—接口支持私有方法](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F4535468349 "https://www.skjava.com/series/article/4535468349")
*   Stream API 增强：[Java 9 新特性—Stream API 的增强](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F2547807010 "https://www.skjava.com/series/article/2547807010")
*   Optional 的增强：J[ava 9 新特性—Optional 的增强](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1624717282 "https://www.skjava.com/series/article/1624717282")
*   改进 try-with-resources：[Java 9 新特性—try-with-resources 的升级](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1487108972 "https://www.skjava.com/series/article/1487108972")
*   JEP 102：Process API
*   JEP 264：平台日志 API 和 服务
*   JEP 266: 反应式流（Reactive Streams）
*   JEP 224: HTML5 Javadoc
*   JEP 238: 多版本兼容 JAR 文件
*   JEP 277：改进的弃用注解 @Deprecated
*   JEP 213：改进钻石操作符 (Diamond Operator)
*   增强 CompletableFuture：[Java 9 新特性—改进 CompletableFuture](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1843207164 "https://www.skjava.com/series/article/1843207164")

Java 10 新特性
-----------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2937c3cacc9446798057305390c10f9b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2516&h=1092&s=265786&e=png&b=ffffff)

*   JEP 286：局部变量类型推断：[Java 10 新特性—局部变量类型推断](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1075067951 "https://www.skjava.com/series/article/1075067951")
*   JEP 304：统一的垃圾回收接口
*   JEP 307：并行全垃圾回收器 G1
*   JEP 310：应用程序类数据共享
*   JEP 312：线程 - 局部变量管控
*   JEP 313：移除 Native-Header 自动生成工具
*   JEP 314：额外的 Unicode 语言标签扩展
*   JEP 316：备用存储装置上的堆分配
*   JEP 317：基于 Java 的 实验性 JIT 编译器
*   JEP 319：根证书认证
*   JEP 322：基于时间的版本发布模式
*   新增 API

Java 11 新特性
-----------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4cd5def904184e6ab8e1d48e7f402e4f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2362&h=1092&s=256734&e=png&b=ffffff)

*   JEP 181: 基于嵌套的访问控制
*   新增 String API：[Java 11 新特性—新增 String API](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F9457186945 "https://www.skjava.com/series/article/9457186945")
*   JEP 321：全新的 HTTP 客户端 API：[Java 11 新特性—全新的 HTTP 客户端 API](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1131244380 "https://www.skjava.com/series/article/1131244380")
*   JEP 323：局部变量类型推断的升级：[Java 11 新特性—局部变量类型推断的增强](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1501251408 "https://www.skjava.com/series/article/1501251408")
*   JEP 318：Epsilon—低开销垃圾回收器
*   ZGC：可伸缩低延迟垃圾收集器
*   JEP 335：废弃 Nashorn JavaScript 引擎
*   增加 Files API：[Java 11 新特性—新增 Files API](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F8589000958 "https://www.skjava.com/series/article/8589000958")
*   Optional API 增强：[Java 11 新特性—Optional API 的增强](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1414447929 "https://www.skjava.com/series/article/1414447929")
*   JEP 328：飞行记录器（Flight Recorder）
*   JEP 330：运行单文件源码程序
*   JEP 320：删除 Java EE 和 corba 模块

Java 12 新特性
-----------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de929d47888c40d6b6f29bff4a738a1a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2362&h=974&s=209212&e=png&b=ffffff)

*   JEP 189：Shenandoah 垃圾收集器（预览特性）
*   JEP 325：Switch 表达式（预览特性）：[Java 12 新特性—Switch 表达式](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F5782529848 "https://www.skjava.com/series/article/5782529848")
*   JEP 334：JVM 常量 API
*   JEP 230：微基准测试套件（JMH）的支持
*   新增 String API：[Java 12 新特性—新增 String API](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F6401260394 "https://www.skjava.com/series/article/6401260394")
*   新增 Files API：[Java 12 新特性—新增 Files API](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F2000691792 "https://www.skjava.com/series/article/2000691792")
*   新增 NumberFormat API：[Java 12 新特性—新增 NumberFormat API](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1940664105 "https://www.skjava.com/series/article/1940664105")
*   新增 Collectors API：[Java 12 新特性—新增 Collectors API](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F2901979956 "https://www.skjava.com/series/article/2901979956")
*   JEP 340：移除多余 ARM64 实现
*   JEP 341：默认 CDS 归档
*   JEP 344：G1 的可中断 mixed GC

Java 13 新特性
-----------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/72d494035c4a4b7fa89306d227c33e19~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2362&h=612&s=113161&e=png&b=fffefe)

*   JEP 354：增强 Switch 表达式（第二次预览）：[Java 13 新特性—增强 Switch 表达式](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F9120322209 "https://www.skjava.com/series/article/9120322209")
*   JEP 355：文本块（预览特性）：[Java 13 新特性—文本块](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F7430445828 "https://www.skjava.com/series/article/7430445828")
*   JEP 353：重构 Socket API
*   JEP 350：动态 CDS
*   JEP 351：增强 ZGC 释放未使用内存

Java 14 新特性
-----------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/04efc62680f84d96bcc1ec01108fa8c4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2798&h=1252&s=352403&e=png&b=fffefe)

*   JEP 361： 表达式（正式特性）
*   JEP 368：增强文本块（第二次预览）
*   JEP 359：Records (预览) ：[Java 14 新特性—新特性 Record 类型](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F8932587293 "https://www.skjava.com/series/article/8932587293")
*   JEP 305：模式匹配的 instanceof（预览）：[Java 14 新特性—模式匹配的 instanceof](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F3436282193 "https://www.skjava.com/series/article/3436282193")
*   JEP 358：改进 NullPointerExceptions 提示信息：[Java 14 新特性—改进 NullPointerExceptions 提示信息](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1662953266 "https://www.skjava.com/series/article/1662953266")
*   JEP 343：打包工具（孵化）
*   JEP 345：NUMA-Aware 内存分配
*   JEP 349：JFR Event Streaming
*   JEP 364：macOS 上的 ZGC（实验性）
*   JEP 365：Windows 上的 ZGC（实验性）
*   JEP 366：弃用 ParallelScavenge + SerialOld GC 组合
*   JEP 367：删除 Pack200 工具和 API
*   JEP 363： 删除 CMS 垃圾收集器
*   JEP 370：外部存储器访问 API（孵化器版）

Java 15 新特性
-----------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/56de911f8c4a4aad982e27eb5f64bbd4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2934&h=1252&s=375134&e=png&b=fffefe)

*   JEP 339：Edwards-Curve 数字签名算法 (EdDSA)
*   JEP 360：密封的类和接口（预览）：[Java 15 新特性—密封的类与接口](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1777232038 "https://www.skjava.com/series/article/1777232038")
*   JEP 371：隐藏类 Hidden Classes：[Java 15 新特性—隐藏类](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1690418335 "https://www.skjava.com/series/article/1690418335")
*   JEP 372：移除 Nashorn JavaScript 引擎
*   JEP 373：重新实现 DatagramSocket API
*   JEP 374：禁用偏向锁定
*   JEP 375：模式匹配的 instanceof（第二次预览）
*   JEP 377：ZGC—可伸缩低延迟垃圾收集器（正式特性）
*   JEP 378：文本块（正式特性）
*   JEP 379：Shenandoah—低暂停时间垃圾收集器（正式特性）
*   JEP 381：移除 Solaris 和 SPARC 支持
*   JEP 383：外部存储器访问 API （二次孵化器版）
*   JEP 384： Record (第二次预览)
*   JEP 385： 废除 RMI 激活

Java 16 新特性
-----------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/070e3f7bd2f24e689dd14fb01da6e4df~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2606&h=1566&s=422218&e=png&b=ffffff)

*   JEP 338：向量 API（孵化器）
*   JEP 347：启用 C++14 语言特性
*   JEP 357：将 JDK 的源代码库从 Mercurial 迁移到 Git
*   JEP 369：将 JDK 的源代码库托管到 GitHub
*   JEP 376：ZGC 并发线程处理
*   JEP 380：Unix-Domain 套接字通道
*   JEP 386：AlpineLinux 移植
*   JEP 387：弹性元空间
*   JEP 388：Windows/AArch64 移植
*   JEP 389：外部函数与内存 API（孵化器）
*   JEP 390：对基于值的类发出警告
*   JEP 392：打包工具（正式版）
*   JEP 393：外部存储器访问 API（第三次孵化）
*   JEP 394：instanceof 模式匹配（正式特性）
*   JEP 395：Records (正式特性)
*   JEP 396：默认强封装 JDK 内部元素
*   JEP 397：密封类（第二预览）

Java 17 新特性
-----------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c4b82613ed67435793207089c512ccda~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2462&h=1252&s=310698&e=png&b=ffffff)

*   JEP 356：增强型伪随机数生成器：[Java 17 新特性—增强型伪随机数生成器](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1777241237 "https://www.skjava.com/series/article/1777241237")
*   JEP 382：新的 macOS 渲染管线
*   JEP 391：macOS/AArch64 端口
*   JEP 398：移除 Applet API
*   JEP 406：模式匹配的 Swith 表达式（预览）：[Java 17 新特性—模式匹配的 Swith 表达式](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F9479813794 "https://www.skjava.com/series/article/9479813794")
*   JEP 407：删除 RMI 激活
*   JEP 409：密封类（正式特性）
*   JEP 410：移除实验性的 AOT 和 JIT 编译
*   JEP 411：废弃安全管理器
*   JEP 412：外部函数与内存 API（第二次孵化）
*   JEP 414：向量 API（第二次孵化）

Java 18 新特性
-----------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/00a631db5bb7450ab645ef5c1bfbbdbb~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2558&h=932&s=212730&e=png&b=ffffff)

*   JEP 400：默认 UTF-8 编码
*   JEP 408：简易 Web 服务器
*   JEP 413：支持在 Java API 文档中加入代码片段
*   JEP 416：用方法句柄重新实现核心反射
*   JEP 417：向量 API（第三孵化器）
*   JEP 418：互联网地址解析 SPI
*   JEP 419：外部函数和内存 API（第三孵化器）
*   JEP 420： 模式匹配 Switch 表达式（预览）
*   JEP 421：弃用 Finalization 功能

Java 19 新特性
-----------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13654de2c3f14cee8fb5be6ce1e695b0~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2386&h=772&s=164317&e=png&b=fffefe)

*   JEP 405：Record 模式（预览）：[Java 19 新特性—Record 模式](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F9674439600 "https://www.skjava.com/series/article/9674439600")
*   JEP 422：JDK 移植到 Linux/RISC-V
*   JEP 424：外部函数和内存 API（预览）
*   JEP 425：虚拟线程（预览）：[Java 19 新特性—虚拟线程](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1865448759 "https://www.skjava.com/series/article/1865448759")
*   JEP 426：向量 API（第四次孵化）
*   JEP 427：模式匹配的 Switch（第三次预览）
*   JEP 428：结构化并发（孵化功能）

Java 20 新特性
-----------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae3c0da9a97740a986a89698f64138d6~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2468&h=772&s=176426&e=png&b=fffefe)

*   JEP 429：作用域值（第一次孵化）
*   JEP 432：Record 模式（第二次预览）
*   JEP 433：模式匹配的 Switch 表达式（第四次预览）
*   JEP 434：外部函数与内存 API（第二次预览）
*   JEP 436：虚拟线程（第二次预览）
*   JEP 437：结构化并发（第二次孵化）
*   JEP 438：向量 API（第五次孵化）

Java 21 新特性
-----------

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/22c22d4eabf24efd8359db52084cac43~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2484&h=1412&s=348028&e=png&b=ffffff)

*   JEP 430：字符串模板 （预览）：[Java 21 新特性—字符串模板](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1880587197 "https://www.skjava.com/series/article/1880587197")
*   JEP 431：有序集合：[Java 21 新特性—有序集合](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1895017259 "https://www.skjava.com/series/article/1895017259")
*   JEP 439：分代 ZGC
*   JEP 440：Record 模式
*   JEP 441：switch 模式匹配
*   JEP 442：外部函数和内存 API （第三次预览）
*   JEP 443：未命名模式和变量 （预览）：[Java 21 新特性—未命名模式和变量](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F1961861176 "https://www.skjava.com/series/article/1961861176")
*   JEP 444：虚拟线程（正式特性）
*   JEP 445：未命名类和 main 方法 （预览）：[Java 21 新特性—未命名类和 main 方法](https://link.juejin.cn?target=https%3A%2F%2Fwww.skjava.com%2Fseries%2Farticle%2F6780334511 "https://www.skjava.com/series/article/6780334511")
*   JEP 446：作用域值 （预览）
*   JEP 448：向量 API（第六次孵化）
*   JEP 449：弃用 Windows 32 位 x86 端口
*   JEP 451：准备禁止动态加载代理
*   JEP 452：密钥封装机制 API 安全库
*   JEP 453：结构化并发（预览）