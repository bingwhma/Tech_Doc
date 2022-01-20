使用packer制作vagrant centos box  
  
制作vagrant box，网上有教程，可以自己step by step的操作。不过直接使用虚拟在VirtualBox中制作vagrant box非常的费劲。网上有操作debian/ubuntu相关的文章，放2篇在这里供参考：  
  
    英文 How to Create and Share a Vagrant Base Box  
    中文 制作 Vagrant Box 简明教程。  
  
做到快结束的时候，就不知道如何解决了。后来发现有个Packer工具。就简单多了。  
  
下面基于packer来操作centos7的box。所有操作在Mac下完成。对于一个devops，Mac自然是最好的工作环境，而centos则是线上的流行版本。  
# 1 准备工具和文件  
mkdir -p /Users/user/myVagrant/packer/src/  
cd /Users/user/myVagrant/packer/src/  
  
下载如下文件：  
## 1.1 镜像 centos7.2  
  
http://mirrors.163.com/centos/7.2.1511/isos/x86_64/sha1sum.txt  
  
http://mirrors.163.com/centos/7.2.1511/isos/x86_64/CentOS-7-x86_64-Minimal-1511.iso  
  
注意验证iso文件的sha1.  
  
shasum CentOS-7-x86_64-Minimal-1511.iso  
783eef50e1fb91c78901d0421d8114a29b998478  CentOS-7-x86_64-Minimal-1511.iso  
  
## 1.2 packer工具  
https://www.packer.io/downloads.html  
https://www.packer.io/intro/getting-started/setup.html  
  
ls -l   
-rw-r--r--@  1 user  staff   10395909 12 17 23:49 packer_0.12.1_darwin_amd64.zip  
  
unzip packer_0.12.1_darwin_amd64.zip   
  
将解压出来的packer文件移到$PATH可见的目录，方便后继操作    
sudo mv packer /usr/local/sbin/  
  
## 1.3 centos.json  
  
这配置也是个神器，和packer配合，真是一对。不然很多工作都得按本文开始说的step by step操作了。先上github:    
    https://github.com/boxcutter/centos.git  
  
cd ../  
pwd  
/Users/user/myVagrant/packer  
#先上退一级目录  
git clone https://github.com/boxcutter/centos.git  
cd centos  
#在这里，能看到各个centos版本的配置文件。而且有一个centos.json文件，是没有版本的  
  
ls -l centos.json centos72.json  
-rw-r--r--  1 user  staff  6207 12 17 23:46 centos.json  
-rw-r--r--  1 user  staff   459 12 18 09:03 centos72.json  
#这是我们关心的2个配置文件。第一个就预设了安装的所有参数和命令。可以打开看看  
#centos72.json则是用来给用户填写需要自定义的部分，用其覆盖centos.json对应部分  
  
# 2 准备工作  
  
pwd  
/Users/user/myVagrant/packer/centos  
  
#文件：/Users/user/myVagrant/packer/src/CentOS-7-x86_64-Minimal-1511.iso  
#命令：/usr/local/sbin/packer  
#配置：centos72.json， centos.json  
  
修改centos72.json  
  
这里就几行配置，需要关心的就是两类：  
  
    系统资源：cpus， disk_size, memory  
    镜像文件：检验码，检验码类型，文件路径  
  
一般只需要修改一下检验码和文件路径即可。检验码即上面使用shasum得到的串。  
  
需要特别说明的是disk_size，预设的值是65536，单位是G。也就是整个虚机系统的磁盘空间是64G。根据实测的结果，根分区/将得到42G，而/home分区得到21G。  
  
/dev/mapper/centos-root   42G  811M   42G   2% /  
/dev/mapper/centos-home   21G   33M   21G   1% /home  
  
这个结果我不太满意，因此设置为128G，反正是动态分配磁盘空间的。也就不怕大，只怕后面用着对21G空间不够就麻烦了。  
  
git diff centos72.json  
  
diff --git a/centos72.json b/centos72.json  
index 83ed3d5..33d1a2e 100644  
--- a/centos72.json  
+++ b/centos72.json  
@@ -2,11 +2,11 @@  
   "_comment": "Build with `packer build -var-file=centos72.json centos.json`",  
   "vm_name": "centos72",  
   "cpus": "1",  
-  "disk_size": "65536",  
-  "iso_checksum": "4c6c65b5a70a1142dadb3c65238e9e97253c0d3a",  
+  "disk_size": "131072",  
+  "iso_checksum": "783eef50e1fb91c78901d0421d8114a29b998478",  
   "iso_checksum_type": "sha1",  
