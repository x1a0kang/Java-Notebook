# MySQL

## 基础

* 数值类型
  * 整型：int，tiny， small，medium，big；浮点型：float，double；定点型：decimal；
  * 字符串：char，varchar，text，blob；
  * 日期：date，datetime，timestamp；
* char和varchar区别：
  * char是定长字符串，用空格填充达到指定长度，最长255；varchar是变长字符串，用额外字节储存长度，最长65535；
  * char适合储存长度固定或变化不大的字段，如ID，电话号码，IP等等；varchar适合用户昵称，文章标题等；
  * varchar(10)和varchar(100)：只是能储存的最大长度不同，存相同字符时所占空间是相同的，但更占内存，因为进入内存时是根据最大值分配空间的；
* text和blob（二进制字符串）：能在varchar范围内的尽量用varchar，不建议使用这两个。不能有默认值，不能直接创建索引，要指定前缀长度，检索效率低，可能消耗大量网络IO和带宽；
* datetime和timestamp
  * datetime没有时区信息，timestamp有时区信息；
  * 也因为有时区信息，timestamp要调用系统的函数，且要加锁，因此性能不好
  * 也可以用数字类型的时间戳，bigint，但是可读性比较差；
* Boolean：MySQL没有布尔类型，一般用tinyint表示。
* count(1)，count(*)，count(列名)的区别
  * count(1)：忽略所有列，不忽略null，当列名不是主键时最快；
  * count(*)：包括所有列，不忽略null，当只有一个列时最快；
  * count(列名)：只包括一列，忽略null，列名是主键时最快；
* 引擎：innodb，myisam，memory
  * innodb：支持事务，有四种隔离级别，支持并发控制，支持行级锁，支持崩溃修复，支持外键。
  * myisam：不支持事务、不支持崩溃修复和行级锁，只支持表级锁，所以相比innodb并发性能更低，主要用于读多写少的情况，因为写要加表锁，索引和数据分开储存。
  * memory：储存在内存，数据处理速度快，但停机后数据丢失，且对表的大小有要求，不能太大。因此适合临时使用，数据量不大，速度要求高，安全性要求不高的情形。


## 事务

* 储存引擎：InnoDB，myISM，memory，只有InnoDB是支持事务的；引擎是基于表的，可以为每张表指定不同的引擎；
* 怎么在MySQL里使用事务？begin/start transaction，commit，rollback
* ACID：
  * 原子性：事务是最小的单位，不可分割。事务中的动作要不全部成功要不全部失败；
  * 隔离性：事务之间互相不影响；
  * 持久性：事务对数据的改变是持久的，即时数据库发生故障也不影响；
  * 一致性：数据在事务发生的前后保持一致，如转账前后两个人钱的总和保持不变；
  * 原子性，隔离性和持久性是一致性的基础，只有满足了前三个条件，才能实现一致性；
* 隔离级别：默认可重复读。
  * 读未提交，可能发生脏读，即读取到其他事务未提交的数据，如果这个数据被回滚，那其他事务读到的就是脏数据；
  * 读已提交，可能发生不可重复读，即一个事务多次读取同一个数据时，得到的结果不同，因为数据中间被另一个事务修改了；
  * 可重复读，可能发生幻读，即一个事务多次读取多行数据时，得到的结果数量不同，因为中间另一个事务新增了数据；
  * 串行化，不会发生以上任何情况，所有事务串行化执行，但是对性能有影响；
* 隔离级别如何保证：
  * undolog：用于回滚的日志，用来保证原子性；
  * redolog：写操作会记录到redolog，如果数据库崩溃，也可以从redolog恢复，用来保证持久性；
  * 锁+MVCC：保证隔离性；
