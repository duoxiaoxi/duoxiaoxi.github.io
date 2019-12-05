# Redis



我们学习Java已经经历了很多技术 那么我们学过的技术可以如何分类呢?

* `解决功能性问题`: Java, JSP, Servlet, Tomcat, Html, Linux, Jdbc, MySQL
* `解决拓展性问题`: Struts2, Hibernate, Spring, SpringMVC, MyBatis
* `解决性能问题`: NoSQL, Java线程, Nginx, RabbitMQ, Hadoop



NoSQL数据库适合的使用场景:

* 高并发读写
* 海量数据读写
* 要求数据高可拓展性

NoSQL数据库不适合的使用场景:

* 需要事务支持
* 处理复杂的关系



常见的NoSQL数据库

* `Memcached`: key-value型数据库, 不支持持久化
* `Redis`: key-value型数据库, 支持持久化
* `mongoDB`: 文档型数据库

* `HBase`: Hadoop项目中的数据库

![1553847138292](1553847138292.png)





## 安装

https://redis.io/





## 命令

### 思想

Redis的本质就Java语言来说就是`Map<String, ?>` 我们平常说的Redis支持的5中数据类型 指的是Map中值的数据类型, 他们都通过键来唯一的确定了...

* Redis中的数据类型指的是**Map中的值的数据类型**
* 不同的数据类型的对象的**查看, 添加,修改的方式不同**
  * 例如字符串通过 `get & set` 查看 和 添加, 但是列表通过`lpush & lrange` 来...



### 基本命令

* `redis-server ../conf/redis.conf`: 启动redis服务器 指定配置文件 默认运行在==6379==端口
* `redis-cli -h host - p port -a password`: 与服务器建立交互
  * `redis-cli`: 可以不指定任何参数 默认去访问==127.0.0.0:6379==
* `redis-cli shutdown`: 通知服务器关闭
* `ping`: 测试连通性
* `config get {pattern}`: 查看数据库的配置信息
* `select {0-15}`: 选中一个数据库 一共16个
* `keys {pattern}`: 用于查看数据库中所有的键
  * `*`: 代表任意字符任意个
  * `?`: 代表一个任意字符
* `exists {key}`: 判断某个键值对是否存在
* `type {key}`: 查看某个键值对的类型
* `del {key}`: 删除某个键值对
* `rename {key} {newname}`: 修改键的名称
  * `renameNX {key} {newname}`: 修改键的名称 当键值对存在的情况下
* `move {key} {dbid}`: 移动某个键值对到其他数据库
* `expire {key} {second}`: 为指定的键值对设定过期时间
  * `pexpire {key} {millisecond}`: 设定毫秒为单位的过气时间
* `ttl {key}`: 查看某个键值对的过气时间
  * `pttl {key}`: 返回毫秒的过期时间
  * `return -1`: 代表不会过期 永久有效
  * `return -2`: 代表键不存在
  * ==tip: redis2.8 版本以前 无论是永久有效 还是键不存在 都返回-1==
* `persist {key}`: 移除某个key的过期时间, 也就是永不过期
* `randomkey`: 随机返回一个key
* `dbsize`: 查看当前数据库拥有多少个键值对
* `flushdb`: 清空当前数据库
* `flushall`: 清空所有数据库



### String命令

* `set {key} {str}`: 添加一个字符串类型的键值对 **如果键已经存在 会覆盖值**
  * `setNX {key} {str}`: 如果不存在才去添加 **如果键已经存在 不会覆盖值**
  * `setEX {key} {second} {str}`: 添加字符串 并且  设置好过期时间
  * `PsetEX {key} {second} {str}`: 和上面的命令相同 但是按照毫秒来
  * `mset {key1} {str1} {key2} {str2}...`: 批量添加字符串
  * `msetNX {key1} {str1}...`: 批量添加字符串 只在指定的键不存在的时候
* `get {key}`: 查看字符串的内容
  * `mget {k1} {k2}...`: 一次性获取多个字符串的内容
