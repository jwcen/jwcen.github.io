---
layout: post
title:  "[算法] 02-同向/相向双指针"
categories: [Algorithms]
tags: [双指针]
permalink: /posts/:categories/:title:output_ext
math: true
---

> 如需转载，请附上链接：[https://jwcen.github.io/](https://jwcen.github.io/)
{: .prompt-tip}

* This will become a table of contents (this text will be scrapped).
{:toc}

## 同向双指针

### 209. 长度最小的子数组

思路：双指针的滑动窗口

> left 移动时子数组的和不断变小，直到满足要求，满足单调性。

时间复杂度：O(n)

> 非$ O(n^2) $，二重循环的复杂度可以理解为 $left+=1 $ 的次数以及 $ right += 1 $ 的次数，至多加到 $ n $

{% details Golang %}
~~~go
func minSubArrayLen(target int, nums []int) int {
    n := len(nums) 
    left := 0
    sum := 0  
    ans := n + 1 
    for right, num := range nums {  // 枚举右端点
        sum += num 
        for sum >= target {
            ans = min(ans, right-left+1)
            sum -= nums[left]
            left++
        }
    }
    if ans <= n {
        return ans 
    }
    return 0 
}
func min(a, b int) int { if a < b { return a }; return b }
~~~
{% enddetails %}

{% details Python code %}
~~~python
class Solution:
    def minSubArrayLen(self, target: int, nums: List[int]) -> int:
        n = len(nums)
        left = 0
        ans = n + 1 
        s = 0  
        for right, num in enumerate(nums):  # 枚举右端点
            s += num 
            while s >= target:
                ans = min(ans, right-left+1)
                s -= nums[left] 
                left += 1

        return ans if ans <= n else 0 
~~~
{% enddetails %}


### [713. 乘积小于 K 的子数组](https://leetcode.cn/problems/subarray-product-less-than-k/)

给你一个整数数组 `nums` 和一个整数 `k` ，请你返回子数组内所有元素的乘积严格小于 `k` 的连续子数组的数目。

```wiki
输入：nums = [10,5,2,6], k = 100
输出：8
解释：8 个乘积小于 100 的子数组分别为：[10]、[5]、[2],、[6]、[10,5]、[5,2]、[2,6]、[5,2,6]。
需要注意的是 [10,5,2] 并不是乘积小于 100 的子数组。
```

> 思路：方法同上，若元素的乘积>=k就把左端点右移，缩小子数组长度，直到乘积小于k。

{% details Golang code %}
~~~go
func numSubarrayProductLessThanK(nums []int, k int) int {
    if k <= 1 {
        return 0 
    }

    ans := 0 // 初始子数组数目
    prod := 1 // 初始乘积
    left := 0 
    for right, n := range nums {
        prod *= n 
        for prod >= k {
            prod /= nums[left]
            left++
        }
        ans += right-left+1
    }
    return ans 
}
~~~
{% enddetails %}


### [3. 无重复字符的最长子串]() ⭐⭐⭐

> 思路：双指针滑窗+hashmap

{% details Golang code %}
~~~go
func lengthOfLongestSubstring(s string) int {
    // freq := map[byte]int{} // key:byte, val:int
    freq := make([]byte, 128) // s 由英文字母、数字、符号和空格组成
    ans := 0 
    left := 0 
    for right := range s {
        freq[s[right]]++
        for freq[s[right]] > 1 {
            freq[s[left]]--
            left++
        }
        ans = max(ans, right-left+1) // 字符个数
    }
    return ans 
}

func max(a, b int) int { if a < b { return b }; return a }
~~~
{% enddetails %}

- 时间复杂度：$ O(n) $
- 空间复杂度：$ O(128)$ or $O(len(set(s)))$ or $O(1)$，常数空间

### [1004. 最大连续1的个数 III](https://leetcode.cn/problems/max-consecutive-ones-iii/description/)

{% details Golang code %}
~~~go
func longestOnes(nums []int, k int) int {
    ans := 0 
    left := 0 
    for right, num := range nums {
        if num == 0 {
            k-- // 相当于翻转
        }

        for k < 0 { // 收缩窗口
            if nums[left] == 0 {
                k++
            }
            left++
        }
        
        ans = max(ans, right-left+1)
    }
    return ans 
}

func max(a, b int) int { if a < b { return b }; return a }
~~~
{% enddetails %}

## 相向双指针

### [1. 两数之和]()

> 思路：hashmap

{% details Golang code %}
~~~go
func twoSum(nums []int, tar int) []int {
    m := map[int]int{} // key存元素,val存索引
    for i, num := range nums {
        if _, exist := m[tar-num]; exist {
            return []int{i, m[tar-num]}
        }
        m[num] = i 
    }  
    return []int{}
} 
~~~
{% enddetails %}

### [167. 两数之和2-有序数组]()

> 注意题目条件：有序！！！直接使用相向双指针
{: .prompt-warning}

{% details Golang code %}
~~~go
func twoSum(nums []int, target int) []int {
    left, right := 0, len(nums)-1
    for true {
        sum := nums[left] + nums[right]
        if sum == target {
            break 
        }

        if sum < target {
            left++
        } else { 
            right--
        }
    }
    return []int{left+1, right+1}
}
~~~
{% enddetails %}
- 时间：$O(n)$-- 每次通过大小关系直接去掉了一个数，每次花费 $O(1)$



### [15. 三数之和](https://leetcode.cn/problems/3sum/)⭐⭐⭐

你返回所有和为 0 且不重复的三元组。

> 时间 $O(n^2)$

{% details Golang code %}
~~~go
func threeSum(nums []int) [][]int {
    // 三元组的顺序并不重要---那就规定一个顺序: i < j < k 
    // 不可以包含重复的三元组---排序，去重
    sort.Ints(nums)
    ans := make([][]int, 0)
    n := len(nums)
    for i := 0; i < n-2; i++ { // n-2是为保证是三元组
        x := nums[i] 
        // 去重
        if i > 0 && nums[i-1] == x {
            continue 
        }
        // 优化1--x和最小的两个数加起来都大于0,那后面不会有等于0的情况了
        if x + nums[i+1] + nums[i+2] > 0 {
            break
        }
        // 优化2--x和最大的两个数加起来小于0,跳出本次循环，后面有可能等于0的情况
        if x + nums[n-2] + nums[n-1] < 0 {
            continue
        }

        // 转为【两数之和2】问题
        j := i + 1 
        k := n - 1 
        for j < k {
            sum := x + nums[j] + nums[k] 
            if sum < 0 {
                j++
            } else if sum > 0 {
                k-- 
            } else {
                tmp := []int{x, nums[j], nums[k]}
                ans = append(ans, tmp)
                j++
                k-- 
                // j k 也要去重
                for j < k && nums[j] == nums[j-1] { j++ }
                for j < k && nums[k] == nums[k+1] { k-- }
            }
        }
    }
    return ans
}
~~~
{% enddetails %}

### 11. 盛最多水的容器

给定一个长度为 `n` 的整数数组 `height` 。有 `n` 条垂线，第 `i` 条线的两个端点是 `(i, 0)` 和 `(i, height[i])` 。
找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水。返回容器可以储存的最大水量。

{% details Golang code %}
~~~go
func maxArea(height []int) int {
    n := len(height) 
    if n < 2 { return 0 }

    ans := 0 
    i, j := 0, n-1
    for i < j {
        area := (j - i) * min(height[i], height[j])
        ans = max(ans, area)
        if height[i] < height[j] {
            i++
        } else {
            j--
        }
    }

    return ans 
}

func max(a, b int) int { if a < b { return b }; return a }
func min(a, b int) int { if a > b { return b }; return a }
~~~
{% enddetails %}

### 42. 接雨水⭐⭐⭐

思路：使用两个额外的数组：
- pre_max数组存储从左到第i各位置的最大高度---前缀最大值
- suf_max从右......最大高度---后缀最大值

- 对每个前缀最大值，用上一个前缀最大值和当前高度取最大值得到当前前缀最大值

- 对每个后缀最大值，同理得道当前后缀最大值

- 最后同时遍历高度h、前后缀最大值，计算

  - **curWater = min(前缀最大值[i]，后缀最大值[i]) - height[i]**

{% details Python code %}
~~~python
class Solution:
    def trap(self, height: List[int]) -> int:
        n = len(height)
        pre_max = [0] * n
        pre_max[0] = height[0]
        for i in range(1, n):
            pre_max[i] = max(pre_max[i-1], height[i])

        suf_max = [0] * n 
        suf_max[-1] = height[-1]
        for i in range(n-2, -1, -1):
            suf_max[i] = max(suf_max[i+1], height[i])

        ans = 0
        for h, pre, suf in zip(height, pre_max, suf_max):
            # 水桶的长度 - 高度 = 能接多少水
            ans += min(pre, suf) - h 
        
        return ans 
  
  # 时空O(n)
~~~
{% enddetails %}

优化：空间O(1)，思路：
- 对于数组的每一个元素，找到其左侧和右侧的最高点，然后取两个最高点较小者，减去当前元素的高度，就是当前位置可存储的雨水量。

{% details Python code %}
~~~python
class Solution:
    def trap(self, height: List[int]) -> int:
        n = len(height) 
        pre_max = 0 
        suf_max = 0 
        left, right = 0, n-1
        ans = 0 
        while left <= right:
            pre_max = max(pre_max, height[left])
            suf_max = max(suf_max, height[right])

            if pre_max < suf_max:
                ans += pre_max - height[left]
                left += 1 
            else:
                ans += suf_max - height[right]
                right -= 1 
        
        return ans 
~~~
{% enddetails %}


------
题目来源
> [0x3f]()  
> leetcode.cn

> 如需转载，请附上链接：[https://jwcen.github.io/](https://jwcen.github.io/)
{: .prompt-tip}