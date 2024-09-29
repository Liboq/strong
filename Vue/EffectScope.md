# effectScope 是干啥的

[Vue官网的effectScope](https://cn.vuejs.org/api/reactivity-advanced.html#effectscope)

> **创建一个 effect 作用域，可以捕获其中所创建的响应式副作用 (即计算属性和侦听器)，这样捕获到的副作用可以一起处理。对于该 API 的使用细节，请查阅对应的 [RFC](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0041-reactivity-effect-scope.md)。**

```typescript
function effectScope(detached?: boolean): EffectScope

interface EffectScope {
  run<T>(fn: () => T): T | undefined // 如果作用域不活跃就为 undefined
  stop(): void
}
```

`const scope = effectScope()`

scope可以运行函数，并将捕获函数同步执行期间创建的所有效果，包括在内部创建效果的任何 API，例如`computed`、`watch`和`watchEffect`,

scope.run方法还可返回执行函数的返回值

## 使用时机

在**组件外使用或者编写一个独立的包**时，需要对effect进行收集和管理并且对响应式进行去除，这个时候`EffectScope`就很方便了

- 创建scope环境收集effect
- 适当的时机去除effect，即stop，随之配套的还有 onScopeDispose 来监听 scope 的销毁、getCurrentScope() 获取当前活跃的 scope

## 简单的状态管理工具

```
// useState
import { effectScope } from '@vue/composition-api'

export default run => {
  let state
  const scope = effectScope(true)
  return () => {
    // 防止重复触发
    if (!state) {
      state = scope.run(run)
    }
    return state
  }
}

// store.js
import { computed, ref } from '@vue/composition-api'
import useState from './useState'

export default useState(
  () => {
    // state
    const count = ref(0)
    // getters
    const doubleCount = computed(() => count.value * 2)
    // actions
    function increment() {
      count.value++
    }
    return { count, doubleCount, increment }
  }
)


```

## useSyncExternalStore

```
useSyncExternalStore<any>(subscribe: (onStoreChange: () => void) => () => void, getSnapshot: () => any, getServerSnapshot?: (() => any) | undefined): any
```



useSyncExternalStore 是一个可以订阅外部 store 的 React Hook。如果是在服务端渲染中使用它，则会执行它的第三个参数 getServerSnapshot 来读取 store 的初始快照，如果是在客户端渲染中使用它，则会执行它的第二个参数 getSnapshot 来读取 store 的数据快照。并通过 Object.is 方法来判断前后读取的数据快照是否一致，如果不一致，则让组件重新渲染。