# Redis 与 Spring

## 概述

Redis（Remote Dictionary Server）是一个使用 **C 语言**编写的，开源的（BSD许可）高性能**非关系型（NoSQL）**的**键值对（Key-Value）**数据库。

Redis是**单线程**的，与传统数据库不同的是Redis的数据是存在**内存**中的，所以读写速度非常快，因此被广泛应用于**缓存**方向。每秒可以处理超过10万次读写操作，是已知**性能最快**的Key-Value DB。另外，Redis也经常用来做分布式锁。除此之外，Redis支持事务 、持久化、LUA脚本、LRU驱动事件、多种集群方案。



优点：

- 读写性能优异，能读的速度是110000次/s，写的速度是81000次/s。
- 支持数据持久化，支持AOF和RDB两种持久化方式。
- 支持事务，所有操作都是原子性的，同时还支持对几个操作合并后的原子性执行。
- 数据结构丰富，值Value支持5种数据结构。
- 支持主从复制，主机会自动将数据同步到从机，可以进行读写分离。

缺点：

- 数据库容量受到物理内存的限制，不能用作海量数据的高性能读写，因此其适合的场景主要局限在较小数据量的高性能操作和运算上，例如缓存。
- 不具备自动容错和恢复功能，主机从机的宕机都会导致前端部分读写请求失败，需要等待机器重启或者手动切换前端的IP才能恢复。
- 主机宕机，宕机前有部分数据未能及时同步到从机，切换IP后还会引入数据不一致的问题，降低了系统的可用性。
- 较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂。为避免这一问题，运维人员在系统上线时必须确保有足够的空间，这对资源造成了很大的浪费。



键（Key）的类型：字符串 String

值（Value）的类型：

| 数据类型 | 可以存储的值           | 操作                                                         | 应用场景                                                     |
| -------- | ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| String   | 字符串、整数或者浮点数 | 对整个字符串或者字符串的其中一部分执行操作 对整数和浮点数执行自增或者自减操作 | 做简单的键值对缓存                                           |
| List     | 列表                   | 从两端压入或者弹出元素 对单个或者多个元素进行修剪， 只保留一个范围内的元素 | 存储一些列表型的数据结构，类似粉丝列表、文章的评论列表之类的数据 |
| Set      | 无序集合               | 添加、获取、移除单个元素 检查一个元素是否存在于集合中 计算交集、并集、差集 从集合里面随机获取元素 | 交集、并集、差集的操作，比如交集，可以把两个人的粉丝列表整一个交集 |
| Hash     | 包含键值对的无序散列表 | 添加、获取、移除单个键值对 获取所有键值对 检查某个键是否存在 | 结构化的数据，比如一个对象                                   |
| ZSst     | 有序集合               | 添加、获取、删除元素 根据分值范围或者成员来获取元素 计算一个键的排名 | 去重但可以排序，如获取排名前几名的用户                       |

一个**字符串**类型的值能存储最大容量是**512M**。

Redis使用的**回收**算法：

- **LRU**算法（Least Recently Used，最近最久未使用）
- **LFU**算法（Least Frequently Used，最近最少使用），Redis 4.0 之后

## 应用场景

### 异步队列

### 延时队列

## 事务和管道

### 事务

事务是一个单独的**隔离**操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。

事务是一个**原子**操作：事务中的命令要么全部被执行，要么全部都不执行。

Redis事务的本质是通过`MULTI`、`EXEC`、`WATCH`等一组命令的集合。事务支持一次执行多个命令，一个事务中所有命令都会被序列化。在事务执行过程，会按照顺序串行化执行队列中的命令，其他客户端提交的命令请求不会插入到事务执行命令序列中。

总结，Redis事务就是**一次性、顺序性、排他性**的执行一个队列中的一系列命令。

Redis事务的3个阶段：

1. 事务开始`MULTI`
2. 命令入队
3. 事务执行`EXEC`

### 管道

## 持久化

Redis是**内存数据库**

### RDB (Redis Database)

默认方式，不需要进行配置，默认使用这种配置。在一定的时间间隔中，检测key的变化,然后持久化数据。将当前数据状态进行保存，**快照**形式，存储结果，存储格式简单，关注点在数据。

1. RDB相关配置

   - 设置RDB文件名称：`dbfilename dump.rdb`一般命名为`dump-端口号.rdb`
   - 设置RDB文件存放位置: dir
   - 是否压缩数据: `rdbcompression yes`通常默认开启，如果设置不开启，可以节约CPU时间，但是文件会变大（巨大）
   - 是否校验数据: `rdbchecksum yes`通常默认开启，如果设置不开启，可以节约时间,但是存在数据损坏的风险

