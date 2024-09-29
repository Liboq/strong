**解决方法**

```
cd /etc/docker/deamon.json
# 没有此文件夹
# 则先创建
touch /etc/docker/deamon.json
# 加入
{
  "registry-mirrors": ["https://registry.docker-cn.com","http://hub-mirror.c.163.com"]
}
# 然后
 systemctl daemon-reload
systemctl  restart docker

```

