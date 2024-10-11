
1、讲在前面（玩这个的心历路程）


最近一段时间想玩一些集群之类的东西，学习搞一下K8s，集群啥的，但是我没有多台服务器，如果购买云服务器成本太高，后来想到了买台台式机弄点虚拟机来玩，于是我就在某鱼上淘了台二手台式机(24核\+32G\+512G\+4G显卡)，价格1280。后来想到要装虚拟机，我就想到了现在经常用到的云平台，于是在网上找了一些开源云平台，于是找了一些开源云平台


2、最终选择KVM的原因


最开始选择的是FreeVM，纯国产化安全虚拟化平台，直接有一体包安装简单，纯中文界面，对于国人相对友好，另外看了介绍操作比较简单并且开源(企业版功能基本上用不到)，所以我就使用了这个，官网有一体包，直接下载ISO镜像，像装操作系统一样的，根据官网流程安装很简单。最终舍弃的原因是，不稳定，短短一个星期我云平台重装了两次(可能是我的硬件也太垃圾，或者搭建好之后我总是能精确命中系统bug，总之我的感觉就是不稳定)，果断舍弃。


舍弃FreeVM我又查了一下最稳定和历史悠久的要属于OpenStack，一个开源的云计算管理平台项目。OpenStack为私有云和公有云提供可扩展的弹性的云计算服务。项目目标是提供实施简单、可大规模扩展、丰富、标准统一的云计算管理平台。当时追求稳定性，没有考虑安装复杂度和配置问题，直接开始找各种教程开始干，趁周末从早上搞到晚上才搞好，最终发现不适用于我这种情况，OpenStack相对来说较重，里面各种组件有十几个，整个服务启动起来，直接把我内存吃完了。于是我果断放弃了


后来想通了，如果想搭建一个即稳定又实用的云平台靠这一台机器这点资源很难实现，于是我就考虑到了最笨的方式，使用虚拟机(类似于VMware)。因为我的原系统定位就是Centos，经过查询之后找到了KVM(一个开源的系统虚拟化模块)。虚拟化需要硬件支持(如Intel VT技术或者AMD V技术)。是基于硬件的完全虚拟化


3、说干就干，开始安装(Centos系统)


我是直接用yum装的，现在因为Centos已经停止维护，yum源可以用阿里源或者腾讯源，阿里源有个问题，使用在虚拟机上时间久了容易被封掉IP，导致你的IP无法再用(当时因为这个让我排查了好久)。如果遇到yum源的问题，可以直接换源解决问题。毕竟都不是慈善家，免费的东西咱也不好说啥。


安装命令步骤直接在下面：



```
# 构建虚拟机的命令行工具
yum -y install virt-install

# 网络支持工具, 默认已安装
yum -y install bridge-utils 

# 安装虚拟机管理工具
yum -y install libvirt libvirt-devel libvirt-daemon-kvm libvirt-client
yum -y install virt-manager

# 开启 libvirtd KVM服务，以开启相关支持：
systemctl start libvirtd
systemctl enable libvirtd --now
 
# 安装其它工具包：
yum install libvirt-python python-virtinst virt-install virt-viewer –y
yum install libguestfs-tools -y


```

截止上面最后一步，KVM已经安装好了


