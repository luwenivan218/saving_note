> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.liaoxuefeng.com](https://www.liaoxuefeng.com/article/1280526512029729)

> 分布式唯一 ID 生成器

在应用程序中，经常需要全局唯一的 ID 作为数据库主键。如何生成全局唯一 ID？

首先，需要确定全局唯一 ID 是整型还是字符串？如果是字符串，那么现有的 UUID 就完全满足需求，不需要额外的工作。缺点是字符串作为 ID 占用空间大，索引效率比整型低。

如果采用整型作为 ID，那么首先排除掉 32 位 int 类型，因为范围太小，必须使用 64 位 long 型。

采用整型作为 ID 时，如何生成自增、全局唯一且不重复的 ID？

方案一：利用数据库的自增 ID，从 1 开始，基本可以做到连续递增。Oracle 可以用`SEQUENCE`，MySQL 可以用主键的`AUTO_INCREMENT`，虽然不能保证全局唯一，但每个表唯一，也基本满足需求。

数据库自增 ID 的缺点是数据在插入前，无法获得 ID。数据在插入后，获取的 ID 虽然是唯一的，但一定要等到事务提交后，ID 才算是有效的。有些双向引用的数据，不得不插入后再做一次更新，比较麻烦。

第二种方式是采用一个集中式 ID 生成器，它可以是 Redis，也可以是 ZooKeeper，也可以利用数据库的表记录最后分配的 ID。

这种方式最大的缺点是复杂性太高，需要严重依赖第三方服务，而且代码配置繁琐。一般来说，越是复杂的方案，越不可靠，并且测试越痛苦。

第三种方式是类似 Twitter 的 Snowflake 算法，它给每台机器分配一个唯一标识，然后通过时间戳 + 标识 + 自增实现全局唯一 ID。这种方式好处在于 ID 生成算法完全是一个无状态机，无网络调用，高效可靠。缺点是如果唯一标识有重复，会造成 ID 冲突。

Snowflake 算法采用 41bit 毫秒时间戳，加上 10bit 机器 ID，加上 12bit 序列号，理论上最多支持 1024 台机器每秒生成 4096000 个序列号，对于 Twitter 的规模来说够用了。

但是对于绝大部分普通应用程序来说，根本不需要每秒超过 400 万的 ID，机器数量也达不到 1024 台，所以，我们可以改进一下，使用更短的 ID 生成方式：

53bitID 由 32bit 秒级时间戳 + 16bit 自增 + 5bit 机器标识组成，累积 32 台机器，每秒可以生成 6.5 万个序列号，核心代码：

```
private static synchronized long nextId(long epochSecond) {
    if (epochSecond < lastEpoch) {
        
        logger.warn("clock is back: " + epochSecond + " from previous:" + lastEpoch);
        epochSecond = lastEpoch;
    }
    if (lastEpoch != epochSecond) {
        lastEpoch = epochSecond;
        reset();
    }
    offset++;
    long next = offset & MAX_NEXT;
    if (next == 0) {
        logger.warn("maximum id reached in 1 second in epoch: " + epochSecond);
        return nextId(epochSecond + 1);
    }
    return generateId(epochSecond, next, SHARD_ID);
}


```

时间戳减去一个固定值，此方案最高可支持到 2106 年。

如果每秒 6.5 万个序列号不够怎么办？没关系，可以继续递增时间戳，向前 “借” 下一秒的 6.5 万个序列号。

同时还解决了时间回拨的问题。

机器标识采用简单的主机名方案，只要主机名符合`host-1`，`host-2`就可以自动提取机器标识，无需配置。

最后，为什么采用最多 53 位整型，而不是 64 位整型？这是因为考虑到大部分应用程序是 Web 应用，如果要和 JavaScript 打交道，由于 JavaScript 支持的最大整型就是 53 位，超过这个位数，JavaScript 将丢失精度。因此，使用 53 位整数可以直接由 JavaScript 读取，而超过 53 位时，就必须转换成字符串才能保证 JavaScript 处理正确，这会给 API 接口带来额外的复杂度。这也是为什么新浪微博的 API 接口会同时返回`id`和`idstr`的原因。

参考源码：[IdUtil.java](https://github.com/michaelliao/itranswarp/blob/master/src/main/java/com/itranswarp/util/IdUtil.java)