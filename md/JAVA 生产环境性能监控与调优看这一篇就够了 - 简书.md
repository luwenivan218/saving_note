> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/a75c1253fc11)

![](http://upload-images.jianshu.io/upload_images/14565748-c95b896f97ba6d57.jpg) JVM 的内存结构
------------------------------------------------------------------------------------------

本文主要内容包含

*   [JVM 的参数类型](#param-type)
*   [jinfo & jps(参数和进程查看)](#jinfo-jps)
*   [jstat(类加载、垃圾收集、JIT 编译)](#jstat)
*   [jmap+MAT(内存溢出)](#jmap)
*   [jstack(线程、死循环、死锁)](#jstack)
*   [JVisualVM(本地和远程可视化监控)](#JVisualVM)
*   [使用 BTrace 进行拦截调试](#BTrace)
*   [Tomcat 性能监控与调优](#Tomcat)
*   [Nginx 性能监控与调优](#Nginx)
*   [JVM 层 GC 调优](#jvm-gc)
*   [JAVA 代码层调优](#java)

### JVM 的参数类型

**标准参数**（各版本中保持稳定）

-help

-server -client

-version -showversion

-cp -classpath

**X 参数**（非标准化参数）

-Xint：解释执行

-Xcomp：第一次使用就编译成本地代码

-Xmixed：混合模式，JVM 自己决定是否编译成本地代码

示例：

java -version（默认是混合模式）

Java HotSpot(TM) 64-Bit Server VM (build 25.40-b25, mixed mode)

java -Xint -version

Java HotSpot(TM) 64-Bit Server VM (build 25.40-b25, interpreted mode)

**XX 参数**（非标准化参数）

主要用于 JVM 调优和 debug

*   Boolean 类型

```
格式：-XX:[+-]<name>表示启用或禁用 name 属性
如：-XX:+UseConcMarkSweepGC
-XX:+UseG1GC


```

*   非 Boolean 类型

```
格式：-XX:<name>=<value>表示 name 属性的值是 value
如：-XX:MaxGCPauseMillis=500
-xx:GCTimeRatio=19
-Xmx -Xms属于 XX 参数
-Xms 等价于-XX:InitialHeapSize
-Xmx 等价于-XX:MaxHeapSize
-xss 等价于-XX:ThreadStackSize


```

### 查看

jinfo -flag MaxHeapSize <pid>

-XX:+PrintFlagsInitial

-XX:+PrintFlagsFinal

-XX:+UnlockExperimentalVMOptions 解锁实验参数

-XX:+UnlockDiagnosticVMOptions 解锁诊断参数

-XX:+PrintCommandLineFlags 打印命令行参数

输出结果中 = 表示默认值，:= 表示被用户或 JVM 修改后的值

示例：java -XX:+PrintFlagsFinal -version

补充：测试中需要用到 Tomcat，CentOS 7 安装示例如下

```
sudo yum -y install java-1.8.0-openjdk*
wget  http://mirror.bit.edu.cn/apache/tomcat/tomcat-8/v8.5.32/bin/apache-tomcat-8.5.32.tar.gz
tar -zxvf apache-tomcat-8.5.32.tar.gz 
mv apache-tomcat-8.5.32 tomcat
cd tomcat/bin/
sh startup.sh


```

pid 可通过类似 ps -ef|grep tomcat 或 jps 来进行查看

### jps

详情参考 [jps 官方文档](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jps.html)

-l

### jinfo

jinfo -flag MaxHeapSize <pid>

jinfo -flags <pid>

### jstat

详情参考 [jstat 官方文档](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html)

![](http://upload-images.jianshu.io/upload_images/14565748-9fd2339713734e13.jpg) jstat 使用示例

**类加载**

```
# 以下1000表每隔1000ms 即1秒，共输出10次
jstat -class <pid> 1000 10


```

**垃圾收集**

-gc, -gcutil, -gccause, -gcnew, -gcold

```
jstat -gc <pid> 1000 10


```

以下大小的单位均为 KB

S0C, S1C, S0U, S1U: S0 和 S1 的总量和使用量

EC, EU: Eden 区总量与使用量

OC, OU: Old 区总量与使用量

MC, MU: Metacspace 区 (jdk1.8 前为 PermGen) 总量与使用量

CCSC, CCSU: 压缩类区总量与使用量

YGC, YGCT: YoungGC 的次数与时间

FGC, FGCT: FullGC 的次数与时间

GCT: 总的 GC 时间

**JIT 编译**

-compiler, -printcompilation

### jmap+MAT

详情参考 [jmap 官方文档](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jmap.html)

内存溢出演示：

[https://start.spring.io/](https://start.spring.io/) 生成初始代码

最终代码：[monitor_tuning](https://github.com/alanhou7/java-codes/tree/master/monitor_tuning)

为快速产生内存溢出，右击 Run As>Run Configurations, Arguments 标签 VM arguments 中填入

-Xmx32M -Xms32M

访问 [http://localhost:8080/heap](http://localhost:8080/heap)

```
Exception in thread "http-nio-8080-exec-2" Exception in thread "http-nio-8080-exec-1" java.lang.OutOfMemoryError: Java heap space
java.lang.OutOfMemoryError: Java heap space


```

-XX:MetaspaceSize=32M -XX:MaxMetaspaceSize=32M（同时在 pom.xml 中加入 asm 的依赖）

访问 [http://localhost:8080/nonheap](http://localhost:8080/nonheap)

```
Exception in thread "main" java.lang.OutOfMemoryError: Metaspace
Exception in thread "ContainerBackgroundProcessor[StandardEngine[Tomcat]]" java.lang.OutOfMemoryError: Metaspace


```

内存溢出自动导出

-XX:+HeapDumpOnOutOfMemoryError

-XX:HeapDumpPath=./

右击 Run As>Run Configurations, Arguments 标签 VM arguments 中填入

-Xmx32M -Xms32M -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./

可以看到自动在当前目录中生成了一个 java_pid660.hprof 文件

```
java.lang.OutOfMemoryError: GC overhead limit exceeded
Dumping heap to ./java_pid660.hprof ...


```

另一种导出溢出也更推荐的方式是 jmap

option: -heap, -clstats, -dump:<dump-options>, -F

jmap -dump:format=b,file=heap.hprof <pid>

![](http://upload-images.jianshu.io/upload_images/14565748-1463877fe6bacd86.jpg) jmap 导出溢出文件

**MAT 下载地址**：[http://www.eclipse.org/mat/](http://www.eclipse.org/mat/)

找开上述导出的内存溢出文件即可进行分析，如下图的溢出源头分析：

![](http://upload-images.jianshu.io/upload_images/14565748-8cceb7f2d01469e0.jpg) Memory Analyzer 内存溢出分析

### jstack

详情参考 [jstack 官方文档](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstack.html)

jstack <pid>

可查看其中包含 java.lang.Thread.State: WAITING (parking)，JAVA 线程包含的状态有：

NEW：线程尚未启动

RUNNABLE：线程正在 JVM 中执行

BLOCKED：线程在等待监控锁 (monitor lock)

WAITING：线程在等待另一个线程进行特定操作（时间不确定）

TIMED_WAITING：线程等待另一个线程进行限时操作

TERMINATED：线程已退出

![](http://upload-images.jianshu.io/upload_images/14565748-ee9f95627ee04455.png) image

[monitor_tuning](https://github.com/alanhou7/java-codes/tree/master/monitor_tuning/src/main/java/org/alanhou/monitor_tuning/chapter2) 中新增 CpuController.java

mvn clean package -Dmaven.test.skip

mvn 打包提速参考 [CSDN](https://blog.csdn.net/xiangxianghehe/article/details/73760819)

此时会生成一个 monitor_tuning-0.0.1-SNAPSHOT.jar 的 jar 包，为避免本地的 CPU 消耗过多导致死机，建议上传上传到虚拟机进行测试

nohup java -jar monitor_tuning-0.0.1-SNAPSHOT.jar &

访问 [http://xx.xx.xx.xx:12345/loop(](http://xx.xx.xx.xx:12345/loop()端口 12345 在 [application.properties](http://application.properties) 文件中定义)

top -p <pid> -H 可以查看线程及 CPU 消耗情况

![](http://upload-images.jianshu.io/upload_images/14565748-4f47ed131d875779.jpg) top 命令打出线程 CPU 消耗

使用 jstack <pid> 可以导出追踪文件，文件中 PID 在 jstack 中显示的对应 nid 为十六进制 (命令行可执行 print '%x' <pid > 可以进行转化，如 1640 对应的十六进制为 668)

```
"http-nio-12345-exec-3" #18 daemon prio=5 os_prio=0 tid=0x00007f10003fb000 nid=0x668 runnable [0x00007f0fcf8f9000]
   java.lang.Thread.State: RUNNABLE
    at org.alanhou.monitor_tuning.chapter2.CpuController.getPartneridsFromJson(CpuController.java:77)
...


```

访问 [http://xx.xx.xx.xx:12345/deadlock(](http://xx.xx.xx.xx:12345/deadlock()如上 jstack <pid> 导出追踪记录会发现如下这样的记录)

```
Java stack information for the threads listed above:
===================================================
"Thread-5":
    at org.alanhou.monitor_tuning.chapter2.CpuController.lambda$deadlock$1(CpuController.java:41)
    - waiting to lock <0x00000000edcf3470> (a java.lang.Object)
    - locked <0x00000000edcf3480> (a java.lang.Object)
    at org.alanhou.monitor_tuning.chapter2.CpuController$Lambda$337/547045985.run(Unknown Source)
    at java.lang.Thread.run(Thread.java:748)
"Thread-4":
    at org.alanhou.monitor_tuning.chapter2.CpuController.lambda$deadlock$0(CpuController.java:33)
    - waiting to lock <0x00000000edcf3480> (a java.lang.Object)
    - locked <0x00000000edcf3470> (a java.lang.Object)
    at org.alanhou.monitor_tuning.chapter2.CpuController$Lambda$336/1704575158.run(Unknown Source)
    at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.


```

### JVisualVM

详情参考[官方文档](http://visualvm.github.io/documentation.html)

Mac 命令行直接输入 jvisualvm 命令，Windows 找到对应的 exe 文件双击即可打开

插件安装 Tools>Plugins>Settings 根据自身版本 (java -version) 更新插件中心地址，各版本查询地址：

[http://visualvm.github.io/pluginscenters.html](http://visualvm.github.io/pluginscenters.html)

建议安装：Visual GC, BTrace Workbench

![](http://upload-images.jianshu.io/upload_images/14565748-a1b9a1c0cd5815aa.jpg) VisualVM

以上是本地的 JAVA 进程监控，还可以进行远程的监控，在上图左侧导航的 Applications 下的 Remote 处右击 Add Remote Host...，输入主机 IP 即可添加，在 IP 上右击会发现有两种连接 JAVA 进程进行监控的方式: JMX, jstatd

bin/catalina.sh(以 192.168.0.5 为例)

```
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=9004 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.net.preferIPv4Stack=true -Djava.rmi.server.host


```

启动 tomcat，以 JMX 为例，在 IP 上右击点击 Add JMX Connection...，输入 IP:PORT

![](http://upload-images.jianshu.io/upload_images/14565748-fc0b9cdc6c2f6a8b.jpg) Add JMX Connection

以上为 Tomcat，其它 JAVA 进程也是类似的，如：

```
nohup java -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=9005 -Dcom.sun.management.jmxremote.local.only=false -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.net.preferIPv4Stack=true -Djava.rmi.server.hostname=192.168.0.5 -jar monitor_tuning-0.0.1-SNAPSHOT.jar &


```

### BTrace

[BTrace](https://github.com/jbachorik/btrace/releases/latest) 可以动态地向目标应用程序的字节码注入追踪代码，使用的技术有 JavaCompilerApi, JVMTI, Agent, Instrumentation+ASM

使用方法：JVisualVM 中添加 BTrace 插件

方法二：btrace <pid> <trace_script>

[monitor_tuning](https://github.com/alanhou7/java-codes/tree/master/monitor_tuning/src/main/java/org/alanhou/monitor_tuning/chapter4) 中新增包 org.alanhou.monitor_tuning.chapter4

安装 BTrace 要记得配置环境变量，以 Mac 为例

```
# vi ~/.bash_profile
BTRACE_HOME=/Applications/btrace
PATH=$PATH:$BTRACE_HOME/bin
export PATH
# source ~/.bash_profile


```

pom.xml 中添加 btrace-agent, btrace-boot, btrace-client 的依赖

访问：[](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

[构造函数：@OnMethod(clazz="", method="<init> ")(PrintContructor.java)](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

```
[192:chapter4 alan$ btrace 3682 PrintConstructor.java 
org.alanhou.monitor_tuning.chapter2.User,<init>
[1, Java, ]](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=) 
```

[拦截同名函数：用参数区分（PrintSame.java）](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

[如下例中虽然方法名相同，但分别有一个和两个参数](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

```
[@RequestMapping("/same1")
public String same(@RequestParam("name")String name) {
    // 访问地址: http://localhost:12345/ch4/same1?name=Java
    return "Hello, "+name;
}

@RequestMapping("/same2")
public String same(@RequestParam("name")String name, @RequestParam("id")int id) {
    // 访问地址: http://localhost:12345/ch4/same2?name=Java&id=1
    return "Hello, "+name+", "+id;
}](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=) 
```

[拦截时机](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

[Kind.ENTRY: 入口，默认值（上述例子均为这种情况）](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

[Kind.RETURN: 返回（PrintReturn.java）](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

```
[192:chapter4 alan$ btrace 3981 PrintReturn.java 
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1,Hello, Java](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=) 
```

[Kind.THROW: 异常（PrintOnThrow.java）](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

```
[192:chapter4 alan$ btrace 4041 PrintOnThrow.java 
java.lang.ClassNotFoundException: org.apache.catalina.webresources.WarResourceSet
...
java.lang.ArithmeticException: / by zero
    org.alanhou.monitor_tuning.chapter4.Ch4Controller.exception(Ch4Controller.java:40)
...](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=) 
```

[Kind.Line: 行 (PrintLine.java)](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

```
[# 打印指定行号是否执行
192:chapter4 alan$ btrace 4149 PrintLine.java 
org.alanhou.monitor_tuning.chapter4.Ch4Controller,exception,39](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=) 
```

[**拦截 this、入参、返回值**](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

[this：@self](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

[入参：可以用 AnyType，也可以用真实类型，同名的用真实的](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

[返回：@Return](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

[获取对象的值](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

[简单类型：直接获取](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

[复杂类型：反射，类名 + 属性名（PrintArgComplex.java）](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

```
[192:chapter4 alan$ btrace -cp "/Users/alan/Desktop/demo/java-code/monitor_tuning/target/classes" 4337 PrintArgComplex.java 
{id=1, name=Java, }
Java
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg2](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=) 
```

[拦截函数中还可以使用正则表达式，如 method="/.*/" 匹配指定类下的所有方法 (PrintRegex.java)](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

[打印环境变量 (PrintJinfo.java)](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

### [Tomcat 性能监控与调优](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

[**Tomcat 远程 Debug**](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

[JDWP](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

[bin/startup.sh 修改最后一行 (添加 jpda)](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

```
[exec "$PRGDIR"/"$EXECUTABLE" jpda start "$@"](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=) 
```

[bin/catalina.sh 为便于远程调试进行如下修改](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

```
[JPDA_ADDRESS="localhost:8000"
# 修改为
JPDA_ADDRESS="54321"](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=) 
```

[若发现 54321 端口启动存在问题可尝试 bin/catalina.sh jpda start](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

[本地添加包 org.alanhou.monitor_tuning.chapter5，修改打包方式为 war，并重写 configure，进入 monitor_tuning 文件夹，执行 mvn clean package 进行打包，target 目录下默认生成的包名为 monitor_tuning-0.0.1-SNAPSHOT.war，为便于访问修改为 monitor_tuning.war 再上传到服务器的 webapps 目录下](http://localhost:12345/ch4/arg1?>http://localhost:12345/ch4/arg1?name=Java</a></p><p></p><pre># 示例输出
192:chapter4 alan$ btrace 2247 PrintArgSimple.java 
[Java, ]
org.alanhou.monitor_tuning.chapter4.Ch4Controller,arg1

</pre><p></p><p><strong>常见问题</strong>：Please set JAVA_HOME before running this script</p><p></p><pre># vi ~/.bash_profile
export JAVA_HOME=$(/usr/libexec/java_home)
# source ~/.bash_profile

</pre><p></p><p><strong>拦截方法</strong></p><p></p><p>普通方法：@OnMethod(clazz=)

[http://192.168.0.5:8080/monitor_tuning/ch5/hello](http://192.168.0.5:8080/monitor_tuning/ch5/hello)

使用 Eclipse 远程调试，右击 Debug As > Debug Configurations... > Remote Java Application > 右击 New 新建

**tomcat-manager 监控**

1.conf/tomcat-users.xml 添加用户

```
 <role role/>
  <role role/>
  <role role/>
  <user user/>


```

2.conf/Catalina/localhost/manager.xml 配置允许的远程连接

```
<?xml version="1.0" encoding="UTF-8"?>
<Context privileged="true" antiResourceLocking="false"
        docBase="$(catalina.home)/webapps/manager">
  <Valve class
        allow="127\.0\.0\.1" />
</Context>


```

远程连接将 allow="127.0.0.1" 修改为 allow="^.*$"，浏览器中输入 [http://127.0.0.1:8080/manage](http://127.0.0.1:8080/manage) 或对应的 IP，用户名密码为 tomcat-users.xml 中所设置的

3. 重启 Tomcat 服务

![](http://upload-images.jianshu.io/upload_images/14565748-77aed84d2acf0a59.jpg) Tomcat Manager

**psi-probe 监控**

下载地址：[https://github.com/psi-probe/psi-probe](https://github.com/psi-probe/psi-probe)，

下载后进入 psi-probe-master 目录，执行：

mvn clean package -Dmaven.test.skip

将 web/target/probe.war 放到 Tomcat 的 webapps 目录下，同样需要 conf/tomcat-users.xml 和 conf/Catalina/localhost/manager.xml 中的配置（可保持不变），启动 Tomcat 服务

浏览器中输入 [http://127.0.0.1:8080/probe](http://127.0.0.1:8080/probe) 或对应的 IP，用户名密码为 tomcat-users.xml 中所设置的

![](http://upload-images.jianshu.io/upload_images/14565748-6f99ff9cc214a845.jpg) PSI Probe 演示

**Tomcat 调优**

线程优化（webapps/docs/config/http.html）：

maxConnections

acceptCount

maxThreads

minSpareThreads

配置优化（webapps/docs/config/host.html）：

autoDeploy

enableLookups（http.html）

reloadable（context.html）

protocol="org.apache.coyote.http11.Http11AprProtocol"

Session 优化：

如果是 JSP, 可以禁用 Session

补充：APR 配置

```
yum install -y apr-devel openssl-devel
cd tomcat/bin
tar -zxvf tomcat-native.tar.gz
cd tomcat-native-1.2.17-src/native/
./configure --with-apr=/usr/bin/apr-1-config --with-java-home=/usr/lib/jvm/java-1.8.0 --with-ssl=yes
make && make install

# vi tomcat/bin/setenv.sh
export CATALINA_OPTS=”$CATALINA_OPTS -Djava.library.path=/usr/local/apr/lib”
#vi tomcat/conf/server.xml
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
# 修改为
<Connector port="8080" protocol="org.apache.coyote.http11.Http11AprProtocol"
               connectionTimeout="20000"
               redirectPort="8443" />

<Listener class />


```

### Nginx 性能监控与调优

**Nginx 安装**

添加 yum 源（/etc/yum.repos.d/nginx.repo）

```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basesearch/
gpgcheck=0
enabled=1


```

安装及常用命令

```
yum install -y nginx
systemctl status|start|stop|reload|restart nginx
nginx -s stop|reload|quit|reopen
cat default.conf | grep -v "#'
nginx -V
nginx -t


```

配置反向代理 setenforce 0

**ngx_http_stub_status 监控连接信息**

```
location = /nginx_status {
    stub_status on;
    access_log off;
    allow 127.0.0.1;
    deny all;
}


```

可通过 curl [http://127.0.0.1/nginx_status](http://127.0.0.1/nginx_status) 进行查看或注释掉 allow 和 deny 两行使用 IP 进行访问

**ngxtop 监控请求信息**

查看官方使用方法：[https://github.com/lebinh/ngxtop](https://github.com/lebinh/ngxtop)

```
# 安装 python-pip
yum install epel-release
yum install python-pip
# 安装 ngxtop
pip install ngxtop


```

使用示例

指定配置文件：ngxtop -c /etc/nginx/nginx.conf

查询状态是 200：ngxtop -c /etc/nginx/nginx.conf -i 'status == 200'

查询访问最多 ip：ngxtop -c /etc/nginx/nginx.conf -g remote_addr

![](http://upload-images.jianshu.io/upload_images/14565748-e8b6d399c81fca6e.jpg) ngxtop 查询访问最多 ip

**nginx-rrd 图形化监控**

nginx-rrd 依赖于前面的 ngx_http_stub_status

```
# 安装 php
yum install php php-gd php-soap php-mbstring php-xmlrpc php-dom php-fpm -y
# Ngnix融合 php-fpm(/etc/php-fpm.d/www.conf)
user = nginx
group = nginx
# 启动 php-fpm
systemctl start php-fpm
# 修改 Nginx 配置文件

location ~ \.php$ {
    root           /usr/share/nginx/html;
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root/$fastcgi_script_name;
    include        fastcgi_params;
}

# 添加 index.php(/usr/share/nginx/html)
<?php phpinfo(); ?>


```

访问 [http://your.ip.address/index.php](http://your.ip.address/index.php) 检测配置是否成功

```
# 安装 rddtool 相关依赖
yum install perl rrdtool perl-libwww-perl libwww-perl perl-rrdtool -y
# 安装 nginx-rdd
wget http://soft.vpser.net/status/nginx-rrd/nginx-rrd-0.1.4.tgz
tar zxvf nginx-rrd-0.1.4.tgz  
cd nginx-rrd-0.1.4  
cp usr/sbin/* /usr/sbin     # 复制主程序文件到 /usr/sbin 下  
cp etc/nginx-rrd.conf /etc  # 复制配置文件到 /etc 下  
cp html/index.php /usr/share/nginx/html/
# 修改配置(/etc/nginx-rrd.conf)
RRD_DIR="/usr/share/nginx/html/nginx-rrd";  
WWW_DIR="/usr/share/nginx/html";  
# 添加定时任务（crontab -e）
*  * * * * /bin/sh /usr/sbin/nginx-collect
*/1 * * * * /bin/sh /usr/sbin/nginx-graph
# 查看定时任务执行情况
tail -f /var/log/cron

# ab 压测（未安装 yum -y install httpd-tools）
ab -n 10000 -c 10 http://127.0.0.1/index.html


```

访问 [http://your.ip.address/index.php](http://your.ip.address/index.php) 即可得到如下这种图形化界面：

![](http://upload-images.jianshu.io/upload_images/14565748-14ba247920a67375.jpg) nginx-rrd 图形化监控

**Nginx 优化**

**增加工作线程数和并发连接数**

```
worker_processes  4; # 一般CPU 是几核就设置为几
events {
    worker_connections  1024; # 每个进程打开的最大连接数，包含了 Nginx 与客户端和 Nginx 与 upstream 之间的连接
    multi_accept on; # 可以一次建立多个连接
    use epoll;
}


```

**启用长连接**

```
upstream server_pool{
    server localhost:8080 weight=1 max_fails=2 fail_timeout=30s;
    server localhost:8081 weight=1 max_fails=2 fail_timeout=30s;
    keepalive 300; # 300个长连接
}
location / {
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_pass http://server_pool;
}


```

**启用缓存压缩**

```
gzip on;
gzip_http_version 1.1;
gzip_disable "MSIE [1-6]\.(?!.*SV1)";
gzip_proxied any;
gzip_types text/plain text/css application/javascript application/x-javascript application/json application/xml application/vnd.ms-fontobject application/x-font-ttf application/svg+xml application/x-icon;
gzip_vary on;
gzip_static on;


```

**操作系统优化**

```
# 配置文件/etc/sysctl.conf
sysctl -w net.ipv4.tcp_syncookies=1 # 防止一个套接字在有过多试图连接到时引起过载
sysctl -w net.core.somaxconn=1024 # 默认128，连接队列
sysctl -w net.ipv4.tcp_fin_timeout=10 # timewait 的超时时间
sysctl -w net.ipv4.tcp_tw_reuse=1 # os 直接使用 timewait的连接
sysctl -w net.ipv4.tcp_tw_recycle=0 # 回收禁用

# /etc/security/limits.conf
*               hard    nofile            204800
*               soft    nofile             204800
*               soft    core             unlimited
*               soft    stack             204800


```

其它优化

```
sendfile    on; # 减少文件在应用和内核之间拷贝
tcp_nopush  on; # 当数据包达到一定大小再发送
tcp_nodelay off; # 有数据随时发送


```

### JVM 层 GC 调优

[https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/toc.html](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/toc.html)

**JVM 的内存结构**

![](http://upload-images.jianshu.io/upload_images/14565748-ab8fcf1e20097783.png) JVM 的内存结构

运行时数据区：

程序计数器 PC Register

虚拟机栈 JVM Stacks

堆 Heap

方法区 Method Area

```
 常量池 Run-Time Constant Pool


```

本地方法栈 Native Method Stacks

常用参数：

-Xms -Xmx

-XX:NewSize -XX:MaxNewSize

-XX:NewRatio -XX:SurvivorRatio

-XX:MetaspaceSize -XX:MaxMetaspaceSize

-XX:+UseCompressedClassPointers

-XX:CompressedClassSpaceSize

-XX:InitialCodeCacheSize

-XX:ReservedCodeCacheSize

**垃圾回收算法**

枚举根节点，做可达性分析

根节点：类加载器、Thread、虚拟机栈的本地变量表、static 成员、常量引用、本地方法栈的变量

算法：标记清除、复制、标记整理、分带垃圾回收

对象分配：对象优先在 Eden 区分配、大对象直接进入 Old 区 (-XX:PretenureSizeThreshold)、长期存活对象进入 Old 区 (-XX:MaxTenuringThreshold, -XX:+PrintTenuringDistribution, -XX:TargetSurvivorRatio

**垃圾收集器**

```
# 常见配置示例(bin/catalina.sh)
PARALLEL_OPTION="-XX:+UseParallelGC -XX:+UseParallelOldGC -XX:MaxGCPauseMillis=200 -XX:GCTimeRatio=99"
CMS_OPTION="-XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:CMSFullGCsBeforeCompaction=5"
G1_OPTION="-XX:+UseG1GC -XX:+UseStringDeduplication -XX:StringDeduplicationAgeThreshold=3 -XX:+UseCompressedClassPointers -XX:MaxGCPauseMillis=200"

JAVA_OPTS="$JAVA_OPTS $CMS_OPTION -Xms128M -Xmx128M -XX:MetaspaceSize=128M -XX:MaxMetaspaceSize=128M -XX:+UseCompressedClassPointers"


```

**串行收集器** Serial:：Serial, Serial Old (-XX:+UseSerialGC -XX:+UseSerialOldGC)

**并行收集器** Parallel（吞吐量优先, Server 模式默认收集器）：Parallel Scavenge, Parallel Old (-XX:+UseParallelGC, -XX:+UseParallelOldGC)

-XX:ParallelGCThreads=<N> 多少个 GC 线程（CPU> 8 N=5/8; CPU<8 N=CPU）

Parallel Collector Ergonomics:

-XX:MaxGCPauseMillis=<N>

-XX:GCTimeRatio=<N>

-Xmx<N>

动态内存调整

-XX:YoungGenerationSizeIncrement=<Y>

-XX:TenuredGenerationSizeIncrement=<T>

-XX:AdaptiveSizeDecrementScaleFactor=<D>

**并发收集器** Concurent（停顿时间优先）：CMS (-XX:+UseConcMarkSweepGC -XX:+UseParNewGC), G1(-XX:UseG1GC)

CMS

1. CMS initial mark: 初始标记 Root, STW

2. CMS concurrent mark：并发标记

3. CMS-concurrent-preclean：并发预清理

4. CMS remark：重新标记，STW

5. CMS concurrent sweep：并发清除

6. CMS-concurrent-reset：并发重置

缺点：CPU 敏感、产生垃圾和空间碎片

相关参数：

-XX:ConcGCThreads：并发的 GC 线程数

-XX:+UseCMSCompactAtFullCollection：FullGC 之后做压缩

-XX:CMSFullGCsBeforeCompaction：多少次 FullGC 之后压缩一次

-XX:CMSInitiatingOccupancyFraction：触发 FullGC

-XX:+UseCMSInitiatingOccupancyOnly：是否动态可调

-XX:+CMSScavengeBeforeRemark：FullGC 之前先做 YGC

-XX:+CMSClassUnloadingEnable：启用回收 Perm 区

iCMS

适用于单核或者双核

G1 Collector(JDK 8 开始，推荐使用）

G1 的几个概念

Region

SATB：Snapshot-At-The-Beginning，它是通过 Root Tracing 得到的，GC 开始时候存活对象的快照。

RSet：记录了其他 Region 中的对象引用本 Region 中对象的关系，属于 points-into 结构

YoungGC

新对象进入 Eden 区

存活对象拷贝到 Survivor 区

存活时间达到年龄阈值时，对象晋升到 Old 区

MixedGC

不是 FullGC，回收所有的 Young 和所有的 Old

global concurrent marking

1. Initial marking phase: 标记 GC Root, STW

2. Root region scanning phase：标记存活 Region

3. Concurrent marking phase：标记存活的对象

4. Remark phase：重新标记，STW

5. Cleanup phase：部分 STW

MixedGC 时机

InitiatingHeapOccupancyPercent

G1HeapWastePercent

G1MixedGCLiveThresholdPercent

G1MixedGCCountTarget

G1OldGCSetRegionThresholdPercent

-XX:+UseG1GC 开启 G1

-XX:G1HeapRegionSize=n， Region 的大小，1-32M，最多 2048 个

-XX:MaxGCPauseMillis=200 最大停顿时间

-XX:G1NewSizePercent、-XX:G1MaxNewSizePercent

-XX:G1ReservePercent=10 保留防止 to space 溢出

-XX:ParallelGCThreads=n SWT 线程数

-XX:ConcGCThreads=n 并发线程数 = 1/4 * 并行

最佳实践

年轻代大小：避免使用 - Xmn, -XX:NewRatio 等显式 Young 区大小，会覆盖暂停时间目标

暂停时间目标：暂停时间不要太严苛，其吞吐量目标是 90% 的应用程序时间和 10% 的垃圾回收时间，太严苛会直接影响到吞吐量

需要切换到 G1 的情况：

1. 50% 以上的堆被存活对象占用

2. 对象分配和晋升的速度变化非常大

3. 垃圾回收时间特别长，超过了 1 秒

查看方法：jinfo -flag xxx <pid>

并行：多条垃圾收集线程并行工作，但用户线程处于等待状态

并发：用户线程与垃圾收集线程同时执行（或交替执行）

停顿时间：垃圾收集器做垃圾回收中断应用执行的时间 -XX:MaxGCPauseMillis

吞吐量：花在垃圾收集的时间和花在应用时间的占比 -XX:GCTimeRatio=<n>，垃圾收集时间占 1/(1+n)

垃圾收集器搭配：

![](http://upload-images.jianshu.io/upload_images/14565748-bb32452ad149da5f.png) 垃圾收集器搭配

如何选择垃圾收集器？

1. 优先调整堆的大小让服务器自己来选择

2. 如果内存小于 100M，使用串行收集器

3。 如果是单核，并且没有停顿时间的要求，选择串行或者 JVM 自己选

4. 如果允许停顿时间超过 1 秒，选择并行或 JVM 自己选

5. 如果响应时间最重要，并且不能超过 1 秒，使用并发收集器

**可视化 GC 日志分析工具**

打印日志相关参数：

```
-XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -Xloggc:$CATALINA_HOME/logs/gc.log -XX:+PrintHeapAtGC -XX:+PrintTenuringDistribution


```

例（默认为 ParallelGC, 其它的添加 - XX:+UseConcMarkSweepGC 或 - XX:+UseG1GC 即可）：

```
JAVA_OPTS="$JAVA_OPTS -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -Xloggc:$CATALINA_HOME/logs/gc.log"


```

[CMS 日志格式](https://blogs.oracle.com/poonam/understanding-cms-gc-logs)  
[G1 日志格式](https://blogs.oracle.com/poonam/understanding-g1-gc-logs)

在线工具：[http://gceasy.io/](http://gceasy.io/)

访问 GCeasy 官网导入日志即可获取可视化分析及优先建议

![](http://upload-images.jianshu.io/upload_images/14565748-f9554870107c5dca.jpg) GCeasy 日志

[GCViewer](https://github.com/chewiebug/GCViewer)

mvn clean package -Dmaven.test.skip 生成 jar 包，双击执行，导入日志即可进入图形化分析页面

![](http://upload-images.jianshu.io/upload_images/14565748-ceb95a2dd42850fc.jpg) GCViewer

**Tomcat 的 GC 调优实战**

ParallelGC 调优

设置 Metaspace 大小 -XX:MetaspaceSize=64M -XX:MaxMetaspaceSize=64M

添加吞吐量和停顿时间参数 -XX:GCTimeRatio=99 -XX:MaxGCPauseMillis=100

修改动态扩容增量 -XX:YoungGenerationSizeIncrement=30

[G1 GC 最佳实践](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/g1_gc_tuning.html#recommendations)

`-XX:InitiatingHeapOccupancyPercent`: Use to change the marking threshold.

`-XX:G1MixedGCLiveThresholdPercent` and `-XX:G1HeapWastePercent`: Use to change the mixed garbage collection decisions.

`-XX:G1MixedGCCountTarget` and `-XX:G1OldCSetRegionThresholdPercent`: Use to adjust the CSet for old regions.

### JAVA 代码层调优

最终代码：[monitor_tuning](https://github.com/alanhou7/java-codes/tree/master/monitor_tuning/src/main/java/org/alanhou/monitor_tuning/chapter8)

** JVM 字节码指令与 javap**

javap <options> <classes>

cd monitor_tuning/target/classes/org/alanhou/monitor_tuning/chapter8/

javap -verbose Test1.class > Test1.txt 即可保存字节码文件

[常量池](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.4)

[字段描述符](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.3.2)

[方法描述符](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.3.3)

[字节码指令](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html)

**i++ 与 ++i，字符串拼接 + 原理**

javap -verbose SelfAdd.class > SelfAdd.txt

通过对 f1() 和 f2() 的字节码，我们得出结论 i++ 和 ++i 的执行效果完全相同

```
  public static void f1();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=0
         0: iconst_0
         1: istore_0
         2: goto          15
         5: getstatic     #25                 // Field java/lang/System.out:Ljava/io/PrintStream;
         8: iload_0
         9: invokevirtual #31                 // Method java/io/PrintStream.println:(I)V
        12: iinc          0, 1
        15: iload_0
        16: bipush        10
        18: if_icmplt     5
        21: return


```

**其他代码优先方法**

字符串拼接 +

javap -verbose StringAdd.class >StringAdd.txt

通过字节码可以看出 + 拼接符效率要低于 append

Try-Finally

javap -verbose TryFinally.class >TryFinally.txt

Constant variable(final)

javap -verbose StringConstant.class >StringConstant.txt

常用代码优化方法

1. 尽量重用对象，不要循环创建对象，比如: for 循环字符串拼接 (不在 for 中使用 + 拼接，先 new 一个 StringBuilder 再在 for 里 append)

2. 容器类初始化的地时候指定长度

List<String> collection = new ArrayLIst<String>(5);

Map<String, String> map = new HashMap<String, String>(32);

3. ArrayList（底层数组）随机遍历快，LinkedList（底层双向链表）添加删除快

4. 集合遍历尽量减少重复计算

5. 使用 Entry 遍历 Map

6. 大数组复制使用 System.arraycopy

7. 尽量使用基本类型而不是包装类型

8. 不要手动调用 System.gc()

9. 及时消除过期对象的引用，防止内存泄漏

10. 尽量使用局部变量，减小变量的作用域

11. 尽量使用非同步的容器 ArraryList vs. Vector

12. 尽量减小同步作用范围, synchronized 方法 vs. 代码块

13. 用 ThreadLocal 缓存线程不安全的对象，SimpleDateFormat

14. 尽量使用延迟加载

15. 尽量减少使用反射，必须用加缓存

16. 尽量使用连接池、线程池、对象池、缓存

17. 及时释放资源， I/O 流、Socket、数据库连接

18. 慎用异常，不要用抛异常来表示正常的业务逻辑

19. String 操作尽量少用正则表达式

20. 日志输出注意使用不同的级别

21. 日志中参数拼接使用占位符

log.info("orderId:" + orderId); 不推荐

log.info("orderId:{}", orderId); 推荐

原文链接：[Alan Hou 的个人博客](http://alanhou.org/java-optimization/)