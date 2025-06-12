+++
author = "yqg"
title = "Leetcode 刷题记录"
date = "2025-06-03"
description = "力扣（LeetCode）算法题解记录"
tags = [
    "Algorithm",
]
+++

## 1. 数组

### 1.1 两数之和

**思路**：哈希表记录访问过元素的索引

```javascript
var twoSum = function (nums, target) {
  const map = new Map();
  for (let i = 0; i < nums.length; i++) {
    const num = target - nums[i];
    if (map.has(num)) {
      return [map.get(num), i];
    } else {
      map.set(nums[i], i);
    }
  }
};
```

### 1.2 删除有序数组中的重复项

**思路**：双指针（快慢指针），元素有序递增，判断元素是否与前一个相同，不同则移动慢指针，返回慢指针索引+1

```javascript
var removeDuplicates = function (nums) {
  let i = 0; // 最后不重复元素的索引位置
  for (let j = 1; j < nums.length; j++) {
    if (nums[j] !== nums[i]) {
      nums[++i] = nums[j];
    }
  }
  return i + 1;
};
```

### 1.3 原地移除元素

**思路**：双指针（首尾指针）

```javascript
var removeElement = function (nums, val) {
  let left = 0;
  let right = nums.length - 1;
  while (left <= right) {
    if (nums[left] === val) {
      nums[left] = nums[right];
      right--;
    } else {
      left++;
    }
  }
  return left;
};
```

### 1.4 搜索插入位置

**思路**：二分查找法

```javascript
var searchInsert = function (nums, target) {
  let left = 0;
  let right = nums.length - 1;
  while (left <= right) {
    const middle = Math.ceil((left + right) / 2);
    if (nums[middle] === target) {
      return middle;
    } else if (nums[middle] > target) {
      right = middle - 1;
    } else {
      left = middle + 1;
    }
  }
  return left;
};
```

### 1.5 加一

**思路**：从后向前遍历，处理进位情况

```javascript
var plusOne = function (digits) {
  for (let i = digits.length - 1; i >= 0; i--) {
    if (digits[i] !== 9) {
      digits[i] += 1;
      return digits;
    } else {
      digits[i] = 0;
    }
  }
  return [1, ...digits];
};
```

### 1.6 合并两个有序数组

**思路**：三指针，数组有序，从后往前对比

```javascript
var merge = function (nums1, m, nums2, n) {
  let i = m - 1; // nums1 的最后一个有效元素
  let j = n - 1; // nums2 的最后一个元素
  let k = m + n - 1; // 合并后的最后一个位置

  while (j >= 0) {
    if (i >= 0 && nums1[i] > nums2[j]) {
      nums1[k--] = nums1[i--];
    } else {
      nums1[k--] = nums2[j--];
    }
  }
  return nums1;
};
```

### 1.7 将有序数组转换为二叉搜索树

**思路**：递归，每一次返回数组中的中间点

```javascript
var sortedArrayToBST = function (nums) {
  if (nums.length === 0) {
    return null;
  }
  const middle = Math.floor(nums.length / 2);
  const root = new TreeNode(nums[middle]);
  root.left = sortedArrayToBST(nums.slice(0, middle));
  root.right = sortedArrayToBST(nums.slice(middle + 1, nums.length));
  return root;
};
```

### 1.8 买卖股票的最佳时机

**思路**：记住最小点，遍历过程中更新最小点并计算利润

```javascript
var maxProfit = function (prices) {
  let min = prices[0];
  let s = 0;
  for (let i = 1; i < prices.length; i++) {
    if (prices[i] < min) {
      min = prices[i];
    } else {
      s = Math.max(s, prices[i] - min);
    }
  }
  return s;
};
```

### 1.9 只出现一次的数字

**思路**：异或运算，空间复杂度 o(1)

```js
var singleNumber = function (nums) {
  let s = 0;
  for (let i = 0; i < nums.length; i++) {
    s ^= nums[i];
  }
  return s;
};
```

### 1.10 多数元素

**思路**：摩尔投票法，空间复杂度 o(1)

```js
var majorityElement = function (nums) {
  let s = null;
  let count = 0;
  for (let num of nums) {
    if (count === 0) {
      s = num;
    }
    count += s === num ? 1 : -1;
  }
  return s;
};
```

### 1.11 存在重复元素

**思路**：set 记录元素

```js
var containsDuplicate = function (nums) {
  const seen = new Set();
  for (let num of nums) {
    if (seen.has(num)) {
      return true;
    }
    seen.add(num);
  }
  return false;
};
```

### 1.12 存在重复元素 II

**思路**：map 记录元素出现的位置，如果满足第一次和第二次的位置 abs(i - j) <= k，返回 true，否则更新元素位置

```js
var containsNearbyDuplicate = function (nums, k) {
  const map = new Map();
  for (let i = 0; i < nums.length; i++) {
    if (map.has(nums[i])) {
      if (Math.abs(i - map.get(nums[i])) <= k) {
        return true;
      } else {
        map.set(nums[i], i);
      }
    } else {
      map.set(nums[i], i);
    }
  }
  return false;
};
```

