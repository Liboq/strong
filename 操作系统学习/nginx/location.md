## location

在Nginx中，`location`指令用于匹配请求的URI以决定请求应该如何处理。匹配规则有以下几种：

1. **精确匹配** (`=`): 当location后面跟着等号和路径时，只有完全匹配这个路径的请求才会被这个规则处理。

   ```nginx
   location = /exact/path {
       # 只有请求为 /exact/path 时才匹配
   }
   ```

2. **最长非正则匹配** (`^~`): 当location使用`^~`修饰符时，Nginx会匹配最长的非正则表达式路径。如果找到匹配，Nginx将停止搜索其他location。

   ```nginx
   location ^~ /longest/non/regex/path {
       # 匹配以 /longest/non/regex/path 开头的请求
   }
   ```

3. **正则表达式匹配** (`~` 和 `~*`): 使用`~`进行区分大小写的正则表达式匹配，使用`~*`进行不区分大小写的匹配。正则表达式匹配按它们在配置文件中出现的顺序进行。

   ```nginx
   location ~* \.(gif|jpg|png)$ {
       # 匹配所有以 .gif, .jpg, .png 结尾的请求
   }
   ```

4. **无修饰符前缀匹配**: 如果location没有任何修饰符，Nginx会匹配请求URI的前缀。

   ```nginx
   location /prefix {
       # 匹配以 /prefix 开头的请求
   }
   ```

Nginx在处理请求时，会按照以下顺序检查location指令：

- 首先查找精确匹配（`=`）。
- 如果没有找到精确匹配，接着查找最长非正则匹配（`^~`）。
- 如果没有找到最长非正则匹配，再按定义顺序查找正则表达式匹配（`~` 或 `~*`）。
- 如果以上都没有匹配，最后使用无修饰符前缀匹配。

[这些规则确保了Nginx可以灵活地处理各种不同的请求路径](https://www.digitalocean.com/community/tutorials/nginx-location-directive)[1](https://www.digitalocean.com/community/tutorials/nginx-location-directive)[2](https://www.slingacademy.com/article/nginx-location-block-the-complete-guide/)。🌐