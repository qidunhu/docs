# cSphere 部署手册 
******************************************************************

### 1 准备工作
 
#### 1.1 下载和安装CentOS 7 64bit

下载__[CentOS 64bit Minimal镜像](http://mirrors.aliyun.com/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1611.iso)__  安装时选用默认的 __英文__ 语言进行安装
#### 1.2 操作系统基本设置
 
##### 1.2.1 设置主机名
  建议根据主机实际角色来对主机命名，如：controller、node1等
  如：``` echo "controller" > /etc/hostname```
##### 1.2.2 关闭SElinux
    
  ```setenforce 0 ``` 
  ``` sed -i '/^SELINUX=/cSELINUX=disabled' /etc/selinux/config ```
      
##### 1.2.3 关闭firewall
``` systemctl disable firewalld.service ```
``` systemctl stop firewalld.service ```
#### 1.3 安装相关依赖包和4.6版本内核
    
##### 1.3.1 安装cSphere yum源
``` curl -Ss http://52.68.20.57/pubrepo/centos/7/x86_64/csphere.repo > /etc/yum.repos.d/csphere.repo ```

``` yum repolist csphere ```
           
注：如果在安装时，所在服务器无法连接互联网，需要提前将cSphere repo目录下载至本地，然后上传至服务器,如： 
``` wget -r -np -nH -R "index.html*" http://52.68.20.57/pubrepo/centos/7/x86_64/ ```

在服务器上创建一个本地的repo配置, 使用下载下来的cSphere repo目录，如：假设repo上传到服务器目录的路径为: /root/pubrepo/，那么服务器本地repo文件应该这么创建:

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
###1.4 Docker数据分区
##### 1.4.1 创建Docker分区
Docker需要一个 **单独的数据分区** 来存放Docker数据，并使用参数```-n ftype=1```格式化为xfs文件系统, 否则在4.6内核上无法正常创建容器。
该分区还要使用参数```prjquota```挂载， 否则无法正常使用容器的磁盘空间配额功能。假设分区设备名为/dev/sdb1, 操作如下:
`mkfs.xfs -n ftype=1 /dev/sdb1`

![xfs](http://git.oschina.net/uploads/images/2017/0322/174445_297e796d_934281.png "xfs")