### 1.13 丢失的数字

**思路**：数组求和后遍历递减，剩余值为丢失数字

```js
var missingNumber = function (nums) {
  let sum = 0;
  for (let i = 1; i <= nums.length; i++) {
    sum += i;
  }
  for (let num of nums) {
    sum -= num;
  }
  return sum;
};
```

### 1.14 找到所有数组中消失的数字

**思路**：第一次遍历每一个元素，元素应该出现的位置为其绝对值减 1，第二次遍历，值为正则保存索引加 1。

```js
var findDisappearedNumbers = function (nums) {
  const res = [];
  for (let i = 0; i < nums.length; i++) {
    const index = Math.abs(nums[i]) - 1;
    if (nums[index] > 0) {
      nums[index] = -nums[index];
    }
  }
  for (let i = 0; i < nums.length; i++) {
    if (nums[i] > 0) {
      res.push(i + 1);
    }
  }
  return res;
};
```

### 1.15 移动零

**思路**：双指针，零往后移动

```js
var moveZeroes = function (nums) {
  let left = 0;
  for (let i = 0; i < nums.length; i++) {
    if (nums[i] !== 0) {
      if (i !== left) {
        nums[left] = nums[i];
        nums[i] = 0;
      }
      left++;
    }
  }
  return nums;
};
```

### 1.16 两个数组的交集 II

**思路**：map 记录第一个数组元素出现的次数，遍历第二个数组时，map 元素出现次数不为 0 的情况下，存储结果。

```js
var intersect = function (nums1, nums2) {
  const map = new Map();
  const res = [];
  for (let item of nums1) {
    map.has(item) ? map.set(item, map.get(item) + 1) : map.set(item, 1);
  }
  for (let item of nums2) {
    if (map.has(item) && map.get(item) !== 0) {
      res.push(item);
      map.set(item, map.get(item) - 1);
    }
  }
  return res;
};
```

### 1.17 第三大的数

**思路**：维护一个长度为 3 的前三大元素的数组，输出数组中的最小值。 或者使用三指针

```js
var thirdMax = function (nums) {
  const set = new Set();
  for (let num of nums) {
    set.add(num);
    if (set.size > 3) {
      const min = Math.min(...set);
      set.delete(min);
    }
  }
  if (set.size < 3) {
    return Math.max(...set);
  }

  return Math.min(...set);
};
```

### 1.18 分发饼干

**思路**：数组排序后，将最小的饼干分给最小饭量的小朋友，不满足下一个饼干。满足，下一个小朋友下一个饼干。

```js
var findContentChildren = function (g, s) {
  g.sort((a, b) => a - b);
  s.sort((a, b) => a - b);
  let index = 0;
  for (let i = 0; i < s.length; i++) {
    if (s[i] >= g[index]) {
      index++;
    }
  }
  return index;
};
```

### 1.19 岛屿周长

**思路**：如果是岛屿，周长+4，然后判断边界情况，如果岛屿周边有岛屿，周长减 1

```js
var islandPerimeter = function (grid) {
  let res = 0;
  let row = grid.length;
  for (let i = 0; i < row; i++) {
    let col = grid[i].length;
    for (let j = 0; j < col; j++) {
      if (grid[i][j] === 1) {
        res += 4;
        if (i - 1 >= 0 && grid[i - 1][j] === 1) {
          res--;
        }
        if (i + 1 < row && grid[i + 1][j] === 1) {
          res--;
        }
        if (j - 1 >= 0 && grid[i][j - 1] === 1) {
          res--;
        }
        if (j + 1 < col && grid[i][j + 1] === 1) {
          res--;
        }
      }
    }
  }
  return res;
};
```

### 1.20 最大连续 1 的个数

**思路**：双指针 i，j。i 记录第一个 1 的位置，j 当前遍历的位置，计算连续 1 的长度，如果 j 位置不为 0，重置 i。

```javascript
var findMaxConsecutiveOnes = function (nums) {
  let res = 0;
  let i = -1;
  for (let j = 0; j < nums.length; j++) {
    if (nums[j] === 1) {
      if (i === -1) {
        i = j;
      }
      res = Math.max(res, j - i + 1);
    } else {
      i = -1;
    }
  }
  return res;
};
```

### 1.21 提莫的攻击

**思路**: max 记录当前中毒结束时间，每次攻击直接加上中毒时间，如果攻击时已经中毒，减去本次攻击和上次攻击的交集

```javascript
var findPoisonedDuration = function (timeSeries, duration) {
  let res = 0;
  let max = -1;
  for (let i = 0; i < timeSeries.length; i++) {
    res += duration; // 直接添加本次攻击时间
    if (timeSeries[i] <= max) {
      // 如何两次攻击存在重叠
      res -= max - timeSeries[i] + 1; // 减去两次攻击时间交集
    }
    max = timeSeries[i] + duration - 1; //当前中毒结束时间
  }
  return res;
};
```

### 1.22 下一个更大元素 I

**思路**: 哈希表 + 单调栈

