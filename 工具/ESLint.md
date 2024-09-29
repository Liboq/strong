# Eslint

## 配置

首先需要安装依赖



## 问题

1.点击两次`ctrl +s`,会触发两次保存

2.prettier格式化问题

![image-20231220135017835](https://pikachu-2022-1305579406.cos.ap-nanjing.myqcloud.com/markdown/image-20231220135017835.png)

## eslint与prettier冲突

冲突太多了，好麻烦啊，真不好处理，烦得很





## 推荐配置

### Vue2 + js

#### .eslintrc.js

```
/* eslint-env node */
module.exports = {
  // 'plugin:prettier/recommended'
  extends: [
    'eslint:recommended',
    'plugin:vue/recommended',
    'plugin:prettier/recommended'
  ],
  globals: {
    inf: true
  },
  parserOptions: {
    ecmaVersion: 2020,
    sourceType: 'module',
    allowImportExportEverywhere: true
  },
  parser: 'vue-eslint-parser',
  ignorePatterns: ['**/build/', '**/static/*.js'],
  rules: {
    'prettier/prettier': 'error',
    // allow async-await
    'generator-star-spacing': 'off',
    // allow debugger during development
    'no-debugger': process.env.NODE_ENV === 'production' ? 'error' : 'off',
    // https://eslint.vuejs.org/rules/html-indent.html
    'vue/html-indent': 0,
    // https://eslint.vuejs.org/rules/max-attributes-per-line.html
    'vue/max-attributes-per-line': 0,
    // https://eslint.vuejs.org/rules/html-self-closing.html
    'vue/html-self-closing': 0,
    // https://eslint.vuejs.org/rules/script-indent.html#options
    // "vue/script-indent": ["error", 'tab'],
    // https://eslint.vuejs.org/rules/html-closing-bracket-newline.html
    'vue/html-closing-bracket-newline': 0,
    // https://eslint.vuejs.org/rules/component-tags-order.html
    'vue/component-tags-order': 0,
    // https://eslint.vuejs.org/rules/singleline-html-element-content-newline.html
    'vue/singleline-html-element-content-newline': 0
  }
}

```

#### .prettierrc

```
{
  "useTabs": false,
  "tabWidth": 2,
  "printWidth": 80,
  "singleQuote": true,
  "trailingComma": "none",
  "bracketSpacing": true,
  "semi": false,
  "arrowParens": "avoid"
}
```

#### .vscode/setting.json

```json
{
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
}
```
