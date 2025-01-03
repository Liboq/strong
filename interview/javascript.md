## JavaScript 面试题

### 场景一：闭包的理解和应用

```text
面试官：能谈谈你对闭包的理解吗？在实际项目中是如何使用的？

候选人：
好的,我来从概念和实践两个角度来说明。

闭包本质上是一个函数能够访问其定义时所在的词法作用域,即使这个函数在其他作用域中执行。这让我们可以:

1. 创建私有变量和方法
2. 保持数据在内存中
3. 实现数据的封装

在实际项目中,我们经常用闭包来处理以下场景：
```

```javascript
// 1. 创建私有状态
function createCounter() {
  let count = 0; // 私有变量

  return {
    increment() {
      count++;
      return count;
    },
    decrement() {
      count--;
      return count;
    },
    getCount() {
      return count;
    },
  };
}

const counter = createCounter();
console.log(counter.getCount()); // 0
counter.increment(); // 1
counter.increment(); // 2
console.log(counter.getCount()); // 2

// 2. 柯里化和函数式编程
function multiply(a) {
  return function (b) {
    return a * b;
  };
}

const multiplyByTwo = multiply(2);
console.log(multiplyByTwo(4)); // 8
console.log(multiplyByTwo(5)); // 10

// 3. 防抖函数实现
function debounce(fn, delay) {
  let timer = null;

  return function (...args) {
    if (timer) clearTimeout(timer);

    timer = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}

const debouncedSearch = debounce((query) => {
  // 执行搜索
  console.log("Searching:", query);
}, 300);
```

使用闭包需要注意：

1. 内存管理：

   - 闭包会保持对外部变量的引用
   - 注意及时清理不需要的闭包
   - 避免创建过多闭包

2. 性能考虑：

   - 合理使用闭包
   - 避免过度使用
   - 注意内存泄漏

3. 代码可读性：
   - 明确闭包的用途
   - 适当的注释说明
   - 遵循命名规范

### 场景二：this 指向问题

```text
面试官：能详细说说 JavaScript 中 this 的指向问题吗？在实际开发中如何正确处理 this 绑定？

候选人：
好的，this 的指向是 JavaScript 中比较容易混淆的概念。我们可以从几个方面来理解：

1. this 的指向取决于函数的调用方式，而不是定义方式
2. 有四种基本的绑定规则：默认绑定、隐式绑定、显式绑定和 new 绑定
3. 箭头函数的 this 有特殊的处理规则

让我用具体的代码来说明：
```

```javascript
// 1. 默认绑定（非严格模式下指向全局对象，严格模式下指向 undefined）
function showThis() {
  console.log(this);
}
showThis(); // window 或 undefined

// 2. 隐式绑定（this 指向调用该方法的对象）
const user = {
  name: "张三",
  greet() {
    console.log(`你好，我是 ${this.name}`);
  },
  friend: {
    name: "李四",
    greet() {
      console.log(`你好，我是 ${this.name}`);
    },
  },
};

user.greet(); // "你好，我是 张三"
user.friend.greet(); // "你好，我是 李四"

// 3. 显式绑定（使用 call、apply、bind）
function introduce(age, hobby) {
  console.log(`我是 ${this.name}，今年 ${age} 岁，爱好是 ${hobby}`);
}

const person = { name: "王五" };

// call 方式
introduce.call(person, 25, "读书");

// apply 方式
introduce.apply(person, [25, "读书"]);

// bind 方式
const boundIntroduce = introduce.bind(person);
boundIntroduce(25, "读书");

// 4. new 绑定
function Person(name) {
  this.name = name;
  this.sayHi = function () {
    console.log(`Hi, I'm ${this.name}`);
  };
}

const person1 = new Person("赵六");
person1.sayHi(); // "Hi, I'm 赵六"

// 5. 箭头函数（this 由定义时的上下文决定）
const obj = {
  name: "小明",
  sayHiArrow: () => {
    console.log(`Hi, ${this.name}`);
  },
  sayHiRegular() {
    setTimeout(() => {
      console.log(`Hi, ${this.name}`);
    }, 100);
  },
};

obj.sayHiArrow(); // "Hi, undefined"（this 指向全局）
obj.sayHiRegular(); // "Hi, 小明"（this 指向 obj）
```

在实际开发中的最佳实践：

1. 使用场景：

   - 方法中的 this：使用常规函数
   - 回调函数中的 this：使用箭头函数
   - 事件处理器：注意绑定问题

2. 常见陷阱：

   - 回调函数丢失 this
   - 方法作为参数传递
   - DOM 事件处理器

3. 解决方案：
   - 使用箭头函数
   - 使用 bind 方法
   - 使用类字段语法

### 场景三：原型链和继承

```text
面试官：能详细讲讲 JavaScript 中的原型链和继承机制吗？在实际开发中如何实现继承？

