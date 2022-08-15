mysql主从复制
===============================
* 1、数据备份-热备份&容灾&高可用
* 2、读写分离、支持更大的并发
### 需要进去mysql SHELL
show processlist;----查看数据库开启的线程
### 流程
两个日志（`binlog二进制日志&relay log日志`）和三个线程（`master的一个线程和slave的2哥线程`）
* 1、主库的更新操作写入binlog二进制日志中
* 2、master服务器创建一个`binlog`转储线程，将二进制日志内容发送到从服务器。
* 3、slave机器执行`START SLAVE`命令会在从服务器创建一个IO线程，接受master的binary log复制到中继日志。
主库会产生一个binlog dump process线程
* 4、sql slave thread(`sql从线程`)处理该过程的最后一步，sql线程从`中继日志`中读取事件，并重放其中的事件
而更新`slave`机器的数量，使其与master的数据一致，只要该线程与IO线程保持一致，中继日志通常会位于`os`缓存中
，所以中继日志的开销很小
### 配置
#### 一、查看防火墙配置
* 1：systemctl status firewalled.service;  查看防火墙状态
* 2：systemctl start firewalled.service;  启动防火墙（临时）
* 3：systemctl stop firewalled.service;    停止防火墙（临时）
* 4：systemctl enable firewalled.service;    开启防火墙
* 5：systemctl disable firewalled.service;    关闭防火墙
#### 二、开放端口
* firewall-cmd --zone=public --add-port=3306/tcp --permanent   ---添加开放端口
* firewall-cmd --reload;                重启防火墙
* firewall-cmd --list-ports            --查询当前开放的端口列表
#### 三、具体配置
* 1、查看机器的ip地址
* 2、保证master和slave之间的网络互通，并保证3306端口是开放的
##### 3.1 master配置（主库）
###### 1、开启二进制日志
> 配置`log_bin`和全局唯一的`server-id`.
* server-id=1
* expire_logs_days=7
* log_bin=mysql-bin
> 配置完成后需要重启mysql服务
###### 2、创建一个用于主从通信用的账号
* 1、mysql>create user 'xxx'@'从库ip' IDENTIFIED BY '密码';
> 说明从库只能通过`从库ip`和`xxx`用户可密码访问主库。
* 2、mysql>GRANT REPLICATION SLAVE ON *.* TO 'xxx'@'从库ip' IDENTIFIED BY '密码';
* 3、mysql>FLUSH PRIVILEGES;
###### 说明：
* '从库ip' 可以使用`%`代替，表示可以任意ip访问
* `*.*`代表任意库任意表，[库名].*：表示只能访问某一库下的所有表
* REPLICATION SLAVE ON代表开启相关权限
* 'xxx'@'从库ip' IDENTIFIED BY '密码'，代表从库只能通过创建的`从库ip`通过`xxx`账号和密码访问主库
###### 3、获取binlog的日志文件名和position
> mysql>show master status;  --查看当前binlog日志的文件名和当前位置
##### 3.2 slave配置（从库）
###### 1、配置全局唯一的server-id
设计修改配置文件需要重启mysqld服务
###### 2、使用master创建的账户读取binlog同步数据
在mysqlshell上执行下面的命令
* mysql>CHANGE MASTER TO MASTER_HOST='主库IP',
* MASTER_PORT=3306,
* MASTER_USER='主服务器数据库登录名',       #在主库上创建的`用户名&密码`
* MASTER_PASSWORD='主服务器数据库登录密码',
* MASTER_LOG_FILE='mysql-bin.00001',
* MASTER_LOG_POS=1106,            #开始复制的位置position
###### 3、START SLAVE开启从库服务（stop slave,start slave）
* 通过`show slave status`命令查看主从复制的状态
* 通过`show processlist`命令查看master和slave相关现成的运行状态

--stop slave，set global sql_slave_skip_counter=1;#跳过错误start slave；重启进程

## 多查看mysql文档查询相关错误