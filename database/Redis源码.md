# 一、课前导读

### 1.1 阅读源码的好处



- 更有章法、高效地解决问题
- 学习源码阅读方法，培养源码习惯，掌握学习主动权
- 学习良好的编程规范和技巧，写出高质量的代码
- 学习计算机系统设计思想，实现职业能力进阶



### 1.2 如何正确学习源码

- 先从整体上掌握源码的结构
- 是一定要有目标牵引和原理支撑，比如看源码是为了解决什么，或者要先懂得原理
- 先处理主线逻辑，再分支细节





### 1.3 目录结构

- deps目录，包含了Redis依赖的第三方代码库	
- src目录，包含了 Redis 所有功能模块的代码文件，也是 Redis 源码的重要组成部分
- tests 目录，单元测试（对应 unit 子目录），Redis Cluster 功能测试（对应 cluster 子目录）、哨兵功能测试（对应 sentinel 子目录）、主从复制功能测试（对应 integration 子目录）
- utils目录，辅助性功能，包括用于创建 Redis Cluster 的脚本、用于测试 LRU 算法效果的程序，以及可视化 rehash 过程的程序

![img](..\docs\redis源码全景图.jpg)





### 1.4 Redis功能模块与源码对应

- 数据类型：
  \- String（t_string.c、sds.c、bitops.c）
  \- List（t_list.c、ziplist.c、quicklist.c）
  \- Hash（t_hash.c、ziplist.c、dict.c）
  \- Set（t_set.c、intset.c）
  \- Sorted Set（t_zset.c、ziplist.c、dict.c）
  \- HyperLogLog（hyperloglog.c）
  \- Geo（geo.c、geohash.c、geohash_helper.c）
  \- Stream（t_stream.c、rax.c、listpack.c）

- 全局：
  \- Server（server.c、anet.c）
  \- Object（object.c）
  \- 键值对（db.c）
  \- 事件驱动（ae.c、ae_epoll.c、ae_kqueue.c、ae_evport.c、ae_select.c、networking.c）
  \- 内存回收（expire.c、lazyfree.c）
  \- 数据替换（evict.c）
  \- 后台线程（bio.c）
  \- 事务（multi.c）
  \- PubSub（pubsub.c）
  \- 内存分配（zmalloc.c）
  \- 双向链表（adlist.c）

- 高可用&集群：
  \- 持久化：RDB（rdb.c、redis-check-rdb.c)、AOF（aof.c、redis-check-aof.c）
  \- 主从复制（replication.c）
  \- 哨兵（sentinel.c）
  \- 集群（cluster.c）

- 辅助功能：
  \- 延迟统计（latency.c）
  \- 慢日志（slowlog.c）
  \- 通知（notify.c）
  \- 基准性能（redis-benchmark.c）





# 二、数据结构



## 2.1 SDS （Simple Dynamic String）

本质：`typedef char *sds;`

特点：1. 支持丰富且高效的字符串操作  2. 能保存任意的二进制  3. 能尽量节省内存开销（类比数据库设计）



### 2.1.1  为什么不直接复用C的字符串

c的字符串是通过char*数组来实现的

- **不能存储任意数据**：C字符数组的结尾位置使用'\0'来表示，保存的数据假如有'\0'，则会有截断的风险
- **操作函数的复杂度高**：很多操作都需要遍历源字符串才能得到结果，比如说字符串追加函数strcat()，和strlen()



### 2.1.2  SDS结构设计

Redis 使用 typedef 给 char* 类型定义了一个别名，这个别名就是 sds

`typedef char *sds`



SDS包含了四个元数据：

- 字符数组 buf[]，用来保存实际数据
- 字符数组现有长度 len
- 分配给字符数组的空间长度 alloc
- SDS 类型 flags



### 2.1.3 SDS的优势

-  操作效率高：获取长度无需遍历，O(1)复杂度
- 二进制安全：因单独记录长度字段，所以可存储包含 \0 的数据
- 兼容 C 字符串函数，可直接使用字符串 API
- 紧凑型内存设计
- 申请、释放的优化
  - 内存预分配，**避免频繁申请**：SDS 扩容，会多申请一些内存（小于 1MB 翻倍扩容，大于 1MB 按 1MB 扩容）
  - 多余内存不释放，**换取追加速度**：SDS 缩容，不释放多余的内存，下次使用可直接复用这些内存 







### 2.1.3 从SDS中学到的思想

- 空间换时间。记录已占用和被分配的空间，减少遍历次数，从而带来更高效的函数操作
- 函数封装。将空间检查和扩容封装在sdsMakeRoomFor函数中，这个函数干的事情是，确保sds有足够的空间
- 节省内存
  - 通过不同的结构头，来灵活保存不同大小的字符串。sdsdhr8，len和alloc各占一个字节，能表示的字符数组长度为256
  - ` __attribute__ ((__packed__))`采用紧凑的方式分配内存，如果按字节对齐的方式，不足8字节的变量，编译器也会分配8个字节



