

# Docker Compose 常用命令

Docker Compose 是 Docker 官方提供的用于定义和运行多容器 Docker 应用程序的工具。它通过一个简单的 YAML 文件来配置服务，然后使用单个命令即可启动、停止和管理整个应用程序。下面是一些常用的 Docker Compose 命令，可以帮助你快速上手使用 Docker Compose。

## 启动和停止应用程序

### 启动应用程序
```bash
docker-compose up [OPTIONS] [SERVICE...]
```

示例：
```bash
docker-compose up
```

### 启动并重新构建镜像
```bash
docker-compose up --build [SERVICE...]
```

### 停止应用程序
```bash
docker-compose down [OPTIONS]
```

示例：
```bash
docker-compose down
```

## 管理服务

### 列出所有服务
```bash
docker-compose ps
```

### 启动单个服务
```bash
docker-compose start [SERVICE...]
```

### 停止单个服务
```bash
docker-compose stop [SERVICE...]
```

### 重启单个服务
```bash
docker-compose restart [SERVICE...]
```

## 查看日志

### 查看日志
```bash
docker-compose logs [SERVICE...]
```

### 查看实时日志
```bash
docker-compose logs -f [SERVICE...]
```

## 其他

### 构建服务
```bash
docker-compose build [SERVICE...]
```

### 运行命令
```bash
docker-compose exec [SERVICE] COMMAND [ARGS...]
```

示例：
```bash
docker-compose exec webserver ls -l
```

### 查看帮助信息
```bash
docker-compose --help
```

---

