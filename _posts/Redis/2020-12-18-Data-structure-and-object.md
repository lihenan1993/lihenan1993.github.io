---
title: 数据结构与对象
category: Redis
typora-root-url: ../../
---

# String - SDS

redis里面，C字符串只会作为字符串字面量用在一些无须对字符串值进行修改的地方，比如打印日志



## 什么是SDS

redis中String键和值用的是 简单动态字符串(simple dynamic string, SDS)

![image-20201218214106744](/assets/img/image-20201218214106744.png)

```c
struct sdshdr {
    // 记录buf数组中已使用字节的数量
    // 等于SDS所保存字符串的长度
    int len;
    // 未使用字节的数量
    int free;
    // 字节数组，用于保存字符串
    char buf[];
}
```



## C 字符串和 SDS 之间的区别

| C 字符串                                             | SDS                                                  |
| :--------------------------------------------------- | :--------------------------------------------------- |
| 获取字符串长度的复杂度为 O(N) 。                     | 获取字符串长度的复杂度为 O(1) 。                     |
| API 是不安全的，可能会造成缓冲区溢出。               | API 是安全的，不会造成缓冲区溢出。                   |
| 修改字符串长度 `N` 次必然需要执行 `N` 次内存重分配。 | 修改字符串长度 `N` 次最多需要执行 `N` 次内存重分配。 |
| 只能保存文本数据。                                   | 可以保存文本或者二进制数据。                         |
| 可以使用所有 `<string.h>` 库中的函数。               | 可以使用一部分 `<string.h>` 库中的函数。             |



## 优点

1. 扩容时候，不会造成缓冲区溢出
2. 减少修改字符串时候内存重分配次数
   1. 空间预分配， 修改后len小于1M，则预分配的free等于len ，最多为1M
   2. 惰性空间释放，缩短时候修改len和free，不是立即释放
3. 二进制安全，因为用的len ,而不是C中'\0'作为结束标志
4. 兼容部分C的函数，因为保留了结尾的'\0' 



# List - 链表

当list元素数量多，或者元素是比较长的字符串，redis使用双向链表作为list的底层实现

> 注意：3.0之后 不再用ziplist 和 双向链表作为list的底层实现，而是quicklist

```c
typedef struct listNode {
	struct listNode *prev;
    struct listNode *next;
    void *value;
} listNode;

typedef struct list {
    listNode *head;
    listNode *tial;
    unsigned long len;
    // 节点值复制函数
    void *(*dup)(void *ptr);
    // 节点值释放函数
    void (*free)(void *ptr);
    // 节点值对比函数
    int (*match)(void *ptr, void *key);
} list;
```

![digraph {图 3-2    由 list 结构和 listNode 结构组成的链表}](/assets/img/graphviz-5f4d8b6177061ac52d0ae05ef357fceb52e9cb90.png)



### Redis 的链表实现的特性可以总结如下：

- 双端： 链表节点带有 `prev` 和 `next` 指针， 获取某个节点的前置节点和后置节点的复杂度都是 O(1) 。
- 无环： 表头节点的 `prev` 指针和表尾节点的 `next` 指针都指向 `NULL` ， 对链表的访问以 `NULL` 为终点。
- 带表头指针和表尾指针： 通过 `list` 结构的 `head` 指针和 `tail` 指针， 程序获取链表的表头节点和表尾节点的复杂度为 O(1) 。
- 带链表长度计数器： 程序使用 `list` 结构的 `len` 属性来对 `list` 持有的链表节点进行计数， 程序获取链表中节点数量的复杂度为 O(1) 。
- 多态： 链表节点使用 `void*` 指针来保存节点值， 并且可以通过 `list` 结构的 `dup` 、 `free` 、 `match` 三个属性为节点值设置类型特定函数， 所以链表可以用于保存各种不同类型的值。

## quicklist

quicklist就是一个双向链表，不过quicklist的每个节点都是一个ziplist

```
// 控制每个节点存储ziplist数量或者大小
list-max-ziplist-size

// 两端保留几个节点不进行压缩，0默认值，不压缩中间节点
list-compress-depth 0
```





# Hash - 哈希表

> - 什么是哈希表
>
>   把关键码值（key-value）通过散列函数，映射到表中一个位置来访问记录，以加快查找的速度。
>
> - 解决冲突的方法 
>
>   链表法和散列法（开放寻址和再散列）
>
> - 适用
>
>   “数量有限”但“不同索引数量极大”的一些数据，必需极高的访问效率同时又不想无端消耗太多的存储空间
>
> - 注意
>
>   查找时间与数据可散列的程度有关，最坏O(N)，最好O(1)

