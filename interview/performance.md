# 性能优化面试题

## 1. 前端性能指标

### 性能指标概述

- 概念解释：
  - FCP (First Contentful Paint): 首次内容绘制
  - LCP (Largest Contentful Paint): 最大内容绘制
  - TTI (Time to Interactive): 可交互时间
  - TBT (Total Blocking Time): 总阻塞时间
  - CLS (Cumulative Layout Shift): 累积布局偏移

```javascript
// 1. Performance API 使用
// 性能指标监控
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.entryType === "largest-contentful-paint") {
      console.log("LCP:", entry.startTime);
    }
    if (entry.entryType === "first-contentful-paint") {
      console.log("FCP:", entry.startTime);
    }
  }
});

observer.observe({
  entryTypes: ["largest-contentful-paint", "first-contentful-paint"],
});

// 2. Web Vitals 库使用
import { getLCP, getFID, getCLS } from "web-vitals";

function sendToAnalytics({ name, delta, id }) {
  // 发送性能数据到分析服务
  fetch("/analytics", {
    method: "POST",
    body: JSON.stringify({ name, delta, id }),
  });
}

getCLS(sendToAnalytics);
getFID(sendToAnalytics);
getLCP(sendToAnalytics);
```

### 性能监控方案

```javascript
// 1. 自定义性能监控
class PerformanceMonitor {
  constructor() {
    this.metrics = {};
    this.init();
  }

  init() {
    // 记录页面加载时间
    window.addEventListener("load", () => {
      const timing = performance.timing;
      this.metrics.loadTime = timing.loadEventEnd - timing.navigationStart;
      this.metrics.domReady =
        timing.domContentLoadedEventEnd - timing.navigationStart;
      this.sendMetrics();
    });

    // 记录资源加载时间
    const observer = new PerformanceObserver((list) => {
      const entries = list.getEntries();
      entries.forEach((entry) => {
        if (entry.entryType === "resource") {
          this.metrics[entry.name] = entry.duration;
        }
      });
    });

    observer.observe({ entryTypes: ["resource"] });
  }

  sendMetrics() {
    // 发送性能数据
    navigator.sendBeacon("/analytics", JSON.stringify(this.metrics));
  }
}

// 2. 错误监控
window.addEventListener("error", (event) => {
  const error = {
    message: event.message,
    filename: event.filename,
    lineno: event.lineno,
    colno: event.colno,
    stack: event.error?.stack,
  };

  // 发送错误信息
  navigator.sendBeacon("/error", JSON.stringify(error));
});

// 3. 资源加载监控
const resourceObserver = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    if (entry.initiatorType === "img" || entry.initiatorType === "script") {
      console.log(`${entry.name} loaded in: ${entry.duration}ms`);
    }
  });
});

resourceObserver.observe({ entryTypes: ["resource"] });
```

## 2. 加载优化

### 懒加载实现

```javascript
// 1. 组件懒加载
const LazyComponent = React.lazy(() => import("./LazyComponent"));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <LazyComponent />
    </Suspense>
  );
}

// 2. 路由懒加载
const routes = [
  {
    path: "/dashboard",
    component: () => import("./views/Dashboard"),
  },
];

// 3. 图片懒加载
function LazyImage({ src, alt }) {
  const [isLoaded, setIsLoaded] = useState(false);
  const imgRef = useRef();

  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) {
        setIsLoaded(true);
        observer.disconnect();
      }
    });

    observer.observe(imgRef.current);
    return () => observer.disconnect();
  }, []);

  return (
    <img ref={imgRef} src={isLoaded ? src : "placeholder.jpg"} alt={alt} />
  );
}
```

### 预加载策略

```javascript
// 1. 资源预加载
document.head.appendChild(Object.assign(
  document.createElement('link'),
  {
    rel: 'preload',
    href: 'heavy-component.js',
    as: 'script'
  }
));

// 2. DNS预解析
<link rel="dns-prefetch" href="//example.com">

// 3. 预连接
<link rel="preconnect" href="https://example.com">

// 4. 条件预加载
const prefetchComponent = () => {
  const link = document.createElement('link');
  link.rel = 'prefetch';
  link.href = '/component.js';
  document.head.appendChild(link);
};

// 在空闲时预加载
if (requestIdleCallback) {
  requestIdleCallback(prefetchComponent);
} else {
  setTimeout(prefetchComponent, 1000);
}
```

### 缓存优化

```javascript
// 1. Service Worker 缓存
// sw.js
const CACHE_NAME = 'v1';
const urlsToCache = [
  '/',
  '/styles.css',
  '/app.js'
];

self.addEventListener('install', event => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => cache.addAll(urlsToCache))
  );
});

self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request)
      .then(response => {
        if (response) {
          return response;
        }
        return fetch(event.request).then(
          response => {
            if (!response || response.status !== 200) {
              return response;
            }
            const responseToCache = response.clone();
            caches.open(CACHE_NAME)
              .then(cache => {
                cache.put(event.request, responseToCache);
              });
            return response;
          }
        );
      })
  );
});

// 2. HTTP 缓存配置
// Nginx配置
location /static/ {
  expires 1y;
  add_header Cache-Control "public, no-transform";
}

// 3. 内存缓存
const cache = new Map();

function memoize(fn) {
  return function (...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}
```
