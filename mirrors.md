# //除了下面的方法，可以通过搜索阿里云镜像站，然后点击对应的系统，相关仓库，然后有简略的步骤！
# 当然我也做了一个shell命令
```shell
sudo bash -c '
cp /etc/apt/sources.list /etc/apt/sources.list.old
sed -i "s@http://.*archive.ubuntu.com@https://mirrors.aliyun.com/@g" /etc/apt/sources.list
sed -i "s@https://.*security.ubuntu.com@https://mirrors.aliyun.com/@g" /etc/apt/sources.list
apt-get update
'
```
```
#!/bin/bash
# 功能：Ubuntu系统apt源替换为阿里云镜像源（自动备份+完整替换+容错）
# 适用：Ubuntu 18.04/20.04/22.04 等主流版本

# 定义目标文件和镜像源地址
SOURCE_FILE="/etc/apt/sources.list"
ALIYUN_MIRROR="https://mirrors.aliyun.com/"

# 1. 提权检查（非root则自动sudo）
if [ "$(id -u)" -ne 0 ]; then
    echo "请以root权限执行，自动尝试sudo提权..."
    exec sudo bash -c "$(cat "$0")" "$@"
    exit 1
fi

# 2. 备份原有配置（若文件存在则备份，避免覆盖）
if [ -f "$SOURCE_FILE" ]; then
    cp -f "$SOURCE_FILE" "${SOURCE_FILE}.old"
    echo "✅ 已备份原有源配置至 ${SOURCE_FILE}.old"
else
    echo "⚠️  未找到 ${SOURCE_FILE}，将创建新文件"
    touch "$SOURCE_FILE"
fi

# 3. 批量替换源地址（兼容http/https开头的archive/security源）
# 替换archive.ubuntu.com（主源）
sed -i "s@http://.*archive.ubuntu.com@${ALIYUN_MIRROR}@g" "$SOURCE_FILE"
sed -i "s@https://.*archive.ubuntu.com@${ALIYUN_MIRROR}@g" "$SOURCE_FILE"

# 替换security.ubuntu.com（安全更新源）
sed -i "s@http://.*security.ubuntu.com@${ALIYUN_MIRROR}@g" "$SOURCE_FILE"
sed -i "s@https://.*security.ubuntu.com@${ALIYUN_MIRROR}@g" "$SOURCE_FILE"

echo "✅ 已将apt源替换为阿里云镜像源"

# 4. 刷新源缓存（带错误提示）
echo "🔄 正在刷新apt源缓存..."
if apt-get update; then
    echo "🎉 源替换并刷新完成！"
else
    echo "❌ 源缓存刷新失败，请检查源配置是否正确！"
    exit 1
fi
```
## 1.备份源文件
通过以下命令对官方源的配置文件进行备份  
```ssh
sudo cp /etc/apt/sources.list /etc/apt/sources.list.old
```  
## 2.修改源
```ssh
sudo vim /etc/apt/sources.list
```  
如果没有默认安装vim，则可以将上述命令中vim替换为vi

## 3.修改源配置文件内容
按两下d键，表示删除，按住d将文件内容全部删除后，按i键进入编辑模式，复制下面的源网址到配置文件中：
```ssh
deb http://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-backports main restricted universe multiverse
```  
#### *注意：本人使用ubuntu系统为Ubuntu 22.04，如果你是用的是别的版本，则上述内容应该使用对应版本的源网址！
查询系统版本的方法：
### 输入lsb_release -a，Release后面的代表版本，Codename后面的代表代号。
<img width="347" height="101" alt="image" src="https://github.com/user-attachments/assets/d2220397-c47a-4108-aa80-b435dfefe57a" />

一般可以直接将你的Codename后面的内容替换上文网址中的 jammy ，比如你的Codename为Orio，则可以直接将上文所有的jammy替换为Orio即可。

## 4.保存文件
按一下Esc键，然后输入:wq(注意wq前面有个冒号！)，即可保存。如果弄错了，则可以输入:q!不保存直接退出。

## 5.更新软件列表
换源后必须要更新一下软件列表，让你的系统知道现在的源里面有什么软件，指令如下：
```ssh
sudo apt-get update
```  
更换其他的源例如清华源、网易源等步骤一致，区别仅在于sources.list文件中网址不同。

版权声明：本文为CSDN博主「Zexora」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
[原文链接：](https://blog.csdn.net/weixin_43996864/article/details/136823530)
