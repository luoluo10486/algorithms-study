# Day 8 — 哈希表 Hot100 + 字符串（代码随想录）

## 题目：两数之和（Two Sum）

**LeetCode 1 | 哈希表 Hot100 | 难度：🟢 简单**

### 题目描述

给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出**和为目标值 `target`** 的那**两个**整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案，并且你不能使用两次相同的元素。

你可以按任意顺序返回答案。

### 示例

```
输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9，返回 [0, 1]

输入：nums = [3,2,4], target = 6
输出：[1,2]

输入：nums = [3,3], target = 6
输出：[0,1]
```

---

## 解法：HashMap（空间换时间）

### 思路

暴力两层循环是 O(n²)，用 HashMap 降到 O(n)。

核心思想：**遍历数组时，把「已经见过的元素」存到 map 中——对于当前元素 `nums[i]`，只需要查 `target - nums[i]` 是否在 map 中就行**。

**关键细节**：先查后存。先检查 map 中是否有配对的元素，再把当前元素入 map。这保证了不会用同一个元素两次。

### 思考方式图解

```
nums = [2, 7, 11, 15], target = 9

i=0: nums[0]=2, temp=7
     map 为空 → 没找到
     存入 map: {2→0}

i=1: nums[1]=7, temp=2
     map 中有 2 → 找到了！
     返回 [map.get(2), 1] = [0, 1] ✅
```

```
nums = [3, 3], target = 6

i=0: nums[0]=3, temp=3
     map 为空 → 没找到
     存入 map: {3→0}

i=1: nums[1]=3, temp=3
     map 中有 3 → 找到了！
     返回 [map.get(3), 1] = [0, 1] ✅
```

如果先存后查，i=1 时会把自己匹配给自己（`map.get(3) == 1`），返回 `[1, 1]` —— 错误。

### 代码实现

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int[] res = new int[2];
        Map<Integer, Integer> map = new HashMap<>();

        for (int i = 0; i < nums.length; i++) {
            int temp = target - nums[i];       // 需要的另一个数
            if (map.containsKey(temp)) {
                res[1] = i;
                res[0] = map.get(temp);
                break;
            }
            map.put(nums[i], i);               // 把当前元素存入 map
        }

        return res;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 只遍历一次数组 |
| 🧠 空间复杂度 | **O(n)** — 哈希表最多存放 n 个元素 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 哈希表（空间换时间） |
| 算法类型 | 哈希表、数组 |
| 核心技巧 | **用 HashMap 存储已遍历元素的「值→下标」映射，实现 O(1) 查找配对** |
| 适用场景 | 两数之和、三数之和（配合双指针）、四数之和等系列问题的基础 |
| 易错点 | **先查后存**——先查 map 中是否有 `target-nums[i]`，再把自己入 map，否则会自我匹配 |

### 一句话记住

> **「查 map 找另一半，先查后存不重复。」**

---

## 题目：两数之和（Two Sum）—— 延伸

**哈希表 Hot100 中最经典的入门题**，很多哈希表题都是它的变体：
- 三数之和（15）—— 固定一个 + 双指针扫另外两个
- 四数之和（18）—— 固定两个 + 双指针扫另外两个
- 两数之和 II（167）—— 有序数组版，用双指针 O(n)
- 和为 K 的子数组（560）—— 前缀和 + HashMap

---

## 题目：字母异位词分组（Group Anagrams）

**LeetCode 49 | 哈希表 Hot100 | 难度：🟡 中等**

### 题目描述

给你一个字符串数组，请你将**字母异位词**组合在一起。可以按任意顺序返回结果列表。

字母异位词是由重新排列源单词的所有字母得到的一个新单词。

### 示例

```
输入：strs = ["eat", "tea", "tan", "ate", "nat", "bat"]
输出：[["bat"],["nat","tan"],["ate","eat","tea"]]

输入：strs = [""]
输出：[[""]]

输入：strs = ["a"]
输出：[["a"]]
```

