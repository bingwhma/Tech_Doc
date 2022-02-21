
## Sentinel（哨兵）模式  
哨兵模式基于主从复制模式，只是引入了哨兵来监控与自动处理故障。  

哨兵顾名思义，就是来为Redis集群站哨的，一旦发现问题能做出相应的应对处理。其功能包括  
  
    监控master、slave是否正常运行  
    当master出现故障时，能自动将一个slave转换为master（大哥挂了，选一个小弟上位）  
    多个哨兵可以监控同一个Redis，哨兵之间也会自动监控  
  
哨兵模式的具体工作机制：  
  
在配置文件中通过 sentinel monitor <master-name> <ip> <redis-port> <quorum> 来定位master的IP、端口，一个哨兵可以监控多个master数据库，只需要提供多个该配置项即可。哨兵启动后，会与要监控的master建立两条连接：  
  
    一条连接用来订阅master的_sentinel_:hello频道与获取其他监控该master的哨兵节点信息  
    另一条连接定期向master发送INFO等命令获取master本身的信息  
  
与master建立连接后，哨兵会执行三个操作：  
  
    定期（一般10s一次，当master被标记为主观下线时，改为1s一次）向master和slave发送INFO命令  
    定期向master和slave的_sentinel_:hello频道发送自己的信息  
    定期（1s一次）向master、slave和其他哨兵发送PING命令  
  
发送INFO命令可以获取当前数据库的相关信息从而实现新节点的自动发现。所以说哨兵只需要配置master数据库信息就可以自动发现其slave信息。获取到slave信息后，哨兵也会与slave建立两条连接执行监控。通过INFO命令，哨兵可以获取主从数据库的最新信息，并进行相应的操作，比如角色变更等。  
  
接下来哨兵向主从数据库的_sentinel_:hello频道发送信息与同样监控这些数据库的哨兵共享自己的信息，发送内容为哨兵的ip端口、运行id、配置版本、master名字、master的ip端口还有master的配置版本。这些信息有以下用处：  
  
    其他哨兵可以通过该信息判断发送者是否是新发现的哨兵，如果是的话会创建一个到该哨兵的连接用于发送PING命令。  
    其他哨兵通过该信息可以判断master的版本，如果该版本高于直接记录的版本，将会更新  
    当实现了自动发现slave和其他哨兵节点后，哨兵就可以通过定期发送PING命令定时监控这些数据库和节点有没有停止服务。  
  
如果被PING的数据库或者节点超时（通过 sentinel down-after-milliseconds master-name milliseconds 配置）未回复，哨兵认为其主观下线（sdown，s就是Subjectively —— 主观地）。如果下线的是master，哨兵会向其它哨兵发送命令询问它们是否也认为该master主观下线，如果达到一定数目（即配置文件中的quorum）投票，哨兵会认为该master已经客观下线（odown，o就是Objectively —— 客观地），并选举领头的哨兵节点对主从系统发起故障恢复。若没有足够的sentinel进程同意master下线，master的客观下线状态会被移除，若master重新向sentinel进程发送的PING命令返回有效回复，master的主观下线状态就会被移除  
  
哨兵认为master客观下线后，故障恢复的操作需要由选举的领头哨兵来执行，选举采用Raft算法：  
  
    发现master下线的哨兵节点（我们称他为A）向每个哨兵发送命令，要求对方选自己为领头哨兵  
    如果目标哨兵节点没有选过其他人，则会同意选举A为领头哨兵  
    如果有超过一半的哨兵同意选举A为领头，则A当选  
    如果有多个哨兵节点同时参选领头，此时有可能存在一轮投票无竞选者胜出，此时每个参选的节点等待一个随机时间后再次发起参选请求，进行下一轮投票竞选，直至选举出领头哨兵  
  
