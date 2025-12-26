# 记录一下使用对于用户的管理，和组管理
## 创建用户-配置密码-添加到组-删除用户-查询所有用户等
```
sudo adduser test01
```
```
sudo paddwd test01
```
```
### 以添加到root组为例
sudo usermod -aG root test01
```
添加到组需要重新连接终端生效  
### 删除用户test01
```
sudo deluser test01
```
### 这里我们传入标准的 UID 范围，通常普通用户的 UID 从 1000 开始：
```
getent passwd | awk -F: '$3 >= 1000 {print $1}'
```

## 创建组-查询组下成员

### 以创建test组为例
```
sudo groupadd test
```
### 查询test组下成员
```
getent group test
```
### 查询test01用户所属的组
```
groups test01
```
### 删除组 
```
sudo groupdel groupname
```
### 查询文件的组通过`ls -al`看第二个列

# 在这里记录一个管理用户组访问文件的授权过程
### 先查询文件的组，判断是否需要添加用户组，修改文件的用户组，比如opt文件，opt-user用户
```
ls -al
```
<img width="928" height="940" alt="image" src="https://github.com/user-attachments/assets/153493bf-01a9-4487-a249-28497cc3c3f5" />

###### 如果需要创建组修改组
```
sudo groupadd opt-gp
```
###### 修改文件的组
```
sudo chgrp -R opt-gp /opt
```
<img width="956" height="940" alt="image" src="https://github.com/user-attachments/assets/cdfcb352-20b4-44a0-ab8d-034916f329e7" />

```
# 可能会用到修改他的访问权限比如不让其他人访问，允许组用户访问和执行
sudo chmod -R 750 /opt
```
<img width="538" height="94" alt="image" src="https://github.com/user-attachments/assets/f35d5f07-c79a-4825-8332-825f42fdfe7f" />

### 将用户添加到组
```
sudo usermod -aG opt-gp opt-user
```
用户opt-user重新连接终端即可生效
<img width="818" height="216" alt="image" src="https://github.com/user-attachments/assets/d77cb599-c496-4942-a717-47bb65be0b52" />



# 同时记录一下更精细的权限配置ACL
### 单条规则，如配置某个人opt-user对opt目录下的rx权限，这样的规则可以使用于多种情况，比如a、b需要读取和执行，d、c需要读写执行，该文件的拥有者又是root等其他人
在使用高级acl前，需要将传统的控制改一下  
将属主设为 root，权限设为仅 root 可读写，清空 group 和 others 的权限
```
sudo chown root:root deploy.sh
sudo chmod 600 deploy.sh
```
参数解释，-R 代表递归，子目录下所有都符合这个要求，-d是该目录下新文件的规则继承，-m是修改acl
```
sudo setfacl -R -d -m u:opt-user:rx /opt
```
### 也有适用于多个组的形式的acl控制，类似于传统的linux系统的三位中的组
创建acl组
```
sudo groupadd opt-gp
```
对acl组授权，同user授权
```
sudo setfacl -R -d -m g:opt-gp:rx /opt
```
添加acl组成员
```
sudo usermod -aG opt-gp opt-user
```
### 删除acl规则
删除用户规则
```
sudo setfacl -R -x u:opt-user /opt
```
删除组的就是p:opt-gp /opt
### 查看当前acl配置
```
sudo getfacl /opt
```
<img width="774" height="444" alt="image" src="https://github.com/user-attachments/assets/dba96a54-dde3-4f48-859e-43d1b951aa4b" />

### 新文件一般是跟着上级目录的acl、如果单独设置了新文件的acl，可以使用setfacl -b删除新文件所有的acl，再对他的父级做acl，以便继承，或者再单独设置acl
使用 setfacl 命令删除目录的默认权限：
```
sudo setfacl -b /path/to/directory
```
这个命令将移除目录的所有 ACL 规则，包括默认设置。

# 还有另一种添加权限的方式
使用sudoer文件，在文件中添加内容，比如opt-user对文件opt的ls命令
```
sudo vi /etc/sudoers # 可以使用vi编辑器G直接跳到尾行
# 在尾行添加内容
opt-user ALL=(ALL) /usr/bin/ls /opt
```
添加后opt-user使用`sudo ls /opt`即可
