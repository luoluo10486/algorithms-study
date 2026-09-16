# Day 35 — 滑动窗口 Hot100

> 本篇收录 Hot100「滑动窗口」板块两题：**3 无重复字符的最长子串**、**438 找到字符串中所有字母异位词**。
> 解法参照灵茶山艾府题解（变长滑动窗口模板）。

## 题目：无重复字符的最长子串（Longest Substring Without Repeating Characters）

**LeetCode 3 | 滑动窗口 Hot100 | 难度：🟡 中等**

### 题目描述

给定一个字符串 `s`，请你找出其中不含有重复字符的**最长子串**的长度。

### 示例

```
输入：s = "abcabcbb"
输出：3   （最长无重复子串是 "abc"）

输入：s = "bbbbb"
输出：1   （最长无重复子串是 "b"）

输入：s = "pwwkew"
输出：3   （最长无重复子串是 "wke"）
```

---

## 解法：变长滑动窗口 + 计数数组

### 思路

维护一个**不含重复字符**的窗口 `[left, right]`：

1. `right` 右移，把 `s[right]` 纳入窗口（计数 `cnt[c]++`）。
2. 若纳入后 `cnt[c] > 1`，说明窗口内出现了重复字母——不断右移 `left` 并把 `s[left]` 移出窗口（`cnt[s[left]]--`），直到窗口重新无重复。
3. 此时窗口 `[left, right]` 已是「以 `right` 结尾的最长无重复子串」，用 `right - left + 1` 更新答案。

**关键**：内层 `while` 里 `cnt[c] > 1` 的字母一定是刚进来的 `c`，所以只需围绕 `c` 收缩，左指针最多右移 n 次，均摊 O(n)。

### 思考方式图解

```
s = "abcabcbb"

right=0(a): cnt{a:1}                 [a]        ans=1
right=1(b): cnt{a:1,b:1}             [ab]       ans=2
right=2(c): cnt{a:1,b:1,c:1}         [abc]      ans=3
right=3(a): a 重复 → 移 a(left=1)     [bca]      ans=3
right=4(b): b 重复 → 移 b(left=2)     [cab]      ans=3
right=5(c): c 重复 → 移 c(left=3)     [abc]      ans=3
right=6(b): b 重复 → 移 a,b(left=5)   [cb]       ans=3
right=7(b): b 重复 → 移 c(left=6)     [b]        ans=3

结果：3 ✅
```

### 代码实现

```java
class Solution {
    public int lengthOfLongestSubstring(String S) {
        char[] s = S.toCharArray(); // 转成 char[] 加快效率（忽略带来的空间消耗）
        int n = s.length;
        int ans = 0;
        int left = 0;
        int[] cnt = new int[128]; // 也可用 HashMap<Character,Integer>，这里为效率用数组
        for (int right = 0; right < n; right++) {
            char c = s[right];
            cnt[c]++;
            while (cnt[c] > 1) {        // 窗口内有重复字母
                cnt[s[left]]--;         // 移除窗口左端点字母
                left++;                 // 缩小窗口
            }
            ans = Math.max(ans, right - left + 1); // 更新窗口长度最大值
        }
        return ans;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 每个字符最多被 left/right 各访问一次 |
| 🧠 空间复杂度 | **O(1)** — `int[128]` 为固定大小（字符集 ASCII） |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 变长滑动窗口 + 计数数组 |
| 算法类型 | 字符串、滑动窗口 |
| 核心技巧 | **右端点进入即计数，一旦某字母数 > 1 就收缩左端点，直到窗口重新无重复** |
| 收缩条件 | `while (cnt[c] > 1)` —— 重复的必是刚进来的 `c`，围绕它收缩即可 |
| 为什么用 int[128] | 覆盖 ASCII 字符，数组计数比 HashMap 快 |
| 关联题目 | [438. 找到字符串中所有字母异位词](day35-滑动窗口hot100.md)——同为「进入即计数、越界即收缩」的定长/变长窗口 |

### 一句话记住

> **「右进计数，重复就左缩——窗口里永远无重复。」**

---

## 题目：找到字符串中所有字母异位词（Find All Anagrams in a String）

**LeetCode 438 | 滑动窗口 Hot100 | 难度：🟡 中等**

### 题目描述

给定两个字符串 `s` 和 `p`，找到 `s` 中所有 `p` 的**字母异位词**的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

**字母异位词**：由相同字母重新排列形成的字符串（字母及出现次数完全相同）。

### 示例

```
输入：s = "cbaebabacd", p = "abc"
输出：[0,6]
解释：
  起始索引 0 的子串 "cba" 是 "abc" 的异位词
  起始索引 6 的子串 "bac" 是 "abc" 的异位词

