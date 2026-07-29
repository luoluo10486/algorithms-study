# Day 20 — 子串 Hot100

## 题目：和为 K 的子数组（Subarray Sum Equals K）

**LeetCode 560 | 子串 Hot100 | 难度：🟡 中等**

### 题目描述

给你一个整数数组 `nums` 和一个整数 `k`，请你统计并返回该数组中和为 `k` 的连续子数组的个数。

### 示例

```
输入：nums = [1,1,1], k = 2
输出：2  （[1,1] 出现两次）

输入：nums = [1,2,3], k = 3
输出：2  （[1,2] 和 [3]）
```

---

## 解法：前缀和 + HashMap

### 思路

暴力枚举所有子数组 O(n²)，用**前缀和 + HashMap** 降到 **O(n)**。

**核心思想**：子数组 `nums[j..i]` 的和 = `s[i+1] - s[j]`（其中 `s` 是前缀和数组，`s[0]=0`）。

对于当前前缀和 `s[i+1]`，如果存在某个更早的前缀和 `s[j]` 满足 `s[i+1] - s[j] = k`，即 `s[j] = s[i+1] - k`，那么子数组 `nums[j..i]` 的和就是 k。

所以只需遍历前缀和数组，一边用 HashMap 记录各前缀和出现的次数，一边累加 `cnt[cur - k]`。

```
前缀和数组 s，s[0]=0

遍历 s 中的每个前缀和 sc：
  1. ans += cnt[sc - k]   ← 以当前位置为终点，有多少个起点使子数组和为 k
  2. cnt[sc]++             ← 记录当前前缀和
```

### 思考方式图解

```
nums = [1, 2, 3], k = 3

前缀和数组 s = [0, 1, 3, 6]

遍历过程：
  sc=0:  cnt[0-3]=cnt[-3]=0 → ans=0    cnt={0:1}
  sc=1:  cnt[1-3]=cnt[-2]=0 → ans=0    cnt={0:1, 1:1}
  sc=3:  cnt[3-3]=cnt[0]=1  → ans=1    cnt={0:1, 1:1, 3:1}
  sc=6:  cnt[6-3]=cnt[3]=1  → ans=2    cnt={0:1, 1:1, 3:1, 6:1}

结果：2 ✅（[1,2] 和 [3]）
```

### 代码实现

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        int n = nums.length;
        // 前缀和数组 s，s[0]=0
        int[] s = new int[n + 1];
        for (int i = 0; i < n; i++) {
            s[i + 1] = s[i] + nums[i];
        }

        Map<Integer, Integer> cnt = new HashMap<>();
        int ans = 0;

        for (int sc : s) {            // sc = s[i+1]
            // 以 sc 为终点，有多少个起点使得子数组和为 k
            ans += cnt.getOrDefault(sc - k, 0);
            // 记录当前前缀和
            cnt.put(sc, cnt.getOrDefault(sc, 0) + 1);
        }

        return ans;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 一次遍历 |
| 🧠 空间复杂度 | **O(n)** — 前缀和数组 + HashMap |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 前缀和 + HashMap |
| 算法类型 | 数组、前缀和 |
| 核心技巧 | **前缀和相减得子数组和：s[i+1] - s[j] = k → s[j] = s[i+1] - k → HashMap 查 cnt[s[i+1] - k]** |
| 初始化 cnt[0]=1 | 遍历 s 的第一个元素就是 0，put 后 cnt={0:1}，这样第一个前缀和本身等于 k 时也能被统计到 |
| 关联题目 | [437. 路径总和 III](day18-二叉树hot100-part4.md)——完全相同的思路，前缀和从数组搬到了二叉树 |

### 一句话记住

> **「前缀和相减得子数组，HashMap 统计出现次数。」**

---

## 题目：滑动窗口最大值（Sliding Window Maximum）

**LeetCode 239 | 子串 Hot100 | 难度：🔴 困难**

### 题目描述

给你一个整数数组 `nums`，有一个大小为 `k` 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 `k` 个数字。滑动窗口每次只向右移动一位。