### 2.1.4 SDS设计精妙的点

#### 2.1.4.1 通过原始的char*数组，定位到SDS的Header

使用__attribute__ ((__packed__))除了节省内存空间之外，还有一个很精妙的设计就是在packed之后可以通过以下的方式来获取flags字段

```c
struct __attribute__ ((__packed__)) sdshdrX {
    uintX_t len; /* used */
    uintX_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};

#define SDS_TYPE_MASK 7
#define SDS_HDR(T,s) ((struct sdshdr##T *)((s)-(sizeof(struct sdshdr##T))))

static inline size_t sdslen(const sds s) {
    unsigned char flags = s[-1]; 
    switch(flags&SDS_TYPE_MASK) {
        case SDS_TYPE_5:
            return SDS_TYPE_5_LEN(flags);
        case SDS_TYPE_8:
            return SDS_HDR(8,s)->len;
        case SDS_TYPE_16:
            return SDS_HDR(16,s)->len;
        case SDS_TYPE_32:
            return SDS_HDR(32,s)->len;
        case SDS_TYPE_64:
            return SDS_HDR(64,s)->len;
    }
    return 0;
}
```

从而更进一步的得到struct的具体类型，如果是非1字节对齐的话，这里就不能这样操作。而sds中通过原始的char* 定位到sds的Header是设计的**灵魂**



#### 2.1.4.2  SDS对外可以像char*一样使用

在sds外面可以像使用char*一样来使用sds，但是使用sds相关函数操作的时候又可以发挥sds的特性(通过偏移量来找到sds的header)





### 2.1.5 疑问

紧凑型内存和字节对齐的优缺点





## 2.2 hash表



### 2.2.1 Hash表要解决的核心问题

- 哈希冲突 - Redis链式哈希
- rehash的开销 - 渐进式rehash



其实都来自于 Hash 表要保存的数据量，超过了当前 Hash 表能容纳的数据量



### 2.2.2  Redis如何实现链式Hash

```c
typedef struct dictht {
    dictEntry **table; // 二维数组
    unsigned long size; // 数组大小
    unsigned long sizemask;
    unsigned long used;
} dictht;
```



链式哈希，每个dictEntry都有个指向下一个dictEntry的指针

```c
typedef struct dictEntry {
    void *key;
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;
} dictEntry;
```



### 2.2.3 设计精妙的点

#### 2.2.3.1 节省内存

dictEntry中的联合体v，因为包含的是无符号的 64 位整数、有符号的 64 位整数，以及 double 类的值，它们本身就是占64位，就可以不用指针指向了，而是可以直接存在键值对的结构体中，这样就避免了再用一个指针，从而节省了内存空间





### 2.2.4 如何实现rehash

#### 2.2.4.1 何时触发rehash

判断都在`_dictExpandIfNeeded`函数中

```c
static int _dictExpandIfNeeded(dict *d)
{
    // 1. hash表正在处于扩容状态
    if (dictIsRehashing(d)) return DICT_OK;

    // 2.  ht[0]的大小为0 （初始化工作）
    // DICT_HT_INITIAL_SIZE 为4
    if (d->ht[0].size == 0) return dictExpand(d, DICT_HT_INITIAL_SIZE);

    // 3. ht[0]承载的元素超过了ht[0]的大小，并且 （可以进行扩容 或者 ht[0]承载的元素是大小的 dict_force_resize_ratio 倍
    // 负载因子是否大于等于1 或 大于等于 dict_force_resize_ratio
    // dict_can_resize为真的条件为 当前没有 RDB 子进程，并且也没有 AOF 子进程
    // 为了避免父进程大量写时复制，如果负载因子超过了 5（哈希冲突已非常严重），依旧会强制做 rehash（重点）
    if (d->ht[0].used >= d->ht[0].size &&
        (dict_can_resize ||
         d->ht[0].used/d->ht[0].size > dict_force_resize_ratio))
    {
        return dictExpand(d, d->ht[0].used*2);  // 扩容两倍
    }
    return DICT_OK;
}
```



首先，通过在dict.c文件中查看 _dictExpandIfNeeded 的被调用关系，我们可以发现，_dictExpandIfNeeded 是被 _dictKeyIndex 函数调用的，而 _dictKeyIndex 函数又会被 dictAddRaw 函数调用，然后 dictAddRaw 会被以下三个函数调用。

- dictAdd：用来往 Hash 表中添加一个键值对。
- dictRelace：用来往 Hash 表中添加一个键值对，或者键值对存在时，修改键值对。
- dictAddorFind：直接调用 dictAddRaw。



#### 2.2.4.2 rehash扩容多少

