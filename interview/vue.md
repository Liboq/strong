## Vue 面试题


### 场景一：Vue2 和 Vue3 的区别

```text
面试官：你能说说 Vue2 和 Vue3 的主要区别吗？

候选人：
好的。我在实际工作中经历过从 Vue2 到 Vue3 的迁移，可以从几个方面来说：

首先是响应式系统的实现原理不同。Vue2 使用 Object.defineProperty 来劫持对象的属性，这导致了一些限制，比如无法检测对象属性的添加和删除，需要使用 Vue.set 来处理。而 Vue3 使用 Proxy 重写了响应式系统，可以直接监听整个对象，不仅性能更好，使用也更自然。

其次是组合式 API 的引入。在我们的项目中，之前用 Vue2 的选项式 API 时，相关的业务逻辑会分散在不同的选项中。比如说，一个用户管理的功能，代码会分散在 data、computed、methods 等不同位置。改用 Vue3 的组合式 API 后，我们可以按照功能模块来组织代码，大大提升了代码的可维护性。

最后是性能方面的提升。Vue3 重写了虚拟 DOM 的实现，增加了静态节点标记，我们的一个列表页面仅仅通过升级框架就获得了约 30% 的性能提升。同时 Tree-shaking 的支持也让我们的打包体积减小了很多。

当然，这些改进也带来了一些迁移成本。我们通过制定详细的迁移计划，先从一些非核心业务开始，逐步过渡到 Vue3，整个过程比较平滑。
```

### 场景二：组件通信方案

````
面试官：在你的项目中，一般用什么方式进行组件通信？

候选人：
这个要根据具体场景来选择。我可以举几个实际的例子：

对于父子组件通信，我们主要用 props 和 emit。比如我们的表单组件：
```vue
<!-- 父组件 -->
<template>
  <user-form
    :initial-data="userData"
    @submit="handleSubmit"
  />
</template>
````

这样可以保持数据流向清晰，容易调试和维护。

对于跨层级组件通信，如果是深层次的 props 传递，我们会使用 provide/inject。比如主题配置：

```vue
<!-- App.vue -->
<script setup>
import { provide, ref } from "vue";
const theme = ref("light");
provide("theme", theme);
</script>
```

而对于全局状态管理，我们使用 Pinia（之前是 Vuex）。特别是在处理用户认证、购物车这类需要在多个页面共享的数据时：

```javascript
// stores/cart.js
export const useCartStore = defineStore("cart", {
  state: () => ({
    items: [],
  }),
  actions: {
    addItem(item) {
      this.items.push(item);
    },
  },
});
```

在选择通信方案时，我们会考虑几个因素：组件之间的关系、数据的作用域、是否需要响应式等。选择合适的方案可以让代码更容易维护和扩展。

### 场景三：性能优化实践

````
面试官：你在 Vue 项目中做过哪些性能优化？

候选人：
在我负责的电商项目中，我们实施了多个层面的优化：

首先是代码层面。我们大量使用 computed 缓存计算结果，特别是在商品列表的筛选和排序功能中：
```javascript
export default {
  computed: {
    filteredProducts() {
      return this.products.filter(p =>
        p.price >= this.minPrice &&
        p.price <= this.maxPrice
      )
    }
  }
}
````

对于大型列表，我们实现了虚拟滚动。比如在订单列表页面，我们使用 vue-virtual-scroller：

```vue
<template>
  <virtual-scroller :items="orders" :item-height="60" v-slot="{ item }">
    <order-item :order="item" />
  </virtual-scroller>
</template>
```

在构建优化方面，我们：

1. 路由懒加载，将每个页面拆分成独立的 chunk
2. 第三方库按需引入，比如 Element Plus
3. 使用 keep-alive 缓存频繁切换的组件

这些优化让我们的首屏加载时间从 3s 降到了 1.8s，大大提升了用户体验。

当然，性能优化是个持续的过程。我们建立了性能监控系统，会根据实际数据持续优化。

```

### 场景四：watch vs computed
```

面试官：能说说 watch 和 computed 的区别，以及它们的使用场景吗？

候选人：
好的，我可以结合实际项目中的使用经验来说明。

computed 主要用于依赖数据的自动计算。比如在我们的商城项目中，购物车的总价计算：

