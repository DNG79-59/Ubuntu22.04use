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

