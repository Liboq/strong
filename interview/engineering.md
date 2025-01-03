## 前端工程化面试题

### 场景一：模块化开发

```text
面试官：能谈谈你对前端模块化的理解吗？在实际项目中是如何做模块化管理的？

候选人：
好的，前端模块化是解决代码组织和依赖管理的重要方案。我们可以从发展历程和实践经验来说明。

1. 模块化的演进过程：
   - 全局函数：命名空间冲突
   - IIFE：闭包隔离
   - CommonJS：Node.js 模块规范
   - AMD/RequireJS：异步加载
   - ES Modules：官方标准
```

```javascript
// 1. ES Modules 基本使用
// math.js
export const add = (a, b) => a + b;
export const multiply = (a, b) => a * b;

// main.js
import { add, multiply } from "./math";
import * as utils from "./utils";
import defaultExport from "./module";

// 2. 动态导入
const loadModule = async () => {
  const module = await import("./dynamic-module");
  module.doSomething();
};

// 3. 模块联邦示例
// webpack.config.js
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: "app1",
      filename: "remoteEntry.js",
      exposes: {
        "./Button": "./src/components/Button",
      },
      shared: ["react", "react-dom"],
    }),
  ],
};
```

在实际项目中的最佳实践：

1. 模块设计原则：

   - 单一职责
   - 封装内部实现
   - 明确的接口定义

2. 依赖管理：

   - 使用包管理器（npm/yarn/pnpm）
   - 版本控制策略
   - 依赖分析和优化

3. 构建优化：
   - Tree Shaking
   - 代码分割
   - 按需加载

### 场景二：构建工具和脚手架

```text
面试官：在项目中使用过哪些构建工具？如何选择和配置这些工具？

候选人：
我们项目中主要使用 Webpack、Vite 等构建工具，选择时主要考虑开发体验和构建性能。
```

```javascript
// 1. Webpack 配置示例
module.exports = {
  entry: "./src/index.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "[name].[contenthash].js",
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: "babel-loader",
        exclude: /node_modules/,
      },
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader", "postcss-loader"],
      },
    ],
  },
  plugins: [new HtmlWebpackPlugin(), new MiniCssExtractPlugin()],
  optimization: {
    splitChunks: {
      chunks: "all",
    },
  },
};

// 2. Vite 配置示例
// vite.config.js
export default defineConfig({
  plugins: [vue(), vueJsx()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ["vue", "vue-router"],
          utils: ["lodash-es", "axios"],
        },
      },
    },
  },
});
```

构建工具的选择考虑：

1. 开发体验：

   - 启动速度
   - 热更新性能
   - 调试便利性

2. 构建性能：

   - 并行处理
   - 缓存机制
   - 增量构建

3. 生态支持：
   - 插件丰富度
   - 社区活跃度
   - 文档完善度

### 场景三：CI/CD 和自动化部署

```text
面试官：能分享一下你们项目的 CI/CD 流程吗？如何实现自动化测试和部署？

候选人：
好的，我们采用 GitLab CI 和 Docker 来实现自动化流程。主要包括测试、构建和部署三个阶段。
```

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

test:
  stage: test
  script:
    - npm install
    - npm run test
  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'

build:
  stage: build
  script:
    - docker build -t myapp .
    - docker push myapp
  only:
    - master

deploy:
  stage: deploy
  script:
    - kubectl apply -f k8s/
  environment:
    name: production
```

自动化流程的关键点：

1. 测试策略：

   - 单元测试
   - E2E 测试
   - 性能测试

2. 部署策略：

   - 蓝绿部署
   - 金丝雀发布
   - 回滚机制

3. 监控告警：
   - 性能监控
   - 错误追踪
   - 服务健康检查

### 场景四：性能优化和监控

```text
面试官：在工程化层面，你们是如何做性能优化和监控的？

候选人：
我们从构建优化、资源加载和运行时性能三个方面来进行优化，同时建立了完整的监控体系。
```

```javascript
// 1. 构建优化配置
module.exports = {
  optimization: {
    moduleIds: "deterministic",
    runtimeChunk: "single",
    splitChunks: {
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: "vendors",
          chunks: "all",
        },
      },
    },
  },
};

