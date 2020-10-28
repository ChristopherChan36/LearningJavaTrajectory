# Redis 基础数据结构

> Redis 是互联网技术领域使用最为广泛的**存储中间件**，它是「**Re**mote **Di**ctionary **S**ervice」的首字母缩写，也就是「远程字典服务」。Redis 以其超高的性能、完美的文档、简洁易懂的源码和丰富的客户端库支持在开源中间件领域广受好评。国内外很多大型互联网公司都在使用 Redis。也可以说，对 Redis 的了解和应用实践已成为当下中高级后端开发者绕不开的必备技能。

## Redis 简介

> **"Redis is an open source (BSD licensed), in-memory data structure store, used as a database, cache and message broker."** —— Redis是一个开放源代码（BSD许可）的内存中数据结构存储，用作数据库，缓存和消息代理。*(摘自官网)*

**Redis** 是一个开源，高性能的键值存储和一个适用的解决方案，用于构建高性能，可扩展的 Web 应用程序。**Redis** 也被作者戏称为 *数据结构服务器* ，这意味着使用者可以通过一些命令，基于带有 TCP 套接字的简单 *服务器-客户端* 协议来访问一组 **可变数据结构** 。*(在 Redis 中都采用键值对的方式，只不过对应的数据结构不一样罢了)*

### Redis 的优点

以下是 Redis 的一些优点：

- **异常快** - Redis 非常快，每秒可执行大约 110000 次的设置(SET)操作，每秒大约可执行 81000 次的读取/获取(GET)操作。
- **支持丰富的数据类型** - Redis 支持开发人员常用的大多数数据类型，例如列表，集合，排序集和散列等等。这使得 Redis 很容易被用来解决各种问题，因为我们知道哪些问题可以更好使用地哪些数据类型来处理解决。
- **操作具有原子性** - 所有 Redis 操作都是原子操作，这确保如果两个客户端并发访问，Redis 服务器能接收更新的值。
- **多实用工具** - Redis 是一个多实用工具，可用于多种用例，如：缓存，消息队列(Redis 本地支持发布/订阅)，应用程序中的任何短期数据，例如，web应用程序中的会话，网页命中计数等。

### Redis 应用场景

Redis的业务应用范围非常广泛，让我们以技术社区的帖子模块为实例，梳理一下，Redis 可以用在哪些地方？

1. 记录帖子的点赞数、评论数和点击数 (hash)。
2. 记录用户的帖子 ID 列表 (排序)，便于快速显示用户的帖子列表 (zset)。
3. 记录帖子的标题、摘要、作者和封面信息，用于列表页展示 (hash)。
4. 记录帖子的点赞用户 ID 列表，评论 ID 列表，用于显示和去重计数 (zset)。
5. 缓存近期热帖内容 (帖子内容空间占用比较大)，减少数据库压力 (hash)。
6. 记录帖子的相关文章 ID，根据内容推荐相关帖子 (list)。
7. 如果帖子 ID 是整数自增的，可以使用 Redis 来分配帖子 ID(计数器)。
8. 收藏集和帖子之间的关系 (zset)。
9. 记录热榜帖子 ID 列表，总热榜和分类热榜 (zset)。
10. 缓存用户行为历史，进行恶意行为过滤 (zset,hash)。

当然，实际情况下需求可能也没这么多，因为在请求压力不大的情况下，很多数据都是可以直接从数据库中查询的。但请求压力一大，以前通过数据库直接存取的数据则必须要挪到缓存里来。

以上提到的只是 Redis 的基础应用，也是日常开发中最常见的应用。

## Redis 安装

要体验 Redis，我们先从 Redis 安装说起。

体验 Redis 需要使用 Linux 或者 Mac 环境，如果是 Windows 可以考虑使用虚拟机。主要方式有四种：

