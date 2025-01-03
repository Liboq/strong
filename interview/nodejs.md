## Node.js 面试题

### 事件驱动模型

- 概念解释：
  - Node.js 使用事件驱动的非阻塞 I/O 模型
  - 通过 EventEmitter 类实现事件的发布订阅
  - 异步操作通过回调函数或 Promise 处理

```javascript
// 1. EventEmitter 基本使用
const EventEmitter = require("events");

class MyEmitter extends EventEmitter {
  constructor() {
    super();
    // 限制最大监听器数量
    this.setMaxListeners(20);
  }
}

const myEmitter = new MyEmitter();

// 注册事件监听器
myEmitter.on("event", (data) => {
  console.log("事件被触发:", data);
});

// 只监听一次
myEmitter.once("oneTimeEvent", () => {
  console.log("这个事件只会触发一次");
});

// 触发事件
myEmitter.emit("event", { message: "Hello" });

// 2. 自定义事件流
class DatabaseEmitter extends EventEmitter {
  query(sql) {
    // 模拟数据库查询
    this.emit("query", sql);

    setTimeout(() => {
      if (Math.random() > 0.5) {
        this.emit("success", { rows: [] });
      } else {
        this.emit("error", new Error("查询失败"));
      }
    }, 100);
  }
}

const db = new DatabaseEmitter();

db.on("query", (sql) => console.log("执行查询:", sql));
db.on("success", (result) => console.log("查询成功:", result));
db.on("error", (err) => console.error("查询错误:", err));
```

### Stream 流操作

- 概念解释：
  - Stream 是处理流式数据的抽象接口
  - 可以处理大文件，避免内存溢出
  - 分为：Readable、Writable、Duplex、Transform

```javascript
// 1. 文件流操作
const fs = require("fs");

// 读取流
const readStream = fs.createReadStream("big-file.txt", {
  highWaterMark: 64 * 1024, // 64KB chunks
});

// 写入流
const writeStream = fs.createWriteStream("output.txt");

// 管道操作
readStream.pipe(writeStream);

// 事件处理
readStream.on("data", (chunk) => {
  console.log("接收到数据:", chunk.length);
});

readStream.on("end", () => {
  console.log("读取完成");
});

// 2. 自定义转换流
const { Transform } = require("stream");

class UppercaseTransform extends Transform {
  _transform(chunk, encoding, callback) {
    // 转换为大写
    this.push(chunk.toString().toUpperCase());
    callback();
  }
}

const uppercaseTransform = new UppercaseTransform();

readStream.pipe(uppercaseTransform).pipe(writeStream);

// 3. 流的错误处理
readStream.on("error", (err) => {
  console.error("读取错误:", err);
});

writeStream.on("error", (err) => {
  console.error("写入错误:", err);
});

// 4. 背压(Backpressure)处理
const writeable = fs.createWriteStream("output.txt");

function write(data, cb) {
  if (!writeable.write(data)) {
    writeable.once("drain", cb);
  } else {
    process.nextTick(cb);
  }
}
```

### Buffer 的使用

- 概念解释：
  - Buffer 用于处理二进制数据
  - 在内存中分配固定大小的空间
  - 常用于文件操作和网络通信

```javascript
// 1. 创建 Buffer
// 分配指定大小的 Buffer
const buf1 = Buffer.alloc(10);

// 创建包含数据的 Buffer
const buf2 = Buffer.from("Hello World");

// 不安全的分配（可能包含旧数据）
const buf3 = Buffer.allocUnsafe(10);

// 2. Buffer 操作
// 写入数据
buf1.write("Hello");

// 读取数据
console.log(buf1.toString()); // Hello

// 复制 Buffer
const bufCopy = Buffer.alloc(buf2.length);
buf2.copy(bufCopy);

// 3. Buffer 转换
// 字符串转 Buffer
const strBuf = Buffer.from("你好", "utf8");

// Buffer 转字符串
console.log(strBuf.toString("utf8")); // 你好

// 4. Buffer 拼接
const chunks = [];
let totalLength = 0;

// 收集数据块
readStream.on("data", (chunk) => {
  chunks.push(chunk);
  totalLength += chunk.length;
});

// 拼接完整数据
readStream.on("end", () => {
  const result = Buffer.concat(chunks, totalLength);
  console.log(result.toString());
});
```