```javascript
export default {
  data() {
    return {
      cartItems: [
        { price: 100, quantity: 2 },
        { price: 200, quantity: 1 },
      ],
    };
  },
  computed: {
    totalPrice() {
      return this.cartItems.reduce(
        (total, item) => total + item.price * item.quantity,
        0
      );
    },
  },
};
```

computed 有缓存机制，只有依赖数据变化时才会重新计算，这对性能很重要。

而 watch 主要用于响应数据的变化并执行异步或复杂的操作。比如我们的搜索功能：

```javascript
export default {
  data() {
    return {
      searchQuery: "",
      searchResults: [],
    };
  },
  watch: {
    searchQuery: {
      handler: async function (newQuery) {
        if (newQuery.length >= 2) {
          // 防抖处理
          clearTimeout(this.searchTimeout);
          this.searchTimeout = setTimeout(async () => {
            this.searchResults = await this.fetchSearchResults(newQuery);
          }, 300);
        }
      },
      immediate: true, // 立即执行一次
    },
  },
};
```

主要区别有：

1. 功能定位：

   - computed 是计算属性，用于简单的同步逻辑
   - watch 是监听器，用于复杂的异步逻辑

2. 性能表现：

   - computed 有缓存，依赖不变不会重新计算
   - watch 每次数据变化都会执行

3. 使用场景：
   - computed：购物车总价、列表过滤等
   - watch：表单验证、搜索请求、数据同步等

在实际开发中，我们会根据具体需求选择。比如需要数据联动时用 watch，需要数据计算时用 computed。

```

### 场景五：生命周期应用
```

面试官：你能结合实际项目，说说在不同的生命周期钩子中，一般会做什么操作吗？

候选人：
好的。在我们的项目中，不同的生命周期钩子有不同的应用场景：

在 created 中，我们主要做一些数据初始化和异步请求。比如我们的商品详情页：

```javascript
export default {
  data() {
    return {
      productInfo: null,
      loading: true,
    };
  },
  async created() {
    try {
      // 获取路由参数
      const { id } = this.$route.params;
      // 请求商品数据
      this.productInfo = await this.fetchProductDetail(id);
    } catch (error) {
      this.$message.error("商品信息获取失败");
    } finally {
      this.loading = false;
    }
  },
};
```

在 mounted 中，我们会做一些需要操作 DOM 的初始化，比如初始化第三方库：

```javascript
export default {
  mounted() {
    // 初始化图表
    this.chart = echarts.init(this.$refs.chartContainer);
    this.chart.setOption({
      /*...*/
    });

    // 添加事件监听
    window.addEventListener("resize", this.handleResize);
  },
  beforeDestroy() {
    // 清理工作
    window.removeEventListener("resize", this.handleResize);
    this.chart.dispose();
  },
};
```

在 updated 中，我们会处理一些数据更新后的操作，但要注意避免无限循环：

```javascript
export default {
  updated() {
    // 数据更新后，重新计算布局
    this.$nextTick(() => {
      if (this.needReLayout) {
        this.updateLayout();
      }
    });
  },
};
```

生命周期的使用要点：

1. created: 数据初始化、异步请求
2. mounted: DOM 操作、第三方库初始化
3. updated: 数据更新后的处理
4. beforeDestroy: 清理工作（事件监听、定时器等）

### 场景六：组件复用策略

````
面试官：在项目中，你是如何实现组件复用的？能具体说说你的实践经验吗？

候选人：
在我们的项目中，主要通过以下几种方式实现组件复用：

1. Mixins (Vue2) / Composables (Vue3)
比如我们封装了一个分页逻辑：
```javascript
// Vue3 Composable
export function usePagination() {
  const currentPage = ref(1)
  const pageSize = ref(10)
  const total = ref(0)

  const handlePageChange = (page) => {
    currentPage.value = page
    // 触发数据重新加载
    loadData()
  }

  return {
    currentPage,
    pageSize,
    total,
    handlePageChange
  }
}

// 组件中使用
export default {
  setup() {
    const { currentPage, handlePageChange } = usePagination()
    return { currentPage, handlePageChange }
  }
}
````

2. 高阶组件
   比如我们封装了一个带权限控制的组件：

```javascript
export function withPermission(WrappedComponent) {
  return {
    props: ["requiredPermission"],
    setup(props) {
      const hasPermission = computed(() => {
        return checkPermission(props.requiredPermission);
      });

      return () =>
        hasPermission.value ? h(WrappedComponent) : h("div", "无权限访问");
    },
  };
}
```

3. Slots 插槽
   比如我们的通用列表组件：

```vue
<!-- BaseList.vue -->
<template>
  <div class="list-container">
    <div class="list-header">
      <slot name="header"></slot>
    </div>
    <div class="list-content">
      <slot v-for="item in items" :item="item" name="item"></slot>
    </div>
    <div class="list-footer">
      <slot name="footer"></slot>
    </div>
  </div>
