# redis

## NoSql入门和概述

### 入门概述

- 1.互联网时代背景下大机遇，为什么用nosql

	- 1.单机MySQL的美好时代

	  一个网站的访问量一般都不大，而且更多的是静态页面，动态交互类型的网站不多，用单个数据库完全可以轻松应对。
	  
	  上述架构下，数据存储的瓶颈是什么？
	  数据量的总大小一个机器放不下时
	  数据的索引（B+Tree）一个机器的内存放不下时
	  访问量（读写混合）一个实例不能承受

	- 2.Memcached（缓存）+MySQL+垂直拆分

	  随着访问量的上升，几乎发部分使用MySQL架构的网站在数据库上都开始出现性能问题，web程序不再仅仅专注在功能上，同时也在追求性能。程序员们开始大量的使用缓存技术来缓解数据库的压力，优化数据库的结构和索引。开始比较流行的是通过文件缓存来缓解数据库压力，但是访问量继续增大的时候，多台web机器通过文件缓存不能共享，大量的小文件缓存也带来了比较高的IO压力。因此，Memcached就成为了一个非常时尚的技术产品。

	- 3.Mysql主从读写分离

	  由于数据库的写入压力增加，Memcached只能缓解数据库的读取压力。读写集中在一个数据库上让数据库不堪重负，大部分网站开始使用主从复制技术来达到读写分离，以提高读写性能和读库的可扩展性。Mysql的master-slave模式成为这个时候的网站标配。

	- 4.分表分库+水平拆分+mysql集群

	  在Memcached的高速缓存，MySQL的主从复制，读写分离的基础之上，这是MySQL的主库的写压力开始出现瓶颈，而数据量的持续猛增，由于MyISAM使用表锁，在高并发下会出现严重的锁问题，大量的高并发MySQL应用开始使用InnoDB引擎代替MyISQM。
	  
	  同时开始流行使用分表分库来缓解写压力和数据增长的扩展问题。这个时候，分表分库成了一个热门技术，是面试的热门问题也是业界讨论的热门技术问题。也就是在这个时候，MySQL推出了还不太稳定的表分区，虽然MySQL推出了MySQL Cluster集群，但性能也不能很好满足互联网的要求，只能在高可靠性上提供了非常大的保证。

	- 5.MySQL的扩展性瓶颈

	  MySQL数据库也经常存储一些大文本字段，导致数据库表非常的大，在做数据库恢复的时候就导致非常的慢，不容易快速回复数据库。比如1000万4KB大小的文本就接近40GB的大小，如果能把这些数据从MySQL省去，MySQL将变得非常的小。关系数据库很强大，但是它并不能很好的应付所有的应用场景。MySQL的扩展性差（需要复杂的技术来实现），大数据下IO压力大，表结构更改困难，正式当前使用MySQL的开发人员面临的问题。

	- 6.今天是什么样子
	- 7.为什么用NoSQL

	  为什么使用NoSQL？
	  今天我们可以通过第三方平台（如Google，Facebook等）可以很容易的访问和抓取数据。用户个人信息，社交网络，地理位置，用户产生的数据和用户操作日志已经成倍的增加。我们如果要对这些用户数据进行挖掘，那SQL数据库已经不适合这些应用了，NoSQL数据库的发展却能很好的处理这些大的数据。

- 2.是什么

  NoSQL（NoSQL = Not Only SQL），意思是不仅仅是SQL，泛指非关系型的数据库。随着互联网web2.0网站的兴起，传统的关系数据库在应付web2.0网站，特别是超大规模和高并发的SNS类型的web2.0纯动态网站已经显得力不从心，暴露了很多难以克服的问题，而非关系型的数据库则由于其本身的特点得到了非常迅速的发展。NoSQL数据库的产生就是为了解决大规模数据集合多重数据种类带来的挑战，尤其是大数据应用难题，包含超大规模数据的存储。
  
  （例如古河或Facebook每天为他们的用户手机万亿比特的数据）。这些类型的数据存储不需要固定的模式，无需多余操作就可以横向扩展。