* `getset {key} {newvalue}`: 设置新值 返回旧值
* `getrange {key} m n`: 获取子串`[m, n]`从`1`开始 支持负数索引
* `strrange {key} {offset} {rep}`: 从offset开始 使用rep替换
* `append {key} {apd}`: 在指定的字符串末尾追加`apd`
* `strlen {key}`: 获取指定字符串的长度
* `incr {key}`: 将指定的数字 自增1
  * `incrby {key} n`: 将指定的数字 自增n
  * `incrbyFloat {key} f`: 将指定的数字 自增float
* `decr {key}`: 将制定的数字 自减1
  * `decry by {key} n`: 将指定的数字 自减n





### List命令

* `lpush key value1 value2...`: 新建一个列表 然后从左侧依次推入若干元素
  * ==tip: 2.4版本之前是不支持推入多个元素的 只支持一个元素==
  * `rpush key value1 value2...`: 同理 从右侧推入
* `lpushX key value1 value2...`: 列表存在的情况下 才会进行推送

  * `rpushX key value1 value2...`: 同理
* `lpop key`: 左侧移除一个元素
* `rpop key`: 右侧移除一个元素
* `RpopLpush source target`: 移除source列表的右侧一个元素 填充到target列表的左侧
* `LLEN key`: 获取列表的长度
* `Lindex key index`: 通过下标来获取元素 从`0`开始

* `Lrange key m n`: 查看子列表 `[m, n]` 从`0`开始

* `Lset key index value`: 通过下标设置元素的值

* `Lrem key count value`: 从列表中删除count个值为value的元素

  * count > 0: 从左边开始删除
  * count < 0: 从右边开始删除
  * count = 0: 全部删除

* `Ltrim key m n`: 将列表修剪为`[m, n]` 从`0`开始

* `Linsert key BEFORE|AFTER pivot value`:  在值为pivot的元素前后插入一个值

  

### Set命令

* `sadd key mem1 mem2...`: 新建或者修改一个集合 添加若干个值
  * ==tip: 同样的 2.4之前只支持单个值==
* `scard key`: 获取集合中的成员数
* `sIsMember key mem`: 判断某个元素是否属于指定集合
* `Smembers`: 获取集合中的所有元素
* `Smove source target mem`: 将指定的元素从source移动到target
* `Spop key`: 随机移除一个元素
* `Srem key mem1 mem2...`: 移除多个值为mem的元素
* `sRandMember key count`: 随机返回count个元素
* `Sunion key1 key2...`: 返回所有指定集合的并集
  * `SunionStore target key1 key2...`: 返回所有指定集合的并集 然后存储在target当中
* `Sinter key1 key2...`: 返回所有指定集合的交集
* `Sdiff key1 key2...`: 返回所有指定集合的差集
  * 同理会有`SinterStore` `SdiffStore`





### Hash命令

* `Hset key field value`: 设置一个属性
  * `HsetNX key field value`: 只有在属性不存在的时候 才会去设置
  * `HMset key f1 v1 f2 v2...`: 设置多个属性
* `Hget key field`: 获取一个属性的值
  * `HMget f1 f2...`: 获取多个属性
* `Hexists key field`: 判断某个属性是否存在
* `Hlen key`: 获取属性的数量
* `Hdel key field1 field2...`: 删除一个或者多个属性
* `Hvals key`: 获取哈希表中的所有值
* `Hkeys key`: 获取哈希表中的所有键
* `HgetALL key`: 获取哈希表中的所有键值对
* `HincreBy key field n`: 将哈希表中的某个字段 自增n
* `HincreByFloat key field n`: 将哈希表中的某个字段自增float





## Jedis