</template>

<!-- 使用组件 -->
<base-list :items="products">
  <template #header>
    <h2>商品列表</h2>
  </template>
  <template #item="{ item }">
    <product-card :product="item" />
  </template>
</base-list>
```

选择复用策略的考虑因素：

1. 复用的是逻辑还是 UI？逻辑用 Composables，UI 用组件
2. 是否需要高度定制？用插槽提供灵活性
3. 是否涉及横切关注点？用高阶组件处理

```

### 场景七：Vue Router 实践
```

面试官：能谈谈你在项目中是如何使用 Vue Router 的？包括路由守卫、懒加载等。

候选人：
好的，我来分享一下在我们项目中的实践经验。

1. 首先是路由配置，我们按照业务模块组织路由：

```javascript
// router/modules/user.js
export default {
  path: "/user",
  component: () => import("@/layouts/UserLayout"),
  children: [
    {
      path: "profile",
      component: () => import("@/views/user/Profile"),
      meta: {
        title: "个人中心",
        requiresAuth: true,
      },
    },
  ],
};
```

````
2. 路由守卫的使用，我们主要处理权限控制和页面标题：
```javascript
router.beforeEach(async (to, from, next) => {
  // 设置页面标题
  document.title = to.meta.title || '默认标题'

  // 权限检查
  if (to.meta.requiresAuth) {
    const isAuthenticated = await checkAuth()
    if (!isAuthenticated) {
      next({
        path: '/login',
        query: { redirect: to.fullPath }
      })
      return
    }
  }

  // 处理keepAlive
  if (to.meta.keepAlive) {
    store.commit('ADD_CACHED_VIEW', to.name)
  }

  next()
})
```

````

3. 对于复杂页面，我们使用路由元信息来控制页面行为：

```javascript
const route = {
  meta: {
    keepAlive: true, // 是否缓存
    roles: ["admin"], // 允许访问的角色
    breadcrumb: true, // 是否显示面包屑
    activeMenu: "/dashboard", // 菜单激活项
  },
};
```

路由最佳实践：

1. 按模块拆分路由配置
2. 使用路由守卫统一处理权限
3. 懒加载优化首屏加载
4. 利用元信息控制页面行为

### 场景八：状态管理方案

````
面试官：在大型项目中，你是如何进行状态管理的？能具体说说你的实践经验吗？

候选人：
在我们的项目中，根据不同场景选择不同的状态管理方案：

1. 对于简单的组件状态，使用 ref 和 reactive：
```javascript
// 商品详情组件
const product = ref({
  id: 1,
  name: '商品名称',
  price: 99
})

// 修改状态
const updatePrice = (newPrice) => {
  product.value.price = newPrice
}
````

````
2. 对于需要跨组件共享的状态，使用 Pinia：
```javascript
// stores/cart.js
export const useCartStore = defineStore('cart', {
  state: () => ({
    items: [],
    total: 0
  }),
  getters: {
    itemCount: (state) => state.items.length,
    totalPrice: (state) => state.items.reduce((sum, item) =>
      sum + item.price * item.quantity, 0
    )
  },
  actions: {
    async addToCart(product) {
      // 检查库存
      const stock = await checkStock(product.id)
      if (stock > 0) {
        this.items.push({
          ...product,
          quantity: 1
        })
        this.updateTotal()
      }
    },
    updateTotal() {
      this.total = this.totalPrice
    }
  }
})
```

````

3. 对于复杂的表单状态，使用 Composition API 封装：

```javascript
// composables/useForm.js
export function useForm(initialData) {
  const formData = ref(initialData);
  const errors = ref({});

  const validate = () => {
    // 表单验证逻辑
  };

  const submit = async () => {
    if (await validate()) {
      try {
        await saveData(formData.value);
        resetForm();
      } catch (error) {
        handleError(error);
      }
    }
  };

  return {
    formData,
    errors,
    validate,
    submit,
  };
}

// 组件中使用
const { formData, submit } = useForm({
  username: "",
  password: "",
});
```