- 3.能干嘛

	- 易扩展

	  NoSQL数据库种类繁多，但是一个共同的特点都是去掉关系数据库的关系型特性。
	  数据之间无关系，这样就非常容易扩展。也无形之间，在架构的层面上带来了可扩展的能力。

	- 大数据量高性能

	  NoSQL数据里都具有非常高的读写性能，尤其在大数据量下，同样表现优秀。
	  这得益于它的无关系性，数据库的结构简单。
	  一般MySQL使用Query Cache，每次表的更新Cache就失效，是一种大粒度的Cache，在针对web2.0的交互频繁的应用，Cache性能不高。而NoSQL的Cache是记录级的，是一种细粒度的Cache，所以NoSQL在这个层面上来说就要性能高很多了。

	- 多样灵活的数据模型

	  NoSQL无需事先为要存储的数据建立字段，随时可以存储自定义的数据格式。而在关系数据库里，增删字段是一件非常麻烦的事情。如果是非常大数据量的表，增加字段简直就是一个噩梦。

	- 传统RDBMS VS NOSQL

	  RDBMS vs NoSQL
	  
	  RDBMS
	  高度组织化结构化数据
	  结构化查询语言（SQL）
	  数据和关系都存储在单独的表中。
	  数据操纵语言，数据定义语言
	  严格的一致性
	  基础事务
	  
	  NoSQL
	  代表着不仅仅是SQL
	  没有声明性查询语言
	  没有预定义的模式
	  键值对存储，列存储，文档存储，图形数据库
	  最终一致性，而非ACID属性
	  非结构化和不可预知的数据
	  CAP定理
	  高性能，高可用和可伸缩性

- 4.去哪儿

	- Redis
	- Memcache
	- Mongdb

- 5.怎么玩

	- KV
	- Cache
	- Persistence
	- 。。。。

### 3V+3高

- 大数据时代的3V

	- 海量Volume
	- 多样Variety
	- 实时Velocity

- 互联网需求的3高

	- 高并发
	- 高可扩
	- 高性能

- 子主题 3

### 当下的NoSQL经典应用

- 当下的应用是sql和nosql一起使用
- 阿里巴巴中文站商品信息如何存放

	- 阿里巴巴中文网站首页

		- 架构发展历程

			- 1.演变过程
			- 2.第5代
			- 3.第5代架构使命
			- 4.....

		- 和我们相关的，多数据源多数据类型的存储问题

	- 1.商品基本信息

		- 名称、价格、出厂日期、生产厂商等
		- 关系性数据库，mysql/oracle目前淘宝在去O化（也即拿掉Oracle），注意淘宝内部使用的Mysql是大牛改造过的

			- 为什么去IOE

	- 2.商品描述、详情、评价信息（多文字类）

		- 多文字信息描述类，IO读写性能变差
		- 文档数据库MongDB中

	- 3.商品的图片

		- 商品图片展现类
		- 分布式的文件系统中

			- 淘宝自己的TFS
			- Google的GFS
			- Hadoop的HDFS

	- 4.商品的关键字

		- 搜索引擎，淘宝内用
		- ISearch

	- 5.商品的波段性的热点高频信息

		- 内存数据库
		- Tair、Redis、Memcache

	- 6.商品的交易、价格计算、积分累计

		- 外部系统，外部第3方支付接口
		- 支付宝

	- 总结大型互联网应用（大数据、高并发、多样数据类型）的难点和解决方案

		- 难点

			- 数据类型多样性
			- 数据源多样性和变化重构
			- 数据源改造而数据服务平台不需要大面积重构

		- 解决办法

			- 统一数据平台服务层
			- 阿里、淘宝干了什么？UDSL

### NoSQL数据模型简介

- 以一个电商客户、订单、订购、地址模型来对比下关系性数据库和非关系型数据库

	- 传统的关系型数据库你如何设计？

		- ER图（1:1/1:N/N:N,主外键等常见）

	- Nosql你如何设计

		- 什么是BSON

		  BSON()是一种类json的一种二进制形式的存储格式，简称为Binary JSON，它和JSON一样，支持内嵌的文档对象和数组对象。

		- 给学生用BSON画出构建的数据模型

	- 两者对比，问题和难点

		- 为什么上述的情况可以用聚合模型来处理

			- 高并发的操作是不太建议有关联查询的，互联网公司用冗余数据来避免关联查询，分布式事务是支持不了太多的并发的

		- 关系型数据库如何查询？如果按照我们新设计的BSON，是不是查询起来很可爱

