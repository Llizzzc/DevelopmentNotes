# Redis
>1. [动力节点2022](https://www.bilibili.com/video/BV1U24y1y7jF?p=3&spm_id_from=pageDriver&vd_source=e5de1dabc977707311263a4bc0f609cc "redis")

## 一 基本概述
### 1.1 NoSQL
NoSQL泛指非关系型的数据库，其产生就是为了解决大规模数据集合、多重数据种类带来的挑战，特别是大数据应用难题。
#### 1.1.1 键值存储数据库
就像map一样，redis就是典型代表。
#### 1.1.2 列存储数据库
关系型数据库就是典型的行存储数据库，其存在的问题是，按行存储在物理层占用的连续的存储空间，不适合存储海量数据。而按列存储则可实现分布式存储，适合海量存储，典型代表是HBase。
#### 1.1.3 文档型数据库
其为NoSQL于关系型数据库的结合，典型代表是MongoDB。
#### 1.1.4 图形数据库
用于存放一个节点关系的数据库，例如描述不同人之间的关系，代表是Noe4J。
### 1.2 Redis的用途
生产环境用的最多的场景就是做数据缓存，即用户从持久层查出的数据先写入到redis中，后续无论哪个用户再访问该数据，就直接读取redis中的数据，这样降低了持久层的压力。缓存一般可以分为实时同步缓存与阶段性同步缓存。前者是指持久层的数据更新之后，立即清除放入redis中的相关数据，下次访问时，必须先从持久层查询到最新的数据，再放入到redis中进行缓存；后者允许在一段时间内，redis中的数据与持久层的数据不一致，这个时间就是缓存数据的过期时间。
![](img/01.png)
### 1.3 Redis的特性
+ 性能极高：redis读的速度可以达到11w次/s，写的速度可以达到8w次/s。因为redis所有操作都是在内存中发生，而且用c语言开发的，源码非常精细。
+ 简单稳定：redis源码很少，早期只有2w行左右，从3.0版本增加了集群之后，变成5w行左右。
+ 持久化：redis内存中的数据可以进行持久化，有两种方式-RDB与AOF。
+ 高可用集群：redis提供了高可用的主从集群功能，可以确保系统的安全性。
+ 丰富的数据类型：支持字符串、链表、集合、有序集合、哈希类型等。
+ 强大的功能：提供了数据过期功能、发布/订阅功能、简单事务功能、还支持Lua脚本拓展功能。
+ 客户端语言广泛：提供了简单的TCP通信协议，编程语言可以很方便的接入redis。
+ 支持ACL权限控制：从6.0开始引入了ACL模块，可以为不同用户定制不同的用户权限。
+ 支持多线程IO模型：从6.0开始支持多线程模型。
### 1.4 Redis的IO模型
#### 1.4.1 单线程模型
优点：可维护性高，性能高，不存在并发读写情况，所以也就不存在执行顺序的不确定性。不存在线程切换开销，不存在死锁问题，不存在为了数据安全而进行的加锁/解锁。
缺点：性能会受到影响，由于单线程只能使用一个处理器，所以会形成处理器浪费。![](img/02.png)
#### 1.4.2 多线程模型
优点：结合了单线程与多线程的优点，避开了它们的不足。
缺点：没有显著不足，但其并非真正意义上的多线程，因为真正处理任务的线程依然是单线程。![](img/03.png)

## 二 安装与配置
### 2.1 Redis的安装
#### 2.1.1 安装虚拟机
#### 2.1.2 安装前的准备工作
+ `yum -y install gcc gcc-c++`	// 安装gcc
+ `tar -xvzf redis.tar.gz`	// 解压redis
#### 2.1.3 安装Redis
+ `make`	// 在redis目录执行make
+ `make install` 	// make完之后执行，在/usr/local/bin可以看到相关命令
#### 2.1.4 启动与停止
+ `redis-server`	// 启动
+ `redis-cli`	// 进入客户端
+ `quit`	// 退出客户端
+ `nohup redis-server &`	// 后台启动，会产生nohup.out文件
+ `redis-cli shutdown`	// 停止
+ `vim redis.conf; redis-server ConfigPath`	// 修改daemonize为yes，以守护进程方式启动，启动时传入配置文件路径
### 2.2 连接前的配置
+ `#bind:127.0.0.1` 	// 修改配置文件，注释掉bind，允许远程访问
+ `protected-mode no`	// 关闭保护模式
+ `requirepass Password`	// 设置密码
+ `auth Password`	// 进入客户端获得权限
+ `rename-command Command "NewName"`	// 重命名命令
+ `redis-cli -h Host -p Port -a Password`	// 远程连接
### 2.3 Redis配置文件详解
+ `include ConfigPath`	// 写到最后一行，用来添加其他配置文件，会覆盖之前相同的属性
+ `timeout Time` // 设置某个时间内不进行操作就断开连接
+ `tcp-keepalive Time`	// 每隔该时间检测一次客户端，两次检测后不存在才会断开，timeout属性为0时才有用
+ `tcp-backlog Length`	// tcp连接的队列长度，主要用于解决高并发下客户端慢连接问题。linux2.2之前，backlog队列中有已完成三次握手和未完成三次握手的两种状态；2.2之后，backlog就是已完成三次握手的队列。tcp中的backlog队列在linux中由参数somaxconn来决定，所以，在redis中该队列的长度由配置文件设置与somaxconn来共同决定，取最小值
+ `cat /proc/sys/net/core/somaxconn`	// 查看linux中somaxconn
+ `vim /etc/sysctl.conf`	// 设置net.core.somaxconn=2048，修改somaxconn的值
+ `sysctl -p`	// 使修改生效
+ `pidfile PidFile`	// 指定redis运行时pid写入的文件，不配置的话，使用前台方式启动则不会产生pid文件
+ `logfile LogFile`	// 指定日志文件，如果设置为空串，则将日志记录到显示器，如果是守护进程模式，则发送到设备/dev/null（空设备）
+ `database Num`	// 设置数据库的数量，编号为0～Num-1
+ `maxclients Num`	// 设置可并发处理的客户端连接数，如果达到了最大值，则会拒绝新的连接，该值不能超过linux系统支持的可打开的文件描述符最大数量阈值
+ `ulimit -n`	// 查看阈值，可以通过修改/etc/security/limits.conf文件来设置
+ `maxmemory Bytes`	// 将内存限制为指定的字节数，当达到限制时，根据maxmemory-policy来尝试删除符合条件的key
+ `maxmemory-policy Policy`	// 选择删除策略，当没有符合相应策略的内容要删除时，在执行写入命令时会给出error响应
+ `maxmemory-samples`	// 用于指定要删除的key的样本数量，样本选择采用的是LRU算法，从样本中选择要删除的key，采用的是maxmemory-policy指定的策略
+ `maxmemory-eviction-tenacity Size`	// 对选择出来的key，什么时候删除，数值越小删的越快
+ `lscpu`	// 查看系统cpu数量
+ `io-thread Num`	// 指定要启用多线程模型时，使用的线程的数量

## 三 Redis命令
### 3.1 基本命令
+ `ping`	// 看到pong响应，说明该客户端与Redis的连接正常
+ `set Key Value`	// 设置键值对
+ `get Key`	// 获取Key对应的值
+ `select DatabaseId`	// 选择数据库
+ `dbsize`	// 查看数据库大小
+ `flushdb`	// 删除当前数据库数据
+ `flushall`	// 删除所有数据库数据
### 3.2 Key操作命令
Redis中存储的数据整体是一个Map，其Key为String类型，而Value则可以是String、Hash、List、Set等类型。
+ `keys Pattern`	//  按正则匹配查找Key，速度快，在大的数据库中使用可能会阻塞服务器，所以生产环境一般不使用
+ `exists Key`	// 存在返回1，否则0
+ `del Key...`	//  删除指定的一个或多个Key，会忽略不存在的Key，返回删除的数量
+ `rename Key NewKey`	// 将指定Key改名，当Key和NewKey相同，或者Key不存在，会返回一个错误，当NewKey已经存在时，rename会覆盖Key的值
+ `move Key DatabasesId`	// 将当前数据库的Key移动到给定数据库，如果存在同名Key或者Key不存在，则move没有效果
+ `type Key`	// 返回Key所存储的类型
+ `expire Key Seconds`	// 为Key设置生存时间，过期会被自动删除，rename不会改变Key生存时间，设置成功返回1
+ `pexpire Key Milliseconds`	// 毫秒单位
+ `ttl Key`	// 查看Key剩余的生存时间，Key不存在返回-2，Key存在但没有设置生存时间返回-1
+ `pttl Key`	// 毫秒单位
+ `persist Key`	// 去除Key的生存时间 ，若Key不存在或Key没有设置生存时间，返回0
+ `randomkey`	// 随机返回一个Key
+ `scan Cursor (match Pattern) count Num`	// 迭代数据库中的Key，Cursor为迭代开始的游标
### 3.3 String类型Value操作命令
Redis存储数据的Value可以是一个String类型数据。String类型的Value是Redis中最基本，最常见的类型。String类型的Value中可以存放任意数据，包括数值型，甚至是二进制的图片、音频、视频、序列化对象等。一个String类型的Value最大是512M大小。
+ `setex Key Seconds Value`	// 设置键值对时指定生存时间，如果Key已经存在，则覆写旧值
+ `psetex Key Milliseconds Value`	// 毫秒单位
+ `setnx Key Value`	// 不存在时才设置键值对
+ `getset Key Value`	// 	将Key的值设置为Value，并且返回旧值
+ `mset [Key Value]...`	// 同时设置一个或多个键值对，对存在的Key会用新值覆盖
+ `msetnx [Key Value]...`	// 只有在Key都不存在的时候才会进行设置
+ `mget Key...`	// 一次获取多个Key
+ `append Key Value`	// 如果Key存在并且是一个字符串，则把Value追加到原来值的末尾，如果不存在，则设置键值对
+ `incr Key`	// 将Key中存储的数字值加1，如果Key不存在，那么Key的值会被初始化为0，如果Key的值不能表示为数字，那么返回错误
+ `decr Key`	// 将Key中存储的数字值减1
+ `incrby Key Increment`	// 将Key中存储的数字增加Increment，只能是整数
+ `decrby Key Increment`	// 将Key中存储的数字减少Increment，只能是整数
+ `incrbyfloat Key Increment`	// 可以是浮点数的增量
+ `strlen Key`	// 返回Key所存储的字符串值的长度
+ `getrange Key Start End`	// 返回Key中存储的字符串的子串，闭区间，支持负偏移量，-1表示最后一个字符，以此类推
+ `setrange Key Offset Value`	// 用Value替换给定Key所储存的字符串值，从偏移量Offset开始

应用场景：
+ 数据缓存：Redis作为数据缓存层，MySQL作为数据存储层。应用服务器首先从Redis中获取数据，如果缓存层中没有，则从MySQL中获取后先存入缓存层再返回给应用服务器。
+ 计数器：在Redis中写入一个value为数值型的key作为平台计数器、视频播放计数器等。每个有效客户端访问一次，或视频每播放一次，都是直接修改Redis中的计数器，然后再以异步方式持久化到其它数据源中，例如持久化到MySQL。
+ 共享Session：对于一个分布式应用系统，如果将类似用户登录信息这样的Session 数据保存在提供登录服务的服务器中，那么如果用户再次提交像收藏、支付等请求时可能会出现问题：在提供收藏、支付等服务的服务器中并没有该用户的Session数据，从而导致该用户需要重新登录。对于用户来说，这是不能接受的。此时，可以将系统中所有用户的Session数据全部保存到Redis中，用户在提交新的请求后，系统先从Redis中查找相应的Session数据，如果存在，则再进行相关操作，否则跳转到登录页面。这样就不会引发“重新登录”问题。![](img/04.png)
+ 限速器：现在很多平台为了防止DoS攻击，一般都会限制一个IP不能在一秒内访问超过n次。而Redis可以结合key的过期时间与incr命令来完成限速功能，充当限速器。
### 3.4 Hash类型Value操作命令
Redis存储数据的Value可以是一个Hash类型。Hash类型也称为Hash表、字典等。Hash表就是一个映射表Map，也是由键值对构成，为了与整体的Key进行区分，这里的键称为Field，值称为Value。注意，Redis的Hash表中的Field-Value对均为String类型。
+ `hset Key Field Value`	// 将Key所存储的哈希表中的Field的值设为Value
+ `hget Key Field`	// 返回Key所存储的哈希表中的Field的值
+ `hmset Key [Field Value]...`	//  创建多个Field-Value对
+  `hmget Key Field...`	// 获取多个Field的值
+  `hgetall Key`	// 返回所有Field-Value对
+  `hsetnx Key Field Value`	// 将Key所存储的哈希表中的Field的值设为Value，当且仅当域Field不存在
+  `hdel Key Field...`	// 删除一个或多个Field
+  `hexists Key Field`	// 查看Field是否存在
+  `hincrby Key Field Increment`	// 为Field的值增加Increment，整数
+  `hincrbyfloat Key Field Increment`	// 可以为小数
+  `hkeys Key`	// 返回所有Field
+  `hvals Key`	// 返回所有Field的值
+  `hlen Key`	// 返回Field的数量
+  `hstrlen Key Field`	// 返回Field所指的字符串长度

应用场景：
+ Hash型Value非常适合存储对象数据。Key为对象名称，Value为描述对象属性的Map，对对象属性的修改在Redis中就可直接完成。其不像String型Value存储对象，那个对象是序列化过的，例如序列化为JSON串，对对象属性值的修改需要先反序列化为对象后再修改，修改后再序列化为JSON串后写入到Redis。
### 3.5 List类型Value操作命令
Redis存储数据的Value可以是一个String列表类型数据。即该列表中的每个元素均为String类型数据。列表中的数据会按照插入顺序进行排序。不过，该列表的底层实际是一个无头节点的双向链表，所以对列表表头与表尾的操作性能较高，但对中间元素的插入与删除的操作的性能相对较差。
+ `lpush Key Value...`	// 各个Value会按从左到右的顺序依次插入到表头
+ `rpush Key Value`	// 各个Value会按从左到右的顺序依次插入到表尾
+ `llen Key`	// 返回列表长度
+ `lindex Key Index`	// 返回列表Key中，下标为Index的元素。列表从0开始计数
+ `lset Key Index Value`	// 将列表Key下标为Index的元素的值设置为Value
+ `lrange Key Start Stop`	// 返回列表Key中指定闭区间内的元素
+ `lpushx Key Value`	// 将值Value插入到列表Key的表头
+ `rpush Key Value`	// 将值Value插入到列表Key的表尾
+ `linsert Key before|after Pivot Value`	// 将值Value插入到列表Key当中，位于元素Pivot之前或之后
+ `lpop Key [Count]`	// 从列表Key的表头移除Count个元素，默认为1
+ `rpop Key [Count]`	// 从列表Key的表尾移除Count个元素，默认为1
+ `blpop Key... Timeout`	// 当给定列表内没有任何元素可供弹出的时候，连接将被blpop/brpop命令阻塞，直到等待Timeout超时或发现可弹出元素为止。当给定多个 Key参数时，按参数Key的先后顺序依次检查各个列表，弹出第一个非空列表的头元素。Timeout为阻塞时长，单位为秒，其值若为0，则表示只要没有可弹出元素，则一直阻塞。
+ `brpop Key... Timeout`	// 同上
+ `rpoplpush Source Destination`	// 将列表Source中的尾元素弹出，并返回给客户端；将Source弹出的元素插入到列表Destination的头部
+ `brpoplpush Source Destination Timeout`	// brpoplpush是rpoplpush的阻塞版本，当给定列表Source不为空时，brpoplpush的表现和rpoplpush一样。当列表Source为空时，brpoplpush命令将阻塞连接，直到等待超时，或有另一个客户端对Source执行lpush或rpush命令为止。Timeout为阻塞时长，单位为秒，其值若为0，则表示只要没有可弹出元素，则一直阻塞
+ `lrem Key Count Value`	// 根据参数Count的值，移除列表中与参数Value相等的Count个元素，大于0从表头开始搜索，小于0从表尾开始搜索，等于0移除所有
+ `ltrim Key Start Stop`	// 让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除

应用场景：
+ 栈：通过lpush + lpop可以实现栈数据结构效果。通过lpush从列表左侧插入数据，通过lpop从列表左侧取出数据。当然，通过rpush + rpop也可以实现相同效果，只不过操作的是列表右侧。
+ 队列：通过lpush + rpop可以实现队列数据结构效果。通过lpush从列表左侧插入数据，通过rpop从列表右侧取出数据。当然，通过rpush + lpop也可以实现相同效果，只不过操作的方向正好相反。
+ 阻塞式消息队列：通过lpush + brpop可以实现阻塞式消息队列效果。作为消息生产者的客户端使用lpush从列表左侧插入数据，作为消息消费者的多个客户端使用brpop阻塞式“抢占”列表尾部数据进行消费，保证了消费的负载均衡与高可用性。brpop的Timeout 设置为0，表示只要没有数据可弹出，就永久阻塞。
+ 动态有限集合：通过lpush + ltrim可以实现有限集合。通过lpush从列表左侧向列表中添加数据，通过ltrim保持集合的动态有限性。像企业的末位淘汰、学校的重点班等动态管理，都可通过这种动态有限集合来实现。当然，通过rpush + ltrim也可以实现相同效果，只不过操作的方向正好相反。
### 3.6 Set类型Value操作命令
Redis存储数据的Value可以是一个Set集合，且集合中的每一个元素均String类型。Set与List非常相似，但不同之处是Set中的元素具有无序性与不可重复性，而List则具有有序性与可重复性。
+ `sadd Key Member...`	// 将一个或多个元素加入集合
+ `smembers Key`	// 返回集合所有元素
+ `scard Key`	// 返回集合的长度
+ `sismember Key Member`	// 判断Member是否是集合的成员
+ `smove Source Destination Member`	// 将Member元素从Source集合移动到Destination集合
+ `srem Key Member...`	// 移除集合中的一个或多个Member元素
+ `srandmember Key [Count]`	// 返回集合中的Count个随机元素。count默认值为1
+ `spop Key [Count]`	// 移除并返回集合中的Count个随机元素。Count必须为正数，且默认值为1
+ `sdiff Key...`	// 返回第一个集合与其它集合之间的差集
+ `sdiffstore Destination Key...`	// 	同上，还能将差集存储到指定的集合 Destination中
+ `sinter Key...`	// 返回多个集合间的交集
+ `sinterstore Destination Key...`	// 	同上，还能将交集存储到指定的集合 Destination中
+ `sunion Key...`	// 返回多个集合间的并集
+ `sunionstore Destination Key...`	// 同上，还能将并集存储到指定的集合 Destination中

应用场景：
+ 动态黑名单：例如某服务器中要设置用于访问控制的黑名单。如果直接将黑名单写入服务器的配置文件，那么存在的问题是，无法动态修改黑名单。此时可以将黑名单直接写入Redis，只要有客户端来访问服务器，服务器在获取到客户端IP后先从Redis的黑名单中查看是否存在该IP，如果存在，则拒绝访问，否则访问通过。![](img/05.png)
+ 有限随机数：有限随机数是指返回的随机数是基于某一集合范围内的随机数，例如抽奖、随机选人。通过spop或srandmember可以实现从指定集合中随机选出元素。
+ 用户画像：社交平台、电商平台等各种需要用户注册登录的平台，会根据用户提供的资料与用户使用习惯，为每个用户进行画像，即为每个用户定义很多可以反映该用户特征的标签，这些标签就可以使用sadd添加到该用户对应的集合中。这些标签具有无序、不重复特征。同时平台还可以使用sinter或sinterstore根据用户画像间的交集进行好友推荐、商品推荐、客户推荐等。
### 3.7 有序Set类型Value操作命令
Redis存储数据的Value可以是一个有序Set，这个有序Set中的每个元素均为String类型。有序Set与Set的不同之处是，有序Set中的每一个元素都有一个分值Score，Redis会根据Score的值对集合进行由小到大的排序。其与Set集合要求相同，元素不能重复，但元素的Score可以重复。由于该类型的所有命令均是字母z开头，所以该Set也称为ZSet。
+ `zadd Key [Score Member]...`	// 将一个或多个Member元素及其Score值加入到有序集中的适当位置
+ `zrange Key Start Stop [withscores]`	// 返回有序集中，指定区间内的成员。按Score值递增排序。具有相同Score值的成员按字典序排列。可以通过使用withscores选项，来让成员和它的Score值一并返回
+ `zrevrange Key Start Stop [withscores]`		// 同上，顺序相反
+ `zrangebyscore Key Min Max [withscores] [limit offset count]`	// 返回有序集中，所有Score值介于Min和Max之间的成员。有序集成员按Score值递增排列。具有相同Score值的成员按字典序排列。limit参数指定返回结果的数量及区间，注意当offset很大时，定位offset的操作可能需要遍历整 个有序集，此过程效率可能会较低
+  `zrevrangebyscore Key Max Min [withscores] [limit offset count]`	// 同上，顺序相反
+ `zcard Key`	// 返回集合的长度
+ `zcount Key Min Max`	// 返回有序集中，Score值在Min和Max之间的成员的数量。闭区间
+ `zscore Key Member`	// 返回有序集中，成员Member的Score值
+ `zincrby Key Increment Member`	// 为有序集的成员Member的Score值加上增量Increment。Increment值可以是整数值或双精度浮点数
+ `zrank Key Member`	// 返回有序集中成员Member的排名。按Score值递增排序
+ `zrevrank Key Member`	// 同上，递减
+ `zrem Key Member...`	// 移除有序集中的一个或多个成员
+ `zremrangebyrank Key Start Stop`	// 移除有序集中，指定排名区间内的所有成员
+ `zremrangebyscore Key Min Max`	// 移除有序集中，所有Score值介于Min和Max之间的成员。闭区间
+ `zrangebylex Key Min Max [limit offset count]`	// 适用于集合中所有成员都具有相同分值的情况。当有序集合的所有成员都具有相同的分值时，有序集合的元素会根据成员的字典序来进行排序。即这个命令返回给定集合中元素值介于Min和Max之间的成员。如果有序集合里面的成员带有不同的分值，那么命令的执行结果与zrange效果相同
+ `zlexcount Key Min Max`	// 适用于集合中所有成员都具有相同分值的情况。该命令返回该集合中元素值本身介于Min和Max范围内的元素数量
+ `zremrangebylex Key Min Max`	// 适用于集合中所有成员都具有相同分值的情况。该命令会移除该集合元素值本身介于Min和Max范围内的所有元素

应用场景：
+ 有序Set最为典型的应用场景就是排行榜，例如音乐、视频平台中根据播放量进行排序的排行榜；电商平台根据用户评价或销售量进行排序的排行榜等。将播放量作为Score，将作品Id作为Member，将用户评价积分或销售量作为Score，将商家Id作为Member。使用zincrby增加排序Score，使用zrevrange获取Top前几名，使用zrevrank查询当前排名，使用zscore查询当前排序Score等。
### 3.8 benchmark测试工具
+ `redis-benchmark (-h -p -c -n -d -t -q)`	// h指定ip；p指定端口；c指定模拟有客户端的数量，默认为50；n指定客户端发出的请求数，默认为100000；d指定get、set命令时其操作的value的数据长度，默认3字节；t指定要测试的命令，多个命令使用逗号分隔，不能有空格；q指定仅给出总述性报告
### 3.9 简单动态字符串SDS
#### 3.9.1 SDS简介
无论是Redis的Key还是Value，其基础数据类型都是字符串。例如，Hash型Value的Field与Value的类型、List型、Set型、ZSet型Value的元素的类型等都是字符串。虽然 Redis是使用标准C语言开发的，但并没有直接使用C语言中传统的字符串表示，而是自定义了一 种字符串。这种字符串本身的结构比较简单，但功能却非常强大，称为简单动态字符串，Simple Dynamic String，简称SDS。

注意，Redis中的所有字符串并不都是 SDS，也会出现C字符串。C字符串只会出现在字符串“字面常量”中，并且该字符串不可能发生变更。

#### 3.9.2 SDS结构
SDS不同于C字符串。C字符串本身是一个以双引号括起来，以空字符’\0’结尾的字符序列。但SDS是一个结构体，定义在Redis安装目录下的src/sds.h中：
```c
struct sdshdr {
		// 字节数组, 用于保存字符串 
		char buf[]; 
		// buf[]中已使用字节数量, 称为SDS的长度 
		int len; 
		// buf[]中尚未使用的字节数量 
		int free;
}
```
#### 3.9.3 SDS的优势
+ 防止”字符串长度获取“性能瓶颈：对于C字符串，若要获取其长度，则必须要通过遍历整个字符串才可获取到的。对于超长字符串的遍历，会成为系统的性能瓶颈。由于SDS结构体中直接就存放着字符串的长度数据，所以对于获取字符串长度需要消耗的系统性能，与字符串本身长度是无关的，不会成为Redis的性能瓶颈。
+ 保障二进制安全：C字符串中只能包含符合某种编码格式的字符，例如ASCII、UTF-8等，并且除了字符串末尾外，其它位置是不能包含空字符’\0’的，否则该字符串就会被程序误解为提前结束。而在图片、音频、视频、压缩文件、office文件等二进制数据中以空字符’\0’作为分隔符的情况是很常见的。故而在C字符串中是不能保存像图片、音频、视频、压缩文件、office文件等二进制数据的。但SDS不是以空字符’\0’作为字符串结束标志的，其是通过len属性来判断字符串是否结束的。所以，对于程序处理SDS中的字符串数据，无需对数据做任何限制、过滤、假设，只需读取即可。数据写入的是什么，读到的就是什么。
+ 减少内存再分配次数：SDS采用了空间预分配策略与惰性空间释放策略来避免内存再分配问题。空间预分配策略是指，每次SDS进行空间扩展时，程序不但为其分配所需的空间，还会为其分配额外的未使用空间，以减少内存再分配次数。而额外分配的未使用空间大小取决于空间扩展后SDS的len属性值：
	+ 如果len属性值小于1M，那么分配的未使用空间free的大小与len属性值相同。
	+ 如果len属性值大于等于1M ，那么分配的未使用空间free的大小固定是1M。

	SDS对于空间释放采用的是惰性空间释放策略。该策略是指SDS字符串长度如果缩短，那么多出的未使用空间将暂时不释放，而是增加到free中。以使后期扩展SDS时减少内存再分配次数。如果要释放SDS的未使用空间，则可通过sdsRemoveFreeSpace()函数来释放。
+ 兼容C函数：Redis中提供了很多的SDS的API，以方便用户对Redis进行二次开发。为了能够兼容C函数，SDS的底层数组buf中的字符串仍以空字符’\0’结尾。
#### 3.9.4 常用的SDS函数
|函数|功能描述|
|:--:|:--:|
|sdsnew()|使用指定的C字符串创建一个SDS|
|sdsempty()|创建一个不包含任何字符串数据的SDS|
|sdsdup()|创建一个指定SDS的副本|
|sdsfree()|释放指定的SDS|
|sdsclear()|清空指定SDS的字符串内容|
|sdslen()|获取指定SDS的已使用空间len值|
|sdsavail()|获取指定SDS的未使用空间free值|
|sdsMakeRoomFor()|使指定的SDS的free空间增加指定的大小|
|sdsRemoveFreeSpace()|释放指定SDS的free空间|
|sdscat()|将指定的C字符串拼接到指定SDS字符串末尾|
|sdscatsds()|将指定的SDS的字符串拼接到另一个指定SDS字符串末尾|
|sdscpy()|将指定的C字符串复制到指定的SDS中，覆盖原SDS字符串内容|
|sdsgrouzero()|扩展SDS字符串到指定长度，这个扩展是使用空字符’\0’填充|
|sdsrange()|截取指定SDS中指定范围内的字符串|
|sdstrim()|在指定SDS中删除所有指定C字符串中出现的所有字符|
|sdsemp()|对比两个给定的SDS字符串是否相同|
|sdstolow()|将指定SDS字符串中的所有字母变为小写|
|sdstoupper()|将指定SDS字符串中的所有字母变为大写|
### 3.10 集合的底层实现原理
Redis中对于Set类型的底层实现，直接采用了hashTable。但对于Hash、ZSet、List集合的底层实现进行了特殊的设计，使其保证了Redis的高性能。
#### 3.10.1 两种实现的选择
对于Hash与ZSet集合，其底层的实现实际有两种：压缩列表zipList，与跳跃列表skipList。这两种实现对于用户来说是透明的，但用户写入不同的数据，系统会自动使用不同的实现。只有同时满足配置文件redis.conf中相关集合元素数量阈值与元素大小阈值两个条件，使用的就是压缩列表zipList，只要有一个条件不满足使用的就是跳跃列表skipList。例如，对于ZSet集合中这两个条件如下：
+ 集合元素个数小于redis.conf中zset-max-ziplist-entries属性的值，其默认值为128
+ 每个集合元素大小都小于redis.conf中zset-max-ziplist-value属性的值，其默认值为64字节
#### 3.10.2 zipList
zipList，通常称为压缩列表，是一个经过特殊编码的用于存储字符串或整数的双向链表。其底层数据结构由三部分构成：head、entries与end。这三部分在内存上是连续存放的。![](img/06.png)
+ zlbytes：占4个字节，用于存放zipList列表整体数据结构所占的字节数，包括zlbytes本身的长度。
+ zltail：占4个字节，用于存放zipList中最后一个entry在整个数据结构中的偏移量（字节）。该数据的存在可以快速定位列表的尾entry位置，以方便操作。
+  zllen：占2字节，用于存放列表包含的entry个数。由于其只有16位，所以zipList最多可以含有的entry个数为65535个。
+ entries：由很多的列表元素entry构成。由于不同的元素类型、数值的不同，从而导致每个entry的长度不同。
+  prevlength：该部分用于记录上一个entry的长度，以实现逆序遍历。默认长度为1字节，只要上一个entry的长度<254字节，prevlength就占1字节，否则其会自动扩展为3字节。
+ encoding：该部分用于标志后面的data的具体类型。如果data为整数类型，encoding
固定长度为1字节。如果data为字符串类型，则encoding长度可能会是1字节、2字节或5字节。data字符串不同的长度，对应着不同的encoding长度。
+ data：真正存储的数据。数据类型只能是整数类型或字符串类型。不同的数据占用的字节长度不同。
+ zlend：占1个字节，值固定为255，即二进制位为全1，表示一个zipList列表的结束。
#### 3.10.3 listPack
对于ziplist，实现复杂，为了逆序遍历，每个entry中包含前一个entry的长度，这样会导致在ziplist中间修改或者插入entry时需要进行级联更新。在高并发的写操作场景下会极度降低Redis的性能。为了实现更紧凑、更快的解析，更简单的实现，重写了ziplist，并命名为listPack。在Redis7.0中，已经将zipList全部替换为了listPack，但为了兼容性，在配置中也保留了zipList的相关属性。![](img/07.png)
+ totalBytes：占4个字节，用于存放listPack列表整体数据结构所占的字节数，包括totalBytes本身的长度。
+ elemNum：占2字节，用于存放列表包含的entry个数。其意义与zipList中zllen相同。
+ entries：也是listPack中真正的列表，由很多的列表元素entry构成。由于不同的元素类型、数值的不同，从而导致每个entry的长度不同。
+ encoding：该部分用于标志后面的data的具体类型。如果data为整数类型，encoding长度可能会是1、2、3、4、5或9字节。不同的字节长度，其标识位不同。如果data为字符串类型，则encoding长度可能会是1、2或5字节。data字符串不同的长度，对应着不同的encoding长度。
+ data：真正存储的数据。数据类型只能是整数类型或字符串类型。不同的数据占用的字节长度不同。
+ element-total-len：该部分用于记录当前entry的长度，用于实现逆序遍历。由于其特殊的记录方式，使其本身占有的字节数据可能会是1、2、3、4或5字节。
#### 3.10.4 skipList
skipList，跳跃列表，简称跳表，是一种随机化的数据结构，基于并联的链表，实现简单，查找效率较高。简单来说跳表也是链表的一种，只不过它在链表的基础上增加了跳跃功能。也正是这个跳跃功能，使得在查找元素时，能够提供较高的效率。

为了提升查找效率，在偶序数结点上增加一个指针，让其指向下一个偶序数结点。这样所有偶结点就连成了一个新的链表（简称高层链表），当然，高层链表包含的节点个数只是原来链表的一半。此时再想查找某个数据时，先沿着高层链表进行查找。当遇到第一个比待查数据大的节点时，立即从该节点的前一个节点回到原链表中进行查找。该方式明显可以减少比较次数，提高查找效率。如果链表元素较多，为了进一步提升查找效率，可以将原链表构建为三层链表，或再高层级链表。

这种对链表分层级的方式从原理上看确实提升了查找效率，但在实际操作时就出现了问题：由于固定序号的元素拥有固定层级，所以列表元素出现增加或删除的情况下，会导致列表整体元素层级大调整，但这样势必会大大降低系统性能。

为了避免前面的问题，skipList采用了随机分配层级方式。即在确定了总层级后，每添加一个新的元素时会自动为其随机分配一个层级。这种随机性就解决了节点序号与层级间的固定关系问题。
#### 3.10.5 quickList
quickList，快速列表，quickList本身是一个双向无循环链表，它的每一个节点都是一个zipList。从Redis3.2版本开始，对于List的底层实现，使用quickList替代了zipList和linkedList。

zipList与linkedList都存在有明显不足，而quickList则对它们进行了改进：吸取了zipList和linkedList的优点，避开了它们的不足。

quickList本质上是zipList和linkedList的混合体。其将linkedList按段切分，每一段使用zipList来紧凑存储若干真正的数据元素，多个zipList之间使用双向指针串接起来。当然，对于每个zipList中最多可存放多大容量的数据元素，在配置文件中通过list-max-ziplist-size属性可以指定。
![](img/08.png)
#### 3.10.6 Key与Value中元素的数量
+ Redis最多可以处理约42亿个Key，并且在实践中经过测试，每个Redis实例至少可以处理2.5亿个Key。
+ 每个Hash、List、Set、ZSet集合都可以包含约42亿个元素。
### 3.11 BitMap操作命令
BitMap是Redis2.2.0版本中引入的一种新的数据类型。该数据类型本质上就是一个仅包含0和1的二进制字符串。而其所有相关命令都是对这个字符串二进制位的操作。用于描述该字符串的属性有三个：Key、Offset、BitValue。
+ Key：BitMap是Redis的Key-Value中的一种Value的数据类型，所以该Value一定有其对应的Key。
+ Offset：每个BitMap数据都是一个字符串，字符串中的每个字符都有其对应的索引，该索引从0开始计数。该索引就称为每个字符在该BitMap中的偏移量Offset。该Offset的最大值为42亿多。
+ BitValue：每个BitMap数据中都是一个仅包含0和1的二进制字符串，每个Offset位上
的字符就称为该位的值BitValue。BitValue的值非0即1。
+ `setbit Key Offset Value`	// 为给定Key的BitMap数据的Offset位置设置值为Value。其返回值为修改前该Offset位置的BitValue
+ `getbit Key Offset`	// 对Key所储存的BitMap字符串值，获取指定Offset偏移量上的位值BitValue。
+ `bitcount Key [Start] [End]`	// 统计给定字符串中被设置为1的Bit位的数量。一般情况下，统计整个BitMap字符串。但也可以通过指定额外的Start或End参数，实现仅对指定字节范围内字符串进行统计，包括Start和End在内。注意，这里的Start与End的单位是字节，不是位，并且从0开始计数
+ `bitpos Key Bit [Start] [End]`	// 返回Key指定的BitMap中第一个值为指定值Bit的二进制位的位置。在默认情况下，命令将检测整个BitMap
+ `bitop Operation Destkey key...`	// 对一个或多个BitMap字符串Key进行二进制位操作，并将结果保存到Destkey上。Operation可以是AND、OR、NOT、XOR

应用场景：
+ 由于Offset的取值范围很大，所以其一般应用于大数据量的二值性统计。例如平台活跃 用户统计、支持率统计、员工考勤统计等。不过，对于数据量较小的二值性统计并不适合BitMap，可能使用Set更为合适。当然，具体多少数据量适合使用Set，超过多少数据量适合使用BitMap，这需要根据具体场景进行具体分析。
### 3.12 HyperLogLog操作命令
HyperLogLog是Redis2.8.9版本中引入的一种新的数据类型，其意义是超级日志记录。该数据类型可以简单理解为一个Set集合，集合元素为字符串。但实际上HyperLogLog是一种基数计数概率算法，通过该算法可以利用极小的内存完成独立总数的统计。其所有相关命令都是对这个“Set集合”的操作。
+ `pfadd Key Element...`	// 将任意数量的元素添加到指定的HyperLogLog集合里面	
+ `pfcount Key...`	// 该命令作用于单个Key时，返回给定Key的HyperLogLog集合的近似基数；作用于多个Key时，返回所有给定Key的HyperLogLog集合的并集的近似基数
+ `pfmerge Destkey Sourcekey...`	// 将多个HyperLogLog集合合并为一个HyperLogLog集合，并存储到Destkey中，合并后的HyperLogLog的基数接近于所有Sourcekey的HyperLogLog集合的并集

应用场景：
+ HyperLogLog可对数据量超级庞大的日志数据做不精确的去重计数统计。当然，这个不精确的度在Redis官方给出的误差是0.81%。这个误差对于大多数超大数据量场景是被允许的。对于平台上每个页面每天的UV数据，非常适合使用HyperLogLog进行记录。
### 3.13 Geospatial操作命令
Geospatial，地理空间。Redis在3.2版本中引入了Geospatial这种新的数据类型。该类型本质上仍是一种集合，只不过集合元素比较特殊，是一种由三部分构成的数据结构，这种数据结构称为空间元素：
+ 经度：longitude。有效经度为[-180, 180]。正的表示东经，负的表示西经。
+ 纬度：latitude。有效纬度为[-85.05112878,  85.05112878]。正的表示北纬，负的表示南纬。
+ 位置名称：为该经纬度所标注的位置所命名的名称，也称为该Geospatial集合的空间元
素名称。
+ `geoadd Key [Longitude Latitude Member]...`	// 将一到多个空间元素添加到指定的空间集合中
+ `geopos Key Member`	// 从指定的地理空间中返回指定元素的位置，即经纬度
+ `geodist Key Member1 Member2 [Unit]`	// 返回两个给定位置之间的距离。其中Unit必须是以下单位中的一种：m、km、mi、ft
+ `geohash Key Member`	// 返回一个或多个位置元素的Geohash值
+ `georadius Key Longitude Latitude Radius [Unit] [withcoord] [withdist] [withhash] [asc|desc] [count Count]`	// 以给定的经纬度为中心，返回指定地理空间中包含的所有位置元素中，与中心距离不超过给定半径的元素。返回时还可携带额外的信息
+ `georadiusbymember Key Member Radius [Unit] [withcoord] [withdist] [withhash] [asc|desc] [count Count]`	// 和georadius命令一样，都可以找出位于指定范围内的元素，但该命令的中心点是由位置元素形式给定的，而不是像georadius那样，使用输入的经纬度来指定中心点

应用场景：
+ Geospatial的意义是地理位置，所以其主要应用地理位置相关的计算。例如，微信发现中的“附近”功能，添加朋友中“雷达加朋友”功能，QQ动态中的“附近”功能，钉钉中的“签到”功能等。
### 3.14 发布/订阅命令
发布/订阅，是一种消息通信模式：发布者也称为消息生产者，生产和发送消息到存储系统；订阅者也称为消息消费者，从存储系统接收和消费消息。这个存储系统可以是文件系统、消息中间件、数据管理系统，也可以是Redis。整个消息发布者、订阅者与存储系统称为消息系统。

消息系统中的订阅者订阅了某类消息后，只要存储系统中存在该类消息，其就可不断的 接收并消费这些消息。当存储系统中没有该消息后，订阅者的接收、消费阻塞。而当发布者将消息写入到存储系统后，会立即唤醒订阅者。当存储系统放满时，不同的发布者具有不同的处理方式：有的会阻塞发布者的发布，等待可用的存储空间；有的则会将多余的消息丢失。

当然，不同的消息系统消息的发布/订阅方式也是不同的。例如RocketMQ、Kafka等消息中间件构成的消息系统中，发布/订阅的消息都是以Topic分类的。而Redis构成的消息系统中，发布/订阅的消息都是以Channel分类的。![](img/09.png)
+ `subscribe Channel...`	// 同时订阅任意数量的频道。在输出了订阅的主题后，命令处于阻塞状态，等待相关频道的消息
+ `psubscribe Pattern...`	// 订阅一个或多个符合给定模式的频道，这里的模式只能使用通配符\*
+ `publish Channel Message`	// 发布一个频道的消息。返回值为接收到该消息的订阅者数量
+ `unsubscribe Channel...`	// 退订指定的频道
+ `punsubscribe Pattern...`	// 退订一个或多个符合给定模式的频道
+ `pubsub channels [Pattern]`	// 列出当前所有的活跃频道。活跃频道指的是那些至少有一个订阅者的频道
+ `pubsub numsub [Channel1 ... ChannelN]`	// 返回给定频道的订阅者数量
+ `pubsub numpat`	// 查询当前Redis所有客户端订阅的所有频道模式的数量总和
### 3.15 事务
Redis的事务的本质是一组命令的批处理。这组命令在执行过程中会被顺序地、一次性全部执行完毕，只要没有出现语法错误，这组命令在执行期间是不会被中断。Redis的事务仅保证了数据的一致性，不具有像DBMS一样的ACID特性：
+ 这组命令中的某些命令的执行失败不会影响其它命令的执行，不会引发回滚。即不具备 原子性。
+ 这组命令通过乐观锁机制实现了简单的隔离性。没有复杂的隔离级别。
+ 这组命令的执行结果是被写入到内存的，是否持久取决于Redis的持久化策略，与事务无关。
+ `muti`	// 开启事务
+ `exec`	// 执行事务。当事务中的命令出现语法错误时，整个事务在exec执行时会被取消；如果事务中的命令没有语法错误，但在执行过程中出现异常，该异常不会影响其它命令的执行
+ `discard`	// 取消事务
+ `watch Key`	// 当某一客户端对Key执行了watch后，系统就会为该Key添加一个乐观锁，并初始化version，初值为1.0 
### 3.16 ACL权限控制
在Redis6之前的版本，我们只能使用`requirepass`参数给default用户配置登录密码，同一个Redis集群的所有开发都共享default用户，难免会出现误操作把别人的key删掉或者数据泄露的情况，那之前我们也可以使用`rename command`的方式给一些危险函数重命名或禁用，但是这样也防止不了自己的key被其他人访问。

因此Redis6版本推出了ACL（Access Control List）访问控制权限的功能，基于此功能，我们可以设置多个用户，并且给每个用户单独设置命令权限和数据权限。 为了保证向下兼容，Redis6保留了default用户和使用requirepass的方式给default用户设置密码，默认情况下default用户拥有Redis最大权限，我们使用redis-cli连接时如果没有指定用户名，用户也是默认default。

配置ACL的方式有两种，这两种方法是相互不兼容的，所以Redis会要求你使用其中一种。在redis.conf中指定用户适用于简单的用例。在复杂的环境中，当需要定义多个用户时，建议使用ACL文件。
+ `auth Password`	// 老认证方式
+ `auth Username Password`	// 新认证方式
+ `user Username`	// 使用ACL文件时需要注释
+ `requirepass Password`	// 同上
+ `aclfile`	// 设置ACL文件路径，必须已建立好
+ `redis-cli --user Username  --pass Password`	// 登录方式

ACL是使用DSL（Domain Specific Language）定义的，该DSL描述了用户能够执行的操作。该规则始终从上到下，从左到右应用，因为规则的顺序对于理解用户的实际权限很重要。ACL可以在redis.conf文件以及users.acl文件中配置DSL，也可以在命令行中通过`acl`命令配置。
|规则|作用|
|-|-|
|on|启用用户|
|off|禁用用户|
|+Command|授权某个命令|
|-Command|撤销授权某个命令|
|+@Category|授权某组类别命令，可通过调用`acl cat`命令查看完整列表|
|-@Category|撤销授权某组类别命令|
|+Command\|Subcommand|允许使用已禁用命令的特定子命令|
|allcommands|+@all的别名|
|nocommands|-@all的别名|
|~Pattern|可以在命令中提及的键的模式，\*匹配任意|
|resetkeys|作废在该命令之前设置的所有键匹配规则|
|>Password|添加密码，可以有多个|
|<Password|删除密码|
|#Hash|将此SHA-256哈希值添加到用户的密码列表中。该哈希值将与ACL用户输入的密码的哈希值进行比较|
|!Hash|从密码列表中删除该哈希值|
|nopass|移除用户已设置的所有密码，并将该用户标记为nopass状态，任何密码都可以登录|
|resetpass|清空用户的所有密码，且移除nopass状态。resetpass之后用户没有设置密码则无法登录，因此resetpass之后必须添加密码或改为nopass状态才能正常登录|
|reset|重置用户状态为初始状态|

+ `acl setuser Username`	// 创建用户
+ `acl list`	// 查看用户列表
+ `acl whoami`	// 查看当前用户
+ `acl getuser Username`	// 查看用户信息
+ `acl cat (Category)`	// 查看所有组别或某个组别下的所有命令
+ `acl getpass`	// 获取随机强密码
+ `acl help`	// 使用帮助
+ `acl save`	// 将用户配置保存至文件
+ `acl load`	// 重启服务时从文件加载用户配置

## 四 持久化
Redis是一个内存数据库，所以其运行效率非常高。但也存在一个问题：内存中的数据是不持久的，若主机宕机或Redis关机重启，则内存中的数据全部丢失。当然，这是不允许 的。Redis具有持久化功能，其会按照设置以快照或操作日志的形式将数据持久化到磁盘。根据持久化使用技术的不同，Redis的持久化分为两种：RDB与AOF。
### 4.1 持久化基本原理
![](img/10.png)
Redis持久化也称为钝化，是指将内存中数据库的状态描述信息保存到磁盘中。只不过是不同的持久化技术，对数据的状态描述信息是不同的，生成的持久化文件也是不同的。但它们的作用都是相同的：避免数据意外丢失。

通过手动方式，或自动定时方式，或自动条件触发方式，将内存中数据库的状态描述信 息写入到指定的持久化文件中。当系统重新启动时，自动加载持久化文件，并根据文件中数据库状态描述信息将数据恢复到内存中，这个数据恢复过程也称为激活。这个钝化与激活的过程就是Redis持久化的基本原理。

不过从以上分析可知，对于Redis单机状态下，无论是手动方式，还是定时方式或条件触发方式，都存在数据丢失问题：在尚未手动/自动保存时发生了Redis宕机状况，那么从上次保存到宕机期间产生的数据就会丢失。不同的持久化方式，其数据的丢失率也是不同的。![](img/11.png)

需要注意的是，RDB是默认持久化方式，但Redis允许RDB与AOF两种持久化技术同时开启，此时系统会使用AOF方式做持久化，即AOF持久化技术的优先级要更高。同样的道理，两种技术同时开启状态下，系统启动时若两种持久化文件同时存在，则优先加载AOF持久化文件。
### 4.2 RDB持久化
RDB，Redis DataBase，是指将内存中某一时刻的数据快照全量写入到指定的RDB文件的持久化技术。RDB持久化默认是开启的。当Redis启动时会自动读取RDB快照文件，将数据从硬盘载入到内存，以恢复Redis关机前的数据库状态。
#### 4.2.1 持久化的执行
+ `save`	// 执行save命令可立即进行一次持久化保存。save命令在执行期间会阻塞redis-server进程，直至持久化过程完毕。而在redis-server进程阻塞期间，Redis不能处理任何读写请求，无法对外提供服务
+ `bgsave`	// 执行bgsave命令可立即进行一次持久化保存。不同于save命令的是，正如该命令的名称一样，background save，后台运行save。bgsave命令会使服务器进程redis-server生成一个子进程，由该子进程负责完成保存过程。在子进程进行保存过程中，不会阻塞redis-server进程对客户端读写请求的处理
+ 用户通过在配置文件中做相应的设置后，Redis会根据设置信息自动调用bgsave命令执行。
+ `lastsave`	// 查看最近一次执行持久化的时间，其返回的是一个Unix时间戳
#### 4.2.2 RDB优化配置
+ `save`	// 该配置用于设置快照的自动保存触发条件，该触发条件是在指定时间段内发生了指定次数的写操作。除非另有规定，默认情况下持久化条件为save 3600 1 300 100 60 10000，在3600秒内发生1次写操作；在300秒内发生100次写操作；在60秒内发生1万次写操作
+ `stop-writes-on-bgsave-error`	// 默认情况下，如果RDB快照已启用，且最近的bgsave命令失败，Redis将停止接受写入。这样设置是为了让用户意识到数据没有正确地保存到磁盘上，否则很可能没有人会注意到，并会发生一些灾难。当然，如果bgsave命令后来可以正常工作了，Redis将自动允许再次写入
+ `rdbcompression`	// 当进行持久化时启用LZF压缩字符串对象。虽然压缩RDB文件会消耗系统资源，降低性能，但可大幅降低文件的大小，方便保存到磁盘，加速主从集群中从节点的数据同步
+ `rdbchecksum`	// 从RDB5开始，RDB文件的CRC64校验和就被放置在了文件末尾。这使格式更能抵抗RDB文件的损坏，但在保存和加载RDB文件时，性能会受到影响，因此可以设置为no禁用校验和以获得最大性能。在禁用校验和的情况下创建的RDB文件的校验和为零，这将告诉加载代码跳过校验检查。默认为 yes。
+ `sanitize-dump-payload`	// 该配置用于设置在加载RDB文件或进行持久化时是否开启对zipList、listPack等数据的全面安全检测。该检测可以降低命令处理时发生系统崩溃的可能。
+ `dbfilename`	// 指定RDB文件的默认名称，默认为dump.rdb
+ `rdb-del-sync-files`	// 主从复制时，是否删除用于同步的从机上的RDB文件。默认是no。不过需要注意，只有当从机的RDB和AOF持久化功能都未开启时才生效
+ `dir`	// 指定RDB与AOF文件的生成目录。默认为Redis安装根目录
#### 4.2.3 RDB文件结构
![](img/12.png)
+ SOF：常量，一个字符串“REDIS”，仅包含这五个字符，其长度为5。用于标识RDB文件的开始，以便在加载RDB文件时可以迅速判断出文件是否是RDB文件。
+ rdb_version：整数，长度为4字节，表示RDB文件的版本号。
+ EOF：常量，占1个字节，用于标识RDB数据的结束，校验和的开始。
+ check_sum：用于判断RDB文件中的内容是否出现数据异常。其采用的是CRC校验算法。
+ databases：数据部分，其可以包含任意多个非空数据库。而每个database又是由三部分构成。
+ SODB：常量，占1个字节，用于标识一个数据库的开始。
+ db_number：数据库编号。
+ key_value_pairs：当前数据库中的键值对数据。
#### 4.2.4 RDB持久化过程
![](img/13.png)
对于Redis默认的RDB持久化，在进行bgsave持久化时，redis-server进程会fork出一个bgsave子进程，由该子进程以异步方式负责完成持久化。而在持久化过程中，redis-server进程不会阻塞，其会继续接收并处理用户的读写请求。

bgsave子进程的详细工作原理如下：
由于子进程可以继承父进程的所有资源，且父进程不能拒绝子进程的继承权。所以，bgsave子进程有权读取到redis-server进程写入到内存中的用户数据，使得将内存数据持久化到dump.rdb成为可能。bgsave子进程在持久化时首先会将内存中的全量数据copy到磁盘中的一个RDB临时文件，copy结束后，再将该文件rename为dump.rdb，替换掉原来的同名文件。
![](img/14.png)
不过，在进行持久化过程中，如果redis-server进程接收到了用户写请求，则系统会将内存中发生数据修改的物理块copy出一个副本。等内存中的全量数据copy结束后，会再将副本中的数据copy到RDB临时文件。这个副本的生成是由于Linux系统的写时复制技术实现的。
### 4.3 AOF持久化
AOF，Append Only File，是指Redis将每一次的写操作都以日志的形式记录到一个AOF文件中的持久化技术。当需要恢复内存数据时，将这些写操作重新执行一次，便会恢复到之前的内存数据状态。
#### 4.3.1 AOF基础配置
+ `appendonly`	// 默认情况下AOF持久化是没有开启的
+ `appendfilename`	// 文件名配置
+ `aof-use-rdb-preamble`	// 对于基本文件可以是RDB格式也可以是AOF格式。默认AOF持久化的基本文件为RDB格式文件，也就是默认采用混合式持久化
+ `appenddirname`	// 为AOF持久化文件指定存放目录。存放在redis.conf配置文件的dir属性指定的目录
#### 4.3.2 AOF文件格式
增量文件采用AOF格式，如appendonly.aof.1.incr.aof。AOF格式其实就是Redis通讯协议格式，AOF持久化文件的本质就是基于Redis通讯协议的文本，将命令以纯文本的方式写入到文件中。Redis协议规定，Redis文本是以行来划分，每行以\r\n行结束。每一行都有一个消息头，以表示消息类型。消息头由六种不同的符号表示，其意义如下：
+ (+)：表示一个正确的状态信息
+ (-)：表示一个错误信息
+ (\*)：表示消息体总共有多少行，不包括当前行
+ ($)：表示下一行消息数据的长度，不包括换行符长度\r\n
+ (空)：表示一个消息数据
+ (:)：表示返回一个数值

appendonly.aof.manifest为清单文件，该文件首先会按照seq序号列举出所有基本文件，基本文件type类型为b，然后再按照seq序号再列举出所有增量文件，增量文件type类型为i。对于Redis启动时的数据恢复，也会按照该文件由上到下依次加载它们中的数据。
#### 4.3.3 Rewrite机制
随着使用时间的推移，AOF文件会越来越大。为了防止AOF文件由于太大而占用大量的磁盘空间，降低性能，Redis引入了Rewrite机制来对AOF文件进行压缩。
+ `bgrewriteaof` // 手动开启。在Rewrite期间，redis-server仍是可以对外提供读写服务的
+ `auto-aof-rewrite-percentage`	// 开启Rewrite的增大比例，默认 100%。指定为0，表示禁用自动Rewrite
+ `auto-aof-rewrite-percentage`	// 开启Rewrite的AOF文件最小值，默认64M。该值的设置主要是为了防止小AOF文件被Rewrite，从而导致性能下降。其工作原理如下：Redis会记住最新Rewrite后的AOF文件大小作为基本大小，如果从主机启动后就没有发生过重写，则基本大小就使用启动时AOF的大小。如果当前AOF文件大于基本大小的配置文件中指定的百分比阈值，且当前AOF文件大于配置文件中指定的最小阈值，则会触发Rewrite

所谓Rewrite其实就是对AOF文件进行重写整理。当Rewrite开启后，主进程redis-server创建出一个子进程bgrewriteaof，由该子进程完成Rewrite过程。其首先对现有AOF文件进行Rewrite计算，将计算结果写入到一个临时文件，写入完毕后，再rename该临时文件为原AOF文件名，覆盖原有文件。Rewrite 计算遵循以下策略：
+ 读操作命令不写入文件
+ 无效命令不写入文件
+ 过期数据不写入文件
+ 多条命令合并写入文件
#### 4.3.4 AOF优化配置
+ `appendfsync`	// 当客户端提交写操作命令后，该命令就会写入到aof_buf中，而aof_buf中的数据持久化到磁盘AOF文件的过程称为数据同步。何时将aof_buf中的数据同步到AOF文件？有三种策略：always、no、everysec
+ `no-appendfsync-on-rewrite`	// 该属性用于指定，sync策略设置为always或everysec，当主进程创建了子进程正在执行bgsave或bgrewriteaof 时，主进程是否不调用fsync()来做数据同步。设置为no，双重否定即肯定，主进程会调用fsync()做同步。而yes则不会调用fsync()做数据同步
+ `aof-rewrite-incremental-fsync`	// 当bgrewriteaof在执行过程也是先将Rewrite计算的结果写入到了aof_rewrite_buf缓存，然后当缓存中数据达到一定量后就会调用fsync()进行刷盘操作，即数据同步，将数据写入到临时文件。该属性用于控制fsync()每次刷盘的数据量最大不超过4MB。这样可以避免由于单次刷盘量过大而引发长时间阻塞
+ `aof-load-truncated`	// 在进行AOF持久化过程中可能会出现系统突然宕机的情况，此时写入到AOF文件中的最后一条数据可能会不完整。当主机启动后，Redis在AOF文件不完整的情况下是否可以启动，取决于该属性的设置。其值为：yes、no
+ `aof-timestamp-enabeld`	// 该属性设置为yes则会开启在AOF文件中增加时间戳的显示功能，可方便按照时间对数据进行恢复。但该方式可能会与AOF解析器不兼容，所以默认值为no，不开启
#### 4.3.5 AOF持久化过程
![](img/15.png)
AOF详细的持久化过程如下：
+ Redis接收到的写操作命令并不是直接追加到磁盘的AOF文件的，而是将每一条写按照Redis通讯协议格式暂时添加到AOF缓冲区aof_buf。
+ 根据设置的数据同步策略，当同步条件满足时，再将缓冲区中的数据一次性写入磁盘的 AOF文件，以减少磁盘IO次数，提高性能。
+ 当磁盘的AOF文件大小达到了Rewrite条件时，redis-server主进程会fork出一个子进程bgrewriteaof，由该子进程完成Rewrite过程。
+ 子进程bgrewriteaof首先对该磁盘AOF文件进行Rewrite计算，将计算结果写入到一个临时文件，全部写入完毕后，再rename该临时文件为磁盘文件的原名称，覆盖原文件。
+ 如果在Rewrite过程中又有写操作命令追加，那么这些数据会暂时写入aof_rewrite_buf缓冲区。等将全部Rewrite计算结果写入临时文件后，会先将aof_rewrite_buf缓冲区中的数据写入临时文件，然后再rename为磁盘文件的原名称，覆盖原文件。
### 4.4 RDB与AOF对比
#### 4.4.1 RDB的优势与不足
优势：
+ RDB文件较小 
+ 数据恢复较快

不足：
+ 数据安全性较差 
+ 写时复制会降低性能
+ RDB文件可读性较差
#### 4.4.2 AOF的优势与不足
优势：
+ 数据安全性高 
+ AOF文件可读性强

不足：
+ AOF文件较大
+ 写操作会影响性能
+ 数据恢复较慢
#### 4.4.3 持久化技术选型
+ 官方推荐使用RDB与AOF混合式持久化。
+ 若对数据安全性要求不高，则推荐使用纯RDB持久化方式。
+ 不推荐使用纯AOF持久化方式。
+ 若Redis仅用于缓存，则无需使用任何持久化技术。

## 五 集群
### 5.1 主从集群搭建
Redis的主从集群是一个“一主多从”的读写分离集群。集群中的Master节点负责处理客户端的读写请求，而Slave节点仅能处理客户端的读请求。只所以要将集群搭建为读写分离模式，主要原因是，对于数据库集群，写操作压力一般都较小，压力大多数来自于读操作请求。所以，只有一个节点负责处理写操作请求即可。
#### 5.1.1 伪集群搭建与配置
在采用单线程IO模型时，为了提高处理器的利用率，一般会在一个主机中安装多台Redis，
构建一个Redis主从伪集群。当然，搭建伪集群的另一个场景是，在学习Redis，而学习用
的主机内存不足以创建多个虚拟机。

下面要搭建的读写分离伪集群包含一个Master与两个Slave。它们的端口号分别是：6380、
6381、6382。
+ `mkdir cluster; cp redis.conf cluster;`	// 创建集群目录，复制配置文件
+ `masterauth`	// 搭建主从集群，由于每个主机都有可能会是Master，所以最好不要设置requirepass。如果真需要设置，一定要每个主机的密码都设置为相同的。masterauth用于指定当前slave访问master时的访问密码
+ `repl-disable-tcp-nodelay`	// 设置为yes则禁用tcp-nodelay，此时Master与Slave间的通信会产生延迟，但使用的TCP包数量会较少，占用的网络带宽会较小。相反，如果设置为no，则网络延迟会变小，但使用的TCP包数量会较多，相应占用的网络带宽会大
+ `touch redis6380.conf redis6381.conf redis6382.conf`	// 添加主从配置文件，如下所示

+ ```shell
    include redis.conf
    pidfile /var/run/redis_6380.pid
    port 6380
    dbfilename dump6380.rdb
    appendfilename appendonly6380.aof
    replica-priority 90
	# logfile access6380.log
	```

+ `slaveof Ip Port`	// 指定上级主机
+ `info replication`	// 查看当前连接的Redis客户端的状态信息
#### 5.1.2 分级管理
若Redis主从集群中的Slave较多时，它们的数据同步过程会对Master形成较大的性能压力。此时可以对这些Slave进行分级管理。设置方式很简单，只需要让低级别Slave指定其 `slaveof`的主机为其上一级Slave即可。不过，上一级Slave的状态仍为Slave，只不过其是更上一级的Slave。![](img/16.png)
#### 5.1.3 容灾冷处理
在Master/Slave的Redis集群中，若Master出现宕机怎么办呢？有两种处理方式，一种是通过手工角色调整，使Slave晋升为Master的冷处理；一种是使用哨兵模式，实现Redis集群的高可用，即热处理。

无论Master是否宕机，Slave都可通过`slaveof no one`将自己由Slave晋升为Master。如果其原本就有下一级的Slave，那么，其就直接变为了这些Slave的真正的Master了。而原来的Master也会失去这个原来的Slave。
### 5.2 主从复制原理
#### 5.2.1 主从复制过程
1. 保存Master地址：当Slave接收到`slaveof`指令后，Slave会立即将新的Master的地址保存下来。
2. 建立连接：Slave中维护着一个定时任务，该定时任务会尝试着与该Master建立socket连接。如果连接无法建立，则其会不断定时重试，直到连接成功或接收到`slaveof no one`指令。
3. Slave发送ping命令：连接建立成功后，Slave会发送ping命令进行首次通信。如果Slave没有收到Master的回复，则Slave会主动断开连接，下次的定时任务会重新尝试连接。
4. 对Slave身份验证：如果Master到了Slave的ping命令，并不会立即对其进行回复，而是会先进行身份验证。如果验证失败，则会发送消息拒绝连接；如果验证成功，则向Slave发送连接成功响应。
5. Master持久化：首次通信成功后，Slave会向Master发送数据同步请求。当Master接收到请求后，会fork出一个子进程，让子进程以异步方式立即进行持久化。
6. 数据发送：持久化完毕后Master会再fork出一个子进程，让该子进程以异步方式将数据发送给Slave。Slave会将接收到的数据不断写入到本地的持久化文件中。在Slave数据同步过程中，Master的主进程仍在不断地接受着客户端的写操作，且不仅将新的数据写入到了Master内存，同时也写入到了同步缓存。当Master的持久化文件中的数据发送完毕后，Master会再将同步缓存中新的数据发送给Slave，由Slave将其写入到本地持久化文件中。
7. Slave恢复内存数据：当Slave与Master的数据同步完成后，Slave就会读取本地的持久化文件，将其恢复到本地内存，然后就可以对外提供读服务了。
8. 持续增量复制：在Slave对外提供服务过程中，Master会持续不断的将新的数据以增量方式发送给Slave，以保证主从数据的一致性。
#### 5.2.2 数据同步演变过程
(1) sync同步
Redis2.8版本之前，首次通信成功后，Slave会向Master发送sync数据同步请求。然后Master就会将其所有数据全部发送给Slave，由Slave保存到其本地的持久化文件中。这个过
程称为全量复制。

但这里存在一个问题：在全量复制过程中可能会出现由于网络抖动而导致复制过程中断。当网络恢复后，Slave与Master重新连接成功，此时Slave会重新发送sync请求，然后会从头开始全量复制。

由于全量复制过程非常耗时，所以期间出现网络抖动的概率很高。而中断后的从头开始不仅需要消耗大量的系统资源、网络带宽，而且可能会出现长时间无法完成全量复制的情况。

(2) psync同步
Redis2.8版本之后，全量复制采用了psync（Partial Sync，不完全同步）同步策略。当全量复制过程出现由于网络抖动而导致复制过程中断时，当重新连接成功后，复制过程可以“断点续传”。即从断开位置开始继续复制，而不用从头再来。这就大大提升了性能。为了实现 psync，整个系统做了三个大的变化：
+ 复制偏移量：系统为每个要传送数据进行了编号，该编号从0开始，每个字节一个编号。该编号称为复制偏移量。参与复制的主从节点都会维护该复制偏移量。Master每发送过一个字节数据后就会进行累计。统计信息通过`info replication`的`master_repl_offset`可查看到。同时，Slave会定时向Master上报其自身已完成的复制偏移量给Master，所以Master也会保存Slave的复制偏移量offset。Slave在接收到Master的数据后，也会累计接收到的偏移量。统计信息通过`info replication`的`slave_repl_offset`可查看到。
+ 主节点复制ID：当Master启动后就会动态生成一个长度为40位的16进制字符串作为当前Master的复制ID，该ID是在进行数据同步时Slave识别Master使用的。通过`info replication`的`master_replid`属性可查看到。
+ 复制积压缓冲区：当Master有连接的Slave时，在Master中就会创建并维护一个队列backlog，默认大小为1MB，该队列称为复制积压缓冲区。Master接收到了写操作数据不仅会写入到Master主存，而且还会写入到复制积压缓冲区。其作用就是用于保存最近操作的数据，以备“断点续传”时做数据补偿，防止数据丢失。
+ psync同步过程：psync是一个由Slave提交的命令，其格式为`psync master_replid repl_offset`，表示当前Slave要从指定的Master中的repl_offset+1处开始复制。repl_offset表示当前Slave已经完成复制的数据的offset。该命令保证了“断点续传”的实现。在第一次开始复制时，Slave并不知道Master的动态ID，并且一定是从头开始复制，所以其提交的psync命令为`psync ? -1`。
+ 存在的问题：在psync数据同步过程中，若Slave重启，在Slave内存中保存的Master的动态ID与续传offset都会消失，“断点续传”将无法进行，从而只能进行全量复制。在psync数据同步过程中，Master宕机后Slave会发生“易主”，从而导致Slave需要重新进行全量复制。

(3) psync同步的改进
+ 解决Slave重启问题：改进后的psync将Master的动态ID直接写入到了Slave的持久化文件中。
+ 解决Slave易主问题：由于改进后的psync中每个Slave都在本地保存了当前Master的动态ID，所以当Slave晋升为新的Master后，其本地仍保存有之前Master的动态ID。而这一点也恰恰为解决“Slave易主”问题提供了条件。通过Master的`info replicaton`中的`master_replid2`可查看到。如果尚未发生过易主，则该值为40个0。

(4) 无盘操作
Redis6.0对同步过程又进行了改进，提出了“无盘全量同步”与“无盘加载”策略，避免了耗时的IO操作。
+ 无盘全量同步：Master的主进程fork出的子进程直接将内存中的数据发送给Slave，无需经过磁盘。
+ 无盘加载：Slave在接收到Master发送来的数据后不需要将其写入到磁盘文件，而是直接写入到内存，这样Slave就可快速完成数据恢复。

(5) 共享复制积压缓冲区
Redis7.0版本对复制积压缓冲区进行了改进，让各个Slave的发送缓冲区共享复制积压缓冲区。这使得复制积压缓冲区的作用，除了可以保障数据的安全性外，还作为所有Slave的发送缓冲区，充分利用了复制积压缓冲区。
### 5.3 哨兵机制实现
#### 5.3.1 简介
对于Master宕机后的冷处理方式是无法实现高可用的。Redis从2.6版本开始提供了高可用的Sentinel哨兵机制。在集群中再引入一个节点，该节点充当Sentinel哨兵，用于监视Master的运行状态，并在Master宕机后自动指定一个Slave作为新的Master。

不过，此时的Sentinel哨兵又成为了一个单点故障点：若哨兵发生宕机，整个集群将瘫痪。所以为了解决Sentinel的单点问题，又要为Sentinel创建一个集群。每个Sentinel都会定时向Master发送心跳，如果Master在有效时间内向它们都进行了响应，则说明Master是活着的。如果Sentinel中有`quorum`个哨兵没有收到响应，那么就认为Master已经宕机，然后会有一个Sentinel做故障转移。即将原来的某一个Slave晋升为Master。
#### 5.3.2 Redis高可用集群搭建
在“不差钱”的情况下，可以让Sentinel占用独立的主机，即在Redis主机上只启动Redis进程，在Sentinel主机上只启动Sentinel进程。下面要搭建一个“一主二从三哨兵”的高可用伪集群，即这些角色全部安装运行在一台主机上。“一主二从”使用前面的主从集群，下面仅搭建一个Sentinel伪集群，端口号分别为：26380、26381、26382。
+ `cp sentinel.conf cluster`	// 复制公共配置文件
+ `sentinel monitor`	// 该配置用于指定Sentinel要监控的Master，并为Master起了一个名字。同时指定Sentinel集群中决定该Master”客观下线状态”判断的法定Sentinel数量`quorum`。`quorum`的另一个用途与Sentinel的Leader选举有关。要求至少要有max(quorum, sentinelNum/2+1)个Sentinel参与，选举才能进行
+ `sentinel auth-pass`	// 如果Redis主从集群中的主机设置了访问密码，那么就需要指定Master的主机名与访问密码，以便Sentinel监控Master
+ `touch sentinel26380.conf sentinel26381.conf sentinel26382.conf`	// 添加哨兵集群配置文件，如下所示

+ ```shell
    include sentinel.conf
    pidfile /var/run/sentinel_26380.pid
    port 26380
    sentinel monitor mymaster 127.0.0.1 6380 2
	# logfile access26380.log
	```
	
+ `redis-sentinel Sentinel.conf; redis-server Sentine.conf --sentinel`	// 启动哨兵
+ `redis-cli -p Port info sentinel`	// 查看哨兵信息
#### 5.3.3 Sentinel优化配置
+ `sentinel down-after-milliseconds`	// 定期发送ping命令来判断Master、Slave及其它Sentinel是否存活
+ `sentinel parallel-syncs`	// 指定在故障转移期间，即老Master出现问题，新Master晋升后，允许多少个Slave同时从新Master进行数据同步。默认值为1表示所有Slave逐个从新Master进行数据同步
+ `sentinel failover-timeout`	// 指定故障转移的超时时间
+ `sentinel deny-scripts-reconfig`	// 指定是否可以通过命令`sentinel set`动态修改`notification-script`与`client-reconfig-script`两个脚本。默认是不能的。这两个脚本如果允许动态修改，可能会引发安全问题
+ `sentinel set Config`	// 在redis-cli中修改配置文件信息
### 5.4 哨兵机制原理
#### 5.4.1 三个定时任务
+ info任务：每个Sentinel节点每10秒就会向Redis集群中的每个节点发送info命令，以获得最新的Redis拓扑结构
+ 心跳任务：每个Sentinel节点每1秒就会向所有Redis节点及其它Sentinel节点发送一条ping命令，以检测这些节点的存活状态。该任务是判断节点在线状态的重要依据
+ 发布/订阅任务：每个Sentinel节点在启动时都会向所有Redis节点订阅`__sentinel__:hello`主题的信息，当Redis节点中该主题的信息发生了变化，就会立即通知到所有订阅者。启动后，每个Sentinel节点每2秒就会向每个Redis节点发布一条`__sentinel__:hello`主题的信息，该信息是当前Sentinel对每个Redis节点在线状态的判断结果及当前Sentinel节点信息
#### 5.4.2 Redis节点下线判断
+ 主观下线：每个Sentinel节点每秒就会向每个Redis节点发送ping心跳检测，如果Sentinel在`down-after-milliseconds`时间内没有收到某Redis节点的回复，则Sentinel节点就会对该Redis节点做出“下线状态”的判断。这个判断仅仅是当前Sentinel节点的“一家之言”，所以称为主观下线
+ 客观下线：当Sentinel主观下线的节点是Master时，该Sentinel节点会向每个其它Sentinel节点发送`sentinel is-master-down-by-addr`命令，以询问其对Master在线状态的判断结果。这些Sentinel节点在收到命令后会向这个发问Sentinel节点响应0（在线）或1（下线）。当Sentinel收到超过`quorum`个下线判断后，就会对Master做出客观下线判断
#### 5.4.3 Sentinel Leader选举
当Sentinel节点对Master做出客观下线判断后会由Sentinel Leader来完成后续的故障转
移，即Sentinel集群中的节点也并非是对等节点，是存在Leader与Follower的。Sentinel集群的Leader选举是通过Raft算法实现的。Raft算法比较复杂，这里仅简单介绍一下大致思路。

每个选举参与者都具有当选Leader的资格，当其完成了“客观下线”判断后，就会立即“毛遂自荐”推选自己做Leader，然后将自己的提案发送给所有参与者。其它参与者在收到提案后，只要自己手中的选票没有投出去，其就会立即通过该提案并将同意结果反馈给提案者，后续再过来的提案会由于该参与者没有了选票而被拒绝。当提案者收到了同意反馈数量大于等于max(quorum, sentinelNum/2+1)时，该提案者当选Leader。

说明：
+ 在网络没有问题的前提下，基本就是谁先做出了“客观下线”判断，谁就会首先发起Sentinel Leader的选举，谁就会得到大多数参与者的支持，谁就会当选Leader
+ Sentinel Leader选举会在次故障转移发生之前进行。
+ 故障转移结束后Sentinel不再维护这种Leader-Follower关系，即Leader不再存在
#### 5.4.4 Master选择算法
在进行故障转移时，Sentinel Leader需要从所有Redis的Slave节点中选择出新的Master。其选择算法为：
1. 过滤掉所有主观下线的，或心跳没有响应Sentinel的，或`replica-priority`值为0的Redis节点
2. 在剩余Redis节点中选择出`replica-priority`最小的的节点列表。如果只有一个节点，则直接返回，否则，继续
3. 从优先级相同的节点列表中选择复制偏移量最大的节点。如果只有一个节点，则直接返
回，否则，继续
4. 从复制偏移值量相同的节点列表中选择动态ID最小的节点返回
#### 5.4.5 故障转移过程
Sentinel Leader负责整个故障转移过程，经历了如下步骤：
1. Sentinel Leader根据Master选择算法选择出一个Slave节点作为新的Master
2. Sentinel Leader向新Master节点发送`slaveof no one`指令，使其晋升为Master
3. Sentinel Leader向新Master发送`info replication`指令，获取到Master的动态ID
4. Sentinel Leader向其余Redis节点发送消息，以告知它们新Master的动态ID
5. Sentinel Leader向其余Redis节点发送`slaveof MasterIp MasterPort`指令，使它们成为新Master的Slave
6. Sentinel Leader从所有Slave节点中每次选择出`parallel-syncs`个Slave从新Master同步数据，直至所有Slave全部同步完毕
#### 5.4.6 节点上线
+ 原Redis节点上线：无论是原下线的Master节点还是原下线的Slave节点，只要是原Redis集群中的节点上线，只需启动Redis即可。因为每个Sentinel中都保存有原来其监控的所有Redis节点列表，Sentinel会定时查看这些Redis节点是否恢复。如果查看到其已经恢复，则会命其从当前Master进行数据同步。不过，如果是原Master上线，在新Master晋升后Sentinel Leader会立即先将原Master节点更新为Slave，然后才会定时查看其是否恢复
+ 新Redis节点上线：如果需要在Redis集群中添加一个新的节点，其未曾出现在Redis集群中，则上线操作只能手工完成。即添加者在添加之前必须知道当前Master是谁，然后在新节点启动后运行`slaveof`命令加入集群
+ Sentinel节点上线：如果要添加的是Sentinel节点，无论其是否曾经出现在Sentinel集群中，都需要手工完成。即添加者在添加之前必须知道当前Master是谁，然后在配置文件中修改`sentinel monitor`属性，之后启动Sentinel即可
### 5.5 CAP定理
#### 5.5.1 概念
CAP定理指的是在一个分布式系统中，一致性Consistency、可用性Availability、分区容
错性Partition tolerance，三者不可兼得。
+ 一致性：分布式系统中多个主机之间是否能够保持数据一致的特性。即当系统数据发生更新操作后，各个主机中的数据仍然处于一致的状态
+ 可用性：系统提供的服务必须一直处于可用的状态，即对于用户的每一个请求，系统总是可以在有限的时间内对用户做出响应
+ 分区容错性：分布式系统在遇到任何网络分区故障时，仍能够保证对外提供满足一致性和可用性的服务
#### 5.5.2 定理
CAP定理的内容是：对于分布式系统，网络环境相对是不可控的，出现网络分区是不可避免的，因此系统必须具备分区容错性。但系统不能同时保证一致性与可用性。即要么CP，要么AP。
#### 5.5.3 BASE理论
BASE是Basically Available（基本可用）、Soft state（软状态）和Eventually consistent（最终一致性）三个短语的简写，BASE是对CAP中一致性和可用性权衡的结果，其来源于对大规模互联网系统分布式实践的结论，是基于CAP定理逐步演化而来的。

BASE理论的核心思想是：即使无法做到强一致性，但每个系统都可以根据自身的业务特点，采用适当的方式来使系统达到最终一致性。
+ 基本可用：指分布式系统在出现不可预知故障的时候，允许损失部分可用性
+ 软状态：指允许系统数据存在的中间状态，并认为该中间状态的存在不会影响系统的整体可用性，即允许系统主机间进行数据同步的过程存在一定延时
+ 最终一致性：强调的是系统中所有的数据副本，在经过一段时间的同步后，最终能够达到一个一致的状态。因此，最终一致性的本质是需要系统保证最终数据能够达到一致，而不需要保证系统数据的实时一致性
### 5.6 Raft算法
#### 5.6.1 基础
Raft算法是一种通过对日志复制管理来达到集群节点一致性的算法。这个日志复制管理发生在集群节点中的Leader与Followers之间。Raft通过选举出的Leader节点负责管理日志复制过程，以实现各个节点间数据的一致性。
#### 5.6.2 角色、任期及角色转变
![](img/17.png)
在Raft中，节点有三种角色：
+ Leader：唯一负责处理客户端写请求的节点；也可以处理客户端读请求；同时负责日志复制工作
+ Candidate：选举的候选人，其可能会成为Leader。是一个选举中的过程角色
+ Follower：可以处理客户端读请求；负责同步来自于Leader的日志；当接收到其它Cadidate的投票请求后可以进行投票；当发现Leader挂了，其会转变为Candidate发起Leader选举
#### 5.6.3 Leader选举
(1) 我要选举
若Follower在心跳超时范围内没有接收到来自于Leader的心跳，则认为Leader挂了。此时其首先会使其本地term增一。然后Follower会完成以下步骤：
+ 此时若接收到了其它Candidate的投票请求，则会将选票投给这个Candidate
+ 由Follower 转变为Candidate
+ 若之前尚未投票，则向自己投一票
+ 向其它节点发出投票请求，然后等待响应

(2) 我要投票
Follower在接收到投票请求后，其会根据以下情况来判断是否投票：
+ 发来投票请求的Candidate的term不能小于我的term
+ 在我当前term内，我的选票还没有投出去
+ 若接收到多个Candidate的请求，我将采取first-come-first-served 方式投票

(3) 等待响应
当一个Candidate发出投票请求后会等待其它节点的响应结果。这个响应结果可能有三种情况：
+ 收到过半选票，成为新的Leader。然后会将消息广播给所有其它节点，以告诉大家我是
新的Leader了
+ 接收到别的Candidate发来的新Leader通知，比较了新Leader的term并不比自己的term小，则自己转变为Follower
+ 经过一段时间后，没有收到过半选票，也没有收到新Leader通知，则重新发起选举

(4) 选举时机
在很多时候，当Leader真的挂了，Follower几乎同时会感知到，所以它们几乎同时会变为Candidate发起新的选举。此时就可能会出现较多Candidate票数相同的情况，即无法选举
出Leader。

为了防止这种情况的发生，Raft算法其采用了`randomized election timeouts`策略来解决
这个问题。其会为这些Follower随机分配一个选举发起时间election timeout，这个timeout在150-300ms范围内。只有到达了election timeout时间的Follower才能转变为Candidate，否则等待。那么election timeout较小的Follower则会转变为Candidate然后先发起选举，一般情况下其会优先获取到过半选票成为新的 leader。
#### 5.6.4 数据同步
(1) 状态机
Raft算法一致性的实现，是基于日志复制状态机的。状态机的最大特征是，不同Server中的状态机若当前状态相同，然后接受了相同的输入，则一定会得到相同的输出。![](img/18.png)

(2) 处理流程
当Leader接收到Client的写操作请求后，大体会经历以下流程：
+ Leader在接收到Client的写操作请求后，Leader会将数据与term封装为一个box，并随着下一次心跳发送给所有Followers，以征求大家对该box的意见。同时在本地将数据封装为日志
+ Follower在接收到来自Leader的box后首先会比较该box的term与本地记录的曾接受过的box的最大term，只要不比自己的小就接受该box，并向Leader回复同意。同时会将该box中的数据封装为日志
+ 当Leader接收到过半同意响应后，会将日志commit到自己的状态机，状态机会输出一个结果，同时日志状态变为了committed
+ 同时Leader还会通知所有Follower将日志commit到它们本地的状态机，日志状态变为了committed
+ 在commit通知发出的同时，Leader也会向Client发出成功处理的响应

(3) AP 支持
![](img/19.png)
Log由term index、log index及command构成。为了保证可用性，各个节点中的日志可以不完全相同，但Leader会不断给Follower发送box，以使各个节点的Log最终达到相同。即Raft算法不是强一致性的，而是最终一致的。
#### 5.6.5 脑裂
Raft集群存在脑裂问题。在多机房部署中，由于网络连接问题，很容易形成多个分区。而多分区的形成，很容易产生脑裂，从而导致数据不一致。
#### 5.6.6 Leader宕机处理
(1) 请求到达前Leader挂了
Client发送写操作请求到达Leader之前Leader就挂了，因为请求还没有到达集群，所以这个请求对于集群来说就没有存在过，对集群数据的一致性没有任何影响。Leader挂了之后，会选举产生新的Leader。由于Stale Leader并未向Client发送成功处理响应，所以Client会重新发送该写操作请求。

(2) 未开始同步数据前Leader挂了
Client发送写操作请求给Leader，请求到达Leader后，Leader还没有开始向Followers发出数据Leader就挂了。这时集群会选举产生新的Leader。Stale Leader重启后会作为Follower 重新加入集群，并同步新Leader中的数据以保证数据一致性。之前接收到Client的数据被丢弃。由于Stale Leader并未向Client发送成功处理响应，所以Client会重新发送该写操作请求。

(3) 同步完部分后 Leader 挂了
Client发送写操作请求给Leader，Leader接收完数据后向所有Follower发送数据。在部分Follower接收到数据后Leader挂了。由于Leader挂了，就会发起新的Leader选举。
+ 若Leader产生于已完成数据接收的Follower，其会继续将前面接收到的写操作请求转换为日志，并写入到本地状态机，并向所有Flollower发出询问。在获取过半同意响应后会向所有Followers发送commit指令，同时向Client进行响应
+ 若Leader产生于尚未完成数据接收的Follower，那么原来已完成接收的Follower则会放弃曾接收到的数据。由于Client没有接收到响应，所以Client会重新发送该写操作请求

(4) commit通知发出后Leader挂了
Client发送写操作请求给Leader，Leader也成功向所有Followers发出commit 指令，并向Client发出响应后，Leader挂了。由于Stale Leader已经向Client发送成功接收响应，且commit通知已经发出，说明这个写操作请求已经被Server成功处理。
#### 5.6.7 Raft算法动画演示
[Raft动画演示](https://thesecretlivesofdata.com/raft/)

## 六 分布式系统
### 6.1 数据分区算法
分布式数据库系统会根据不同的数据分区算法，将数据分散存储到不同的数据库服务器节点上，每个节点管理着整个数据集合中的一个子集。常见的数据分区规则有两大类：顺序分区与哈希分区。![](img/20.png)
#### 6.1.1 顺序分区
+ 轮询分区算法：每产生一个数据，就依次分配到不同的节点。该算法适合于数据问题不确定的场景。其分配的结果是，在数据总量非常庞大的情况下，每个节点中数据是很平均的。但生产者与数据节点间的连接要长时间保持
+ 时间片轮转分区算法：在某固定长度的时间片内的数据都会分配到一个节点。时间片结束，再产生的数据就会被分配到下一个节点。这些节点会被依次轮转分配数据。该算法可能会出现节点数据不平均的情况（因为每个时间片内产生的数据量可能是不同的）。但生产者与节点间的连接只需占用当前正在使用的这个就可以，其它连接使用完毕后就立即释放
+ 数据块分区算法：在整体数据总量确定的情况下，根据各个节点的存储能力，可以将连接的某一整块数据分配到某一节点
+ 业务主题分区算法：数据可根据不同的业务主题，分配到不同的节点
#### 6.1.2 哈希分区
+ 节点取模分区算法
+ 一致性哈希分区算法
+ 虚拟槽分区算法

## 七 缓存
### 7.1 高并发问题
#### 7.1.1 缓存穿透
当用户访问的数据既不在缓存也不在数据库中时，就会导致每个用户查询都会“穿透”缓存“直抵”数据库。这种情况就称为缓存穿透。当高并发的访问请求到达时，缓存穿透不仅增加了响应时间，而且还会引发对DBMS的高并发查询，这种高并发查询很可能会导致DBMS的崩溃。缓存穿透产生的主要原因有两个：一是在数据库中没有相应的查询结果，二是查询结果为空时，不对查询结果进行缓存。所以，针对以上两点，解决方案也有两个：
+ 对非法请求进行限制
+ 对结果为空的查询给出默认值
#### 7.1.2 缓存击穿
对于某一个缓存，在高并发情况下若其访问量特别巨大，当该缓存的有效时限到达时，可能会出现大量的访问都要重建该缓存，即这些访问请求发现缓存中没有该数据，则立即到DBMS中进行查询，那么这就有可能会引发对DBMS的高并发查询，从而接导致DBMS的崩溃。这种情况称为缓存击穿，而该缓存数据称为热点数据。对于缓存击穿的解决方案，较典型的是使用“双重检测锁”机制。
#### 7.1.3 缓存雪崩
对于缓存中的数据，很多都是有过期时间的。若大量缓存的过期时间在同一很短的时间段内几乎同时到达，那么在高并发访问场景下就可能会引发对DBMS的高并发查询，而这将可能直接导致DBMS的崩溃。这种情况称为缓存雪崩。

对于缓存雪崩没有很直接的解决方案，最好的解决方案就是预防，即提前规划好缓存的过期时间。要么就是让缓存永久有效，当DB中数据发生变化时清除相应的缓存。如果DBMS采用的是分布式部署，则将热点数据均匀分布在不同数据库节点中，将可能到来的访问负载均衡开来。
#### 7.1.4 数据库缓存双写不一致
以上三种情况都是针对高并发读场景中可能会出现的问题，而数据库缓存双写不一致问题，则是在高并发写场景下可能会出现的问题。对于数据库缓存双写不一致问题，以下两种场景下均有可能会发生：

(1) “修改DB更新缓存”场景
对于具有缓存warmup功能的系统，DBMS中常用数据的变更，都会引发缓存中相关数据的更新。在高并发写请求场景下，若多个请求要对DBMS中同一个数据进行修改，修改后还需要更新缓存中相关数据，那么就有可能会出现缓存与数据库中数据不一致的情况。![](img/21.png)

(2) “修改DB删除缓存”场景
在很多系统中是没有缓存warmup功能的，为了保持缓存与数据库数据的一致性，一般都是在对数据库执行了写操作后，就会删除相应缓存。在高并发读写请求场景下，若这些请求对DBMS中同一个数据的操作既包含写也包含读，且修改后还要删除缓存中相关数据，那么就有可能会出现缓存与数据库中数据不一致的情况。![](img/22.png)

(3) 解决方案：延迟双删
延迟双删方案是专门针对于“修改DB删除缓存”场景的解决方案。但该方案并不能彻底解决数据不一致的状况，其只可能降低发生数据不一致的概率。延迟双删方案是指，在写操作完毕后会立即执行一次缓存的删除操作，然后再停上一段时间（一般为几秒）后再进行一次删除。而两次删除中间的间隔时长，要大于一次缓存写操作的时长。

(4) 解决方案：队列
以上两种场景中，之所以会出现数据库与缓存中数据不一致，主要是因为对请求的处理出现了并行。只要将请求写入到一个统一的队列，只有处理完一个请求后才可处理下一个请求，即使系统对用户请求的处理串行化，就可以完全解决数据不一致的问题。

(5) 解决方案：分布式锁
使用队列的串行化虽然可以解决数据库与缓存中数据不一致，但系统失去了并发性，降低了性能。使用分布式锁可以在不影响并发性的前提下，协调各处理线程间的关系，使数据库与缓存中的数据达成一致性。只需要对数据库中的这个共享数据的访问通过分布式锁来协调对其的操作访问即可。

## 八 Lua脚本
## 九 分布式锁
### 9.1 分布式锁的工作原理
当有多个线程要访问某一个共享资源（DBMS中的数据或Redis中的数据，或共享文件等）时，为了达到协调多个线程的同步访问，此时就需要使用分布式锁了。

为了达到同步访问的目的，规定，让这些线程在访问共享资源之前先要获取到一个令牌token，只有具有令牌的线程才可以访问共享资源。这个令牌就是通过各种技术实现的分布式锁。而这个分布锁是一种“互斥资源”，即只有一个。只要有线程抢到了锁，那么其它线程只能等待，直到锁被释放或等待超时。
### 9.2 setnx实现方式
该实现方式主要是通过`setnx`命令完成的。其基本原理是，setnx只有在指定key不存在时才能执行成功，分布式系统中的哪个节点抢先成功执行了setnx，谁就抢到了锁，谁就拥有了对共享资源的操作权限。当然，其它节点只能等待锁的释放。一旦拥有锁的节点对共享资源操作完毕，其就可以主动删除该key，即释放锁。然后其它节点就可重新使用setnx命令抢注该key，即抢注锁。

但是，若处理当前请求的节点主机在执行完“添加锁”语句后突然宕机，其finally中的释放锁代码根本就没有执行，那么，其它客户端通过其它节点主机申请资源时，将会由于无法获得锁而永久性阻塞。
### 9.3 为锁添加过期时间
为了解决前述方式中存在的问题，可以为锁添加过期时间，这样就不会出现锁被某节点主机永久性占用的情况，即不会出现节点被永久性阻塞的情况。不过，为key添加过期时间的方式有两种：一种是通过`expire`命令为key指定过期时间，还有一种是在`setnx`命令中直接给出该key的过期时间。第一种方式中setnx与expire命令是分别执行的，不具备原子性，仍然可能会出现问题。而第二种方式则是直接在setnx中完成了两步操作，具有原子性。故应采用第二种方式。

但是，假设指定锁的过期时间为5s，如果请求a的处理时间超过了5s（假设6s)，而当5s过去后，这个锁自动过期了。由于锁已过期，另一个请求b申请到了锁。此时如果请求a处理完了，回来继续执行程序，请求a就会把请求b设置的锁给删除了。此时其它请求就可申请到锁，并与请求b同时访问共享资源，很可能会引发数据的不一致。
### 9.4 为锁添加标识
之所以会出现那种锁被误删的情况，主要是因为所有客户端添加的锁的value值完全相同，而我们想要的效果是“谁添加的锁，该锁只能由谁来删”。为了实现这个效果，为每个申请锁的客户端随机生成一个UUID，使用这个UUID作为该客户端的标识，然后将该UUID作为该客户端申请到的锁的value。在删除锁时，只有在发起当前删除操作的客户端的UUID与锁的value相同时才可以。
### 9.5 添加Lua脚本
对客户端身份的判断与删除锁操作的合并，是没有专门的原子性命令的。此时可以通过Lua脚本来实现它们的原子性。而对Lua脚本的执行，可以通过`eval`命令来完成。不过，eval命令在RedisTemplate中没有对应的方法，而Jedis中具有该同名方法。所以，需要在代码中首先获取到Jedis客户端，然后才能调用`jedis.eval()`。

以上方式仍然是存在问题的：请求a的锁过期，但其业务还未执行完毕；请求b申请到了锁，其也正在处理业务。如果此时两个请求都同时修改了共享的库存数据，那么就又会出现数据不一致的问题，即仍然存在并发问题。

对于该问题，可以采用“锁续约”方式解决。即在当前业务进程开始执行时，fork出一个子进程，用于启动一个定时任务。该定时任务的定时时间小于锁的过期时间，其会定时查看处理当前请求的业务进程的锁是否已被删除。如果已被删除，则子进程结束；如果未被删除，说明当前请求的业务还未处理完毕，则将锁的时间重新设置为“原过期时间”。
### 9.6 Redisson可重入锁
Redisson内部使用Lua脚本实现了对可重入锁的添加、重入、续约、释放。Redisson需要用户为锁指定一个key，但无需为锁指定过期时间，因为它有默认过期时间。由于该锁具有“可重入”功能，所以Redisson会为该锁生成一个计数器，记录一个线程重入锁的次数。

但是，在Redis主从集群中，假设节点A为Master，节点B、C为Slave。如果一个请求a在处
理时申请锁，即向节点A添加一个key。当节点A收到请求后写入key成功，然后会立即向处理a请求的应用服务器响应，然后会向Slave同步该key。不过，在同步还未开始时，节点A宕机，节点B晋升为Master。此时正好有一个请求b申请锁，由于节点B中并没有该key，所以该key写入成功，然后会立即向处理b请求的应用服务器响应。因此它们都可同时对共享数据进行处理。
### 9.7 Redisson红锁
Redisson红锁可以防止主从集群锁丢失问题。Redisson红锁要求，必须要构建出至少三个Redis主从集群。若一个请求要申请锁，必须向所有主从集群中提交key写入请求，只有当大多数集群锁写入成功后，该锁才算申请成功。

无论前面使用的是哪种锁，它们解决并发问题的思路都是相同的，那就将所有请求通过锁实现串行化。而串行化在高并发场景下势必会引发性能问题。
### 9.8 分段锁
解决锁的串行化引发的性能问题的方案就是使访问并行化。将要共享访问的一个资源拆分为多个共享访问资源，这样就会将一把锁的需求转变为多把锁，实现并行化。
### 9.9 Redisson详解
在生产中，对于Redisson使用最多的场景就是其分布式锁RLock。当然，RLock仅仅是Redisson的线程同步方案之一。Redisson提供了8种线程同步方案，用户可针对不同场景选用不同方案。

需要注意的是，为了避免锁到期但业务逻辑没有执行完毕而引发的多个线程同时访问共享资源的情况发生，Redisson内部为锁提供了一个监控锁的看门狗watch dog，其会在锁到期前不断延长锁的到期时间，直到锁被主动释放。
#### 9.9.1 可重入锁
Redisson的分布式锁RLock是一种可重入锁。当一个线程获取到锁之后，这个线程可以再次获取本对象上的锁，而其他的线程是不可以的。
+ JDK中的ReentrantLock是可重入锁，其是通过AQS（抽象队列同步器）实现的锁机制
+ Synchronized也是可重入锁，其是通过监视器模式实现的锁机制
#### 9.9.2 公平锁
Redisson的可重入锁RLock默认是一种非公平锁，但也支持可重入公平锁FairLock。当有多个线程同时申请锁时，这些线程会进入到一个FIFO队列，只有队首元素才会获取到锁，其它元素等待。只有当锁被释放后，才会再将锁分配给当前的队首元素。
#### 9.9.3 联锁
Redisson分布式锁可以实现联锁MultiLock。当一个线程需要同时处理多个共享资源时，可使用联锁。即一次性申请多个锁，同时锁定多个共享资源。联锁可预防死锁。相当于对共享资源的申请实现了原子性：要么都申请到，只要缺少一个资源，则将申请到的所有资源全部释放。
#### 9.9.4 红锁
Redisson分布式锁可以实现红锁RedLock。红锁由多个锁构成，只有当这些锁中的大部分锁申请成功时，红锁才申请成功。红锁一般用于解决Redis主从集群锁丢失问题。红锁与联锁的区别是，红锁实现的是对一个共享资源的同步访问控制，而联锁实现的是多个共享资源的同步访问控制。
#### 9.9.5 读写锁
通过Redisson可以获取到读写锁RReadWriteLock。通过RReadWriteLock实例可分别获取到读锁RedissonReadLock与写锁RedissonWriteLock。读锁与写锁分别是实现了RLock的可重入锁。一个共享资源，在没有写锁的情况下，允许同时添加多个读锁。只要添加了写锁，任何读锁与写锁都不能再次添加。即读锁是共享锁，写锁为排他锁。
#### 9.9.6 信号量
通过Redisson可以获取到信号量RSemaphore。RSemaphore的常用场景有两种：一种是，无论谁添加的锁，任何其它线程都可以解锁，就可以使用RSemaphore。另外，当一个线程需要一次申请多个资源时，可使用RSemaphore。
#### 9.9.7 可过期信号量
通过Redisson可以获取到可过期信号量PermitExpirableSemaphore。该信号量是在RSemaphore基础上，为每个信号增加了一个过期时间，且每个信号都可以通过独立的ID来辨识。释放时也只能通过提交该ID才能释放。不过，一个线程每次只能申请一个信号量和释放一个信号量。这是与RSemaphore不同的地方。

该信号量为互斥信号量时，其就等同于可重入锁。或者说，可重入锁就相当于信号量为1的可过期信号量。注意，可过期信号量与可重入锁的区别：
+ 可重入锁：相当于用户每次只能申请1个信号量，且只有一个用户可以申请成功
+ 可过期信号量：用户每次只能申请1个信号量，但可以有多个用户申请成功
#### 9.9.8 分布式闭锁
通过Redisson可以获取到分布式闭锁RCountDownLatch，其与JDK的JUC中的闭锁CountDownLatch原理相同，用法类似。其常用于一个或者多个线程的执行必须在其它某些任务执行完毕的场景。例如，大规模分布式并行计算中，最终的合并计算必须基于很多并行计算的运行完毕。

闭锁中定义了一个计数器和一个阻塞队列。阻塞队列中存放着待执行的线程。每当一个并行任务执行完毕，计数器就减1。当计数器递减到0时就会唤醒阻塞队列的所有线程。通常使用Barrier队列解决该问题，而Barrier队列通常使用Zookeeper实现。