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


# 1. 判断是否为 apt 安装（含 PPA 源）
apt是 Ubuntu 默认包管理器，软件来自系统官方源或手动添加的 PPA 源，可通过包管理记录查询：
基础查询  
执行以下命令，若能查到结果，大概率是apt安装的（apt底层依赖dpkg，所以dpkg命令也能查到这类软件）：  
```bash 方式1：apt直接查询
apt list --installed | grep 软件名
```
```bash 方式2：dpkg辅助验证
dpkg -l | grep 软件名
```
## 区分是否为 PPA 源安装
若想进一步确认是否是 PPA 源的apt安装，可查看软件的源信息：  
```bash 安装依赖工具
sudo apt install apt-show-versions
apt-show-versions 软件名
```  
输出结果中若包含ppa.launchpad.net相关路径，就说明该软件是通过 PPA 源的apt命令安装的。也可以安装新立得包管理器可视化查看，执行sudo apt install synaptic安装后，打开软件选择 “源自”，就能看到软件对应的 PPA 源。
# 2. 判断是否为 dpkg 手动安装 deb 包
dpkg -i用于安装本地下载的.deb文件，这类软件同样会被dpkg记录，和apt安装的软件在基础查询中难区分，可通过以下特征判断：  
排查是否来自系统源  
用apt policy命令查看软件是否在系统官方源中，若显示 “无可用版本” 但软件已安装，大概率是手动用dpkg装的deb包：
```bash
apt policy 软件名
```
## 从 apt policy unzip 的输出里，能精准判断它是 dpkg 安装的，而非 apt 直接安装：  
```bash
unzip:
  Installed: 6.0-9ubuntu1.5  # 你装的版本
  Candidate: 6.0-26ubuntu3.2 # 系统源里的最新版本
  Version table:
     6.0-26ubuntu3.2 500     # 官方源的版本（优先级500）
        500 http://mirrors.aliyun.com/ubuntu jammy-updates/main amd64 Packages
 *** 6.0-9ubuntu1.5 100      # 你安装的版本（优先级100）
        100 /var/lib/dpkg/status  # 关键：这个版本来自dpkg本地记录，而非系统源
```
标记 *** 的是当前安装版本，它的来源是 /var/lib/dpkg/status（dpkg 的本地状态文件），而非系统源（http://mirrors.aliyun.com）；

系统源里的版本（6.0-26ubuntu3.2）优先级更高（500），但你装的是本地 deb 包（6.0-9ubuntu1.5），所以 apt 只是 “识别到它已安装”，但并不会把它归为 “apt 从源安装”。
# 3. 判断是否为 snap 安装
snap是 Ubuntu 自带的沙箱化包管理器，安装的软件独立存储，查询命令很专属：直接执行以下命令，若列表中有目标软件，就是snap安装的：
```bash
snap list | grep 软件名
```
比如snap list | grep firefox，若有输出则说明 Firefox 是通过snap安装的（Ubuntu 部分版本默认用snap装 Firefox）。
# 4. 判断是否为源码编译安装
源码安装是下载软件源码后，通过./configure && make && sudo make install编译安装的，这类软件无包管理记录，需通过安装路径判断：  
查看常用源码安装路径  
源码编译的软件默认常安装在/usr/local/目录下，执行以下命令排查：  
```bash 查看软件可执行文件路径
which 软件名
```bash 查看软件相关文件全路径
whereis 软件名
```
若输出路径是/usr/local/bin/软件名或/usr/local/lib/软件名等，大概率是源码安装的。  
检查编译残留文件  
若当初编译时保留了源码文件夹，文件夹中会有configure、Makefile等文件，也能佐证是源码安装。  
# 5. 判断是否为二进制解压安装
部分软件提供免编译的二进制包（如.tar.gz格式），解压后直接运行，这类软件无任何包管理记录，只能通过路径和文件特征判断：  
查看自定义安装目录  
这类软件通常会被解压到/opt/、~/opt/或用户自定义文件夹，比如解压后的jdk、nodejs等，执行：
```bash
Ls/opt | grep software name
Ls~/opt | grep software name
```  
确认无包管理记录  
若apt list --installed、dpkg -l、snap list都查不到该软件，但能通过which 软件名找到可执行文件，且路径在上述自定义目录，就是二进制解压安装的。