状态管理的选择原则：

1. 基本类型优先使用 ref
2. 复杂对象使用 reactive
3. 需要解构的场景使用 ref
4. 配合 computed 时都可以使用

````

### 场景九：项目架构设计
+ ```
+ 面试官：你能分享一下在大型 Vue 项目中，是如何做架构设计的吗？
+
+ 候选人：
+ 好的。在我负责的一个企业级 SaaS 项目中，我们的架构设计主要考虑以下几个方面：
+
+ 1. 目录结构设计：
+ ```
+ ├── src/
+ │   ├── api/                # API 接口管理
+ │   ├── assets/             # 静态资源
+ │   ├── components/         # 公共组件
+ │   │   ├── basic/         # 基础组件
+ │   │   └── business/      # 业务组件
+ │   ├── composables/        # 组合式函数
+ │   ├── config/            # 配置文件
+ │   ├── layouts/           # 布局组件
+ │   ├── router/            # 路由配置
+ │   │   ├── modules/       # 路由模块
+ │   │   └── guards/        # 路由守卫
+ │   ├── stores/            # 状态管理
+ │   ├── styles/            # 样式文件
+ │   ├── utils/             # 工具函数
+ │   └── views/             # 页面组件
+ ```
+ ```
+ 2. 模块化设计，以用户管理模块为例：
+ ```javascript
+ // stores/modules/user.js
+ export const useUserStore = defineStore('user', {
+   state: () => ({
+     userInfo: null,
+     permissions: []
+   }),
+   actions: {
+     async login(credentials) {
+       // 登录逻辑
+     },
+     async getUserInfo() {
+       // 获取用户信息
+     }
+   }
+ })
+
+ // composables/useUser.js
+ export function useUser() {
+   const userStore = useUserStore()
+   const router = useRouter()
+
+   const logout = async () => {
+     await userStore.logout()
+     router.push('/login')
+   }
+
+   return {
+     userInfo: computed(() => userStore.userInfo),
+     logout
+   }
+ }
+ ```
+ ```
+ 3. 权限控制系统：
+ ```javascript
+ // utils/permission.js
+ export function checkPermission(permission) {
+   const userStore = useUserStore()
+   return userStore.permissions.includes(permission)
+ }
+
+ // 指令方式使用
+ app.directive('permission', {
+   mounted(el, binding) {
+     if (!checkPermission(binding.value)) {
+       el.parentNode?.removeChild(el)
+     }
+   }
+ })
+
+ // 组件中使用
+ <button v-permission="'user:edit'">编辑用户</button>
+ ```
+
+ 4. 错误处理机制：
+ ```javascript
+ // utils/error.js
+ export class BusinessError extends Error {
+   constructor(code, message) {
+     super(message)
+     this.code = code
+   }
+ }
+
+ // plugins/error-handler.js
+ export function setupErrorHandler(app) {
+   app.config.errorHandler = (err, vm, info) => {
+     if (err instanceof BusinessError) {
+       // 处理业务错误
+       showErrorMessage(err.message)
+     } else {
+       // 处理其他错误
+       console.error(err)
+       reportError(err)
+     }
+   }
+ }
+ ```
+
+ 架构设计的核心原则：
+ 1. 高内聚低耦合，每个模块职责单一
+ 2. 可扩展性，便于添加新功能
+ 3. 可维护性，代码结构清晰
+ 4. 复用性，提取公共逻辑
+ 5. 性能优化，按需加载
+ ```
+
```

### 场景十：Vue Router 原理解析

```
面试官：你能说说 Vue Router 的实现原理吗？特别是它是如何实现前端路由的？

候选人：
好的。Vue Router 主要有两种模式：Hash 模式和 History 模式。我可以结合实际经验来解释一下。

Hash 模式的原理是利用 URL 中的 hash（就是 # 后面的部分）来模拟一个完整的 URL。当 hash 改变时，页面不会重新加载，但是会触发 hashchange 事件。这种模式的好处是兼容性好，因为即使在老版本的浏览器中也可以正常工作。

比如说，当用户访问 http://example.com/#/about 时，实际的页面路径还是 http://example.com，浏览器不会向服务器发送新的请求，而是通过监听 hashchange 事件来处理路由变化。

