# doc

MySQL 是怎样运行的：从根儿上理解 MySQL - 小孩子4919 - 掘金小册
https://juejin.cn/book/6844733769996304392/section/6844733770042441736

# 基础运用

## 数据类型

如果1个字符用一个字节表示：
varchar 是变长字段，varchar(255) 表示最大255个Byte，其长度表示用1个Byte表示，最多会占用256Byte，而varchar(256)需要2个字节表示其长度。

char 定长，占用大小由用户决定，类似数组的分配，最多255Byte (只用1byte表示长度）。

int 定长，占用大小固定4个字节

# Mysql 实战 45 讲

| * 编号｜建议阅读时长｜文章标题

## 基础知识【12’】
### 01 |12’| 基础架构：一条SQL查询语句是如何执行的？ 
### 13 |15’| 为什么表数据删掉一半，表文件大小不变？

参数 innodb_file_per_table表数据既可以存在共享表空间里，也可以是单独的文件。这个行为是由参数 innodb_file_per_table 控制的：我建议你不论使用 MySQL 的哪个版本，都将这个值设置为 ON。因为，一个表单独存储为一个文件更容易管理，而且在你不需要这个表的时候，通过 drop table 命令，系统就会直接删除这个文件。而如果是放在共享表空间中，即使表删掉了，空间也是不会回收的。

- 数据空洞

delete 命令其实只是把记录的位置，或者数据页标记为了“可复用”，但磁盘文件的大小是不会变的。也就是说，通过 delete 命令是不能回收表空间的。这些可以复用，而没有被使用的空间，看起来就像是“空洞”。

不止是删除数据会造成空洞，插入数据也会。

- 重建表

你可以使用 alter table A engine=InnoDB 命令来重建表。在 MySQL 5.5 版本之前，这个命令的执行流程跟我们前面描述的差不多，区别只是这个临时表 B 不需要你自己创建，MySQL 会自动完成转存数据、交换表名、删除旧表的操作。

alter table t engine = InnoDB（也就是 recreate）默认的就是上面图 4 的流程了；

analyze table t 其实不是重建表，只是对表的索引信息做重新统计，没有修改数据，这个过程中加了 MDL 读锁；

optimize table t 等于 recreate+analyze。

- 重建表空间预留

页会按90%满的比例来重新整理页数据（10%留给UPDATE使用），
未整理之前页已经占用90%以上，收缩之后，文件就反而变大了。

> 22 |15’| MySQL有哪些“饮鸩止渴”提高性能的方法？

- 短连接风暴

正常的短连接模式就是连接到数据库后，执行很少的 SQL 语句就断开，下次需要的时候再重连。

设置 wait_timeout 参数表示的是，一个线程空闲 wait_timeout 这么多秒之后，就会被 MySQL 直接断开连接。腾出位置来

> 32 |14’| 为什么还有kill不掉的语句？

kill query + 线程 id，表示终止这个线程中正在执行的SQL语句；

一个是 kill connection + 线程 id，这里 connection 可缺省，表示断开这个线程的连接，当然如果这个线程有语句正在执行，也是要先停止正在执行的语句的。

> 33 |15’| 我查这么多数据，会不会把数据库内存打爆？

MySQL 是“边读边发的”，这个概念很重要。这就意味着，如果客户端接收得慢，会导致 MySQL 服务端由于结果发不出去，这个事务的执行时间变长。

–quick 参数，会使用 mysql_use_result 方法。这个方法是 client 读一行处理一行，如果client读的很慢会影响MySQL server 性能

如果一个查询的返回结果不会很多的话，建议使用 mysql_store_result 这个接口，直接把查询结果保存到 client 本地内存。当然前提是查询返回结果不多。在第 30 篇文章评论区，有同学说到自己因为执行了一个大查询导致客户端占用内存近 20G，这种情况下就需要改用 mysql_use_result 接口了。

> 39 |18’| 自增主键为什么不是连续的？

- 自增值被同时间段的多个操作顶上去，中间自增值不会回退，要保证安全
- 表的自增值(下一条插入row的自增ID值)是会持久化的（容灾）
- innodb 只保证自增ID递增，但不保证连续

> 41 |14’| 怎么最快地复制一张表？

用 mysqldump 生成包含 INSERT 语句文件的方法，可以在 where 参数增加过滤条件，来实现只导出部分数据。这个方式的不足之一是，不能使用 join 这种比较复杂的 where 条件写法。

用 select … into outfile 的方法是最灵活的，支持所有的 SQL 写法。但，这个方法的缺点之一就是，每次只能导出一张表的数据，而且表结构也需要另外的语句单独备份。

另外一种是物理拷贝

### 42 |13’| grant之后要跟着flush privileges吗？

grant 和 revoke 命令的执行逻辑。grant 语句会同时修改数据表和内存，判断权限的时候使用的是内存数据。因此，规范地使用 grant 和 revoke 语句，是不需要随后加上 flush privileges 语句的。

flush privileges 使用场景那么，flush privileges 是在什么时候使用呢？显然，当数据表中的权限数据跟内存中的权限数据不一致的时候，flush privileges 语句可以用来重建内存数据，达到一致状态。

使用 grant 语句赋权时，你可能还会看到这样的写法：
```sql
grant super on *.* to 'ua'@'%' identified by 'pa';
```
这条命令加了 identified by ‘密码’， 语句的逻辑里面除了赋权外，还包含了：如果用户’ua’@’%'不存在，就创建这个用户，密码是 pa；如果用户 ua 已经存在，就将密码修改成 pa。

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

在 InnoDB 实现上，按照 5:3 的比例把整个 LRU 链表分成了 young 区域和 old 区域

这个策略，就是为了处理类似全表扫描的操作量身定制的。还是以刚刚的扫描 200G 的历史数据表为例，我们看看改进后的 LRU 算法的操作逻辑：
1. 扫描过程中，需要新插入的数据页，都被放到 old 区域 ;
2. 一个数据页里面有多条记录，这个数据页会被多次访问到，但由于是顺序扫描，这个数据页第一次被访问和最后一次被访问的时间间隔不会超过 1 秒，因此还是会被保留在 old 区域；
3. 再继续扫描后续的数据，之前的这个数据页之后也不会再被访问到，于是始终没有机会移到链表头部（也就是 young 区域），很快就会被淘汰出去。

可以看到，这个策略最大的收益，就是在扫描这个大表的过程中，虽然也用到了 Buffer Pool，但是对 young 区域完全没有影响，从而保证了 Buffer Pool 响应正常业务的查询命中率。


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

> 14 |16’| count(* )这么慢，我该怎么办？

MyISAM 引擎把一个表的总行数存在了磁盘上，因此执行 count(*) 的时候会直接返回这个数，效率很高；(不加where筛选条件下)

而 InnoDB 引擎就麻烦了，因为使用了 MVCC 它执行 count(*) 的时候，需要把数据一行一行地从引擎里面读出来，然后累积计数。

还记得在第 10 篇文章《 MySQL 为什么有时候会选错索引？》中我提到过，索引统计的值是通过采样来估算的。实际上，TABLE_ROWS 就是从这个采样估算得来的，因此它也很不准。有多不准呢，官方文档说误差可能达到 40% 到 50%。所以，show table status 命令显示的行数也不能直接使用。

- 业务层面解决这个问题

使用Redis计数，会有缓存——DB数据不一致的问题

存放在DB，利用事务去跟新数据。注意计数表是公共的，锁竞争几率大，因此尽量减少锁占用时间，放到事务最后执行

- count不同字段的区别

`count(字段) < count(主键 id) < count(1) ≈ count(*)，所以我建议你，尽量使用 count(*)`

count() 是一个聚合函数，对于返回的结果集，一行行地判断，如果 count 函数的参数不是 NULL，累计值就加 1，否则不加。最后返回累计值。

对于 count(字段) 来说：如果这个“字段”是定义为 not null 的话，一行行地从记录里面读出这个字段，判断不能为 null，按行累加；如果这个“字段”定义允许为 null，那么执行的时候，判断到有可能是 null，还要把值取出来再判断一下，不是 null 才累加。

对于 count(主键 id) 来说，InnoDB 引擎会遍历整张表，把每一行的 id 值都取出来，返回给 server 层。server 层拿到 id 后，判断是不可能为空的，就按行累加。

对于 count(1) 来说，InnoDB 引擎遍历整张表，但不取值。server 层对于返回的每一行，放一个数字“1”进去，判断是不可能为空的，按行累加。单看这两个用法的差别的话，你能对比出来，count(1) 执行得要比 count(主键 id) 快。因为从引擎返回 id 会涉及到解析数据行，以及拷贝字段值的操作。

`但是 count(*) 是例外，并不会把全部字段取出来，而是专门做了优化，不取值。count(*) 肯定不是 null，按行累加。`

> 38 |14’| 都说InnoDB好，那还要不要使用Memory引擎？

可见，InnoDB 和 Memory 引擎的数据组织方式是不同的：InnoDB 引擎把数据放在主键索引上，其他索引上保存的是主键 id。这种方式，我们称之为索引组织表（Index Organizied Table）。

而 Memory 引擎采用的是把数据单独存放，索引上保存数据位置的数据组织形式，我们称之为堆组织表（Heap Organizied Table）。内存表

### 45 |18’| 自增id用完怎么办？ 

每种自增 id 有各自的应用场景，在达到上限后的表现也不同：
- 表的自增 id 达到上限后，再申请时它的值就不会改变，进而导致继续插入数据时报主键冲突的错误。
- 表没有自增ID，会使用 InnoDB 全局维护的 row_id 达到上限后，则会归 0 再重新递增，如果出现相同的 row_id，后写的数据会覆盖之前的数据。
- binlog 和 redolog 想关联用的 Xid 只需要不在同一个 binlog 文件中出现重复值即可。虽然理论上会出现重复值，但是概率极小，可以忽略不计。

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

删除数据的种类：
- 删除数据行 delete：
    可以通过 binlog 来恢复

- 删表 drop table 或者 truncate table：
    delete 全表是很慢的，需要生成回滚日志、写 redo、写 binlog。所以，从性能角度考虑，你应该优先考虑使用 truncate table 或者 drop table 命令。
- 删库 drop database：删表删库都只能通过全量数据备份+增量日志备份恢复
- 删除实例 rm：高可用最不怕删除一个实例，别集群下线就行

四个安全脚本：备份脚本、执行脚本、验证脚本和回滚脚本

## 临时表相关【95’】
> 17 |17’| 如何正确地显示随机消息？

order by rand()  会扫描大量的行数，排序过程消耗内存（甚至磁盘）资源；Using temporary 和 Using filesort(需要排序) 新建临时表

* 34 |17’| 到底可不可以使用join？
    
遍历驱动表Left，查找被驱动表Right：驱动表Left行数少，Right在连接字段上有索引，则比较快

被驱动表可以使用索引： Index Nested-Loop Join 
不能使用索引 Block Nested-Loop Join 

如果被驱动表是大表，且是冷数据，则LRU会将内存中的热数据淘汰冷数据前移，导致业务热数据内存命中率低，查询速度慢

> 35 |17’| join语句怎么优化？

1. 被驱动表有索引时的优化 Index Nested-Loop Join (NLJ)

- Multi-Range Read 优化

索引范围，得到多个主键ID，对主键ID排序，通过主键ID索引数据，达到顺序性读磁盘的优势。

原理：因为大多数的数据都是按照主键递增顺序插入得到的，所以我们可以认为，如果按照主键的递增顺序查询的话，对磁盘的读比较接近顺序读，能够提升读性能。

- Batched Key Access 优化 BKA

对于被驱动表的访问是一行行的，因此可以先把Left数据取出来放到临时内存 join_buffer 赞一批再去查 被驱动表 Right

2. 被驱动表没有索引的优化 Block Nested-Loop Join（BNL）的优化方法

尽量转成上面带索引的情况 BKA，可以通过 原表加索引 或者 临时表
```sql
create temporary table temp_t like t1;
alter table temp_t add index(b);
insert into temp_t select * from t2 where b>=1 and b<=2000;
```

> 36 |17’| 为什么临时表可以重名？

- 建表语法是 create temporary table …。
- 一个临时表只能被创建它的 session 访问，对其他线程不可见。所以，图中 session A 创建的临时表 t，对于 session B 就是不可见的。
- 临时表可以与普通表同名。
- session A 内有同名的临时表和普通表的时候，show create 语句，以及增删改查语句访问的是临时表。
- show tables 命令不显示临时表。

临时表经常会被用在复杂查询的优化过程中。其中，分库分表系统的 跨库 查询就是一个典型的使用场景。

会写 binlog，就意味着备库也需要创建临时表

对于临时表重命名需要用 alter table temp_t rename to temp_t2, 直接修改的是table_def_key，

为什么不能用 rename 修改临时表的改名。在实现上，执行 rename table 语句的时候，要求按照“库名 / 表名.frm”的规则去磁盘找文件，但是临时表在磁盘上的 frm 文件是放在 tmpdir 目录下的，并且文件名的规则是“#sql{进程 id}_{线程 id}_ 序列号.frm”，因此会报“找不到文件名”的错误。

> 37 |14’| 什么时候会使用内存(内部)临时表？

union、union all 和 group by 语句的执行过程的分析，我们来回答文章开头的问题：MySQL 什么时候会使用内部临时表？
- 如果语句执行过程可以一边读数据，一边直接得到结果，是不需要额外内存的，否则就需要额外的内存，来保存中间结果；

- join_buffer 是无序数组，sort_buffer 是有序数组，临时表是二维表结构；如果执行逻辑需要用到二维表特性，就会优先考虑使用临时表。比如我们的例子中，union 需要用到唯一索引约束， group by 还需要用到另外一个字段来存累积计数。

group by 的几种实现算法，从中可以总结一些使用的指导原则：- 如果对 group by 语句的结果没有排序要求，要在语句后面加 order by null；
- 尽量让 group by 过程用上表的索引，确认方法是 explain 结果里没有 Using temporary 和 Using filesort；
- 如果 group by 需要统计的数据量不大，尽量只使用内存临时表；也可以通过适当调大 tmp_table_size 参数，来避免用到磁盘临时表；
- 如果数据量实在太大，使用 SQL_BIG_RESULT 这个提示，来告诉优化器直接使用磁盘临时表，进行排序

### 43 |13’| 要不要使用分区表？

分区SQL
```sql
CREATE TABLE `t` (
  `ftime` datetime NOT NULL,
  `c` int(11) DEFAULT NULL,
  KEY (`ftime`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
PARTITION BY RANGE (YEAR(ftime))
(PARTITION p_2017 VALUES LESS THAN (2017) ENGINE = InnoDB,
 PARTITION p_2018 VALUES LESS THAN (2018) ENGINE = InnoDB,
 PARTITION p_2019 VALUES LESS THAN (2019) ENGINE = InnoDB,
PARTITION p_others VALUES LESS THAN MAXVALUE ENGINE = InnoDB);
insert into t values('2017-4-1',1),('2018-4-1',1);
```

分区表和手工分表，一个是由 server 层来决定使用哪个分区，一个是由应用层代码来决定使用哪个分表。因此，从引擎层看，这两种方式也是没有差别的。

如果从 server 层看的话，一个分区表就只是一个表。

分区表的一个显而易见的优势是对业务透明，相对于用户分表来说，使用分区表的业务代码更简洁。还有，分区表可以很方便的清理历史数据。如果一项业务跑的时间足够长，往往就会有根据时间删除历史数据的需求。这时候，按照时间分区的分区表，就可以直接通过 alter table t drop partition ... 这个语法删掉分区，从而删掉过期的历史数据。这个 alter table t drop partition ... 操作是直接删除分区文件，效果跟 drop 普通表类似。与使用 delete 语句删除数据相比，优势是速度快、对系统影响小。

### 44 |17’| 答疑文章（三）：说一说这些好问题 

left join 的语义，就不能把被驱动表的字段放在 where 条件里面做等值判断或不等值判断，必须都写在 on 里面。

# SQL 

## 问题思考

- Mysql中使用json格式存储数据好吗？

https://blog.csdn.net/zhongzhen7355/article/details/100092893

不能用于结构化查询！

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