// 2. 性能监控
class Performance {
  constructor() {
    this.data = {};
    this.init();
  }

  init() {
    this.observeResource();
    this.observeLCP();
    this.observeFID();
  }

  observeResource() {
    new PerformanceObserver((list) => {
      const entries = list.getEntries();
      entries.forEach((entry) => {
        if (
          entry.initiatorType === "fetch" ||
          entry.initiatorType === "xmlhttprequest"
        ) {
          this.logApiPerformance(entry);
        }
      });
    }).observe({ entryTypes: ["resource"] });
  }

  logApiPerformance(entry) {
    const metrics = {
      name: entry.name,
      duration: entry.duration,
      startTime: entry.startTime,
      transferSize: entry.transferSize,
    };
    this.send(metrics);
  }
}
```

性能优化的关键领域：

1. 构建优化：

   - 代码分割
   - 资源压缩
   - 缓存优化

2. 加载优化：

   - 懒加载
   - 预加载
   - CDN 分发

3. 监控指标：
   - 核心性能指标
   - 业务指标
   - 用户体验指标

### 场景五：代码规范和质量控制

```text
面试官：你们团队是如何保证代码质量的？有哪些具体的规范和工具？

候选人：
我们通过代码规范、Code Review 和自动化工具等多个层面来保证代码质量。
```

```javascript
// 1. ESLint 配置示例
module.exports = {
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'prettier'
  ],
  plugins: ['@typescript-eslint', 'prettier'],
  rules: {
    'prettier/prettier': 'error',
    'no-console': ['warn', { allow: ['warn', 'error'] }],
    'no-unused-vars': 'error',
    '@typescript-eslint/explicit-function-return-type': 'error'
  },
  overrides: [
    {
      files: ['**/__tests__/**'],
      env: {
        jest: true
      }
    }
  ]
};

// 2. Git Hooks 配置
// .husky/pre-commit
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npm run lint-staged

// package.json
{
  "lint-staged": {
    "*.{js,ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{css,scss}": [
      "stylelint --fix"
    ]
  }
}

// 3. Jest 测试配置
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'jsdom',
  collectCoverage: true,
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80
    }
  }
};
```

代码质量控制的关键点：

1. 代码规范：

   - 编码风格规范
   - 命名规范
   - 文档规范

2. 自动化工具：

   - ESLint/TSLint
   - Prettier
   - StyleLint

3. 测试覆盖：
   - 单元测试
   - 集成测试
   - 快照测试

### 场景六：微前端架构

```text
面试官：能谈谈你对微前端的理解和实践经验吗？

候选人：
微前端是一种将前端应用分解成更小、更易管理的独立部分的架构模式。我们在实践中主要考虑应用隔离、通信机制和部署策略。
```

```javascript
// 1. qiankun 配置示例
// main.js (主应用)
import { registerMicroApps, start } from "qiankun";

registerMicroApps([
  {
    name: "app1",
    entry: "//localhost:8081",
    container: "#container",
    activeRule: "/app1",
  },
  {
    name: "app2",
    entry: "//localhost:8082",
    container: "#container",
    activeRule: "/app2",
  },
]);

start();

// 2. 子应用配置
// vue.config.js (子应用)
module.exports = {
  devServer: {
    port: 8081,
    headers: {
      "Access-Control-Allow-Origin": "*",
    },
  },
  configureWebpack: {
    output: {
      library: "app1",
      libraryTarget: "umd",
    },
  },
};

// 3. 应用间通信
// 主应用
import { initGlobalState } from "qiankun";

const actions = initGlobalState({
  user: "admin",
  theme: "dark",
});

actions.onGlobalStateChange((state, prev) => {
  console.log("主应用: 变更前", prev);
  console.log("主应用: 变更后", state);
});