- 聚合模型

	- KV键值
	- Bson
	- 列族

	  按列存储的。最大的特点是方便存储结构化和半结构化数据，方便做数据压缩，对针对某一列或者某几列的查询有非常大的IO优势。

	- 图形

### NoSQL数据库的四大分类

- KV键值：典型介绍

	- 新浪：BerkeleyDB+redis
	- 美团：redis+tair
	- 阿里、百度：memcache+redis

- 文档型数据库（bson格式比较多）：典型介绍

	- CouchDB
	- MongoDB

	  MongoDB是一个基于分布式文件存储的数据库。由C++语言编写，旨在为web应用提供可扩展的高性能数据存储解决方案。
	  
	  MongoDB是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。

- 列存储数据库

	- Cassandra，HBase
	- 分布式文件系统

- 图关系数据库

	- 它不是放图形的，放的是关系比如：朋友圈社交网络、广告推荐系统、社交网络，推荐系统等。专注于构建关系图谱。
	- Neo4j，InfoGrid

- 四者对比

  分类 举例	应用场景	数据模型		优缺点
  KV    Redis   内容缓存， KV键值     查找速
  					主要用于     通常用     度快。
  					处理大量    hash table
  					数据的高    实现           
  					访问负载
  					，也用于
  					一些日志
  					系统等。

### 在分布式数据库中CAP原理CAP+BASE

- 传统的ACID分别是什么

	- A（Atomicity）原子性
	- C（Consistency）一致性
	- I（Isolation）独立性
	- D（Durability）持久性

- CAP

	- C：Consistency（强一致性）
	- A：Availability（可用性）
	- P：Partion tolerance（分区容错性）

- CAP的3进2

  CAP的理论就是说在分布式存储系统中，最多只能实现上面的两点。
  而由于当前的网络硬件肯定会出现延迟丢包等问题，所以分区容忍性是我们必须要实现的。
  
  所以我们只能在一致性和可用性之间进行权衡，没有NoSQL系统能同时保证这一点。
  
  C：强一致性  A：高可用性   P：分布式容错性
  
  CA 传统Oracle数据库
  AP大多数网站架构的选择
  CP Redis、Mongodb

- 经典CAP图

  CAP理论图的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，最多只能同时较好的满足两个。
  因此，根据CAP原理将NoSQL数据库分成了满足CA原则、满足CP原则和满足AP原则三大类：
  CA-单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。
  CP-满足一致性，分区容忍性的系统，通常性能不是特别高。
  AP-满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。

- BASE

	- 是什么

	  BASE就是为了解决关系数据库强一致性引起的问题而引起的可用性降低而提出的解决方案。
	  
	  BASE其实是下面三个术语的缩写：
	  基本可用（Basically Available）
	  软状态（Soft state）
	  最终一致（Eventually consistent）
	  
	  它的思想是通过让系统放松对某一时刻数据一致性的要求来换取系统整体伸缩性和性能上改观。缘由就在于大型系统往往由于地域分布和极高性能的要求，不可能采用分布式事务来完成这些指标，我们必须采用另外一种方式来完成，这里BASE就是解决这个问题的办法。

- 分布式+集群简介

  分布式：不同的多台服务器上面部署不同的服务模块（工程），他们之间通过Rpc/Rmi之间通信和调用，对外提供服务和组内协作。
  集群：不同的多台服务器上面部署相同的服务模块，通过分布式调度软件进行统一的调度，对外提供服务和访问。

## Redis入门介绍

### 1.入门概述

- 1.是什么

	- Redis：REmote DIctionary Server（远程字典服务器）
	- 是完全开源免费的，用C语言编写的，遵守BSD协议，是一个高性能的（key/value）分布式内存数据库，基于内存运行并支持持久化的NoSQL数据库，被称为数据结构数据库。
	- Redis与其他key-value缓存产品有一下三个特点

		- Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用
		- Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储
		- Redis支持数据的备份，即master-slave模式的数据备份。

- 2.能干嘛

	- 内存存储和持久化：redis支持异步将内存中的数据写到硬盘上，同时不影响继续服务。
	- 取最新N个数据的操作，如：可以将最新的10条评论的ID放在Redis的List集合里面
	- 模拟类似于HttpSession这种需要设定过期时间的功能。
	- 发布、订阅消息系统
	- 定时器、计数器

