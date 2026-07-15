# Day 7 — 哈希表代码随想录（剩余）

## 题目：四数相加 II（4Sum II）

**LeetCode 454 | 代码随想录 | 难度：🟡 中等**

### 题目描述

给你四个整数数组 `nums1`、`nums2`、`nums3` 和 `nums4`，数组长度都是 `n`，请你计算有多少个元组 `(i, j, k, l)` 满足：

> `nums1[i] + nums2[j] + nums3[k] + nums4[l] == 0`

### 示例

```
输入：nums1 = [1,2], nums2 = [-2,-1], nums3 = [-1,2], nums4 = [0,2]
输出：2
解释：
两个元组如下：
1. (0, 0, 0, 1) → nums1[0] + nums2[0] + nums3[0] + nums4[1] = 1 + (-2) + (-1) + 2 = 0
2. (1, 1, 0, 0) → nums1[1] + nums2[1] + nums3[0] + nums4[0] = 2 + (-1) + (-1) + 0 = 0

输入：nums1 = [0], nums2 = [0], nums3 = [0], nums4 = [0]
输出：1
```

---

## 解法：分组 + HashMap

### 思路

暴力四层循环是 O(n⁴)，显然不可取。

核心思想：**把四数之和拆成两半，两两分组**。

> **AB 一组，CD 一组**：
> - 先统计 A+B 的所有可能和，以及每个和出现的次数 → 存入 HashMap
> - 再遍历 C+D，对于每个和，在 map 中查找 `0 - (c + d)`，累加次数

这样复杂度从 O(n⁴) 降到 **O(n²)**。

### 思考方式图解

```
nums1 = [1,2], nums2 = [-2,-1], nums3 = [-1,2], nums4 = [0,2]

Step 1: 统计 A+B 的所有和
  nums1\nums2  -2   -1
       1      -1    0       → map: {-1:1, 0:1}
       2       0    1       → map: {-1:1, 0:2, 1:1}

Step 2: 遍历 C+D，在 map 中找 -(c+d)
  nums3\nums4   0    2
      -1       -1   -3      → -(c+d): 1→res+=map.get(1)=1,  3→res+=0
       2       -2   -4      → -(c+d): 2→res+=0,  4→res+=0

结果：res = 2 ✅
```

### 代码实现

```java
class Solution {
    public int fourSumCount(int[] nums1, int[] nums2, int[] nums3, int[] nums4) {
        int res = 0;
        Map<Integer, Integer> map = new HashMap<>();

        // Step 1: 统计 A+B 的所有和及出现次数
        for (int i : nums1) {
            for (int j : nums2) {
                int sum = i + j;
                map.put(sum, map.getOrDefault(sum, 0) + 1);
            }
        }

        // Step 2: 遍历 C+D，找 -(c+d) 在 map 中的次数
        for (int i : nums3) {
            for (int j : nums4) {
                res += map.getOrDefault(0 - i - j, 0);
            }
        }

        return res;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(n²)** — 两个双层循环 |
| 🧠 空间复杂度 | **O(n²)** — HashMap 最多存储 n² 种和 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 分组 + HashMap |
| 算法类型 | 哈希表、分治 |
| 核心技巧 | **AB 一组、CD 一组，空间换时间——四数降两数** |
| 适用场景 | 多数组求和为定值，分组降低复杂度 |
| 对比 | 四数之和（一个数组内）用排序+双指针，四数相加 II（四个数组）用 HashMap |
| 易错点 | 用 `getOrDefault` 代替手动判空；`0 - i - j` 注意运算顺序 |

### 一句话记住

> **「两两分组算和，HashMap 次数累加。」**

---

## 题目：赎金信（Ransom Note）

**LeetCode 383 | 代码随想录 | 难度：🟢 简单**

### 题目描述

给你两个字符串：`ransomNote` 和 `magazine`，判断 `ransomNote` 能不能由 `magazine` 里面的字符构成。

如果可以，返回 `true`；否则返回 `false`。

`magazine` 中的每个字符只能在 `ransomNote` 中使用一次。

### 示例

```
输入：ransomNote = "a", magazine = "b"
输出：false

输入：ransomNote = "aa", magazine = "ab"
输出：false