// 子应用
export function mount(props) {
  props.onGlobalStateChange((state, prev) => {
    console.log("子应用: 变更前", prev);
    console.log("子应用: 变更后", state);
  });
}
```

微前端实践要点：

1. 应用拆分：

   - 业务边界划分
   - 技术栈选择
   - 共享资源管理

2. 应用通信：

   - 状态共享
   - 事件通信
   - 路由同步

3. 部署策略：
   - 独立部署
   - 版本控制
   - 环境隔离

### 场景七：qiankun 的选择和实践

```text
面试官：为什么选择 qiankun 作为微前端方案？与其他方案相比有什么优势？

候选人：
选择 qiankun 主要基于以下几个方面的考虑：

1. 技术方案成熟：
   - 蚂蚁金服内部大规模验证
   - 社区活跃度高
   - 文档完善

2. 功能完备：
   - 样式隔离
   - JS 沙箱
   - 预加载能力

让我详细说明一下具体实践：
```

```javascript
// 1. 样式隔离方案
// 主应用配置
registerMicroApps([
  {
    name: "app1",
    entry: "//localhost:8081",
    container: "#container",
    activeRule: "/app1",
    props: {
      sandbox: {
        strictStyleIsolation: true, // 严格的样式隔离
        experimentalStyleIsolation: true, // 实验性的样式隔离
      },
    },
  },
]);

// 2. JS 沙箱实现
// 快照沙箱（单实例）
class SnapshotSandbox {
  constructor() {
    this.windowSnapshot = {};
    this.modifyPropsMap = {};
  }

  active() {
    // 记录当前 window 状态
    for (const prop in window) {
      this.windowSnapshot[prop] = window[prop];
    }
    // 恢复之前的状态
    Object.keys(this.modifyPropsMap).forEach((prop) => {
      window[prop] = this.modifyPropsMap[prop];
    });
  }

  inactive() {
    // 记录更改的状态
    for (const prop in window) {
      if (window[prop] !== this.windowSnapshot[prop]) {
        this.modifyPropsMap[prop] = window[prop];
        window[prop] = this.windowSnapshot[prop];
      }
    }
  }
}

// 3. 预加载实现
import { prefetchApps } from "qiankun";

prefetchApps([
  { name: "app1", entry: "//localhost:8081" },
  { name: "app2", entry: "//localhost:8082" },
]);

// 4. 生命周期管理
export async function bootstrap() {
  console.log("子应用启动");
}

export async function mount(props) {
  console.log("子应用挂载");
  props.onGlobalStateChange((state, prev) => {
    // 状态变更处理
  });
  render(props);
}

export async function unmount() {
  console.log("子应用卸载");
  instance.$destroy();
}
```

与其他方案对比：

1. single-spa：

   - 优势：轻量、灵活
   - 劣势：
     - 需要自行处理样式隔离
     - 缺少 JS 沙箱
     - 配置较复杂

2. iframe：

   - 优势：天然隔离
   - 劣势：
     - 性能较差
     - 通信不便
     - 用户体验欠佳

3. 自研方案：
   - 优势：完全可控
   - 劣势：
     - 开发成本高
     - 维护负担重
     - 需要解决的问题多

qiankun 的最佳实践：

1. 应用配置：

   - 合理的路由规划
   - 正确的生命周期管理
   - 完善的错误处理

2. 性能优化：

   - 预加载策略
   - 资源加载优化
   - 缓存处理

3. 开发规范：
   - 统一的开发规范
   - 明确的发布流程
   - 完善的监控体系

### 场景八：包管理和版本控制

```text
面试官：能谈谈你们项目中的包管理策略吗？如何处理依赖和版本控制？

候选人：
我们主要使用 pnpm 作为包管理工具，并采用 monorepo 的方式管理多个项目。

主要考虑以下几个方面：
1. 包管理工具的选择
2. 依赖管理策略
3. 版本控制方案
```

```javascript
// 1. pnpm workspace 配置
// pnpm-workspace.yaml
packages:
  - 'packages/*'
  - 'apps/*'
  - '!**/test/**'

