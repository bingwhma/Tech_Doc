# CPU资源监控   sar -u 
CPU：all 表示统计信息为所有 CPU 的平均值。  
%user：显示在用户级别(application)运行使用 CPU 总时间的百分比  
%nice：显示在用户级别，用于nice操作，所占用 CPU 总时间的百分比  
%system：在核心级别(kernel)运行所使用 CPU 总时间的百分比  
%iowait：显示用于等待I/O操作占用 CPU 总时间的百分比  
%steal：管理程序(hypervisor)为另一个虚拟进程提供服务而等待虚拟 CPU 的百分比  
%idle：显示 CPU 空闲时间占用 CPU 总时间的百分比  
备注：  
1.若 %iowait 的值过高，表示硬盘存在I/O瓶颈  
2.若 %idle 的值高但系统响应慢时，有可能是 CPU 等待分配内存，此时应加大内存容量  
3.若 %idle 的值持续低于1，则系统的 CPU 处理能力相对较低，表明系统中最需要解决的资源是 CPU
# 内存和交换空间监控   sar -r 
kbmemfree：这个值和free命令中的free值基本一致,所以它不包括buffer和cache的空间.  
kbmemused：这个值和free命令中的used值基本一致,所以它包括buffer和cache的空间.  
%memused：这个值是kbmemused和内存总量(不包括swap)的一个百分比.  
kbbuffers和kbcached：这两个值就是free命令中的buffer和cache.  
kbcommit：保证当前系统所需要的内存,即为了确保不溢出而需要的内存(RAM+swap).  
%commit：这个值是kbcommit与内存总量(包括swap)的一个百分比.  
Kbactive：活动内存量（以千字节计算）（最近使用的内存，通常不会被收回，除非绝对必要）  
kbinact：不活动内存量（以千字节计算的内存（最近使用的内存），更有资格被用于其他目的）。  
kbdirty：以KB为单位的内存量等待写入磁盘  
# 内存分页监控 sar -B 
pgpgin/s：表示每秒从磁盘或SWAP置换到内存的字节数(KB)  
pgpgout/s：表示每秒从内存置换到磁盘或SWAP的字节数(KB)  
fault/s：每秒钟系统产生的缺页数,即主缺页与次缺页之和(major + minor)  
majflt/s：每秒钟产生的主缺页数.  
pgfree/s：每秒被放入空闲队列中的页个数  
pgscank/s：每秒被kswapd扫描的页个数  
pgscand/s：每秒直接被扫描的页个数  
pgsteal/s：每秒钟从cache中被清除来满足内存需要的页个数  
%vmeff：每秒清除的页(pgsteal)占总扫描页(pgscank+pgscand)的百分比  
# I/O和传送速率监控 sar -b
tps：每秒钟物理设备的 I/O 传输总量  
rtps：每秒钟从物理设备读入的数据总量  
wtps：每秒钟向物理设备写入的数据总量  
bread/s：每秒钟从物理设备读入的数据量，单位为 块/s  
bwrtn/s：每秒钟向物理设备写入的数据量，单位为 块/s  
# 进程队列长度和平均负载状态监控 sar -q
runq-sz：运行队列的长度（等待运行的进程数）  
plist-sz：进程列表中进程（processes）和线程（threads）的数量  
ldavg-1：最后1分钟的系统平均负载（System load average）  
ldavg-5：过去5分钟的系统平均负载  
ldavg-15：过去15分钟的系统平均负载  
blocked：当前阻塞的任务数，等待I/O完成  
# 系统交换活动信息监控 sar -W 
pswpin/s：每秒系统换入的交换页面（swap page）数量  
pswpout/s：每秒系统换出的交换页面（swap page）数量  
# 设备使用情况监控（磁盘的读写性能） sar -d
参数-p可以打印出sda,hdc等磁盘设备名称,如果不用参数-p,设备节点则有可能是dev8-0,dev22-0
tps：每秒从物理磁盘 I/O 的次数。注意，多个逻辑请求会被合并为一个 I/O 磁盘请求，一次传输的大小是不确定的  
rd_sec/s：每秒读扇区的次数  
wr_sec/s：每秒写扇区的次数  
avgrq-sz：平均每次设备 I/O 操作的数据大小（扇区）  
avgqu-sz：磁盘请求队列的平均长度  
await：从请求磁盘操作到系统完成处理，每次请求的平均消耗时间，包括请求队列等待时间，单位是毫秒（1 秒=1000 毫秒）  
svctm：系统处理每次请求的平均时间，不包括在请求队列中消耗的时间  
%util：I/O 请求占 CPU 的百分比，比率越大，说明越饱和  
# inode、文件和其他内核表监控 sar -v
dentunusd：目录高速缓存中未被使用的条目数量  
file-nr：文件句柄（file handle）的使用数量  
inode-nr：索引节点句柄（inode handle）的使用数量  
# 网络接口的统计信息 sar -n DEV
IFACE：报告统计信息的网络接口的名称  
rxpck/s：每秒接收的数据包总数  
txpck/s：每秒传输的报文总数  
rxkB/s：每秒接收的千字节总数  
txkB/s：每秒传输的总千字节数  
rxcmp/s：每秒接收的压缩数据包数  
txcmp/s：每秒传输的压缩包数  
rxmcst/s：每秒接收的组播数据包数  
pty-nr：使用的pty数量  
