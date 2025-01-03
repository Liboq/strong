# HTML 面试题

## 1. 语义化标签的理解和使用

### 为什么要使用语义化标签？

- 提升代码可读性和可维护性
- 有利于 SEO 优化
- 提升无障碍访问体验
- 便于团队协作开发

### 常用的语义化标签有哪些？

- `<header>`: 页面或区块的头部
- `<nav>`: 导航栏
- `<main>`: 主要内容
- `<article>`: 独立的文章内容
- `<section>`: 区块
- `<aside>`: 侧边栏
- `<footer>`: 页脚
- `<figure>` 和 `<figcaption>`: 图片及其描述

实践示例：

```html
<header>
  <nav>
    <ul>
      <li><a href="#home">首页</a></li>
      <li><a href="#about">关于</a></li>
    </ul>
  </nav>
</header>
<main>
  <article>
    <h1>文章标题</h1>
    <section>
      <h2>章节标题</h2>
      <p>章节内容</p>
    </section>
  </article>
  <aside>
    <h3>相关文章</h3>
    <ul>
      <li>推荐文章1</li>
      <li>推荐文章2</li>
    </ul>
  </aside>
</main>
<footer>
  <p>版权信息 © 2024</p>
</footer>
```

## 2. HTML5 新特性详解

### 存储相关

#### localStorage 和 sessionStorage 的区别：

- 生命周期：
  - localStorage: 永久存储，除非手动删除
  - sessionStorage: 仅在当前会话期间有效
- 存储大小：两者一般都是 5MB
- 作用域：
  - localStorage: 同源的所有标签页共享
  - sessionStorage: 仅在当前标签页有效

```javascript
// localStorage 示例
localStorage.setItem("user", JSON.stringify({ name: "张三" }));
const user = JSON.parse(localStorage.getItem("user"));

// sessionStorage 示例
sessionStorage.setItem("token", "xxx");
const token = sessionStorage.getItem("token");
```

### Web Workers

#### 使用场景：

- 复杂计算（如图像处理）
- 大数据处理
- 实时数据处理

示例代码：

```javascript
// main.js
const worker = new Worker("worker.js");
worker.postMessage({ data: [1, 2, 3, 4] });
worker.onmessage = function (e) {
  console.log("计算结果：", e.data);
};

// worker.js
self.onmessage = function (e) {
  const result = e.data.data.map((x) => x * x);
  self.postMessage(result);
};
```

### Service Workers

#### 主要用途：

- 离线缓存
- 消息推送
- 后台同步

基本实现：

```javascript
// 注册 Service Worker
if ("serviceWorker" in navigator) {
  navigator.serviceWorker
    .register("/sw.js")
    .then((registration) => {
      console.log("SW 注册成功：", registration.scope);
    })
    .catch((err) => {
      console.log("SW 注册失败：", err);
    });
}

// sw.js
self.addEventListener("install", (event) => {
  event.waitUntil(
    caches.open("v1").then((cache) => {
      return cache.addAll(["/", "/index.html", "/styles.css", "/app.js"]);
    })
  );
});

self.addEventListener("fetch", (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => response || fetch(request))
  );
});
```

## 3. 跨域解决方案

### CORS（跨源资源共享）

服务器配置示例：

```javascript
// Node.js Express 示例
app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Methods", "GET,POST,PUT,DELETE,OPTIONS");
  res.header("Access-Control-Allow-Headers", "Content-Type");
  next();
});
```

### JSONP 实现

```javascript
function jsonp(url, callback) {
  const script = document.createElement("script");
  const callbackName = "jsonp_" + Date.now();

  window[callbackName] = function (data) {
    callback(data);
    document.body.removeChild(script);
    delete window[callbackName];
  };

  script.src = `${url}?callback=${callbackName}`;
  document.body.appendChild(script);
}

// 使用示例
jsonp("http://api.example.com/data", function (data) {
  console.log("获取的数据：", data);
});
```
