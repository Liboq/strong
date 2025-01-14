---
description: 滑动窗口与双指针联系与学习
icon: '2'
---

# 滑动窗口与双指针



{% embed url="https://leetcode.cn/circle/discuss/0viNMK/" %}
滑动窗口与双指针
{% endembed %}

### 练习

### 一、定长滑动窗口 <a href="#yi-ding-chang-hua-dong-chuang-kou" id="yi-ding-chang-hua-dong-chuang-kou"></a>

#### 1.[定长子串中元音的最大数目](https://leetcode.cn/problems/maximum-number-of-vowels-in-a-substring-of-given-length/)

> 给你字符串 `s` 和整数 `k` 。
>
> 请返回字符串 `s` 中长度为 `k` 的单个子字符串中可能包含的最大元音字母数。
>
> 英文中的 **元音字母** 为（`a`, `e`, `i`, `o`, `u`）。

```javascript

/**
 * @param {string} s
 * @param {number} k
 * @return {number}
 */
var maxVowels = function (s, k) {
    const point = ['a', 'e', 'i', 'o', 'u']
    let i = 0
    let max = 0
    let score = 0
    while (i < s.length) {
        if (point.includes(s[i])) {
            score++
        }
        if (i < k - 1) { // 窗口大小不足 k
            i++
            continue;
        }
        max = Math.max(max, score)

        const m = i - k + 1
        if (point.includes(s[m])) {
            score--
        }
        i++

    }
    return max

};
```

#### 2.[子数组最大平均数 I](https://leetcode.cn/problems/maximum-average-subarray-i/)

> 给你一个由 `n` 个元素组成的整数数组 `nums` 和一个整数 `k` 。
>
> 请你找出平均数最大且 **长度为 `k`** 的连续子数组，并输出该最大平均数。

```javascript

/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number}
 */
var findMaxAverage = function (nums, k) {
    let res = -999999999
    let cur = 0
    for (let i = 0; i < nums.length; i++) {
        cur += nums[i]
        if (i < k - 1) {
            continue
        }
        res = Math.max(res, cur)
        const m = i - k + 1
        cur = cur - nums[m]
    }
    return res / k
};
```

#### 3.[大小为 K 且平均值大于等于阈值的子数组数目](https://leetcode.cn/problems/number-of-sub-arrays-of-size-k-and-average-greater-than-or-equal-to-threshold/)

> 给你一个整数数组 `arr` 和两个整数 `k` 和 `threshold` 。
>
> 请你返回长度为 `k` 且平均值大于等于 `threshold` 的子数组数目。

```javascript
/**
 * @param {number[]} arr
 * @param {number} k
 * @param {number} threshold
 * @return {number}
 */
var numOfSubarrays = function (arr, k, threshold) {
    let res = 0
    let cur = 0
    for (let i = 0; i < arr.length; i++) {
        cur += arr[i]
        if (i < k - 1) {
            continue;
        }
        if (cur / k >= threshold) {
            res++
        }
        const m = i - k + 1
        cur -= arr[m]
    }
    return res
};
```

#### 4.[半径为 k 的子数组平均值](https://leetcode.cn/problems/k-radius-subarray-averages/)

> 给你一个下标从 **0** 开始的数组 `nums` ，数组中有 `n` 个整数，另给你一个整数 `k` 。
>
> **半径为 k 的子数组平均值** 是指：`nums` 中一个以下标 `i` 为 **中心** 且 **半径** 为 `k` 的子数组中所有元素的平均值，即下标在 `i - k` 和 `i + k` 范围（**含** `i - k` 和 `i + k`）内所有元素的平均值。如果在下标 `i` 前或后不足 `k` 个元素，那么 **半径为 k 的子数组平均值** 是 `-1` 。
>
> 构建并返回一个长度为 `n` 的数组 `avgs` ，其中 `avgs[i]` 是以下标 `i` 为中心的子数组的 **半径为 k 的子数组平均值** 。
>
> `x` 个元素的 **平均值** 是 `x` 个元素相加之和除以 `x` ，此时使用截断式 **整数除法** ，即需要去掉结果的小数部分。

```javascript
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number[]}
 */
var getAverages = function (nums, k) {
    let res = 0
    let ans = new Array(nums.length).fill(-1)
    for (let i = 0; i < nums.length; i++) {
        res += nums[i]
        if (i < 2 * k) {
            continue
        }
        const average = Math.floor(res / (2 * k + 1))
        ans[i - k] = average
        res -= nums[i - 2 * k]
    }
    return ans
};
```

#### 5.[得到 K 个黑块的最少涂色次数](https://leetcode.cn/problems/minimum-recolors-to-get-k-consecutive-black-blocks/)