- 3.去哪下

	- http://redis.io/
	- http://www.redis.cn/

- 4.怎么玩

	- 数据类型、基本操作和配置
	- 持久化和复制，RDB/AOF
	- 事务的控制
	- 复制
	- .........

### 2.VMWare+VMTools

### 3.Redis的安装

- Windows版安装
- 重要提示：

	- 企业做Redis开发，99%都是Linux版的运用和安装。

- Linux版安装

	- 下载获得redis-3.0.4.tar.gz后将它放入我们的Linux目录/root
	- /opt目录下，解压命令：tar -zxvf redis-3.0.4.tar.gz
	- 解压完成后出现文件夹：redis-3.0.4
	- 在redis-3.0.4目录下执行make命令

		- 运行make命令是故意出现的错误的解析：

			- 安装gcc

			  gcc是linux下的一个编译程序，是C程序的编译工具。

				- 能上网：yum install gcc-c++
				- 不能上网：

			- 二次make
			- Jemalloc/jemalloc.h：没有那个文件目录

				- 运行make distclean之后再make

			- Redis Test（可以不用执行）

	- 如果make完成后继续执行make install
	- 查看默认安装目录：usr/local/bin
	- 启动
	- 永远的helloworld
	- 关闭

### 4.Redis启动后杂项基础知识讲解

- 单进程

	- 单进程模型来处理客户端的请求。对读写等时间的响应是通过对epoll函数的包装来做到的。Redis的实际处理速度完全依靠主进程的执行效率。
	- Epoll是Linux内核为处理大批量文件描述符而作了改进的epoll，是Linux下多路复用IO接口select/poll的增强版本，它能显著提高程序在大量并发连接中只有少量活跃情况下的系统CPU利用率。

- 默认16个数据库，类似数组下表从零开始，初始默认使用零号库
- Select命令切换数据库
- Dbsize查看当前数据库的key的数量
- Flushdb：清空当前库
- Flushall：通杀全部库
- 同一密码管理，16个库都是同样密码，要么都ok要么一个也连不上
- Redis索引都是从零开始
- 为什么默认端口是6379

## Redis数据类型

### Redis的五大数据类型

- String（字符串）

  String（字符串）
  
  string是redis最基本的类型，可以理解为Memcached一样的类型，一个key对应一个value。
  
  string类型是二进制安全的，意思是redis的string可以包含任何数据。比如ipg图片或者序列化的对象。
  
  string类型是Redis最基本的数据结构，一个redis中字符串value最多可以是512M。

- Hash（哈希，类似java里的Map）

  Hash（哈希）
  Redis hash是一个键值对集合。
  Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。
  
  类似java里面的Map<String,Object>

- List（列表）

  List(列表)
  Redis列表是简单的字符串列表，按照插入顺序排序。可以添加一个元素到列表的头部（左边）或者尾部（右边）。
  它的底层是一个链表。

- Set（集合）

  Set（集合）
  Redis的Set是string类型的无序集合。它是通过HashTable实现的。

- Zset（sorted set：有序集合）

  zset（有序集合）
  Redis zset和set一样也是string类型元素的集合，且不允许重复的成员。
  不同的是每个元素都会关联一个double类型的分数。
  redis正是通过分数来为集合中的成员进行从小到大排序。zset的成员是唯一的，但分数（score）却可以重复。

### 哪里去获得redis常见数据类型操作命令

- http://redisdoc.com/

### Redis键（key）

- 常用
- 案例

	- keys *
	- exists key的名字,判断某个key是否存在
	- move key db -->当前库就没有了，被移出了
	- expire key 秒钟：为给定的key设置过期时间
	- ttl key查看还有多少秒过期，-1表示永不过期，-2表示已过期
	- type key 查看你的key是什么类型

### Redis字符串（String）

- 常用
- 单值单value
- 案例

	- set/get/del/append/strlen
	- Incr/decr/incrby/decrby,一定要是数字才能进行加减
	- getrange/setrange

	  getrange：获取指定区间范围内的值，类似between....and的关系。
	  
	  从0到-1表示全部
	  
	  
	  setrange设置指定区间范围内的值，格式是setrange key值，具体值。

	- setex(set with expire)键秒值/setnx(set if not exist)
	- mset/mget/msetnx
	- getset(先get在set)

