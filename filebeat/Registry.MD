每个字段的意义解释：  
source： 记录采集日志的完整路径  
offset： 采集这个日志文件到了哪个位置，总采集字节数  
inode： 日志文件的inode号，关于inode的详细解释看下文  
device： 日志所在的磁盘编号，下文stat命令中Device的值  
timestamp： 日志最后一次发生变化的时间戳  
ttl： 采集失效时间，-1表示永不失效  

Filebeat在每次启动时都会来读取这个文件，如果文件不存在则会创建新文件  
