# Mysql 实战 45 讲


## 01 | 基础架构：一条SQL查询语句是如何执行的？: https://time.geekbang.org/column/article/68319

SQL 执行、MySQL系统组成：


## 02 | 日志系统：一条SQL更新语句是如何执行的？: https://time.geekbang.org/column/article/68633

- binlog
server系统SQL操作日志

- redo log
储存引擎的操作系统层级的变更日志，在某个数据页上做了什么修改。


## 索引：04 | 深入浅出索引（上）: https://time.geekbang.org/column/article/69236


# 锁

## 全局锁：
全库逻辑备份。Flush tables with read lock (FTWRL)，如果存储引擎支持事务，也可以在 Repeatable Read 隔离级别下操作
mysqldump 使用参数–single-transaction 

## 表级别的锁

表锁：LOCK TABLES ... READ/WRITE
有更细粒度行锁，表锁用得少

元数据锁 Metadata Lock（MDL）：

ALTER TABLE tbl_name WAIT N add column ..


## 行锁

innodb 的 行锁通过锁住索引来实现，如果update检索的列没有索引则会锁住所有整个表。此情况下可以通过加 limit 1 减少被锁住的行数





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

- 一致性读

InnoDB为每个事务构造一个 trx_id 数组，保存TX启动瞬间正在活跃的ID（启动未提交），利用这个活跃 TX数组 和DB里面 trx_id 最大值+1 组成
一致性读视图，即 consistent read view，规定能看到哪些数据

版本trx_id未提交，不可见；

版本trx_id已提交，但是是在视图创建后提交的，不可见；

版本trx_id已提交，而且是在视图创建前提交的，可见。

- 读提交RC 和 可重复读RR

而读提交的逻辑和可重复读的逻辑类似，它们最主要的区别是：

在可重复读RR隔离级别下，只需要在事务开始的时候创建一致性视图，之后事务里的其他查询都共用这个一致性视图；

在读提交RC隔离级别下，每一个语句执行前都会重新算出一个新的视图。