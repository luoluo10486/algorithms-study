# Day 10 — 双指针 Hot100 + 代码随想录

## 题目：移除元素（Remove Element）

**LeetCode 27 | 代码随想录 | 难度：🟢 简单**

### 题目描述

给你一个数组 `nums` 和一个值 `val`，你需要**原地**移除所有数值等于 `val` 的元素。元素的顺序可能发生改变。然后返回 `nums` 中与 `val` 不同的元素的数量。

不要使用额外的数组空间，你必须仅使用 **O(1) 额外空间**并**原地**修改输入数组。

### 示例

```
输入：nums = [3,2,2,3], val = 3
输出：2, nums = [2,2,_,_]

输入：nums = [0,1,2,2,3,0,4,2], val = 2
输出：5, nums = [0,1,4,0,3,_,_,_]
```

---

## 解法：快慢指针

### 思路

双指针（快慢指针）是最经典的做法——**快指针遍历整个数组，慢指针指向下一个要放置合法元素的位置**。

```
快指针 fast：负责探路，找不等于 val 的元素
慢指针 slow：负责占位，指向下一个要覆盖的位置
```

当 `nums[fast] != val` 时，把它复制到 `nums[slow]` 位置，`slow++`。这样遍历结束后，`[0, slow)` 区间内的所有元素都不是 `val`。

### 思考方式图解

```
nums = [3, 2, 2, 3], val = 3

初始：fast=0, slow=0
  fast=0: nums[0]=3 == val → 跳过，slow 不动
  fast=1: nums[1]=2 != val → nums[0]=2, slow=1
  fast=2: nums[2]=2 != val → nums[1]=2, slow=2
  fast=3: nums[3]=3 == val → 跳过

结果：[2, 2, ...], slow=2 ✅
```

```
nums = [0,1,2,2,3,0,4,2], val = 2

fast=0: 0 != 2 → nums[0]=0, slow=1
fast=1: 1 != 2 → nums[1]=1, slow=2
fast=2: 2 == 2 → 跳过
fast=3: 2 == 2 → 跳过
fast=4: 3 != 2 → nums[2]=3, slow=3
fast=5: 0 != 2 → nums[3]=0, slow=4
fast=6: 4 != 2 → nums[4]=4, slow=5
fast=7: 2 == 2 → 跳过

结果：[0,1,3,0,4, ...], slow=5 ✅
```

### 代码实现

```java
class Solution {
    public int removeElement(int[] nums, int val) {
        int slow = 0;
        for (int fast = 0; fast < nums.length; fast++) {
            if (nums[fast] != val) {
                nums[slow] = nums[fast];
                slow++;
            }
        }
        return slow;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 一次遍历 |
| 🧠 空间复杂度 | **O(1)** — 原地修改 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 快慢指针 |
| 算法类型 | 双指针、数组 |
| 核心技巧 | **快指针探路找合法元素，慢指针占位标记新数组长度** |
| 适用场景 | 原地删除、原地去重（同款模板） |
| 关联题目 | [26. 删除有序数组中的重复项](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/)—同一套快慢指针模板 |
| 易错点 | slow 指向的是**下一个要填的位置**，所以最终 slow 就是新数组的长度；不需要真的删除元素，覆盖即可 |

### 一句话记住

> **「快针探路慢针等，不等于 val 就覆盖。」**

---

## 题目：盛最多水的容器（Container With Most Water）

**LeetCode 11 | 双指针 Hot100 | 难度：🟡 中等**

### 题目描述

给定一个长度为 `n` 的整数数组 `height`。有 `n` 条垂线，第 `i` 条线的两个端点是 `(i, 0)` 和 `(i, height[i])`。

找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水。

返回容器可以储存的最大水量。

**说明**：你不能倾斜容器。

### 示例

```
输入：[1,8,6,2,5,4,8,3,7]
输出：49
解释：选择 height[1]=8 和 height[8]=7，宽度为 7，高度为 min(8,7)=7，面积 = 7×7 = 49