---

## 解法：排序 + HashMap

### 思路

把字母异位词映射到同一个 key 上——**排序后的字符串就是天然的 key**。

```
"eat" 排序 → "aet"  ←  key
"tea" 排序 → "aet"  ←  同一个 key
"tan" 排序 → "ant"  ←  不同 key
```

这样就不需要两两比较是否互为异位词了，一次遍历搞定分组。

### 思考方式图解

```
strs = ["eat", "tea", "tan", "ate", "nat", "bat"]

遍历过程：
  "eat" → 排序 "aet" → map: {"aet": ["eat"]}
  "tea" → 排序 "aet" → map: {"aet": ["eat", "tea"]}
  "tan" → 排序 "ant" → map: {"aet": ["eat", "tea"], "ant": ["tan"]}
  "ate" → 排序 "aet" → map: {"aet": ["eat", "tea", "ate"], "ant": ["tan"]}
  "nat" → 排序 "ant" → map: {"aet": [...], "ant": ["tan", "nat"]}
  "bat" → 排序 "abt" → map: {"aet": [...], "ant": [...], "abt": ["bat"]}

结果：[[eat, tea, ate], [tan, nat], [bat]]
```

### 代码实现

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> map = new HashMap<>();

        for (String str : strs) {
            // 对每个字符串排序，得到唯一的 key
            char[] arrays = str.toCharArray();
            Arrays.sort(arrays);
            String key = new String(arrays);

            // key 不存在时创建新列表
            if (map.get(key) == null) {
                List<String> newList = new ArrayList<>();
                newList.add(str);
                map.put(key, newList);
            } else {
                map.get(key).add(str);
            }
        }

        return new ArrayList<List<String>>(map.values());
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(nk log k)** — n 为字符串个数，k 为最长字符串长度（排序开销） |
| 🧠 空间复杂度 | **O(nk)** — 存储所有字符串到 HashMap |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 排序 + HashMap |
| 算法类型 | 哈希表、字符串 |
| 核心技巧 | **排序字符串作为 HashMap 的 key，异位词天然归到同一个 key 下** |
| 关联题目 | [242. 有效的字母异位词](day6-哈希表代码随想录.md)——判断两个字符串是否互为异位词 |
| 优化方向 | 如果用计数数组代替排序（26 个字母的 int[] 拼接成 key），可将 k log k 降为 **O(k)** |
| 易错点 | `map.get(key) == null` 时记得先 `new` 再 `put`，或用 `computeIfAbsent` 简化 |

### 一句话记住

> **「排完序就是同一个妈——异位词分组，排序做 key 全拿下。」**

---

## 题目：最长连续序列（Longest Consecutive Sequence）

**LeetCode 128 | 哈希表 Hot100 | 难度：🟡 中等**

### 题目描述

给定一个未排序的整数数组 `nums`，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。

请你设计并实现时间复杂度为 **O(n)** 的算法解决此问题。

### 示例

```
输入：nums = [100,4,200,1,3,2]
输出：4
解释：最长数字连续序列是 [1,2,3,4]，长度为 4

输入：nums = [0,3,7,2,5,8,4,6,0,1]
输出：9
```

---

## 解法：HashSet + 只从起点开始找

### 思路

暴力的排序解法是 O(n log n)，但题目要求 O(n)，那就要用 HashSet。

核心思想：**把数组全部丢进 HashSet，然后遍历 set——只从「没有前驱」的元素开始往后找**。

为什么只从没有前驱的元素开始？如果不是起点，找出来的序列一定是某个更长序列的子集，纯属浪费。

### 思考方式图解