返回**每个滑动窗口中的最大值**。

### 示例

```
输入：nums = [1,3,-1,-3,5,3,6,7], k = 3
输出：[3,3,5,5,6,7]

输入：nums = [1], k = 1
输出：[1]
```

---

## 解法：单调队列（双端队列）

### 思路

暴力解法是每个窗口扫一遍 O(n·k)，不可接受。

用一个**单调递减队列**，队列中存的是元素的下标，对应的值是递减的，在 O(n) 时间内解决。

```
单调递减队列的三条规则：

1. 入队（添加新元素）：
   while 队尾元素的值 ≤ 当前元素 → 队尾出队（它不可能是最大值了）
   当前元素入队

2. 出队（移除过期元素）：
   if 队首元素下标 < 窗口左边界 → 队首出队（不在窗口中了）

3. 记录结果：
   if 窗口已形成（i ≥ k-1）→ 队首元素就是当前窗口最大值
```

### 思考方式图解

```
nums = [1,3,-1,-3,5,3,6,7], k = 3

i=0(1):  q=[] → 入队0                  q=[0]
i=1(3):  q=[0], 1≤3 → 移除0，入队1     q=[1]
i=2(-1): q=[1], 3>-1 → 入队2           q=[1,2]
         left=0, 队首1≥0 → 有效        ans[0]=nums[1]=3 ✅
i=3(-3): q=[1,2], -1>-3 → 入队3       q=[1,2,3]
         left=1, 队首1≥1 → 有效        ans[1]=nums[1]=3 ✅
i=4(5):  q=[1,2,3], -3,-1,3 全部 ≤5 → 全部移除，入队4  q=[4]
         left=2, 队首4≥2 → 有效        ans[2]=nums[4]=5 ✅
i=5(3):  q=[4], 5>3 → 入队5           q=[4,5]
         left=3, 队首4≥3 → 有效        ans[3]=nums[4]=5 ✅
i=6(6):  q=[4,5], 3≤6 → 移除5
                5≤6 → 移除4，入队6       q=[6]
         left=4, 队首6≥4 → 有效        ans[4]=nums[6]=6 ✅
i=7(7):  q=[6], 6≤7 → 移除6，入队7     q=[7]
         left=5, 队首7≥5 → 有效        ans[5]=nums[7]=7 ✅

结果：[3,3,5,5,6,7] ✅
```

### 代码实现

```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        int n = nums.length;
        int[] ans = new int[n - k + 1];
        Deque<Integer> q = new ArrayDeque<>();  // 存下标，单调递减

        for (int i = 0; i < n; i++) {
            // 1. 入队：移除所有 ≤ 当前元素的队尾元素
            while (!q.isEmpty() && nums[q.getLast()] < nums[i]) {
                q.removeLast();
            }
            q.addLast(i);

            // 2. 出队：移除已移出窗口的队首元素
            int left = i - k + 1;
            if (q.getFirst() < left) {
                q.removeFirst();
            }

            // 3. 记录结果：窗口形成后，队首就是最大��
            if (left >= 0) {
                ans[left] = nums[q.getFirst()];
            }
        }

        return ans;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 每个元素入队一次、出队最多一次 |
| 🧠 空间复杂度 | **O(k)** — 队列最多同时存放 k 个元素 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 单调队列（双端队列） |
| 算法类型 | 队列、滑动窗口 |
| 核心技巧 | **单调递减队列：新元素入队时清掉所有 ≤ 它的队尾，淘汰过期元素时看队首是否 < 左边界** |
| 为什么存下标 | 只有存下标才能判断元素是否过期 |
| 关联题目 | [76. 最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/)——另一类滑动窗口问题 |

### 一句话记住

> **「单调递减队列，新来大的清队尾，过期小的清队首。」**

---

## 题目：最小覆盖子串（Minimum Window Substring）

**LeetCode 76 | 子串 Hot100 | 难度：🔴 困难**

### 题目描述

给你一个字符串 `s` 和一个字符串 `t`。返回 `s` 中涵盖 `t` 所有字符的最小子串。如果 `s` 中不存在涵盖 `t` 所有字符的子串，则返回空字符串 `""`。

### 示例

```
输入：s = "ADOBECODEBANC", t = "ABC"
输出："BANC"

