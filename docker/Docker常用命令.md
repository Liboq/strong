# Docker 常用命令

Docker 是一个用于构建、发布和运行容器化应用程序的开源平台。下面是一些常用的 Docker 命令，可以帮助你快速上手使用 Docker。

## 容器管理

### 启动容器
```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

示例：
```bash
docker run -it --rm ubuntu:latest /bin/bash
```

### 列出所有正在运行的容器
```bash
docker ps
```

### 列出所有容器（包括停止的）
```bash
docker ps -a
```

### 停止容器
```bash
docker stop CONTAINER [CONTAINER...]
```

### 删除容器
```bash
docker rm CONTAINER [CONTAINER...]
```

### 查看容器日志
```bash
docker logs CONTAINER
```

## 镜像管理

### 拉取镜像
```bash
docker pull IMAGE[:TAG]
```

### 查看本地所有镜像
```bash
docker images
```

### 删除镜像
```bash
docker rmi IMAGE [IMAGE...]
```

### 构建镜像
```bash
docker build [OPTIONS] PATH | URL | -
```

## 网络管理

### 查看 Docker 网络
```bash
docker network ls
```

### 创建网络
```bash
docker network create NETWORK
```

### 删除网络
```bash
docker network rm NETWORK
```

## 数据卷管理

### 创建数据卷
```bash
docker volume create VOLUME
```

### 查看数据卷
```bash
docker volume ls
```

### 删除数据卷
```bash
docker volume rm VOLUME
```

## 其他

### 查看 Docker 版本信息
```bash
docker version
```

### 查看 Docker 系统信息
```bash
docker info
```

### 在容器内执行命令
```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

示例：
```bash
docker exec -it CONTAINER /bin/bash
```

