# 安全面试题

## 1. 常见攻击

### XSS 防范

- 概念解释：
  - XSS（跨站脚本攻击）是最常见的 Web 攻击之一
  - 分为存储型、反射型和 DOM 型 XSS
  - 通过注入恶意脚本来获取用户信息

```javascript
// 1. 输入过滤
function sanitizeInput(input) {
  return input.replace(/[&<>"']/g, char => ({
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&#39;'
  }[char]));
}

// 2. CSP配置
// 在服务器响应头中设置
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline';

// 3. HttpOnly Cookie
// Express中设置
app.use(session({
  cookie: {
    httpOnly: true,
    secure: true,
    sameSite: 'strict'
  }
}));
```

### CSRF 防护

- 概念解释：
  - CSRF（跨站请求伪造）利用用户已登录的身份
  - 通过伪造请求来执行未经授权的操作
  - 需要同时使用多种防护手段

```javascript
// 1. CSRF Token
// 服务端生成token
app.use((req, res, next) => {
  const token = crypto.randomBytes(16).toString('hex');
  req.session.csrfToken = token;
  res.locals.csrfToken = token;
  next();
});

// 前端发送请求
fetch('/api/data', {
  method: 'POST',
  headers: {
    'CSRF-Token': document.querySelector('meta[name="csrf-token"]').content
  },
  body: JSON.stringify(data)
});

// 2. SameSite Cookie
// 设置Cookie
Set-Cookie: session=123; SameSite=Strict; Secure

// 3. 验证请求来源
app.use((req, res, next) => {
  const origin = req.get('Origin');
  const referer = req.get('Referer');

  if (!origin && !referer) {
    return res.status(403).send('Invalid origin');
  }
  next();
});
```

### SQL 注入

- 概念解释：
  - SQL 注入通过构造特殊的输入来执行恶意 SQL 语句
  - 可能导致数据泄露或被破坏
  - 使用参数化查询和 ORM 可以有效防范

```javascript
// 1. 参数化查询
// 使用prepared statements
const mysql = require("mysql2/promise");

async function getUserById(id) {
  const connection = await mysql.createConnection({
    /*配置*/
  });
  try {
    const [rows] = await connection.execute(
      "SELECT * FROM users WHERE id = ?",
      [id]
    );
    return rows[0];
  } finally {
    await connection.end();
  }
}

// 2. ORM使用
// 使用Sequelize
const User = sequelize.define("User", {
  username: {
    type: DataTypes.STRING,
    validate: {
      isAlphanumeric: true,
    },
  },
});

// 安全的查询
const user = await User.findOne({
  where: {
    id: userId,
  },
});

// 3. 输入验证
function validateInput(input) {
  // 移除SQL关键字
  return input.replace(/[\0\x08\x09\x1a\n\r"'\\\%]/g, (char) => {
    switch (char) {
      case "\0":
        return "\\0";
      case "\x08":
        return "\\b";
      case "\x09":
        return "\\t";
      case "\x1a":
        return "\\z";
      case "\n":
        return "\\n";
      case "\r":
        return "\\r";
      case '"':
      case "'":
      case "\\":
      case "%":
        return "\\" + char;
    }
  });
}
```

## 2. 安全最佳实践

### HTTPS 原理

- 概念解释：
  - HTTPS 通过 SSL/TLS 协议加密 HTTP 通信
  - 使用非对称加密交换密钥，对称加密传输数据
  - 证书验证确保服务器身份

```javascript
// 1. Node.js HTTPS服务器
const https = require("https");
const fs = require("fs");

const options = {
  key: fs.readFileSync("private-key.pem"),
  cert: fs.readFileSync("certificate.pem"),
};

https
  .createServer(options, (req, res) => {
    res.writeHead(200);
    res.end("Hello Secure World\n");
  })
  .listen(443);

// 2. Express HTTPS配置
const express = require("express");
const https = require("https");
const app = express();

const server = https.createServer(
  {
    key: fs.readFileSync("private-key.pem"),
    cert: fs.readFileSync("certificate.pem"),
    ciphers: [
      "ECDHE-RSA-AES128-GCM-SHA256",
      "ECDHE-ECDSA-AES128-GCM-SHA256",
    ].join(":"),
  },
  app
);

// 3. HSTS配置
app.use((req, res, next) => {
  res.setHeader(
    "Strict-Transport-Security",
    "max-age=31536000; includeSubDomains; preload"
  );
  next();
});
```

### CSP 策略

- 概念解释：
  - CSP（内容安全策略）限制资源加载和脚本执行
  - 可以有效防止 XSS 和其他注入攻击
  - 通过 HTTP 头或 meta 标签配置

```javascript
// 1. 服务端配置
app.use((req, res, next) => {
  res.setHeader('Content-Security-Policy',
    "default-src 'self';" +
    "script-src 'self' 'unsafe-inline' 'unsafe-eval';" +
    "style-src 'self' 'unsafe-inline';" +
    "img-src 'self' data: https:;" +
    "font-src 'self';" +
    "connect-src 'self' https://api.example.com;"
  );
  next();
});

// 2. HTML meta标签
<meta
  http-equiv="Content-Security-Policy"
  content="default-src 'self'; img-src https://*; child-src 'none';"
>

// 3. 违规报告
app.post('/csp-report', (req, res) => {
  console.log('CSP违规:', req.body);
  res.status(204).end();
});
```

### 密码加密存储

- 概念解释：
  - 密码不能明文存储，需要使用加密算法
  - 使用加盐哈希增加安全性
  - 选择合适的哈希算法（如 bcrypt）

```javascript
// 1. 使用bcrypt
const bcrypt = require("bcrypt");

async function hashPassword(password) {
  const saltRounds = 10;
  try {
    const salt = await bcrypt.genSalt(saltRounds);
    const hash = await bcrypt.hash(password, salt);
    return hash;
  } catch (err) {
    throw new Error("Password hashing failed");
  }
}

async function verifyPassword(password, hash) {
  try {
    return await bcrypt.compare(password, hash);
  } catch (err) {
    throw new Error("Password verification failed");
  }
}

// 2. 密码重置流程
async function generateResetToken() {
  return crypto.randomBytes(32).toString("hex");
}

async function resetPassword(token, newPassword) {
  // 验证token
  const user = await User.findOne({
    resetToken: token,
    resetTokenExpiry: { $gt: Date.now() },
  });

  if (!user) {
    throw new Error("Invalid or expired reset token");
  }

  // 更新密码
  user.password = await hashPassword(newPassword);
  user.resetToken = undefined;
  user.resetTokenExpiry = undefined;
  await user.save();
}

// 3. 密码策略实施
function validatePassword(password) {
  const minLength = 8;
  const hasUpperCase = /[A-Z]/.test(password);
  const hasLowerCase = /[a-z]/.test(password);
  const hasNumbers = /\d/.test(password);
  const hasSpecialChar = /[!@#$%^&*]/.test(password);

  if (password.length < minLength) {
    throw new Error("Password must be at least 8 characters long");
  }

  if (!(hasUpperCase && hasLowerCase && hasNumbers && hasSpecialChar)) {
    throw new Error(
      "Password must contain uppercase, lowercase, numbers and special characters"
    );
  }

  return true;
}
```