* MVCC：多版本并发控制：
  * InnoDB有两个和MVCC相关的隐藏列：1. 记录最近一次修改的事务ID，2. 回滚指针，指向该行的undolog，可以访问到改行的历史版本；
  * ReadView：一种数据结构，用来保证读已提交和可重复读。包含以下字段：
    * 目前已出现的最大事务ID+1：版本大于等于这个ID的数据不可见；
    * 活跃事务ID列表：即目前还没有提交的事务，这些版本的ID的数据不可见；
    * 活跃事务ID列表中的最小ID：版本号小于这个ID的数据均可见；
    * 当前事务ID；
  * 读已提交：每次select查询都生成一个read view
  * 可重复读：只在事务的第一个select生成一个read view，此后都用同一个read view；
  * 使用read view的是快照读，一般的select语句是使用的快照读，不用加锁，并发性能高。要加锁的是当前读，比如select for update，或者update/delete操作，需要加行锁或者间隙锁。
  * MySQL在可重复读级别下解决幻读：在执行当前读时，通过next-key lock，锁住gap来保证不会出现新纪录，但是在有快照读和当前读混合时，还是有可能出现幻读，在可重复读级别下，不能完全解决幻读；
  * 如果A先快照读，即select一个范围，B在这个范围内插入一条数据，这时A再快照读是看不见的，但是如果A当前读，比如update，这时是可以修改到B新增的数据的，并且这个数据的事务id也会变成A的id，然后A再快照读的时候就能看到这条数据了，就是发生了幻读。

## 锁

### 行级锁：

* myism只有表级锁，innodb既有表级锁也有行级锁

* 记录锁record lock：只锁定单行记录；
* 间隙锁gap lock：锁定一个范围，但不包括记录本身；
* 临键锁next-key lock：记录锁加间隙锁，锁定一个范围，包括记录本身，主要用于解决幻读问题；
* 在innodb默认隔离级别下，默认行锁是next-key lock，但如果操作的索引是主键或唯一索引，引擎会把锁降级为record lock；
* select for update会加写锁，如果查询条件中有索引列，特别是唯一索引或主键索引，数据库只会锁当前行；但是如果查询条件没有索引，数据库可能会锁住大量的行或者全表，因为要全表扫描；

### 间隙锁

* 在范围查询时，对唯一索引范围查询，或者for update等主动加锁；

* 在查找一个数据不存在的等值查询时，MySQL会找到相邻的数据，对范围加间隙锁，防止有数据插入；

### 共享锁和排他锁

* 共享锁/S锁：又称读锁，事务在读时获取共享锁，其他事务可以读，但不可以写；
* 排它锁/X锁：又称写锁，事务在写时获取排他锁，其他事务读写都不可以；
* **因为有MVCC的存在，对于一般的select语句，innodb不会加任何锁**；

### 表级锁：

* 意向锁是一种表级锁，分为两种

  * 意向共享锁，IS锁：事务有意向对表中的行加共享锁S锁，在之前要先获得该表的IS锁；

  * 意向排他锁，IX锁：事务有意向对表中的行加排他锁X锁，在之前要先获得该表的IX锁；


* 意向锁是有引擎自己维护的，在为数据行加行锁前，引擎会先获取该表的对应意向锁，用户不能手动加意向锁，意向锁之间是相互兼容的，也就是多个事务都可以获取意向锁并且不干扰；
* 意向锁的作用是，当一个事务需要给表加锁时，要先看当前表有没有锁，尤其是行锁，如果一行一行去找不现实，于是在加行锁前要先获取表级的意向锁，这样就可以快速判断当前表能否加锁了；

## 索引

* 聚簇索引：索引结构和数据存放在一起的索引，innodb的主键是聚簇索引，叶子节点存储完整行数据，非叶子节点存储索引键值和子节点指针；
* 非聚簇索引：索引结构和数据分开存放的索引，innodb主键以外的索引是非聚簇索引，叶子节点存储索引字段键值，且指针指向主键索引，通过当前索引找到主键索引，再通过聚簇索引查找完整数据行（回表）；
* 覆盖索引：查询的条件字段全部命中了索引，且所需结果字段在条件字段内，这种情况下可以直接从索引中获取全部结果，不需要再回表查询，这种就叫覆盖索引；
* 主键索引：innodb的表一定要有主键，如果没有指定，会查找有没有唯一索引的列，如果没有，就自动生成一个自增主键，主键索引的存储结构是聚簇索引，不允许有空值；
* 联合/复合索引：多个列组成的索引，会根据从左到右列的顺序进行排列，前一列相同时，根据后一列排序：
  * 最左前缀匹配原则：在联合索引中，索引从左到右匹配，查询条件必须有索引最左列才能使用索引加速，且不跳过中间的列，条件中包含索引的前缀都可以使用索引加速；
  * 联合索引要把最常查询的字段、区分度最高的字段放在前面；