### Redis列表（List）

- 常用
- 单值多value
- 案例

	- lpush/rpush/lrange
	- lpop/rpop
	- lindex,按照索引下表获得元素（从上到下）
	- llen
	- lren key 删N个value
	- ltrim key 开始index 结束index，截取指定范围的值后再赋值给key
	- rpoplpush 源列表 目的列表
	- lset key index value
	- linsert key before/after 值1 值2
	- 性能总结

	  它是一个字符链表，left，right都可以插入添加。
	  如果键不存在，创建新的链表。
	  如果键已经存在，新增内容。
	  如果值全移除，对应的键也就消失了。
	  链表的操作无论是头和尾效率都极高，但假如是对中间元素进行操作，效率就很惨淡了。

### Redis集合（Set）

- 常用
- 单值多value
- 案例

	- sadd/smembers/sismember
	- scard,获取集合里面的元素个数
	- srem key value 删除集合中元素
	- srandmember key 某个整数（随机出几个数）
	- spop key 随机出栈
	- smove key2 key2 在key1里某个值    作用是将key1里的某个值赋给key2
	- 数学集合类

		- 差集：sdiff
		- 交集：sinter
		- 并集：sunion

### Redis哈希（Hash）

- 常用
- KV模式不变，但V是一个键值对
- 案例

	- hset/hget/hmset/hgetall/hdel
	- hlen
	- hexists key 在key里面的某个值的key
	- hkey/hvals
	- hincrby/hincrbyfloat
	- hsetnx

### Redis有序集合Zset（sorted set）

- 和set的区别：在set基础上加上一个score值，之前set是k1 v1 v2 v3,现在zset是k1 score v1 score v2
- 案例

	- zadd/arange
	- zrangebyscore key 开始score 结束score

		- withscores
		- （  不包含
		- limit作用是返回限制  limit开始下标步  多少步

	- zrem key 某score下对应的value值，作用是删除元素
	- zcard/zcount key score区间/zrank key values值，作用是获得下标值/zscore key 对应值，获得分数
	- zrevrank key values值，作用是逆序获得下标值
	- zrevrange
	- zrevrangebyscore key 结束score 开始score

## 解析配置文件redis.conf

### 1.它在哪

### 2.Units单位

配置大小单位，开头定义了一些基本的度量单位，只支持bytes，不支持bit
对大小写不敏感

### 3.INCLUDES包含

可以通过includes包含，redis.conf可以作为总闸，包含其他。

### 4.GENERAL通用

- Daemonize

  守护线程启动

- Pidfile

  进程管道文件

  /var/run/redis.conf

- Port

  6397

- Tcp-backlog

  tcp-backing 551

  设置tcp的backlog，backlog其实是一个连接队列，backlog队列总和=未完成三次握手队列+已经完成三次握手队列。

  

  在高并发环境下你需要一个高backlog值来避免慢客户端连接问题。注意linux内核会将这个值减小到/proc/sys/net/core/somaxconn的值，所以需要确认增大somaxconn和tcp_max_syn_backing两个值来达到想要的效果。

- Timeout

  0 不关闭，超时连接

- Bind
- Tcp-keepalive

  单位为秒，如果设置为0，则不会进行Keepalive检测，建议设置成60

  心跳机制！！！

- Loglevel

  debug

  verbose

  notice

  warning 生产模式

  

  级别越高，日志越少

- Logfile
- Syslog-enabled

  是否把日志输出到syslog中

- Syslog-ident

  指定syslog里的日志标志

- Syslog-facility

  指定syslog设备，值可以是user或local0-local7

  默认0

- Databases

  默认16个库

### 5.SNAPSHOTTING快照

- Save

	- save 秒钟  写操作次数

	  RDB是整个内存压缩过的Snapshot，RDB的数据结构，可以配置复合的快照触发条件，默认
	  1分钟内改变了1万次，
	  或5分钟内改了10次，
	  或15分钟内改了1次。

	- 禁用

	  如果想禁用RDB持久化策略，只要不设置任何save指令，或者给save传入一个空字符串参数也可以。

