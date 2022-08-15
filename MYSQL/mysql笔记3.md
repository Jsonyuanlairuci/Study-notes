mysql笔记3
===============================
## mysql优化
### sql和索引优化：
> 慢查询日志=》设置慢查询时间=》记录慢查询sql=》使用`explain`分析sql=>优化措施
### 一、应用上优化:
* 1:引入连接池
* 2：引入缓存（redis）
### mysql server上的优化
* 1：修改参数
* 2：关闭自适应hash索引
* 3：修改`innodb_log_buffer`和`innodb_buffer_pool_size`等参数

### 二、查询缓存
#### show variables like "%query_cache%";查询缓存的设置
> 查询多更改少，可以开启查询缓存
* 使用`show status`查看缓存的使用情况
### 索引和数据缓存
* innodb_log_buffer
* innodb_buffer_pool_size
### 三、线程缓存
* thread_cache_size=10;线程缓存数量
### 并发连接数量和超时时间
#### show variables like "%connect%"; 查看连接情况
#### show global variables like "%timeout%"; 查看超时情况
* 设置超时时间，超过设置的时间没有请求就会主动断开连接，单位是秒，在配置文件中添加配置
> wait_timeout=600;