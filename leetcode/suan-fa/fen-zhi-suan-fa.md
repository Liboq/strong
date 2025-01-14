---
description: 分而治之
icon: '1'
---

# 分治算法

### 原理

`分(划分阶段)` ：递归的将原问题分解为两个或多个子问题，直至到达最小子问题时终止。

`治 (合并阶段)` : 从已知解的最小问题开始，从底至顶地将子问题的姐进行合并，从而构建出原问题的解。

### 练习

#### 1.括号生成

> ```
> 数字 n 代表⽣成括号的对数，请你设计⼀个函数，⽤于能够⽣成所有可能的并且 有效的 括号组合
> ```

```javascript
/**
 * @param {number} n
 * @return {string[]}
 * @param l 左括号已经⽤了⼏个
 * @param r 右括号已经⽤了⼏个
 * @param str 当前递归得到的拼接字符串结果
 * @param res 结果集
 */
const generateParenthesis = (n) => {
  const res = [];
  const dfs = (l, r, str) => {
    if (l === n && r == n) {
      return res.push(str);
    }
    if (l < r) {
      return;
    }
    if (l < n) {
      dfs(l + 1, r, str + "(");
    }
    if (r < n) {
      dfs(l, r + 1, str + ")");
    }
  };
  dfs(0, 0, "");
  return res;
};

const testAns = ["((()))", "(()())", "(())()", "()(())", "()()()"];

console.log(generateParenthesis(3)===testAns)

```

#### 2.合并k个排序列表

> 给你一个链表数组，每个链表都已经按升序排列。
>
> 请你将所有链表合并到一个升序链表中，返回合并后的链表。

```javascript
/**
 *
 * @param {number} val
 * @param {ListNode|null} next
 */
function ListNode(val, next) {
  this.val = val === undefined ? 0 : val;
  this.next = next === undefined ? null : next;
}

/**
 * 输入：lists = [[1,4,5],[1,3,4],[2,6]]
 * 输出：[1,1,2,3,4,4,5,6]
 * 解释：链表数组如下：
 * [1->4->5,1->3->4,2->6]
 * 将它们合并到一个有序链表中得到。
 * 1->1->2->3->4->4->5->6
 * @param {ListNode[]} lists
 * @return {ListNode}
 * https://leetcode.cn/problems/merge-k-sorted-lists/
 */
var mergeKLists = function (lists) {
  function dfs(i, j) {
    const m = j - i;
    if (m === 0) {
      return null;
    }
    if (m === 1) {
      return lists[i];
    }
    const left = dfs(i, i + (m >> 1));
    const right = dfs(i + (m >> 1), j);
    return mergeTwoLists(left, right);
  }
  return dfs(0, lists.length);
};
const mergeTwoLists = function (list1, list2) {
  const dummy = new ListNode();
  let cur = dummy;
  while (list1 && list2) {
    if (list1.val < list2.val) {
      cur.next = list1;
      list1 = list1.next;
    } else {
      cur.next = list2;
      list2 = list2.next;
    }
    cur = cur.next;
  }
  cur.next = list1 ? list1 : list2;
  return dummy.next;
};

```

#### 3.二分查找

> #### 给定一个长度为 n 的有序数组 `nums` ，其中所有元素都是唯一的，请查找元素 `target` 。

<pre class="language-javascript"><code class="lang-javascript"><strong>
</strong><strong>const dfs = (nums, target, start, end) => {
</strong>  if (start > end) {
    return -1;
  }
  const m = start + ((end - start) >> 1);
  if (nums[m] &#x3C; target) {
    return dfs(nums, target, m + 1, end);
  } else if (nums[m] > target) {
    return dfs(nums, target, start, m - 1);
  } else {
    return m;
  }
};
function binarySearch(nums, target) {
  const n = nums.length;
  return dfs(nums, target, 0, n - 1);
}
</code></pre>

#### 4.