```
nums = [100, 4, 200, 1, 3, 2]

set = {100, 4, 200, 1, 3, 2}

遍历 set 中的元素：

x=100 → set.contains(99)? false → 是起点！
    y=101 → set 中无 → 长度 = 101-100 = 1
    ans = max(0, 1) = 1

x=4   → set.contains(3)? true → 不是起点，跳过

x=200 → set.contains(199)? false → 是起点！
    y=201 → set 中无 → 长度 = 201-200 = 1
    ans = max(1, 1) = 1

x=1   → set.contains(0)? false → 是起点！
    y=2 → set 中有 → y++
    y=3 → set 中有 → y++
    y=4 → set 中有 → y++
    y=5 → set 中无 → 长度 = 5-1 = 4
    ans = max(1, 4) = 4 ✅

x=3   → set.contains(2)? true → 不是起点，跳过
x=2   → set.contains(1)? true → 不是起点，跳过

结果：4
```

### 代码实现

```java
class Solution {
    public int longestConsecutive(int[] nums) {
        Set<Integer> set = new HashSet<>();
        for (int i : nums) {
            set.add(i);
        }

        int ans = 0;
        for (int x : set) {                    // 遍历 set（自动去重）
            if (set.contains(x - 1)) {          // 有前驱 → 不是起点，跳过
                continue;
            }
            int y = x + 1;
            while (set.contains(y)) {            // 从起点往后数
                y++;
            }
            ans = Math.max(ans, y - x);          // 序列长度 = y - x
        }
        return ans;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 每个元素最多被检查两次（一次 `contains` 前驱，一次被 `while` 访问） |
| 🧠 空间复杂度 | **O(n)** — HashSet 存放所有元素 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | HashSet + 起点枚举 |
| 算法类型 | 哈希表 |
| 核心技巧 | **用 HashSet 去重 + O(1) 查邻居；只从没有前驱的元素（序列起点）开始查找** |
| 为什么不是 O(n²) | 内层 while 的总执行次数 = 所有连续序列的总长度 ≤ n，均摊 O(1) |
| 对比排序 | 排序 O(n log n) 也能做，但题目要求 O(n) |
| 易错点 | 循环遍历的是 **set** 而不是原数组——遍历数组会导致重复元素多算几次，虽然不影响最终答案但影响效率 |

### 一句话记住

> **「无前驱才起步，HashSet O(1) 查邻居——连续序列一次搞定。」**

---

## 题目：反转字符串（Reverse String）

**LeetCode 344 | 代码随想录 | 难度：🟢 简单**

### 题目描述

编写一个函数，其作用是将输入的字符串反转过来。输入字符串以字符数组 `char[] s` 的形式给出。

不要给另外的数组分配额外的空间，你必须**原地修改输入数组**、使用 **O(1) 的额外空间**解决这一问题。

### 示例

```
输入：s = ["h","e","l","l","o"]
输出：["o","l","l","e","h"]

输入：s = ["H","a","n","n","a","h"]
输出：["h","a","n","n","a","H"]
```

---

## 解法：双指针相向交换

### 思路

字符串反转是最基础的双指针应用——**一头一尾，相向而行，逐个交换**。

```
初始：  [h]  [e]  [l]  [l]  [o]
          ↑                   ↑
        left                right

交换后：[o]  [e]  [l]  [l]  [h]
                   ↑
                left/right → 结束
```

核心就是两个指针从两端向中间靠近，每次交换左右元素。

**注意点**：
- 循环条件是 `left < right`（而不是 `left <= right`），因为中间元素不需要和自己交换
- 数组元素个数为奇数时，中间那个元素不动

### 思考方式图解

```
s = ['h','e','l','l','o']