如果当前表的已用空间大小为 size，那么就将表扩容到 size*2 的大小

如果扩容大小超过最大值，则返回最大值加1



### 2.2.5 渐进式 rehash 如何实现

#### 2.2.5.1 为什么要实现渐进式rehash

当hash表进行rehash的时候，由于表空间的扩大，很多键可能会移动到新的位置上。当发生这样的键拷贝时，主线程是会阻塞的，这些会使主线程无法执行其他操作



#### 2.2.5.2 dictRehash函数

这个函数实际执行键拷贝，它的输入参数有两个，分别是全局哈希表（即前面提到的 dict 结构体，包含了 ht[0]和 ht[1]）和需要进行键拷贝的桶数量（bucket 数量）

```c
int dictRehash(dict *d, int n) {
    int empty_visits = n*10; /* Max number of empty buckets to visit. */
    if (!dictIsRehashing(d)) return 0;

    //主循环，根据要拷贝的bucket数量n，循环n次后停止或ht[0]中的数据迁移完停止
    while(n-- && d->ht[0].used != 0) {
        dictEntry *de, *nextde;

        /* Note that rehashidx can't overflow as we are sure there are more
         * elements because ht[0].used != 0 */
        assert(d->ht[0].size > (unsigned long)d->rehashidx);
        // 如果当前要迁移的bucket中没有元素
        while(d->ht[0].table[d->rehashidx] == NULL) {
            d->rehashidx++;
            if (--empty_visits == 0) return 1;
        }
        de = d->ht[0].table[d->rehashidx];
        
        // 有数据可以迁移，依次把哈希项取出来
        while(de) {
            uint64_t h;

            nextde = de->next;
            // 根据ht[1]的表大小，和哈希函数算出来key的hash值，得到元素所在ht[1]的下表
            h = dictHashKey(d, de->key) & d->ht[1].sizemask;
            // 采用的是头插法
            // 原因，操作简单，不用遍历所有项即可插入bucket中，热点数据放到最前
            de->next = d->ht[1].table[h];
            d->ht[1].table[h] = de;
            d->ht[0].used--;
            d->ht[1].used++;
            // 指向下一个哈希项
            de = nextde;
        }
        // 如果当前bucket中已经没有哈希项了，将该bucket置为NU
        d->ht[0].table[d->rehashidx] = NULL;
        // 将rehash加1，下一次将迁移下一个bucket中的元素
        d->rehashidx++;
    }

   // 判断ht[0]的数据是否迁移完成
    if (d->ht[0].used == 0) {
        // 释放ht[0]的空间
        zfree(d->ht[0].table);
        // 让ht[0]指向ht[1]，以便接受正常的请求
        d->ht[0] = d->ht[1];
        // 重置ht[1]的大小为0
        _dictReset(&d->ht[1]);
        //设置全局哈希表的rehashidx标识为-1，表示rehash结束
        d->rehashidx = -1;
        //返回0，表示ht[0]中所有元素都迁移完
        return 0;
    }

    //返回1，表示ht[0]中仍然有元素没有迁移完
    return 1;
}
```



#### 2.2.5.3 _dictRehashStep函数

这个函数实现了每次只对一个 bucket 执行 rehash，调用的 _dictRehashStep 函数，给 dictRehash 传入的循环次数变量 n 的值都为 1



_dictRehashStep会有五个函数调用

- dictAddRaw
- dictGenericDelete
- dictFind
- dictGetRandomKey
- dictGetSomeKeys

这样一来，每次迁移完一个 bucket，Hash 表就会执行正常的增删查请求操作，这就是在代码层面实现渐进式 rehash 的方法



### 2.2.6 全局哈希表何时触发rehash

- 增删改查哈希表时：每次迁移 1 个哈希桶（文章提到的 dict.c 中的 _dictRehashStep 函数）
- 定时 rehash：如果 dict 一直没有操作，无法渐进式迁移数据，那主线程会默认每间隔 100ms 执行一次迁移操作。这里一次会以 100 个桶为基本单位迁移数据，并限制如果一次操作耗时超时 1ms 就结束本次任务，待下次再次触发迁移（文章没提到这个，详见 dict.c 的 dictRehashMilliseconds 函数）



（注意：定时 rehash 只会迁移全局哈希表中的数据，不会定时迁移 Hash/Set/Sorted Set 下的哈希表的数据，这些哈希表只会在操作数据时做实时的渐进式 rehash）





### 2.2.7 为什么dict不用红黑树

- hash冲突不使用红黑树：redis需要高性能，如果hash冲突使用红黑树，红黑树和链表的转换会引起不必要的开销（hash冲突不大的情况下红黑树其实比链表沉重，还会浪多余的空间）
- dict不采用红黑树：在负载因子较低，hash冲突较低的情况下，hash表的效率O(1)远远高于红黑树