// 2. 包管理配置
// package.json
{
  "name": "root",
  "private": true,
  "workspaces": [
    "packages/*",
    "apps/*"
  ],
  "scripts": {
    "preinstall": "npx only-allow pnpm",
    "build": "pnpm -r build",
    "test": "pnpm -r test",
    "clean": "pnpm -r clean"
  }
}

// 3. 版本控制
// lerna.json
{
  "version": "independent",
  "npmClient": "pnpm",
  "useWorkspaces": true,
  "command": {
    "publish": {
      "conventionalCommits": true,
      "message": "chore(release): publish"
    }
  }
}

// 4. 依赖管理
// packages/utils/package.json
{
  "name": "@myapp/utils",
  "version": "1.0.0",
  "dependencies": {
    "lodash-es": "^4.17.21"
  },
  "peerDependencies": {
    "react": "^17.0.0"
  }
}
```

包管理策略的关键点：

1. 工具选择：

   - pnpm：硬链接共享、节省空间
   - yarn：稳定性好、功能完善
   - npm：生态最大、兼容性好

2. Monorepo 优势：

   - 代码共享和复用
   - 统一的工具链
   - 原子提交和发布

3. 版本管理：
   - 语义化版本
   - Changelog 生成
   - 发布流程自动化

最佳实践：

1. 依赖管理：

   ```bash
   # 安装依赖到工作区根目录
   pnpm add -w husky

   # 安装依赖到特定包
   pnpm add lodash --filter @myapp/utils

   # 包之间的依赖
   pnpm add @myapp/utils --filter @myapp/web
   ```

2. 版本控制：

   ```bash
   # 生成 changelog
   pnpm run changelog

   # 发布包
   pnpm publish -r

   # 指定包发布
   pnpm publish --filter @myapp/utils
   ```

3. 工作流程：
   - 统一的 commit 规范
   - 自动化的版本管理
   - CI/CD 集成

### 场景九：前端安全防护

```text
面试官：在工程化层面，你们是如何处理前端安全问题的？有哪些具体的防护措施？

候选人：
我们从多个层面来构建前端安全防护体系，包括开发规范、运行时防护和部署配置等。

主要防护方向：
1. XSS 防护
2. CSRF 防护
3. 敏感信息保护
```

```javascript
// 1. CSP 配置
// 在 HTML 中
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline';">

// 或在服务器响应头中
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'

// 2. XSS 防护
class XSSFilter {
  static sanitize(input) {
    return input.replace(/[&<>"']/g, char => ({
      '&': '&amp;',
      '<': '&lt;',
      '>': '&gt;',
      '"': '&quot;',
      "'": '&#39;'
    }[char]));
  }

  static validateURL(url) {
    const urlPattern = /^(https?:\/\/)?([\da-z.-]+)\.([a-z.]{2,6})([/\w .-]*)*\/?$/;
    return urlPattern.test(url);
  }
}

// 3. CSRF Token 实现
class CSRFProtection {
  constructor() {
    this.token = this.generateToken();
  }

  generateToken() {
    return Math.random().toString(36).substring(2);
  }

  // 添加 token 到请求头
  setupAxiosInterceptor(axios) {
    axios.interceptors.request.use(config => {
      config.headers['X-CSRF-Token'] = this.token;
      return config;
    });
  }
}

// 4. 敏感信息加密
class Encryption {
  static encryptData(data, key) {
    // 使用 Web Crypto API
    return crypto.subtle.encrypt(
      {
        name: "AES-GCM",
        iv: window.crypto.getRandomValues(new Uint8Array(12))
      },
      key,
      new TextEncoder().encode(JSON.stringify(data))
    );
  }
}
```

安全防护的关键点：

1. 开发规范：

   - 输入验证
   - 输出转义
   - 安全编码规范

2. 运行时防护：

   - CSP 策略
   - 请求安全
   - 数据加密

3. 部署配置：
   ```nginx
   # Nginx 安全配置
   server {
     # 开启 HSTS
     add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

     # 防止点击劫持
     add_header X-Frame-Options "SAMEORIGIN" always;

     # XSS 防护
     add_header X-XSS-Protection "1; mode=block" always;

     # 禁止嗅探 MIME 类型
     add_header X-Content-Type-Options "nosniff" always;

     # 引用策略
     add_header Referrer-Policy "strict-origin-when-cross-origin" always;
   }
   ```

最佳实践：

1. 漏洞防护：

   - 定期安全扫描
   - 依赖包更新
   - 漏洞修复流程

2. 监控告警：

   - 异常监控
   - 攻击检测
   - 实时告警

3. 应急响应：
   - 应急预案
   - 快速回滚
   - 事件追踪

### 场景十：全方位性能优化

```text
面试官：能详细讲讲前端性能优化的完整方案吗？从工程化角度如何保证性能？

