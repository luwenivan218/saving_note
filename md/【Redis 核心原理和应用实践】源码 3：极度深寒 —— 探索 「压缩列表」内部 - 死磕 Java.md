> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.skjava.com](https://www.skjava.com/series/article/1808980061)

> Redis 为了节约内存空间使用，zset 和 hash 容器对象在元素个数较少的时候，采用压缩列表 (ziplist) 进行存储。压缩列表是一块连续的内存空间，元素之间紧挨着存储，没有任

转

 2023-03-26  阅读 (167)  点赞 (0)

原文地址：https://blog.csdn.net/shenchaohao12321/article/details/88209381

Redis 为了节约内存空间使用，zset 和 hash 容器对象在元素个数较少的时候，采用压缩列表 (ziplist) 进行存储。压缩列表是一块连续的内存空间，元素之间紧挨着存储，没有任何冗余空隙。

```
    > zadd programmings 1.0 go 2.0 python 3.0 java 
    (integer) 3 
    > debug object programmings 
    Value at:0x7fec2de00020 refcount:1 encoding:ziplist serializedlength:36 lru:6022374 lru_seconds_idle:6 
    > hmset books go fast python slow java fast 
    OK 
    > debug object books 
    Value at:0x7fec2de000c0 refcount:1 encoding:ziplist serializedlength:48 lru:6022478 lru_seconds_idle:1


```

这里，注意观察 debug object 输出的 encoding 字段都是 ziplist，这就表示内部采用压缩列表结构进行存储。

```
    struct ziplist<T> { 
        int32 zlbytes; 
        int32 zltail_offset; 
        int16 zllength; 
        T[] entries; 
        int8 zlend; 
    }


```

![](http://image.skjava.com/article/series/redis/202303261122563841.png)

压缩列表为了支持双向遍历，所以才会有 ztail_offset 这个字段，用来快速定位到最后一个元素，然后倒着遍历。  
entry 块随着容纳的元素类型不同，也会有不一样的结构。

```
    struct entry { 
        int<var> prevlen; 
        int<var> encoding; 
        optional byte[] content; 
    } 


```

它的 prevlen 字段表示前一个 entry 的字节长度，当压缩列表倒着遍历时，需要通过这个字段来快速定位到下一个元素的位置。它是一个变长的整数，当字符串长度小于 254(0xFE) 时，使用一个字节表示；如果达到或超出 254(0xFE) 那就使用 5 个字节来表示。第一个字节是 0xFE(254)，剩余四个字节表示字符串长度。你可能会觉得用 5 个字节来表示字符串长度，是不是太浪费了。我们可以算一下，当字符串长度比较长的时候，其实 5 个字节也只占用了不到 (5/(254+5))<2% 的空间。

![](http://image.skjava.com/article/series/redis/202303261122569692.png)

encoding 字段存储了元素内容的编码类型信息，ziplist 通过这个字段来决定后面的 content 内容的形式。  
Redis 为了节约存储空间，对 encoding 字段进行了相当复杂的设计。Redis 通过这个字段的前缀位来识别具体存储的数据形式。下面我们来看看 Redis 是如何根据 encoding 的前缀位来区分内容的：  
1、00xxxxxx 最大长度位 63 的短字符串，后面的 6 个位存储字符串的位数，剩余的字节就是字符串的内容。  
2、01xxxxxx xxxxxxxx 中等长度的字符串，后面 14 个位来表示字符串的长度，剩余的字节就是字符串的内容。  
3、10000000 aaaaaaaa bbbbbbbb cccccccc dddddddd 特大字符串，需要使用额外 4 个字节来表示长度。第一个字节前缀是 10，剩余 6 位没有使用，统一置为零。后面跟着字符串内容。不过这样的大字符串是没有机会使用的，压缩列表通常只是用来存储小数据的。  
4、11000000 表示 int16，后跟两个字节表示整数。  
5、11010000 表示 int32，后跟四个字节表示整数。  
6、11100000 表示 int64，后跟八个字节表示整数。  
7、11110000 表示 int24，后跟三个字节表示整数。  
8、11111110 表示 int8，后跟一个字节表示整数。  
9、11111111 表示 ziplist 的结束，也就是 zlend 的值 0xFF。  
10、1111xxxx 表示极小整数，xxxx 的范围只能是 (0001~1101), 也就是 1~13，因为 0000、1110、1111 都被占用了。读取到的 value 需要将 xxxx 减 1，也就是整数 0~12 就是最终的 value。  
注意到 content 字段在结构体中定义为 optional 类型，表示这个字段是可选的，对于很小的整数而言，它的内容已经内联到 encoding 字段的尾部了。

因为 ziplist 都是紧凑存储，没有冗余空间 (对比一下 Redis 的字符串结构)。意味着插入一个新的元素就需要调用 realloc 扩展内存。取决于内存分配器算法和当前的 ziplist 内存大小，realloc 可能会重新分配新的内存空间，并将之前的内容一次性拷贝到新的地址，也可能在原有的地址上进行扩展，这时就不需要进行旧内容的内存拷贝。  
如果 ziplist 占据内存太大，重新分配内存和拷贝内存就会有很大的消耗。所以 ziplist 不适合存储大型字符串，存储的元素也不宜过多。

```
     
    unsigned char *__ziplistCascadeUpdate(unsigned char *zl, unsigned char *p) { 
        size_t curlen = intrev32ifbe(ZIPLIST_BYTES(zl)), rawlen, rawlensize; 
        size_t offset, noffset, extra; 
        unsigned char *np; 
        zlentry cur, next; 
     
        while (p[0] != ZIP_END) { 
            zipEntry(p, &cur); 
            rawlen = cur.headersize + cur.len; 
            rawlensize = zipStorePrevEntryLength(NULL,rawlen); 
             
            if (p[rawlen] == ZIP_END) break; 
            zipEntry(p+rawlen, &next); 
     
             
            
            if (next.prevrawlen == rawlen) break; 
     
            if (next.prevrawlensize < rawlensize) { 
                 
                
                offset = p-zl; 
                extra = rawlensize-next.prevrawlensize; 
                
                zl = ziplistResize(zl,curlen+extra); 
                p = zl+offset; 
     
                 
                np = p+rawlen; 
                noffset = np-zl; 
     
                 
                
                if ((zl+intrev32ifbe(ZIPLIST_TAIL_OFFSET(zl))) != np) { 
                    ZIPLIST_TAIL_OFFSET(zl) = 
                        intrev32ifbe(intrev32ifbe(ZIPLIST_TAIL_OFFSET(zl))+extra); 
                } 
     
                 
                
                memmove(np+rawlensize, 
                    np+next.prevrawlensize, 
                    curlen-noffset-next.prevrawlensize-1); 
                zipStorePrevEntryLength(np,rawlen); 
     
                 
                p += rawlen; 
                curlen += extra; 
            } else { 
                if (next.prevrawlensize > rawlensize) { 
                     
                    
                    
                    zipStorePrevEntryLengthLarge(p+rawlen,rawlen); 
                } else { 
                    
                    zipStorePrevEntryLength(p+rawlen,rawlen); 
                } 
     
                 
                break; 
            } 
        } 
        return zl; 
    } 


```

前面提到每个 entry 都会有一个 prevlen 字段存储前一个 entry 的长度。如果内容小于 254 字节，prevlen 用 1 字节存储，否则就是 5 字节。这意味着如果某个 entry 经过了修改操作从 253 字节变成了 254 字节，那么它的下一个 entry 的 prevlen 字段就要更新，从 1 个字节扩展到 5 个字节；如果这个 entry 的长度本来也是 253 字节，那么后面 entry 的 prevlen 字段还得继续更新。  
如果 ziplist 里面每个 entry 恰好都存储了 253 字节的内容，那么第一个 entry 内容的修改就会导致后续所有 entry 的级联更新，这就是一个比较耗费计算资源的操作。  
删除中间的某个节点也可能会导致级联更新，读者可以思考一下为什么？

当 set 集合容纳的元素都是整数并且元素个数较小时，Redis 会使用 intset 来存储结合元素。intset 是紧凑的数组结构，同时支持 16 位、32 位和 64 位整数。

```
    struct intset<T> { 
        int32 encoding; 
        int32 length; 
        int<T> contents; 
    }


```

![](http://image.skjava.com/article/series/redis/202303261122576183.png)

老钱也不是很能理解为什么 intset 的 encoding 字段和 length 字段使用 32 位整数存储。毕竟它只是用来存储小整数的，长度不应该很长，而且 encoding 只有 16 位、32 位和 64 位三个类型，用一个字节存储就绰绰有余。关于这点，读者们可以进一步讨论。

```
    > sadd codehole 1 2 3 
    (integer) 3 
    > debug object codehole 
    Value at:0x7fec2dc2bde0 refcount:1 encoding:intset serializedlength:15 lru:6065795 lru_seconds_idle:4 
    > sadd codehole go java python 
    (integer) 3 
    > debug object codehole 
    Value at:0x7fec2dc2bde0 refcount:1 encoding:hashtable serializedlength:22 lru:6065810 lru_seconds_idle:5


```

注意观察 debug object 的输出字段 encoding 的值，可以发现当 set 里面放进去了非整数值时，存储形式立即从 intset 转变成了 hash 结构。

1、为什么 set 集合在数量很小的时候不使用 ziplist 来存储？  
2、执行 rpush codehole 1 2 3 命令后，请写出列表内容的 16 进制形式。