```javascript
var nextGreaterElement = function (nums1, nums2) {
  const map = new Map();
  const stack = [];
  for (let num of nums2) {
    //单调递减栈，遇到大的元素，清空栈，为每个出栈的元素设置右侧最大值
    while (stack.length > 0 && stack[stack.length - 1] < num) {
      map.set(stack.pop(), num);
    }
    stack.push(num);
  }

  for (let num of stack) {
    map.set(num, -1);
  }

  return nums1.map((num) => map.get(num));
};
```

### 1.23 相对名次

**思路**: 排序 + 哈希表

```javascript
var findRelativeRanks = function (score) {
  const map = new Map();
  const sortedScores = [...score].sort((a, b) => b - a);
  for (let i = 0; i < sortedScores.length; i++) {
    if (i === 0) {
      map.set(sortedScores[i], "Gold Medal");
    } else if (i === 1) {
      map.set(sortedScores[i], "Silver Medal");
    } else if (i === 2) {
      map.set(sortedScores[i], "Bronze Medal");
    } else {
      map.set(sortedScores[i], (i + 1).toString());
    }
  }

  return score.map((s) => map.get(s));
};
```

### 1.24 重塑矩阵

**思路**: 二维变一维，一维再变二维

```javascript
var matrixReshape = function (mat, r, c) {
  // 变成一维数组，然后再改变
  const row = mat.length;
  const col = mat[0].length;
  // 不能重塑矩阵
  if (row * col !== r * c || (row === r && col === c)) {
    return mat;
  }
  const flat = mat.flat();
  const res = [];
  let index = 0;
  for (let i = 0; i < r; i++) {
    const arr = [];
    for (let j = 0; j < c; j++) {
      arr[j] = flat[index++];
    }
    res.push(arr);
  }
  return res;
};
```

### 1.25 最长和谐子序列

**思路**: map 记录每个元素出现的次数，遍历 map，判断 map 是否存在比当前元素大 1 的元素，更新最大值

```javascript
var findLHS = function (nums) {
  const map = new Map();
  for (let num of nums) {
    map.has(num) ? map.set(num, map.get(num) + 1) : map.set(num, 1);
  }
  let res = 0;
  for (const [num, cnt] of map) {
    if (map.has(num + 1)) {
      res = Math.max(res, map.get(num) + map.get(num + 1));
    }
  }
  return res;
};
```

### 1.26 区间求和 II

**思路**: 区间范围最小值就是重复次数最多的区域

```javascript
var maxCount = function (m, n, ops) {
  if (ops.length === 0) {
    return m * n;
  }
  let minA = Infinity;
  let minB = Infinity;
  for (let o of ops) {
    minA = Math.min(minA, o[0]);
    minB = Math.min(minB, o[1]);
  }
  return minA * minB;
};
```

## 2. 字符串

### 2.1 最长公共前缀

**思路：横向遍历，将第一个字符串作为初始公共字符前缀，与第二个字符串比较，循环判断是否第二个字符串包括公共字符串并且 index 为 0,并不断更新公共字符串前缀。之后第三个字符串与公共前缀比较，
直到最后一个，如果中途判断公共前缀为空字符串，返回空字符串。**

```javascript
var longestCommonPrefix = function (strs) {
  let s = strs[0];
  for (let i = 1; i < strs.length; i++) {
    while (strs[i].indexOf(s) !== 0) {
      s = s.slice(0, -1);
      if (s === "") {
        return "";
      }
    }
  }
  return s;
};
```

### 2.2 有效的括号

**思路：利用 map 匹配括号，利用栈存储括号，如果字符属于左边的直接放入栈，如果属于右边的，与栈顶元素匹配，如果匹配成功，弹出栈，否则元素入栈，最后判断栈是否为空**

```javascript
var isValid = function (s) {
  const leftStrs = "([{";
  const map = { ")": "(", "]": "[", "}": "{" };
  const stack = [];
  for (let c of s) {
    if (leftStrs.indexOf(c) === -1) {
      if (map[c] === stack[stack.length - 1]) {
        stack.pop();
      } else {
        stack.push(c);
      }
    } else {
      stack.push(c);
    }
  }
  return stack.length === 0;
};
```

### 2.3 罗马数字转整数

**思路：如 IV I = 1， V = 5，IV 为 4，如果一个罗马数字的右边罗马数字比其大，则该罗马数字取负。IM 这样的不是合法数字**

```javascript
var romanToInt = function (s) {
  const map = { I: 1, V: 5, X: 10, L: 50, C: 100, D: 500, M: 1000 };
  let n = 0;
  let num = 0;
  while (n < s.length) {
    let cur = s[n];
    if (n + 1 < s.length) {
      let next = s[n + 1];
      if (map[cur] < map[next]) {
        num -= map[cur];
      } else {
        num += map[cur];
      }
    } else {
      num += map[cur];
    }
    n++;
  }
  return num;
};
```

## 3. 链表

## 4. 二叉树

## 5. 贪心

## 6. 动态规划

## 7. 回溯

## 8. 图
