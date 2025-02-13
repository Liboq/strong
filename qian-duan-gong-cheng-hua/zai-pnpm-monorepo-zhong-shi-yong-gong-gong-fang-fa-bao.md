---
description: 把项目的pc和app整合到一个文件中，方便管理，如何引用公共的方法呢，而不出现ts报错
cover: ../.gitbook/assets/文字排版设计 (1).png
coverY: 0
---

# 在 pnpm Monorepo 中使用公共方法包

在 `pnpm` Monorepo 中使用公共方法包

在一个 `pnpm` monorepo 中，可能会有多个项目共享一些通用的功能或方法。为了避免代码重复，可以将这些通用的方法提取到一个公共方法项目中，然后在其他项目中引用和使用它。本文将介绍如何在 Vue 项目中使用公共方法包。

#### 1. 设置公共方法项目

首先，我们需要创建一个公共方法项目，并确保它可以被其他项目依赖。假设你已经有了一个包含公共方法的项目（例如 `common-utils`），其目录结构可能如下：

```
/common-utils
  ├── package.json
  ├── index.js
  └── README.md
```

在 `common-utils/package.json` 文件中，你需要定义项目的基本信息：

```json
{
  "name": "common-utils",
  "version": "1.0.0",
  "main": "index.js"
}
```

`main` 字段指向公共方法包的入口文件。你可以在 `index.js` 中编写一些常用的方法。例如，一个简单的加法方法：

**`common-utils/index.js` 示例：**

```js
export function sum(a, b) {
  return a + b;
}

export function multiply(a, b) {
  return a * b;
}
```

#### 2. 在 Vue 项目中安装公共方法包

接下来，我们需要在 Vue 项目中使用 `pnpm` 安装并引用 `common-utils` 包。首先，确保你已经将 `common-utils` 包添加到你的 `pnpm` 工作空间中：

```bash
pnpm add common-utils --workspace
```

`--workspace` 参数告诉 `pnpm` 在工作区中查找并安装该包，而不是去外部的 npm registry 下载。

#### 3. 在 Vue 项目中引入并使用公共方法

安装完成后，你就可以在 Vue 项目中引用并使用 `common-utils` 中的方法了。例如，在某个 Vue 组件中使用 `sum` 方法：

**`src/components/SomeComponent.vue` 示例：**

```vue
<template>
  <div>
    <p>Sum of 2 and 3 is: {{ result }}</p>
  </div>
</template>

<script setup>
import { sum } from 'common-utils';

const result = sum(2, 3);
console.log(result); // 输出 5
</script>

<style scoped>
/* 样式 */
</style>
```

在上面的示例中，我们通过 `import { sum } from 'common-utils'` 导入了 `common-utils` 包中的 `sum` 函数，并在组件中使用了它。

#### 4. 确保路径和依赖关系正确

在 `pnpm` monorepo 中，工作空间（workspace）功能会自动管理项目间的依赖关系。因此，只要你正确地在 Vue 项目中引用了包名 `common-utils`，`pnpm` 会自动解决该包的依赖关系并将其引入 Vue 项目。

需要注意的是，`pnpm` 的工作空间依赖会根据 `pnpm-workspace.yaml` 文件中配置的路径进行管理，因此确保该配置正确。

#### 5. 编译与构建

如果你的公共方法项目使用了 TypeScript 或其他构建工具（例如 Rollup、Webpack）进行构建，确保项目已经按照正确的格式编译为可以供外部引用的包。例如，使用 `ES Module` 导出，或者编译为 CommonJS 格式。

**例如，`common-utils` 项目如果使用 TypeScript，可以在 `tsconfig.json` 中配置如下：**

```json
{
  "compilerOptions": {
    "module": "ESNext",
    "target": "ESNext",
    "moduleResolution": "node",
    "outDir": "./dist",
    "esModuleInterop": true
  },
  "include": ["src/**/*"]
}
```

#### 6. 处理版本控制和发布

为了确保公共方法项目能够始终保持最新，你可以使用 `pnpm` 的版本控制功能来管理各个包的版本。在 `pnpm` 中，工作区的包版本管理非常灵活，你可以随时调整某个包的版本，或者通过 `pnpm` 的 `workspace` 功能来更新所有包的版本。

如果你的 `common-utils` 项目修改了方法，记得更新 `version`，并在需要时重新执行 `pnpm update` 来同步更新到 Vue 项目中。

#### 总结

在 `pnpm` monorepo 中使用公共方法项目，可以有效地共享通用逻辑，减少代码重复，提升开发效率。只需确保：

1. 正确设置公共方法项目并将其发布为一个可以依赖的包。
2. 在需要使用这些方法的项目中安装并引用该包。
3. 确保构建和依赖关系正确配置，保证公共方法包能够被正确引用。

通过这种方式，你可以在多个项目之间共享和维护公共方法，保持代码的整洁与模块化。