- Stop-writes-on-bgsave-error

  默认是yes，如果配置成no，表示不在乎数据不一致或者有其他的手段发现和控制。

- rdbcompression

  默认是yes
  rdbcompression：对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，redis会采用LZF算法进行压缩。如果不想消耗CPU来进行压缩的话，可以设置为no关闭此功能。

- rdbchecksum

  默认是yes
  rdbchecksum：在存储快照后，还可以让redis使用CRC64算法来进行数据校验，但是这样做会增大大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能。

- dfgilename
- dir

### 6.REPLICATION复制

### 7.SECURITY安全

### 8.LIMITS限制

### 9.APPEND ONLY MODE追加

- appendonly
- appendfilename
- Appendfsync

	- Always：同步持久化，每次发生数据变更会立即记录到磁盘，性能较差但数据完整性比较好
	- Everysec：出厂默认推荐，异步操作，每秒记录，如果一秒内宕机，有数据丢失
	- No

- No-appendfsync-on-rewrite：重写时是否可以运行Appendfsync，用默认no即可，保证数据安全性。
- Auto-aof-rewrite-min-size：设置重写的基准值
- Auto-aof-rewrite-perentage：设置重写的基准值

### 10.常见配置redis.conf介绍

## Redis的持久化

### 总体介绍（官网）

### RDB(Redis DataBase)

- 官网介绍
- 是什么

	- 在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是Snapshot快照，它恢复时将快照文件直接读到内存里。
	- Redis会单独创建（Fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。                                                 整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能                                             如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。

- Fork

	- Fork的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量‘程序计数器等）数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程。

- RDB保存的是dump.rdb文件
- 配置位置
- 如何触发RDB快照

	- 配置文件中默认的快照配置

		- 冷拷贝后重新使用

			- 可以cp dump.rdb dump_new.rdb

	- 命令save或者是bgsave

		- Save：save时只管保存，其他不管，全部zuse
		- BGSAVE：Redis会在后台异步进行快照操作，快照操作同时还可以响应客户端请求。可以通过lastsave命令获取最后一次成功执行快照的时间。

	- 执行flushall命令，也会产生dump.rdb文件，但里面是空的，无意义。

- 如何恢复

	- 将备份文件（dump.rdb）移动到redis安装目录并启动服务即可
	- CONFIG GET dir获取目录

- 优势

	- 适合大规模的数据恢复
	- 对数据完整性和一致性要求不高

- 劣势

	- 在一定间隔时间做一次备份，所以如果redis意外down掉的话，就会丢失最后一次快照后的所有修改。
	- Fork的时候，内存中的数据被克隆了一份，大约2倍的膨胀性需要考虑。

- 如何停止

	- 动态所有停止RDB保存规则的方法：redis-cli config set save ""

- 小总结

  内存中的   ---rdbsave--->   磁盘中的
  数据对象   <---rdbLoad---   RDB文件
  
  RDB是一个非常紧凑的文件
  RDB在保存RDB文件时父进程唯一需要做的就是fork出一个子进程，接下来的工作全部由子进程来做，父进程不需要再做其他IO操作，所有RDB持久化方式可以最大化redis的性能。
  与AOF相比，在恢复大的数据集的时候，RDB方式会更快一些。
  
  
  缺点：
  数据丢失风险大
  RDB需要经常fork子进程来保存数据集到硬盘上，当数据集比较大的时候，fork的过程是非常耗时的，可能导致Redis在一些毫秒级不能响应客户端请求。

### AOF(Append Only File)

- 官网介绍
- 是什么：

	- 以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来（读操作不记录），只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话根据日志文件的内容将写指令从前到后执行一次已完成数据的恢复工作。

- AOF保存的是appendonly.aof文件
- 配置位置

  默认是no，改成yes就打开了aof持久化

- AOF启动/修复/恢复

	- 正常恢复

		- 启动：设置Yes
		- 将有数据的aof文件复制一份保存到对应目录（config get dir）
		- 恢复：重启redis然后重新加载

	- 异常恢复

		- 启动：设置Yes
		- 备份被写坏的AOF文件
		- 修复：

			- Redis-check-aof   --fix进行修复

		- 恢复：

			- 重启redis然后重新加载

