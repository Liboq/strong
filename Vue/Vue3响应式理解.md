# 响应式理解

面试必问：首先查看官网 **[深入响应式系统 | Vue.js (vuejs.org)](https://cn.vuejs.org/guide/extras/reactivity-in-depth.html)**

响应式：变量的依赖发生变化时，变量也变化

## **两种劫持方式**

**getter/setter**

```
function ref(value) {
  const refObject = {
    get value() {
      track(refObject, 'value')
      return value
    },
    set value(newValue) {
      value = newValue
      trigger(refObject, 'value')
    }
  }
  return refObject
}
```

vue2的响应式和 vue3的ref都是基于 getter setter

**proxy**

```js
function reactive(obj) {
  return new Proxy(obj, {
    get(target, key) {
      track(target, key)
      return target[key]
    },
    set(target, key, value) {
      target[key] = value
      trigger(target, key)
    }
  })
}

```

proxy:用于reactive

在get 中执行追踪 set中执行触发

## 两种声明响应式状态的方法

**ref()**

主要用于基础数据类型

为什么要使用这个函数 ：依赖追踪 组件首次渲染时 会追踪渲染过程中的每个ref 改变了此ref 会触发它的组件重新渲染 

接收一个参数 将其包裹在一个带有 .value的属性中

模板字符串中会自动解包 不需要 带 .value,特殊情况`const count =ref({a:ref(1)}) ` 此时在 template中需要 `count.a.value` 才能访问值

非原始值会通过 `reactive`来转换响应式代理



**reactive**

主要用来包装 复杂数据类型

不需要使用.value 就可以使用 

局限性：

不能直接替换整个对象 ，这意味着 该变量将失去响应式，变量的引用地址发生了改变

例如：

```
let state = reactive({ count: 0 })

// 上面的 ({ count: 0 }) 引用将不再被追踪
// (响应性连接已丢失！)
state = reactive({ count: 1 })
```



**dom更新时机**

当修改了响应式状态后，dom会自动更新，dom更新不是同步的，vue会在“next tick"周期中缓冲所有的状态，确保不管做了多少次状态修改，只做一次dom更新，如果需要等待dom更新完成后再执行额外的代码可以使用nextTick（）

```
import { nextTick } from 'vue'

async function increment() {
  count.value++
  await nextTick()
  // 现在 DOM 已经更新了
}
```