输入：[1,1]
输出：1
```

---

## 解法：左右对撞双指针

### 思路

容器盛水的面积 = **宽度 × 最小高度**。

暴力解是 O(n²) 枚举所有左右边界组合，用双指针可以降到 **O(n)**。

核心思想：**左右指针从两端向中间靠拢，每次都移动较短的那条边**。

为什么移动较短的？因为面积受限于短板——移动长板，宽度变小但高度不可能超过短板，面积只会更小；移动短板，虽然宽度也变小，但**有可能遇到更高的板子，面积有可能变大**。

这就是所谓的 **「向内移动短板的收益有可能为正，移动长板的收益一定为负」**。

### 思考方式图解

```
height = [1, 8, 6, 2, 5, 4, 8, 3, 7]

left=0(1),   right=8(7):  area=8×min(1,7)=8    ans=8
  移动短板 → left=1(8)

left=1(8),   right=8(7):  area=7×min(8,7)=49   ans=49
  移动短板 → right=7(3)

left=1(8),   right=7(3):  area=6×min(8,3)=18   ans=49
  移动短板 → right=6(8)

left=1(8),   right=6(8):  area=5×min(8,8)=40   ans=49
  移动短板 → 都可以，right-- (假设)

...后续面积均小于 49

结果：49 ✅
```

### 代码实现

```java
class Solution {
    public int maxArea(int[] height) {
        int ans = 0;
        int left = 0;
        int right = height.length - 1;

        while (left < right) {
            // 计算当前左右指针围成的面积
            int area = (right - left) * Math.min(height[left], height[right]);
            ans = Math.max(ans, area);

            // 移动较短的边（短板向内移动）
            if (height[left] < height[right]) {
                left++;
            } else {
                right--;
            }
        }

        return ans;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 左右指针各遍历一次 |
| 🧠 空间复杂度 | **O(1)** — 仅用了几个变量 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 对撞双指针 |
| 算法类型 | 双指针、贪心 |
| 核心技巧 | **每次移动较短的那条边（短板），因为只有移动短板面积才有可能变大** |
| 为什么移动短板 | 移动长板：宽度↓，高度 ≤ 短板 → 面积一定变小；移动短板：宽度↓，但高度**可能**变大 → 面积有可能变大 |
| 适用场景 | 数组/字符串中需要两端逼近找最优解 |
| 关联题目 | [42. 接雨水](https://leetcode.cn/problems/trapping-rain-water/)——同样是双指针 + 短板思路，更复杂 |
| 易错点 | 两边相等时移动哪边都可以，不影响最终结果 |

### 一句话记住

> **「左右对撞，移动短板——面积只由短板决定。」**

---

## 题目：三数之和（3Sum）

**LeetCode 15 | 双指针 Hot100 | 难度：🟡 中等**

### 题目描述

给你一个整数数组 `nums`，判断是否存在三元组 `[nums[i], nums[j], nums[k]]` 满足 `i != j`、`i != k` 且 `j != k`，同时还满足 `nums[i] + nums[j] + nums[k] == 0`。

请你返回所有和为 0 且**不重复**的三元组。

### 示例

```
输入：nums = [-1,0,1,2,-1,-4]
输出：[[-1,-1,2],[-1,0,1]]

输入：nums = [0,1,1]
输出：[]

输入：nums = [0,0,0]
输出：[[0,0,0]]
```

---

## 解法：排序 + 双指针

### 思路

暴力三层循环是 O(n³)，排序 + 双指针降到 **O(n²)**。

核心思想：**先排序，固定一个数 x，剩下的两个数用双指针在有序区间内对撞查找**。

```
与「盛最多水的容器」对比：

                        盛最多水的容器                三数之和
    ─────────────────────────────────────────────────────────
    指针移动依据    哪边矮移哪边               和与 0 的大小关系
    固定部分        无                        外层 for 循环固定第一个数
    区间            全部                      固定数之后的子区间
```

**去重是关键**：排序后跳过相邻重复元素，总共三层去重（i、j、k 各一层）。

### 思考方式图解

```
nums = [-1,0,1,2,-1,-4]
排序后 = [-4,-1,-1,0,1,2]

i=0, x=-4
  j=1(-1), k=5(2) → s=-3 <0 → j++
  j=2(-1), k=5(2) → s=-3 <0 → j++
  j=3(0),  k=5(2) → s=-2 <0 → j++
  j=4(1),  k=5(2) → s=-1 <0 → j++ → j≥k 结束

i=1, x=-1
  j=2(-1), k=5(2) → s=0 → [-1,-1,2] ✅ j++,k-- 跳过重复
  j=3(0),  k=4(1) → s=0 → [-1,0,1]  ✅ j++,k--
  之后 j≥k 结束

i=2, x=-1 → 与 i=1 重复，continue

i=3, x=0
  j=4(1), k=5(2) → s=3 >0 → k-- → j≥k 结束

i=4 → i > n-2 退出

结果：[[-1,-1,2], [-1,0,1]]
```

### 代码实现

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        Arrays.sort(nums);
        List<List<Integer>> ans = new ArrayList<>();
        int n = nums.length;

        for (int i = 0; i < n - 2; i++) {
            int x = nums[i];

            // 跳过重复的固定元素
            if (i > 0 && nums[i - 1] == nums[i]) continue;

            // 剪枝一：最小的三个数之和 > 0，后面更不可能
            if (x + nums[i + 1] + nums[i + 2] > 0) break;

            // 剪枝二：当前数 + 最大的两个数 < 0，当前数太小
            if (x + nums[n - 1] + nums[n - 2] < 0) continue;

            int j = i + 1, k = n - 1;
            while (j < k) {
                int s = x + nums[j] + nums[k];
                if (s > 0) {
                    k--;
                } else if (s < 0) {
                    j++;
                } else {
                    ans.add(List.of(x, nums[j], nums[k]));
                    // 跳过重复数字（j 和 k 各一层）
                    for (j++; j < k && nums[j] == nums[j - 1]; j++);
                    for (k--; j < k && nums[k] == nums[k + 1]; k--);
                }
            }
        }
        return ans;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n²)** — 排序 O(n log n) + 双指针 O(n²) |
| 🧠 空间复杂度 | **O(1)** — 不计排序和输出数组 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 排序 + 双指针 |
| 算法类型 | 双指针、数组 |
| 核心技巧 | **外层固定一个数，内层双指针在对撞中找两数之和** |
| 剪枝优化 | 最小三数和 > 0 → break；当前 + 最大两个 < 0 → continue |
| 去重策略 | 3 层去重（i、j、k 各一层），排序后跳过相邻重复元素 |
| 适用场景 | nSum 系列问题的核心模板 |
| 关联题目 | [167. 两数之和 II](https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted/)——有序数组双指针 |
| 易错点 | 排序必须放在最前面；去重是跳「已经处理过的」重复，不是跳「后面会出现的」重复 |

### 一句话记住

> **「排序定一，双指针扫二——剪枝去重一步到位。」**

---

## 题目：接雨水（Trapping Rain Water）

**LeetCode 42 | 双指针 Hot100 | 难度：🔴 困难**

### 题目描述

给定 `n` 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

### 示例

```
输入：height = [0,1,0,2,1,0,1,3,2,1,2,1]
输出：6

输入：height = [4,2,0,3,2,5]
输出：9
```

---

## 解法：对撞双指针

### 思路

每个位置能接的雨水量取决于它**左右两侧最大高度的较小值 - 当前高度**。

即：`water[i] = min(leftMax[i], rightMax[i]) - height[i]`

暴力做法是每个位置都往左右扫一次找最大值，O(n²)。预处理前缀最大值和后缀最大值可以降到 O(n) + O(n) 空间。

双指针解法更进一步——**在 O(n) 时间 + O(1) 空间内完成**。

核心思想：**左右指针向中间靠拢，同时维护 `premax`（左侧已遍历的最大值）和 `sufmax`（右侧已遍历的最大值）来决定哪边先结算**。

```
对于位置 left：
  它左边的最大高度 premax 是确定的（已经遍历过）
  它右边的最大高度至少是 sufmax（右侧遍历过的最大值）
  
  如果 premax < sufmax：
    那么 left 位置能接的水量 = premax - height[left]（因为短板在左边，右边的最大值只可能 >= sufmax > premax，所以实际短板就是 premax）
    结算后 left++
  
  反之：
    同理，right 位置的水量 = sufmax - height[right]
    结算后 right--
```

**为什么是对的？**「短板原理」——`premax` 和 `sufmax` 中较小的那个就是当前边界在结算时的实际限制，因为在另一侧即使有更高的柱子，水也只会从较矮的一边流出去。

### 思考方式图解

```
height = [0,1,0,2,1,0,1,3,2,1,2,1]

初始：premax=0, sufmax=0, left=0(0), right=11(1)

left=0(0):  premax=max(0,0)=0,  sufmax=max(0,1)=1
  premax(0) < sufmax(1) → ans+=0-0=0, left=1

left=1(1):  premax=max(0,1)=1,  sufmax=max(1,1)=1
  premax(1) >= sufmax(1) → ans+=1-1=0, right=10

right=10(2): premax=1, sufmax=max(1,2)=2
  premax(1) < sufmax(2) → ans+=1-1=0, left=2

left=2(0):  premax=max(1,0)=1,  sufmax=2
  premax(1) < sufmax(2) → ans+=1-0=1, left=3  ← 接到 1 单位水

left=3(2):  premax=max(1,2)=2,  sufmax=2
  premax(2) >= sufmax(2) → ans+=2-2=0, right=9

right=9(1): premax=2, sufmax=max(2,1)=2
  premax(2) >= sufmax(2) → ans+=2-1=1, right=8  ← 接到 1 单位水

... 继续结算

结果：6 ✅
```

### 代码实现

```java
class Solution {
    public int trap(int[] height) {
        int ans = 0;
        int premax = 0;   // 左侧已遍历的最大高度
        int sufmax = 0;   // 右侧已遍历的最大高度
        int left = 0;
        int right = height.length - 1;

        while (left < right) {
            // 先更新两侧的最大值
            premax = Math.max(premax, height[left]);
            sufmax = Math.max(sufmax, height[right]);

            // 短板决定了当前能接多少水
            if (premax < sufmax) {
                // 左边是短板，结算 left 位置
                ans += premax - height[left];
                left++;
            } else {
                // 右边是短板（或相等），结算 right 位置
                ans += sufmax - height[right];
                right--;
            }
        }

        return ans;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 左右指针各遍历一次 |
| 🧠 空间复杂度 | **O(1)** — 仅用了几个变量 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 对撞双指针（维护左右最大高度） |
| 算法类型 | 双指针、动态规划 |
| 核心技巧 | **同时维护 premax 和 sufmax，两者中较小的就是当前柱子的"短板"，直接结算** |
| 与 11. 盛水容器区别 | 盛水容器找的是**两根柱子之间的最大面积**；接雨水是**每根柱子单独结算自己的积水量** |
| 为什么能 O(1) 空间 | 因为 `premax < sufmax` 时，left 位置的水量已经由 premax 决定了，不需要知道右边具体的最大值（已知它 ≥ sufmax > premax 就够了） |
| 关联题目 | [11. 盛最多水的容器](day10-双指针hot100+代码随想录.md)——对撞双指针前驱题 |

### 一句话记住

> **「左右对撞，维护两侧最大值，短板在哪边就结算哪边。」**

---

*练习日期：2026-07-19*
