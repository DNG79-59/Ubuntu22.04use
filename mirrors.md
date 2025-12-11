# //除了下面的方法，可以通过搜索阿里云镜像站，然后点击对应的系统，相关仓库，然后有简略的步骤！
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
