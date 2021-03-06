title: linux基本环境配置
date: 2019-11-21 10:22:26
categories: 环境部署
tags: 
	- linux
---
##### 基本环境配置
在所有服务器上面进行，修改系统语言为英文，退出重新登录即可生效
```
localectl set-locale LANG=en_US.UTF-8
```
所有服务器配置使用国内阿里云的源
```
cd /etc/yum.repos.d/
mkdir repo_bak
mv *.repo repo_bak/
wget http://mirrors.aliyun.com/repo/Centos-7.repo
wget http://mirrors.aliyun.com/repo/epel-7.repo
```

<!-- more -->

所有服务器的操作系统进行升级
```
yum clean all
yum makecache
yum repolist
yum update -y
```
所有服务器校准系统时间
```
yum -y install ntp
systemctl enable ntpd.service
ntpdate 0.centos.pool.ntp.org
systemctl start ntpd.service
date
```
永久关闭所有服务器上面的 selinux 和 NetworkManager 这两个服务
```
systemctl stop NetworkManager.service
systemctl disable NetworkManager.service
```
```
setenforce 0
vim /etc/selinux/config
SELINUX=disabled
SELINUXTYPE=targeted
```
永久关闭所有服务器上的 firewalld，启用 network
```
systemctl disable firewalld
systemctl stop firewalld
systemctl enable network
systemctl start network
```
修改所有服务器的主机名，如果需要的话
```
hostnamectl set-hostname <you.com>
```
在所有服务器上面修改 /etc/hostname 和 /etc/hosts 两个文件，如下：（改完，logout，重新登录一次，就可以看到已经生效）
```
vim /etc/hostname
you.com
```
```
vim /etc/hosts
host_ip_addr you.com you
```
##### 修改IP
查看网卡信息:
```
ifconfig -a
```
修改ip，eth0是网卡名，我的是ifcfg-enp0s3
```
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
```
重启网络服务
```
service network restart
```
##### 在Linux上安装lrzsz
```
yum -y install lrzsz
```
* sz中的s意为send（发送），告诉客户端，我（服务器）要发送文件 send to cilent，就等同于客户端在下载。
* rz中的r意为received（接收），告诉客户端，我（服务器）要接收文件 received by cilent，就等同于客户端在上传。
用lszrz非常方便，但是有一点不足之处： 无法传输大于 4G 的文件。
##### 安装命令自动补全
因为Centos7的默认安装类型是最小安装，所以默认安装没有自动补全的功能。要已用这个功能，需要安装一个bash-completion包，然后退出bash，重新登录即可。
```
yum install bash-completion -y
```
##### CentOS 7快速开放端口
CentOS升级到7之后，发现无法使用iptables控制Linuxs的端口，baidu之后发现Centos 7使用firewalld代替了原来的iptables。下面记录如何使用firewalld开放Linux端口：

开启端口
```
[root@centos7 ~]# firewall-cmd --zone=public --add-port=80/tcp --permanent
```

查询端口号80 是否开启：
```
[root@centos7 ~]# firewall-cmd --query-port=80/tcp
```
重启防火墙：
```
[root@centos7 ~]# firewall-cmd --reload
```
查询有哪些端口是开启的:
```
[root@centos7 ~]# firewall-cmd --list-port
```
命令含义：

*  --zone #作用域
*  --add-port=80/tcp #添加端口，格式为：端口/通讯协议
*  --permanent #永久生效，没有此参数重启后失效

关闭firewall：
```
#停止
systemctl stop firewalld.service 
#禁止firewall开机启动
firewallsystemctl disable firewalld.service 
```