候选人：
我们从构建层面、网络层面、运行时三个维度来做性能优化，并建立了完整的性能监控体系。

主要优化方向：
1. 构建优化
2. 加载优化
3. 运行时优化
```

```javascript
// 1. Webpack 构建优化
module.exports = {
  // 持久化缓存
  cache: {
    type: "filesystem",
    buildDependencies: {
      config: [__filename],
    },
  },

  // 多进程构建
  module: {
    rules: [
      {
        test: /\.js$/,
        use: [
          {
            loader: "thread-loader",
            options: {
              workers: 4,
            },
          },
          "babel-loader",
        ],
      },
    ],
  },

  // 代码分割
  optimization: {
    splitChunks: {
      chunks: "all",
      cacheGroups: {
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
        },
        common: {
          minChunks: 2,
          priority: -20,
        },
      },
    },
  },
};

// 2. 资源加载优化
// 图片懒加载
class LazyLoad {
  constructor() {
    this.observer = new IntersectionObserver(this.handleIntersection);
  }

  observe(el) {
    this.observer.observe(el);
  }

  handleIntersection(entries) {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        const img = entry.target;
        img.src = img.dataset.src;
        this.observer.unobserve(img);
      }
    });
  }
}

// 3. 运行时性能优化
class PerformanceOptimization {
  // 虚拟列表实现
  createVirtualList(container, items, itemHeight) {
    let startIndex = 0;
    let endIndex = 0;
    const visibleCount = Math.ceil(container.clientHeight / itemHeight);

    const updateVisibleItems = () => {
      const scrollTop = container.scrollTop;
      startIndex = Math.floor(scrollTop / itemHeight);
      endIndex = startIndex + visibleCount;

      const visibleItems = items.slice(startIndex, endIndex);
      this.renderItems(visibleItems, startIndex * itemHeight);
    };

    container.addEventListener("scroll", updateVisibleItems);
    updateVisibleItems();
  }

  // 防抖节流
  debounce(fn, delay) {
    let timer = null;
    return function (...args) {
      if (timer) clearTimeout(timer);
      timer = setTimeout(() => fn.apply(this, args), delay);
    };
  }

  throttle(fn, delay) {
    let last = 0;
    return function (...args) {
      const now = Date.now();
      if (now - last > delay) {
        fn.apply(this, args);
        last = now;
      }
    };
  }
}

// 4. 性能监控
class PerformanceMonitor {
  constructor() {
    this.metrics = {};
    this.init();
  }

  init() {
    // 监控核心指标
    this.observeCLS();
    this.observeFID();
    this.observeLCP();

    // 监控资源加载
    this.observeResource();

    // 监控长任务
    this.observeLongTask();
  }