```c
// 字典
typedef struct dict {
    // 类型特定函数
    dictType *type;
    // 私有数据
    void *privdata;
    // 哈希表
    dictht ht[2];
    // rehash 索引
    // 当 rehash 不在进行时，值为 -1
    int rehashidx; /* rehashing not in progress if rehashidx == -1 */
} dict;

// 哈希表
typedef struct dictht {
    // 哈希表数组
    dictEntry **table;
    // 哈希表大小
    unsigned long size;
    // 哈希表大小掩码，用于计算索引值
    // 总是等于 size - 1
    unsigned long sizemask;
    // 该哈希表已有节点的数量
    unsigned long used;
} dictht;

// 哈希表节点，链表法
typedef struct dictEntry {
    // 键
    void *key;
    // 值
    union {
        // 对象指针
        void *val;
        // 无符号64位整数
        uint64_t u64;
        // 有符号64位整数
        int64_t s64;
    } v;
    // 指向下个哈希表节点，形成链表
    struct dictEntry *next;
} dictEntry;
```

![image-20201219133119437](/assets/img/image-20201219133119437.png)

- 散列算法

  murmurHash，优点即使输入的key有规律，算法扔能给出一个很好的随机分布性，并且算法的计算速度也非常快

- 解决键冲突

  链表法，哈希表节点中的next存有下一个哈希表节点的指针

- rehash

  1. 负载因子

     ```
     # 负载因子 = 哈希表已保存节点数量 / 哈希表大小
     load_factor = ht[0].used / ht[0].size
     ```

     当以下条件中的任意一个被满足时， 程序会自动开始对哈希表执行扩展操作：

     1. 服务器目前没有在执行 BGSAVE 命令或者 BGREWRITEAOF 命令， 并且哈希表的负载因子大于等于 `1` ；
     2. 服务器目前正在执行 BGSAVE 命令或者 BGREWRITEAOF 命令， 并且哈希表的负载因子大于等于 `5` ；
     3. 当哈希表的负载因子小于 `0.1` 时， 程序自动开始对哈希表执行收缩操作。

  2. 如果size过大，扩容是渐进式的，将h[0]刷到h[1]，再调换。期间key直接加到h[1]

  3. 扩容和缩容的大小

     1. ht[1].size 为第一个 >= ht[0].used * 2 的  2^n

        例如： ht[0].used = 10

        10 * 2 = 20

        2^4 < 20 < 2^5=32

        h[1].size = 32

     2. 缩容是 ht[1].size 为第一个大于等于 ht[0].used 的 2^n

> 关于负载因子被提高的解释 copy-on-write
>
> - Redis在持久化时，如果是采用BGSAVE命令或者BGREWRITEAOF的方式，那Redis会**fork出一个子进程来读取数据，从而写到磁盘中**。
> - 总体来看，Redis还是读操作比较多。如果子进程存在期间，发生了大量的写操作，那可能就会出现**很多的分页错误(页异常中断page-fault)**，这样就得耗费不少性能在复制上。
> - 而在**rehash阶段上，写操作是无法避免**的。所以Redis在fork出子进程之后，**将负载因子阈值提高**，尽量减少写操作，避免不必要的内存写入操作，最大限度地节约内存。

# ZSET - 跳跃表

表实现

```c
typedef struct zskiplist {
    // 表头节点和表尾节点
    struct zskiplistNode *header, *tail;

    // 表中节点的数量
    unsigned long length;

    // 表中层数最大的节点的层数
    int level;

} zskiplist;
```

节点实现

```c
typedef struct zskiplistNode {
    // 后退指针
    struct zskiplistNode *backward;

    // 分值
    double score;

    // 成员对象
    robj *obj;

    // 层
    struct zskiplistLevel {

        // 前进指针
        struct zskiplistNode *forward;

        // 跨度
        unsigned int span;

    } level[];

} zskiplistNode;
```

![img](/assets/img/v2-fe149d6946043f043c8b1372e2a944ea_720w.jpg)

层高是1-32之间的随机数，总共能存2^64节点



### 排序方式

在同一个跳跃表中， 各个节点保存的成员对象必须是唯一的， 但是多个节点保存的分值却可以是相同的： 分值相同的节点将按照成员对象在字典序中的大小来进行排序， 成员对象较小的节点会排在前面（靠近表头的方向）， 而成员对象较大的节点则会排在后面（靠近表尾的方向）。

### 跳跃表结构

![img](/assets/img/v2-83708fbbffeee5c33d12e0a6a155526c_720w.jpg)

### 查找方式

![img](/assets/img/v2-be6d1566e00cc463cfb4b0adb8d48b40_720w.jpg)



# 整数集合intset - SET底层实现之一

- SET只包含整数值元素，并且这个集合的元素数量不多

- intset 可以保存int16 32 64，集合中的元素不重复

### 数据结构定义

