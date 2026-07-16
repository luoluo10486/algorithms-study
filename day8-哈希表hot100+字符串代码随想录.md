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

*练习日期：2026-07-16*
