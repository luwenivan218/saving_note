> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/3459343637)

> 本章，我们来看下 Elasticsearch 中的读写优化。ES 中几乎所有的操作都是读操作或写操作，所以我们需要掌握一些对读写操作优化的方法。一、数据写入优化 1.1bulk 批量写入

本章，我们来看下 Elasticsearch 中的读写优化。ES 中几乎所有的操作都是读操作或写操作，所以我们需要掌握一些对读写操作优化的方法。

一、数据写入优化
--------

### 1.1 bulk 批量写入

如果我们要往 Elasticsearch 中批量写入数据的话，尽量采用 bulk 方式，每次批量写个几百条的样子。

bulk 批量写入的性能比一条一条写入大量的 document 的性能要好很多。但是，如果要知道一个 bulk 请求最佳的大小，需要对单个 ES node 的单个 shard 做压测。比如，先往 bulk 写入 100 个 document，然后 200 个，500 个，以此类推，每次都将 bulk size 加倍一次。如果 bulk 写入性能开始变平缓，那么这个就是最佳的 bulk 大小。

### 1.2 多线程写入

单线程发送 bulk 请求是无法最大化 Elasticsearch 集群写入吞吐量的。如果要利用集群的所有资源，就需要使用多线程并发的将数据 bulk 写入集群中。多线程并发写入，可以减少每次磁盘 fsync 的次数和开销。

_**线程的数量如何确定？**_

对单个 ES node 的单个 shard 做压测，比如说，先是 2 个线程，然后是 4 个线程，然后是 8 个线程……，每次线程数量倍增。一旦发现 ES 返回了`TOO_MANY_REQUESTS`的错误，那么就说明 ES 集群已经达到了并发写入的最大瓶颈，此时我们就知道最多只能支撑这么高的并发写入了。

### 1.3 增加 refresh 间隔

默认的 refresh 间隔是 1s，用`index.refresh_interval`参数可以设置，这样会其强迫 Elasticsearch 每隔 1s 就将创建一个新的 segment file。如果我们可以接受写入的数据较长时间后才查询到，比如 30s，就可以调大这个参数，此时就可以获取更大的写入吞吐量，因为 30s 内都是写内存的，每隔 30s 才会创建一个 segment file。

### 1.4 禁止 refresh 和 replia

如果我们要一次性加载大批量的数据进 Elasticsearch，可以先禁止 refresh 和 replia 复制，将`index.refresh_interval`设置为 - 1，将 ·index.number_of_replicas· 设置为 0。此时就没有 refresh 和 replica 机制了，写入的速度会非常快，一旦写完之后，我可以将 refresh 和 replica 修改回正常的状态。

### 1.5 禁止 swapping

之前讲解过，可以将 swapping 内存页交换禁止掉，因为 swapping 会导致大量磁盘 IO，性能很差。

### 1.6 增加 filesystem cache 大小

filesystem cache 被用来执行更多的 IO 操作，如果我们能给 filesystem cache 更多的内存资源，那么 Elasticsearch 的写入性能会好很多。

### 1.7 使用自动生成的 id

如果我们要手动给 es document 设置一个 id，那么 es 需要每次都去确认一下那个 id 是否存在，这个过程是比较耗费时间的。如果我们使用自动生成的 id，那么 es 就可以跳过这个步骤，写入性能会更好。对于你的业务表中的 id，可以作为 es document 的一个 field。

### 1.8 提升硬件

我们可以给 filesystem cache 更多的内存，也可以使用 SSD 替代机械硬盘，避免使用 NAS 等网络存储，考虑使用 RAID 0 来提升磁盘并行读写效率等等。

### 1.9 index buffer

如果我们的写入并发量非常高，那么最好将 index buffer 调大一些，可以通过`indices.memory.index_buffer_size`参数设置。Elasticsearch 会将这个设置作为每个 shard 共享的 index buffer，对于每个 shard 来说，最多给 512MB。默认这个参数的值是 10%，也就是 jvm heap 的 10%，如果我们给 jvm heap 分配 10gb 内存，那么这个 index buffer 就有 1gb，对于两个 shard 共享来说是足够的了。

二、数据搜索优化
--------

### 2.1 增加 filesystem cache 大小

Elasticsearch 的搜索引擎严重依赖于底层的 filesystem cache，你如果给 filesystem cache 更多的内存，尽量让内存可以容纳所有的 indx segment file 索引数据文件，那么你搜索的时候就基本都是走内存的，性能会非常高。

举个例子，如果有 3 个 ES 节点（3 台机器），每台机器 64G 内存，每台机器给 es jvm heap 是 32G，剩下的留给 filesystem cache，那么集群给 filesystem cache 的总内存就是 96G。

如果你的索引数据文件有 1T，那么由于 filesystem cache 的内存才 100g，所以只有十分之一的数据可以放入内存，其它的都在磁盘，此时搜索操作大部分都是走磁盘，性能肯定差。

归根结底，要让 Elasticsearch 性能要好，你的机器的总内存至少要容纳 ES 总数据量的一半。

### 2.2 提升硬件

*   给 filesystem cache 更多的内存资源；
*   用 SSD 固态硬盘；
*   使用本地存储系统，不要用 NFS 等网络存储系统；
*   给更多的 CPU 资源。

### 2.3 document 模型设计

document 模型设计是非常重要的，一般有两个思路：