```c
typedef struct intset {
    // 编码方式
    uint32_t encoding;
    // 集合包含的元素数量
    uint32_t length;
    // 保存元素的数组，从小到大排列，类型根据encoding定义
    int8_t contents[];
} intset;
```

![image-20201222123838577](/assets/img/image-20201222123838577.png)

### 升级

如果现在存的是int8 ，有个int64的数要存入，就要进行升级

步骤

1. 根据新元素类型，扩展底层数组的空间大小，并为新元素分配空间
2. 将旧的转移入新的
3. 将新加入的元素存入新数组

好处：

1. 节省内存

2. 不同类型存在不同数组，不容易出错



### 降级

整数集合不支持降级操作



# 压缩列表 - list -hash

- 少量数据项，并且每个列表项是小整数值

ziplist是list键、hash键以及zset键的底层实现之一（**3.0之后list键已经不直接用ziplist和linkedlist作为底层实现了，取而代之的是quicklist**）
这些键的常规底层实现如下：

- **list键**：双向链表
- **hash键**：字典dict
- **zset键**：跳跃表zskiplist

但是当list键里包含的元素较少、并且每个元素要么是小整数要么是长度较小的字符串时，redis将会用ziplist作为list键的底层实现。同理hash和zset在这种场景下也会使用ziplist。

既然已有底层结构可以实现list、hash、zset键，为什么还要用ziplist呢？
当然是为了节省内存空间
我们先来看看ziplist是如何压缩的

## 原理

整体布局

ziplist是由***一系列特殊编码的连续内存块组成的顺序存储结构\***，类似于数组，ziplist在内存中是连续存储的，但是不同于数组，为了节省内存 ziplist的每个元素所占的内存大小可以不同（数组中叫元素，ziplist叫节点**entry**，下文都用“节点”），每个节点可以用来存储一个整数或者一个字符串。
下图是ziplist在内存中的布局

![img](/assets/img/20190430153606404.png)

 

- zlbytes: ziplist的长度（单位: 字节)，是一个32位无符号整数
- zltail: ziplist最后一个节点的偏移量，反向遍历ziplist或者pop尾部节点的时候有用。
- zllen: ziplist的节点（entry）个数
- entry: 节点
- zlend: 值为0xFF，用于标记ziplist的结尾

普通数组的遍历是根据数组里存储的数据类型 找到下一个元素的，例如int类型的数组访问下一个元素时每次只需要移动一个sizeof(int)就行（实际上开发者只需让指针p+1就行，在这里引入sizeof(int)只是为了说明区别）。
上文说了，ziplist的每个节点的长度是可以不一样的，而我们面对不同长度的节点又不可能直接sizeof(entry)，那么它是怎么访问下一个节点呢？
**ziplist将一些必要的偏移量信息记录在了每一个节点里，使之能跳到上一个节点或下一个节点。**
接下来我们看看节点的布局

节点的布局(entry)

每个节点由三部分组成：prevlength、encoding、data

- prevlengh: 记录上一个节点的长度，为了方便反向遍历ziplist
- encoding: 当前节点的编码规则，下文会详细说
- data: 当前节点的值，可以是数字或字符串 

为了节省内存，**根据上一个节点的长度prevlength 可以将ziplist节点分为两类：**

 ![img](/assets/img/20190430153651857.png)

- entry的前8位小于254，则这8位就表示上一个节点的长度
- entry的前8位等于254，则意味着上一个节点的长度无法用8位表示，后面32位才是真实的prevlength。**用254 不用255(11111111)作为分界是因为255是zlend的值，它用于判断ziplist是否到达尾部。**

**根据当前节点存储的数据类型及长度，可以将ziplist节点分为9类**：
其中整数节点分为6类： 

![img](/assets/img/20190430153745298.png)

 整数节点的encoding的长度为8位，其中**高2位**用来区分整数节点和字符串节点（高2位为11时是整数节点），**低6位**用来区分整数节点的类型，定义如下:

```
#define ZIP_INT_16B (0xc0 | 0<<4)//整数data,占16位（2字节）



#define ZIP_INT_32B (0xc0 | 1<<4)//整数data,占32位（4字节）



#define ZIP_INT_64B (0xc0 | 2<<4)//整数data,占64位（8字节）



#define ZIP_INT_24B (0xc0 | 3<<4)//整数data,占24位（3字节）



#define ZIP_INT_8B 0xfe //整数data,占8位（1字节）



/* 4 bit integer immediate encoding */



//整数值1~13的节点没有data，encoding的低四位用来表示data



#define ZIP_INT_IMM_MASK 0x0f



#define ZIP_INT_IMM_MIN 0xf1    /* 11110001 */



#define ZIP_INT_IMM_MAX 0xfd    /* 11111101 */
```