1. 使用 Docker 安装。
2. 通过 Github 源码编译。
3. 直接安装 apt-get install(Ubuntu)、yum install(RedHat) 或者 brew install(Mac)。
4. 使用网页版的 [Web Redis](http://try.redis.io/#run) 直接体验。

具体操作如下：

### 源码编译方式

**下载并上传至 linux**

官网：https://redis.io/download
选择下载稳定版本，不稳定版本可以尝鲜，但是不推荐在生产使用。

![上传至 linux](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-31-094757.png)

**安装 Redis**

1. 解压 redis

   ```bash
   tar -zxvf redis-5.0.5.tar.gz
   ```

2. 安装gcc编译环境，如果已经安装过了，那么就是 nothing to do

   ```bash
   yum install gcc-c++ 
   ```

3. 进入到 `redis-5.0.5` 目录，进行安装：

   ```bash
   make && make install
   ```

执行完毕后安装成功

**Redis 开机自启动**

1. 配置redis，在utils下，拷贝`redis_init_script`到`/etc/init.d`目录，目的要把redis作为开机自启动

   ![拷贝自启动脚本](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-31-110831.png)

2. 创建 `/usr/local/redis`，用于存放配置文件

   ![创建配置文件](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-31-111245.png)

3. 拷贝redis配置文件：拷贝至 `/usr/local/redis`下

   ![拷贝 redis 配置文件](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-31-111853.png)

4. 修改 redis.conf 核心配置文件

   - 修改 daemonize no -> daemonize yes，目的是为了让redis启动在linux后台运行

     ![daemonize](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-31-113113.png)

   - 修改 redis 的工作目录

     ![redis 工作目录](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-31-113536.png)

     建议修改为： `/usr/local/redis/working`，名称随意

   - 修改如下内容，绑定IP改为 0.0.0.0 ，代表可以让远程连接，不收ip限制

     ![远程连接](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-31-114125.png)

   - 最关键的是密码，默认是没有的，一定要设置

     ![密码设置](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-31-114452.png)

5. 修改 `redis` 文件中的redis核心配置文件为如下：

   ![修改 redis 配置文件路径](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-31-114943.png)

6. 为 redis 启动脚本添加执行权限，随后运行启动redis

   ![启动脚本](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-31-115401.png)

7. 检查 redis 进程

   ![检查 redis 进程](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-31-115626.png)

8. 设置 redis 开机自启动，修改 `redis`，添加如下内容

   ```bash
   #chkconfig: 22345 10 90
   #description: Start and Stop redis
   ```

   ![自启动脚本](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-31-120057.png)

   随后执行如下操作：

   ```bash
   chkconfig redis on
   ```

   重启 redis，再看进程：

### Docker 方式

```bash
# 拉取 redis 镜像
> docker pull redis
# 运行 redis 容器
> docker run --name myredis -d -p6379:6379 redis
# 执行容器中的 redis-cli，可以直接使用命令行操作 redis
> docker exec -it myredis redis-cli
```

### 直接安装方式

```bash
# mac
> brew install redis
# ubuntu
> apt-get install redis
# redhat
> yum install redis
# 运行客户端
> redis-cli
```

##  测试本地 Redis 性能

当你安装完成之后，你可以先执行 `redis-server` 让 Redis 启动起来，然后运行命令 `redis-benchmark -n 100000 -q` 来检测本地同时执行 10 万个请求时的性能：

![redis 性能](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-05-31-121702.png)

### Redis 命令行客户端

安装好 Redis，我们可以使用 `redis-cli`来对 Redis 服务器进行命令行操作，当然 Redis 官方也提供了在线调试器：http://try.redis.io/#run

`redis-cli -a password shutdown`：关闭redis

`redis-cli`：进入到redis客户端

`auth pwd`：输入密码

`redis-cli -a password ping`：查看是否存活

## Redis 基础数据结构

Redis 有 5 种基础数据结构，分别为：**string(字符串)**、**list(列表)**、**hash(字典)**、**set(集合)** 和 **zset(有序集合)**。熟练掌握这 5 种基本数据结构的使用是 Redis 知识最基础也最重要的部分，它也是在 Redis 面试题中问到最多的内容。

下面我们结合源码以及一些实践来给大家分别讲解一下。

### 字符串 string

Redis 中的字符串是一种 **动态字符串**，这意味着使用者可以修改，它的底层实现有点类似于 Java 中的 **ArrayList**，有一个字符数组，从源码的 **sds.h/sdshdr 文件** 中可以看到 Redis 底层对于字符串的定义 **SDS**，即 *Simple Dynamic String* 结构：

```c
/* Note: sdshdr5 is never used, we just access the flags byte directly.
 * However is here to document the layout of type 5 SDS strings. */
struct __attribute__ ((__packed__)) sdshdr5 {
    unsignedchar flags; /* 3 lsb of type, and 5 msb of string length */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; /* used */
    uint8_t alloc; /* excluding the header and null terminator */
    unsignedchar flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* used */
    uint16_t alloc; /* excluding the header and null terminator */
    unsignedchar flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; /* used */
    uint32_t alloc; /* excluding the header and null terminator */
    unsignedchar flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsignedchar flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
```

你会发现同样一组结构 Redis 使用泛型定义了好多次，**为什么不直接使用 int 类型呢？**

因为当字符串比较短的时候，len 和 alloc 可以使用 byte 和 short 来表示，**Redis 为了对内存做极致的优化，不同长度的字符串使用不同的结构体来表示。**

#### SDS 与 C 字符串的区别

为什么不考虑直接使用 C 语言的字符串呢？因为 C 语言这种简单的字符串表示方式 **不符合 Redis 对字符串在安全性、效率以及功能方面的要求**。我们知道，C 语言使用了一个长度为 N+1 的字符数组来表示长度为 N 的字符串，并且字符数组最后一个元素总是 `'\0'`。*(下图就展示了 C 语言中值为 "Redis" 的一个字符数组)*

![C 语言中的字符串](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-06-01-235809.png)

这样简单的数据结构可能会造成以下一些问题：

- **获取字符串长度为 O(N) 级别的操作** → 因为 C 不保存数组的长度，每次都需要遍历一遍整个数组
- 不能很好的杜绝 **缓冲区溢出/内存泄漏** 的问题 → 跟上述问题原因一样，如果执行拼接或者缩短字符串的操作，如果操作不当就很容易造成上述问题；
- C 字符串 **只能保存文本数据** → 因为 C 语言中的字符串必须符合某种编码（比如 ASCII），例如中间出现的 `'\0'` 可能会被判定为提前结束的字符串而识别不了；

我们以追加字符串的操作举例，Redis 源码如下：

```c
/* Append the specified binary-safe string pointed by 't' of 'len' bytes to the
 * end of the specified sds string 's'.
 *
 * After the call, the passed sds string is no longer valid and all the
 * references must be substituted with the new pointer returned by the call. */
sds sdscatlen(sds s, const void *t, size_t len) {
    // 获取原字符串的长度
    size_t curlen = sdslen(s);
  
    // 按需调整空间，如果容量不够容纳追加的内容，就会重新分配字节数组并复制原字符串的内容到新数组中
    s = sdsMakeRoomFor(s,len);
    if (s == NULL) returnNULL;   // 内存不足
    memcpy(s+curlen, t, len);     // 追加目标字符串到字节数组中
    sdssetlen(s, curlen+len);     // 设置追加后的长度
    s[curlen+len] = '\0';         // 让字符串以 \0 结尾，便于调试打印
    return s;
}
```

- **注：Redis 规定了字符串的长度不得超过 512 MB。**

字符串结构使用非常广泛，一个常见的用途就是缓存用户信息。我们将用户信息结构体使用 JSON 序列化成字符串，然后将序列化后的字符串塞进 Redis 来缓存。同样，取用户信息会经过一次反序列化的过程。

**小结**

![redis-string 底层结构](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/05/2020-06-02-000350.png)

Redis 的字符串是`动态字符串`，是可以修改的字符串，内部结构实现上类似于 Java 的 `ArrayList`，采用`预分配冗余空间`的方式来减少内存的频繁分配，如图中所示，内部为当前字符串实际分配的空间 capacity 一般要高于实际字符串长度 len。当字符串长度小于 1M 时，扩容都是加倍现有的空间，如果超过 1M，扩容时一次只会多扩 1M 的空间。需要注意的是字符串最大长度为 512M。

#### 字符串的基本操作

**设置和获取键值对**

```bash
> SET key value
OK
> GET key
"value"
```

正如你看到的，我们通常使用 `SET` 和 `GET` 来设置和获取字符串值。

值可以是任何种类的字符串（包括二进制数据），例如你可以在一个键下保存一张 `.jpeg` 图片，只需要注意不要超过 512 MB 的最大限度就好了。

当 key 存在时，`SET` 命令会覆盖掉你上一次设置的值：

```bash
> SET key newValue
OK
> GET key
"newValue"
```

另外你还可以使用 `EXISTS` 和 `DEL` 关键字来查询是否存在和删除键值对：

```bash
> EXISTS key
(integer) 1
> DEL key
(integer) 1
> GET key
(nil)
```

**批量设置键值对**

可以批量对多个字符串进行读写，节省网络耗时开销。

```bash
> set name1 codehole
OK
> set name2 holycoder
OK
> mget name1 name2 name3 # 返回一个列表
1) "codehole"
2) "holycoder"
3) (nil)
> mset name1 boy name2 girl name3 unknown
> mget name1 name2 name3
1) "boy"
2) "girl"
3) "unknown"
```

**过期和 set 命令扩展**

可以对 key 设置过期时间，到点自动删除，这个功能常用来控制缓存的失效时间。不过这个「自动删除」的机制是比较复杂的，如果你感兴趣，可以参考：https://juejin.im/book/5afc2e5f6fb9a07a9b362527/section/5b4c42405188251b3950d251

```bash
> set name codehole
> get name
"codehole"
> expire name 5  # 5s 后过期
...  # wait for 5s
> get name
(nil)
```

等价于 `SET` + `EXPIRE` 的 `SETEX` 命令

```bash
> setex name 5 codehole  # 5s 后过期，等价于 set+expire
> get name
"codehole"
... # wait for 5s
> get name
(nil)
```

`SETNX` key 不存在则创建，存在则创建失败

```bash
> setnx name codehole  # 如果 name 不存在就执行 set 创建
(integer) 1
> get name
"codehole"
> setnx name holycoder
(integer) 0  # 因为 name 已经存在，所以 set 创建不成功
> get name
"codehole"  # 没有改变
```

**计数**

如果 value 值是一个整数，还可以对它使用 `INCR` 命令进行**原子性**的自增操作。自增是有范围的，它的范围是 signed long 的最大最小值，超过了这个值，Redis 会报错。

```bash
> set age 30
OK
> incr age
(integer) 31
> incrby age 5
(integer) 36
> incrby age -5
(integer) 31
> set codehole 9223372036854775807  # Long.Max
OK
> incr codehole
(error) ERR increment or decrement would overflow
```

**返回原值的 GETSET 命令**

对字符串，还有一个`GETSET` 比较有意思，它的功能跟它的名字一样，为 key 设置一个值并返回原值

```bash
> SET key value
> GETSET key value1
"value"
```

这可以对于某一些需要隔一段时间就统计的 key 很方便的设置和查看，例如：系统每当由用户进入的时候你就是用 `INCR` 命令操作一个 key，当需要统计时候你就把这个 key 使用 `GETSET` 命令重新赋值为 0，这样就达到了统计的目的。

### 列表 list

Redis 的列表相当于 Java 语言里面的 **LinkedList**，注意它是链表而不是数组。这意味着 list 的插入和删除操作非常快，时间复杂度为 O(1)，但是索引定位很慢，时间复杂度为 O(n)。

我们可以从源码的 `adlist.h/listNode` 来看到对其的定义：

```c
/* Node, List, and Iterator are the only data structures used currently. */
typedef struct listNode {
    struct listNode *prev;
    struct listNode *next;
    void *value;
} listNode;

typedef struct listIter {
    listNode *next;
    int direction;
} listIter;

typedef struct list {
    listNode *head;
    listNode *tail;
    void *(*dup)(void *ptr);
    void (*free)(void *ptr);
    int (*match)(void *ptr, void *key);
    unsigned long len;
} list;
```

可以看到，多个 listNode 可以通过 `prev` 和 `next` 指针组成双向链表：

![Redis listNode](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/06/2020-06-03-131613.png)

虽然仅仅使用多个 listNode 结构就可以组成链表，但是使用 `adlist.h/list` 结构来持有链表的话，操作起来会更加方便：

![Redis list 结构](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/06/2020-06-03-131727.png)

#### 链表的基本操作

`LPUSH` 和 `RPUSH` 分别可以向 list 的左边（头部）和右边（尾部）添加一个新元素；

`LRANGE` 命令可以从 list 中取出一定范围的元素；

`LINDEX` 命令可以从 list 中取出指定下表的元素，相当于 Java 链表操作中的 `get(int index)` 操作；

**list 实现队列**

队列是先进先出的数据结构，常用于消息排队和异步逻辑处理，它会确保元素的访问顺序：

右进左出实现

```bash
> rpush books python java golang
(integer) 3
> llen books
(integer) 3
> lpop books
"python"
> lpop books
"java"
> lpop books
"golang"
> lpop books
(nil)
```

**list 实现栈**

栈是先进后出的数据结构，跟队列正好相反：

右进右出实现

```bash
> rpush books python java golang
(integer) 3
> rpop books
"golang"
> rpop books
"java"
> rpop books
"python"
> rpop books
(nil)
```

#### 慢操作

`lindex` 相当于 Java 链表的`get(int index)`方法，它需要对链表进行遍历，性能随着参数`index`增大而变差。

`lrange` 相当于遍历打印链表中的元素，使用`start_index`和`end_index`两个参数定义遍历元素区间。

`ltrim` 和字面上的含义不太一样，个人觉得它叫 lretain(保留) 更合适一些，因为 ltrim 跟的两个参数`start_index`和`end_index`定义了一个区间，在这个区间内的值，ltrim 要保留，区间之外统统砍掉。我们可以通过ltrim来实现一个定长的链表，这一点非常有用。

index 可以为负数，`index=-1`表示倒数第一个元素，同样`index=-2`表示倒数第二个元素。

```bash
> rpush books python java golang
(integer) 3
> lindex books 1  # O(n) 慎用
"java"
> lrange books 0 -1  # 获取所有元素，O(n) 慎用
1) "python"
2) "java"
3) "golang"
> ltrim books 1 -1 # O(n) 慎用
OK
> lrange books 0 -1
1) "java"
2) "golang"
> ltrim books 1 0 # 这其实是清空了整个列表，因为区间范围长度为负
OK
> llen books
(integer) 0
```

#### 快速列表

![img](https://user-gold-cdn.xitu.io/2018/7/27/164d975cac9559c5)



如果再深入一点，你会发现 Redis 底层存储的还不是一个简单的 `linkedlist`，而是称之为快速链表 `quicklist` 的一个结构。

首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构是 `ziplist`，也即是压缩列表。它将所有的元素紧挨着一起存储，分配的是一块连续的内存。当数据量比较多的时候才会改成 `quicklist`。因为普通的链表需要的附加指针空间太大，会比较浪费空间，而且会加重内存的碎片化。比如这个列表里存的只是 `int` 类型的数据，结构上还需要两个额外的指针 `prev` 和 `next` 。所以 Redis 将链表和 `ziplist` 结合起来组成了 `quicklist`。也就是将多个 `ziplist` 使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。

关于压缩列表和快速列表的内部结构实现参考：

- https://juejin.im/book/5afc2e5f6fb9a07a9b362527/section/5b5c95226fb9a04fa42fc3f6
- https://juejin.im/book/5afc2e5f6fb9a07a9b362527/section/5b5c963be51d45199154e82e

### 字典 hash

Redis 的字典相当于 Java 语言里面的 `HashMap`，它是无序字典。内部实现结构上同 Java 的 HashMap 也是一致的，同样的`『数组 + 链表』`二维结构。第一维 hash 的数组位置碰撞时，就会将碰撞的元素使用链表串接起来。不同的是，**Redis 的字典的值只能是字符串**。

![hash 字典结构](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/06/2020-06-07-140554.png)

#### 字典 hash 源码

源码定义 `dict.h/dictht`：

```c
typedef struct dictht {
    // 哈希表数组
    dictEntry **table;
    // 哈希表大小
    unsigned long size;
    // 哈希表大小掩码，用于计算索引值，总是等于 size - 1
    unsigned long sizemask;
    // 该哈希表已有节点的数量
    unsigned long used;
} dictht;

typedef struct dict {
    dictType *type;
    void *privdata;
    // 内部有两个 dictht 结构
    dictht ht[2];
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    unsigned long iterators; /* number of iterators currently running */
} dict;
```

`table` 属性是一个数组，数组中的每个元素都是一个指向 `dict.h/dictEntry` 结构的指针，而每个 `dictEntry` 结构保存着一个键值对：

```c
typedef struct dictEntry {
    // 键
    void *key;
    // 值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    // 指向下个哈希表节点，形成链表
    struct dictEntry *next;
} dictEntry;
```

可以从上面的源码中看到，**实际上字典结构的内部包含两个 hashtable**，通常情况下只有一个 hashtable 是有值的，但是在字典扩容缩容时，需要分配新的 hashtable，然后进行 **渐进式搬迁** *(下面说原因)*。

#### 渐进式 rehash

Java 的 HashMap 在字典很大时，rehash 是个耗时的操作，需要重新申请新的数组，然后将旧字典中所有链表中的元素重新rehash 到新的数组下面，这是一个O(n)级别的操作。Redis 为了高性能，不能堵塞服务，所以采用了**渐进式 rehash 策略**。

![Redis rehash 过程](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/06/2020-06-07-140817.png)



渐进式 rehash 会在 rehash 的同时，保留新旧两个 hash 结构，查询时会同时查询两个 hash 结构，然后在后续的**定时任务中以及 hash 操作指令**中，循序渐进地将旧 hash 的内容一点点迁移到新的 hash 结构中。当搬迁完成了，就会使用新的hash结构取而代之。

#### 扩缩容的条件

正常情况下，当 hash 表中 **元素的个数等于第一维数组的长度时**，就会开始扩容，扩容的新数组是 **原数组大小的 2 倍**。不过如果 Redis 正在做 `bgsave(持久化命令)`，为了减少内存也得过多分离，Redis 尽量不去扩容，但是如果 hash 表非常满了，**达到了第一维数组长度的 5 倍了**，这个时候就会 **强制扩容**。

当 hash 表因为元素逐渐被删除变得越来越稀疏时，Redis 会对 hash 表进行缩容来减少 hash 表的第一维数组空间占用。所用的条件是 **元素个数低于数组长度的 10%**，缩容不会考虑 Redis 是否在做 `bgsave`。

#### 字典的基本操作

hash 结构也可以用来存储用户信息，不同于字符串一次性需要全部序列化整个对象，hash 可以对用户结构中的每个字段单独存储。这样当我们需要获取用户信息时可以进行部分获取。而以整个字符串的形式去保存用户信息的话就只能一次性全部读取，这样就会比较浪费网络流量。

hash 也有缺点，hash 结构的存储消耗要高于单个字符串，到底该使用 hash 还是字符串，需要根据实际情况再三权衡。

```bash
> hset books java "think in java"  # 命令行的字符串如果包含空格，要用引号括起来
(integer) 1
> hset books golang "concurrency in go"
(integer) 1
> hset books python "python cookbook"
(integer) 1
> hgetall books  # entries()，key 和 value 间隔出现
1) "java"
2) "think in java"
3) "golang"
4) "concurrency in go"
5) "python"
6) "python cookbook"
> hlen books
(integer) 3
> hget books java
"think in java"
> hset books golang "learning go programming"  # 因为是更新操作，所以返回 0
(integer) 0
> hget books golang
"learning go programming"
> hmset books java "effective java" python "learning python" golang "modern golang programming"  # 批量 set
OK
```

同字符串对象一样，hash 结构中的单个子 key 也可以进行计数，它对应的指令是 `hincrby`，和 `incr` 使用基本一样。

```bash
# 老钱又老了一岁
> hincrby user-laoqian age 1
(integer) 30
```

### 集合 set

Redis 的集合相当于 Java 语言里面的 `HashSet`，它内部的键值对是无序、唯一的。它的内部实现相当于一个特殊的字典，字典中所有的 value 都是一个值`NULL`。

#### 集合 set 基本使用

set 结构可以用来存储活动中奖的用户 ID，因为有去重功能，可以保证同一个用户不会中奖两次。

```bash
> sadd books python
(integer) 1
> sadd books python  #  重复
(integer) 0
> sadd books java golang
(integer) 2
> smembers books  # 注意顺序，和插入的并不一致，因为 set 是无序的
1) "java"
2) "python"
3) "golang"
> sismember books java  # 查询某个 value 是否存在，相当于 contains(o)
(integer) 1
> sismember books rust
(integer) 0
> scard books  # 获取长度相当于 count()
(integer) 3
> spop books  # 弹出一个
"java"
```

### 有序集合 zset

zset 可能是 Redis 提供的最为特色的数据结构，它也是在面试中面试官最爱问的数据结构。它类似于 Java 的 `SortedSet` 和 `HashMap` 的结合体，一方面它是一个 set，保证了内部 value 的唯一性，另一方面它可以给每个 value 赋予一个 score，代表这个 value 的排序权重。它的内部实现用的是一种叫做**「跳跃表」**的数据结构。

#### 跳跃表

zset 内部的排序功能是通过「跳跃列表」数据结构来实现的，它的结构非常特殊，也比较复杂。

因为 zset 要支持**随机的插入和删除**，所以它不好使用数组来表示。我们先看一个普通的链表结构。

![普通链表结构](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/06/2020-06-07-143138.png)

我们需要这个链表按照 score 值进行排序。这意味着当有新元素需要插入时，要定位到特定位置的插入点，这样才可以继续保证链表是有序的。通常我们会通过二分查找来找到插入点，但是二分查找的对象必须是数组，只有数组才可以支持快速位置定位，链表做不到，那该怎么办？

![Redis 跳跃表简单原理](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/06/2020-06-07-143301.png)

想想一个创业公司，刚开始只有几个人，团队成员之间人人平等，都是联合创始人。随着公司的成长，人数渐渐变多，团队沟通成本随之增加。这时候就会引入组长制，对团队进行划分。每个团队会有一个组长。开会的时候分团队进行，多个组长之间还会有自己的会议安排。公司规模进一步扩展，需要再增加一个层级 —— 部门，每个部门会从组长列表中推选出一个代表来作为部长。部长们之间还会有自己的高层会议安排。

跳跃列表就是类似于这种层级制，最下面一层所有的元素都会串起来。然后每隔几个元素挑选出一个代表来，再将这几个代表使用另外一级指针串起来。然后在这些代表里再挑出二级代表，再串起来。最终就形成了**金字塔结构**。

想想你老家在世界地图中的位置：亚洲-->中国->安徽省->安庆市->枞阳县->汤沟镇->田间村->xxxx号，也是这样一个类似的结构。

![跳跃表结构](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/06/2020-06-07-143422.png)

「跳跃列表」之所以「跳跃」，是因为内部的元素可能「身兼数职」，比如上图中间的这个元素，同时处于 L0、L1 和 L2 层，可以快速在不同层次之间进行「跳跃」。

定位插入点时，先在顶层进行定位，然后下潜到下一级定位，一直下潜到最底层找到合适的位置，将新元素插进去。你也许会问，那新插入的元素如何才有机会「身兼数职」呢？

跳跃列表采取一个**随机策略**来决定新元素可以兼职到第几层。

首先 L0 层肯定是 100% 了，L1 层只有 50% 的概率，L2 层只有 25% 的概率，L3 层只有 12.5% 的概率，一直随机到最顶层 L31 层。绝大多数元素都过不了几层，只有极少数元素可以深入到顶层。列表中的元素越多，能够深入的层次就越深，能进入到顶层的概率就会越大。

#### 有序列表 zset 基础操作

zset 可以用来存粉丝列表，value 值是粉丝的用户 ID，score 是关注时间。我们可以对粉丝列表按关注时间进行排序。

zset 还可以用来存储学生的成绩，value 值是学生的 ID，score 是他的考试成绩。我们可以对成绩按分数进行排序就可以得到他的名次。

```bash
> zadd books 9.0 "think in java"
(integer) 1
> zadd books 8.9 "java concurrency"
(integer) 1
> zadd books 8.6 "java cookbook"
(integer) 1
> zrange books 0 -1  # 按 score 排序列出，参数区间为排名范围
1) "java cookbook"
2) "java concurrency"
3) "think in java"
> zrevrange books 0 -1  # 按 score 逆序列出，参数区间为排名范围
1) "think in java"
2) "java concurrency"
3) "java cookbook"
> zcard books  # 相当于 count()
(integer) 3
> zscore books "java concurrency"  # 获取指定 value 的 score
"8.9000000000000004"  # 内部 score 使用 double 类型进行存储，所以存在小数点精度问题
> zrank books "java concurrency"  # 排名
(integer) 1
> zrangebyscore books 0 8.91  # 根据分值区间遍历 zset
1) "java cookbook"
2) "java concurrency"
> zrangebyscore books -inf 8.91 withscores # 根据分值区间 (-∞, 8.91] 遍历 zset，同时返回分值。inf 代表 infinite，无穷大的意思。
1) "java cookbook"
2) "8.5999999999999996"
3) "java concurrency"
4) "8.9000000000000004"
> zrem books "java concurrency"  # 删除 value
(integer) 1
> zrange books 0 -1
1) "java cookbook"
2) "think in java"
```

## 容器型数据结构的通用规则

list/set/hash/zset 这四种数据结构是容器型数据结构，它们共享下面两条通用规则：

1. create if not exists

   如果容器不存在，那就创建一个，再进行操作。比如 rpush 操作刚开始是没有列表的，Redis 就会自动创建一个，然后再 rpush 进去新元素。

2. drop if no elements

   如果容器里元素没有了，那么立即删除元素，释放内存。这意味着 lpop 操作到最后一个元素，列表就消失了。

## 过期时间

Redis 所有的数据结构都可以设置过期时间，时间到了，Redis 会自动删除相应的对象。需要注意的是过期是以对象为单位，比如一个 hash 结构的过期是整个 hash 对象的过期，而不是其中的某个子 key。

还有一个需要特别注意的地方是如果一个字符串已经设置了过期时间，然后你调用了 set 方法修改了它，它的过期时间会消失。

```bash
127.0.0.1:6379> set codehole yoyo
OK
127.0.0.1:6379> expire codehole 600
(integer) 1
127.0.0.1:6379> ttl codehole
(integer) 597
127.0.0.1:6379> set codehole yoyo
OK
127.0.0.1:6379> ttl codehole
(integer) -1
```

## 扩展/相关阅读

1. 阿里云 Redis 开发规范 - https://www.infoq.cn/article/K7dB5AFKI9mr5Ugbs_px
2. 为什么要防止 bigkey？ - https://mp.weixin.qq.com/s?__biz=Mzg2NTEyNzE0OA==&mid=2247483677&idx=1&sn=5c320b46f0e06ce9369a29909d62b401&chksm=ce5f9e9ef928178834021b6f9b939550ac400abae5c31e1933bafca2f16b23d028cc51813aec&scene=21#wechat_redirect
3. Redis【入门】就这一篇！ - https://www.wmyskxz.com/2018/05/31/redis-ru-men-jiu-zhe-yi-pian/

## 参考资料

1. 《Redis 设计与实现》 - http://redisbook.com/
2. 【官方文档】Redis 数据类型介绍 - http://www.redis.cn/topics/data-types-intro.html
3. 《Redis 深度历险》 - https://book.douban.com/subject/30386804/
4. 阿里云 Redis 开发规范 - https://www.infoq.cn/article/K7dB5AFKI9mr5Ugbs_px
5. Redis 快速入门 - 易百教程 - https://www.yiibai.com/redis/redis_quick_guide.html
6. Redis【入门】就这一篇! - https://www.wmyskxz.com/2018/05/31/redis-ru-men-jiu-zhe-yi-pian/