输入：ransomNote = "aa", magazine = "aab"
输出：true
```

---

## 解法：数组计数（与字母异位词同款）

### 思路

和 Day 6 的「有效的字母异位词」几乎一模一样——都是 26 个小写字母的数组计数。

核心区别在于**方向**：

| | 字母异位词 | 赎金信 |
|---|-----------|--------|
| 判断条件 | 两个串**完全一样** | magazine **包含** ransomNote |
| 数组用法 | s 加、t 减，**全为 0** | magazine 加、ransom 减，**不能有负数** |
| 逻辑 | 互相抵消 | magazine 要"够用" |

**剪枝优化**：如果 `ransomNote.length() > magazine.length()`，直接返回 `false`，不用算了。

### 思考方式图解

```
ransomNote = "aa",  magazine = "aab"

Step 1: 统计 magazine
  record: a→2, b→1, 其余→0

Step 2: 消耗 ransomNote
  record: a→2-2=0, b→1

Step 3: 检查负数
  全部 >= 0 → true ✅
```

```
ransomNote = "aa",  magazine = "ab"

Step 1: 统计 magazine
  record: a→1, b→1

Step 2: 消耗 ransomNote
  record: a→1-2=-1  ← 负数！

Step 3: 存在负数 → false ❌
```

### 代码实现

```java
class Solution {
    public boolean canConstruct(String ransomNote, String magazine) {
        // 剪枝：赎金信比杂志长，肯定不够用
        if (ransomNote.length() > magazine.length()) {
            return false;
        }

        int[] record = new int[26];

        // 统计 magazine 中每个字母的出现次数
        for (char c : magazine.toCharArray()) {
            record[c - 'a'] += 1;
        }

        // 消耗 ransomNote 中每个字母
        for (char c : ransomNote.toCharArray()) {
            record[c - 'a'] -= 1;
        }

        // 如果存在负数，说明 magazine 不够用
        for (int i : record) {
            if (i < 0) {
                return false;
            }
        }

        return true;
    }
}
```

### 复杂度分析

| 维度 | 结果 |
|------|------|
| ⏱ 时间复杂度 | **O(m + n)** — 分别遍历两个字符串 |
| 🧠 空间复杂度 | **O(1)** — 固定 26 个字母的数组 |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 数组计数（数组哈希） |
| 算法类型 | 哈希表、字符串 |
| 核心技巧 | **magazine 先加后消，出现负数就说明不够用** |
| 适用场景 | 字符串包含关系判断、字母可用性检查 |
| 关联题目 | [242. 有效的字母异位词](day6-哈希表代码随想录.md)——同一套数组计数模板 |
| 易错点 | 剪枝容易忘——先比长度可以省掉很多无用计算 |

### 一句话记住

> **「杂志先存够，赎金再扣减，出现负数就不行。」**

---

## 题目：三数之和（3Sum）

**LeetCode 15 | 代码随想录 | 难度：🟡 中等**

### 题目描述

给你一个整数数组 `nums`，判断是否存在三元组 `[nums[i], nums[j], nums[k]]` 满足 `i != j`、`i != k` 且 `j != k`，同时还满足 `nums[i] + nums[j] + nums[k] == 0`。

请你返回**所有和为 0 且不重复**的三元组。

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

暴力三层循环是 O(n³)，显然不可取。

核心思想：**先排序，再固定一个数，然后用双指针找另外两个数**。

```
排序 → 固定 x = nums[i] → 双指针 j, k 在 [i+1, n-1] 中找 -x
```

**为什么用双指针而不用 HashMap？**

| | HashMap 版本 | 双指针版本 |
|---|-------------|-----------|
| 去重 | 麻烦——需要额外记录已用元素 | **天然支持去重**——排序后跳过相邻重复元素 |
| 空间 | O(n) 额外空间 | **O(1)**（不计排序） |
| 时间 | O(n²) | O(n²) |
| 适用性 | 设计巧妙但不通用 | 通用模板，一题学会 nSum 系列 |

**四项优化**：
1. **排序去重基础**：`i > 0 && x == nums[i-1]` — 跳过重复的固定元素
2. **剪枝一（提前终止）**：`x + nums[i+1] + nums[i+2] > 0` → 最小的三个数之和都大于 0，后面更大，直接 break
3. **剪枝二（当前轮跳过）**：`x + nums[n-2] + nums[n-1] < 0` → 当前 x 加上最大的两个数都小于 0，说明 x 太小，continue 到下一轮
4. **双指针去重**：找到一组后，`j++` 和 `k--` 跳过已经用过的重复数字

### 思考方式图解

```
nums = [-1,0,1,2,-1,-4]
排序后 = [-4,-1,-1,0,1,2]