输入：s = "abab", p = "ab"
输出：[0,1,2]
```

---

## 解法一：变长滑动窗口（推荐）

### 思路

把窗口当作「**最多包含 `p` 中字母各一份**」的容器：

1. `cnt` 初始为 `p` 的字母计数。
2. `right` 右移：`cnt[s[right]]--`（消耗一份该字母）。
3. 若 `cnt[c] < 0`，说明该字母在窗口内**超出**了 `p` 的配额——右移 `left` 把 `s[left]` 还回去（`cnt[s[left]]++`），直到 `cnt[c] == 0`。
4. 当窗口长度恰为 `p.length()` 时，窗口内各字母计数与 `p` 完全一致，`left` 即为一个答案。

**为什么长度相等就能保证一致**：窗口内所有字母计数都 ≥ 0（没有超额），且总字母数恰好等于 `p` 的长度，因此每类字母计数只能与 `p` 完全相同。

### 思考方式图解

```
s = "cbaebabacd", p = "abc"
cnt 初始 = {a:1, b:1, c:1}

right=0(c): cnt{c:0...} [c]          len<3 跳过
right=1(b): [cb]                     len<3 跳过
right=2(a): [cba]                    len=3 → ans=[0]
right=3(e): cnt{e:-1}<0 → 移 c(left=1)；e 仍 -1 → 移 b(left=2)；仍 -1 → 移 a(left=3) → [e]  len<3
right=4(b): [eb]                     len<3
right=5(a): [eba]                    len=3 → 检查字母不符（需 cnt 全 0）→ 实际 e 超额已在此前移位，此处窗口=[eba] 非异位 → 不加
right=6(b): [ebab]... 收缩至 [bab]   len=3 → ... 
right=9(d): 收缩后窗口=[bac]          len=3 → ans=[0,6]

结果：[0,6] ✅
```

> 注：图示为便于理解做了简化，实际以「窗口长度 == p.length()」且「无字母超额」为加答案条件。

### 代码实现

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        // 统计 p 的每种字母的出现次数
        int[] cnt = new int[26];
        for (char c : p.toCharArray()) {
            cnt[c - 'a']++;
        }

        List<Integer> ans = new ArrayList<>();
        int left = 0;
        for (int right = 0; right < s.length(); right++) {
            int c = s.charAt(right) - 'a';
            cnt[c]--;                     // 右端点字母进入窗口
            while (cnt[c] < 0) {          // 字母 c 太多了
                cnt[s.charAt(left) - 'a']++; // 左端点字母离开窗口
                left++;
            }
            if (right - left + 1 == p.length()) { // 窗口长度 == p.length()，字母计数必相同
                ans.add(left);            // t 左端点下标加入答案
            }
        }
        return ans;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n + m)** — n 为 s 长度，m 为 p 长度；left/right 各最多移动 n 次 |
| 🧠 空间复杂度 | **O(1)** — `int[26]` 固定大小 |

---

## 解法二：定长滑动窗口（对照）

### 思路

固定窗口长度为 `p.length()`，每次滑动一格：

1. `cntP` 统计 `p` 的字母计数；`cntS` 统计当前窗口（长度固定为 `p.length()`）的字母计数。
2. `right` 右移，`cntS[s[right]]++`；窗口左端点 `left = right - p.length() + 1`。
3. 窗口长度不足时（`left < 0`）跳过。
4. 用 `Arrays.equals(cntS, cntP)` 判断是否异位词，是则把 `left` 加入答案。
5. 把 `s[left]` 移出窗口（`cntS[s[left]]--`），为下一格做准备。

### 代码实现

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        // 统计 p 的每种字母的出现次数
        int[] cntP = new int[26];
        for (char c : p.toCharArray()) {
            cntP[c - 'a']++; // 统计 p 的字母
        }

        List<Integer> ans = new ArrayList<>();
        int[] cntS = new int[26]; // 统计 s 的长为 p.length() 的子串 t 的字母计数
        for (int right = 0; right < s.length(); right++) {
            cntS[s.charAt(right) - 'a']++; // 右端点字母进入窗口
            int left = right - p.length() + 1;
            if (left < 0) { // 窗口长度不足 p.length()
                continue;
            }
            if (Arrays.equals(cntS, cntP)) { // t 和 p 的每种字母的出现次数都相同
                ans.add(left); // t 左端点下标加入答案
            }
            cntS[s.charAt(left) - 'a']--; // 左端点字母离开窗口
        }
        return ans;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(26·n) ≈ O(n)** — 每步 `Arrays.equals` 比对 26 个字母 |
| 🧠 空间复杂度 | **O(1)** — 两个 `int[26]` |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 滑动窗口 + 计数数组 |
| 算法类型 | 字符串、滑动窗口 |
| 核心技巧 | 解法一：**进入即扣减配额，超额就左缩，长度达标即答案**；解法二：**窗口定长，比对两个计数数组** |
| ✅ 面试最优解 | **解法一（变长窗口）**——一个数组、无 `Arrays.equals`，时间常数更小、写法更紧凑 |
| 解法二定位 | 定长窗口写法更直白，但每步 O(26) 比对，作为对照理解 |
| 关联题目 | [3. 无重复字符的最长子串](day35-滑动窗口hot100.md)——同属「进入即计数、越界即收缩」的滑动窗口 |

### 一句话记住

> **「变长窗口：进一个扣一份配额，超额就左缩；窗口长度等于 p 时即异位词。」**

---

*练习日期：2026-09-16*
