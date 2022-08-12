Mysql笔记
=========================
mysql默认的隔离级别是`REPEATABLE-READ`   ---可重复读
## 一、慢查询日志
    SHOW VARIABLES LIKE "%slow_query%";  查看慢查询日志相关配置
    set global slow_query_log=ON         全局开启慢查询
    SHOW VARIABLES LIKE "%long_query%"   查看触发的时间
    set long_query=0.1;                  修改触发时间
    show VARIABLES LIKE "profiling";      
    set profiling=on;                    打开配置，可以查看sql的具体执行时间
* 1、开启慢查询日志：设置合理的业务可以接受的查询时间
* 2、进行压力测试
* 3、查看日志，找出耗时的sql
* 4、使用`explain` 对相关sql进行分析

## 二、事务
* select @@autocommit;查看是否自动提交
* set autocommit=1;  自动提交：1：是，0：否
`开启-》提交-》回滚`,要么同时成功，要么同时失败。
### 2.1、特性（ACID）
* `原子性（Atomic）`：要么全部成功，要么全部失败。
* `隔离性（Consistency）`：sql的执行前后，数据库数据必须保持一致,由`锁机制`来实现保证的。
* `一致性（Isolation）`：并发执行的各个事务必须互相隔离。
* `持久性（Durability）`：事务完成后，对数据的修改是永久性的。

> `redo log`:重做日志，保证数据库的永久性（断电|宕机，数据恢复的原因）。

> `undo log`:回滚日志
#### 备注
* mysql最重要的是日志，不是数据。
* 隔离的越严格，并发性越差。
### 2.2、事务并发存在的问题
* 1、`脏读`：一个事务读取里另一个事务未提交的数据。
* 2、`不可重复读`:一个事物的操作导致另一个事务前后两次读取到不同的数据。
* 3、`虚读`:幻读：一个事物的操作导致另一个事务前后两次查询的结果数据量不同。
---（事务B读取了事务A新增的数据或者读不到事务A删除的数据）
### 2.3、事物的隔离级别
    隔离级别       脏读       不可重复读       幻读
    未提交读       可以         可以           可以
    已提交读       不可以       可以           可以
    可重复读       不可以       不可以          可以
    串行读         不可以       不可以         不可以
* 隔离级别越高，并发性越差
#### select @@tx_isolation;  查询数据库的隔离级别   
#### set tx_isolation='REPEATABLE-READ';  修改数据库的隔离级别
* TRANSACTION_READ_UNCOMMITTED---未提交读
* TRANSACTION_READ_COMMITTED   ---已提交读
* TRANSACTION_REPEATABLE_READ   ---可重复读（无法防止`update`产生的幻读）
* TRANSACTION_SERIALIZABLE      ---串行读(`间隙锁，相当于表锁吗？`)
### 2.4、事物的命令
* begin   开启事务
* commit   提交数据
* rollback   回滚