  observeLongTask() {
    const observer = new PerformanceObserver((list) => {
      list.getEntries().forEach((entry) => {
        if (entry.duration > 50) {
          console.warn("检测到长任务:", entry);
        }
      });
    });

    observer.observe({ entryTypes: ["longtask"] });
  }
}
```

性能优化的关键点：

1. 构建优化：

   - 减小包体积
   - 提升构建速度
   - 优化缓存策略

2. 加载优化：

   - 资源预加载
   - 按需加载
   - 并行加载

3. 运行时优化：
   - 代码执行效率
   - DOM 操作优化
   - 内存管理

最佳实践：

1. 构建阶段：

   ```javascript
   // webpack.config.js
   module.exports = {
     // Tree Shaking
     optimization: {
       usedExports: true,
       minimize: true,
     },

     // 资源压缩
     optimization: {
       minimizer: [new TerserPlugin(), new CssMinimizerPlugin()],
     },
   };
   ```

2. 部署阶段：

   ```nginx
   # Nginx 缓存配置
   location /static/ {
     expires 1y;
     add_header Cache-Control "public, no-transform";
   }

   # Gzip 压缩
   gzip on;
   gzip_types text/plain text/css application/json application/javascript;
   ```

3. 监控告警：
   - 性能指标监控
   - 异常监控
   - 性能预警

### 场景十一：构建工具生态应用

```text
面试官：能详细说说你在项目中使用过的 Webpack loader 和 plugin，以及为什么选择 Vite？

候选人：
我在项目中使用过多种 loader 和 plugin 来处理不同类型的资源和优化构建流程。同时，我们新项目选择了 Vite 主要考虑其开发体验和构建性能。

让我详细说明一下：
```

```javascript
// 1. 常用的 Webpack Loader
module.exports = {
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        use: [
          {
            loader: "babel-loader",
            options: {
              cacheDirectory: true, // 启用缓存
              presets: ["@babel/preset-react"],
            },
          },
        ],
      },
      {
        test: /\.tsx?$/,
        use: [
          {
            loader: "ts-loader",
            options: {
              transpileOnly: true, // 只做语言转换，不做类型检查
            },
          },
        ],
      },
      {
        test: /\.scss$/,
        use: [
          "style-loader", // 将 CSS 注入到 DOM
          {
            loader: "css-loader",
            options: {
              modules: true, // 启用 CSS 模块
              importLoaders: 2, // 在 css-loader 前应用的 loader 数量
            },
          },
          "postcss-loader", // 处理兼容性
          "sass-loader", // 编译 SCSS
        ],
      },
      {
        test: /\.(png|jpe?g|gif|svg)$/,
        use: [
          {
            loader: "url-loader",
            options: {
              limit: 8192, // 小于 8kb 的图片转为 base64
              name: "images/[name].[hash:8].[ext]",
            },
          },
        ],
      },
    ],
  },
};

// 2. 常用的 Webpack Plugin
const plugins = [
  new HtmlWebpackPlugin({
    template: "./public/index.html",
    minify: {
      removeComments: true,
      collapseWhitespace: true,
    },
  }),
  new MiniCssExtractPlugin({
    filename: "css/[name].[contenthash:8].css",
  }),
  new webpack.DefinePlugin({
    "process.env": {
      NODE_ENV: JSON.stringify(process.env.NODE_ENV),
    },
  }),
  new CopyWebpackPlugin({
    patterns: [{ from: "public", to: "public" }],
  }),
  new BundleAnalyzerPlugin(), // 分析包体积
];

// 3. Vite 配置示例
// vite.config.ts
export default defineConfig({
  plugins: [
    vue(),
    vueJsx(),
    legacy({
      targets: ["defaults", "not IE 11"],
    }),
  ],
  resolve: {
    alias: {
      "@": "/src",
    },
  },
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ["vue", "vue-router", "vuex"],
          lodash: ["lodash-es"],
        },
      },
    },
    terserOptions: {
      compress: {
        drop_console: true,
      },
    },
  },
});
```

Loader 和 Plugin 的使用场景：

1. 常用 Loader：

   - babel-loader：转换 ES6+ 语法
   - ts-loader：处理 TypeScript
   - style-loader/css-loader：处理样式
   - file-loader/url-loader：处理资源文件

2. 常用 Plugin：
   - HtmlWebpackPlugin：生成 HTML 文件
   - MiniCssExtractPlugin：提取 CSS 文件
   - DefinePlugin：定义环境变量
   - CopyWebpackPlugin：复制静态资源

选择 Vite 的原因：

1. 开发体验：

   - 快速的冷启动
   - 即时的模块热更新
   - 真正的按需加载

2. 构建性能：

   - 基于 ESM 的开发服务器
   - 基于 Rollup 的生产构建
   - 更好的缓存策略

3. 生态支持：
   - 兼容 Webpack 生态
   - 原生 CSS 变量支持
   - TypeScript 开箱即用

### 场景十二：前端工程化的整体理解

```text
面试官：能系统地讲讲你对前端工程化的理解吗？在实际项目中是如何落地的？