候选人：
好的，JavaScript 的继承主要是通过原型链来实现的。每个对象都有一个原型对象，对象会从原型"继承"属性和方法。

我们可以从以下几个方面来理解：
1. 原型链的基本概念
2. 不同的继承实现方式
3. ES6 class 语法的底层实现

让我用代码来展示：
```

```javascript
// 1. 原型链基础
function Animal(name) {
  this.name = name;
}

Animal.prototype.sayName = function () {
  console.log(`我是 ${this.name}`);
};

const cat = new Animal("小猫");
cat.sayName(); // "我是 小猫"
console.log(cat.__proto__ === Animal.prototype); // true
console.log(Animal.prototype.__proto__ === Object.prototype); // true

// 2. 构造函数继承
function Dog(name, breed) {
  Animal.call(this, name); // 继承属性
  this.breed = breed;
}

// 原型链继承
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function () {
  console.log("汪汪汪！");
};

const dog = new Dog("小狗", "柴犬");
dog.sayName(); // "我是 小狗"
dog.bark(); // "汪汪汪！"

// 3. ES6 class 实现
class Pet {
  constructor(name) {
    this.name = name;
  }

  sayName() {
    console.log(`我是 ${this.name}`);
  }

  static createPet(name) {
    return new Pet(name);
  }
}

class Rabbit extends Pet {
  constructor(name, color) {
    super(name);
    this.color = color;
  }

  jump() {
    console.log(`${this.color}色的${this.name}跳了一下`);
  }
}

const rabbit = new Rabbit("小兔", "白");
rabbit.sayName(); // "我是 小兔"
rabbit.jump(); // "白色的小兔跳了一下"
```

在实际开发中的最佳实践：

1. 继承方式选择：

   - 优先使用 ES6 class
   - 需要时使用组合而非继承
   - 避免过深的继承层级

2. 原型链注意事项：

   - 属性和方法的查找过程
   - 原型污染问题
   - 性能考虑

3. 实现技巧：

   - 合理使用 super
   - 静态方法继承
   - 私有属性处理

4. 常见问题：
   - 继承中的 this 绑定
   - 属性遮蔽
   - 方法重写

### 场景四：异步编程和事件循环

```text
面试官：能详细讲讲 JavaScript 中的异步编程和事件循环机制吗？在实际项目中是如何处理异步操作的？

候选人：
好的，JavaScript 的异步编程是处理非阻塞操作的核心机制。事件循环则是实现异步的基础。

我们可以从以下几个方面来理解：
1. 事件循环的基本原理
2. 不同的异步处理方式
3. 宏任务和微任务的区别

让我用代码来展示：
```

```javascript
// 1. Promise 的基本使用
function fetchUserData(userId) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const user = {
        id: userId,
        name: "张三",
        age: 25,
      };
      resolve(user);
    }, 1000);
  });
}

// Promise 链式调用
fetchUserData(1)
  .then((user) => {
    console.log("用户数据:", user);
    return fetchUserOrders(user.id);
  })
  .then((orders) => {
    console.log("订单数据:", orders);
  })
  .catch((error) => {
    console.error("错误:", error);
  });

// 2. async/await 使用
async function getUserInfo(userId) {
  try {
    const user = await fetchUserData(userId);
    const orders = await fetchUserOrders(user.id);
    const details = await fetchUserDetails(user.id);

    return {
      ...user,
      orders,
      details,
    };
  } catch (error) {
    console.error("获取用户信息失败:", error);
    throw error;
  }
}

// 3. 并发控制
async function fetchAllUsers(userIds) {
  // 并行请求
  const promises = userIds.map((id) => fetchUserData(id));
  const users = await Promise.all(promises);
  return users;
}

// 限制并发数的请求
async function fetchUsersWithLimit(userIds, limit = 3) {
  const results = [];
  const executing = new Set();

  for (const id of userIds) {
    if (executing.size >= limit) {
      await Promise.race(executing);
    }

    const promise = fetchUserData(id).then((result) => {
      executing.delete(promise);
      return result;
    });

    executing.add(promise);
    results.push(promise);
  }

  return Promise.all(results);
}

// 4. 事件循环示例
console.log("1");

setTimeout(() => {
  console.log("2");
}, 0);

Promise.resolve().then(() => {
  console.log("3");
});

console.log("4");

// 输出顺序：1, 4, 3, 2
```

在实际开发中的最佳实践：

1. 异步处理方式：

   - 优先使用 async/await
   - 合理使用 Promise
   - 避免回调地狱

2. 错误处理：

   - try/catch 捕获异常
   - Promise 错误链
   - 全局错误处理

3. 性能优化：

   - 并发请求控制
   - 缓存结果
   - 请求取消

4. 调试技巧：
   - 异步堆栈追踪
   - 性能分析
   - 内存泄漏检测

### 场景五：Promise 和 async/await 深入理解

```text
面试官：能深入讲讲你对 Promise 和 async/await 的理解吗？它们的原理和使用场景是什么？

