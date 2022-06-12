ArchLinux 安装 docker

### 安装

就一条命令就可以了
> sudo pacman -S docker


### 启动 docker 服务
> sudo systemctl start docker.service

### 设置 docker 服务开机自启
> sudo systemctl enable docker.service

### 将当前用户添加到 docker 用户组中
> sudo usermod -aG docker $USER
