# Mysql 实战 45 讲

| * 编号｜建议阅读时长｜文章标题

## 基础知识【12’】
* 01 |12’| 基础架构：一条SQL查询语句是如何执行的？ 

## 锁相关【85’】
> 弹锁要谈事务隔离级别

* 06 |13’| 全局锁和表锁 ：给表加个字段怎么有这么多阻碍？
* 07 |12’| 行锁功过：怎么减少行锁对性能的影响？
* 19 |13’| 为什么我只查一行的语句，也执行这么慢？
    > 构造表锁、等flush、行锁、一致性读 导致SQL查询慢的场景
* 20 |20’| 幻读是什么，幻读有什么问题？
    > insert into t values(0,0,0),(5,5,5),(10,10,10),(15,15,15),(20,20,20),(25,25,25);
    > 为什么需要间隙锁解决幻读；间隙锁和行锁合称 next-key lock，前开后闭合；
    RR 级别下间隙锁就生效；

* 21 |18’| 为什么我只改一行的语句，锁这么多？
    > insert into t values(0,0,0),(5,5,5),(10,10,10),(15,15,15),(20,20,20),(25,25,25);

    > RR级别下加锁规则。
    - 原则 1：加锁的基本单位是 next-key lock。希望你还记得，next-key lock 是前开后闭区间。（但是加锁步骤是先gap lock 然后是行 x 锁，所以 gap 可能Lock成功但是行锁Lock等待中）

    - 原则 2：【范围查询】查找过程中访问到的对象才可能会加锁。（也就是说范围查询会锁得比较多）

    - 优化 1：左开区间优化。索引上的【等值查询】，给【唯一索引】加锁的时候，next-key lock 退化为行锁。非唯一索引不会退化，保持为范围锁next-key lock
    
    - 优化 2：右闭区间优化。【索引】上的【等值查询】，向右遍历时且最后一个值不满足等值条件的时候，next-key lock 退化为间隙锁gap lock。去掉不满足行的行锁

    - 一个 bug：【唯一索引】上的【范围查询】会访问到不满足条件的第一个值为止。（高版本已修复）

    > 锁是加在 索引 上的。lock in share mode 只锁覆盖索引，但是如果是 for update 就不一样了。 执行 for update 时，系统会认为你接下来要更新数据，因此会顺便给主键索引上满足条件的行加上行锁。

    > 锁是一句SQL就加的，每个事务包含多个SQL，可能锁不同的行。事务实行完毕释放这些锁

    > 在读提交隔离级别 RC 下还有一个优化，即：语句执行过程中加上的行锁，在语句执行完成后，就要把“不满足条件的行”上的行锁直接释放了


* 30 |16’| 答疑文章（二）：用动态的观点看加锁
    > 锁（间隙锁、行锁）是一个个加的
    > 死锁、锁等待排查：`show engine innodb status` 


* 40 |13’| insert语句的锁为什么这么多？
    > 案例
    > insert...select 插入时的锁问题
    > 唯一键冲突

## 事务相关【68’】
* 03 |11’| 事务隔离：为什么你改了我还看不见？
    > 事务概述

    > MVCC事务视图（数据快照，read-view）：RR隔离级别，视图在事务启动时创建；RC级别下是每个SQL执行开始时创建

    > RC和RR推荐使用场景：导出表数据过程中，不想受到数据变化的干扰，这时使用RR事务启动即创建视图，不受其他事务更新的影响。

    > 事务未提交，会保留事务期间所有的回滚日志。因此避免使用长事务，其保留的回滚日志会很多。

    > 使用 set autocommit=1，并使用显式语句的方式来启动事务。

    > 查询持续时间超过60s的长事务
select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60

* 08 |19’| 事务到底是隔离的还是不隔离的？
    > innodb 每个事务都有唯一ID transaction id。
    
    > 每次事务更新数据的时候，都会生成一个新的数据版本，并且把 transaction id 赋值给这个数据版本的事务 ID，记为 row trx_id。同时，旧的数据版本要保留，并且在新的数据版本中，能够有信息可以直接拿到它。

    > 一个数据版本，对于一个事务视图来说，除了自己的更新总是可见以外，有三种情况：版本未提交，不可见；版本已提交，但是是在视图创建后提交的，不可见；版本已提交，而且是在视图创建前提交的，可见。

    > 更新数据的时候，就不能再在历史版本上更新了，否则事务的更新就丢失了。更新数据都是先读后写的，而这个读，只能读当前的值，称为“当前读”（current read）。

    > 另外加锁也是当前读

