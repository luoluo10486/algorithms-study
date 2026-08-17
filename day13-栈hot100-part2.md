# Day 13 — 栈 Hot100（剩余）

## 题目：每日温度（Daily Temperatures）

**LeetCode 739 | 栈 Hot100 | 难度：🟡 中等**

### 题目描述

给定一个整数数组 `temperatures`，表示每天的温度，返回一个数组 `answer`，其中 `answer[i]` 是指对于第 `i` 天，下一个更高温度出现在几天后。如果气温在这之后都不会升高，请在该位置用 `0` 来代替。

### 示例

```
输入：temperatures = [73,74,75,71,69,72,76,73]
输出：[1,1,4,2,1,1,0,0]

输入：temperatures = [30,40,50,60]
输出：[1,1,1,0]

输入：temperatures = [30,60,90]
输出：[1,1,0]
```

---

## 解法：单调栈（递减栈）

### 思路

暴力解是每个位置往后扫，O(n²)。单调栈降到 **O(n)**。

核心思想：**维护一个栈，存的是温度数组的下标，栈内下标对应的温度保持递减（从栈底到栈顶）。**

```
单调递减栈的规则：

遇到新温度 t：
  while 栈不空 且 t > 栈顶下标对应的温度：
      说明"下一个更高温度"找到了 → 弹出栈顶下标 j，计算天数差 ans[j] = i - j
  当前温度入栈（作为未来的"候选"）
```

**为什么是递减？** 因为栈里存的是还没遇到更高温度的日子。一旦新温度比栈顶高，栈顶那天的答案就确定了。

### 思考方式图解

```
temperatures = [73, 74, 75, 71, 69, 72, 76, 73]

i=0: t=73, st=[] → 入栈0                  st=[0]
i=1: t=74, st=[0], 74>73 → pop0, ans[0]=1
            入栈1                          st=[1]
i=2: t=75, st=[1], 75>74 → pop1, ans[1]=1
            入栈2                          st=[2]
i=3: t=71, st=[2], 71≤75 → 入栈3           st=[2,3]
i=4: t=69, st=[2,3], 69≤71 → 入栈4         st=[2,3,4]
i=5: t=72, st=[2,3,4]
     72>69 → pop4, ans[4]=1
     72>71 → pop3, ans[3]=2
     72≤75 → 停止
            入栈5                          st=[2,5]
i=6: t=76, st=[2,5]
     76>72 → pop5, ans[5]=1
     76>75 → pop2, ans[2]=4
            入栈6                          st=[6]
i=7: t=73, st=[6], 73≤76 → 入栈7           st=[6,7]

ans = [1,1,4,2,1,1,0,0] ✅
```

### 代码实现

```java
class Solution {
    public int[] dailyTemperatures(int[] temperatures) {
        int n = temperatures.length;
        int[] ans = new int[n];
        Deque<Integer> st = new ArrayDeque<>();   // 存下标，单调递减

        for (int i = 0; i < n; i++) {
            int t = temperatures[i];
            // 当前温度比栈顶高 → 栈顶那天的答案确定了
            while (!st.isEmpty() && t > temperatures[st.peek()]) {
                int j = st.pop();
                ans[j] = i - j;                    // 天数差
            }
            st.push(i);                            // 当前天入栈（等待未来更高温度）
        }

        return ans;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 每个下标入栈一次、出栈最多一次 |
| 🧠 空间复杂度 | **O(n)** — 栈最多存放 n 个元素 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 单调栈（递减栈） |
| 算法类型 | 栈、数组 |
| 核心技巧 | **栈存下标，保持栈内温度递减；新温度 > 栈顶时，栈顶那天的答案就确定了** |
| 适用场景 | 找下一个更大的元素 / 下一个更高的温度 / 下一个更大的数 |
| 关联题目 | [496. 下一个更大元素 I](https://leetcode.cn/problems/next-greater-element-i/)——单调栈模板延伸 |
| 易错点 | 栈里存的是**下标**，不是温度值；比较时用 `t > temperatures[st.peek()]`；天数差是 `i - j` |

### 一句话记住

> **「单调递减栈存日子，遇到更高就出栈算天数。」**

---

## 题目：柱状图中最大的矩形（Largest Rectangle in Histogram）

**LeetCode 84 | 栈 Hot100 | 难度：🔴 困难**

### 题目描述

给定 n 个非负整数，用来表示柱状图中各个柱子的高度。每个柱子彼此相邻，且宽度为 1。

求在该柱状图中，能够勾勒出来的矩形的最大面积。

### 示例

```
输入：heights = [2,1,5,6,2,3]
输出：10
解释：由高度 5 和柱子 6 形成的矩形面积为 5×2=10

