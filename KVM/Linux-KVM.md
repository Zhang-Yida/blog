### 查看环境基本参数

```shell
# 192.168.0.173
# 物理 CPU - 2
[root@localhost ~]# grep 'physical id' /proc/cpuinfo | sort -u | wc -l
2
# CPU 核心数量 - 12
[root@localhost ~]# grep 'core id' /proc/cpuinfo | sort -u | wc -l
12
# 逻辑 CPU - 48
[root@localhost ~]# grep 'processor' /proc/cpuinfo | sort -u | wc -l
48
# 内存 125G
[root@localhost ~]# free -h
total    used    free   shared buff/cache  available
Mem:      125G     14G     75G     53M     34G    110G
Swap:     4.0G     0B    4.0G
```



### KVM 虚拟机安装

#### 为宿主机创建桥接网卡

```shell
# 关闭NetwokManager服务 (因为其会自动配置管理网络设置，不关闭的话，我们做的修改会失效) 且设置开机禁止
systemctl stop NetworkManager
systemctl disable NetworkManager

# 
cd /etc/sysconfig/network-scripts
vim ifcfg-em3

# 修改内容如下
------------------------------------------------------------
TYPE=Ethernet                                                                                                 
PROXY_METHOD=none                                                                                             
BROWSER_ONLY=no                                                                                               
BOOTPROTO=none                                                                                                
DEFROUTE=yes                                                                                                  
IPV4_FAILURE_FATAL=no                                                                                         
IPV6INIT=yes                                                                                                  
IPV6_AUTOCONF=yes                                                                                             
IPV6_DEFROUTE=yes                                                                                             
IPV6_FAILURE_FATAL=no                                                                                         
IPV6_ADDR_GEN_MODE=stable-privacy                                                                             
NAME=em3                                                                                                      
UUID=54ac89e3-fa49-43c4-902c-95f7616d1802                                                                     
DEVICE=em3                                                                                                    
ONBOOT=yes                                                                                                    
NM_CONTROLLED=yes                                                                                             
                                                                                                              
# IPADDR=10.0.0.14                                                                                            
# PREFIX=24                                                                                                   
# GATEWAY=10.0.0.1                                                                                            
# DNS1=114.114.114.114                                                                                        
                                                                                                              
BRIDGE=br0
------------------------------------------------------------

vim ifcfg-br0
# 添加内容如下
------------------------------------------------------------
TYPE=Bridge                                                                                                   
# PROXY_METHOD=none                                                                                           
# BROWSER_ONLY=no                                                                                             
BOOTPROTO=static                                                                                              
# DEFROUTE=yes                                                                                                
# IPV4_FAILURE_FATAL=no                                                                                       
# IPV6INIT=yes                                                                                                
# IPV6_AUTOCONF=yes                                                                                           
# IPV6_DEFROUTE=yes                                                                                           
# IPV6_FAILURE_FATAL=no                                                                                       
# IPV6_ADDR_GEN_MODE=stable-privacy                                                                           
NAME=br0                                                                                                      
# UUID=62e5d73d-726b-430f-91dd-d32ea29b6c41                                                                   
DEVICE=br0                                                                                                    
ONBOOT=yes                                                                                                    
NM_CONTROLLED=yes                                                                                             
                                                                                                              
IPADDR=10.0.0.14                                                                                              
PREFIX=24                                                                                                     
GATEWAY=10.0.0.1                                                                                              
DNS1=114.114.114.114
------------------------------------------------------------

# 想要远程连接，最好关闭防火墙
systemctl stop firewalld
systemctl disable firewalld
```



```
yum -y install kvm python-virtinst libvirt tunctl bridge-utils virt-manager qemu-kvm-tools virt-viewer virt-v2v virt-install libguestfs-tools
systemctl restart libvirtd

[root@controller ~]# yum install qemu-kvm libvirt virt-manager  
其中qemu-kvm是使我们的I/0设备支持虚拟化   
libvirt是管理kvm虚拟机的工具  
virt-manager是一种可视化的安装工具  
virt-install是一种命令行安装kvm虚拟机的方式
```

