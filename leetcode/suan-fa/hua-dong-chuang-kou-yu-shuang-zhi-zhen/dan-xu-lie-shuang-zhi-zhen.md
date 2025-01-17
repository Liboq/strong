---
cover: >-
  https://images.unsplash.com/photo-1722255564100-2b1647405b3e?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHw5fHwlRTYlQjUlQjd8ZW58MHx8fHwxNzM3MDgzMjQ1fDA&ixlib=rb-4.0.3&q=85
coverY: 0
layout:
  cover:
    visible: true
    size: hero
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 🚴‍♂️ 单序列双指针

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

#### 1.[反转字符串](https://leetcode.cn/problems/reverse-string/)

> 编写一个函数，其作用是将输入的字符串反转过来。输入字符串以字符数组 `s` 的形式给出。
>
> 不要给另外的数组分配额外的空间，你必须[**原地**](https://baike.baidu.com/item/%E5%8E%9F%E5%9C%B0%E7%AE%97%E6%B3%95)**修改输入数组**、使用 O(1) 的额外空间解决这一问题。

```javascript
/**
 * @param {character[]} s
 * @return {void} Do not return anything, modify s in-place instead.
 */
var reverseString = function (s) {
    let left = 0  // 初始化左指针
    let right = s.length-1  // 初始化右指针
    while (left <= right) {  // 当左指针小于等于右指针时继续循环
        const temp = s[left]  // 交换元素：将左指针的值存储到临时变量
        s[left] = s[right]  // 将右指针的值赋给左指针
        s[right] = temp  // 将临时变量的值赋给右指针
        left++  // 左指针右移
        right--  // 右指针左移
    }
    return s  // 返回修改后的数组
};
```

#### 2.[验证回文串](https://leetcode.cn/problems/valid-palindrome/)

> 如果在将所有大写字符转换为小写字符、并移除所有非字母数字字符之后，短语正着读和反着读都一样。则可以认为该短语是一个 **回文串** 。
>
> 字母和数字都属于字母数字字符。
>
> 给你一个字符串 `s`，如果它是 **回文串** ，返回 `true` ；否则，返回 `false` 。