输入：heights = [2,4]
输出：4
```

---

## 解法：单调栈（左右各扫一遍）

### 思路

对于每根柱子，它能形成的最大矩形 = **高度 × 宽度**，其中宽度由「左边第一个比它矮的柱子」和「右边第一个比它矮的柱子」决定。

```
以 heights[i] 为高能形成的最大矩形：
  左边界 left[i] = 左边第一个 < heights[i] 的柱子下标（不存在则为 -1）
  右边界 right[i] = 右边第一个 < heights[i] 的柱子下标（不存在则为 n）
  宽度 = right[i] - left[i] - 1
```

核心思想：**用单调递增栈（栈内柱子高度递增），左边界扫一遍，右边界扫一遍，汇总结果。**

```
为什么是递增栈？
  栈里存的是"还没找到右边更矮柱子的下标"
  当新柱子 h ≤ 栈顶高度时，栈顶柱子的右边界就确定了（就是 i）
  同时，左边第一个更矮的柱子就是栈顶的下一个元素
```

### 思考方式图解

```
heights = [2, 1, 5, 6, 2, 3]

扫描左边界 left[]（左边第一个 < heights[i] 的下标）：
  i=0(2): st=[] → left[0]=-1, push0     st=[0]
  i=1(1): st=[0], 1≤2 → pop0, st=[] → left[1]=-1, push1     st=[1]
  i=2(5): st=[1], 5>1 → left[2]=1, push2     st=[1,2]
  i=3(6): st=[1,2], 6>5 → left[3]=2, push3     st=[1,2,3]
  i=4(2): st=[1,2,3], 2≤6 → pop3
           st=[1,2], 2≤5 → pop2
           st=[1], 2>1 → left[4]=1, push4   st=[1,4]
  i=5(3): st=[1,4], 3>2 → left[5]=4, push5     st=[1,4,5]

扫描右边界 right[]（从右往左，同样逻辑）：
  i=5(3): st=[] → right[5]=6, push5      st=[5]
  i=4(2): st=[5], 2≤3 → pop5, st=[] → right[4]=6, push4  st=[4]
  i=3(6): st=[4], 6>2 → right[3]=4, push3     st=[4,3]
  i=2(5): st=[4,3], 5≤6 → pop3
           st=[4], 5>2 → right[2]=4, push2     st=[4,2]
  i=1(1): st=[4,2], 1≤2 → pop2
           st=[4], 1≤2 → pop4, st=[]
            → right[1]=-1, push1    st=[1]
  i=0(2): st=[1], 2>1 → right[0]=1, push0   st=[1,0]

汇总面积：
  i=0: h=2, w=right[0]-left[0]-1=1-(-1)-1=1  → 2×1=2
  i=1: h=1, w=-1-(-1)-1=6                     → 1×6=6
  i=2: h=5, w=4-1-1=2                        → 5×2=10 ✅
  i=3: h=6, w=4-2-1=1                        → 6×1=6
  i=4: h=2, w=6-1-1=4                        → 2×4=8
  i=5: h=3, w=6-4-1=1                        → 3×1=3