-  "iso_name": "CentOS-7-x86_64-DVD-1511.iso",  
-  "iso_url": "http://mirrors.sonic.net/centos/7.2.1511/isos/x86_64/CentOS-7-x86_64-DVD-1511.iso",  
+  "iso_name": "CentOS-7-x86_64-Minimal-1511.iso",  
+  "iso_url": "file:///Users/user/myVagrant/packer/src/CentOS-7-x86_64-Minimal-1511.iso",  
   "kickstart": "ks7.cfg",  
   "memory": "512",  
   "parallels_guest_os_type": "centos7"  
  
# 3 操作  
## 3.1 说明  
  
如https://github.com/boxcutter/centos首页所说，生成box是调用的packer工具，可以有不同的方法：  
  
    with packer: packer build xxx  
    with box script: bin/box，是对packer的封装。  
    with Makefile: 使用make命令  
  
直接使用packer的命令更长，下面使用box命令来操作。其实  
  
$ packer build -var-file=centos71.json centos.json  
$ bin/box build centos71  
#这两个配置是等价的。  
  
因为默认只用于virtualBox虚机环境，因此下面的操作是指明了类型为virtualbox的。  
  
在操作前需要再提醒一个关键事项：不要干预后继命令及virtualBox环境的执行！！！让它自动完成  
你要做的事就是看着下面的注释，与终端和virtualBox的显示对比就是了。  
## 3.2 action!!!  
#下面2个命令都可以  
#packer build -only=virtualbox-iso -var-file=centos72.json centos.json  
bin/box build centos72 virtualbox  
 #注意，在此命令启动后，会调用virtualBox的启动虚机。  
 #此时一定不要去点击该虚机。它会自动向下执行！！！  
 #如果人工干预了，实测系统会中断在那里。而且进入的是图形模式  
 #你只有中断了重来一遍  
   
 #如果你没有指定virtualbox，则会生成多个虚机平台的box，下面只有virtualbox  
virtualbox-iso output will be in this color.  
  
==> virtualbox-iso: Downloading or copying Guest additions  
    virtualbox-iso: Downloading or copying: file:///Applications/VirtualBox.app/Contents/MacOS/VBoxGuestAdditions.iso  
==> virtualbox-iso: Downloading or copying ISO  
    virtualbox-iso: Downloading or copying: file:///Volumes/Storage/software/centos/CentOS-7-x86_64-Minimal-1511.iso  
    virtualbox-iso: Error downloading: stat /Volumes/Storage/software/centos/CentOS-7-x86_64-Minimal-1511.iso: no such file or directory  
    virtualbox-iso: Downloading or copying: file:///Users/user/myVagrant/packer/src/CentOS-7-x86_64-Minimal-1511.iso  
#virtualBox中的虚机在字符模式下自动运行  
#上面是在读取镜像文件  
  
==> virtualbox-iso: Starting HTTP server on port 8478  
==> virtualbox-iso: Creating virtual machine...  
==> virtualbox-iso: Creating hard drive...  
==> virtualbox-iso: Creating forwarded port mapping for communicator (SSH, WinRM, etc) (host port 2224)  
==> virtualbox-iso: Executing custom VBoxManage commands...  
    virtualbox-iso: Executing: modifyvm centos72 --memory 512  
    virtualbox-iso: Executing: modifyvm centos72 --cpus 1  
    virtualbox-iso: Executing: modifyvm centos72 --paravirtprovider default  
    virtualbox-iso: Executing: modifyvm centos72 --nictype1 virtio  
==> virtualbox-iso: Starting the virtual machine...  
==> virtualbox-iso: Waiting 10s for boot...  
#这时在等开机的安装选择，系统会自动处理。virtulBox已经启动了  
  
==> virtualbox-iso: Typing the boot command...  
==> virtualbox-iso: Waiting for SSH to become available...  
#这里自动在自动的安装加载中，直到系统装好并重启，日志都不会再变化  
  
#观察这个文件，大小一直在增加，大概有1G左右   
ls -lh output-centos72-virtualbox-iso/centos72.vdi  
-rw-------  1 user  staff   576M 12 18 09:13 output-centos72-virtualbox-iso/centos72.vdi  
ls -lh output-centos72-virtualbox-iso/centos72.vdi  
-rw-------  1 user  staff   867M 12 18 09:14 output-centos72-virtualbox-iso/centos72.vdi  
ls -thrl output-centos72-virtualbox-iso/centos72.vdi  
-rw-------  1 user  staff   915M 12 18 10:21 output-centos72-virtualbox-iso/centos72.vdi  
  
#系统主要执行如下相关操作：  
#安装包，执行post-install处理，安装boot loader，配置用户环境。。。  
#准备好后，会自动重启，重启时也不要去干预它！  
#重启后，virtualBox进行等登陆的界面。这时该命令继续运行了  
==> virtualbox-iso: Connected to SSH!  
==> virtualbox-iso: Uploading VirtualBox version info (5.1.10)  
==> virtualbox-iso: Uploading VirtualBox guest additions ISO...  
  