History 模式则是利用了 HTML5 History API 中的 pushState 和 replaceState 方法。这种模式下的 URL 看起来更加自然，没有 # 号。比如 http://example.com/about。但是这种模式需要服务器的配置支持，否则用户直接访问 URL 时会出现 404 错误。

在我们的项目中，之前使用 Hash 模式是因为要兼容一些老系统，后来迁移到了 History 模式，主要考虑是：
1. URL 更加美观，没有 # 号
2. 可以获得更好的 SEO 效果
3. 可以充分利用 History API 的特性

在实现层面，Vue Router 主要做了这几件事：

1. 路由匹配：
当 URL 发生变化时，Vue Router 会根据路由表找到对应的组件。这个过程中会处理动态路由参数、嵌套路由等情况。

2. 组件渲染：
Vue Router 通过 `<router-view>` 组件来渲染匹配到的组件。它的实现原理是一个函数式组件，会根据当前路由状态动态渲染不同的组件。

3. 导航守卫：
路由变化时会触发一系列的导航守卫，这些守卫形成一个调用链：
```

导航触发 -> 失活组件守卫 -> 全局前置守卫 -> 组件内守卫 -> 全局解析守卫 -> 活动组件守卫

````

4. 路由记录：
   Vue Router 会维护一个路由栈，用于实现前进、后退功能。在 History 模式下，它会调用 History API：

```javascript
// 前进到新页面
history.pushState(state, "", url);

// 替换当前页面
history.replaceState(state, "", url);
```

````

### 场景十二：Hooks vs Mixins

```text
面试官：能谈谈 Composition API 中的 Hooks 和 Mixins 的区别吗？为什么说 Hooks 比 Mixins 更好？

候选人：
好的，我可以结合实际项目经验来说明。我们团队在从 Vue2 迁移到 Vue3 的过程中，就经历了从 Mixins 到 Hooks 的转变。

先说 Mixins 的问题：

1. 命名冲突：
```javascript
// userMixin.js
export default {
  data() {
    return {
      name: 'user'  // 可能与组件中的 name 冲突
    }
  },
  methods: {
    load() {  // 方法名也可能冲突
      // ...
    }
  }
}
````

2. 来源不清晰：

```javascript
// 使用多个 mixin 时，很难知道属性来自哪个 mixin
export default {
  mixins: [userMixin, logMixin, permissionMixin],
  methods: {
    async submit() {
      this.log(); // 这个方法来自哪个 mixin？
      await this.checkPermission(); // 这个呢？
    },
  },
};
```

而使用 Hooks（Composables）可以解决这些问题：

1. 数据来源清晰：

```javascript
// useUser.js
export function useUser() {
  const name = ref("user");
  const load = () => {
    // ...
  };
  return { name, load };
}

// useLog.js
export function useLog() {
  const log = (message) => {
    console.log(message);
  };
  return { log };
}

// 组件中使用
export default {
  setup() {
    const { name, load } = useUser();
    const { log } = useLog();

    return {
      name,
      load,
      log, // 清晰的数据来源
    };
  },
};
```

2.更好的类型推导：

```javascript
// useUser.ts
interface UserInfo {
  id: number;
  name: string;
}

export function useUser() {
  const userInfo = (ref < UserInfo) | (null > null);

  const loadUser = async (id: number) => {
    userInfo.value = await fetchUser(id);
  };

  return {
    userInfo,
    loadUser,
  };
}
```

1. 更灵活的组合方式：

```javascript
// 可以根据条件使用不同的逻辑
export default {
  setup(props) {
    // 基础功能
    const { user } = useUser();

    // 条件性地添加功能
    if (props.needLog) {
      const { log } = useLog();
      return { user, log };
    }

    return { user };
  },
};
```

在我们的实际项目中，使用 Hooks 的几个最佳实践：

1. 功能单一原则：每个 Hook 只负责一个功能

```javascript
// 好的实践
const { user } = useUser();
const { permission } = usePermission();

// 避免这样
const { user, permission, log } = useEverything();
```

2. 命名规范：使用 use 前缀

```javascript
// 好的命名
useUser();
usePermission();
useTheme();

// 避免这样
getUser();
permissionHook();
```

3. 返回值结构清晰：

```javascript
export function useCounter() {
  const count = ref(0);

  // 返回一个对象，包含状态和操作方法
  return {
    // 状态
    count,
    // 操作方法
    increment: () => count.value++,
    decrement: () => count.value--,
  };
}
```
