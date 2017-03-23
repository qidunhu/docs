# 希云cSphere 部署手册 



### 1 准备工作
 
#### 1.1 下载和安装 CentOS 7 64bit

下载  __[CentOS 64bit Minimal镜像](http://mirrors.aliyun.com/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1611.iso)__   安装时选用默认的 __英文__ 语言进行安装
#### 1.2 操作系统基本设置
 
##### 1.2.1 设置主机名
建议根据主机实际角色来对主机命名，例如： _controller、node1_ 等，

`echo "controller" > /etc/hostname`
##### 1.2.2 关闭SElinux
    
`setenforce 0`

`sed -i '/^SELINUX=/cSELINUX=disabled' /etc/selinux/config`
      
##### 1.2.3 关闭firewall
`systemctl disable firewalld.service`

`systemctl stop firewalld.service`
#### 1.3 安装相关依赖包和4.6版本内核
    
##### 1.3.1 安装cSphere yum源
``` curl -Ss http://52.68.20.57/pubrepo/centos/7/x86_64/csphere.repo > /etc/yum.repos.d/csphere.repo ```

``` yum repolist csphere ```
           
>注：如果在安装时，所在服务器无法连接互联网，需要提前将cSphere repo目录下载至本地，然后上传至服务器,如： 
>`wget -r -np -nH -R "index.html*" http://52.68.20.57/pubrepo/centos/7/x86_64/`
>
>在服务器上创建一个本地的repo配置文件, 使用下载下来的cSphere repo目录，例如：将提前下载的repo目录上传到服务器的路径为: `/root/pubrepo/`，那么服务器本地repo配置文件应该这么创建:

```bash 
    cat<<-EOS>/etc/yum.repos.d/csphere.repo 
              [csphere]
              name=csphere local repo
              baseurl=file:///root/pubrepo/centos/7/x86_64/
              gpgcheck=0
              enabled=1
              EOS
```
##### 1.3.2 安装依赖包
`yum -y --disablerepo='*' --enablerepo=csphere install bridge-utils net-tools psmisc subversion git fuse ntp rng-tools bash-completion`

##### 1.3.3 安装4.6版本内核
`yum -y --disablerepo='*' --enablerepo=csphere install kernel-ml-4.6.0 iproute`
         
##### 1.3.4 启用4.6版本内核

`grub2-set-default 0`

`reboot`

`uname -r`

![4.6内核](http://git.oschina.net/uploads/images/2017/0322/171435_523fbbcd_934281.png "4.6内核")

### 1.4 Docker数据分区

##### 1.4.1 创建Docker分区

Docker需要一个**单独的数据分区**来存放Docker数据，需要新增一个硬盘或者单独划一个数据分区给Docker使用，并使用参数`-n ftype=1`格式化为xfs文件系统，否则在4.6内核上无法正常创建容器。该分区还要使用`-o prjquota`参数挂载， 否则无法正常使用容器的磁盘空间配额功能。假设分区设备名为/dev/sdb1, 操作如下:

`mkfs.xfs -n ftype=1 /dev/sdb1`

_如图_ 

![xfs](http://git.oschina.net/uploads/images/2017/0322/174445_297e796d_934281.png "xfs")
##### 1.4.2 挂载文件系统
创建挂载点，用来挂载新创建的xfs文件系统，操作如下：

`mkdir /docker_data`

使用`-o prjquota`参数和UUID来挂载该文件系统，首先使用`blkid`命令获取到对应的分区的UUID，然后再挂载该分区，如图：

![挂载](http://git.oschina.net/uploads/images/2017/0323/155220_9cc09f9c_934281.jpeg "挂载")

>使用UUID方式挂载，主要是防止设备名变化导致文件系统无法挂载，尤其是在云环境下部署时。

再将该挂载点写入/etc/fstba文件当中，实现开机启动，如图：

`echo UUID=3de7c0ff-bdde-4a95-8727-b52246c328b9 /docker_data xfs defaults,prjquota 0 0 >> /etc/fstab`

`ln -sv /docker_data/docker /var/lib/docker`

### 2 安装希云cSphere Controller
#### 2.1 安装rpm包
将csphere-controller-x.x.x-rhel7.x86_64.rpm 安装包上传至服务器，执行安装命令

`rpm -vih csphere-controller-x.x.x-rhel7.x86_64.rpm`

#### 初始化controller
设置集群节点数量和端口

`Role=controller ClusterSize=3 Port=80 MongoRepl=NO csphere_init`

> _参数说明_ ：
*_Port: 管理节点控制台HTTP服务端口_ 
*_ClusterSize： Etcd集群节点数量, 此处设置的值就是Agent最小的安装数目,如ClusterSize=3,则最少需要安装3台Agent来完成Etcd集群初始化_ 

#### 2.2 启动controller

`cspherectl  start`

### 3 安装希云cSphere Agent
> _安装controller时，设置的 `ClusterSize` 值是多少，则首次部署Agent时就至少需要安装
`ClusterSize`台Agent_ 

#### 3.1 设置网络
##### 3.1.1 新建br网桥

> _如果容器使用bridge网络模式，则此步骤为必须，ipvlan网络可跳过该步骤,如下：_ 


`cd /etc/sysconfig/network-scripts`
```bash
cat <<-EOS>ifcfg-br0 
DEVICE=br0
TYPE=Bridge
ONBOOT=yes
IPADDR=192.168.2.18      #IP地址,根据实际填写
NETMASK=255.255.255.0    #掩码，根据实际填写
GATEWAY=192.168.2.1      #默认网关，根据实际填写
DNS1=192.168.2.1         #主DNS，根据实际填写
BOOTPROTO=static
NM_CONTROLLED=no
EOS
```
##### 3.1.2 绑定br0网卡
> _假如物理网卡名称为ens33_ 

```bash
cat <<-EOS>ifcfg-ens33
TYPE=Ethernet
DEVICE=ens33            #物理网卡名称，根据实际填写
NAME=ens33              #物理网卡名称，根据实际填写
ONBOOT=yes
BRIDGE=br0              #上个步骤新建的br0网桥
NM_CONTROLLED=no
EOS
```
重启系统，使设置生效

`reboot`

重新登陆服务器，使用如下命令查看配置是否生效

```bash
ifconfig br0           # 确认IP地址在br0上
brctl show br0         # 物理网卡被连接到br0
```
 _如图_ 

![查看网桥设置是否成功](http://git.oschina.net/uploads/images/2017/0323/120825_d3a9746c_934281.jpeg "网桥")


##### 3.2 安装cSphere Agent
将csphere-agent-x.x.x-rhel7.x86_64.rpm 安装包上传至服务器，执行安装命令

`rpm -ivh csphere-agent-x.x.x-rhel7.x86_64.rpm`

初始化Agent
> _说明：可以使用ipvlan或者bridge网络模式初始化，请根据实际网络模式进行操作_ 

 _bridge模式初始化_ 
```bash
Role=agent ControllerAddr=192.168.2.1:80 InstCode=6906 NetMode=bridge csphere_init 
```
 _ipvlan模式初始化_ 
```bash
Role=agent ControllerAddr=192.168.2.1:80 InstCode=6906 NetMode=ipvlan InetDev=eth0 csphere_init
```
>参数解释：
* `ControllerAddr` 控制节点的地址:端口
* `InstCode` 安装码，需要在控制节点的控制台页面生成
* `NetMode` Docker容器网络模式，ipvlan或bridge，根据实际情况填写
* `InetDev` 物理网卡名字，只有在ipvlan模式下，才会使用到该参数

启动Agent

`cspherectl  start`

> _如果 `NetMode=ipvlan` 的话，**docker**会启动失败, 继续执行如下命令_

`net-plugin ip-range  --ip-start=172.17.0.1/24 --ip-end=172.17.0.254/24`

稍等片刻，就会在控制节点页面上看到节点加入
##### 3.3 设置容器使用的IP地址池
> _如不设置，容器将无法正常启动，根据实际情况设置地址池，在任意一台agent节点执行如下命令_ 

`net-plugin ip-range  --ip-start=192.168.2.20/24 --ip-end=192.168.2.200/24`