- Rewrite

	- 是什么

		- AOF采用文件追加方式，文件会越来越大，为避免出现此种情况，新增了重写机制，当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集。可以使用命令gbrewriteaof

	- 重写原理

		- AOF文件持续增长而过大时，会fork出一条新进程来将文件重写（也是先写临时文件最后在rename），遍历新进程的内存中数据，每条记录有一条的Set语句。重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似。

	- 触发机制

		- Redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发。

- 优势

	- 每秒同步：appendfsync always  同步持久化  每次发生数据变更会被立即记录到磁盘  性能较差但数据完整性比较好
	- 每修改同步：appendfsync everysec  异步操作 ，每秒记录  如果一秒内宕机，有数据丢失。
	- 不同步：appendfsync no  从不同步

- 劣势

	- 相同数据集的数据而言aof文件要远大于rdb文件，恢复速度慢于rdb
	- AOF运行效率要慢于rdb，每秒同步策略效率较好，不同步效率和rdb相同。

- 小总结

  网络协议格式
         命令请求                 的命令内容
  客户端-------->服务器---------->AOF文件
  
  AOF文件时一个只进行追加的日志文件
  Redis可以在AOF文件体积变得过大时，自动地在后台对AOF进行重写
  AOF文件有序地保存了对数据执行的所有写入操作，这些写入操作以Redis协议的格式保存，因此AOF文件的内容非常容易被人读懂，对文件进行分析也很轻松。
  
  缺点：
  对相同的数据集来说，AOF文件的体积通常要大于RDB文件的体积。
  根据所使用的fsync策略，AOF的速度可能会慢于RDB。

### 总结（Which one）

- RDB和AOF可以共存，但是恢复的时候找的是AOF，如果AOF文件异常，可以通过check-aof进行AOF修复。

## Redis的事务

### 是什么

- 可以一次执行多个命令，本质是一组命令的集合。一个事务中的所有命令都会序列化，按顺序地串行化执行而不会被其他命令插入，不许加塞。
- 官网

### 作用

- 一个队列中，一次性、顺序性、排他性地执行一系列命令

### 怎么玩

通过MULTI命令进入Redis事务。通过EXEC执行。

- 常用命令

  DISCARD：取消事务，放弃执行事务块内的所有命令。
  EXEC：执行所有事务块的命令。
  MULTI：标记一个事务块的开始。
  UNWATCH：取消WATCH命令对多有key的监视。
  WATCH key [key......]：监视一个（或多个）key，如果在事务执行之前这个key被其他命令所改动，那么事务将打断。

- Case1：正常放行
- Case2：放弃事务
- Case3：全体连坐
- Case4：冤头债主
- Case5：watch监控

	- 悲观锁/乐观锁/CAS(Check And Set)

		- 悲观锁

		  悲观锁：顾名思义，每次去拿数据的时候都被认为别人会修改，所以每次在拿数据的时候都会被锁上，这样别人想拿这个数据就会block直到它拿到锁，传统的关系型数据库里边就用到了很多这种锁机制，比如行锁、表锁等，读锁、写锁等，都是在做操作之前先锁上。

		- 乐观锁

		  乐观锁：每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多度的应用类型，这样可以提高吞吐量。
		  
		  乐观锁策略：提交版本必须大于记录当前版本才能执行更新。

		- CAS

	- 初始化信用卡可用余额和欠额
	- 无加塞篡改，先监控再开启multi，保证两笔金额变动在同一个事务内
	- 有加塞篡改
	- unwatch
	- 一旦执行了exec，之前加的监控锁都会被取消掉
	- 小结

		- Watch指令，类似乐观锁，事务提交时，如果Key的值已经被别的客户端改变，比如某个list已经被别的客户端push/pop过了，整个事务队列都不会被执行。
		- 通过WATCH命令在事务执行之前监控了多个keys，倘若在WATCH之后有任何key的值的变化，EXEC命令执行的事务都将被放弃，同时返回Nullmulti-bulk应答以通知调用者事务执行失败。

### 3阶段

- 开启：以MULTI开始一个事务
- 入队：将多个命令入队到事务中，接到这些命令并不会立即执行，而是放到等待执行的事务队列里面。
- 执行：由EXEC命令触发事务

### 3特性

