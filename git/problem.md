![image-20240522110732290](https://cdn.liboqiao.top/markdown/image-20240522110732290.png)

![image-20240522110844133](https://cdn.liboqiao.top/markdown/image-20240522110844133.png)

```
在.ssh 中创建 `config` 文件 可以解决  IdentityFile 指你的私钥路径 
Host github.com
    Port 443
    HostName ssh.github.com
    User git
    IdentityFile ~/.ssh/umi

```