2. RDB自动保存 : 在配置文件中编辑 save time changes

   根据业务设置频率,过高或者过低都会产生问题

   - // 900 sec 之后 至少1个key改变,持久化一次
     save 900 1
   - // 300 sec 之后, 至少10个key改变,持久化一次
     save 300 10
   - // 60 sec之后, 至少 10000个key改变,持久化一次
     save 60 10000

3. 使用save指令保存
   save指令的执行会**阻塞**当前Redis服务器，知道当前save指令完成，可能会造成长时间阻塞，线上环境**不建议**使用

4. 后台执行: bgsave
   发送指令→→调用fork函数 生成**子进程**→→创建rdb文件→→返回消息
   Redis 内相关保存操作都使用bgsave

5. RDB优缺点

   - RDB优点
     - RDB是紧凑压缩的二进制文件，存储效率高
     - RDB内部存储的是redis在某个时间点的数据快照，非常适合用于**全场景**备份，全场景复制
     - RDB**恢复**数据速度比AOF**快很多**
     - 应用: 服务器每X小时执行bgsave, 并将RDB文件拷贝到远程机器,用于**灾难恢复**
   - RDB缺点
     - **无法做到实时**持久化, 存在丢失数据的可能性
     - bgsave需要fork子进程, **牺牲性能**
     - Redis 不同版本RDB文件格式不统一,可能出现各版本之间数据格式**不兼容**的情况

### AOF (Append Only File)

日志记录,可以记录每一条命令的操作,可以每一次命令操作后来持久化数据. 将数据操作过程进行保存,**日志形式**,存储操作过程,存储过程复杂,关注点在**过程**. 目前是Redis持久化的主流方式

1. AOF写数据过程

2. 编辑配置文件

   - appendonly no →→appendonly yes 开启AOF
   - appendfsync always: 每次写入操作均同步到AOF文件中，数据零误差，性能较低
   - appendfsync everysec: 每秒同步一次，性能较高, 但系统宕机时会丢失1秒内的数据
   - appendfsync no: 由系统控制
   - appendfilename filename: AOF文件名, 建议为appendonly-端口号.aof

3. AOF重写

   随着命令不断写入AOF, 文件会越来越大. 为了解决这个问题,Redis引入了AOF重写机制压缩文件体积. AOF文件重写是将Redis进程内的数据转化为写命令同步到新AOF文件的过程. 简单说, 就是对同一个key若干命令执行结果转化为**最终结果**对应的指令进行记录.

   - AOF重写规则

     进程内已经超时的数据不再写入文件

     忽略无效指令,重写时使进程内数据直接生成

     对同一数据的多条写命令**合并**为一条命令. 为防止缓冲区溢出, 对list, set, hash, zset等类型,每条指令最多写入64个元素

     - AOF重写方式
       手动: bgrewriteaof
             工作方式: 发送指令→→调用fork函数 生成子进程→→重写AOF文件→→返回消息
       自动: auto-aof-rewrite-min-size size
             auto-aof-rewrite-percentage percentage

### RDB 和 AOF 区别

|    持久化    |    RDB     |     AOF      |
| :----------: | :--------: | :----------: |
| 占用存储空间 | 小(数据级) |  大(指令级)  |
|   存储速度   |     慢     |      快      |
|   恢复速度   |     快     |      慢      |
|  数据安全性  | 会丢失数据 | 根据策略决定 |
|   资源消耗   |     高     |      低      |
|  启动优先级  |     低     |      高      |

- 对数据非常敏感,建议使用AOF,策略选择everysec(既能保持较好的处理性能,又能兼顾数据安全)

- 数据呈现阶段有效性,建议使用RDB(手工维护可以做到阶段内无丢失,且恢复速度快)

- 综合对比:

  - RDB与AOF的选择实际上是一种权衡
  - 如不能承受分钟以内的数据丢失,对业务数据非常敏感,选AOF
  - 如追求大数据集的恢复速度,选RDB
  - 灾难恢复选RDB
  - 双保险策略,同时开启RDB与AOF, Redis优先使用AOF来恢复数据,降低数据丢失量

  使用AOF的较多。

## Spring 连接 Redis

### 连接配置类

遵循NoSQL数据库的配置流程：
1. 工厂Bean
2. 通过工厂Bean生成模板Bean