候选人：
好的，Promise 是异步编程的一种解决方案，而 async/await 是基于 Promise 的更优雅的异步处理方式。

我们可以从以下几个方面来理解：
1. Promise 的状态和特性
2. async/await 的执行原理
3. 两者在实际应用中的差异

让我用代码来详细说明：
```

```javascript
// 1. Promise 的基本特性
const promise = new Promise((resolve, reject) => {
  // Promise 有三种状态：pending, fulfilled, rejected
  // 状态一旦改变就不能再变
  setTimeout(() => {
    const random = Math.random();
    if (random > 0.5) {
      resolve("成功"); // 变为 fulfilled
    } else {
      reject("失败"); // 变为 rejected
    }
  }, 1000);
});

// Promise 的链式调用
promise
  .then((result) => {
    console.log(result);
    return "处理后的" + result; // 返回新的 Promise
  })
  .catch((error) => {
    console.error(error);
    throw new Error("处理后的" + error); // 抛出错误
  })
  .finally(() => {
    console.log("无论成功失败都会执行");
  });

// 2. Promise 的常用方法
// 并行执行
Promise.all([fetch("/api/users"), fetch("/api/orders"), fetch("/api/products")])
  .then(([users, orders, products]) => {
    // 所有请求都成功才会执行
  })
  .catch((error) => {
    // 任一请求失败就会执行
  });

// 竞态执行
Promise.race([fetch("/api/fast"), fetch("/api/slow")]).then((result) => {
  // 最快的请求完成时执行
});

// 3. async/await 的使用
async function handleUserData() {
  try {
    // await 会暂停函数执行，等待 Promise 完成
    const user = await fetchUser();

    // 错误处理更直观
    if (!user.isActive) {
      throw new Error("用户未激活");
    }

    // 可以使用同步的写法处理异步逻辑
    const [orders, profile] = await Promise.all([
      fetchOrders(user.id),
      fetchProfile(user.id),
    ]);

    return {
      user,
      orders,
      profile,
    };
  } catch (error) {
    // 统一的错误处理
    console.error("处理用户数据失败:", error);
    throw error;
  }
}

// 4. async/await 的一些特殊情况
async function example() {
  // 并行执行优化
  const ordersPromise = fetchOrders();
  const productsPromise = fetchProducts();

  // 等待所有请求完成
  const [orders, products] = await Promise.all([
    ordersPromise,
    productsPromise,
  ]);

  // await 在循环中的使用
  for (const order of orders) {
    // 注意：这样会串行执行
    await processOrder(order);
  }

  // 并行执行的正确方式
  await Promise.all(orders.map(processOrder));
}
```

在实际开发中的最佳实践：

1. Promise 使用场景：

   - 封装异步操作
   - 并行处理
   - 错误处理链

2. async/await 使用场景：

   - 顺序执行异步操作
   - 复杂的异步流程
   - 更清晰的错误处理

3. 性能考虑：

   - 合理使用 Promise.all
   - 避免串行执行
   - 注意内存泄漏

4. 常见陷阱：
   - Promise 的状态不可逆
   - await 的阻塞效应
   - 未捕获的异常

### 场景六：事件机制和事件委托

```text
面试官：能详细讲讲 JavaScript 的事件机制吗？事件委托的原理是什么？在实际项目中如何应用？

候选人：
好的，JavaScript 的事件机制包括事件捕获和冒泡两个阶段，而事件委托则是基于事件冒泡的一种优化方案。

我们可以从以下几个方面来理解：
1. 事件流的三个阶段
2. 事件委托的实现原理
3. 实际应用中的最佳实践

让我用代码来详细说明：
```

```javascript
// 1. 事件流的基本概念
const parent = document.querySelector(".parent");
const child = document.querySelector(".child");

// 捕获阶段
parent.addEventListener(
  "click",
  (e) => {
    console.log("父元素捕获:", e.target.className);
  },
  true
);

// 冒泡阶段
parent.addEventListener("click", (e) => {
  console.log("父元素冒泡:", e.target.className);
});

child.addEventListener("click", (e) => {
  console.log("子元素冒泡:", e.target.className);
  // e.stopPropagation(); // 阻止冒泡
});

// 2. 事件委托的实现
const list = document.querySelector(".list");

list.addEventListener("click", (e) => {
  // 检查点击的是否是目标元素
  if (e.target.matches("li")) {
    const id = e.target.dataset.id;
    console.log("点击了列表项:", id);
  }
});

