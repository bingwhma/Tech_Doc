# Redis基础命令
特点  
 redis采用单线程多路复用模式  
 所有操作都具有原子性  
 定期删除+惰性删除+内存淘汰机制  
 AOF+RDB持久化  
 
 ## 基础命令
|  命令  |  作用  |  举例  
| :-----| ----: | :----: |
| select index	|  选择redis数据库 | select 1
| dbsize		|  返回当前数据库的 key 的数量		|  dbsize
| del key1 key2		|  删除指定key，返回删除数量		|  del key
| flushdb [ASYNC]		|  清空当前数据库中的所有 key		|  flushdb
| flushall[ASYNC]		|  清空整个 Redis 服务器的数据		|  flushall
| keys pattern		|  查看当前数据库中的所有	|  key	keys *
| scan cursor match count type		|  增量式迭代命令		|  scan 0 match * count 20
| exists key1 key2		|  检查给定 key 是否存在。不存在:0		|  exists a b c
| expire key seconds		|  为给定 key 设置生存时间		|  expire a 5
| ttl key		|  返回给定 key 的剩余生存时间		|  ttl a
| persist key		|  移除给定 key 的生存时间		|  persist a
| type key		|  返回 key 所储存的值的类型		|  type a
| move key db		|  将当前数据库的 key 移动到给定的数据库 db 当中。	|  	move a 1

## String相关命令
值得大小不超过512m  
推荐key命名规则object：id：field，方便在redis桌面管理中查看  

|  命令  |  作用  |  举例
| :-----| ----: | :----: |
| set key value [NX PX][NX XX]			|  将字符串值 value 关联到 key 。			|  set a 1
| mset k1 v1 k2 v2			|  同时为多个键设置值。(原子)			|  mset a 1 b 2
| msetnx k1 v1 k2 v2			|  当且仅当所有给定键都不存在时， 为所有给定键设置值。			|  msetnx a 1 b 2
| setRange offset key value			|  从offset开始，用value参数覆写key储存的字符串值。			|  setrange a 1 abc
| get			|  返回与键 key 相关联的字符串值			|  get a
| getset key value			|  将键 key 的值设为新value,返回旧value，不存在返回nil			|  getset a 2
| strlen key			|  返回键 key 储存的字符串值的长度			|  strlen a
| getRange key start end			|  返回键 key 储存的字符串值的指定部分			|  getrange a 0 -1
| incr key			|  对应键值自增加+1			|  incr a
| incrby key increment			|  对应键值自增加+increment			|  incr a 3
| decr key			|  对应键值自减1			|  decr a
| decrby key decrement			|  对应键值自减			|  decrby a 1
| append key value			|  在key末尾添加value			|  append a abc
| rename key newkey			|  将 key 改名为 newkey			|  rename a b

## List相关命令  
底层实现为LinkedList链表  
list 最多可以有 2^32 - 1 个元素  
  
| 命令  |  作用  |  举例  
| :-----| ----: | :----: |  
| lpush key value…  |  将一个或多个值 value 插入到列表 key 的表头  |  lpush alist a b c  
| lpushx key value  |  当列表存在，将值 value 插入到列表 key 的表头  |  lpushx alist d  
| rpushx key value  |  当列表存在，将值 value 插入到列表 key 的表尾  |  rpushx alist d  
| lpop key  |  移除并返回列表 key 的头元素  |  lpop alist  
| rpop key  |  移除并返回列表 key 的尾元素。  |  rpop alist  
| rpoplpush source destination  |  将source尾部元素添加到destination头部，返回source尾部  |  rpoplpush alist blist  
| lrem key count value  |  删除列表中与value值相同的元素，count表示删除个数和遍历位置  |  lpush alist a b c  
| linsert key [before|after] pivot value  |  在列表中pivot元素前后插入value值  |  linsert alist before b a  
| lset key index value  |  将列表index位置上的元素改为value值  |  lset alist 0 b  
| ltrim key start stop  |  将列表修剪为start-stop范围  |  ltrim alist 0 10  
| blpop key… timeout  |  阻塞式弹出第一个不为空列表的表头  |  blpop alist blist 10  
| brpop key… timeout  |  阻塞式弹出第一个不为空列表的表尾  |  brpop alist blist 10  
| brpoplpush source destination timeout  |  阻塞式弹出表头插入表尾  |  brpoplpush alist blist 10  
  