值得注意的是 最后一种encoding是存储整数0~12的节点的encoding，它没有额外的data部分，encoding的**高4位**表示这个类型，**低4位**就是它的data。这种类型的节点的encoding大小介于ZIP_INT_24B与ZIP_INT_8B之间（1~13），但是为了表示整数0，取出低四位xxxx之后会将其**-1**作为实际的data值（0~12）。在函数zipLoadInteger中，我们可以看到这种类型节点的取值方法：

```
...



 } else if (encoding >= ZIP_INT_IMM_MIN && encoding <= ZIP_INT_IMM_MAX) {



        ret = (encoding & ZIP_INT_IMM_MASK)-1;



 }



...
```

字符串节点分为3类：

![img](/assets/img/2019043015390263.png)

- 当data小于63字节时(2^6)，节点存为上图的第一种类型，**高2位**为00，**低6位**表示data的长度。
- 当data小于16383字节时(2^14)，节点存为上图的第二种类型，**高2位**为01，后续14位表示data的长度。
- 当data小于4294967296字节时(2^32)，节点存为上图的第二种类型，**高2位**为10，下一字节起连续32位表示data的长度。

上图可以看出：
不同于整数节点encoding永远是8位，字符串节点的encoding可以有8位、16位、40位三种长度
相同encoding类型的**整数节点** data长度是固定的，但是相同encoding类型的**字符串节点**，data长度取决于encoding后半部分的值



## 对象

Redis没有直接使用数据结构来实现键值对数据库，而是基于他们封装了五个对象

字符串对象、列表对象、哈希对象、集合对象和有序集合对象

对象会记录自己最后一次被访问的时间，这个时间可以用于计算对象的空转时间。

### 对象类型和编码

TYPE key

查看类型

| 对象         | 对象 `type` 属性的值 | TYPE 命令的输出 |
| :----------- | :------------------- | :-------------- |
| 字符串对象   | `REDIS_STRING`       | `"string"`      |
| 列表对象     | `REDIS_LIST`         | `"list"`        |
| 哈希对象     | `REDIS_HASH`         | `"hash"`        |
| 集合对象     | `REDIS_SET`          | `"set"`         |
| 有序集合对象 | `REDIS_ZSET`         | `"zset"`        |



OBJECT ENCODING key

查看编码

| 对象所使用的底层数据结构             | 编码常量                    | OBJECT ENCODING 命令输出 |
| :----------------------------------- | :-------------------------- | :----------------------- |
| 整数                                 | `REDIS_ENCODING_INT`        | `"int"`                  |
| `embstr` 编码的简单动态字符串（SDS） | `REDIS_ENCODING_EMBSTR`     | `"embstr"`               |
| 简单动态字符串                       | `REDIS_ENCODING_RAW`        | `"raw"`                  |
| 字典                                 | `REDIS_ENCODING_HT`         | `"hashtable"`            |
| 双端链表                             | `REDIS_ENCODING_LINKEDLIST` | `"linkedlist"`           |
| 压缩列表                             | `REDIS_ENCODING_ZIPLIST`    | `"ziplist"`              |
| 整数集合                             | `REDIS_ENCODING_INTSET`     | `"intset"`               |
| 跳跃表和字典                         | `REDIS_ENCODING_SKIPLIST`   | `"skiplist"`             |



## 字符串对象

字符串对象保存各类型值的编码方式

| 值                                                           | 编码                |
| :----------------------------------------------------------- | :------------------ |
| 可以用 `long` 类型保存的整数。                               | `int`               |
| 可以用 `long double` 类型保存的浮点数。                      | `embstr` 或者 `raw` |
| 字符串值， 或者因为长度太大而没办法用 `long` 类型表示的整数， 又或者因为长度太大而没办法用 `long double` 类型表示的浮点数。 | `embstr` 或者 `raw` |



embstr 创建和回收只需要一次内存分配，内存连续



使用条件：

1. 小于等于44字节用embstr



## 列表对象

ziplist 或者 quicklist

 

使用条件：

1. 元素个数小于 list-max-ziplist-entries 521字节，同时值小于list-max-ziplist-value 64字节



## 哈希对象

ziplist 或者 hashtable

使用条件：

1. 哈希对象键和值的字符串长度小于64字节
2. 键小于512个



## 集合对象

intset 或者 hashtable

使用条件：

1. 集合对象保存的所有元素都是整数值
2. 元素不超过512个



## 有序集合对象

ziplist 或者 skiplist

使用条件：

1. 有序集合的元素数量小于128
2. 成员长度小于64字节



## 内存回收

引用计数

1. 创建时候计数值为1
2. 引用一次加1，不引用了减1
3. 0的时候回收



## 对象共享

多个键同时引用一个对象

Redis初始化会创建 0 - 9999 数值型字符串对象 ，以便其他对象共享