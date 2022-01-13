1、查看运行的虚拟机
#  virsh list
查看所有的虚拟机（关闭和运行的，不包括摧毁的）
#  virsh list --all

2.、启动虚拟机
# virsh start  虚拟机名称

虚拟机随物理机启动而启动
# virsh autostart 虚拟机名称

取消虚拟机随物理机启动而启动
# virsh autostart --disable 虚拟机名称

3、连接虚拟机
#virsh console 虚拟机名称

4、退出虚拟机
快捷键： ctrl+]


5、关闭虚拟机
virsh shutdown  虚拟机名称
#前提虚拟机需要（安装acpid服务）
yum install -y acpid
/etc/init.d/acpid start
 
6、在virsh list中删除该虚拟机
virsh destroy 虚拟机名称

7、彻底删除虚拟机

#解除标记
virsh undefine 虚拟机名称
然后删除虚拟机存储所在的位置

8、挂起虚拟机
virsh suspend 虚拟机名称 #挂起
virsh resume 虚拟机名称  #恢复被挂起的

virsh console 无法连接虚机#
进入虚拟机操作

grubby --update-kernel=ALL --args="console=ttyS0"
reboot

未试过

1、在KVM（客户机）下开机启动并启动服务
systemctl enable serial-getty@ttyS0.service
systemctl start serial-getty@ttyS0.service

2、在宿主机下访问
virsh console KVM名称

其他#
1,建立磁盘
#qemu-img create /opt/kvm/centos7.img 50G
#qemu-img create -f qcow2 -o preallocation=metadata /opt/kvm/centos7.qcow2 50G （支持快照）

以上两条命令都表示在/opt/kvm下创建50G系统盘。
第一是以默认格式img创建一个50G系统盘，测试使用时推荐使用。
第二条以qcow2格式创建一个50G系统盘，推荐在正式环境中使用，支持快照等高级功能，该功能将在拓展篇中记录。
验证方法：#ls -lh /opt/kvm/centos7.img 看是否显示占用空间与输入的一致

 

2,安装操作系统
#virt-install -n Centos --ram=1024 --disk centos7.img --location CentOS-6.7-x86_64-bin-DVD1.iso --graphics none --check all=off --network network=default --vcpus=1 --extra-args='console=tty0 console=ttyS0'

注：
以上命令在当前/opt/kvm目录下操作，所有文件都为使用绝对路径-n  --指定系统名称
--ram  --指定内存大小
--disk  --指定系统盘
--location --指定安装的镜像盘路径、名称
--network network=default  --指定使用的网卡
--vcpus=1  --指定CPU核实
--check all=off --可删除
--check path_in_use=off 或 --check all=off 选项覆盖
--extra-args='console=tty0 console=ttyS0' 这串表示可以通过console的方式连接已经创建的虚拟机。
--graphics none这一串是表示在把图形化输出到命令行终端上。
另外一种把图形化输出到命令行操作终端的方法是修改系统上的/boot/grub/grub.conf文件，在kernel后面加上console ttyS0，vim /etc/securetty，添加ttyS0，然后重启系统。

验证方法：
1）安装过程无报错、输入#virsh list --all可以查看到已安装的系统信息。
2）virsh start Centos
3）virsh console Centos
4）成功进入虚拟系统

# KVM格式转换 快照管理
raw:裸格式，占用空间大，不支持快照，性能好
qcow2:qemu copy on write(qemu写时复制，thin)支持快照，性能较raw稍差

qemu-img info                        #查看磁盘信息  
qemu-img convert -f raw -O qcow2 name.raw name.qcow2         #转换格式,支持vmdk  
virsh snapshot-create            #快照  
virsh snapshot-list                  #快照列表,快照直接放在虚拟磁盘中  
virsh snapshot-delete vmname --snapshotname      #删除快照，删除快照文件不会变小  
virsh snapshot-revert vmname      #还原快照  