## SET相关命令  
  
  一个无序的字符串集合，其中元素不重复  
  set中最多可以有 2^32 - 1 个元素  
  
| 命令  |  作用  |  举例  
| :-----| ----: | :----: |  
| sadd key member…  |  将一个或多个 member 元素加入到集合 key 当中  |  sadd aset a b c  
| smove source destination member  |  将member从source集合移动到destination集合  |  smove aset bset d  
| spop key count  |  移除并返回集合中的count个随机元素  |  spop aset 2  
| srem key member…  |  移除集合 key 中的一个或多个 member 元素  |  srem aset d j  
| srandmember key count  |  随机返回count个元素  |  srandmember aset 2  
| smembers key  |  返回集合 key 中的所有成员  |  smembers aset  
| scard key  |  返回集合中元素的数量  |  scard aset  
| sismember key member  |  判断 member 元素是否集合 key 的成员  |  sismember aset a  
| sdiff key…  |  返回所有给定集合之间的差集  |  sdiff aset bset  
| sdiffstore destination key…  |  返回所有给定集合之间的差集并存储在destination中  |  sdiffstore cset aset bset  
| sinter key…  |  返回所有给定集合的交集  |  sinter aset bset  
| sinterstore destination key…  |  返回所有给定集合的交集并存储在destination中  |  sinterstore cset aset bset  
| sunion key…  |  返回所有给定集合的并集  |  sunion aset bset  
| sunionstore destination key…  |  返回所有给定集合的并集并存储在destination中  |  sunionstore cset aset bset  
  
  
## HASH相关命令  
 hash中最多可以有 2^32 - 1 个key/value 键值对  
  
| 命令  |  作用  |  举例  
| :-----| ----: | :----: |  
| hset key field value  |  将哈希表 hash 中域 field 的值设置为 value  |  hset ahash name king  
| hsetnx key field value  |  当且仅当域 field 尚未存在于哈希表的情况下， 将它的值设置为 value  |  hsetnx ahash name king  
| hmset key (field value)…  |  同时将多个 field-value (域-值)对设置到哈希表 key 中  |  hmset ahash name king age 10  
| hget key field  |  返回哈希表中给定域的值  |  hget ahash name  
| hmget key field…  |  返回哈希表 key 中，一个或多个给定域的值  |  hmget ahash name age  
| hgetall key  |  返回哈希表 key 中，所有的域和值  |  hgetall ahash  
| hdel key field…  |  删除指定key中的域值  |  hdel ahash name age  
| hincrby key field increment  |  为哈希表 key 中的域 field 的值加上增量 increment  |  hincrby ahash age 2  
| hincrbyfloat key field increment  |  为哈希表 key 中的域 field 加上浮点数增量 increment  |  hincrbyfloat ahash height 2.3  
| hkeys key  |  返回哈希表 key 中的所有域  |  hkeys ahash  
| hvals key  |  返回哈希表 key 中所有域的值。  |  hvals ahash  
| hexists key field  |  判断指定key的域是否存在  |  hexists ahash name  
| hlen key  |  返回哈希表 key 中域的数量  |  hlenahash age  
| hstrlen key field  |  返回哈希表 key 中， 与给定域 field 相关联的值的字符串长度  |  hstrlen ahash height  
  
## Sorted Set相关命令  
  