## 索引相关【126’】
* 04 |15’| 深入浅出索引（上）
* 05 |10’| 深入浅出索引（下）
* 09 |16’| 普通索引和唯一索引，应该怎么选择？
  
  知识点总结：更新数据时，innodb 使用change buffer优化数据页的读取
    > 使用索引检索过程：找到记录所在的数据页，读入内存，innodb中每个数据页大小默认16KB

    > change buffer、更新一个数据页，如果数据页不在内存，在不影响数据一致性的前提下，InnoDB 会将这些更新操作缓存在 change buffer 中，这样就不需要从磁盘中读入这个数据页了。在下次查询需要访问这个数据页的时候，将数据页读入内存，然后执行 change buffer 中与这个页有关的操作，merge数据到原数据页。change buffer 用的是 buffer pool 里的内存，因此不能无限增大。
    
    > 对于写多读少的业务来说，页面在写完以后马上被访问到的概率比较小，此时 change buffer 的使用效果最好。这种业务模型常见的就是账单类、日志类的系统。否则频繁触发merge反而不能提高效率

    > 对于唯一索引来说，所有的更新操作都要先判断这个操作是否违反唯一性约束，校验是否唯一，而这必须要将数据页读入内存才能判断，都已经读到内存了也就不需要使用 change buffer。因此非唯一的普通索引才能使用 change buffer

    > 普通索引和唯一索引应该怎么选择。其实，这两类索引在查询能力上是没差别的，主要考虑的是对更新性能的影响。所以，我建议你尽量选择普通索引。如果所有的更新后面，都马上伴随着对这个记录的查询，那么你应该关闭 change buffer。而在其他情况下，change buffer 都能提升更新性能。

    > redo log 主要节省的是随机写磁盘的 IO 消耗（转成顺序写）WAL，而 change buffer 主要节省的则是随机读磁盘的 IO 消耗。

* 10 |18’| MySQL为什么有时候会选错索引？

  知识点总结：优化器的索引选择
    > 这个例子对应的是我们平常不断地删除历史数据和新增数据的场景。这时，MySQL 竟然会选错索引，是不是有点奇怪呢？今天，我们就从这个奇怪的结果说起吧。

    > 扫描行数是影响执行代价的因素之一。扫描的行数越少，意味着访问磁盘数据的次数越少，消耗的 CPU 资源越少。当然，扫描行数并不是唯一的判断标准，优化器还会结合是否使用临时表、是否排序等因素进行综合判断。

    > analyze table t 可以用来解决索引很多问题

* 11 |13’| 怎么给字符串字段加索引？

  知识点总结：全字段加索引；前缀索引；字段倒叙存，建索引；字段 hash
    > 字符串前缀可以优化存储，但是会增加检索次数，同时需要回表确认原值（用不了覆盖索引），因此需要做取舍，优化前缀的长度`alter table SUser add index index2(email(6));`

    > 都不支持范围查询。倒序存储的字段上创建的索引是按照倒序字符串的方式排序的，已经没有办法利用索引方式查出身份证号码在[ID_X, ID_Y]的所有市民了。同样地，hash 字段的方式也只能支持等值查询。


* 15 |21’| 答疑文章（一）：日志和索引相关问题
    > 更新语句是否会先比较再更新

* 16 |17’| “order by”是怎么工作的？
    > 分配sort buffer 排序，如果内存放不下数据集，则会利用磁盘来辅助排序。

    > 全字段排序：将select数据全部从表里捞出来，放入内存排序后返回；
    > rowid排序：如果select字段太多，内存放不下，则先把排序字段和主键ID捞出来，排序好了后再回表捞出其他字段。

