mysql笔记
===============================
## 一、完整性约束
    1:唯一键         primary key
    2:自增约束       auto_increment
    3:唯一键         unique
    4:非空约束       not null
    5:默认值约束     default
    6:外键约束       foreign key

## 二、sql语句
#### 1、常用的库操作SQL
    show databases;       查看数据库
    show engines;         查看数据库引擎（innoDB,myisam...）
    use xxx;              使用xxx库
    create database xxx;  创建一个xxx的库
    drop database xxx;    删除xxx的数据库
#### 2、常用的表操作SQL
    show tables;                   查看表列表
    create table xxx(
      id INT primary key auto_increment unsigned not null  comment "主键ID",
    )engine=INNODB default charset=utf8;  创建表   
    desc 表名 or 表名\G;            查看表结构
    drop table 表名；               删除表
    show create 表名；              查看表创建的语句
* mysql设置默认存储引擎修改   win(my.ini)      linux(my.conf)

#### 3、查看sql的执行计划
>使用explain select * from xxx;查看sql执行计划是否使用了索引，
> 如果：`Extra`显示`Using filesort`
>      `index`显示 `all`
> 则证明使用了外排序，并没有使用索引查询，会影响查询速度，因为会回表查询

## 三、表设计
>规范设计是尽量少的减少重复字段的显示，多做分表处理，以减少冗余数据。
* 但是根据执行效率来说，应该尽量的减少链表查询操作。

#### 关于链表查询
* left JOIN和inner join 表顺序都是  左->右
* right join 表顺序   右->左
    
## 备注
链表查询尽量不适用范围查询（in,not in,between），因为很难命中索引，会影响查询速度。

#### left join 的问题
    1:explain select a.* from xxxx a left join bbb b on a.uid=b.uid where b.cid=3;
    2:explain select a.* from xxxx a left join bbb b on a.uid=b.uid and b.cid=3;
* 1,是先从B表进行筛选，在从a表进行筛选，顺序错误会导致与inner join的查询结果相同
* 2,才是正确的执行顺序 ,后面在加 where 进行条件判断
>在进行sql编写的时候最好注意此处的问题。

## 四、存储引擎
    种类       锁机制    B-树索引     哈希索引     外键      事务      索引缓存      数据缓存
    MyISAM     表锁       支持        不支持      不支持     不支持     支持         不支持
    InnoDB     行锁       支持        不支持      支持       支持       支持         支持
    Memory     表锁       支持        支持        不支持     不支持     支持         支持
* MyISAM和InnoDB采用的是`B+树`的原理
* 使用 show engines;查看存储引擎的相关信息
* `Memory`：是基于内存的存储引擎
* `MyISAM`存储特点：`结构，数据，索引`都是单独存储的，---对应三个不同的文件。所以需要自行添加索引。
* `InnoDb`:`结构，数据，索引` 是放在一个文件中的。---会自动添加主键索引。


## 五、索引-》提高查询效率
* 索引也会涉及磁盘IO操作，所以要避免大量的无用索引。
#### 1、物理上
    聚集索引   非聚集索引
#### 2、逻辑上
* 1、普通索引（二级索引）数量不限
* 2、唯一性索引：unique，值不能重复。
* 3、主键索引：primary key
* 4、单例索引：在一个字段上创建索引。
* 5、多列索引：多个字段创建索引。（必须用到第一个列，才能用到多列索引，否则无法命中索引）
* 6、全文索引：使用`FULLTEXT`参数可以创建全文索引，只支持`CHAR`,`VARCHAR`,`TEXT`类型的字段上。
通常使用`elasticsearch（java）简称：ES`代替全文索引。
#### 备注
一张表的一次sql查询只能用一个索引
* 创建索引：create [unique] index 索引名 on 表名（字段名 （字符串使用索引的长度length） [ASC|DESC]）
using hash|B tree.
* 删除索引：drop 索引名 on 表名
##### 说明：
    1：经常作为where条件过滤的字段需要考虑添加索引
    2：字符串列创建索引时，尽量规定索引长度
    3：查询字段与表字段类型不匹配，或者使用mysql函数、表达式计算等，无法使用索引。
#### 备注
* 平衡树：所有叶子节点都在同一层。（2000W数据最多3层）
* AVL:二叉平衡树。（2000W数据需要25层，读取一个索引需要花费25次磁盘IO)
* 数据、索引都是放在磁盘上   mysql_server->操作系统-》加载到进程中    需要消费磁盘IO
* 将数据读取到内存中    内存的读取是按照页面为单位的。页面以4k为单位。

    



    


