> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1128498847)

> 我们前面章节对分词的讲解全是基于英文文本的。本章，我们就来看看如何对中文短语进行分词。Elasticsearch 中，最常用的中文分词器就是 IK。一、IK 分词器 1.1 安装首先，从

转

 2023-08-08  阅读 (134)  点赞 (0)

原文地址：https://www.tpvlog.com/article/146

我们前面章节对分词的讲解全是基于英文文本的。本章，我们就来看看如何对中文短语进行分词。Elasticsearch 中，最常用的中文分词器就是 IK。

一、IK 分词器
--------

### 1.1 安装

首先，从 GitHub 上下载预编译好的 IK 包，比如，我的 Elasticsearch 版本是 v7.6.0，我就下载 7.6.0 版本的 IK：[https://github.com/medcl/elasticsearch-analysis-ik/releases。](https://github.com/medcl/elasticsearch-analysis-ik/releases%E3%80%82)

IK 和 Elasticsearch 主要的版本对照如下表：

<table><thead><tr><th>IKversion</th><th>ESversion</th></tr></thead><tbody><tr><td>master</td><td>7.x-&gt;master</td></tr><tr><td>6.x</td><td>6.x</td></tr><tr><td>5.x</td><td>5.x</td></tr></tbody></table>

然后解压缩放置到`YOUR_ES_ROOT/plugins/ik/`目录下，最后，重启 Elasticsearch 即可。

### 1.2 基本使用

IK 分词器有两种 analyzer： **ik_max_word** 、 **ik_smart** ，但是一般是选用 **ik_max_word** 。

*   ik_max_word：会将文本做最细粒度的拆分，比如会将 “中华人民共和国国歌” 拆分为 “中华人民共和国, 中华人民, 中华, 华人, 人民共和国, 人民, 人, 民, 共和国, 共和, 和, 国国, 国歌” 等等，会穷尽各种可能的组合。
*   ik_smart：只做最粗粒度的拆分，比如会将 “中华人民共和国国歌” 拆分为“中华人民共和国, 国歌”。

我们可以看下用 IK 分词器的分词效果，先将改变指定字段的 mapping：

```
    PUT /my_index 
    {
      "mappings": {
          "properties": {
            "text": {
              "type": "text",
              "analyzer": "ik_max_word"
            }
          }
      }
    }


```

然后看下分词效果：

```
    GET /my_index/_analyze
    {
      "text": "美专家称疫情在美国还未达到顶峰",
      "analyzer": "ik_max_word"
    }


```

### 1.3 配置文件

IK 的配置文件存在于`YOUR_ES_ROOT/plugins/ik/config`目录下，我们可以看下这个目录下的各个文件的作用：

*   **main.dic：** IK 原生内置的中文词库，总共有 27 万多条，只要是这些单词，都会被分在一起；
*   **quantifier.dic：** 放了一些单位相关的词；
*   **suffix.dic：** 放了一些后缀；
*   **surname.dic：** 中国的姓氏；
*   **stopword.dic：** 英文停用词。

如果我们希望自定义词库，比如加入一些当下的流行词，就可以修改`IKAnalyzer.cfg.xml`的`ext_dict`，配置我们扩展的词库，然后重启 ES 就可以生效了。

二、热更新词库
-------

上一节中，如果我们希望自定义词库，每次都必须修改配置文件然后重启 Elasticsearch，这种做法只适合测试环境。如果在生产环境，我们希望热更新词库，比如基于 MySQL 中的热点数据来更新词库，那该怎么做呢？

目前有两种方案，业界一般采用第一种：

1.  修改 IK 分词器源码，然后每隔一定时间，自动从 MySQL 中加载新的词库；
2.  基于 IK 分词器原生支持的热更新方案：部署一个 web 服务器，提供一个 http 接口，通过 modified 和 tag 两个 http 响应头，来提供词语的热更新。

修改 IK 的源码，网上有很多现有示例，我这边就不再赘述了。

三、总结
----

本章，我介绍了 IK 中文分词器的安装及基本使用，生产环境中，我们一般会修改 IK 的源码，使它支持热更新词库。