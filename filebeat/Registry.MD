每个字段的意义解释：  
source： 记录采集日志的完整路径  
offset： 采集这个日志文件到了哪个位置，总采集字节数  
inode： 日志文件的inode号，关于inode的详细解释看下文  
device： 日志所在的磁盘编号，下文stat命令中Device的值  
timestamp： 日志最后一次发生变化的时间戳  
ttl： 采集失效时间，-1表示永不失效  

Filebeat在每次启动时都会来读取这个文件，如果文件不存在则会创建新文件  

source 日志文件完整路径  
offset　已经采集的日志的字节数;已经采集到日志的哪个字节位置  
filestateos　　操作系统相关  
　　inode　　日志文件的inode号  
　　device    日志所在磁盘的磁盘编号  
timestamp　　日志最后一次发生变化的时间戳  
ttl　　采集失效时间。-1表示只要日志存在，就一直采集该日志  

# registry太大的问题

## filebeat 5.6.4文档看到，有两个参数clean_removed和clean_inactive可以清除掉registry文件中无用的state信息。
clean_removed: 默认打开，清除已经被删除文件的state信息  
clean_inactive：默认关闭，清除掉已经不活跃的文件的state信息，必须配合ignore_older参数使用，并且ignore_older +scan_frequency必须小于clean_inactive，否则可能造成日志重复收集。  

既然clean_removed参数是默认打开的，并且可以清除掉已经被删除文件的state信息，那为什么在上述场景下并没有生效呢？原因就是日志滚动时是先rename重命名了access.log文件为acess.log.1，   
然后把acess.log.1文件压缩成了acess.log.1.gz之后再删除acess.log.1文件，也就是说并没有直接删除掉access.log文件，而clean_removed参数对重命名的文件是不起作用的，所以state信息没有被清除。  

5.6版本中有关clean_removed参数的描述：    

    This setting does not apply to renamed files or files that were moved to another directory that is still visible to Filebeat
  
 已经在6.0之后的版本得到了修复，即使文件被重命名，在收集完该文件后后续state信息也会被清理。  
6.0版本中有关clean_removed参数的描述：  

    When this option is enabled, Filebeat cleans files from the registry if they cannot be found on disk anymore under the last known name. This means also files
which were renamed after the harvester was finished will be removed.


# 解决办法1
通过使用6.4.3版本的filebeat重新采集access.log日志，发现registry文件越来越大的问题已经得到了解决，所以最好的解决办法是把filebeat升级到6.4.3版本。

# 解决办法2
如果不能立即升级filebeat怎么办？那就可以通过配置ignore_older+clean_inactive参数解决问题.

解释一下ignore_older参数：  

    ignore_older: 默认为0不启用，启用该参数可以忽略掉比较旧的日志文件(根据modify time判断)，从较新的文件开始收集日志。如果日志文件第一次被采集到但是modify time超出了ignore_older，则会从该文件末尾开始采集日志，registry文件中的state信息为日志文件的末尾；如果日志文件的state信息已经存在，但是文件的modify time超出了ignore_older，则继续从state中记录的offset开始读取日志。ignore_older必须大于close_inactive, 以确保文件在被忽略之前已经被关闭。如果文件modify time已经超出了ignore_older, 但是仍然在被采集中，此时filebeat会把文件读取完毕后等待close_inactive时间后关闭该文件。