```java
@Configuration
public class RedisConfig {
    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        // 不传入config时默认为localhost:6379，无密码
        return new JedisConnectionFactory(new RedisStandaloneConfiguration("localhost", 6379));
    }

    @Bean
    public RedisTemplate redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        StringRedisSerializer serializer = new StringRedisSerializer();
        template.setDefaultSerializer(serializer); // 配置默认的序列化器
        template.setKeySerializer(serializer); // 配置key的序列化器
        template.setValueSerializer(serializer); // 配置value的序列化器
        return template;
    }
}
```

### 使用连接的 Redis

```java
ApplicationContext context;
// 用配置类的方式
context = new AnnotationConfigApplicationContext(CachingConfig.class);
// 用xml的方式
// context = new ClassPathXmlApplicationContext("redis-config.xml");
RedisTemplate template = context.getBean(RedisTemplate.class);
Set<ZSetOperations.TypedTuple<String>> set = template.opsForZSet().rangeWithScores("myZSet", 0, -1);
for (ZSetOperations.TypedTuple<String> tuple : set) {
	System.out.println(tuple.getScore().intValue() + " : " + tuple.getValue());
}
// 绑定操作
BoundListOperations<?, ?> myList = template.boundListOps("myList");
System.out.println(myList.range(0, 2));
```

## 缓存

### 设置缓存管理器

1. 配置对象
2. 通过上文中的工厂Bean和配置对象生成管理器Bean

```java
@Configuration
@EnableCaching
@ComponentScan
public class CachingConfig {
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
        // 创建一个缓存配置，此处必须采用链式编程，涉及的每个方法都会产生新的对象
        RedisCacheConfiguration config = RedisCacheConfiguration
                .defaultCacheConfig() // 从默认配置的基础上修改
                .serializeKeysWith(RedisSerializationContext.SerializationPair
                        .fromSerializer(new StringRedisSerializer())) // 配置key的序列化器
                .serializeValuesWith(RedisSerializationContext.SerializationPair
                        .fromSerializer(new GenericJackson2JsonRedisSerializer())) // 配置value的序列化器
                .prefixCacheNameWith("defaultPrefix::") // 设置一个默认的前缀
                .disableCachingNullValues() // 不缓存空值
                .entryTtl(Duration.ofMinutes(3)); // 3分钟过期
        // 所有config，cacheName作为键，config作为值
        Map<String, RedisCacheConfiguration> configMap = new HashMap<>();
        configMap.put("cache1", config.prefixCacheNameWith("cache1Prefix")); // 各有各的前缀
        configMap.put("cache2", config.prefixCacheNameWith("cache2Prefix")); // 各有各的前缀
        // 缓存管理器
        RedisCacheManager manager = RedisCacheManager
                .builder(redisConnectionFactory)
                .cacheDefaults(config) // 与configMap中的所有key都不匹配时，采用的默认config
                .initialCacheNames(configMap.keySet()) // 设定所有可供匹配的cacheName，对应@CacheConfig中的cacheName，@Cacheable和@CachePut中的value
                .withInitialCacheConfigurations(configMap) // 设定所有config
                .build();
        return manager;
    }
}
```

### 实体类示例

```java
public class Spittle {
    public int id;
    public String name;

    // 必须要有默认构造器
    public Spittle() {
    }

    public Spittle(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public String toString() {
        return "Spittle{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}
```


### 注释缓存的方法

```java
/*
 * 若未配置@CacheConfig的cacheName, 则@Cacheable一定要配置value
 * 若@CacheConfig的cacheName与@Cacheable的value都配置了，则后者的value会覆盖前者的cacheName
 */
@Component
@CacheConfig(cacheNames = "cache", cacheManager = "cacheManager")
public class RedisCacheRepository {
    // 先看看缓存里有没有，没有再执行方法，并把返回值存入缓存，用于获取缓存
    @Cacheable(value = "cache1", key = "#root.method")
    public String getMessage(String message) {
        return "message";
    }

    // 不管缓存里有没有，直接执行方法，再把返回值存入缓存，用于更新缓存
    @CachePut(value = "cache2", key = "#root.methodName")
    public String setMessage(String msg) {
        return "msg";
    }

    // 通过SpEL表达式算出要缓存的对象在redis中的key
    @Cacheable(key = "{#root.targetClass.simpleName,#id}")
    public Spittle findOne(int id) {
        return new Spittle(123, "No cache!");
    }

    @CachePut(key = "{#root.targetClass.simpleName,#spittle.id}")
    public Spittle save(Spittle spittle) {
        return spittle;
    }
}
```