输入：s = "a", t = "a"
输出："a"

输入：s = "a", t = "aa"
输出：""
```

---

## 解法：滑动窗口 + 计数数组

### 思路

滑动窗口的经典应用——**右指针扩展窗口直到涵盖 t，然后左指针收缩窗口找最短**。

```
1. 用计数数组 cntT 统计 t 中各字符的出现次数
2. 用计数数组 cntS 统计当前窗口内各字符的出现次数
3. 右指针 right 不断右移，将字符纳入窗口
4. 当窗口涵盖 t（isCovered 返回 true）时：
   - 如果当前窗口比已记录的最短窗口更短，更新记录
   - 左指针 left 右移，移出字符，收缩窗口
5. 重复直到 right 走完整个 s
```

### 思考方式图解

```
s = "ADOBECODEBANC", t = "ABC"

right=5, 窗口=[ADOBEC] → 涵盖 ✅
  left=0: 记录 [ADOBEC] 长度6
  left=1(移A): [DOBEC] → 缺A → 不再涵盖 ❌

right=10, 窗口=[DOBECODEBA] → 涵盖 ✅
  收缩到 left=5(移C): [ODEBA] → 缺C → 不再涵盖 ❌
  记录 [CODEBA] 长度5（更新）

right=12, 窗口=[ODEBANC] → 涵盖 ✅
  收缩到 left=9(移B): [ANC] → 缺B → 不再涵盖 ❌
  记录 [BANC] 长度4（更新 ✅ 最短）

结果："BANC"
```

### 代码实现

```java
class Solution {
    public String minWindow(String s, String t) {
        int[] cntS = new int[128];
        int[] cntT = new int[128];
        for (char c : t.toCharArray()) {
            cntT[c]++;
        }

        char[] S = s.toCharArray();
        int m = S.length;
        int ansLeft = -1;
        int ansRight = m;
        int left = 0;

        for (int right = 0; right < m; right++) {
            cntS[S[right]]++;  // 右端点字母进入窗口

            while (isCovered(cntS, cntT)) {  // 窗口已涵盖 t
                if (right - left < ansRight - ansLeft) {
                    ansLeft = left;
                    ansRight = right;
                }
                cntS[S[left]]--;  // 左端点字母移出窗口
                left++;
            }
        }

        return ansLeft < 0 ? "" : s.substring(ansLeft, ansRight + 1);
    }

    // 检查 cntS 是否涵盖了 cntT（即每个字母出现次数 ≥ t 中的次数）
    private boolean isCovered(int[] cntS, int[] cntT) {
        for (int i = 'A'; i <= 'Z'; i++) {
            if (cntS[i] < cntT[i]) return false;
        }
        for (int i = 'a'; i <= 'z'; i++) {
            if (cntS[i] < cntT[i]) return false;
        }
        return true;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(m·Σ)** — m 为 s 长度，Σ=52（大小写字母），每次 isCovered 检查 52 个字符 |
| 🧠 空间复杂度 | **O(128) = O(1)** — 两个固定大小的计数数组 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 滑动窗口 + 计数数组 |
| 算法类型 | 字符串、滑动窗口 |
| 核心技巧 | **右扩到涵盖，左缩到最短——双指针维护最短覆盖子串** |
| 计数数组 | `int[128]` 覆盖 ASCII 码，比 HashMap 更快 |
| 为什么用 char[] S | 字符数组比 String.charAt() 更快（常量级优化） |
| 关联题目 | [560. 和为 K 的子数组](day20-子串hot100.md)——另一类子串问题（前缀和） |

### 一句话记住

> **「右扩到涵盖，左缩到最短——双指针滚动找最小覆盖。」**

---

*练习日期：2026-07-29*
