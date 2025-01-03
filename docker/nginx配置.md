Nginx 配置

docker 用来部署 nginx 十分 方便 

> 由于linux 安装nginx 一直报错 主要是openssl 下载失败 等，无法使用 ssl，然后 就 试试 dokcer 来部署 nginx

## 步骤

### 1、docker拉取镜像

```
//docker 拉取镜像
docker pull nginx@latest

// docker 部署 nginx镜像 
docker run --name nginx666 -p 80:80 -p 443:443     -v /path/to/nginx/ssl:/etc/nginx/ssl     -v /path/to/nginx/config/nginx.conf:/etc/nginx/nginx.conf     -v /path/to/nginx/config/conf.d:/etc/nginx/conf.d     -d nginx

```

### 2、配置文件

**/path/to/nginx/ssl **

```
此文件主要是放 ssl证书
```

****

**/path/to/nginx/config/nginx.conf**

```
user  nginx;                    # 启动 Nginx 主进程的用户名
worker_processes  auto;         # 启动的 worker 进程数，默认为 1 ，设置成 auto 则会等于 CPU 核数
# worker_shutdown_timeout 60s;  # 限制 worker 进程优雅终止的超时时间。默认没有超时时间，可能一直于 shutting down 状态
# daemon off;                   # 是否以 daemon 方式运行 Nginx 进程，默认为 on
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;  # 将 Nginx 主进程的 PID 记录到该文件中

events {
    worker_connections  512;    # 每个 worker 进程的最大并发连接数，包括与客户端的连接、与 upsteam 的连接。默认为 512 。该值应该不能超过 ulimit -n 限制
    # use epoll;                # 处理请求事件的方法，默认会自动选择。在 Linux 上优先采用 epoll ，其次是 poll、select 等
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    # tcp_nopush     on;
    keepalive_timeout  65;
    # gzip  on;
    include /etc/nginx/conf.d/*.conf;
}
```

****

**/path/to/nginx/config/conf.d**

```
server {
    listen       80;            # 该 server 监听的地址（必填）
    server_name  localhost;     # 监听的域名（可选）

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

**ssl配置**

```
 server {
        listen 443 ssl;
        # 域名名称
        server_name  score.liboqiao.top;
        #证书地址
        ssl_certificate /etc/nginx/ssl/score.liboqiao.top_nginx/score.liboqiao.top_bundle.pem;
        ssl_certificate_key /etc/nginx/ssl/score.liboqiao.top_nginx/score.liboqiao.top.key;
        location / {
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header Host $http_host;
             proxy_set_header X-Nginx-Proxy true;
             proxy_set_header Connection "";
             #代理的域名
            proxy_pass http://110.42.211.49:3008;
        }
}
```

### 重启 nginx

```
docker ps 
docker restart [containerId]
```



