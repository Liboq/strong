---
description: >-
  技术栈：vue2.x + vuex +element-ui + vue-router2.x
  使用vue-router的addRoutes，优化公司后台项目的权限管理
---

# 🧐 权限管理(动态路由)

<pre class="language-markdown" data-full-width="false"><code class="lang-markdown"><strong>## 现状：
</strong>-router.js 所有路由
-resource.js 资源码文件
步骤：

路由拦截器中根据是否存在token来请求接口，获取用户所有资源码，若无token，返回到登录页
获取到资源码后，使用vuex储存有权限的路由并生成 menu菜单导航

缺点:
只是单纯控制了菜单，没有对路由进行权限控制
解决方案：
使用vue-router的 addRouters 对路由进行动态添加，
步骤: 
</code></pre>

### 路由

在路由中添加字段 `btnCodes` 表示 该页面下的权限按钮的内容 `code`表示路由权限码

```javascript
btncodes:{code:'',name:''}
```

### 路由守卫

> 使用\`vuex\`对获取到资源码进行处理，获取到最新的`addRouters` 然后添加到路由中

在使用动态添加路由addRoutes()会遇到下面的情况：&#x20;

在addRoutes()之后第一次访问被添加的路由会白屏，这是因为刚刚addRoutes()就立刻访问被添加的路由，然而此时addRoutes()没有执行结束，因而找不到刚刚被添加的路由导致白屏。因此需要从新访问一次路由才行。

该如何解决这个问题 ? 此时就要使用next({ ...to, replace: true })来确保addRoutes()时动态添加的路由已经被完全加载上去。

next({ ...to, replace: true })中的replace: true只是一个设置信息，告诉VUE本次操作后，不能通过浏览器后退按钮，返回前一个路由。

因此next({ ...to, replace: true })可以写成next({ ...to })，不过你应该不希望用户在addRoutes()还没有完成的时候，可以点击浏览器回退按钮.

```javascript
import store from '../store/index'
import { getResources} from '@/api'
import { Message } from 'ELEMENT'
import { errorRouter, homeRouter } from './routes'

// 路由守卫
export default function guards(router) {
  router.beforeEach(async (to, from, next) => {
    const token = localStorage.getItem('token')
    console.log(to)
    if (to.path === '/user/login') {
      next()
    } else {
      if (token) {
        if (store.state.permission.registerRouteFresh) {
            const result =await getResources()
              
              if (result.code === 200) {
                const resources = result.data || []
                store.dispatch('GenerateRoutes', resources).then(() => {
                  const currentAddRouter = [{...homeRouter,children:[...store.getters.addRouters]},{...errorRouter}]
                  router.addRoutes([...currentAddRouter])
                  store.commit('SET_REGISTER_ROUTE_FRESH', false)
                  next({ ...to, replace: true })
                })
              } else {
                Message.error('菜单数据获取失败, 请刷新页面')
              }
          
        }
        next()
      } else {
        next('/user/login')
      }
    }
    // 兜底执行
    next()
  })
}

```

### store

使用`vuex` 对路由权限 和权限码进行存储

<pre class="language-javascript"><code class="lang-javascript">import {asyncRouterMap, constantRouterMap } from '@/pages/mall/router/routes'
<strong>import { deepCopy,getPermissionRouter } from '@/utils'
</strong>
function contrast(a, b) {
  if (a.code == b.code &#x26;&#x26; a.path &#x26;&#x26; !a.isProhibit) {
    a.display = true
  }
  if (a.children &#x26;&#x26; a.children.length > 0) {
    a.children.forEach(item => {
      contrast(item, b)
    })
  }
}

function setFlagRouter(routerMap, resources) {
  resources.forEach(res => {
    routerMap.forEach(route => {
      contrast(route, res)
    })
  })
}

function filterRouter(routers) {
  if (!routers.length) {
    return
  }

  let i = routers.length
  while (i--) {
    if (!routers[i].display) {
      continue
    }

    if (routers[i].children &#x26;&#x26; routers[i].children.length > 0) {
      if (!routers[i].hasChildren) {
        continue
      }
      let arr = routers[i].children
      let flag = true
      for (let k = 0; k &#x3C; arr.length; k++) {
        if (arr[k].display) {
          flag = false
          break
        }
      }
      if (flag) {
        routers[i].display = false
      } else {
        filterRouter(routers[i].children)
      }
    }
  }
}

const permission = {
  state: {
    routers: constantRouterMap,
    resources: [],
    registerRouteFresh: true,
    addRouters: null
  },
  mutations: {
    SET_REGISTER_ROUTE_FRESH: (state, flag) => {
      state.registerRouteFresh = flag
    },
    SET_RESOURCES: (state, resources) => {
      state.resources = resources
    },
    SET_ROUTERS: (state, routers) => {
      state.addRouters = routers
      state.routers = constantRouterMap.concat(routers)
    },
  },
  actions: {
    GenerateRoutes({ commit }, data) {
      return new Promise(resolve => {
        let routerMap = deepCopy(asyncRouterMap )
        if (data.length) {
          setFlagRouter(routerMap, data)
          filterRouter(routerMap)
        }
        const accessedRouters = getPermissionRouter(routerMap, data)
        commit('SET_RESOURCES', data)
        commit('SET_ROUTERS', accessedRouters)
        resolve()
      })
    }
  }
}

export default permission

</code></pre>

### 按钮权限

> 自定义指令v-permission

```javascript
import store from '../store'

const permissionDirective = {
  componentUpdated: function (el, binding, vnode) {
    let timer = null
    timer = setInterval(() => {
      if (!store.state.permission.registerRouteFresh) {
        const { value } = binding
        // 获取没有权限资源码列表数据
        const points = store.state.permission.resources
        if (value && value instanceof Array) {
          console.log(value);
          
          匹配对应的指令
          const hasPermission = points.some((point) => {
            return value.includes(point)
          })
          // 如果匹配, 则表示当前用户无该权限, 那么删除对应的功能按钮
          if (!hasPermission) {
            el.parentNode && el.parentNode.removeChild(el)
          }
        } else {
          throw new Error('v-premission Error')
        }
        clearInterval(timer)
      }
    }, 100)
  },
  unbind: function (el, binding) {}
}

export function setPermissionDirective(app) {
  app.directive('permission', permissionDirective)
}

export default permissionDirective

```
