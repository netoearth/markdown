> **摘要：**  你真的懂 Redis 的 5 种基本数据结构吗？这些知识点或许你还需要看看。

本文分享自华为云社区《[你真的懂 Redis 的 5 种基本数据结构吗？这些知识点或许你还需要看看](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fbbs.huaweicloud.com%2Fblogs%2F303862%3Futm_source%3Doschina%26utm_medium%3Dbbs-ex%26utm_campaign%3Dother%26utm_content%3Dcontent)》，作者：李子捌。

## 一、简介

Redis 中所有的的数据结构都是通过一个唯一的字符串 key 来获取相应的 value 数据。  
Redis 有 5 种基础数据结构，分别是：

-   string（字符串）
-   list（列表）
-   hash（字典）
-   set（集合）
-   zset（有序集合）

其中 list、set、hash、zset 这四种数据结构是容器型数据结构，它们共享下面两条通用规则：

-   create if not exists：容器不存在则创建
-   drop if no elements：如果容器中没有元素，则立即删除容器，释放内存

本文将详细讲述的是 Redis 的 5 种基础数据结构。

## 二、string（字符串）

### 1、string (字符串) 相关介绍

### **1.1 string (字符串) 的内部结构**

string (字符串) 是 Redis 最简单也是使用最广泛的数据结构，它的内部是一个字符数组。如图所示：

