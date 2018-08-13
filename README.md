## Redis 源码阅读
一个C小白，苍老着黄大大的书学习Redis源码！

### 001 [简单动态字符串](https://github.com/liub1993/redis/commit/c020cbb3e869d9e62581d82c8e8b189d55ff50d8): Simple dynamic string
> 2018-08-04  `src/sds.h src/sds.c`
##### 实现的数据结构（简要）
```c
struct sdshdr {
    uint16_t len; /* 使用了的长度 */
    uint16_t alloc; /* 除了malloc的header和字符串最后的空字符外的长度 */
    unsigned char flags; /* 定义sdshr的类型，3 lsb of type, 5 unused bits */
    char buf[]; /* 字节数组，用与保存字符串 */
};
```

##### SDC和C字符串的区别
1. 常数获取字符串长度
2. 杜绝缓冲区溢出
3. 减少字符串修改时带来的内存重分配次数
4. 二进制安全(通过 `len` 而不是 `\0` 判断字符结束)
5. 兼容部分C字符串函数


### 002 [链表的实现](https://github.com/liub1993/redis/commit/3474520c72af187f74c4e5bd566bb27a0a27e9b2): 
> 2018-08-05 `src/adlist.h src/adlist.c`
##### 实现的数据结构
```c
/* 链表节点数据结构 */
typedef struct listNode {
    struct listNode *prev;
    struct listNode *next;
    void *value;
} listNode;

/* 使用list结构体维护链表 */
typedef struct list {
    listNode *head; /* 节点表头 */
    listNode *tail; /* 节点表尾 */
    /* 在结构体中定义函数指针 */ 
    void *(*dup)(void *ptr); /* 节点复制函数，返回指针类型的数据 */
    void (*free)(void *ptr); /* 节点值释放函数 */
    int (*match)(void *ptr, void *key); /* 节点值比对函数 */
    unsigned long len; /* 链表所包含的节点个数 */
} list;

```

##### Redis链表特点
1. 双端
2. 无环，头尾指针指向NULL
3. 带有表的头和尾指针
4. 带链表计数器
5. 多态。使用 `void *` 指针来保存节点值 

Redis链表广泛用于Redis的各种功能，比如列表键、发布与订阅、慢查询、监视器等。


### 003 [字典]()
> 2018-08-05 `src/dict.c dict.h`
##### 实现的数据结构
```c
typedef struct dictEntry {          /* 保存hash表中的键值对 */
    void *key;                      /* 键 */
    union {                         /* 值 */
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;         /* 指向哈希表中哈希值相同的下一个节点，用于解决键冲突问题 */
} dictEntry;

typedef struct dictType {
    uint64_t (*hashFunction)(const void *key);
    void *(*keyDup)(void *privdata, const void *key);
    void *(*valDup)(void *privdata, const void *obj);
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);
    void (*keyDestructor)(void *privdata, void *key);
    void (*valDestructor)(void *privdata, void *obj);
} dictType;

/* This is our hash table structure. Every dictionary has two of this as we
 * implement incremental rehashing, for the old to the new table. */
typedef struct dictht {         /* 哈希表的结构 */
    dictEntry **table;          /* 哈希表数组，每个元素都是指向dictEntry的指针。起始元素个数为 DICT_HT_INITIAL_SIZE */
    unsigned long size;         /* 哈希表的大小 */
    unsigned long sizemask;     /* 哈希表大小掩码，用于计算索引值，总是等于size-1*/
    unsigned long used;         /* 该哈希表已有的节点（键值对）数目 */
} dictht;

typedef struct dict {           /* 字典的结构 */
    dictType *type;             /* 类型特定函数 */
    void *privdata;             /* 私有数据 */
    dictht ht[2];               /* 二元哈希表 */

    /* rehash时，将ht[0]中的键值对重新hash并迁移到ht[1]中。
     * 当ht[0]为空时，替换ht[1]为ht[0]，新建ht[1]
     * rehash索引进度，值为-1时表示没有在进行rehash
     * */
    long rehashidx;
    unsigned long iterators;    /* 当前运行的迭代器个数 */
} dict;
```

##### Redis字典重点
1. [字典的遍历dictScan原理](https://blog.csdn.net/gqtcgq/article/details/50533336)



> 【参考文献和资料】
> 1. https://github.com/antirez/redis
> 2. 黄健宏. Redis设计与实现[M]. 机械工业出版社, 2014.
