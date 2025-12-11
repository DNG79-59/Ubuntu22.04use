## 软件包分源码包和二进制包
软件包一般为.rpm 源码包一般为.tar.gz  
源码包需要配置、编译、安装。但可定制性好。  
| 系统平台 |包类型 | 工具|在线安装|
| ----------- | ----- | ------ | ------ |
|RedHat、CentOS、Fedora、SUSE|rpm|rpm、rpmbuild|yum|
|Ubuntu、Debian|deb|dpkg|apt|

rpm需要安装，后使用rpm安装软件  
在线安装自动更新，可以自动安装所需依赖  
# 一、 查看已安装程序的常用命令
## 1. 最常用：apt list --installed（查所有 apt 安装的软件）  
这个命令专门列出通过 apt/apt-get 安装的包，输出清晰，带版本号：
```bash 查看所有已安装包
apt list --installed
```  
```bash 过滤特定包（比如查 nginx 是否安装）
apt list --installed | grep nginx
```  
## 2. 传统命令：dpkg -l（查所有 dpkg 管理的包，包括手动装的 deb 包）  
dpkg 是底层包管理器，不管是 apt 装的还是手动装的 deb 包，都能查到：  
```bash 看所有已安装包（输出格式：状态+包名+版本+描述）
dpkg -l
```  
```bash 过滤特定包（比如查 unzip）
dpkg -l | grep unzip
```  
## 3. 查单个包的详细信息：apt show 包名
如果想知道某个已安装包的用途、依赖、安装时间，用这个：  
```bash 比如查 nginx 的详细信息
apt show nginx
```
## 4. 查软件安装路径：which 命令名
适合查可执行程序的位置（比如知道 python3 装在哪）：
```bash 比如查 docker 命令的路径
which docker
```
## 查更详细的路径（包括软链接）
```
whereis docker
```  
# 二、 卸载软件的常用命令
## 1. 普通卸载（保留配置文件）：sudo apt remove 包名
适合临时卸载，以后重装还能复用配置，比如：
```bash 卸载 nginx，保留配置文件
sudo apt remove nginx
```
## 2. 彻底卸载（删除配置文件）：sudo apt purge 包名
运维常用！彻底清除软件的所有文件（包括 /etc 下的配置），比如：
```bash
sudo apt purge nginx
```
## 3. 清理残留依赖：sudo apt autoremove
卸载软件后，会留下一些 “没用的依赖包”，用这个命令一键清理：
```bash 清理系统中所有未被使用的依赖
sudo apt autoremove
```  
## 4. 手动卸载 deb 包：sudo dpkg -r 包名
如果是手动下载 deb 包安装的，用 dpkg 卸载：
```bash 普通卸载（保留配置）
sudo dpkg -r unzip
```  
# 彻底卸载（删除配置）
`sudo dpkg -P unzip`
# 三、 高频组合用法（直接抄）
查是否安装 → 确认后卸载 → 清理残留
```bash 1. 查 nginx 是否安装
apt list --installed | grep nginx
# 2. 彻底卸载
sudo apt purge nginx
# 3. 清理依赖
sudo apt autoremove
```  
## 卸载后想重装（保留配置）
```bash
sudo apt remove nginx && sudo apt install nginx
```
## 卸载后想彻底重装（删配置）
```bash
sudo apt purge nginx && sudo apt autoremove && sudo apt install nginx
```  
小提示
卸载软件时一定要加 sudo，否则会提示权限不足；
不确定包名时，先用 grep 过滤，避免输错包名卸载错软件。