**Maven依赖**

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```

**示例代码**

```java
public class RedisTest {
    @Test
    public void test() {
        String host = "192.168.124.141";
        int port = 6379;
        Jedis jedis = new Jedis(host, port);
        System.out.println(jedis.ping());
    }
}
```



## 持久化

由于是内存型数据库 所以说肯定要面临数据丢失的问题... 所以说当然也要考虑持久化问题... 看看memcache的排行你就知道持久化的重要性了... Redis的持久化支持两种:

* `RDB(redis database)`: 按时将内存的快照写入磁盘
  * Redis会单独的创建(fork)一个子进程来进行持久化, 整个过程主进程是不进行任何操作的, 这样确保了极高的性能. 缺点就是==最后一次持久化之后的数据可能会丢失== 如果对数据的完整性不敏感, 建议使用RDB.
  * Linux程序中, 处于效率的考虑引入了"写时复制"的技术, 一般父进程和子进程会共用一段物理内存, 只有当某个进程发生写操作的时候, 才会复制一份给其他进程... 所以说Redis中的fork操作是非常高效的.
* `AOF(append of file)`: 利用日志记录每一个写操作
  * 将Redis执行过的所有写操作记录下来, 在下一次Redis服务器启动的时候读取这个日志文件, 然后依次执行所有的命令, 就可以恢复到以前的状态了



### RDB

在`redis.conf`中有几个关于RDB的配置:

* `dbfilename dump.rdb`: 持久化文件的名称... 一般不做修改

* `dir ./`: 表示RDB的持久化文件`dump.rdb`存储的位置... 以`redis-server`运行的所在目录为相对路径.
* `save 300 10`: 代表300秒内如果进行了10次写操作 就会进行一次持久化操作

```ini
save 900 1				# 900秒内修改了一次
save 300 10				# 或者 300秒内修改了10次
save 600 10000			# 或者 600秒内修改了10000次 均会进行持久化操作
```

---



我们可以在`redis-cli`客户端中进行如下的几个RDB操作:

* `config get save`: 查看Reids的配置参数`save`的值
* `save`: 这个命令可以强制执行一次RDB持久化操作, 会阻塞其他线程 影响效率
* `bgsave`: 这个命令可以进行一次创建子线程的持久化操作, ==和自动持久化的方式相同==



> 提到了`dir ./` 我们就应该说一下, 相对路径的问题.... 虽然`dir ./`写在了配置文件`redis.conf`当中, 但是这个配置文件可不能直接运行, 他只是`redis-server`运行所需要读取的一个文本文件罢了, 所以说相对路径与`redis-server`有关, 与`redis.conf`无关, 而具体是什么路径, 要根据你的`redis-server`命令在什么`工作路径`运行, 也就是`pwd`查询到的路径.
>
> 通过上面的分析, 我们现在不难理解为什么**Java中的File类相对路径是项目文件夹** 因为`Java虚拟机`的默认启动路径已经被Idea/Eclipse等软件改变成为了我们项目所在的路径....



### AOF

默认Reids是关闭AOF的, 需要我们手动在配置文件当中开启:

* `appendonly no`: 需要我们将这一项修改为`yes`, 才能开启Redis的AOF功能
* `appendfilename appendonly.aof`: AOP的持久化文件的名称... 一般不做修改
* `dir ./`: 同理, AOF的配置文件位置也是通过这个参数来指定的...
* `appendsync ?`:  配置AOF进行持久化的频率, 可以有如下三个取值
  * `always`: 每有一个写操作 就进行一次持久化
  * `everysec`: 每秒进行一次持久化
  * `no`: 交给操作系统... 操作系统自行决定, 很怪



#### RDB和AOF同时开启 听谁的?

答案是听`AOF`的, 因为`AOP`总是会保存更多的信息...



#### AOP文件鼓掌恢复

如果我们AOF的持久化文件`appendonly.aof`出现了错误命令等情况 可以通过下面的程序修复:

```linux
[root@localhost redis]$ /bin/reids-check-aof --fix ../conf/appendonly.aof
```



#### 异常: Can't open the append-only file Permission denied

这是因为AOF持久化生成的`appendonly.aof`是`只读`的, 当我们第二次启动Redis服务器的会读取这个文件... 就会出现这个异常了... 所以说我们只需要==除去appendonly的只读权限==即可.



## 主从复制&读写分离