- 单独的隔离操作：事务中的所有命令都会被序列化、按顺序地执行。事务在执行的构成中，不会被其他客户端发送来的命令请求所打断。
- 没有隔离级别的概念：队列中的命令没有提交之前都不会实际的被执行，因为事务提交前任何指令都不会被实际执行，也就不存在“事务内的查询要看到事务里的更新，在事务外查询不能看到”这个让人万分头痛的问题。
- 不保证原子性：redis同一个事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚。

## Redis的发布订阅

### 是什么

- 进程间的一种消息通信模式：发送者（pub）发送消息，订阅者（sub）接受消息。
- 订阅/发布消息图

### 命令

PSUBSCRIBE pattem [pattem......]：订阅一个或多个符合给定模式的频道
PUBSUB subcommand [argument [argument.....]]：查看订阅与发布系统状态。
PUBLISH channel message：将信息发送到指定的频道。
PUNSUBSCRIBE [pattem [pattem....]]：退订所有给定模式的频道。
SUBSCRIBE channel [channel...]：订阅给定的一个或多个频道的信息。
UNSUBSCRIBE [channel [channel...]]：指退订给定的频道。

### 案例

先订阅后发布才能收到信息，
可以一次性订阅多个，SUBSCRIBE c1 c2 c3
消息发布，PUBLISH c2 hello-redis
订阅多个，通配符*，PSUBSCRIBE new*
收到消息，PUBLISH new1 redis2015

## Redis的复制（Master/Slave）

### 是什么

- 官网
- 就是我们所说的主从复制，主机数据更新后根据配置和策略，自动同步到备机的master/slver机制，Master以写为主，Slave以读为主。

### 作用

- 读写分离
- 容灾恢复

### 怎么玩

- 1.配从（库）不配主（库）
- 2.从库配置：slaveof主库IP主库端口

	- 每次与master断开之后，都需要重新连接，除非你配置进redis.conf文件
	- Info replication

- 3.修改配置文件细节操作

	- 拷贝多个redis.conf文件
	- 开启daemonize yes
	- Pid文件名字
	- 指定端口
	- Log文件名字
	- Dump.rdb名字

- 4.常用3招

	- 一主二仆

		- Init
		- 一个Master两个Slave
		- 日志查看
		- 主从问题演示

	- 薪火相传

		- 上一个Slave可以是下一个slave的Master，slave同样可以接受其他slaves的连接和同步请求，那么该slave作为链条中下一个的master，可以有效减轻master的写压力。
		- 中途变更转向：会清楚之前的数据，重新建立拷贝最新的。
		- slaveof  新主库IP  新主库端口

	- 反客为主

		- SLAVEOF no one：使当前数据库通知与其他数据库的同步，转成主数据库。

### 复制原理

- Slave启动成功连接到master后会发送一个sync命令
- Master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，master将传送整个数据文件到slave，以完成一次完全同步。
- 全量控制：而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。
- 增量控制：master继续将新的所有收集到的修改命令一次传给slave，完成同步
- 但是只要是重新连接master，一次完全同步（全量复制）将被自动执行。

### 哨兵模式（setinel）

- 是什么

	- 反客为主的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库

- 怎么玩

	- 调整结构，6379带着80、81
	- 自定义的/myredis目录下新建sentinel.conf文件，名字决不能错。
	- 配置哨兵。填写内容

		- sentinel monitor被监控数据库名字（自己起名字）127.0.0.1 6379 1
		- 上面最后一个数字1，表示主机挂掉后slave投票看让谁解题成为主机，得票数多的成为主机

	- 启动哨兵

		- Redis-sentinel  /myredis/sentinel.conf
		- 上述目录依照各自的实际情况配置，可能目录不同

	- 正常主从演示
	- 原有的master挂了
	- 投票新选
	- 重新主从继续开工，info replication查查看
	- 问题：如果之前的master重启回来，会不会双master冲突？

	  不会冲突，之前的master回来之后，哨兵监测到，之前的master会编程slave

- 一组sentinel能同时监控多个Master

### 复制的缺点

- 复制延时

  由于所有的写操作都是先在Master上操作，然后同步更新到slave上，所以从master同步到slave机器有一定的延迟，当系统很繁忙的时候，延迟问题会更加严重，slave机器数量的增加也会使这个问题更加严重。

## Redis的Java客户端Jedis

*XMind: ZEN - Trial Version*