![](https://pic1.zhimg.com/80/v2-7065c2ab8d52270779b1201cd6779ad8_720w.jpg)

Redis 中 string (字符串) 是动态字符串，允许修改；它在结构上的实现类似于 Java 中的 ArrayList（默认构造一个大小为 10 的初始数组），这是冗余分配内存的思想，也称为预分配；这种思想可以减少扩容带来的性能消耗。

![](https://pic1.zhimg.com/80/v2-283ca1397bafa13e95317fbc906ee49c_720w.jpg)

### **1.2 string (字符串) 的扩容**

当 string (字符串) 的大小达到扩容阈值时，将会对 string (字符串) 进行扩容，string (字符串) 的扩容主要有以下几个点：

1.  长度小于 1MB，扩容后为原先的两倍；length = length \* 2
2.  长度大于 1MB，扩容后增加 1MB; length = length + 1MB
3.  字符串的长度最大值为 512MB

### 2、string (字符串) 的指令

### **2.1 单个键值对增删改查操作**

set -> key 不存在则新增，存在则修改

> set key value

get -> 查询，返回对应 key 的 value，不存在返回 (nil)

> get key

del -> 删除指定的 key (key 可以是多个)

> del key \[key …\]

示例：

```
 1127.0.0.1:6379> set name liziba
 2OK
 3127.0.0.1:6379> get name
 4"liziba"
 5127.0.0.1:6379> set name liziba001
 6OK
 7127.0.0.1:6379> get name
 8"liziba001"
 9127.0.0.1:6379> del name
10(integer) 1
11127.0.0.1:6379> get name
12(nil)
```

### **2.2 批量键值对**

**批量键值读取和写入最大的优势在于节省网络传输开销**

mset -> 批量插入

> mset key value \[key value …\]

mget -> 批量获取

> mget key \[key …\]

示例：

```
1127.0.0.1:6379> mset name1 liziba1 name2 liziba2 name3 liziba3
2OK
3127.0.0.1:6379> mget name1 name2 name3
41) "liziba1"
52) "liziba2"
63) "liziba3"
```

### **2.3 过期 set 命令**

过期 set 是通过设置一个缓存 key 的过期时间，使得缓存到期后自动删除从而失效的机制。  
  
方式一：

> expire key seconds

示例：

```
1127.0.0.1:6379> set name liziba
2OK
3127.0.0.1:6379> get name
4"liziba"
5127.0.0.1:6379> expire name 10   # 10s 后get name 返回 nil
6(integer) 1
7127.0.0.1:6379> get name
8(nil)
```

方式二：

> setex key seconds value

示例：

```
1127.0.0.1:6379> setex name 10 liziba    # 10s 后get name 返回 nil
2OK
3127.0.0.1:6379> get name
4(nil)
```

### **2.4 不存在创建存在不更新**

上面的 set 操作不存在创建，存在则更新；此时如果需要存在不更新的场景，那么可以使用如下这个指令  
  
setnx -> 不存在创建存在不更新

> setnx key value

示例：

```
 1127.0.0.1:6379> get name
 2(nil)
 3127.0.0.1:6379> setnx name liziba        
 4(integer) 1
 5127.0.0.1:6379> get name
 6"liziba"
 7127.0.0.1:6379> setnx name liziba_98        # 已经存在再次设值，失败
 8(integer) 0
 9127.0.0.1:6379> get name
10"liziba"
```

### **2.5 计数**

string (字符串) 也可以用来计数，前提是 value 是一个整数，那么可以对它进行自增的操作。自增的范围必须在 signed long 的区间访问内，\[-9223372036854775808,9223372036854775808\]

incr -> 自增 1

> incr key

示例：

```
1127.0.0.1:6379> set fans 1000
2OK
3127.0.0.1:6379> incr fans # 自增1
4(integer) 1001
```

incrby -> 自定义累加值

> incrby key increment

```
1127.0.0.1:6379> set fans 1000
2OK
3127.0.0.1:6379> incr fans
4(integer) 1001
5127.0.0.1:6379> incrby fans 999
6(integer) 2000
```

测试 value 为整数的自增区间

最大值：

```
1127.0.0.1:6379> set fans 9223372036854775808
2OK
3127.0.0.1:6379> incr fans
4(error) ERR value is not an integer or out of range
```

最小值：

```
1127.0.0.1:6379> set money -9223372036854775808
2OK
3127.0.0.1:6379> incrby money -1
4(error) ERR increment or decrement would overflow
```

## 三、list（列表）

### 1、list (列表) 相关介绍

### **1.1 list (列表) 的内部结构**

Redis 的列表相当于 Java 语言中的 LinkedList，它是一个双向链表数据结构（但是这个结构设计比较巧妙，后面会介绍），支持前后顺序遍历。链表结构插入和删除操作快，时间复杂度 O (1)，查询慢，时间复杂度 O (n)。

![](https://pic4.zhimg.com/80/v2-403b6435558f9901596dff89e0e34897_720w.jpg)

### **1.2 list (列表) 的使用场景**

根据 Redis 双向列表的特性，因此其也被用于异步队列的使用。实际开发中将需要延后处理的任务结构体序列化成字符串，放入 Redis 的队列中，另一个线程从这个列表中获取数据进行后续处理。其流程类似如下的图：

![](https://pic1.zhimg.com/80/v2-26317ca2a140be45f942c5a4901f83c8_720w.jpg)

### 2、list (列表) 的指令

### **2.1 右进左出 — 队列**

队列在结构上是先进先出（FIFO）的数据结构（比如排队购票的顺序），常用于消息队列类似的功能，例如消息排队、异步处理等场景。通过它可以确保元素的访问顺序。  
lpush -> 从左边边添加元素

> lpush key value \[value …\]

rpush -> 从右边添加元素

> rpush key value \[value …\]

llen -> 获取列表的长度

> llen key

lpop -> 从左边弹出元素

> lpop key

```
1127.0.0.1:6379> rpush code java c python    # 向列表中添加元素
 2(integer) 3
 3127.0.0.1:6379> llen code    # 获取列表长度
 4(integer) 3
 5127.0.0.1:6379> lpop code # 弹出最先添加的元素
 6"java"
 7127.0.0.1:6379> lpop code    
 8"c"
 9127.0.0.1:6379> lpop code
10"python"
11127.0.0.1:6379> llen code
12(integer) 0
13127.0.0.1:6379> lpop code
14(nil)
```

### **2.2 右进右出 —— 栈**

栈在结构上是先进后出（FILO）的数据结构（比如弹夹压入子弹，子弹被射击出去的顺序就是栈），这种数据结构一般用来逆序输出。

lpush -> 从左边边添加元素

> lpush key value \[value …\]

rpush -> 从右边添加元素

> rpush key value \[value …\]

rpop -> 从右边弹出元素

> rpop code

```
1127.0.0.1:6379> rpush code java c python
 2(integer) 3
 3127.0.0.1:6379> rpop code            # 弹出最后添加的元素
 4"python"
 5127.0.0.1:6379> rpop code
 6"c"
 7127.0.0.1:6379> rpop code
 8"java"
 9127.0.0.1:6379> rpop code
10(nil)
```

### **2.3 慢操作**

列表 (list) 是个链表数据结构，它的遍历是慢操作，所以涉及到遍历的性能将会遍历区间 range 的增大而增大。注意 list 的索引运行为负数，-1 代表倒数第一个，-2 代表倒数第二个，其它同理。  
lindex -> 遍历获取列表指定索引处的值

> lindex key ind

lrange -> 获取从索引 start 到 stop 处的全部值

> lrange key start stop

ltrim -> 截取索引 start 到 stop 处的全部值，其它将会被删除

> ltrim key start stop

```
1127.0.0.1:6379> rpush code java c python
 2(integer) 3
 3127.0.0.1:6379> lindex code 0        # 获取索引为0的数据
 4"java"
 5127.0.0.1:6379> lindex code 1   # 获取索引为1的数据
 6"c"
 7127.0.0.1:6379> lindex code 2        # 获取索引为2的数据
 8"python"
 9127.0.0.1:6379> lrange code 0 -1    # 获取全部 0 到倒数第一个数据  == 获取全部数据
101) "java"
112) "c"
123) "python"
13127.0.0.1:6379> ltrim code 0 -1    # 截取并保理 0 到 -1 的数据 == 保理全部
14OK
15127.0.0.1:6379> lrange code 0 -1
161) "java"
172) "c"
183) "python"
19127.0.0.1:6379> ltrim code 1 -1    # 截取并保理 1 到 -1 的数据 == 移除了索引为0的数据 java
20OK
21127.0.0.1:6379> lrange code 0 -1
221) "c"
232) "python"
```

### 3、list (列表) 深入理解

Redis 底层存储 list (列表) 不是一个简单的 LinkedList，而是 quicklist ——“快速列表”。关于 quicklist 是什么，下面会简单介绍，具体源码我也还在学习中，后面大家一起探讨。  
quicklist 是多个 ziplist (压缩列表) 组成的双向列表；而这个 ziplist (压缩列表) 又是什么呢？ziplist 指的是一块连续的内存存储空间，Redis 底层对于 list (列表) 的存储，当元素个数少的时候，它会使用一块连续的内存空间来存储，这样可以减少每个元素增加 prev 和 next 指针带来的内存消耗，最重要的是可以减少内存碎片化问题。

### **3.1 常见的链表结构示意图**

每个 node 节点元素，都会持有一个 prev-> 执行前一个 node 节点和 next-> 指向后一个 node 节点的指针（引用），这种结构虽然支持前后顺序遍历，但是也带来了不小的内存开销，如果 node 节点仅仅是一个 int 类型的值，那么可想而知，引用的内存比例将会更大。

![](https://pic1.zhimg.com/80/v2-4e3fa64d9de7ee5b286d6c0143d12ffc_720w.jpg)

### **3.2 ziplist 示意图**

ziplist 是一块连续的内存地址，他们之间无需持有 prev 和 next 指针，能通过地址顺序寻址访问。

![](https://pic2.zhimg.com/80/v2-d6ca1404e926812ed862a8cc9cd13585_720w.jpg)

### **3.3 quicklist 示意图**

quicklist 是由多个 ziplist 组成的双向链表。

![](https://pic2.zhimg.com/80/v2-0eb20c1edcc089aa7e8626f774548efd_720w.jpg)

## 四、hash（字典）

### 1、hash (字典) 相关介绍

### **1.1 hash (字典) 的内部结构**

Redis 的 hash (字典) 相当于 Java 语言中的 HashMap，它是根据散列值分布的无序字典，内部的元素是通过键值对的方式存储。

![](https://pic4.zhimg.com/80/v2-3716f37721d3461438ac7766de435417_720w.jpg)

hash (字典) 的实现与 Java 中的 HashMap（JDK1.7）的结构也是一致的，它的数据结构也是数组 + 链表组成的二维结构，节点元素散列在数组上，如果发生 hash 碰撞则使用链表串联在数组节点上。

![](https://pic4.zhimg.com/80/v2-da6099294f078e8775718b89fe0a150b_720w.jpg)

### **1.2 hash (字典) 扩容**

Redis 中的 hash (字典) 存储的 value 只能是字符串值，此外扩容与 Java 中的 HashMap 也不同。Java 中的 HashMap 在扩容的时候是一次性完成的，而 Redis 考虑到其核心存取是单线程的性能问题，为了追求高性能，因而采取了渐进式 rehash 策略。

渐进式 rehash 指的是并非一次性完成，它是多次完成的，因此需要保理旧的 hash 结构，所以 Redis 中的 hash (字典) 会存在新旧两个 hash 结构，在 rehash 结束后也就是旧 hash 的值全部搬迁到新 hash 之后，新的 hash 在功能上才会完全替代以前的 hash。

![](https://pic4.zhimg.com/80/v2-4bc40f27102eb01389174b586acd883b_720w.jpg)

### **1.3 hash (字典) 的相关使用场景**

hash (字典) 可以用来存储对象的相关信息，一个 hash (字典) 代表一个对象，hash 的一个 key 代表对象的一个属性，key 的值代表属性的值。hash (字典) 结构相比字符串来说，它无需将整个对象进行序列化后进行存储。这样在获取的时候可以进行部分获取。所以相比之下 hash (字典) 具有如下的优缺点：

-   读取可以部分读取，节省网络流量
-   存储消耗的高于单个字符串的存储

### 2 hash (字典) 相关指令

### **2.1 hash (字典) 常用指令**

hset -> hash (字典) 插入值，字典不存在则创建 key 代表字典名称，field 相当于 key，value 是 key 的值

> hset key field value

hmset -> 批量设值

> hmset key field value \[field value …\]

示例：

```
17.0.0.1:6379> hset book java "Thinking in Java"        # 字符串包含空格需要""包裹
2(integer) 1
3127.0.0.1:6379> hset book python "Python code"
4(integer) 1
5127.0.0.1:6379> hset book c "The best of c"
6(integer) 1
7127.0.0.1:6379> hmset book go "concurrency in go" mysql "high-performance MySQL" # 批量设值
8OK
```

hget -> 获取字典中的指定 key 的 value

> hget key field

hgetall -> 获取字典中所有的 key 和 value，换行输出

> hgetall key

示例：

```
1127.0.0.1:6379> hget book java
2"Thinking in Java"
3127.0.0.1:6379> hgetall book
41) "java"
52) "Thinking in Java"
63) "python"
74) "Python code"
85) "c"
96) "The best of c"
```

hlen -> 获取指定字典的 key 的个数

> hlen key

举例：

```
1127.0.0.1:6379> hlen book
2(integer) 5
```

### **2.2 hash (字典) 使用小技巧**

在 string (字符串) 中可以使用 incr 和 incrby 对 value 是整数的字符串进行自加操作，在 hash (字典) 结构中如果单个子 key 是整数也可以进行自加操作。

hincrby -> 增对 hash (字典) 中的某个 key 的整数 value 进行自加操作

> hincrby key field increment

```
1127.0.0.1:6379> hset liziba money 10
2(integer) 1
3127.0.0.1:6379> hincrby liziba money -1
4(integer) 9
5127.0.0.1:6379> hget liziba money
6"9"
```

注意如果不是整数会报错。

```
1127.0.0.1:6379> hset liziba money 10.1
2(integer) 1
3127.0.0.1:6379> hincrby liziba money 1
4(error) ERR hash value is not an integer
```

## 五、set（集合）

### 1、set (集合) 相关介绍

### **1.1 set (集合) 的内部结构**

Redis 的 set (集合) 相当于 Java 语言里的 HashSet，它内部的键值对是无序的、唯一的。它的内部实现了一个所有 value 为 null 的特殊字典。  
集合中的最后一个元素被移除之后，数据结构被自动删除，内存被回收。

![](https://pic3.zhimg.com/80/v2-b4389599f0ecd76bf111c0070cb18efe_720w.jpg)

### **1.2 set (集合) 的使用场景**

set (集合) 由于其特殊去重复的功能，我们可以用来存储活动中中奖的用户的 ID，这样可以保证一个用户不会中奖两次。

### 2、set (集合) 相关指令

sadd -> 添加集合成员，key 值集合名称，member 值集合元素，元素不能重复

> sadd key member \[member …\]

```
1127.0.0.1:6379> sadd name zhangsan
2(integer) 1
3127.0.0.1:6379> sadd name zhangsan        # 不能重复，重复返回0
4(integer) 0
5127.0.0.1:6379> sadd name lisi wangwu liumazi # 支持一次添加多个元素
6(integer) 3
```

smembers -> 查看集合中所有的元素，注意是无序的

> smembers key

```
1127.0.0.1:6379> smembers name    # 无序输出集合中所有的元素
21) "lisi"
32) "wangwu"
43) "liumazi"
54) "zhangsan"
```

sismember -> 查询集合中是否包含某个元素

> sismember key member

```
1127.0.0.1:6379> sismember name lisi  # 包含返回1
2(integer) 1
3127.0.0.1:6379> sismember name tianqi # 不包含返回0
4(integer) 0
```

scard -> 获取集合的长度

> scard key

```
1127.0.0.1:6379> scard name
2(integer) 4
```

spop -> 弹出元素，count 指弹出元素的个数

> spop key \[count\]

```
1127.0.0.1:6379> spop name            # 默认弹出一个
2"wangwu"
3127.0.0.1:6379> spop name 3    
41) "lisi"
52) "zhangsan"
63) "liumazi"
```

## 六、zset（有序集合）

### 1、zset (有序集合) 相关介绍

### **1.1 zset (有序集合) 的内部结构**

zset (有序集合) 是 Redis 中最常问的数据结构。它类似于 Java 语言中的 SortedSet 和 HashMap 的结合体，它一方面通过 set 来保证内部 value 值的唯一性，另一方面通过 value 的 score（权重）来进行排序。这个排序的功能是通过 Skip List（跳跃列表）来实现的。  
zset (有序集合) 的最后一个元素 value 被移除后，数据结构被自动删除，内存被回收。

![](https://pic1.zhimg.com/80/v2-f2b53bc835028b8d2972f143b3e35e50_720w.jpg)

### **1.2 zset (有序集合) 的相关使用场景**

利用 zset 的去重和有序的效果可以由很多使用场景，举两个例子：

-   存储粉丝列表，value 是粉丝的 ID，score 是关注时间戳，这样可以对粉丝关注进行排序
-   存储学生成绩，value 使学生的 ID，score 是学生的成绩，这样可以对学生的成绩排名

### 2、zset (有序集合) 相关指令

### **1、zadd -> 向集合中添加元素，集合不存在则新建，key 代表 zset 集合名称，score 代表元素的权重，member 代表元素**

> zadd key \[NX|XX\] \[CH\] \[INCR\] score member \[score member …\]

```
1127.0.0.1:6379> zadd name 10 zhangsan
2(integer) 1
3127.0.0.1:6379> zadd name 10.1 lisi
4(integer) 1
5127.0.0.1:6379> zadd name 9.9 wangwu
6(integer) 1
```

### **2、zrange -> 按照 score 权重从小到大排序输出集合中的元素，权重相同则按照 value 的字典顺序排序 (\[lexicographical order\])**

超出范围的下标并不会引起错误。 比如说，当 start 的值比有序集的最大下标还要大，或是 start > stop 时， zrange 命令只是简单地返回一个空列表。 另一方面，假如 stop 参数的值比有序集的最大下标还要大，那么 Redis 将 stop 当作最大下标来处理。

可以通过使用 WITHSCORES 选项，来让成员和它的 score 值一并返回，返回列表以 value1,score1, …, valueN,scoreN 的格式表示。 客户端库可能会返回一些更复杂的数据类型，比如数组、元组等。

> zrange key start stop \[WITHSCORES\]

```
 1127.0.0.1:6379> zrange name 0 -1 # 获取所有元素，按照score的升序输出
 21) "wangwu"
 32) "zhangsan"
 43) "lisi"
 5127.0.0.1:6379> zrange name 0 1        # 获取第一个和第二个slot的元素
 61) "wangwu"
 72) "zhangsan"
 8127.0.0.1:6379> zadd name 10 tianqi    # 在上面的基础上添加score为10的元素
 9(integer) 1
10127.0.0.1:6379> zrange name 0 2    # key相等则按照value字典排序输出
111) "wangwu"
122) "tianqi"
133) "zhangsan"
14127.0.0.1:6379> zrange name 0 -1 WITHSCORES # WITHSCORES 输出权重
151) "wangwu"
162) "9.9000000000000004"
173) "tianqi"
184) "10"
195) "zhangsan"
206) "10"
217) "lisi"
228) "10.1"
```

### **3、zrevrange -> 按照 score 权重从大到小输出集合中的元素，权重相同则按照 value 的字典逆序排序**

其中成员的位置按 score 值递减 (从大到小) 来排列。 具有相同 score 值的成员按字典序的逆序 (reverse lexicographical order) 排列。 除了成员按 score 值递减的次序排列这一点外， ZREVRANGE 命令的其他方面和 ZRANGE key start stop \[WITHSCORES\] 命令一样

> zrevrange key start stop \[WITHSCORES\]

```
1127.0.0.1:6379> zrevrange name 0 -1 WITHSCORES
21) "lisi"
32) "10.1"
43) "zhangsan"
54) "10"
65) "tianqi"
76) "10"
87) "wangwu"
98) "9.9000000000000004"
```

### **4、zcard -> 当 key 存在且是有序集类型时，返回有序集的基数。 当 key 不存在时，返回 0**

> zcard key

```
1127.0.0.1:6379> zcard name
2(integer) 4
```

### **5、zscore -> 返回有序集 key 中，成员 member 的 score 值，如果 member 元素不是有序集 key 的成员，或 key 不存在，返回 nil**

> zscore key member z

```
1127.0.0.1:6379> zscore name zhangsan
2"10"
3127.0.0.1:6379> zscore name liziba
4(nil)
```

### **6、zrank -> 返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递增 (从小到大) 顺序排列。**

排名以 0 为底，也就是说，score 值最小的成员排名为 0

> zrank key member

```
1127.0.0.1:6379> zrange name 0 -1
21) "wangwu"
32) "tianqi"
43) "zhangsan"
54) "lisi"
6127.0.0.1:6379> zrank name wangwu
7(integer) 0
```

### **7、zrangebyscore -> 返回有序集 key 中，所有 score 值介于 min 和 max 之间 (包括等于 min 或 max) 的成员。有序集成员按 score 值递增 (从小到大) 次序排列。**

min 和 max 可以是 -inf 和 +inf ，这样一来，你就可以在不知道有序集的最低和最高 score 值的情况下，使用 \[ZRANGEBYSCORE\] 这类命令。

默认情况下，区间的取值使用闭区间，你也可以通过给参数前增加 (符号来使用可选的 \[开区间\] 小于或大于)

> zrangebyscore key min max \[WITHSCORES\] \[LIMIT offset count\]

```
 1127.0.0.1:6379> zrange name 0 -1 WITHSCORES # 输出全部元素
 21) "wangwu"
 32) "9.9000000000000004"
 43) "tianqi"
 54) "10"
 65) "zhangsan"
 76) "10"
 87) "lisi"
 98) "10.1" 
10127.0.0.1:6379> zrangebyscore name 9 10
111) "wangwu"
122) "tianqi"
133) "zhangsan"
14127.0.0.1:6379> zrangebyscore name 9 10 WITHSCORES    # 输出分数
151) "wangwu"
162) "9.9000000000000004"
173) "tianqi"
184) "10"
195) "zhangsan"
206) "10"
21127.0.0.1:6379> zrangebyscore name -inf 10 # -inf 从负无穷开始
221) "wangwu"
232) "tianqi"
243) "zhangsan"
25127.0.0.1:6379> zrangebyscore name -inf +inf    # +inf 直到正无穷
261) "wangwu"
272) "tianqi"
283) "zhangsan"
294) "lisi"
30127.0.0.1:6379> zrangebyscore name (10 11  #  10 < score <=11
311) "lisi"
32127.0.0.1:6379> zrangebyscore name (10 (10.1  # 10 < socre < -11
33(empty list or set)
34127.0.0.1:6379> zrangebyscore name (10 (11 
351) "lisi"
```

### **8、zrem -> 移除有序集 key 中的一个或多个成员，不存在的成员将被忽略**

> zrem key member \[member …\]

```
 1127.0.0.1:6379> zrange name 0 -1
 21) "wangwu"
 32) "tianqi"
 43) "zhangsan"
 54) "lisi"
 6127.0.0.1:6379> zrem name zhangsan # 移除元素
 7(integer) 1
 8127.0.0.1:6379> zrange name 0 -1
 91) "wangwu"
102) "tianqi"
113) "lisi"
```

## 七、Skip List

### 1、简介

跳表全称叫做跳跃表，简称跳表。跳表是一个随机化的数据结构，实质就是一种可以进行二分查找的有序链表。跳表在原有的有序链表上面增加了多级索引，通过索引来实现快速查找。跳表不仅能提高搜索性能，同时也可以提高插入和删除操作的性能。

Skip List (跳跃列表) 这种随机的数据结构，可以看做是一个二叉树的变种，它在性能上与红黑树、AVL 树很相近；但是 Skip List (跳跃列表) 的实现相比前两者要简单很多，目前 Redis 的 zset 实现采用了 Skip List (跳跃列表)（其它还有 LevelDB 等也使用了跳跃列表）。

RBT 红黑树与 Skip List (跳跃列表) 简单对比：  
RBT 红黑树

1.  插入、查询时间复杂度 O (logn)
2.  数据天然有序
3.  实现复杂，设计变色、左旋右旋平衡等操作
4.  需要加锁

Skip List 跳跃列表

1.  插入、查询时间复杂度 O (logn)
2.  数据天然有序
3.  实现简单，链表结构
4.  无需加锁

### 2、Skip List 算法分析

### **2.1 Skip List 论文**

这里贴出 Skip List 的论文，需要详细研究的请看论文，下文部分公式、代码、图片出自该论文。  
Skip Lists: A Probabilistic Alternative to Balanced Trees

> https://www. cl.cam.ac.uk/teaching/2005/Algorithms/skiplists.pdf

### **2.2 Skip List 动态图**

先通过一张动图来了解 Skip List 的插入节点元素的流程，此图来自维基百科。

![](https://pic2.zhimg.com/80/v2-8bac4b692e6215fb228c36d97699a0a9_720w.jpg)

### **2.3 Skip List 算法性能分析**

**2.3.1 计算随机层数算法**

首先分析的是执行插入操作时计算随机数的过程，这个过程会涉及层数的计算，所以十分重要。对于节点他有如下特性：

-   节点都有第一层的指针
-   节点有第 i 层指针，那么第 i+1 层出现的概率为 p
-   节点有最大层数限制，MaxLevel

计算随机层数的伪代码：

**论文中的示例**

![](https://pic3.zhimg.com/80/v2-1fc4009ba6cd598d6acca0d1247a96ba_720w.jpg)

Java 版本

```
1public int randomLevel(){
2    int level = 1;
3    // random()返回一个[0...1)的随机数
4    while (random() < p && level < MaxLevel){ 
5        level += 1;
6    }
7    return level;
8}
```

代码中包含两个变量 P 和 MaxLevel，在 Redis 中这两个参数的值分别是：

**2.3.2 节点包含的平均指针数目**

Skip List 属于空间换时间的数据结构，这里的空间指的就是每个节点包含的指针数目，这一部分是额外的内内存开销，可以用来度量空间复杂度。random () 是个随机数，因此产生越高的节点层数，概率越低（Redis 标准源码中的晋升率数据 1/4，相对来说 Skip List 的结构是比较扁平的，层高相对较低）。其定量分析如下：

-   level = 1 概率为 1-p
-   level >=2 概率为 p
-   level = 2 概率为 p (1-p)
-   level >= 3 概率为 p^2
-   level = 3 概率为 p^2 (1-p)
-   level >=4 概率为 p^3
-   level = 4 概率为 p^3 (1-p)
-   ……

得出节点的平均层数（节点包含的平均指针数目）：

![](https://pic3.zhimg.com/80/v2-4ff49ca050406f3100b17747c5c97516_720w.jpg)

所以 Redis 中 p=1/4 计算的平均指针数目为 1.33  


**2.3.3 时间复杂度计算**

以下推算来自论文内容

假设 p=1/2，在以 p=1/2 生成的 16 个元素的跳过列表中，我们可能碰巧具有 9 个元素，1 级 3 个元素，3 个元素 3 级元素和 1 个元素 14 级（这不太可能，但可能会发生）。我们该怎么处理这种情况？如果我们使用标准算法并在第 14 级开始我们的搜索，我们将会做很多无用的工作。那么我们应该从哪里开始搜索？此时我们假设 SkipList 中有 n 个元素，第 L 层级元素个数的期望是 1/p 个；每个元素出现在 L 层的概率是 p^(L-1), 那么第 L 层级元素个数的期望是 n \* (p^L-1)；得到 1 /p =n \* (p^L-1)

```
11 / p = n * (p^L-1)
2n = (1/p)^L
3L = log(1/p)^n
```

所以我们应该选择 MaxLevel = log (1/p)^n  
定义：MaxLevel = L (n) = log (1/p)^n

推算 Skip List 的时间复杂度，可以用逆向思维，从层数为 i 的节点 x 出发，返回起点的方式来回溯时间复杂度，节点 x 点存在两种情况：

-   节点 x 存在 (i+1) 层指针，那么向上爬一级，概率为 p，对应下图 situation c.
-   节点 x 不存在 (i+1) 层指针，那么向左爬一级，概率为 1-p，对应下图 situation b.

![](https://pic3.zhimg.com/80/v2-5fa007669124dbafd83add9788744196_720w.jpg)

设 C (k) = 在无限列表中向上攀升 k 个 level 的搜索路径的预期成本（即长度）那么推演如下：

```
1C(0)=0
2C(k)=(1-p)×(情况b的查找长度) + p×(情况c的查找长度)
3C(k)=(1-p)(C(k)+1) + p(C(k-1)+1)
4C(k)=1/p+C(k-1)
5C(k)=k/p
```

上面推演的结果可知，爬升 k 个 level 的预期长度为 k/p，爬升一个 level 的长度为 1/p。

由于 MaxLevel = L (n)， C (k) = k /p，因此期望值为：(L (n) – 1) /p；将 L (n) = log (1/p)^n 代入可得：(log (1/p)^n - 1) /p；将 p = 1 / 2 代入可得：2 \* log2^n - 2，即 O (logn) 的时间复杂度。

### 3、Skip List 特性及其实现

### **2.1 Skip List 特性**

Skip List 跳跃列表通常具有如下这些特性

1.  Skip List 包含多个层，每层称为一个 level，level 从 0 开始递增
2.  Skip List 0 层，也就是最底层，应该包含所有的元素
3.  每一个 level / 层都是一个有序的列表
4.  level 小的层包含 level 大的层的元素，也就是说元素 A 在 X 层出现，那么 想 X>Z>=0 的 level / 层都应该包含元素 A
5.  每个节点元素由节点 key、节点 value 和指向当前节点所在 level 的指针数组组成

### **2.2 Skip List 查询**

假设初始 Skip List 跳跃列表中已经存在这些元素，他们分布的结构如下所示：

![](https://pic4.zhimg.com/80/v2-59a9d9868be1cb16f4c03f66acd41cf7_720w.jpg)

此时查询节点 88，它的查询路线如下所示：

![](https://pic1.zhimg.com/80/v2-55587717f4733948c6f2d66eb13dba04_720w.jpg)

1.  从 Skip List 跳跃列表最顶层 level3 开始，往后查询到 10 < 88 && 后续节点值为 null && 存在下层 level2
2.  level2 10 往后遍历，27 < 88 && 后续节点值为 null && 存在下层 level1
3.  level1 27 往后遍历，88 = 88，查询命中

### **2.3 Skip List 插入**

Skip List 的初始结构与 2.3 中的初始结构一致，此时假设插入的新节点元素值为 90，插入路线如下所示：

1.  查询插入位置，与 Skip List 查询方式一致，这里需要查询的是第一个比 90 大的节点位置，插入在这个节点的前面， 88 < 90 < 100
2.  构造一个新的节点 Node (90)，为插入的节点 Node (90) 计算一个随机 level，这里假设计算的是 1，这个 level 时随机计算的，可能时 1、2、3、4… 均有可能，level 越大的可能越小，主要看随机因子 x ，层数的概率大致计算为 (1/x)^level ，如果 level 大于当前的最大 level3，需要新增 head 和 tail 节点
3.  节点构造完毕后，需要将其插入列表中，插入十分简单步骤 -> Node (88).next = Node (90); Node (90).prev = Node (80); Node (90).next = Node (100); Node (100).prev = Node (90);

![](https://pic4.zhimg.com/80/v2-aadb77974a8e0e50f4aa4da21dee018b_720w.jpg)

### **2.4 Skip List 删除**

删除的流程就是查询到节点，然后删除，重新将删除节点左右两边的节点以链表的形式组合起来即可，这里不再画图

### 4、手写实现一个简单 Skip List

实现一个 Skip List 比较简单，主要分为两个步骤：

1.  定义 Skip List 的节点 Node，节点之间以链表的形式存储，因此节点持有相邻节点的指针，其中 prev 与 next 是同一 level 的前后节点的指针，down 与 up 是同一节点的多个 level 的上下节点的指针
2.  定义 Skip List 的实现类，包含节点的插入、删除、查询，其中查询操作分为升序查询和降序查询（往后和往前查询），这里实现的 Skip List 默认节点之间的元素是升序链表

### **3.1 定义 Node 节点**

Node 节点类主要包括如下重要属性：

1.  score -> 节点的权重，这个与 Redis 中的 score 相同，用来节点元素的排序作用
2.  value -> 节点存储的真实数据，只能存储 String 类型的数据
3.  prev -> 当前节点的前驱节点，同一 level
4.  next -> 当前节点的后继节点，同一 level
5.  down -> 当前节点的下层节点，同一节点的不同 level
6.  up -> 当前节点的上层节点，同一节点的不同 level

```
 1package com.liziba.skiplist;
 2
 3/**
 4 * <p>
 5 *      跳表节点元素
 6 * </p>
 7 *
 8 * @Author: Liziba
 9 * @Date: 2021/7/5 21:01
10 */
11public class Node {
12
13    /** 节点的分数值，根据分数值来排序 */
14    public Double score;
15    /** 节点存储的真实数据 */
16    public String value;
17    /** 当前节点的 前、后、下、上节点的引用 */
18    public Node prev, next, down, up;
19
20    public Node(Double score) {
21        this.score = score;
22        prev = next = down = up = null;
23    }
24
25    public Node(Double score, String value) {
26        this.score = score;
27        this.value = value;
28    }
29}
```

### **3.2 SkipList 节点元素的操作类**

SkipList 主要包括如下重要属性：

1.  head -> SkipList 中的头节点的最上层头节点（level 最大的层的头节点），这个节点不存储元素，是为了构建列表和查询时做查询起始位置的，具体的结构请看 2.3 中的结构
2.  tail -> SkipList 中的尾节点的最上层尾节点（level 最大的层的尾节点），这个节点也不存储元素，是查询某一个 level 的终止标志
3.  level -> 总层数
4.  size -> Skip List 中节点元素的个数
5.  random -> 用于随机计算节点 level，如果 random.nextDouble () < 1/2 则需要增加当前节点的 level，如果当前节点增加的 level 超过了总的 level 则需要增加 head 和 tail (总 level)

```
  1package com.liziba.skiplist;
  2
  3import java.util.Random;
  4
  5/**
  6 * <p>
  7 *      跳表实现
  8 * </p>
  9 *
 10 * @Author: Liziba
 11 */
 12public class SkipList {
 13
 14    /** 最上层头节点 */
 15    public Node head;
 16    /** 最上层尾节点 */
 17    public Node tail;
 18    /** 总层数 */
 19    public int level;
 20    /** 元素个数 */
 21    public int size;
 22    public Random random;
 23
 24    public SkipList() {
 25        level = size = 0;
 26        head = new Node(null);
 27        tail = new Node(null);
 28        head.next = tail;
 29        tail.prev = head;
 30    }
 31
 32    /**
 33     * 查询插入节点的前驱节点位置
 34     *
 35     * @param score
 36     * @return
 37     */
 38    public Node fidePervNode(Double score) {
 39        Node p = head;
 40        for(;;) {
 41            // 当前层(level)往后遍历，比较score，如果小于当前值，则往后遍历
 42            while (p.next.value == null && p.prev.score <= score)
 43                p = p.next;
 44            // 遍历最右节点的下一层(level)
 45            if (p.down != null)
 46                p = p.down;
 47            else
 48                break;
 49        }
 50        return p;
 51    }
 52
 53    /**
 54     * 插入节点，插入位置为fidePervNode(Double score)前面
 55     *
 56     * @param score
 57     * @param value
 58     */
 59    public void insert(Double score, String value) {
 60
 61        // 当前节点的前置节点
 62        Node preNode = fidePervNode(score);
 63        // 当前新插入的节点
 64        Node curNode = new Node(score, value);
 65        // 分数和值均相等则直接返回
 66        if (curNode.value != null && preNode.value != null && preNode.value.equals(curNode.value)
 67                  && curNode.score.equals(preNode.score)) {
 68            return;
 69        }
 70
 71        preNode.next = curNode;
 72        preNode.next.prev = curNode;
 73        curNode.next = preNode.next;
 74        curNode.prev = preNode;
 75
 76        int curLevel = 0;
 77        while (random.nextDouble() < 1/2) {
 78            // 插入节点层数(level)大于等于层数(level)，则新增一层(level)
 79            if (curLevel >= level) {
 80                Node newHead = new Node(null);
 81                Node newTail = new Node(null);
 82                newHead.next = newTail;
 83                newHead.down = head;
 84                newTail.prev = newHead;
 85                newTail.down = tail;
 86                head.up = newHead;
 87                tail.up = newTail;
 88                // 头尾节点指针修改为新的，确保head、tail指针一直是最上层的头尾节点
 89                head = newHead;
 90                tail = newTail;
 91                ++level;
 92            }
 93
 94            while (preNode.up == null)
 95                preNode = preNode.prev;
 96
 97            preNode = preNode.up;
 98
 99            Node copy = new Node(null);
100            copy.prev = preNode;
101            copy.next = preNode.next;
102            preNode.next.prev = copy;
103            preNode.next = copy;
104            copy.down = curNode;
105            curNode.up = copy;
106            curNode = copy;
107
108            ++curLevel;
109        }
110        ++size;
111    }
112
113    /**
114     * 查询指定score的节点元素
115     * @param score
116     * @return
117     */
118    public Node search(double score) {
119        Node p = head;
120        for (;;) {
121            while (p.next.score != null && p.next.score <= score)
122                p = p.next;
123            if (p.down != null)
124                p = p.down;
125            else // 遍历到最底层
126                if (p.score.equals(score))
127                    return p;
128                return null;
129        }
130    }
131
132    /**
133     * 升序输出Skip List中的元素 （默认升序存储，因此从列表head往tail遍历）
134     */
135    public void dumpAllAsc() {
136        Node p = head;
137        while (p.down != null) {
138            p = p.down;
139        }
140        while (p.next.score != null) {
141            System.out.println(p.next.score + "-->" + p.next.value);
142            p = p.next;
143        }
144    }
145
146    /**
147     * 降序输出Skip List中的元素
148     */
149    public void dumpAllDesc() {
150        Node p = tail;
151        while (p.down != null) {
152            p = p.down;
153        }
154        while (p.prev.score != null) {
155            System.out.println(p.prev.score + "-->" + p.prev.value);
156            p = p.prev;
157        }
158    }
159
160
161    /**
162     * 删除Skip List中的节点元素
163     * @param score
164     */
165    public void delete(Double score) {
166        Node p = search(score);
167        while (p != null) {
168            p.prev.next = p.next;
169            p.next.prev = p.prev;
170            p = p.up;
171        }
172    }
173}
```

[**点击关注，第一时间了解华为云新鲜技术～**](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fbbs.huaweicloud.com%2Fblogs%3Futm_source%3Doschina%26utm_medium%3Dbbs-ex%26utm_campaign%3Dother%26utm_content%3Dcontent)