```shell
# 10.0.0.13 web_app
qemu-img create -f qcow2 /data/kvm_qcow2/web_app.qcow2 200G

virt-install -n web_app -r 131072 --vcpus 16 -l /opt/CentOS-7-x86_64-DVD-1908.iso --disk path=/data/kvm_qcow2/web_app.qcow2,format=qcow2 --nographics -x console=ttyS0 --network bridge=br0 --os-type=linux

# 10.0.0.13 service_app
qemu-img create -f qcow2 /data/kvm_qcow2/service_app.qcow2 500G

virt-install -n service_app -r 262144 --vcpus 32 -l /opt/CentOS-7-x86_64-DVD-1908.iso --disk path=/data/kvm_qcow2/service_app.qcow2,format=qcow2 --nographics -x console=ttyS0 --network bridge=br0 --os-type=linux

# 10.0.0.13 file_nginx
qemu-img create -f qcow2 /data/kvm_qcow2/file_nginx.qcow2 100G

virt-install -n file_nginx -r 32768 --vcpus 4 -l /opt/CentOS-7-x86_64-DVD-1908.iso --disk path=/data/kvm_qcow2/file_nginx.qcow2,format=qcow2 --nographics -x console=ttyS0 --network bridge=br0 --os-type=linux

# 10.0.0.13 file_tracker
qemu-img create -f qcow2 /data/kvm_qcow2/file_tracker.qcow2 100G

virt-install -n file_tracker -r 32768 --vcpus 4 -l /opt/CentOS-7-x86_64-DVD-1908.iso --disk path=/data/kvm_qcow2/file_tracker.qcow2,format=qcow2 --nographics -x console=ttyS0 --network bridge=br0 --os-type=linux

# 10.0.0.13 file_storage
qemu-img create -f qcow2 /data/kvm_qcow2/file_storage.qcow2 4T

virt-install -n file_storage -r 32768 --vcpus 4 -l /opt/CentOS-7-x86_64-DVD-1908.iso --disk path=/data/kvm_qcow2/file_storage.qcow2,format=qcow2 --nographics -x console=ttyS0 --network bridge=br0 --os-type=linux

# 10.0.0.15 middleware
qemu-img create -f qcow2 /data/kvm_qcow2/middleware.qcow2 500G

virt-install -n middleware -r 262144 --vcpus 32 -l /opt/CentOS-7-x86_64-DVD-1908.iso --disk path=/data/kvm_qcow2/middleware.qcow2,format=qcow2 --nographics -x console=ttyS0 --network bridge=br0 --os-type=linux

# 10.0.0.15 file_nginx
qemu-img create -f qcow2 /data/kvm_qcow2/file_nginx.qcow2 200G

virt-install -n file_nginx -r 32768 --vcpus 8 -l /opt/CentOS-7-x86_64-DVD-1908.iso --disk path=/data/kvm_qcow2/file_nginx.qcow2,format=qcow2 --nographics -x console=ttyS0 --network bridge=br0 --os-type=linux

# 10.0.0.15 file_storage
qemu-img create -f qcow2 /data/kvm_qcow2/file_storage.qcow2 4T

virt-install -n file_storage -r 32768 --vcpus 4 -l /opt/CentOS-7-x86_64-DVD-1908.iso --disk path=/data/kvm_qcow2/file_storage.qcow2,format=qcow2 --nographics -x console=ttyS0 --network bridge=br0 --os-type=linux

# 10.0.0.16 file_storage
qemu-img create -f qcow2 /data/kvm_qcow2/file_storage.qcow2 4T

virt-install -n file_storage -r 32768 --vcpus 4 -l /opt/CentOS-7-x86_64-DVD-1908.iso --disk path=/data/kvm_qcow2/file_storage.qcow2,format=qcow2 --nographics -x console=ttyS0 --network bridge=br0 --os-type=linux


IPADDR=10.0.0.201
NETMASK=255.255.255.0
GATEWAY=10.0.0.1
DNS1=114.114.114.114

// 设置 ntp
yum install ntp ntpdate -y
sed -i 's@^server@#server@' /etc/ntp.conf
ntpdate 192.168.0.222
date
vim /etc/crontab
* */1 * * * ntpdate -s 192.168.0.222




参数详解，参考 https://www.cnblogs.com/saryli/p/11827903.html

-n 虚拟机名称
-r 虚拟机内存大小，单位 MB
--vcps cpu 个数

-l 系统镜像文件物理地址
--disk 指定存储设备及属性
--nographics 安装过程中不启动图形界面
-x 安装系统镜像时，传递给内核的额外参数
--network 将虚拟机连入宿主机的网络中
--os-type 操作系统类型
```





```shell
# 更换虚拟机 IP
virsh list
```



```shell
# 更改 yum 源

cd /etc/yum.repos.d
rm -f *
vi localyum.repo


[localyum]
name=centos7
baseurl=http://10.0.0.246:8080/centos/7/os/x86_64/
enable=1
gpgcheck=0

[localepel]
name=epel
baseurl=http://10.0.0.246:8080/epel/7/x86_64/
enable=1
gpgcheck=0

[localextra]
name=extra
baseurl=http://10.0.0.246:8080/centos/7/extras/x86_64/
enable=1
gpgcheck=0


```

#### 删除虚拟机

```sh
virsh destroy NAME
virsh undefine NAME

# 查看列表
virsh list --all
```





### OpenStack

#### Controller

```shell l
# 待补充

# 数据库
[root@sungrow-prod-16 my.cnf.d]# rabbitmqctl add_user openstack openstack 

# Memcached for RHEL and CentOS
[root@sungrow-prod-16 my.cnf.d]# yum install memcached python-memcached.noarch -y  

```



```
keystone-manage bootstrap --bootstrap-password keystone \
  --bootstrap-admin-url http://controller:5000/v3/ \
  --bootstrap-internal-url http://controller:5000/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne
```



glance / glance 



### 扩容根目录

```shell
# fuser 如果找不到
yum install psmisc
```



### MySQL 离线安装



```shell
rpm -qa | grep mariadb
rpm -e --nodeps mariadb-libs
# 安装 rpm 离线安装包
rpm -Uvh *.rpm --nodeps --force

# 数据库初始化
mysqld --initialize --user=mysql

# 查看生成的密码
grep 'temporary password' /var/log/mysqld.log

# 修改 mysql 的 root 密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';

```



###  Ambari + HDP 生产环境搭建



