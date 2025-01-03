# CSS 面试题

## 1. 布局相关

### Flex 布局的核心概念

#### 容器属性：

```css
.container {
  display: flex;
  /* 主轴方向 */
  flex-direction: row | row-reverse | column | column-reverse;
  /* 是否换行 */
  flex-wrap: nowrap | wrap | wrap-reverse;
  /* 主轴对齐 */
  justify-content: flex-start | flex-end | center | space-between | space-around;
  /* 交叉轴对齐 */
  align-items: flex-start | flex-end | center | baseline | stretch;
  /* 多行对齐 */
  align-content: flex-start | flex-end | center | space-between | space-around |
    stretch;
}
```

#### 项目属性：

```css
.item {
  /* 排序 */
  order: 0;
  /* 放大比例 */
  flex-grow: 0;
  /* 缩小比例 */
  flex-shrink: 1;
  /* 基准大小 */
  flex-basis: auto;
  /* flex-grow, flex-shrink 和 flex-basis的简写 */
  flex: 0 1 auto;
  /* 单个项目对齐方式 */
  align-self: auto | flex-start | flex-end | center | baseline | stretch;
}
```

### Grid 布局与 Flex 的区别

- Grid 是二维布局，Flex 是一维布局
- Grid 布局示例：

```css
.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: 100px auto;
  grid-gap: 20px;
}

.item {
  grid-column: span 2;
  grid-row: 1 / 3;
}
```

### 居中对齐的实现方式

```css
/* 1. Flex方式 */
.flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}

/* 2. Grid方式 */
.grid-center {
  display: grid;
  place-items: center;
}

/* 3. 绝对定位 + transform */
.absolute-center {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

/* 4. 绝对定位 + margin */
.absolute-margin {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  margin: auto;
}

/* 5. table-cell方式 */
.table-cell {
  display: table-cell;
  vertical-align: middle;
  text-align: center;
}
```

## 2. CSS3 新特性

### 动画和过渡

```css
/* 过渡效果 */
.transition-demo {
  transition: all 0.3s ease-in-out;
}

/* 动画效果 */
@keyframes slide-in {
  from {
    transform: translateX(-100%);
  }
  to {
    transform: translateX(0);
  }
}

.animation-demo {
  animation: slide-in 0.5s ease-out forwards;
}
```

### transform 和 transition 的区别

```css
/* transform: 变换属性，立即生效 */
.transform {
  transform: scale(1.2) rotate(45deg) translateX(100px);
}

/* transition: 状态过渡，渐进式变化 */
.transition {
  transition: transform 0.3s ease;
}
```

### 媒体查询的使用

```css
/* 响应式设计 */
/* 移动优先 */
.container {
  width: 100%;
  padding: 15px;
}

/* 平板 */
@media screen and (min-width: 768px) {
  .container {
    width: 750px;
    margin: 0 auto;
  }
}

/* 桌面 */
@media screen and (min-width: 1024px) {
  .container {
    width: 970px;
  }
}

/* 大屏 */
@media screen and (min-width: 1200px) {
  .container {
    width: 1170px;
  }
}
```

## 3. 性能优化

### CSS 性能优化的方法

```css
/* 1. 选择器优化 */
/* 避免 */
.header .nav .list .item {
}

/* 推荐 */
.nav-item {
}

/* 2. 使用简写属性 */
/* 避免 */
.element {
  margin-top: 10px;
  margin-right: 20px;
  margin-bottom: 10px;
  margin-left: 20px;
}

/* 推荐 */
.element {
  margin: 10px 20px;
}

/* 3. 避免使用@import */
/* 使用link标签替代 */
```

### 重排(reflow)和重绘(repaint)

#### 触发重排的属性：

```css
/* 这些属性会触发重排 */
.trigger-reflow {
  width: 100px;
  height: 100px;
  padding: 20px;
  margin: 10px;
  position: absolute;
  top: 50px;
  left: 100px;
}
```

#### 优化建议：

```css
/* 使用 transform 代替位置改变 */
.better-performance {
  transform: translateX(100px);
  /* 比 left: 100px 性能更好 */
}

/* 使用 opacity 代替 visibility */
.fade {
  opacity: 0;
  /* 比 visibility: hidden 性能更好 */
}
```
