基于Ubuntu的虚拟机创建自己的box。  
  
过程中主要参考了这篇文章：http://www.360doc.com/content/13/0826/17/13047933_310047914.shtml  
  
主要过程简述如下：  
  
1、安装virtualbox  
2、安装 vagrant  
3、下载Ubuntu的iso文件，并安装虚拟机，注意要采用以下用户名、密码  
 root密码：vagrant  
 用户名：vagrant  
 用户密码： vagrant  
4、完成后进入虚拟机安装必要组件，并进行设置  
  
安装vim  
安装openssh-server  
安装vagrant的public keys（参考文较详细）  
  
cd ~/.ssh  
wget http：//github.com/mitchellh/raw/master/keys/vagrant  
wget http：//github.com/mitchellh/raw/master/keys/vagrant.pub  
mv vagrant.pub authorized_keys  
  
  注意文件名要正确，因为文件名错误耽误了好长时间。  
5、权限变更（非常重要，原文部分没有），否则ssh有问题  
  sudo gpassd -a vagrant sudo  //加入sudo   
  sudo vim /etc/sudoers  
 添加   Default env_keep = "SSH_AUTH_SOCK"  
  更改  %admin ALL=(ALL)   NOPASSWD:ALL  
  cd ~  
   chmod 700 ~/.ssh  
   chmod 600 ~/.ssh/autorized_keys  
6、设置虚拟机端口映射  
进入virtualbox 图形界面->网络->高级->端口映射  
  ssh2222 映射到22，不用设置宿主机及客户机  
7、打包  box文件  
   进入virtuallbox默认虚拟电脑位置所在的目录。可通过管理全局设定查看。  
   vagrant package  --base    boxname   --output centos-xxxx.box  
8、在vagrant下安装使用   
  vagrant box add centos centos-xxxx.box  #add box  
  vagrant init centos  #init box
