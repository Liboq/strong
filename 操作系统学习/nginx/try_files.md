## try_files



`try_files`指令在nginx中用于按顺序检查文件或目录是否存在，并根据检查结果进行处理。如果指定的文件或目录存在，nginx会进行内部重定向来处理请求；如果不存在，则会返回指定的状态码。

以下是`try_files`指令的一个基本用法示例：

```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

这里的`try_files`指令做了以下几件事情：

1. 首先检查请求的URI（`$uri`）对应的文件是否存在，如果存在就直接提供该文件。
2. 如果第一步没有找到文件，那么会检查URI对应的目录是否存在（`$uri/`），如果存在目录且有默认的索引文件，比如`index.html`，就提供该索引文件。
3. 如果前两步都没有找到文件，那么会作为最后的备选项，提供`/index.html`文件。

`try_files`指令通常放在`location`块中，用于处理静态内容的请求。它可以与其他指令结合使用，例如`error_page`，来自定义处理不存在文件时的行为。

[更多详细信息和高级用法，可以参考nginx的官方文档或相关社区讨论](https://stackoverflow.com/questions/17798457/how-can-i-make-this-try-files-directive-work)[1](https://stackoverflow.com/questions/17798457/how-can-i-make-this-try-files-directive-work)[2](https://docs.nginx.com/nginx/admin-guide/web-server/serving-static-content/)。🌐