left=0, right=4 → 交换 'h' 和 'o' → ['o','e','l','l','h']
left=1, right=3 → 交换 'e' 和 'l' → ['o','l','l','e','h']
left=2, right=2 → left < right 不成立 → 结束 ✅
```

### 代码实现

```java
class Solution {
    public void reverseString(char[] s) {
        int n = s.length;
        for (int left = 0, right = n - 1; left < right; left++, right--) {
            char temp = s[left];
            s[left] = s[right];
            s[right] = temp;
        }
    }
}
```

> **简化写法**（不熟悉位运算可以不用，正常交换最清晰）：
> ```java
> // 异或交换（无需临时变量）
> s[left] ^= s[right];
> s[right] ^= s[left];
> s[left] ^= s[right];
> ```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 交换 n/2 次 |
| 🧠 空间复杂度 | **O(1)** — 仅用了几个变量 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 双指针交换 |
| 算法类型 | 字符串、双指针 |
| 核心技巧 | **一头一尾双指针相向而行，依次交换** |
| 适用场景 | 反转字符串、反转数组、反转链表（206）的思路同源 |
| 循环条件 | `left < right`，中间元素不动 |
| 易错点 | 数组长度为奇数时，中间一个元素不需要交换 |

### 一句话记住

> **「一头一尾相向走，逐对交换反转成。」**

---

## 题目：反转字符串 II（Reverse String II）

**LeetCode 541 | 代码随想录 | 难度：🟢 简单**

### 题目描述

给定一个字符串 `s` 和一个整数 `k`，从字符串开头算起，每计数至 `2k` 个字符，就反转这 `2k` 字符中的前 `k` 个字符。

- 如果剩余字符少于 `k` 个，则将剩余字符全部反转
- 如果剩余字符介于 `k` 和 `2k` 之间，则反转前 `k` 个字符，其余字符保持原样

### 示例

```
输入：s = "abcdefg", k = 2
输出："bacdfeg"

输入：s = "abcd", k = 2
输出："bacd"
```

---

## 解法：分段反转（每 2k 一组）

### 思路

直接按照题目规则模拟——**每 2k 个字符为一组，反转组内前 k 个**。

核心就是确定每组的反转区间 `[i, i+k-1]`，但最后一段可能不足 k 个，所以右边界要取 `min(i+k, n) - 1`。

```
分组规则（k=2）：
  2k = 4 个一组
  [0,1,2,3] → 反转 [0,1]
  [4,5,6,7] → 反转 [4,5]
  不足 4 个但 ≥ 2 → 反转前 2 个
  不足 2 个 → 全部反转
```

### 思考方式图解

```
s = "abcdefg", k = 2

i=0 (第 1 组):
  反转 [0, min(2,7)-1] = [0,1]
  "ab" → "ba"
  结果: "bacdefg"

i=4 (第 2 组):
  反转 [4, min(6,7)-1] = [4,5]
  "ef" → "fe"
  结果: "bacdfeg"

i=8 → i >= n 结束 ✅
```

### 代码实现

```java
class Solution {
    public String reverseStr(String s, int k) {
        char[] c = s.toCharArray();
        int n = c.length;

        // 每 2k 个字符为一组
        for (int i = 0; i < n; i += 2 * k) {
            // 反转组内前 k 个，不足 k 个则反转剩余全部
            reverse(c, i, Math.min(i + k, n) - 1);
        }

        return new String(c);
    }

    private void reverse(char[] c, int left, int right) {
        while (left < right) {
            char temp = c[left];
            c[left++] = c[right];
            c[right--] = temp;
        }
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n)** — 每个字符被反转一次 |
| 🧠 空间复杂度 | **O(n)** — `toCharArray()` 复制了一份 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 分段反转 |
| 算法类型 | 字符串、模拟 |
| 核心技巧 | **以 2k 为步长跳着走，`Math.min(i+k, n)-1` 处理最后一段边界** |
| 关联题目 | [344. 反转字符串](day8-哈希表hot100+字符串代码随想录.md)——复用的 `reverse` 辅助方法 |
| 易错点 | 右边界 `Math.min(i+k, n)-1`，不是 `i+k-1`（最后一组可能不满 k 个） |

### 一句话记住

> **「每 2k 跳一步，反转前 k 个——`Math.min` 兜住尾巴。」**

---

*练习日期：2026-07-17*
