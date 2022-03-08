jstat是hotspot自带的工具，和java一样也位于JAVA_HOME/bin下面，我们通过该工具可以实时了解当前进程的gc，compiler，class，memory等相关的情况  

jstat -options  
-class-compiler  
-gc  
-gccapacity  
-gccause  
-gcmetacapacity  
-gcnew  
-gcnewcapacity  
-gcold  
-gcoldcapacity  
-gcutil  
-printcompilation  

jstat -gcutil 692 1000  
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT  
  0.00  41.49  59.79  83.66  89.92  78.74    295    5.436    10    3.855    9.291  
  0.00  41.49  59.80  83.66  89.92  78.74    295    5.436    10    3.855    9.291  
  0.00  41.49  59.80  83.66  89.92  78.74    295    5.436    10    3.855    9.291  
  0.00  41.49  59.80  83.66  89.92  78.74    295    5.436    10    3.855    9.291  
  0.00  41.49  59.80  83.66  89.92  78.74    295    5.436    10    3.855    9.291  

变量值显然是从目标进程里获取来的，但是是怎样来的？local socket还是memory share？  
其实是从一个共享文件里来的，这个文件叫PerfData，主要指的是/tmp/hsperfdata_<user>/<pid>这个文件

# 文件创建
这个文件是否存在取决于两个参数，一个UsePerfData，另一个是PerfDisableSharedMem，如果设置了-XX:+PerfDisableSharedMem或者-XX:-UsePerfData，那这个文件是不会存在的，  
默认情况下PerfDisableSharedMem是关闭的，UsePerfData是打开的，所以默认情况下PerfData文件是存在的。对于UsePerfData和PerfDisableSharedMem这两个参数，这里着重讲一下：

    UsePerfData：如果关闭了UsePerfData这个参数，那么jvm启动过程中perf memory都不会被创建，jvm运行过程中自然不会再将这些性能数据保存起来，默认情况是是打开的  
    PerfDisableSharedMem：该参数决定了存储PerfData的内存是不是可以被共享，也就是说不管这个参数设置没设置，jvm在启动的时候都会分配一块内存来存PerfData，只是说这个PerfData是不是其他进程可见的问题，  
    如果设置了这个参数，说明不能被共享，此时其他进程将访问不了该内存，这样一来，譬如我们jps，jstat等都无法工作。默认这个参数是关闭的，也就是默认支持共享的方式  

# 文件更新

由于这个文件是通过mmap的方式映射到了内存里，而jstat是直接通过DirectByteBuffer的方式从PerfData里读取的，所以只要内存里的值变了，那我们从jstat看到的值就会发生变化，内存里的值什么时候变，  
取决于-XX:PerfDataSamplingInterval这个参数，默认是50ms，也就是说50ms更新一次值，基本上可以认为是实时的了。  
PerfData其他相关VM参数  
    -XX:PerfDataMemorySize：指定/tmp/hsperfdata_<user>  
    下perfData文件的大小，默认是32KB，如果用户设置了该值，jvm里会自动和os的page size对齐，比如linux下pagesize默认是4KB，那如果你设置了31KB，那自动会分配32KB  
    -XX:+PerfDataSaveToFile：是否在进程退出的时候将PerfData里的数据保存到一个特定的文件里，文件路径由下面的参数指定，否则就在当前目录下  
    -XX:PerfDataSaveFile：指定保存PerfData文件的路径  