* 索引的类型：有B+树和哈希索引

### B树和B+树

* B树所有节点都包含键值和数据，还有指向子节点的指针，也就是在查找的过程中不用到达叶子节点就可能找到数据并返回
* B+树非叶子节点只保存键值和指向子节点的指针，只有叶子节点保存索引列数据和**对应的主键值**，并且叶子节点之间通过双向链表连接，支持顺序访问和范围查询；
* B树的缺点：范围查询可能分布在多层，涉及磁盘随机IO，效率低。B+树则可以在叶子节点中顺序查找；
* B树的缺点：每个节点储存的数据大，因此键值少，树高就更高，那IO次数就更多，导致性能不好；
* 为什么B树不能更宽，因为为了优化IO，B树或者B+树都让自己的一个节点保持一个磁盘页的大小，这样一个节点就可以在一次IO全部获取到，而这就限制了节点能储存的键值的量，B树就只能储存更少的键值，而B树的分支树m，最大只能是每个节点储存的键值数量+1，也就限制了B树的宽度，导致B树只能往更深去发展；

### MySQL架构

* 服务层：包含连接器，分析器，优化器，执行器，还有缓存等，负责过滤等操作；
* 引擎层：innodb等引擎，保存数据；
* 引擎内部：有缓存池缓存数据和索引页的，也有log buffer，如redolog和undolog。磁盘上还有数据表的存储，索引存储，日志存储等；

### 索引下推

* 本该在服务层做的过滤操作，下放到引擎在索引遍历过程中，执行部分where的判断条件，直接过滤掉不满足条件的记录，从而减少回表次数；
* 必须是非聚簇索引，不然不用回表；
* 适用于联合索引的多列过滤，或范围查询后的条件过滤；
* 未启用索引下推时，如（a,b）索引，查询a = 1 and b = 2，要先从索引中查到a = 1的数据，回表取出所有行，返回到服务层，再筛序b = 2，又要查一次索引回一次表（脑残吧）；
* 启用后，上述就直接在索引里把a = 1 and b = 2都查出，再回表返回到服务层，减少回表次数，同时也能减少服务层和引擎层之间传输的数据量；

### 索引失效

* 在索引列上使用函数或表达式，因为无法预先计算出结果；
* 使用不等<>或not符，因为会扫描全表；
* 使用like语句，但通配符在前，以%或_开头；
* 联合索引但不满足最左前缀原则；

## 日志

### redolog

* innodb引擎独有的，让MySQL有崩溃恢复的能力，是物理日志，记录在某个数据页上做了什么修改；
* Write-ahead logging（WAL）：在事务提交之前，将事务所做的修改操作记录到redo log中，然后再将数据写入磁盘。这样即使在数据写入磁盘之前发生了宕机，系统可以通过redolog中的记录来恢复数据。
* redolog有缓存，不是直接写入文件，因此有刷盘时机的选择
  * 配置0：事务提交时不刷盘，性能最高，但有可能丢数据；
  * 配置1：每次事务提交刷盘，性能最低，但最安全，默认是1；
  * 配置2：每次事务提交把缓存写入page cahe（文件系统的缓存），性能和安全性介于中间。
* 还有一个后台线程，每一秒刷一次盘；
* 当redolog buffer占用空间达到一定值时，刷一次盘；

### binlog

* 在服务层，是逻辑日志，记录语句的原始逻辑，不管什么引擎，都会产生binlog
* binlog主要用来数据备份，主从复制等等，同步数据使用；
* binlog记录所有涉及更新数据的操作，并且是顺序写；
* binlog也有binlog buffer，执行事务时先写入缓存，事务结束后写入文件；
* 配置1：每次事务提交刷盘；
* 配置0：事务提交时不刷盘，性能最高，但有可能丢数据；

