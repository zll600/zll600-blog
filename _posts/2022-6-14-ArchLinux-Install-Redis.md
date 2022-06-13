# ArchLinux Install Redis 

### 安装命令
> sudo pacman -S redis

### 开始 redis.service
> sudo systemctl start redis.service

这里会读取默认配置，你也可以自定义配置文件路径，然后使用 `redis-server` 来启动