#登陆进系统后，执行 script/fix-slow-dns.sh命令  
==> virtualbox-iso: Provisioning with shell script: script/fix-slow-dns.sh  
    virtualbox-iso: ==> Applying slow DNS fix  
    virtualbox-iso: Restarting network (via systemctl):  [  OK  ]  
    virtualbox-iso: ==> Slow DNS fix applied (single-request-reopen)  
  
#执行script/sshd.sh修改配置。  
==> virtualbox-iso: Provisioning with shell script: script/sshd.sh  
    virtualbox-iso: ==> Configuring sshd_config options  
    virtualbox-iso: ==> Turning off sshd DNS lookup to prevent timeout delay  
    virtualbox-iso: ==> Disablng GSSAPI authentication to prevent timeout delay  
#执行其它 脚本      
==> virtualbox-iso: Provisioning with shell script: script/update.sh  
==> virtualbox-iso: Provisioning with shell script: script/vagrant.sh  
    virtualbox-iso: ==> Configuring settings for vagrant  
    virtualbox-iso: ==> Installing Vagrant SSH key  
==> virtualbox-iso: Provisioning with shell script: script/desktop.sh  
==> virtualbox-iso: Provisioning with shell script: script/vmware.sh  
==> virtualbox-iso: Provisioning with shell script: script/virtualbox.sh  
    virtualbox-iso: ==> Installing VirtualBox guest additions  
    virtualbox-iso: mount: /dev/loop0 is write-protected, mounting read-only  
    virtualbox-iso: Verifying archive integrity... All good.  
    virtualbox-iso: Uncompressing VirtualBox 5.1.10 Guest Additions for Linux...........  
    virtualbox-iso: VirtualBox Guest Additions installer  
    virtualbox-iso: Copying additional installer modules ...  
    virtualbox-iso: ./install.sh: line 349: bzip2: command not found  
    virtualbox-iso: tar: This does not look like a tar archive  
    virtualbox-iso: tar: Exiting with failure status due to previous errors  
    virtualbox-iso: ./install.sh: line 362: bzip2: command not found  
    virtualbox-iso: tar: This does not look like a tar archive  
    virtualbox-iso: tar: Exiting with failure status due to previous errors  
    virtualbox-iso: ==> Removing packages needed for building guest tools  
    virtualbox-iso: Loaded plugins: fastestmirror  
    virtualbox-iso: No Match for argument: gcc  
    virtualbox-iso: No Match for argument: libmpc  
    virtualbox-iso: No Match for argument: mpfr  
    virtualbox-iso: No Match for argument: kernel-devel  
    virtualbox-iso: No Match for argument: kernel-headers  
    virtualbox-iso: No Packages marked for removal  
    virtualbox-iso: Loaded plugins: fastestmirror  
    virtualbox-iso: No Match for argument: cpp  
    virtualbox-iso: No Match for argument: perl  
    virtualbox-iso: No Packages marked for removal  
==> virtualbox-iso: Provisioning with shell script: script/parallels.sh  
==> virtualbox-iso: Provisioning with shell script: script/motd.sh  
    virtualbox-iso: ==> Recording box generation date  
    virtualbox-iso: ==> Customizing message of the day  
==> virtualbox-iso: Provisioning with shell script: custom-script.sh  
==> virtualbox-iso: Provisioning with shell script: script/cleanup.sh  
    virtualbox-iso: ==> Clear out machine id  
    virtualbox-iso: ==> Cleaning up temporary network addresses  
    virtualbox-iso: ==> Remove unused man page locales  
    virtualbox-iso: /usr/share/man /home/vagrant  
    virtualbox-iso: mv: cannot stat ‘en’: No such file or directory  
    virtualbox-iso: /home/vagrant  
    virtualbox-iso: ==> Clean up yum cache of metadata and packages to save space  
    virtualbox-iso: Loaded plugins: fastestmirror  
    virtualbox-iso: Cleaning repos: C7.0.1406-base C7.0.1406-centosplus C7.0.1406-extras  
    virtualbox-iso: : C7.0.1406-fasttrack C7.0.1406-updates C7.1.1503-base  
    virtualbox-iso: : C7.1.1503-centosplus C7.1.1503-extras C7.1.1503-fasttrack  
    virtualbox-iso: : C7.1.1503-updates base base-debuginfo base-source c7-media  
    virtualbox-iso: : centosplus centosplus-source cr extras extras-source fasttrack  
    virtualbox-iso: : updates updates-source  
    virtualbox-iso: Cleaning up everything  
    virtualbox-iso: ==> Clear core files  
    virtualbox-iso: ==> Removing temporary files used to build box  
    virtualbox-iso: ==> Rebuild RPM DB  
    virtualbox-iso: ==> Clear out swap and disable until reboot  
    virtualbox-iso: dd: error writing ‘/dev/dm-1’: No space left on device  
    virtualbox-iso: 1025+0 records in  
    virtualbox-iso: 1024+0 records out  
    virtualbox-iso: 1073741824 bytes (1.1 GB) copied, 3.07468 s, 349 MB/s  
    virtualbox-iso: dd exit code 1 is suppressed  
    virtualbox-iso: Setting up swapspace version 1, size = 1048572 KiB  
    virtualbox-iso: no label, UUID=3c9082d3-be31-4f58-8700-9ab428f5f052  
    virtualbox-iso: ==> Zeroing out empty area to save space in the final image  
