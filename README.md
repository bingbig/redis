## Redis 源码阅读

### 001 [简单动态字符串](https://github.com/liub1993/redis/commit/c020cbb3e869d9e62581d82c8e8b189d55ff50d8): Simple dynamic string
> 2018-08-04
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














> 【参考文献和资料】
> 1. https://github.com/antirez/redis
> 2. 黄健宏. Redis设计与实现[M]. 机械工业出版社, 2014.
