# cSphere 部署手册 
******************************************************************

 ### 1 准备工作
 
 #### 1.1 下载和安装[CentOS 7 64bit](http://mirrors.aliyun.com/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1611.iso) 

下载[CentOS 64bit Minimal镜像](http://mirrors.aliyun.com/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1611.iso)  安装时选用默认的 __英文__ 语言进行安装
 * #### 1.2 操作系统基本设置
 
      * ##### 1.2.1 设置主机名
      建议根据主机实际角色来对主机命名，如：controller、node1等
       如：``` echo "controller" > /etc/hostname```
      * ##### 1.2.2 关闭SElinux
      
        ```setenforce 0 ``` 
        ``` sed -i '/^SELINUX=/cSELINUX=disabled' /etc/selinux/config ```
      
   * ##### 1.2.3 关闭firewall
      ``` systemctl disable firewalld.service ```
      ``` systemctl stop firewalld.service ```
  * #### 1.3 安装相关依赖包和4.6版本内核
    
       * ##### 1.3.1 安装cSphere yum源
           ``` curl -Ss http://52.68.20.57/pubrepo/centos/7/x86_64/csphere.repo > /etc/yum.repos.d/csphere.repo ```
           ``` yum repolist csphere ```
           
         *注：如果在安装时，所在服务器无法连接互联网，需要提前将cSphere repo目录下载至本地，然后上传至服务器,如：* 
   ``` wget -r -np -nH -R "index.html*" http://52.68.20.57/pubrepo/centos/7/x86_64/ ```
               在服务器上创建一个本地的repo配置, 使用下载下来的cSphere repo目  录，如：   假设repo目录的路径为:  /root/pubrepo/，那么本地repo文件应该这么创建
             ``` cat<<-EOS>/etc/yum.repos.d/csphere.repo 
              [csphere]
              name=csphere local repo
              baseurl=file:///root/pubrepo/centos/7/x86_64/
              gpgcheck=0
              enabled=1
              EOS
              ```
     * ##### 1.3.2 安装依赖包
       ``` yum -y --disablerepo='*' --enablerepo=csphere install bridge-utils net-tools psmisc subversion git fuse ntp rng-tools bash-completion ```
    * ##### 1.3.3 安装4.6版本内核
         ``` yum -y --disablerepo='*' --enablerepo=csphere install kernel-ml-4.6.0 iproute ```
         
     * ##### 1.3.4 启用4.6版本内核
          ``` grub2-set-default 0 ```
          ``` reboot ```
          ``` uname -r ```
      ![enter description here][1]
         


  [1]: ./images/4.6%E5%86%85%E6%A0%B8.png "4.6内核.png"
  
               