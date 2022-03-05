# 停止filebeat

# 删除filebeat日志记录日志位置文件mv /var/lib/filebeat/registry /tmp/

# 重新启动filebeat

# 全部重新收取
即：删除registry文件并重启filebeat  
PS: 这种方法会将所有文件重新收取  

# 重新收取某个文件
即：修改registry文件中对应文件的offset信息并重启filebeat  
offset修改为0则代表从头开始  
PS: 关于registry文件的格式参见”registry文件内容解析“  

# 从指定位置收取某个文件
指定offset值, 从指定的offset位置收取  