1.  在写入数据的时候，就设计好模型，把处理好的数据放到一个新字段，避免 ES 执行复杂的搜索；
2.  自己用 Java/Python 之类的程序封装，Elasticsearch 仅做普通数据搜索，在 Java/Python 程序里去做复杂的业务逻辑处理。

### 2.3 预热 filesystem cache

如果我们重启了 Elasticsearch，那么 filesystem cache 是空的，就需要不断的查询才能重新让 filesystem cache 热起来，所以我们可以先做数据预热。

比如说，本来一个查询，要用户点击以后才执行，才能从磁盘加载到 filesystem cache 里，第一次执行要 10s，以后每次就几百毫秒。那么完全可以，后台先将数据加载到 filesystem cahce，以后用户真的访问时消耗时间就大大缩短。

### 2.6 避免使用 script 脚本

生产环境中，一般是避免使用 Elasticsearch script，性能不高。

### 2.7 使用固定范围的日期查询

尽量不要使用 now 这种内置函数来执行日期查询，因为默认 now 是到毫秒级的，是无法缓存结果，尽量使用一个阶段范围，比如分钟级，那么如果一分钟内都执行这个查询，是可以取用查询缓存的：

```
    PUT index/1
    {
      "my_date": "2016-05-11T16:30:55.328Z"
    }
    
    GET index/_search
    {
      "query": {
        "constant_score": {
          "filter": {
            "range": {
              "my_date": {
                "gte": "now-1h",
                "lte": "now"
              }
            }
          }
        }
      }
    }


```

上面的请求完全可以替换成下面这样，性能会更高：

```
    GET index/_search
    {
      "query": {
        "constant_score": {
          "filter": {
            "range": {
              "my_date": {
                "gte": "now-1h/m",
                "lte": "now/m"
              }
            }
          }
        }
      }
    }


```

三、磁盘读写优化
--------

磁盘读写优化，主要是优化磁盘空间的占用，减少磁盘空间的占用，让更多的数据可以进入 filesystem cache。比如原来磁盘空间占用 1T，内存只有 512G，现在优化了磁盘空间占用后，减少了数据量，可能数据量就只有 512G 了，那么就可以全部进入内存。

### 3.1 禁用不用的功能

禁用不用的功能是指，如果我们不需要使用 Elasticsearch 的以下功能，完全可以禁用掉以提升性能：

*   聚合：doc values
*   搜索：倒排索引，index
*   评分：norms
*   近似匹配：index_options（freqs）

上述任何一个功能不需要，就把对应的存储数据给清理掉，这样可以节约磁盘空间的占用，也可以优化磁盘的读写性能。

#### 禁用倒排索引

默认情况下，Elasticsearch 在写入 document 到索引的时候，都会给大多数的 field 增加一份`doc values`，就是正排索引，用来进行聚合或者排序。比如说，如果我们有一个叫做`foo`的数字类型 field，我们要对这个字段运行 histograms aggr 聚合操作，但是可能我们并不需要对这个字段进行搜索，那么就可以禁止为这个字段生成倒排索引，只需要 doc value 正排索引即可。禁用倒排索引的方式如下：

```
    PUT index
    {
      "mappings": {
          "properties": {
            "foo": {
              "type": "integer",
              "index": false
            }
          }
      }
    }


```

#### 禁用评分

text 类型的 field 会存储 norm 值，用来计算 doc 的相关度分数，如果我们需要对一个 text field 进行搜索，但是不关心这个 field 的分数，那么可以禁用 norm 值：

```
    PUT index
    {
      "mappings": {
          "properties": {
            "foo": {
              "type": "text",
              "norms": false
            }
          }
        }
    }


```

#### 禁用近似匹配

text field 还会存储出现频率以及位置，出现频率也是用来计算相关度分数的，位置是用来进行 phrase query 这种近似匹配操作的，如果我们不需要执行 phrase query 近似匹配，那么可以禁用这个属性：

```
    PUT index
    {
      "mappings": {
          "properties": {
            "foo": {
              "type": "text",
              "index_options": "freqs"
            }
          }
        }
    }


```

### 3.2 禁用动态 string 类型映射

默认的动态 string 类型映射，会将 string 类型的 field 同时映射为 text 类型以及 keyword 类型，这会浪费磁盘空间。如果我们不是两种类型都需要，可以通过手动设置 mappings 映射来避免字符串类型的 field 被自动映射为 text 和 keyword：

```
    PUT index
    {
      "mappings": {
          "dynamic_templates": [
            {
              "strings": {
                "match_mapping_type": "string",
                "mapping": {
                  "type": "keyword"
                }
              }
            }
          ]
      }
    }


```

### 3.3 禁止_all field

`_all field`会将 document 中所有 field 的值都合并在一起进行索引，很耗费空间，如果不需要一次性对所有的 field 都进行搜索，那么最好禁用`_all field`。

### 3.4 使用 best_compression

`_source field`和其他 field 都很耗费磁盘空间，最好是对其使用`best_compression`进行压缩。用 elasticsearch.yml 中的 index.codec 来设置，将其设置为 best_compression 即可。

### 3.5 合理使用数字类型

Elasticsearch 支持 4 种数字类型：`byte`、`short`、`integer`、`long`。如果最小的类型就合适，那么就用最小的类型，以减少 doc 大小。

四、总结
----

本章，我介绍了一些 Elasticsearch 中的常用读写优化建议。ES 中几乎所有的操作都是读操作或写操作，所以我们需要掌握这些对读写操作优化的方法。