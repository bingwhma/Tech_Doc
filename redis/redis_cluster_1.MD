# Redis支持三种集群方案

## 主从复制模式
主从复制模式中包含一个主数据库实例（master）与一个或多个从数据库实例（slave）  
客户端可对主数据库进行读写操作，对从数据库进行读操作，主数据库写入的数据会实时自动同步给从数据库。    
具体工作机制为：  
    slave启动后，向master发送SYNC命令，master接收到SYNC命令后通过bgsave保存快照（即上文所介绍的RDB持久化），并使用缓冲区记录保存快照这段时间内执行的写命令   
    master将保存的快照文件发送给slave，并继续记录执行的写命令  
    slave接收到快照文件后，加载快照文件，载入数据  
    master快照发送完后开始向slave发送缓冲区的写命令，slave接收命令并执行，完成复制初始化  
    此后master每次执行一个写命令都会同步发送给slave，保持master与slave之间数据的一致性  

SAMPLE:  
```
###网络相关###
# bind 127.0.0.1 # 绑定监听的网卡IP，注释掉或配置成0.0.0.0可使任意IP均可访问
protected-mode no # 关闭保护模式，使用密码访问
port 6379  # 设置监听端口，建议生产环境均使用自定义端口
timeout 30 # 客户端连接空闲多久后断开连接，单位秒，0表示禁用

###通用配置###
daemonize yes # 在后台运行
pidfile /var/run/redis_6379.pid  # pid进程文件名
logfile /usr/local/redis/logs/redis.log # 日志文件的位置

###RDB持久化配置###
save 900 1 # 900s内至少一次写操作则执行bgsave进行RDB持久化
save 300 10
save 60 10000 
# 如果禁用RDB持久化，可在这里添加 save ""
rdbcompression yes #是否对RDB文件进行压缩，建议设置为no，以（磁盘）空间换（CPU）时间
dbfilename dump.rdb # RDB文件名称
dir /usr/local/redis/datas # RDB文件保存路径，AOF文件也保存在这里

###AOF配置###
appendonly yes # 默认值是no，表示不使用AOF增量持久化的方式，使用RDB全量持久化的方式
appendfsync everysec # 可选值 always， everysec，no，建议设置为everysec

###设置密码###
requirepass 123456 # 设置复杂一点的密码
```
部署主从复制模式只需稍微调整slave的配置，在redis.conf中添加  
```
replicaof 127.0.0.1 6379 # master的ip，port
masterauth 123456 # master的密码
replica-serve-stale-data no # 如果slave无法与master同步，设置成slave不可读，方便监控脚本发现问题
```
### 主从复制的优缺点
优点：  
    master能自动将数据同步到slave，可以进行读写分离，分担master的读压力  
    master、slave之间的同步是以非阻塞的方式进行的，同步期间，客户端仍然可以提交查询或更新请求  
缺点：
    不具备自动容错与恢复功能，master或slave的宕机都可能导致客户端请求失败，需要等待机器重启或手动切换客户端IP才能恢复  
    master宕机，如果宕机前数据没有同步完，则切换IP后会存在数据不一致的问题  
    难以支持在线扩容，Redis的容量受限于单机配置  

