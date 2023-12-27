> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/3200738836)

> 本章，我们看下如何对部署 Elasticsearch 进程的机器进行调优。因为生产环境下，为了提升 Elasticsearch 性能，很多 OS 的默认配置是不能满足要求的，我们需要做些调

转

 2023-08-08  阅读 (214)  点赞 (0)

原文地址：https://www.tpvlog.com/article/170

本章，我们看下如何对部署 Elasticsearch 进程的机器进行调优。因为生产环境下，为了提升 Elasticsearch 性能，很多 OS 的默认配置是不能满足要求的，我们需要做些调整。

一、禁止 swapping
-------------

大多数操作系统都会使用尽量多的内存来进行 file system cache，并且尽量将不经常使用的 java 应用内存 swap 到磁盘中去。这会导致 jvm heap 的部分内存，甚至是用来执行代码的内存页被 swap 到磁盘中去。

swapping 对于非常影响应用性能，为了 Elasticsearch 节点的稳定性考虑，应该尽量避免这种 swapping。因为 swapping 会导致 GC 过程从毫秒级变成分钟级，在 GC 的时候需要将内存从磁盘 swapping 到内存里，特别耗时，这会导致 es 节点响应请求变得很慢，甚至导致 ES node 跟 cluster 失联。

有三种方法可以 disable swapping。推荐彻底禁用 swap，如果做不到的话，也得尽量最小化 swappiness 的影响，比如通过 lock memory 的方法。

### 1.1 禁用 swapping

通常来说，Elasticsearch 进程会在一个节点上单独运行，那么 es 进程的内存使用是由 jvm option 控制的。所以可以使用下面的命令临时禁止 swapping：

```
    swapoff -a


```

要永久性的禁止 swap，需要修改`/etc/fstab`文件，将所有包含 swap 的行都注释掉。

### 1.2 配置 swappiness

另外一个方法就是通过`sysctl`，将`vm.swappiness`设置为 1，这可以尽量减少 Linux 内核 swap 的倾向，在正常情况下，就不会进行 swap，但是在紧急情况下，还是会进行 swap 操作：

```
    sysctl -w vm.swappiness=1


```

### 1.3 启用 bootstrap.memory_lock

最后一个选项，就是用`mlockall`，将 es jvm 进程的 address space 锁定在内存中，阻止 es 内存被 swap out 到磁盘上去。在`config/elasticsearch.yml`中，可以配置：

```
    bootstrap.memory_lock: true


```

我们通过这下面的请求检查 mlockall 是否开启了：

```
    GET _nodes?filter_path=**.mlockall，


```

如果发现 mlockall 是 false，那么意味着 mlockall 请求失败了，会看到一行日志，unable to lock jvm memory。最可能的原因，就是在 linux 系统中，启动 es 进程的用户没有权限去 lock memory，需要通过以下方式进行授权：

```
    ulimit -l unlimited


```

另外一个原因可能是临时目录使用`noexec option`来`mount`了。可以通过指定一个新的临时目录来解决：

```
    export ES_JAVA_OPTS="$ES_JAVA_OPTS -Djava.io.tmpdir=/path/to/temp/dir"


```

二、使用虚拟内存
--------

Elasticsearch 使用`hybrid mmapfs / niofs`目录来存储 index 数据，操作系统的默认 mmap count 限制是很低的，可能会导致内存耗尽的异常。所以需要提升 mmap count 的限制：

```
    sysctl -w vm.max_map_count=262144


```

如果要永久性的设置这个值，要修改`/etc/sysctl.conf`，将`vm.max_map_count`的值修改一下，重启过后，用`sysctl vm.max_map_count`命令来验证一下数值是否修改成功。

Elasticsearch 同时会用 NioFS 和 MMapFS 来处理不同的文件，我们需要有足够的虚拟内存来给 mmapped 文件使用，可以用 sysctl 来设置：`sysctl -w vm.max_map_count=262144`。还可以在`/etc/sysctl.conf`中，通过`vm.max_map_count`来设置。

三、设置线程数
-------

Elasticsearch 用了很多线程池来应对不同类型的操作，在需要的时候创建新的线程是很重要的。要确保 Elasticsearch 用户能创建的最大线程数量至少在 2048 以上。

可以通过`ulimit -u 2048`来临时设置，也可以在`/etc/security/limits.conf`中设置 nproc 为 2048 来永久性设置。

四、总结
----

本章，我介绍了针对操作系统的一些核心参数的调优。需要特别关注的是 swapping 机制，swapping 对于性能影响是非常严重的，为了 ES 节点的稳定性考虑，应该尽量避免 swapping。