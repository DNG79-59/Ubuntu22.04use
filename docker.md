# 下面是一键安装命令
```shell
sudo bash -c '
# ==============================================
# 功能：Ubuntu系统通过中科大镜像安装Docker CE
# 适配：Ubuntu 18.04/20.04/22.04（x86_64/arm64）
# ==============================================

# 1. 更新系统包索引（加-y避免交互）
apt-get update -y

# 2. 安装Docker依赖的基础工具
apt-get install -y apt-transport-https ca-certificates curl software-properties-common lsb-release

# 3. 添加Docker官方GPG密钥（避免包校验失败）
# 中科大镜像同步了官方GPG密钥，也可直接用官方地址
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 4. 配置中科大Docker软件源（替换阿里云）
# 新版Ubuntu推荐的源格式（兼容安全策略）
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5. 再次更新源索引（加载新配置的中科大源）
apt-get update -y

# 6. 安装Docker全家桶（CE版+容器运行时+插件）
apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 7. 启动Docker并设置开机自启
systemctl start docker
systemctl enable docker

# 8. 验证安装结果
if docker --version; then
    echo -e "\033[32m✅ Docker通过中科大镜像安装成功！\033[0m"
    # 可选：测试Docker基础功能（需联网，可注释）
    # docker run --rm hello-world
else
    echo -e "\033[31m❌ Docker安装失败，请检查网络/源配置！\033[0m"
    exit 1
fi
'
```

# 在配置软件源时使用的中科大，docker时使用了阿里云导致没法安装，下面是一次性解决 Docker 安装时的源配置、GPG 密钥、残留配置等高频问题
```shell
sudo bash -c '
set -e
# ===================== 配置项（可修改） =====================
MIRROR_TYPE="ustc"  # ustc=中科大，tsinghua=清华
DOCKER_VERSION="latest"
# ===================== 核心函数 =====================
red_echo() { echo -e "\033[31m$1\033[0m"; }
green_echo() { echo -e "\033[32m$1\033[0m"; }
yellow_echo() { echo -e "\033[33m$1\033[0m"; }

# 1. 定义镜像源
set_mirror() {
    if [ "$MIRROR_TYPE" = "ustc" ]; then
        GPG_URL="https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg"
        DOCKER_SOURCE="https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu"
        REGISTRY_MIRROR="https://docker.mirrors.ustc.edu.cn"
    elif [ "$MIRROR_TYPE" = "tsinghua" ]; then
        GPG_URL="https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu/gpg"
        DOCKER_SOURCE="https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu"
        REGISTRY_MIRROR="https://docker.mirrors.tuna.tsinghua.edu.cn"
    else
        red_echo "❌ 仅支持 ustc/tsinghua"
        exit 1
    fi
}

# 2. 清理残留配置
clean_docker_config() {
    green_echo "🔧 清理Docker残留配置..."
    rm -rf /etc/apt/sources.list.d/*docker* 2>/dev/null
    sed -i '/mirrors.aliyun.com\/docker-ce/d' /etc/apt/sources.list 2>/dev/null
    sed -i '/mirrors.cloud.aliyuncs.com\/docker-ce/d' /etc/apt/sources.list 2>/dev/null
    apt-key del 0EBFCD88 2>/dev/null || true
    green_echo "✅ 残留配置清理完成"
}

# 3. 校验镜像源连通性
check_mirror_connect() {
    green_echo "🔧 校验${MIRROR_TYPE}镜像源连通性..."
    if ! curl -fsSL --connect-timeout 5 "$GPG_URL" >/dev/null; then
        red_echo "❌ ${MIRROR_TYPE}镜像源无法访问"
        exit 1
    fi
    green_echo "✅ 镜像源连通性正常"
}

# 4. 配置Docker源+安装依赖
config_docker_source() {
    green_echo "🔧 配置Docker源..."
    apt-get update -y
    apt-get install -y apt-transport-https ca-certificates curl software-properties-common lsb-release dpkg
    curl -fsSL "$GPG_URL" | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    OS_CODENAME=$(lsb_release -cs)
    ARCH=$(dpkg --print-architecture)
    echo "deb [arch=${ARCH} signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] ${DOCKER_SOURCE} ${OS_CODENAME} stable" | tee /etc/apt/sources.list.d/docker.list >/dev/null
    green_echo "✅ Docker源配置完成"
}

# 5. 安装Docker
install_docker() {
    green_echo "🔧 安装Docker..."
    apt-get update -y
    if [ "$DOCKER_VERSION" = "latest" ]; then
        apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    else
        apt-get install -y docker-ce=${DOCKER_VERSION} docker-ce-cli=${DOCKER_VERSION} containerd.io docker-buildx-plugin docker-compose-plugin
    fi
    systemctl start docker || red_echo "⚠️ Docker启动失败，尝试重启..."
    systemctl enable docker >/dev/null 2>&1
    # 配置镜像加速
    mkdir -p /etc/docker
    cat > /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["${REGISTRY_MIRROR}"]
}
EOF
    systemctl daemon-reload
    systemctl restart docker
    if docker --version >/dev/null 2>&1; then
        green_echo "✅ Docker安装成功！版本：$(docker --version | awk '{print $3}' | sed 's/,//')"
    else
        red_echo "❌ Docker安装失败"
        exit 1
    fi
}

# ===================== 主流程 =====================
clear
green_echo "========================================"
green_echo "  Docker源修复&安装脚本（${MIRROR_TYPE}镜像）"
green_echo "========================================"
set_mirror
clean_docker_config
check_mirror_connect
config_docker_source
install_docker
green_echo "🎉 所有操作完成！测试命令：docker run --rm hello-world"
'
```
# 登录容器[镜像服务控制台](https://cr.console.aliyun.com/?spm=a2c4g.11186623.0.0.6bd92071fTW9er)，在左侧导航栏选择镜像工具 > 镜像加速器，在镜像加速器页面获取加速器地址。
```shell
sudo bash -c '
cat > /etc/docker/daemon.json << EOF
{
    "registry-mirrors": ["https://7o3mmczj.mirror.aliyuncs.com"]
}      
EOF
systemctl restart docker
'
```
## 当然一般无法使用，这个需要自己在控制台添加镜像（暂时没有研究这个），下面是另一种方法
sudo bash -c '
# 定义要测试的镜像源列表 可以自己添加或修改
MIRROR_LIST=(
  "https://docker.xuanyuan.me"
  "https://docker.1ms.run"
  "https://docker.m.daocloud.io"
  "https://docker.hlmirror.com"
  "https://dockerpull.pw"
)