选出领头哨兵后，领头者开始对系统进行故障恢复，从出现故障的master的从数据库中挑选一个来当选新的master,选择规则如下：  
  
    所有在线的slave中选择优先级最高的，优先级可以通过slave-priority配置  
    如果有多个最高优先级的slave，则选取复制偏移量最大（即复制越完整）的当选  
    如果以上条件都一样，选取id最小的slave  
  
挑选出需要继任的slave后，领头哨兵向该数据库发送命令使其升格为master，然后再向其他slave发送命令接受新的master，最后更新数据。将已经停止的旧的master更新为新的master的从数据库，使其恢复服务后以slave的身份继续运行。  
    
## 部署演示    
哨兵模式基于前文的主从复制模式。哨兵的配置文件为sentinel.conf，在文件中添加  
  
sentinel monitor mymaster 127.0.0.1 6379 1 # mymaster定义一个master数据库的名称，后面是master的ip， port，1表示至少需要一个Sentinel进程同意才能将master判断为失效，如果不满足这个条件，则自动故障转移（failover）不会执行  
sentinel auth-pass mymaster 123456 # master的密码  
  
sentinel down-after-milliseconds mymaster 5000 # 5s未回复PING，则认为master主观下线，默认为30s  
sentinel parallel-syncs mymaster 2  # 指定在执行故障转移时，最多可以有多少个slave实例在同步新的master实例，在slave实例较多的情况下这个数字越小，同步的时间越长，完成故障转移所需的时间就越长  
sentinel failover-timeout mymaster 300000 # 如果在该时间（ms）内未能完成故障转移操作，则认为故障转移失败，生产环境需要根据数据量设置该值  
  
    一个哨兵可以监控多个master数据库，只需按上述配置添加多套  
  
分别以26379,36379,46379端口启动三个sentinel  
  
[root@dev-server-1 sentinel]# redis-server sentinel1.conf --sentinel  
[root@dev-server-1 sentinel]# redis-server sentinel2.conf --sentinel  
[root@dev-server-1 sentinel]# redis-server sentinel3.conf --sentinel  
  
也可以使用redis-sentinel sentinel1.conf 命令启动。此时集群包含一个master、两个slave、三个sentinel，如图，  
  
redis-cluster-instance  
  
我们来模拟master挂掉的场景，执行 kill -9 3017 将master进程干掉，进入slave中执行 info replication查看，  
 
 ```
[root@dev-server-1 sentinel]# redis-cli -p 7001  
127.0.0.1:7001> auth 123456  
OK  
127.0.0.1:7001> info replication  
# Replication  
role:slave  
master_host:127.0.0.1  
master_port:7002  
master_link_status:up  
master_last_io_seconds_ago:1  
master_sync_in_progress:0  
#省略  
127.0.0.1:7001> exit  
[root@dev-server-1 sentinel]# redis-cli -p 7002  
127.0.0.1:7002> auth 123456  
OK  
127.0.0.1:7002> info replication  
# Replication  
role:master  
connected_slaves:1  
slave0:ip=127.0.0.1,port=7001,state=online,offset=13642721,lag=1  
#省略  
```

可以看到slave 7002已经成功上位晋升为master（role：master），接收一个slave 7001的连接。此时查看slave2.conf配置文件，发现replicaof的配置已经被移除了，slave1.conf的配置文件里replicaof 127.0.0.1 6379 被改为 replicaof 127.0.0.1 7002。重新启动master，也可以看到master.conf配置文件中添加了replicaof 127.0.0.1 7002的配置项，可见大哥（master）下位后，再出来混就只能当当小弟（slave）了，三十年河东三十年河西。  

## 哨兵模式的优缺点  
  
优点：  
  
    哨兵模式基于主从复制模式，所以主从复制模式有的优点，哨兵模式也有  
    哨兵模式下，master挂掉可以自动进行切换，系统可用性更高  
  
缺点：  
  
    同样也继承了主从模式难以在线扩容的缺点，Redis的容量受限于单机配置  
    需要额外的资源来启动sentinel进程，实现相对复杂一点，同时slave节点作为备份节点不提供服务  
