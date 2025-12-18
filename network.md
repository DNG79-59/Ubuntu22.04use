# 静态ip配置
对我的网络来说，当前的适配器是eth0。你的系统可能与此不同  
Netplan是最新版 Ubuntu 的默认网络管理工具。Netplan 的配置文件使用 YAML 编写，扩展名为.yaml。

注意：要小心配置文件中的空格，因为它们是语法的一部分。如果缩进不对，文件将无法正常读取。

转到位于/etc/netplan 的netplan目录。  
ls进入/etc/netplan目录。

如果你没有看到任何文件，可以创建一个。文件名可以是任何名字，但按照惯例，应该以01-这样的数字开头，以.yaml 结尾。如果有多个配置文件，数字会设定优先级。

我将创建一个名为01-network-manager-all.yaml 的文件。  
让我们把这些行添加到文件中。我们将逐步创建文件。
```
network:
 version: 2
01-network-manager-all.yaml
```
Netplan 配置文件的顶层节点是一个network:映射，其中包含 version:2（表示使用网络定义版本 2）。

接下来，我们将添加一个渲染器来控制整个网络。默认情况下，渲染器为systemd-networkd，但我们将其设置为NetworkManager。

现在，我们的文件看起来是这样的：
```
network:
 version: 2
 renderer: NetworkManager
```
接下来，我们将添加ethernets，并参考之前在步骤#2 中查找的网络适配器名称。其他支持的设备类型包括 modems:、wifis: 或 bridges:。
```
network:
 version: 2
 renderer: NetworkManager
 ethernets:
   eth0:
```
由于我们设置的是静态 IP，不想为该网络适配器动态分配 IP，因此将dhcp4设置为 no。
```
network:
 version: 2
 renderer: NetworkManager
 ethernets:
   eth0:
     dhcp4: no
```

现在，我们将根据子网和可用 IP 范围指定第 2 步中提到的特定静态 IP。它是172.23.207.254。

接下来，我们要指定网关，即分配 IP 地址的路由器或网络设备。我的网关是192.168.1.1。
```
network:
 version: 2
 renderer: NetworkManager
 ethernets:
   eth0:
     dhcp4: no
     addresses: [172.23.207.254/20]
     gateway4: 192.168.1.1
```
接下来，我们将定义 nameservers。这是定义 DNS 服务器或第二个 DNS 服务器的地方。这里的第一个值是 8.8.8.8，它是 Google 的主 DNS 服务器，第二个值是8.8.8.4，它是 Google 的辅助 DNS 服务器。这些值可根据你的要求而有所不同。
```
network:
 version: 2
 renderer: NetworkManager
 ethernets:
   eth0:
     dhcp4: no
     addresses: [172.23.207.254/20]
     gateway4: 192.168.1.1
     nameservers:
         addresses: [8.8.8.8,8.8.8.4]
```
第 4 步：应用并测试更改
在永久应用更改之前，我们可以先使用该命令测试更改：

```
sudo netplan try
```
如果没有错误，它会询问你是否要应用这些设置。

最后，使用 ip a 命令测试更改，你会发现静态 IP 已被应用。