// 动态添加列表项
function addItem(id, text) {
  const li = document.createElement("li");
  li.dataset.id = id;
  li.textContent = text;
  list.appendChild(li);
}

// 3. 自定义事件
class EventEmitter {
  constructor() {
    this.events = {};
  }

  on(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
  }

  emit(event, data) {
    const callbacks = this.events[event];
    if (callbacks) {
      callbacks.forEach((callback) => callback(data));
    }
  }

  off(event, callback) {
    const callbacks = this.events[event];
    if (callbacks) {
      this.events[event] = callbacks.filter((cb) => cb !== callback);
    }
  }
}

// 使用示例
const emitter = new EventEmitter();

emitter.on("userLogin", (user) => {
  console.log("用户登录:", user);
  updateUI(user);
});

// 4. 性能优化
function debounceEvent(fn, delay = 300) {
  let timer = null;

  return function (...args) {
    if (timer) clearTimeout(timer);

    timer = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}

// 使用防抖处理滚动事件
window.addEventListener(
  "scroll",
  debounceEvent(() => {
    console.log("处理滚动事件");
  })
);
```

在实际开发中的最佳实践：

1. 事件绑定：

   - 合理使用事件委托
   - 及时解绑事件
   - 避免内存泄漏

2. 性能优化：

   - 使用事件委托减少事件绑定
   - 合理使用防抖和节流
   - 避免频繁操作 DOM

3. 事件处理：

   - 合理使用事件对象
   - 正确处理事件冒泡
   - 注意事件顺序

4. 常见问题：
   - 事件绑定的 this 指向
   - 事件对象的兼容性
   - 事件委托的边界处理

### 场景七：call、bind、apply 的原理和应用

```text
面试官：能详细讲讲 call、bind、apply 这三个方法的区别和使用场景吗？能手写实现一下吗？

候选人：
好的，这三个方法都是用来改变函数执行时的上下文（this 指向），但使用方式和场景有所不同：

1. call：立即执行，参数列表形式
2. apply：立即执行，参数数组形式
3. bind：返回新函数，不立即执行

让我用代码来详细说明：
```

```javascript
// 1. 基本使用对比
function greet(greeting, punctuation) {
  console.log(`${greeting}, ${this.name}${punctuation}`);
}

const person = { name: "张三" };

// call 方式
greet.call(person, "Hello", "!"); // "Hello, 张三!"

// apply 方式
greet.apply(person, ["Hi", "..."]); // "Hi, 张三..."

// bind 方式
const boundGreet = greet.bind(person);
boundGreet("Hey", "?"); // "Hey, 张三?"

// 2. 手写实现 call
Function.prototype.myCall = function (context, ...args) {
  // 处理 null 或 undefined 的情况
  context = context || window;

  // 创建唯一的属性名，避免覆盖原有属性
  const fnSymbol = Symbol("fn");

  // 将函数设为对象的属性
  context[fnSymbol] = this;

  // 执行函数
  const result = context[fnSymbol](...args);

  // 删除临时属性
  delete context[fnSymbol];

  return result;
};

// 3. 手写实现 apply
Function.prototype.myApply = function (context, argsArray = []) {
  context = context || window;
  const fnSymbol = Symbol("fn");

  context[fnSymbol] = this;
  const result = context[fnSymbol](...argsArray);
  delete context[fnSymbol];

  return result;
};

// 4. 手写实现 bind
Function.prototype.myBind = function (context, ...args1) {
  const fn = this;

  return function (...args2) {
    // 支持函数柯里化，合并参数
    return fn.apply(context, [...args1, ...args2]);
  };
};

// 5. 实际应用场景
// 5.1 继承实现
function Animal(name) {
  this.name = name;
}

function Dog(name, breed) {
  Animal.call(this, name); // 继承属性
  this.breed = breed;
}

// 5.2 方法借用
const numbers = { 0: 1, 1: 2, 2: 3, length: 3 };
Array.prototype.slice.call(numbers); // 类数组转数组

// 5.3 参数处理
function sum() {
  return Array.prototype.reduce.call(arguments, (sum, num) => sum + num, 0);
}

// 5.4 上下文绑定
class Counter {
  constructor() {
    this.count = 0;
    // 绑定方法到实例
    this.increment = this.increment.bind(this);
  }

  increment() {
    this.count++;
  }
}
```

使用时的注意事项：

1. 性能考虑：

   - bind 创建新函数，注意内存
   - call/apply 立即执行，适合一次性调用
   - 避免频繁绑定

2. 使用场景：

   - call：参数明确，一次性调用
   - apply：参数数组，动态参数
   - bind：需要重复使用，异步调用

3. 常见问题：

   - this 绑定丢失
   - 箭头函数无法改变 this
   - 原型链方法继承

4. 最佳实践：
   - 优先使用 class 语法
   - 合理使用箭头函数
   - 注意内存管理