### binlog和redolog的两阶段提交

* binlog和redolog为什么不能只有一个：binlog是服务层的日志，redolog是innodb引擎层的日志，binlog无法记录引擎哪些数据（脏页）还没刷盘，redolog可以，因此崩溃恢复时使用redolog可以恢复没有刷盘的脏数据。
  * 事务修改数据后，写入redolog，标记prepare状态，将redolog持久化到磁盘。
  * 再写入binlog，binlog持久化到磁盘。
  * 再把redolog标记为commit状态；
* 两阶段提交是以 binlog 写成功为事务提交成功的标识，因为binlog 写成功了，就意味着能在binlog 中查找到与 redo log 相同的 XID；
* 写入binlog失败，恢复时发现redolog（不管什么状态），但binlog不存在，会回滚；
* 写入binlog成功，但redo改commit失败，恢复时看redo和bin都在，会设redo为commit，不回滚；

### undolog

* 逻辑日志，每一个事务都会记录undo，记的是上一个版本的数据，事务回滚时使用undo；
* 如执行一个delete语句，undo会记录一个insert，用于回滚；
* insert事务提交后，undolog可以直接删除
* update和delete事务提交后，会保留至没有依赖后再由后台线程删除；

### 主从复制

* 主服务器上的修改语句写入binlog；
* 从库请求同步，主服务器启动binlog dump线程读取binlog发送给从服务器；
* 从服务器IO线程把接收到是日志写入中继日志relay log；
* 从服务器SQL线程读取relay log，并在本地库执行；

## 高可用

主从复制

* **原理**：主库（Master）通过 binlog 异步同步数据到从库（Slave），支持读写分离。
* **优点**：部署简单，资源消耗低，适合读多写少的场景
* **缺点**：数据异步同步可能导致延迟，主库故障时需手动切换从库，存在数据丢失风险

双主复制：两个主库互为主从，实现双向复制，也可以每个主配两个从节点

MySQL cluster，MySQL group replication

### 慢查询怎么定位和优化

1. 打开慢查询日志，定位慢查询SQL；
2. 执行计划分析Explain，重点关注：
   1. type：ALL是全表扫描；range是范围，用到了索引，查询的是范围；ref是非唯一索引，用到了索引，但查询到的数据不唯一；eq_ref是唯一索引；const是唯一索引和常量值比较，如id=1；
   2. key：命中的索引，未命中索引为null；
   3. rows：扫描行数超过10W行需要注意；
3. 优化，如果没有命中索引，新建查询字段的索引，或者修改SQL使其命中索引
4. 避免索引失效
5. 优化，重写低效SQL，如子查询转join，join时小表驱动大表，最好可以通过冗余字段设计，避免联表查询；
6. 库表层面，读写分离，分库分表，或者复杂SQL改用其他数据库，如大数据套件Hadoop或spark运算；

### 深度翻页

* 深度翻页的问题
  1. 全表扫描和数据跳过：MySQL需扫描`offset+size`行数据，并丢弃前`offset`行，导致**扫描行数随页数呈线性增长**。例如查询第10万页（`LIMIT 1000000,10`），需扫描1000010行，仅返回10行数据，数据越深，扫过的数据越多，需要的IO越多，导致耗时增加；
  2. 回表：如果查询的字段没有完全覆盖索引，offset+limit值全部要回表，回表100W次造成性能雪崩；
* 深度翻页优化
  1. 延迟关联：通过子查询排序并且定位到数据行，再关联主键获取完整数据；比如先通过时间字段查到第10万行数据，子查询提供主键，外层查询再通过主键去获取完整数据。子查询利用覆盖索引，避免回表；
  2. 游标分页：基于上一次查询的游标，不需要offset。但是不支持跳页，只能顺序翻页；
  3. 覆盖索引：查询的列都在索引中，避免回表。