候选人：
前端工程化是为了解决前端开发过程中的效率、规范、质量等问题而产生的一系列解决方案。
我从以下几个维度来理解：

1. 开发效率
2. 规范和质量
3. 部署和运维
4. 性能和体验
```

```javascript
// 1. 项目脚手架示例
// create-app.js
class ProjectGenerator {
  constructor(options) {
    this.options = options;
    this.templates = {
      react: "./templates/react",
      vue: "./templates/vue",
    };
  }

  async generate() {
    // 1. 选择技术栈
    const { framework } = await inquirer.prompt([
      {
        type: "list",
        name: "framework",
        message: "选择框架:",
        choices: ["React", "Vue"],
      },
    ]);

    // 2. 复制模板
    await this.copyTemplate(framework.toLowerCase());

    // 3. 安装依赖
    await this.installDeps();

    // 4. 初始化 Git
    await this.initGit();
  }
}

// 2. 工程化配置示例
// 统一的项目配置
const projectConfig = {
  // 开发规范
  development: {
    eslint: {
      extends: ["airbnb", "prettier"],
      rules: {
        // 自定义规则
      },
    },
    prettier: {
      semi: true,
      singleQuote: true,
    },
    commitlint: {
      extends: ["@commitlint/config-conventional"],
    },
  },

  // 构建配置
  build: {
    target: ["chrome >= 87", "edge >= 88"],
    optimization: {
      splitChunks: true,
      treeshaking: true,
    },
    analysis: {
      bundleSize: true,
      performance: true,
    },
  },

  // 部署配置
  deploy: {
    environments: ["dev", "staging", "prod"],
    cdn: {
      domain: "https://cdn.example.com",
      cache: true,
    },
    docker: {
      enable: true,
      base: "node:16-alpine",
    },
  },
};

// 3. 工程化工具链
class EngineeringToolchain {
  // 开发环境配置
  setupDevelopment() {
    return {
      devServer: {
        hot: true,
        proxy: {
          "/api": "http://localhost:3000",
        },
      },
      plugins: [new ESLintPlugin(), new StylelintPlugin()],
    };
  }

  // 构建流程配置
  setupBuild() {
    return {
      optimization: {
        minimizer: [new TerserPlugin(), new CssMinimizerPlugin()],
      },
      plugins: [new BundleAnalyzerPlugin()],
    };
  }

  // CI/CD 配置
  setupCI() {
    return {
      test: {
        runner: "jest",
        coverage: 80,
      },
      deploy: {
        strategy: "blue-green",
        rollback: true,
      },
    };
  }
}
```

工程化的核心内容：

1. 开发效率提升：

   - 脚手架工具
   - 开发服务器
   - 热更新机制

2. 规范和质量：

   - 代码规范
   - 提交规范
   - 自动化测试

3. 构建和部署：
   - 资源构建
   - 环境部署
   - 监控告警

最佳实践：

1. 技术选型：

   ```javascript
   // 技术栈评估示例
   const techEvaluation = {
     framework: {
       options: ["React", "Vue"],
       criteria: ["团队熟悉度", "生态完整性", "性能表现", "维护活跃度"],
     },
     buildTool: {
       options: ["Webpack", "Vite"],
       criteria: ["构建性能", "配置灵活性", "插件丰富度"],
     },
   };
   ```

2. 工程化流程：

   ```bash
   # 开发流程
   git checkout -b feature/xxx
   npm run dev
   npm run test
   git commit -m "feat: xxx"

   # 构建流程
   npm run build
   npm run analyze

   # 部署流程
   docker build .
   kubectl apply -f k8s/
   ```

3. 监控和优化：
   - 性能监控
   - 错误监控
   - 用户体验监控

````

### 场景十三：AST 和编译原理

```text
面试官：能详细讲讲 AST 在前端工程化中的应用？你是如何理解和使用 AST 的？

