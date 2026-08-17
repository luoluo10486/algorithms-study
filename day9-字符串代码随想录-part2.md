# Day 9 — 字符串代码随想录（剩余）

## 题目：反转字符串中的单词（Reverse Words in a String）

**LeetCode 151 | 代码随想录 | 难度：🟡 中等**

### 题目描述

给你一个字符串 `s`，请你反转字符串中**单词**的顺序。

单词是由非空格字符组成的字符串。`s` 中使用至少一个空格将字符串中的**单词**分隔开。

返回**单词顺序颠倒且单词之间用单个空格连接**的结果字符串。

**注意**：输入字符串 `s` 中可能会存在前导空格、尾随空格或者单词间的多个空格。返回的结果字符串中，单词间应当仅用单个空格分隔，且不包含任何额外的空格。

### 示例

```
输入：s = "the sky is blue"
输出："blue is sky the"

输入：s = "  hello world  "
输出："world hello"

输入：s = "a good   example"
输出："example good a"
```

---

## 解法：三步走（去空格 → 整体反转 → 单词内反转）

### 思路

不使用 `split` + 逆序这种取巧方式，手动模拟整个过程——这也是面试时更考察基本功的做法。

核心是**三步走**策略：

```
1. 去除多余空格（前导、尾随、单词间多个 → 一个）
2. 整体反转整个字符串
3. 逐个单词内部反转回来
```

为什么整体反转后再反转每个单词？因为反转后单词的顺序已经对了，但每个单词本身也被反转了——再反转回来就回到原样。两步反转相互抵消，最后只剩单词顺序的颠倒。

### 思考方式图解

```
s = "  hello world  "

Step 1: 去空格
  → "hello world"

Step 2: 整体反转
  → "dlrow olleh"

Step 3: 逐个单词反转
  "dlrow" → "world"
  "olleh" → "hello"
  → "world hello" ✅
```

```
s = "a good   example"

Step 1: 去空格
  → "a good example"

Step 2: 整体反转
  → "elpmaxe doog a"

Step 3: 逐个单词反转
  "elpmaxe" → "example"
  "doog" → "good"
  "a" → "a"
  → "example good a" ✅
```

### 代码实现

```java
class Solution {
    public String reverseWords(String s) {
        StringBuilder sb = removeSpace(s);              // Step 1: 去多余空格
        reverseString(sb, 0, sb.length() - 1);          // Step 2: 整体反转
        reversePerWords(sb);                            // Step 3: 反转每个单词
        return sb.toString();
    }

    // Step 1: 去除前导、尾随、单词间多余空格
    private StringBuilder removeSpace(String s) {
        int start = 0;
        int end = s.length() - 1;

        // 去掉前导空格
        while (start <= end && s.charAt(start) == ' ') {
            start++;
        }
        // 去掉尾随空格
        while (start <= end && s.charAt(end) == ' ') {
            end--;
        }

        StringBuilder sb = new StringBuilder();
        while (start <= end) {
            char c = s.charAt(start);
            if (c != ' ') {
                sb.append(c);                            // 字母直接加
            } else {
                // 空格：只有 sb 不为空且最后一位不是空格才加
                if (sb.length() > 0 && sb.charAt(sb.length() - 1) != ' ') {
                    sb.append(c);
                }
            }
            start++;
        }
        return sb;
    }

    // Step 2 & 3 通用：反转 StringBuilder 中 [start, end] 区间
    private void reverseString(StringBuilder sb, int start, int end) {
        while (start < end) {
            char temp = sb.charAt(start);
            sb.setCharAt(start, sb.charAt(end));
            sb.setCharAt(end, temp);
            start++;
            end--;
        }
    }

    // Step 3: 找到每个单词的边界，逐个反转
    private void reversePerWords(StringBuilder sb) {
        int start = 0;
        int end = 1;
        int n = sb.length();

        while (start < n) {
            // 找到单词的右边界（空格或末尾）
            while (end < n && sb.charAt(end) != ' ') {
                end++;
            }
            // 反转当前单词
            reverseString(sb, start, end - 1);
            // 移动到下一个单词
            start = end + 1;
            end = start + 1;
        }
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 三次线性遍历 |
| 🧠 空间复杂度 | **O(n)** — StringBuilder 存储结果 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 三步反转法 |
| 算法类型 | 字符串、双指针 |
| 核心技巧 | **去除多余空格 → 整体反转 → 每个单词内部反转（两步反转相互抵消）** |
| 关联题目 | [344. 反转字符串](day8-哈希表hot100+字符串代码随想录.md)——复用了双指针反转 |
| 去空格思路 | 用 `StringBuilder` 手动过滤，避免用 `split`（无法正确处理多个连续空格） |
| 易错点 | `reversePerWords` 中单词右边界是 `end-1`，因为 `end` 停在空格上；注意空字符串和全是空格的边界 |

### 一句话记住

> **「先去空格再整体反转，最后每个单词反转回来——两步反转抵消得顺序。」**

---

*练习日期：2026-07-17*