# 临时文件存储可用地址
AVAILABLE_MIRRORS=$(mktemp)
echo -n "[" > $AVAILABLE_MIRRORS

# 批量测试每个镜像源
green_echo() { echo -e "\033[32m$1\033[0m"; }
red_echo() { echo -e "\033[31m$1\033[0m"; }
yellow_echo() { echo -e "\033[33m$1\033[0m"; }

green_echo "========================================"
green_echo "🔍 开始批量测试镜像源..."
FIRST_OK=1
for url in "${MIRROR_LIST[@]}"; do
  echo -n "测试 $url ... "
  # 临时替换配置并测试
  TEMP_CONFIG=$(mktemp)
  echo "{\"registry-mirrors\":[\"$url\"]}" > $TEMP_CONFIG
  sudo cp $TEMP_CONFIG /etc/docker/daemon.json
  sudo systemctl restart docker >/dev/null 2>&1
  
  # 测试拉取hello-world
  if docker pull hello-world >/dev/null 2>&1; then
    green_echo "OK"
    # 收集可用地址（处理逗号分隔）
    if [ $FIRST_OK -eq 1 ]; then
      echo -n "\"$url\"" >> $AVAILABLE_MIRRORS
      FIRST_OK=0
    else
      echo -n ",\"$url\"" >> $AVAILABLE_MIRRORS
    fi
  else
    red_echo "FAIL"
  fi
  rm -f $TEMP_CONFIG
done

# 完成JSON格式
echo "]" >> $AVAILABLE_MIRRORS
AVAILABLE_CONTENT=$(cat $AVAILABLE_MIRRORS)
rm -f $AVAILABLE_MIRRORS

# 处理无可用地址的情况
if [ "$AVAILABLE_CONTENT" = "[]" ]; then
  red_echo "========================================"
  red_echo "❌ 所有镜像源测试失败！请检查网络或更换镜像源列表"
  # 恢复默认配置（清空registry-mirrors）
  echo "{}" | sudo tee /etc/docker/daemon.json >/dev/null
  sudo systemctl restart docker
  exit 1
fi

# 写入可用地址到daemon.json（标准化JSON）
green_echo "========================================"
green_echo "✅ 测试完成，可用镜像源：$AVAILABLE_CONTENT"
echo "{\"registry-mirrors\":$AVAILABLE_CONTENT}" | sudo tee /etc/docker/daemon.json >/dev/null

# 重启Docker并验证
sudo systemctl daemon-reload
sudo systemctl restart docker
green_echo "========================================"
green_echo "📌 最终配置已生效，当前镜像源："
docker info | grep -A 2 "Registry Mirrors" | grep -v "Registry Mirrors" | tr -d ' \t'
'
# Docker配置非root用户权限
默认情况下，Docker命令需要root权限（即使用sudo）。为了避免每次都输入sudo并遵循最小权限原则，应将当前用户添加到docker用户组。

```
# 将当前用户添加到docker组。
sudo usermod -aG docker $USER
# 方式1：退出当前终端，重新登录（最稳妥）
exit  # 退出后重新ssh/打开终端

# 方式2：刷新当前会话的用户组（临时生效，仅当前终端）
newgrp docker
```
执行newgrp docker命令生效后。直接使用docker命令，无需添加sudo。