候选人：
AST（抽象语法树）是源代码语法结构的一种抽象表示，在前端工程化中有广泛应用，比如代码转换、代码检查、代码压缩等。

让我从以下几个方面来说明：
1. AST 的生成过程
2. AST 的应用场景
3. 实际开发中的使用
````

```javascript
// 1. 简单的 AST 转换示例
const parser = require("@babel/parser");
const traverse = require("@babel/traverse").default;
const generate = require("@babel/generator").default;
const t = require("@babel/types");

// 源代码
const code = `
function square(n) {
  return n * n;
}
`;

// 解析为 AST
const ast = parser.parse(code);

// 遍历和修改 AST
traverse(ast, {
  // 访问函数声明
  FunctionDeclaration(path) {
    // 修改函数名
    path.node.id.name = "squared";
  },
  // 访问二元表达式
  BinaryExpression(path) {
    // 将乘法改为幂运算
    if (path.node.operator === "*") {
      path.replaceWith(
        t.callExpression(
          t.memberExpression(t.identifier("Math"), t.identifier("pow")),
          [path.node.left, t.numericLiteral(2)]
        )
      );
    }
  },
});

// 生成新代码
const output = generate(ast, {}, code);
console.log(output.code);
// 输出: function squared(n) { return Math.pow(n, 2); }

// 2. 自定义 Babel 插件示例
module.exports = function () {
  return {
    visitor: {
      // 移除 console.log
      CallExpression(path) {
        const callee = path.node.callee;
        if (
          t.isMemberExpression(callee) &&
          t.isIdentifier(callee.object, { name: "console" }) &&
          t.isIdentifier(callee.property, { name: "log" })
        ) {
          path.remove();
        }
      },

      // 添加函数调用追踪
      FunctionDeclaration(path) {
        const name = path.node.id.name;
        const body = path.node.body.body;

        body.unshift(
          t.expressionStatement(
            t.callExpression(
              t.memberExpression(
                t.identifier("console"),
                t.identifier("trace")
              ),
              [t.stringLiteral(`Entering function: ${name}`)]
            )
          )
        );
      },
    },
  };
};

// 3. ESLint 规则实现示例
module.exports = {
  create(context) {
    return {
      // 检查函数复杂度
      FunctionDeclaration(node) {
        const complexity = calculateComplexity(node);
        if (complexity > 10) {
          context.report({
            node,
            message: `Function is too complex (${complexity})`,
          });
        }
      },

      // 检查变量命名
      Identifier(node) {
        if (node.name.length < 2) {
          context.report({
            node,
            message: "Variable names should be longer than 1 character",
          });
        }
      },
    };
  },
};
```

AST 的应用场景：

1. 代码转换：

   - ES6+ 转 ES5
   - TypeScript 转 JavaScript
   - JSX 转 JavaScript

2. 代码分析：

   - 代码检查（ESLint）
   - 代码格式化（Prettier）
   - 代码高亮

3. 代码优化：
   - 死代码删除
   - 代码压缩
   - 自动引入依赖

最佳实践：

1. AST 操作：

   ```javascript
   // AST 节点创建
   const createImport = (source, specifiers) => {
     return t.importDeclaration(
       specifiers.map((name) =>
         t.importSpecifier(t.identifier(name), t.identifier(name))
       ),
       t.stringLiteral(source)
     );
   };

   // AST 节点判断
   const isConsoleCall = (path) => {
     return (
       t.isMemberExpression(path.node.callee) &&
       t.isIdentifier(path.node.callee.object, { name: "console" })
     );
   };
   ```

2. 性能优化：

   - 避免重复遍历
   - 缓存查询结果
   - 合理使用作用域

3. 调试技巧：
   - AST Explorer 工具
   - 断点调试
   - 日志输出