i=0, x=-4
  j=1(-1), k=5(2) → s=-3 <0 → j++    [-4,-1,0,1,2]
  j=2(-1), k=5(2) → s=-3 <0 → j++
  j=3(0),  k=5(2) → s=-2 <0 → j++
  j=4(1),  k=5(2) → s=-1 <0 → j++ → j≥k 结束

i=1, x=-1  ← 前面有 -1，已被跳过不使用此处逻辑
  实际 i=2, x=-1  ← 跳过重复
  j=3(0),  k=5(2) → s=1  >0 → k--
  j=3(0),  k=4(1) → s=0  → [-1,0,1] ✅ j++,k--
  j=4(1),  k=3  → j≥k 结束

i=3, x=0
  j=4(1),  k=5(2) → s=3  >0 → k--
  j=4(1),  k=4  → j≥k 结束

i≥4, x 为正数，最小三数和 >0 → break

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

            // 跳过重复数字
            if (i > 0 && x == nums[i - 1]) continue;

            // 优化一：最小的三个数之和 > 0，后面不可能有解
            if (x + nums[i + 1] + nums[i + 2] > 0) break;

            // 优化二：当前数 + 最大的两个数 < 0，当前数太小，跳过
            if (x + nums[n - 2] + nums[n - 1] < 0) continue;

            int j = i + 1, k = n - 1;
            while (j < k) {
                int s = x + nums[j] + nums[k];
                if (s > 0) {
                    k--;
                } else if (s < 0) {
                    j++;
                } else {
                    ans.add(List.of(x, nums[j], nums[k]));
                    // 跳过重复数字
                    for (j++; j < k && nums[j] == nums[j - 1]; j++);
                    for (k--; k > j && nums[k] == nums[k + 1]; k--);
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
| 🧠 空间复杂度 | **O(1)**（不计排序及输出数组） |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 排序 + 双指针 |
| 算法类型 | 双指针、数组 |
| 核心技巧 | **先排序固定一个数，双指针线性扫描剩下两个** |
| 去重策略 | 排序后跳过相邻相同元素，三层去重（i、j、k 各一层） |
| 优化项 | 最小三数和 > 0 直接 break；当前 + 最大两个 < 0 直接 continue |
| 适用场景 | nSum 系列问题的核心模板 |
| 易错点 | 去重逻辑：`i > 0 && x == nums[i-1]` 是跳过已处理的重复，**不是**跳过后面的重复值（即只保留第一个重复） |

### 一句话记住

> **「排序定一，双指针扫二——剪枝去重一步到位。」**

---

## 题目：四数之和（4Sum）

**LeetCode 18 | 代码随想录 | 难度：🟡 中等**

### 题目描述

给你一个由 `n` 个整数组成的数组 `nums` 和一个目标值 `target`，请你找出并返回满足下述全部条件且**不重复**的四元组 `[nums[a], nums[b], nums[c], nums[d]]`：

- `0 <= a, b, c, d < n`
- `a、b、c` 和 `d` **互不相同**
- `nums[a] + nums[b] + nums[c] + nums[d] == target`

### 示例

```
输入：nums = [1,0,-1,0,-2,2], target = 0
输出：[[-2,-1,1,2],[-2,0,0,2],[-1,0,0,1]]

输入：nums = [2,2,2,2,2], target = 8
输出：[[2,2,2,2]]
```

---

## 解法：排序 + 双指针（nSum 模板）

### 思路

四数之和是**三数之和的嵌套扩展**——外面再套一层循环就对了。

```
三数之和：  固定 i → 双指针 j, k
四数之和：  固定 a → 固定 b → 双指针 j, k
```

这样从 O(n⁴) 降到 **O(n³)**，三数之和的剪枝和去重策略全部复用过来。

**与三数之和对比：**

| | 三数之和 | 四数之和 |
|---|---------|---------|
| 外层循环 | 固定 1 个数 | 固定 **2** 个数（双层） |
| 内层双指针 | 1 层 while | 1 层 while |
| 去重 | 3 层（i, j, k） | **4 层**（a, b, j, k） |
| 剪枝 | 2 条（break + continue） | **4 条**（内外各 2 条） |
| 溢出风险 | 不存在 | `int` 相加可能溢出 → 用 **`long`** |

**关键细节：`long` 类型**——`nums` 是 `int[]`，但四个 `int` 相加可能溢出（如 `10⁹ + 10⁹ + 10⁹ + 10⁹ > 2³¹-1`），所以 `x`、`y`、`s` 都声明为 `long`。

### 思考方式图解

```
nums = [1,0,-1,0,-2,2], target = 0
排序后 = [-2,-1,0,0,1,2]

a=0, x=-2
  b=1, y=-1
    j=2(0), k=5(2) → s=-1 <0 → j++
    j=3(0), k=5(2) → s=-1 <0 → j++
    j=4(1), k=5(2) → s=0  → [-2,-1,1,2] ✅
  b=2, y=0  ← b>a+1 且 y==nums[1]=0? 不，y(0)≠nums[1](-1)
    j=3(0), k=5(2) → s=0  → [-2,0,0,2] ✅
  b=3, y=0, 与 nums[2] 重复 → continue
  ...后续 b 的剪枝...

a=1, x=-1
  b=2, y=0
    j=3(0), k=5(2) → s=1 >0 → k--
    j=3(0), k=4(1) → s=0  → [-1,0,0,1] ✅
  ...

结果：[[-2,-1,1,2], [-2,0,0,2], [-1,0,0,1]]
```

### 代码实现

```java
class Solution {
    public List<List<Integer>> fourSum(int[] nums, int target) {
        Arrays.sort(nums);
        List<List<Integer>> ans = new ArrayList<>();
        int n = nums.length;

        for (int a = 0; a < n - 3; a++) {
            long x = nums[a];

            // 跳过重复
            if (a > 0 && x == nums[a - 1]) continue;

            // 剪枝一：最小的四个数之和 > target → 不可能
            if (x + nums[a + 1] + nums[a + 2] + nums[a + 3] > target) break;

            // 剪枝二：当前数 + 最大的三个数 < target → 当前数太小
            if (x + nums[n - 1] + nums[n - 2] + nums[n - 3] < target) continue;

            for (int b = a + 1; b < n - 2; b++) {
                long y = nums[b];

                // 跳过重复（注意：b 只跳过 a+1 之后的重复）
                if (b > a + 1 && y == nums[b - 1]) continue;

                // 剪枝三：a+b+最小的两个 > target → break
                if (x + y + nums[b + 1] + nums[b + 2] > target) break;

                // 剪枝四：a+b+最大的两个 < target → continue
                if (x + y + nums[n - 1] + nums[n - 2] < target) continue;

                int j = b + 1, k = n - 1;
                while (j < k) {
                    long s = x + y + nums[j] + nums[k];
                    if (s < target) {
                        j++;
                    } else if (s > target) {
                        k--;
                    } else {
                        ans.add(List.of((int) x, (int) y, nums[j], nums[k]));
                        // 跳过重复
                        for (j++; j < k && nums[j] == nums[j - 1]; j++);
                        for (k--; j < k && nums[k] == nums[k + 1]; k--);
                    }
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
| ⏱ 时间复杂度 | **O(n³)** — 三层循环 + 双指针扫描 |
| 🧠 空间复杂度 | **O(1)**（不计排序及输出数组） |

---

## 小总结

| 要点 | 说明 |
|------|------|
| 算法名称 | 排序 + 双指针 |
| 算法类型 | 双指针、数组 |
| 核心技巧 | **三数之和外面再套一层循环，nSum 系列模板化** |
| 去重策略 | 4 层去重（a、b、j、k 各一层），注意 b 的去重起点是 `a+1` |
| 剪枝优化 | 内外各 2 条剪枝：最小的 N 个 > target break；当前 + 最大的 N 个 < target continue |
| 与三数之区别 | 多一层循环 + `long` 防溢出 + 剪枝多 2 条 |
| 易错点 | **`long`**——四个 int 相加可能溢出，`x`、`y`、`s` 都要用 `long` |

### 一句话记住

> **「三数套一层就是四数，long 防溢出，剪枝翻倍。」**

---

*练习日期：2026-07-15*
