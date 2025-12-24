# Ubuntu22.04use
记录使用Ubuntu2204系统的过程，用于记录
## 本次使用的是公司的电脑安装的vm虚拟机，连接在自己笔记本上使用，其中公司电脑与笔记本是同一网段，vm通过nat模式连接Ubuntu虚拟机，在虚拟网络编辑器VMnet8上配置nat设置的端口转发，完成配置，实现连接宿主机内的虚拟机。
<img width="1235" height="1123" alt="image" src="https://github.com/user-attachments/assets/2ca94398-3e58-43f1-a7ea-268d5af0f634" />

# 记录一个Centos7minimal使用过程，因为他停止维护了
首先需要配置ip
`vi /etc/sysconfig/network-scripts/ifcfg-ens33`
```
TYPE=Ethernet
BOOTPROTO=static
IPADDR=192.168.30.10
NETMASK=255.255.255.0
GATEWAY=192.168.30.2
DNS1=202.38.93.153
DNS2=223.6.6.6
ONBOOT=yes
NAME=ens33
DEVICE=ens33
NM_CONTROLLED=no
```
## 重启网络
`systemctl restart network`

# 接来下配置他的镜像源[原文地址](https://www.cnblogs.com/lvzhenjiang/articles/18350828)
### 确定/etc/yum.repos.d/目录中的源配置没用，也可以将/etc/yum.repos.d/目录清空
`rm -f /etc/yum.repos.d/*`
## 配置Centos7源（x86_64架构）
`curl -o /etc/yum.repos.d/Centos7-aliyun.repo https://mirrors.wlnmp.com/centos/Centos7-aliyun-x86_64.repo`  
`curl -o /etc/yum.repos.d/Centos7-ustc.repo https://mirrors.wlnmp.com/centos/Centos7-ustc-x86_64.repo`
# 刷新缓存
`yum clean all && yum makecache fast`  
## 接来下安装ssh
`yum install openssh-server -y`
## 进入编辑可以改端口和ssh版本
`vi /ect/ssh/sshd_config`
```
# 将ssh服务添加到自启动列表中：
systemctl enable sshd.service
```
首先，我们查看ssh所用的端口是否已经打开，这里我已22端口为例：  
```
firewall-cmd --state //查看防火墙是否打开，打开为running
firewall-cmd --zone=public --query-port=22/tcp //端口如果开放则为yes,否则为no
```
如果22端口没有开放，我们可以用以下两种方法解决：
（1）、关闭防火墙：
`/bin/systemctl stop firewalld.service`

```
firewall-cmd --add-port=22/tcp --permanent   //添加（--permanent永久生效，没有此参数重启后失效）
firewall-cmd --reload //添加完成后要重新载入
firewall-cmd --zone= public --query-port=22/tcp //查看端口是否开启
```