答案：10 ✅
```

### 代码实现

```java
class Solution {
    public int largestRectangleArea(int[] heights) {
        int n = heights.length;
        int[] left = new int[n];     // 左边第一个 < heights[i] 的下标
        Deque<Integer> st = new ArrayDeque<>();

        // 扫描左边界
        for (int i = 0; i < n; i++) {
            int h = heights[i];
            while (!st.isEmpty() && h <= heights[st.peek()]) {
                st.pop();            // 栈顶高度 ≥ 当前高度，出栈（不可能是左边界）
            }
            left[i] = st.isEmpty() ? -1 : st.peek();  // 左边界：栈顶的下一个元素
            st.push(i);
        }

        // 扫描右边界（从右往左）
        int[] right = new int[n];
        st.clear();
        for (int i = n - 1; i >= 0; i--) {
            int h = heights[i];
            while (!st.isEmpty() && h <= heights[st.peek()]) {
                st.pop();
            }
            right[i] = st.isEmpty() ? n : st.peek();  // 右边界：栈顶的下一个元素
            st.push(i);
        }

        // 汇总面积
        int ans = 0;
        for (int i = 0; i < n; i++) {
            int width = right[i] - left[i] - 1;
            ans = Math.max(ans, width * heights[i]);
        }
        return ans;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 三遍线性扫描 |
| 🧠 空间复杂度 | **O(n)** — left + right + 栈 |

---

### 💡 优化：一次遍历确定左右边界

上述解法需要两遍扫描（左→右求 left，右→左求 right），灵茶山艾府的版本用**一次遍历同时确定左右边界**——核心思路是：

```
当 heights[st.peek()] >= heights[i]，弹出栈顶 j 时：
  → 栈顶 j 的右边界就是 i（因为 i 是右边第一个 ≤ heights[j] 的）
  → 栈顶 j 的左边界就是新的栈顶（因为栈里存的是左边第一个比 heights[j] 小的）
```

**关键区别**：
- `left[i]` 在**入栈前**确定（看栈顶剩下的元素）
- `right[i]` 在**出栈时**确定（弹出的元素的右边界就是当前 i）
- 未出栈的元素（遍历结束后还在栈里的），其右边界保持初始值 `n`

```java
class Solution {
    public int largestRectangleArea(int[] heights) {
        int n = heights.length;
        int[] left = new int[n];
        int[] right = new int[n];
        Arrays.fill(right, n);            // 默认右边界为 n
        Deque<Integer> st = new ArrayDeque<>();

        for (int i = 0; i < n; i++) {
            int h = heights[i];
            // 弹出所有 ≥ 当前高度的栈顶元素
            while (!st.isEmpty() && heights[st.peek()] >= h) {
                right[st.pop()] = i;      // 被弹出的元素，右边界就是 i
            }
            // 此时栈顶就是左边第一个 < heights[i] 的元素
            left[i] = st.isEmpty() ? -1 : st.peek();
            st.push(i);
        }

        // 汇总面积（left/right 均已就绪）
        int ans = 0;
        for (int i = 0; i < n; i++) {
            ans = Math.max(ans, heights[i] * (right[i] - left[i] - 1));
        }
        return ans;
    }
}
```

---

### 💡 终极优化：哨兵栈 + 单次遍历（灵茶山艾府）

上面的一遍扫描法仍然需要两个数组和一个额外的汇总循环。灵茶山艾府的**终极版**用一个技巧把空间和代码量压缩到极致——**栈中直接塞哨兵 `-1`，出栈时当场算面积**。

**核心思想：**

```
1. 栈底放哨兵 -1（代替 left[i] = -1 的判空逻辑，保证栈永不为空）
2. 遍历到 i = n 时，虚拟高度为 -1，触发所有剩余元素出栈
3. 出栈时：right = i（当前下标），left = 栈顶（栈顶的下面那个），当场算面积
```

**为什么栈底放 -1？**

不用在求 left 时判空了——`st.peek()` 拿到 -1 就相当于 left = -1。同时 `st.size() > 1` 保证了栈里始终至少有一个哨兵，不会出现空栈异常。

**为什么 right 要遍历到 n？**

当 `right = n` 时，虚拟高度 `h = -1` 比任何柱子都矮，会触发栈中所有剩余元素出栈，确保每根柱子的面积都被计算过。这相当于一次性完成「正常遍历 + 清栈」两个任务。

```java
class Solution {
    public int largestRectangleArea(int[] heights) {
        int n = heights.length;
        Deque<Integer> st = new ArrayDeque<>();
        st.push(-1); // 哨兵，栈永不为空，对应 left[i] = -1 的情况
        int ans = 0;

        // right 遍历到 n（用虚拟高度 -1 强制清空栈）
        for (int right = 0; right <= n; right++) {
            int h = right < n ? heights[right] : -1; // 虚拟高度 -1

            // 当前高度 ≤ 栈顶高度 → 栈顶的右边界确定了
            while (st.size() > 1 && heights[st.peek()] >= h) {
                int i = st.pop();          // 矩形的高（的下标）
                int left = st.peek();      // 栈顶下面那个数就是 left
                ans = Math.max(ans, heights[i] * (right - left - 1));
            }
            st.push(right);                // 当前下标入栈
        }

        return ans;
    }
}
```

### 思考方式图解

```
heights = [2, 1, 5, 6, 2, 3]

初始：st = [-1]

right=0(2): st=[-1], -1<2 → 入栈0          st=[-1, 0]
right=1(1): st=[-1,0], 2≥1 → pop0, 算面积
    i=0, left=-1, right=1, w=2, h=2 → ans=4
    st=[-1], -1<1 → 入栈1                  st=[-1, 1]
right=2(5): st=[-1,1], 1<5 → 入栈2         st=[-1, 1, 2]
right=3(6): st=[-1,1,2], 5<6 → 入栈3      st=[-1, 1, 2, 3]
right=4(2): st=[-1,1,2,3], 6≥2 → pop3, 算面积
    i=3, left=2, right=4, w=1, h=6 → ans=6
    st=[-1,1,2], 5≥2 → pop2, 算面积
    i=2, left=1, right=4, w=2, h=5 → ans=10 ✅
    st=[-1,1], 1<2 → 入栈4                st=[-1, 1, 4]
right=5(3): st=[-1,1,4], 2<3 → 入栈5       st=[-1, 1, 4, 5]
right=6(-1): 虚拟高度 -1
    st=[-1,1,4,5], 3≥-1 → pop5, 算面积
    i=5, left=4, right=6, w=1, h=3 → ans=10
    st=[-1,1,4], 2≥-1 → pop4, 算面积
    i=4, left=1, right=6, w=4, h=2 → ans=10
    st=[-1,1], 1≥-1 → pop1, 算面积
    i=1, left=-1, right=6, w=6, h=1 → ans=10
    st.size()=1 → 停止（只剩哨兵 -1）

结果：10 ✅
```

**三种写法对比：**

| | 两遍扫描 | 一遍扫描（left/right 数组） | 一遍扫描（哨兵栈） |
|---|---------|--------------------------|-----------------|
| 遍历次数 | 3 次 | 2 次 | **1 次** |
| right[] 数组 | 需要 | 需要 | **不需要** |
| left[] 数组 | 需要 | 需要 | **不需要** |
| 判空 | 三元表达式 | 三元表达式 | **哨兵 -1** |
| 清栈 | 不需要 | 不需要 | **虚拟高度 -1 强制清栈** |
| 代码量 | 多 | 中等 | **最精简** |

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 单调栈（递增栈） |
| 算法类型 | 栈、数组 |
| 核心技巧 | **每根柱子作为高时，最大矩形宽度由左右第一个更矮的柱子决定——单调递增栈找边界** |
| 三种写法 | 两遍扫描 / 一遍扫描+left/right 数组 / **一遍扫描+哨兵栈**（最推荐） |
| 哨兵栈精髓 | 栈底塞 `-1` 代替判空，遍历到 n 设虚拟高度 `-1` 强制清栈 |
| 左边界默认 -1 | 左边没有更矮的柱子，宽度从 0 开始 |
| 右边界默认 n | 右边没有更矮的柱子，宽度到 n-1 结束 |
| 关联题目 | [42. 接雨水](day10-双指针hot100+代码随想录.md)——同样是柱子问题，但思路相反 |
| 易错点 | `st.size() > 1` 保证哨兵不被弹出；虚拟高度用 `-1` 保证比所有柱子都矮 |