![](https://img2024.cnblogs.com/blog/1470032/202410/1470032-20241010192344769-58451425.png)


这些都是相关的一些命令，看着很多实际上用到的也就两三个(因为我目前只用了两三个)。virsh、virt\-install、virt\-manager


4、使用kvm创建虚机，virt\-install命令


​ 上面已经安装好了KVM，接下来要开始用KVM创建虚机，虚机需要有镜像，这里用还是用Centos来做例子,我是在阿里云的下载的：[https://mirrors.aliyun.com/centos/7/isos/x86\_64](https://github.com)



```
# 下载镜像
wget https://mirrors.aliyun.com/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-2009.iso

# 把镜像放到你的自定义位置
mv CentOS-7-x86_64-DVD-2009.iso /data/iso/

# 创建一个名称为master 内存8196M 8个C 的虚机 存储卷在/var/lib/libvirt/images/master.qcow2
virt-install --name=master --memory=8196 --vcpus=8 --os-type=linux --location=/data/iso/CentOS-7-x86_64-DVD-2009.iso --disk /data/vmdisk/images/master.qcow2,device=disk,bus=virtio,size=80 --network network=default --network bridge=virbr0 --nographics --extra-args='console=tty0 console=ttyS0,115200n8 serial'

```

virt\-install命令相关



```
#虚拟机镜像文件默认路径：/var/lib/libvirt/images/
磁盘镜像文件以qcow2、img、raw等格式后缀

磁盘镜像文件格式:
  虚拟机磁盘文件有raw、qcow2格式和qed（这种已经不用了）。qcow2格式是kvm支持的标准格式，raw格式为虚拟磁盘文件通用格式。raw格式性能最好，速度最快，其缺点是不支持一些新的功能，如镜像，Zlib磁盘压缩、AES加密、快照等，另外raw格式文件比qcow2格式文件大很多，将近15倍吧。而qcow2格式是支持快照模式，做快照要把它转换成qcow2格式。
 
#命令创建虚拟机示例
virt-install \        #创建命令 
-n kvm1 \          #虚拟机显示名（非虚拟机主机名）
-r 4096 \          #虚拟机内存大小 
--vcpus 2 \          #虚拟机cpu个数 
--disk path=/var/lib/libvirt/images/kvm1.qcow2,size=50,format=qcow2,bus=virtio \     #指定硬盘路径，大小，格式为qcow2,总线类型为virtio 
--location /root/iso/CentOS-7-x86_64-Minimal-2009.iso \    #系统安装iso路径 
--nographics \                    #不调用图形化界面 
--network network=default \                #网卡1指定网桥 
--network bridge=br0 \                #网卡2指定网桥 
--console pty,target_type=serial \          #console控制通道 
--extra-args 'console=ttyS0,115200n8 serial'      #文本输出 
 
或者vnc方式连接安装
 
virt-install \
--name=kvm001 --ram 1024 --vcpus=1 \
--disk path=/home/raw/kvm001.raw,size=10,format=raw,bus=virtio \
--cdrom=/mnt/CentOS-7-x86_64-Minimal-1810.iso --network bridge=br0,model=virtio \
--graphics vnc,listen=0.0.0.0 --noautoconsole
 
参数说明：
 
--name    #虚拟机名称
--ram     #分配给虚拟机的内存，单位MB
--vcpus   #分配给虚拟机的cpu个数
--cdrom   #指定CentOS镜像ISO文件路径
--disk    #指定虚拟机raw文件路径
  size    #虚拟机文件大小，单位GB
  bus     #虚拟机磁盘使用的总线类型，为了使虚拟机达到好的性能，这里使用virtio
  cache   #虚拟机磁盘的cache类型
--network bridge    #指定桥接网卡
   model            #网卡模式，这里也是使用性能更好的virtio
--graphics          #图形参数 
 
 

```

5、虚机管理 virsh


虚拟机状态维护



```
virsh list --all                 #查看所有虚拟机
virsh dominfo 虚拟机名或虚拟机ID    #查看虚拟机信息概览
virsh console 虚拟机名或虚拟机ID    #进入虚拟机
快捷键： ctrl+]   								 #退出虚拟机
virsh shutdown 虚拟机名或虚拟机ID   #关闭虚拟机 
virsh destroy 虚拟机名或虚拟机ID    #强制关闭虚拟机 
virsh start 虚拟机名或虚拟机ID      #开机虚拟机  
virsh suspend 虚拟机名或虚拟机ID    #挂起虚拟机 
virsh resume 虚拟机名或虚拟机ID     #恢复虚拟机 
virsh reset 虚拟机名或虚拟机ID      #重置虚拟机 
virsh undefine 虚拟机名或虚拟机ID   #删除虚拟机
virsh autostart 虚拟机名或虚拟机ID  #设置虚拟机自动启动
virsh autostart --disable 虚拟机名 #关闭虚拟机自动启动
virsh dumpxml 虚拟机名或虚拟机ID    #查看虚拟机配置文件 
virsh edit 虚拟机名或虚拟机ID       #修改虚拟机配置，必须关机
virsh snapshot-create-as 虚拟机名 快照名   #创建虚拟机快照
virsh snapshot-list 虚拟机名       #查看虚拟机快照列表  
virsh snapshot-revert 虚拟机名 虚拟机快照名   #恢复虚拟机快照
virt-clone -o 源虚拟机名 -n 新虚拟机名 -f 存储新虚拟机的文件路径 #克隆虚拟机

```

![](https://img2024.cnblogs.com/blog/1470032/202410/1470032-20241010192406576-1788984284.png)


### KVM存储池管理



```
virsh pool-list --all    #查看当前存储池列表 
virsh pool-info 存储池名     #查看存储池信息 
virsh  pool-dumpxml 存储池名    #查看存储池信息  注：存储池的配置信息也是xml的格式，存放在/etc/libvirt/storage中
virsh pool-destroy vmdisk   #取消激活存储池
virsh pool-undefine vmdisk    #取消定义存储池
virsh pool-delete vmdisk    #删除存储池定义的目录

#创建本地存储池,存储池所在的目录
mkdir -p /data/vmfs     # 存储池所在的目录
virsh pool-define-as vmdisk --type dir --target /data/vmfs/   #定义存储池
virsh pool-build vmdisk     #创建已定义的存储池
virsh pool-start vmdisk     #激活并启动已定义的存储池，存储池不激活是无法使用的
virsh pool-autostart vmdisk #激活并自动启动已定义的存储池，存储池不激活是无法使用的

```

### 存储卷管理



```
#创建存储卷，在vmdisk存储池中，创建一个容量为80G、格式为qcow2的虚拟机存储卷，名称为master.qcow2
virsh vol-create-as vmdisk master.qcow2 80G --format qcow2   

#删除存储卷
virsh vol-delete --pool vmdisk kvm2_2.qcow2   

```

 本博客参考[悠兔机场](https://xinnongbo.com)。转载请注明出处！
