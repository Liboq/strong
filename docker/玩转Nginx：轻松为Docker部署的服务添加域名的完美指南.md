## 玩转Nginx：轻松为Docker部署的服务添加域名的完美指南

在使用 Docker 部署服务并添加域名时，遇到了问题。

![Nginx Configuration Issue](https://cdn.liboqiao.top/markdown/image-20240329092218946.png)

无法正常访问。

后来咨询了运维同学，经过查看我的 Nginx 配置和 DNS 解析，发现将 `http` 改为 `https` 后问题解决了。运维同学解释说，HTTP 默认使用 80 端口，而 HTTPS 默认使用 443 端口。我的配置中只有 443 端口，所以改成了 `https` 才能正常访问。

我记录下了如何使用 Nginx 给服务添加域名的步骤：

**备注：** 我使用的是腾讯云，并且域名已备案。

首先，在腾讯云控制台搜索 SSL 证书，并申领免费证书。

![image-20240329101138373](https://cdn.liboqiao.top/markdown/image-20240329101138373.png)

填写域名/子域名，点击自动 DNS 验证即可。

![DNS 验证](https://cdn.liboqiao.top/markdown/image-20240329093156232.png)

签发完成后，下载证书，选择 Nginx 下载。

![下载证书](https://cdn.liboqiao.top/markdown/image-20240329093313305.png)

![证书下载选项](https://cdn.liboqiao.top/markdown/image-20240329093403571.png)

下载完成后，将证书放到服务器的 Nginx 上。您可以在配置文件夹下创建一个 `cert` 文件夹，用于存储域名证书。然后解压缩证书。我的配置文件夹位于 `/etc/nginx/`，以下是解压缩的命令（实际命令可能会有所不同）：

```bash
unzip /etc/nginx/cert/umi.liboqiao.top_nginx.zip -d /etc/nginx/cert/
```

接着，在您的 `nginx.conf` 文件中，查看是否有类似于 `include /etc/nginx/conf.d/*.conf;` 的配置。如果没有，您可以在 `http` 部分添加这一行。`conf.d` 文件夹下的配置文件都可以作为域名的配置文件，这样配置会更清晰。

![nginx.conf 配置](https://cdn.liboqiao.top/markdown/image-20240329094744957.png)

```nginx
#umi.conf


 server {
        listen 443 ssl;
        # 域名名称
        server_name  umi.liboqiao.top;
        #证书地址
		ssl_certificate /etc/nginx/cert/umi.liboqiao.top_nginx/umi.liboqiao.top_bundle.pem;
		ssl_certificate_key /etc/nginx/cert/umi.liboqiao.top_nginx/umi.liboqiao.top.key;
		location / {
		    proxy_set_header X-Real-IP $remote_addr;
		    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header Host $http_host;
             proxy_set_header X-Nginx-Proxy true;
             proxy_set_header Connection "";
             #代理的域名
			proxy_pass http://127.0.0.1:8000;
		}
}
```

这是我的 `umi.conf` 文件内容。由于我在 Docker 中已经部署了 8000 端口，所以我直接代理到了 `http://127.0.0.1:8000`。

**总结**

通过这次使用 Nginx 给服务添加域名的经验，我学到了许多问题的解决方法，也加深了对 Nginx 的理解。希望这篇文章对您有所帮助。如果有任何错误或疑问，请指出，我们一起学习进步。

- [Github](https://github.com/Liboq/umi-admin)
- [umiadmin](https://umi.liboqiao.top)
