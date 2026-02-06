

# 📝 LeetCode [3306] 元音辅音字符串计数 II - 深度复盘

### 1. 核心考点

* **多重约束滑窗**：题目同时包含了“数量约束”（辅音）和“集合约束”（元音）。
* **恰好 K 的转化**：如何将 `== k` 转化为滑动窗口可处理的不等式。
* **单调性方向判定**：识别题目条件是“越长越好”还是“越短越好”，从而决定使用 `AtMost` 还是 `AtLeast`。

### 2. 核心难点与陷阱

**题目要求**：

1. **必须包含所有 5 种元音** (`a, e, i, o, u`) —— **下限约束**（越长越容易满足）。
2. **辅音数量恰好为 k** —— **等值约束**。

#### ❌ 陷阱思维：惯用的 `AtMost` (至多 K)

* **思路**：尝试计算 `AtMost(k) - AtMost(k-1)`。
* **公式**：`ans += right - left + 1`。
* **致命缺陷**：
* `right - left + 1` 统计的是“当前窗口内部的所有子数组”（即更短的子数组）。
* **冲突**：虽然缩短窗口能满足“辅音 ”，但缩短窗口极易破坏“必须包含 5 种元音”的条件。
* **结果**：会把大量缺元音的非法子数组算入答案。



---

### 3. ✅ 正解思维：`AtLeast` (至少 K)

**转化思路**：


**为什么选 `AtLeast`？**

* **条件对齐**：
* 条件 A：包含 5 种元音  **越长越好**。
* 条件 B：辅音数量   **越长越好**。


* **单调性统一**：两个条件方向一致，都是“下限约束”。
* **统计逻辑**：
* 当窗口 `[left, right]` 满足条件时，它是一个“最小合法核心”。
* 那么 `[0...right]`, `[1...right]` ... `[left...right]` 肯定都合法（因为更长）。
* **公式**：`ans += left`（或者 `left + 1`，取决于指针定义）。统计的是“外部延伸”的合法起点。



---

### 4. 两种公式的本质对比（背诵全文）

| 维度 | `AtMost` (至多) | `AtLeast` (至少) |
| --- | --- | --- |
| **适用场景** | 上限约束（越短越好） | 下限约束（越长越好） |
| **典型题目** | [930] 和为 K, [1248] 优美子数组, [992] K个不同整数 | [3306] 元音辅音 II (含集合完整性约束) |
| **滑窗动作** | 爆了就缩 (`while > k` `left++`) | 够了就缩 (`while >= k` `left++`) |
| **统计公式** | `ans += right - left + 1`<br>

<br>(切蛋糕：算内部碎片) | `ans += left`<br>

<br>(滚雪球：算外部延伸) |
| **差分公式** | `f(k) - f(k-1)` | `f(k) - f(k+1)` |

---

### 5. 标准代码模板 (C++)

这是基于 `AtLeast` 逻辑的标准写法，逻辑清晰，无歧义。

```cpp
class Solution {
public:
    long long countOfSubstrings(string word, int k) {
        // 核心公式：恰好 k = 至少 k - 至少 k+1
        return f(word, k) - f(word, k + 1);
    }

    // 计算：包含所有 5 种元音 且 辅音数量 >= k 的子数组个数
    long long f(string& word, int k) {
        long long ans = 0;
        int left = 0;
        
        // 两个计数器互不干扰
        map<char, int> vowel_cnt; 
        int consonant_cnt = 0;
        string vowels = "aeiou";

        for (int right = 0; right < word.size(); right++) {
            // 1. 进窗口
            char r_char = word[right];
            if (vowels.find(r_char) != string::npos) {
                vowel_cnt[r_char]++;
            } else {
                consonant_cnt++;
            }

            // 2. 满足条件时（5种元音都有 且 辅音够了）
            // 注意：这里是 while，目的是找到“临界点”
            while (vowel_cnt.size() == 5 && consonant_cnt >= k) {
                char l_char = word[left];
                if (vowels.find(l_char) != string::npos) {
                    if (--vowel_cnt[l_char] == 0) {
                        vowel_cnt.erase(l_char);
                    }
                } else {
                    consonant_cnt--;
                }
                left++; 
            }
            
            // 3. 统计答案
            // while 循环结束时，left 停在“第一个不合法的位置”
            // 这意味着 [0, 1, ..., left-1] 都是合法的起点
            // 数量正好是 left
            ans += left;
        }
        return ans;
    }
};

```

### 6. 💡 进阶思考：为什么你的“错误代码”AC了？

(这一点作为你的独家心法)

* **现象**：你用了 `AtMost` 的名字，配合 `right - left + 1`，居然也过了。
* **本质**：你算的是 **“补集”**。
* 你的 `while` 是把所有“合法”的都跳过了。
* 剩下的 `right - left + 1` 统计的是 **“所有不合法（失败者）”** 的数量。


* **数学巧合**：
* 你的主函数：`Fail(k+1) - Fail(k)`。
* 等价推导：`[Total - Valid(k+1)] - [Total - Valid(k)]`。
* 结果：`Valid(k) - Valid(k+1)`。


* **结论**：这是容斥原理的补集形式。虽然是对的，但在面试中解释起来很费劲，**推荐使用上面的正向 `AtLeast` 解法**。
