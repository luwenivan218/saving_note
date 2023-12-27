> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1159890811)

> 当我们使用聚合分析，对某个的字段进行分析时，如果该字段是可分词的类型，比如 text 这种，就会抛出异常：```"caused_by":{"type":"illegal_argum

转

 2023-08-08  阅读 (133)  点赞 (0)

原文地址：https://www.tpvlog.com/article/146

当我们使用聚合分析，对某个的字段进行分析时，如果该字段是可分词的类型，比如 text 这种，就会抛出异常：

```
    "caused_by": {
          "type": "illegal_argument_exception",
          "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [test_field] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory."
    }


```

上面报错大概的意思是说：必须要打开`fielddata`，然后将正排索引数据加载到内存中，才可以对分词 field 执行聚合操作，并且很消耗内存。

所以，我们需要先改变字段的 mapping，将`fielddata`置为 true：

```
    POST /test_index/_mapping 
    {
      "properties": {
        "test_field": {
          "type": "text",
          "fielddata": true
        }
      }
    }


```

一、fielddata 原理
--------------

当开启`fielddata: true`后，Elasticsearch 会在执行聚合操作时，实时地将 field 对应的数据建立一份 fielddata 正排索引，索引会被加载到 JVM 内存中，然后基于内存中的索引执行分词 field 的聚合操作。

所以，如果 doc 数量非常多，这个过程会非常消耗内存，因为分词的 field 需要按照 term 进行聚合，其中涉及很多复杂的算法和操作，Elasticsearch 为了提升性能，对于这些操作全部是基于 JVM 内存进行的。

### 1.1 懒加载

fielddata 是通过懒加载的方式加载到内存中的，所以只有对一个 analzyed field 执行聚合操作时，才会执行加载。这种懒加载的方式更加降低了性能。

### 1.2 内存限制

Elasticsearch 可以通过配置`indices.fielddata.cache.size`参数来限制 fielddata 对内存的使用。超出限制，就会清除内存已有的 fielddata 数据，但是一旦限制内存使用，又会导致频繁的 evict 和 reload，产生大量内存碎片，同时降低 IO 性能。

### 1.3 circuit breaker

如果一次 query 操作加载的 feilddata 数据量大小超过了总内存，就会发生内存溢出，circuit breaker 会估算 query 要加载的 fielddata 大小，如果超出总内存就短路，query 直接失败。可以通过以下参数进行设置：

```
    indices.breaker.fielddata.limit：fielddata的内存限制，默认60%
    indices.breaker.request.limit：执行聚合的内存限制，默认40%
    indices.breaker.total.limit：综合上面两个，限制在70%以内


```

二、fielddata 优化
--------------

一般来讲，不会对 text 类型的字段执行聚合操作，也不建议做这个操作。如果确实有这个需求，需要对 fielddata 做些优化，以提升性能。

### 2.1 fielddata 预加载

第一个优化点是_预加载_ （与默认的 延迟加载相对）。随着新 segment 的创建（通过刷新、写入或合并等方式）， 启动字段预加载可以使那些对搜索不可见的分段里的 fielddata _提前_ 加载。这就意味着首次命中分段的查询不需要促发 fielddata 的加载，因为 fielddata 已经被载入到内存，避免了用户遇到搜索卡顿的情形。

预加载是按字段启用的，所以我们可以控制具体哪个字段可以预先加载：

```
    POST /test_index/_mapping
    {
      "properties": {
        "test_field": {
          "type": "string",
          "fielddata": {
            "loading" : "eager" 
          }
        }
      }
    }


```

> 预加载只是简单的将载入 fielddata 的代价转移到索引刷新的时候，而不是查询时，从而大大提高了搜索体验。

### 2.2 全局序号

另一种用来降低 fielddata 内存使用的技术叫做 全局序号（Global Ordinals）。

设想我们有十亿文档，每个文档都有自己的 `status` 状态字段，状态总共有三种： `status_pending` 、 `status_published` 、 `status_deleted` 。如果我们为每个文档都保留其状态的完整字符串形式，那么每个文档就需要使用 14 到 16 字节，或总共 15 GB。

取而代之的是，我们可以指定三个不同的字符串对其排序、编号：0，1，2。

```
    Ordinal | Term
    
    0       | status_deleted
    1       | status_pending
    2       | status_published


```

序号字符串在序号列表中只存储一次，每个文档只要使用数值编号的序号来替代它原始的值。

```
    Doc     | Ordinal
    -------------------------
    0       | 1  
    1       | 1  
    2       | 2  
    3       | 0  


```

这样可以将内存使用从 15 GB 降到 1 GB 以下！

全局序号是一个构建在 fielddata 之上的数据结构，它只占用少量内存。唯一值是 _跨所有分段_ 识别的，然后将它们存入一个序号列表中，`terms` 聚合可以对全局序号进行聚合操作，将序号转换成真实字符串值的过程只会在聚合结束时发生一次。这会将聚合（和排序）的性能提高三到四倍。

三、总结
----

本章，我介绍了 Elasticsearch 中的`fielddata`的作用和优化方式。一般来讲，我们最好不要对 string、text 等可分词类型的字段进行聚合操作，因为即使进行了优化，性能开销也会非常大。