#上面在清除一些无用的文件，减少box的大小。并将空白区域置0  
  
    virtualbox-iso: dd exit code 1 is suppressed  
    virtualbox-iso: dd: error writing ‘/EMPTY’: No space left on device  
    virtualbox-iso: 50365+0 records in  
    virtualbox-iso: 50364+0 records out  
    virtualbox-iso: 52811284480 bytes (53 GB) copied, 139.999 s, 377 MB/s  
    virtualbox-iso: ==> Disk usage before cleanup  
#这里的dd命令，可以对应到：http://www.tuicool.com/articles/F7ZjQvy中的：  
#sudo dd if=/dev/zero of=/EMPTY bs=1M  
#sudo rm -f /EMPTY      
      
    virtualbox-iso: Filesystem Size Used Avail Use% Mounted on /dev/mapper/centos-root 50G 777M 50G 2% / devtmpfs 235M 0 235M 0% /dev tmpfs 245M 0 245M 0% /dev/shm tmpfs 245M 4.3M 241M 2% /run tmpfs 245M 0 245M 0% /sys/fs/cgroup /dev/mapper/centos-home 77G 33M 77G 1% /home /dev/sda1 497M 60M 437M 13% /boot tmpfs 49M 0 49M 0% /run/user/1000  
    virtualbox-iso: ==> Disk usage after cleanup  
    virtualbox-iso: Filesystem               Size  Used Avail Use% Mounted on  
    virtualbox-iso: /dev/mapper/centos-root   50G  811M   50G   2% /  
    virtualbox-iso: devtmpfs                 235M     0  235M   0% /dev  
    virtualbox-iso: tmpfs                    245M     0  245M   0% /dev/shm  
    virtualbox-iso: tmpfs                    245M  4.3M  241M   2% /run  
    virtualbox-iso: tmpfs                    245M     0  245M   0% /sys/fs/cgroup  
    virtualbox-iso: /dev/mapper/centos-home   77G   33M   77G   1% /home  
    virtualbox-iso: /dev/sda1                497M   64M  433M  13% /boot  
    virtualbox-iso: tmpfs                     49M     0   49M   0% /run/user/1000  
#上面输出了清理环境后，整个磁盘空间的大小情况。  
#根分区有50G，/home分区的77G。够用了  
  
==> virtualbox-iso: Gracefully halting virtual machine...  
==> virtualbox-iso: Preparing to export machine...  
    virtualbox-iso: Deleting forwarded port mapping for the communicator (SSH, WinRM, etc) (host port 4046)  
==> virtualbox-iso: Exporting virtual machine...  
    virtualbox-iso: Executing: export centos72 --output output-centos72-virtualbox-iso/centos72.ovf  
==> virtualbox-iso: Unregistering and deleting virtual machine...  
==> virtualbox-iso: Running post-processor: vagrant  
==> virtualbox-iso (vagrant): Creating Vagrant box for 'virtualbox' provider  
    virtualbox-iso (vagrant): Copying from artifact: output-centos72-virtualbox-iso/centos72-disk1.vmdk  
    virtualbox-iso (vagrant): Copying from artifact: output-centos72-virtualbox-iso/centos72.ovf  
    virtualbox-iso (vagrant): Renaming the OVF to box.ovf...  
    virtualbox-iso (vagrant): Compressing: Vagrantfile  
    virtualbox-iso (vagrant): Compressing: box.ovf  
    virtualbox-iso (vagrant): Compressing: centos72-disk1.vmdk  
    virtualbox-iso (vagrant): Compressing: metadata.json  
Build 'virtualbox-iso' finished.  
  
==> Builds finished. The artifacts of successful builds are:  
--> virtualbox-iso: 'virtualbox' provider box: box/virtualbox/centos72-2.0.22.box  
#系统将output-centos72-virtualbox-iso/下的临时文件处理压缩为：  
#box/virtualbox/centos72-2.0.22.box      
#整个命令结束。  
  
ls -hl box/virtualbox/centos72-2.0.22.box  
-rw-r--r--  1 user  staff   363M 12 18 10:37 box/virtualbox/centos72-2.0.22.box  
#box文件只有363M  
  
至此，box成功生成。

REF URL：
https://www.cnblogs.com/qinqiao/p/vagrant-install-and-questions.html