> 给你一个长度为 `n` 下标从 **0** 开始的字符串 `blocks` ，`blocks[i]` 要么是 `'W'` 要么是 `'B'` ，表示第 `i` 块的颜色。字符 `'W'` 和 `'B'` 分别表示白色和黑色。
>
> 给你一个整数 `k` ，表示想要 **连续** 黑色块的数目。
>
> 每一次操作中，你可以选择一个白色块将它 **涂成** 黑色块。
>
> 请你返回至少出现 **一次** 连续 `k` 个黑色块的 **最少** 操作次数。

```javascript
/**
 * @param {string} blocks
 * @param {number} k
 * @return {number}
 */
var minimumRecolors = function (blocks, k) {
    let res = 99999999
    let cur = 0
    for (let i = 0; i < blocks.length; i++) {
        if ('B' !== blocks[i]) {
            cur++
        }
        if (i < k - 1) {
            continue
        }
        res = Math.min(cur, res)
        const m = i - k + 1
        if (blocks[m] !== 'B') {
            cur--
        }
    }
    return res
};
```

#### 6.[爱生气的书店老板](https://leetcode.cn/problems/grumpy-bookstore-owner/)

> 有一个书店老板，他的书店开了 `n` 分钟。每分钟都有一些顾客进入这家商店。给定一个长度为 `n` 的整数数组 `customers` ，其中 `customers[i]` 是在第 `i` 分钟开始时进入商店的顾客数量，所有这些顾客在第 `i` 分钟结束后离开。
>
> 在某些分钟内，书店老板会生气。 如果书店老板在第 `i` 分钟生气，那么 `grumpy[i] = 1`，否则 `grumpy[i] = 0`。
>
> 当书店老板生气时，那一分钟的顾客就会不满意，若老板不生气则顾客是满意的。
>
> 书店老板知道一个秘密技巧，能抑制自己的情绪，可以让自己连续 `minutes` 分钟不生气，但却只能使用一次。
>
> 请你返回 _这一天营业下来，最多有多少客户能够感到满意_ 。

```javascript
/**
 * @param {number[]} customers
 * @param {number[]} grumpy
 * @param {number} minutes
 * @return {number}
 */
var maxSatisfied = function (customers, grumpy, minutes) {
    let maxS1 = 0
    let s = [0, 0]
    for (let i = 0; i < customers.length; i++) {
        s[grumpy[i]] += customers[i]
        if (i < minutes.length - 1) {
            continue
        }
        maxS1 = Math.max(maxS1, s[1])
        s[1] -= grumpy[i - minutes + 1] ? customers[i - minutes + 1] : 0
    }
    return maxS1 + s[0]
};
```

#### 7.[最少交换次数来组合所有的 1 II](https://leetcode.cn/problems/minimum-swaps-to-group-all-1s-together-ii/)

> **交换** 定义为选中一个数组中的两个 **互不相同** 的位置并交换二者的值。
>
> **环形** 数组是一个数组，可以认为 **第一个** 元素和 **最后一个** 元素 **相邻** 。
>
> 给你一个 **二进制环形** 数组 `nums` ，返回在 **任意位置** 将数组中的所有 `1` 聚集在一起需要的最少交换次数。

```javascript
/**
 * @param {number[]} nums
 * @return {number}
 */
var minSwaps = function (nums) {
    let oneNums = nums.reduce((prev, cur) => prev + cur, 0)
    let zeroNums = 0
    for (let i = 0; i < oneNums; i++) {
        if (nums[i] === 0) {
            zeroNums++
        }
    }
    let newNums = nums.concat(nums)
    let changeNums = zeroNums
    let left = 0
    for (let right = oneNums; right < nums.length + oneNums; right++) {
        if (!newNums[left++]) changeNums--
        if (!newNums[right]) changeNums++
        zeroNums = Math.min(zeroNums, changeNums)
    }
    right = 0
    return zeroNums
}
```

#### 8.[滑动子数组的美丽值](https://leetcode.cn/problems/sliding-subarray-beauty/)

> 给你一个长度为 `n` 的整数数组 `nums` ，请你求出每个长度为 `k` 的子数组的 美丽值 。
>
> 一个子数组的 **美丽值** 定义为：如果子数组中第 `x` **小整数** 是 **负数** ，那么美丽值为第 `x` 小的数，否则美丽值为 `0` 。
>
> 请你返回一个包含 `n - k + 1` 个整数的数组，**依次** 表示数组中从第一个下标开始，每个长度为 `k` 的子数组的 **美丽值** 。
>
> * 子数组指的是数组中一段连续 **非空** 的元素序列。

