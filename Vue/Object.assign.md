# Object.assign()

在阅读pinia等源码时,发现了`Object.assign`这个方法，开始有点疑惑,这个是啥用法去了

## 用法

`Object.assign()` 方法用于将一个或多个源对象的属性复制到目标对象。它接受目标对象作为第一个参数，后跟一个或多个源对象，并返回目标对象。如果多个源对象具有相同的属性，则后续对象的属性将覆盖先前对象的属性。

基本语法如下所示：

```javascript
Object.assign(target, ...sources)
```

- `target`: 目标对象，要将属性复制到的对象。
- `sources`: 一个或多个源对象，它们的属性将被复制到目标对象中。可以传递多个源对象。

示例：

```javascript
const target = { a: 1, b: 2 };
const source = { b: 3, c: 4, d:{e:1}};

Object.assign(target, source);

console.log(target); // 输出: { a: 1, b: 3, c: 4, d:{e:1} }, 目标对象被修改了
source.d.e = 6
console.log(target);// 输出: { a: 1, b: 3, c: 4, d:{e:6} }, 目标对象被修改了
```

在这个例子中，目标对象`target`的属性被源对象`source`的属性修改。`Object.assign()`方法修改了目标对象，并返回了合并后的对象。

值得注意的是，`Object.assign()`方法执行的是浅拷贝，即它只复制对象的属性的引用，而不是属性的实际值。这意味着，如果源对象的属性是对象或数组等引用类型，它们的改变也会影响到目标对象。

如果要进行深拷贝，需要考虑使用其他方法，如递归地遍历对象并复制其属性。



