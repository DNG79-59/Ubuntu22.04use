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
### 查询文件的组通过`ls -al`看第二个列

# 在这里记录一个管理用户组访问文件的授权过程
### 先查询文件组，判断是否需要添加用户组，修改用户组，比如opt文件，opt-user用户
```
ls -al
```
###### 如果需要创建组修改组
```
sudo groupadd opt-gp
```
###### 修改文件的组
```
sudo chgrp opt-gp /opt
```
```
# 可能会用到修改他的访问权限比如不让其他人访问，允许组用户访问和执行
sudo chmod 750 /opt
```
### 将用户添加到组
```
sudo usermod -aG opt-gp opt-user
```
用户opt-user重新连接终端即可生效
