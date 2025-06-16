## Docker
### 安装Docker
```shell
# 执行以下命令，添加 Docker 软件源。
dnf config-manager --add-repo=https://mirrors.cloud.tencent.com/docker-ce/linux/centos/docker-ce.repo

# 执行以下命令，查看已添加的 Docker 软件源。
dnf list docker-ce

# 执行以下命令，安装 Docker。
dnf install -y docker-ce --nobest

# 执行以下命令，运行 Docker。
systemctl start docker

# 执行以下命令，检查安装结果。
docker info

# 设置docker自启动 
systemctl enable docker.service
```


