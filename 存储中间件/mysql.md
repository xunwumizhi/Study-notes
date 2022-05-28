# Mysql 实战 45 讲

| * 编号｜建议阅读时长｜文章标题

## 基础知识【12’】
* 01 |12’| 基础架构：一条SQL查询语句是如何执行的？ 

## 锁相关【85’】
> 弹锁要谈事务隔离级别

* 06 |13’| 全局锁和表锁 ：给表加个字段怎么有这么多阻碍？
* 07 |12’| 行锁功过：怎么减少行锁对性能的影响？
* 19 |13’| 为什么我只查一行的语句，也执行这么慢？
    > 构造表锁、等flush、行锁、一致性读 导致SQL查询慢的c场景景
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
    > 锁是一句SQL就加的，每个事务包含多个SQL，可能锁不同的行
    > 在读提交隔离级别 RC 下还有一个优化，即：语句执行过程中加上的行锁，在语句执行完成后，就要把“不满足条件的行”上的行锁直接释放了


* 30 |16’| 答疑文章（二）：用动态的观点看加锁
* 40 |13’| insert语句的锁为什么这么多？

## 事务相关【68’】
* 03 |11’| 事务隔离：为什么你改了我还看不见？ 
* 08 |19’| 事务到底是隔离的还是不隔离的？
* 39 |18’| 自增主键为什么不是连续的？

## 索引相关【126’】
* 04 |15’| 深入浅出索引（上）
* 05 |10’| 深入浅出索引（下）
* 09 |16’| 普通索引和唯一索引，应该怎么选择？
* 10 |18’| MySQL为什么有时候会选错索引？
* 11 |13’| 怎么给字符串字段加索引？
* 15 |21’| 答疑文章（一）：日志和索引相关问题 
* 16 |17’| “order by”是怎么工作的？
* 18 |16’| 为什么这些SQL语句逻辑相同，性能却差异巨大？



## 日志与主备相关【183’】
* 02 |15’| 日志系统：一条SQL更新语句是如何执行的？ 
* 12 |16’| 为什么我的MySQL会“抖”一下？
* 23 |17’| MySQL是怎么保证数据不丢的？
* 24 |20’| MySQL是怎么保证主备一致的？
* 25 |18’| MySQL是怎么保证高可用的？
* 26 |24’| 备库为什么会延迟好几个小时？
* 27 |20’| 主库出问题了，从库怎么办？
* 28 |22’| 读写分离有哪些坑？
* 29 |15’| 如何判断一个数据库是不是出问题了？
* 31 |16’| 误删数据后除了跑路，还能怎么办？

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
* 41 |14’| 怎么最快地复制一张表？
* 42 |13’| grant之后要跟着flush privileges吗？
* 44 |17’| 答疑文章（三）：说一说这些好问题 
* 45 |18’| 自增id用完怎么办？ 

# SQL 

## CMD

```sql
-- 命令查看 Waiting for table metadata lock 的示意图
show processlist 

通过查询 sys.schema_table_lock_waits 这张表，我们就可以直接找出造成阻塞的 process id，把这个连接用 kill 命令断开即可。

查看慢日志 slow log
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

## 两种视图

- view虚拟表

- innoDB实现MVCC时的 一致性读视图 consistent read view 用于支持 RC，RR 隔离级别实现。是一个逻辑算法概念


## 一致性读consistent read：

InnoDB为每个事务构造一个 trx_id 数组，保存TX启动瞬间正在活跃的ID（启动未提交），利用这个活跃 TX数组 和DB里面 trx_id 最大值+1 组成
一致性读视图，即 consistent read view，规定能看到哪些数据

版本trx_id未提交，不可见；

版本trx_id已提交，但是是在视图创建后提交的，不可见；

版本trx_id已提交，而且是在视图创建前提交的，可见。

## 当前读current read

# Mysql 实战 45 讲


## 01 | 基础架构：一条SQL查询语句是如何执行的？: https://time.geekbang.org/column/article/68319

SQL 执行、MySQL系统组成：


## 02 | 日志系统：一条SQL更新语句是如何执行的？: https://time.geekbang.org/column/article/68633

- binlog
server系统SQL操作日志

- redo log
储存引擎的操作系统层级的变更日志，在某个数据页上做了什么修改。


## 索引：04 | 深入浅出索引（上）: https://time.geekbang.org/column/article/69236


# 事务

```sql
-- 开启事务
START TRANSACTION;
BEGIN;

--
COMMIT;
--
ROLLBACK;
```

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


# undo log | binlog | redo log

undo log MVCC 中版本回滚