### 使用缓存

```java
// 上文中配置过的context
RedisCacheRepository repository = context.getBean(RedisCacheRepository.class);
repository.getMessage("message");
repository.setMessage("msg");
Spittle spittle = new Spittle(9527, "This is a spittle.")
repository.save(spittle);
repository.findOne(9527);
```

### 缓存结果

```shell
# 匹配包含"cache"字符串的key
127.0.0.1:6379> keys *cache*
 1) "defaultPrefix::cache::RedisCacheRepository,9527"
 2) "cache1Prefixcache1::public java.lang.String com.springdemo.RedisCacheRepository.getMessage(java.lang.String)"
 3) "cache2Prefixcache2::setMessage"

127.0.0.1:6379> get defaultPrefix::cache::RedisCacheRepository,9527
"{\"@class\":\"com.springdemo.Spittle\",\"id\":9527,\"name\":\"This is a spittle.\"}"

127.0.0.1:6379> get cache2Prefixcache2::setMessage
"\"msg\""

# key之间不能有空格，命令执行失败
127.0.0.1:6379> get cache1Prefixcache1::public java.lang.String com.springdemo.RedisCacheRepository.getMessage(java.lang.String)
(error) ERR wrong number of arguments for 'get' command

# 如果key有空格需要用引号包起来
127.0.0.1:6379> get "cache1Prefixcache1::public java.lang.String com.springdemo.RedisCacheRepository.getMessage(java.lang.String)"
"\"message\""
```

## 缓存应用场景

## 缓存会遇到的问题

## SpEL 表达式

| 名字          | 位置               | 描述                                                         | 示例                 |
| ------------- | ------------------ | ------------------------------------------------------------ | -------------------- |
| methodName    | root object        | 当前被调用的方法名                                           | #root.methodName     |
| method        | root object        | 当前被调用的方法                                             | #root.method .name   |
| target        | root object        | 当前被调用的目标对象                                         | #root.target         |
| targetClass   | root object        | 当前被调用的目标对象类                                       | #root.targetClass    |
| args          | root object        | 当前被调用的方法的参数列表                                   | #root.args[0]        |
| caches        | root object        | 当前方法调用使用的缓存列表                                   | #root.caches[0].name |
| argument name | evaluation context | 方法参数的名字，可以直接#参数名，也可以使用#p0或#a0的形式，0代表参数的索引 | #id、#a0、#p0        |
| result        | evaluation context | 方法执行后的返回值                                           |                      |

## 集群方案

### 哨兵模式 Sentinel

### 服务端路由查询 Redis Cluster

Redis集群没有使用一致性hash,而是引入了**哈希槽**的概念，Redis集群有**16384**个哈希槽，每个key通过CRC16校验后对16384取模来决定放置哪个槽，集群的每个节点负责一部分hash槽。Redis集群目前无法做数据库选择，默认在0数据库。

16384=0x4000

### 基于客户端分配 Redis Sharding

### 基于代理服务器分片 Proxy Sharding

### 主从架构 Master And Slave

## 分布式锁 RedLock

[基于 Redis 的分布式锁 Redlock](https://zhuanlan.zhihu.com/p/40915772)

特点：

- 安全特性：互斥访问，即永远只有一个 client 能拿到锁

- 避免死锁：最终 client 都可能拿到锁，不会出现死锁的情况，即使原本锁住某资源的 client crash 了或者出现了网络分区

- 容错性：只要大部分 Redis 节点存活就可以正常提供服务

实现：

```shell
SET resource_name my_random_value NX PX 30000
```

主要依靠上述命令，该命令仅当名为`resource_name`的 Key 不存在时（`NX`保证）赋值，并且设置过期时间 30000ms （`PX`保证），值`my_random_value`必须是所有 client 和所有锁请求发生期间唯一的，释放锁的逻辑是：

```shell
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

上述实现，通过`redis.call("get",KEYS[1]) == ARGV[1]`先验证该锁`KEYS[1]`的值还是不是client1设置的`ARGV[1]`，可以避免释放另一个client创建的锁，如果只有`del`命令的话，那么如果client1拿到lock1之后因为某些操作阻塞了很长时间，此时Redis端lock1已经过期了并且已经被重新分配给client2，那么client1此时再去释放这把锁就会造成client2原本获取到的锁被client1无故释放了，但现在为每个client分配一个unique的string值可以避免这个问题。至于如何去生成这个unique string，方法很多随意选择一种就行了。