| 命令  |  作用  |  举例  
| :-----| ----: | :----: |  
| zadd [NX|XX] (score member)…  |  将一个或多个 member 元素及其 score 值加入到有序集 key 当中。  |  zadd azset 1 name 2 king  
| zrem key member…  |  移除有序集 key 中的一个或多个成员，不存在的成员将被忽略。  |  zrem azset name king  
| zrange key start stop withscores  |  返回有序集 key 中，指定区间内的成员。  |  zrange azset 0 -1  
| zcard key  |  返回有序集 key 的基数  |  zcard azset  
| zcount key min max  |  返回有序集 key 中，score 值在 min 和 max 之间的成员的数量。  |  zcount azset 0 2  
| zscore key member  |  返回有序集 key 中，成员 member 的 score 值  |  zscore azset name  
| zrank key member  |  返回有序集 key 中成员 member 的排名  |  zrank azset name  
| zrevrank key member  |  集合按照score降序排列，返回有序集 key 中成员 member 的排名  |  zrevrank azset name  
| zincrby key increment member  |  为有序集 key 的成员 member 的 score 值加上增量 increment  |  zincrby azset 2 name  
| zinterstore destination numkeys key…  |  计算给定的一个或多个有序集的交集，存储到destination  |  zinterstore czset 2 azset bzset  
| zunionstore destination numkeys key…  |  计算给定的一个或多个有序集的并集，存储到destination  |  zunionstore czset 2 azset bzset  
  
## HyperLogLog相关命令  
  
用于统计基数，牺牲准确性来节省空间  
  
| 命令  |  作用  |  举例  
| :-----| ----: | :----: |  
pfadd key element…  |  将任意数量的元素添加到指定的 HyperLogLog 里面。  |  pfadd alog a b  
pfcount key…  |  返回所有给定 HyperLogLog 的并集的近似基数  |  pfcount alog blog  
pfmerge destkey sourcekey…  |  将多个 HyperLogLog 合并（merge）为一个 HyperLogLog  |  pfmerge clog blog ablog  
  
## BitMap相关命令  
  
定义在字符串类型上的面向位的操作的集合  
  
| 命令  |  作用  |  举例  
| :-----| ----: | :----: |  
setbit key offset value  |  对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)  |  setbit abit 0 1  
getbit key offset  |  对 key 所储存的字符串值，获取指定偏移量上的位(bit)  |  getbit abit 0  
bitcount key start end  |  计算给定字符串中，被设置为 1 的比特位的数量。  |  bitcount abit 0 1  
bitpos key bit start end  |  返回位图中第一个值为 bit 的二进制位的位置。  |  bitpos abit 0 0 2  
bitop operation destkey key…  |  对一个或多个 key 进行逻辑运算，并将结果保存到 destkey  |  bitop and cbit abit bbit  
bitfield key [get type offset][incrby type offset increment]  |  接受一系列待执行的操作作为参数， 并返回一个数组作为回复  |  bitfield abit get i8 100  
  
## 事务相关命令  
| 命令  |  作用  |  举例  
| :-----| ----: | :----: |  
| multi  |  标记一个事务块的开始。  |  multi  
| exec  |  执行所有事务块内的命令  |  exec  
| discard  |  取消事务，放弃执行事务块内的所有命令。  |  discard  
| watch key…  |  监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。  |  watch a b  
| unwatch  |  取消 WATCH 命令对所有 key 的监视。  |  unwatch  
  
## 持久化相关命令  
  
| 命令  |  作用  |  举例  
| :-----| ----: | :----: |  
save  |  将当前 Redis 实例的所有数据快照(snapshot)以 RDB 文件的形式保存到硬盘。  |  save  
bgsave  |  在后台异步(Asynchronously)保存当前数据库的数据到磁盘。  |  exec  
bgrewriteaof  |  执行一个 AOF文件 重写操作。  |  bgrewriteaof  
lastsave  |  返回最近一次 Redis 成功将数据保存到磁盘上的时间  |  lastsave  

