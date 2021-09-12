# android_termux_install_linux_ssh_telnet
安卓termux安装linux正式发行版并配置SSH及telnet过程

play store 安装 termux   
   
启动进入终端命令行界面   
   
首先安装proot（伪root），第一次运行一定会报源出错，需要运行第二次，等于马上运行两次   
$pkg install proot   
   
如果网络不好请富强  
   
每次重新开机都要执行update才能安装软件，否则可能报告源有问题，当然不安装软件就可以跳过  
$pkg update  
  
安装基本软件主要是ssh软件  
$pkg install nano openssh wget -y  
  
  
启动ssh进程，sshd缺省监听端口8022，因为安卓系统不允许监听低于1024号的端口号  
$sshd  
  
一定要查看现在自己的用户名，新装机的用户名每次不一样，但不卸载的话就一直一样  
$whoami  
  
输出：  
  
u0_a395  

更改用户名的登录密码  
$passwd  
  
获取本机ip，由于termux是一个功能受限的虚拟机平台，所以它使用了类似容器的网络主机模式(host)，  
即与主机共用所有的网络接口，也即android主机的网络就是它的网络，android主机的ip地址就是它的ip地址，  
它监听的端口也等于主机监听的端口  
$ifconfig wlan0  
  
  
至此就可以在其他机器用u0_a395这个用户名及刚才修改的密码ssh登录termux了，  
然后在其他机器终端软件操作下面的命令会比较方便,   
也可以用下面命令改sshd进程的监听端口，不过一般不需要  
$sshd -p 8888  
  
  
---------安装linux正式发行版的容器版部署工具及安装指定的linux发行版---------  
  
安装linux正式发行版的容器版部署工具  
$pkg install proot-distro  
   
容器部署命令  
  
proot-distro list - show the supported distributions and their status.  
proot-distro install - install a distribution.  
proot-distro login - start a root shell for the distribution.  
proot-distro remove - uninstall the distribution.  
proot-distro reset - reinstall the distribution.  
  
先用proot-distro list命令查看可用的发行版及别名  
$proot-distro list  
  
会输出很多linux的正式发行版，举例我们安装ubuntu的话，看下面的输出：  
* Ubuntu (hirsute)  
  
Alias: ubuntu  
Status: NOT installed  
  
其中Alias的小写名字ubuntu就是我们安装需要用到的名字  
  
安装容器ubuntu发行版  
$proot-distro install ubuntu  
  
运行ubuntu容器的命令，进入后就是root用户，但安装好后一般不要先进去，而是先做好下面的步骤，以方便后面启动  
$proot-distro login ubuntu  
  
编辑运行ubuntu容器的脚本  
$nano ~/u  
  
写入下面内容  
  
proot-distro login ubuntu  
  
保存退出并置运行权限  
  
$chmod +x ~/u  
  
追加termux开机执行命令，增加命令提示符内容，映射ll命令等于ls -laF命令，开机启动sshd进程，用source命令运行u脚本，  
      
$touch ~/.bashrc  
$echo "PS1=\"\u@\s:\w\$\"" >> ~/.bashrc  
$echo "alias ll='ls -alF'" >> ~/.bashrc  
$echo "sshd" >> ~/.bashrc  
$echo "source /data/data/com.termux/files/home/u" >> ~/.bashrc  
  
  
重新启动termux，确认手机终端界面已经自动登录ubuntu的root用户，然后通过termux终端界面配置ubuntu用户名及密码登录，  
由于现在termux的~/.bashrc文件已经自动登录ubuntu了，所以其他机器终端程序登录termux会出现ubuntu的root用户界面  
  
先改ubuntu的root密码  
  
#passwd  
  
一定要先用apt update更新源，另外因为termux安装的linux不能用密码登录ssh，只能是证书登录，比较麻烦，   
所以需要安装telnet服务，才能用telnet密码方式登录，所以安装telnet服务  
  
#apt update -y  
#apt install openbsd-inetd telnetd -y  
  
  
修改telnet的配置文件/etc/inetd.conf里面的telnet监听端口号，因为安卓系统不允许监听低于1024号的端口号  
  
#nano /etc/inetd.conf  
  
找到其中这行的开始服务名“telnet”： telnet stream tcp nowait telnetd /usr/sbin/tcpd /usr/sbin/in.telnetd  
将“telnet”修改为自己需要的监听端口，如：5223 stream tcp nowait telnetd /usr/sbin/tcpd /usr/sbin/in.telnetd  
  
保存并退出  
  
设置termux启动linux后立即运行telnet服务  
  
#echo "/etc/init.d/openbsd-inetd start" >> ~/.bashrc  
  
使用source运行一次~/.bashrc文件  
  
#source ~/.bashrc  
  
  
至此已经可以在其他机器上使用终端程序telnet直接以用户root及密码方式登录这个ubuntu容器了，telnet端口号就是上面的5223，  
由于ubuntu使用了容器的网络主机模式(host)，即与主机共用所有的网络接口，也即android主机的网络就是它的网络，  
android主机的ip地址就是它的ip地址，查登录接口ip地址的命令：  
  
#ifconfig wlan0  
  
这个linux的正式发行版支持安装源库拥有的全部语言编译工具  
  