* 18 |16’| 为什么这些SQL语句逻辑相同，性能却差异巨大？
    > 索引上使用函数会导致无法使用索引，优化器则全表扫描，造成没走索引的效果，因为函数有可能会破坏索引值的有序性，走不了索引树
    以及隐式的转换
    > 类型转换
    > 编码集

## 日志与主备相关【183’】
* 02 |15’| 日志系统：一条SQL更新语句是如何执行的？ 
    ![](https://static001.geekbang.org/resource/image/0d/d9/0d2070e8f84c4801adbfa03bda1f98d9.png?wh=1920*1440)
    
    redolog/binlog 两阶段提交

* 12 |16’| 为什么我的MySQL会“抖”一下？
    > WAL 先写内存，一定的时机刷脏页。内存 buffer bool
    > 利用 WAL 技术，数据库实现顺序写日志到磁盘，大大提升了数据库的性能。但是，由此也带来了内存脏页的问题。脏页会被后台线程自动 flush，也会由于数据页淘汰而触发 flush，而刷脏页的过程由于会占用资源，可能会让你的更新和查询语句的响应时间长一些。



* 23 |17’| MySQL是怎么保证数据不丢的？
    > MySQL 写入 binlog 和 redo log 的流程。
    binlog 的写入逻辑比较简单：事务执行过程中，先把日志写到 binlog cache，事务提交的时候，再把 binlog cache 写到 binlog 文件中。

    ![](https://static001.geekbang.org/resource/image/9e/3e/9ed86644d5f39efb0efec595abb92e3e.png?wh=1142*748)

    图中的 write，指的就是指把日志写入到文件系统的 page cache，并没有把数据持久化到磁盘，所以速度比较快。图中的 fsync，才是将数据持久化到磁盘的操作。一般情况下，我们认为 fsync 才占磁盘的 IOPS。

    > 内存 -> page cache -> 磁盘 
    存在 redo log buffer 中，物理上是在 MySQL 进程内存中，就是图中的红色部分；
    写到磁盘 (write)，但是没有持久化（fsync)，物理上是在文件系统的 page cache 里面，也就是图中的黄色部分；
    持久化到磁盘，对应的是 hard disk，也就是图中的绿色部分。

* 24 |20’| MySQL是怎么保证主备一致的？
    > ![](https://static001.geekbang.org/resource/image/a6/a3/a66c154c1bc51e071dd2cc8c1d6ca6a3.png?wh=1142*856)
    binlog 以及两种模式：一种是 statement，一种是 row。可能你在其他资料上还会看到有第三种格式，叫作 mixed，其实它就是前两种格式的混合。

`insert into t values(10,10, now());` 如果直接用statement模式，直接拿去执行，会不一致；mix模式下 原来 binlog 在记录 event 的时候，多记了一条命令：SET TIMESTAMP=1546103491。它用 SET TIMESTAMP 命令约定了接下来的 now() 函数的返回时间。

> 25 |18’| MySQL是怎么保证高可用的？

数据同步会有延迟，延迟的几个原因：
- 从库性能交差，处理 binlog 不及时
- 从库读请求很多，压力大
- 主库有大事务（操作数据量多）长事务（耗时长）：主库必须等事务执行完才会去写binlog，比如delete大量数据，大表的DDL

> 26 |24’| 备库为什么会延迟好几个小时？

原因 sql_thread 更新数据时，速度跟不上。开多线程去执行SQL提高处理速度：

- 按表分发策略；
- 按行分发

> 27 |20’| 主库出问题了，从库怎么办？

主备切换时的同步问题

![](https://static001.geekbang.org/resource/image/00/53/0014f97423bd75235a9187f492fb2453.png?wh=1142*856)

主备切换，备节点与其他从节点的同步：
- 位点切换：从节点获取备节点
- GTID

  GTID 的全称是 Global Transaction Identifier，也就是全局事务 ID，是一个事务在提交的时候生成的，是这个事务的唯一标识

  我们在实例 B 上执行 start slave 命令，取 binlog 的逻辑是这样的：
  1. 实例 B 指定主库 A’，基于主备协议建立连接。
  2. 实例 B 把 set_b 发给主库 A’。
  3. 实例 A’算出 set_a 与 set_b 的差集，也就是所有存在于 set_a，但是不存在于 set_b 的 GTID 的集合，判断 A’本地是否包含了这个差集需要的所有 binlog 事务。

    - 如果不包含，表示 A’已经把实例 B 需要的 binlog 给删掉了，直接返回错误；
    - 如果确认全部包含，A’从自己的 binlog 文件里面，找出第一个不在 set_b 的事务，发给 B；
  4. 之后就从这个事务开始，往后读文件，按顺序取 binlog 发给 B 去执行。

> 28 |22’| 读写分离有哪些坑？

如果解决因为从库时延导致的过期读

两种集群架构：
![](https://static001.geekbang.org/resource/image/13/aa/1334b9c08b8fd837832fdb2d82e6b0aa.png?wh=1142*637)
![](https://static001.geekbang.org/resource/image/1b/45/1b1ea74a48e1a16409e9b4d02172b945.jpg?wh=1142*668)

直连与proxy：

- 客户端直连方案，因为少了一层 proxy 转发，所以查询性能稍微好一点儿，并且整体架构简单，排查问题更方便。但是这种方案，由于要了解后端部署细节，所以在出现主备切换、库迁移等操作的时候，客户端都会感知到，并且需要调整数据库连接信息。你可能会觉得这样客户端也太麻烦了，信息大量冗余，架构很丑。其实也未必，一般采用这样的架构，一定会伴随一个负责管理后端的组件，比如 Zookeeper，尽量让业务端只专注于业务逻辑开发。

- 带 proxy 的架构，对客户端比较友好。客户端不需要关注后端细节，连接维护、后端信息维护等工作，都是由 proxy 完成的。但这样的话，对后端维护团队的要求会更高。而且，proxy 也需要有高可用架构。因此，带 proxy 架构的整体就相对比较复杂。

> 29 |15’| 如何判断一个数据库是不是出问题了？

健康探测的几种方式：select 1 看库进程是否存活；查表判断，查询并发线程数是否剩余可用；更新操作，判断磁盘可用，更新会有binlog使用磁盘；系统库performance_schema 下面的表的指标统计

> 31 |16’| 误删数据后除了跑路，还能怎么办？

## 临时表相关【95’】
* 17 |17’| 如何正确地显示随机消息？
* 34 |17’| 到底可不可以使用join？
* 35 |17’| join语句怎么优化？
* 36 |17’| 为什么临时表可以重名？
* 37 |14’| 什么时候会使用内部临时表？
* 43 |13’| 要不要使用分区表？

## 实用性归类【151’】
* 13 |15’| 为什么表数据删掉一半，表文件大小不变？
* 14 |16’| count(* )这么慢，我该怎么办？
* 22 |15’| MySQL有哪些“饮鸩止渴”提高性能的方法？
* 32 |14’| 为什么还有kill不掉的语句？
* 33 |15’| 我查这么多数据，会不会把数据库内存打爆？
* 38 |14’| 都说InnoDB好，那还要不要使用Memory引擎？
* 39 |18’| 自增主键为什么不是连续的？
    > 表的自增值(下一条插入row的自增ID值)是会持久化的
    > innodb 只保证自增ID递增，但不保证连续
    > 自增值被同时间段的多个操作顶上去，中间自增值不会回退，要保证安全
* 41 |14’| 怎么最快地复制一张表？
* 42 |13’| grant之后要跟着flush privileges吗？
* 44 |17’| 答疑文章（三）：说一说这些好问题 
* 45 |18’| 自增id用完怎么办？ 

# SQL 

## CMD

```sql
-- 查看变量 事务隔离级别
show variables like 'transaction_isolation';

-- 命令查看 Waiting for table metadata lock 的示意图
show processlist 

通过查询 sys.schema_table_lock_waits 这张表，我们就可以直接找出造成阻塞的 process id，把这个连接用 kill 命令断开即可。

-- 查看innodb状态，包含锁信息
show engine innodb status


-- 查看慢日志 slow log

```

## 编程
```sql
-- 创建表，声明函数调用
mysql> CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `c` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB;

-- sql语句一般遇到符号;，就会被sql解析器立刻执行，但被delimiter包裹的sql语句代码块，则不会立刻被sql解析器一句一句执行，而是作为一个整体被sql解析器执行

-- 插入10万条数据
delimiter ;;
create procedure idata()
begin
  declare i int;
  set i=1;
  while(i<=100000) do
    insert into t values(i,i);
    set i=i+1;
  end while;
end;;
delimiter ;

-- 开启事务一次提交
delimiter ;; 
create procedure idata() 
begin 
    declare i int; 
    set i = 1; 
    start transaction ; 
    while(i <= 100000) do 
        insert into t values (i, i); 
        set i = i + 1; 
    end while; 
    commit ; 
end ;; 
delimiter ;

call idata();
```

# 概念

## 索引

唯一索引，非唯一索引；

覆盖索引、索引回表；

## 两种视图

- view虚拟表

- innoDB实现MVCC时的 一致性读视图 consistent read view 用于支持 RC，RR 隔离级别实现。是一个逻辑概念。有时会用read view指代

- 基于一致性视图 consistent read view 的读行为，称为一致性读 consistent read。与之对应，如果读的row最新数据，就是当前读 current read

## select for update
两个事务同时进行中，双方使用 `SELELT FOR UPDATE` 会互斥;

未获取到锁的一方，select 就会失败，更别说 Update 了；

TX A：select for udpate;
TX B：使用普通select会成功，然后update被阻塞，A提交后，B update开始执行，此时B提交后，会成功，此时A的提交被覆盖；

如果双方修改同一记录，都需要使用 SELECT FOR UPDATE，保证B看到的记录是A提交后新数据。


## MVCC

查询SELECT: 使用是一致性读consistent read：

更新UPDATE: 使用当前读current read，即只能最新版本上更新；

如果让SELECT读到最新版本数据，实现current read，可以加锁实现
读锁(S锁共享锁)或者写锁(X锁排他锁)
```sql
select ... lock in share mode;
select ... for update;
```

## 一致性读

InnoDB为每个事务构造一个 trx_id 数组，保存TX启动瞬间正在活跃的ID（启动未提交），利用这个活跃 TX数组 和DB里面 trx_id 最大值+1 组成
一致性读视图，即 consistent read view，规定能看到哪些数据

版本trx_id未提交，不可见；

版本trx_id已提交，但是是在视图创建后提交的，不可见；

版本trx_id已提交，而且是在视图创建前提交的，可见。

- 读提交RC 和 可重复读RR

而读提交的逻辑和可重复读的逻辑类似，它们最主要的区别是：

在可重复读RR隔离级别下，只需要在事务开始的时候创建一致性视图，之后事务里的其他查询都共用这个一致性视图；

在读提交RC隔离级别下，每一个语句执行前都会重新算出一个新的视图。


# 日志 undo log | binlog | redo log

undo log MVCC 中版本回滚

## binlog（归档日志）。

binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用。

binlog 是逻辑日志，记录的是这个语句的原始逻辑，比如“给 ID=2 这一行的 c 字段加 1 ”。

binlog 是可以追加写入的。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。

Binlog有两种模式，`statement` 格式的话是记sql语句，`row` 格式会记录行的内容，记两条，更新前和更新后都有。

还会看到有第三种格式，叫作 mixed，其实它就是前两种格式的混合。

## redo log

redo log 是 InnoDB 引擎特有的；
redo log（重做日志）,WAL 先写内存再更新磁盘文件，redolog buffer 就是分配的内存
redo log 是循环写的，分配固定空间，写到末尾是要回到开头继续写的。这样历史日志没法保留，redo log 也就起不到归档的作用。
redo log 是物理日志，记录的是“在某个数据页上做了什么修改”；

## 事务的两阶段提交

redolog 先prepare，等待binlog写完之后，redolog再commit，事务完成。

binlog 通过 XID 与 redolog 关联。

- 崩溃恢复的时候，会按顺序扫描 redo log：如果碰到既有 prepare、又有 commit 的 redo log，就直接提交；

- 如果碰到只有 parepare、而没有 commit 的 redo log，就拿着 XID 去 binlog 找对应的事务。
