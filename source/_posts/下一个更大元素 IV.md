---
title: 下一个更大元素 IV
date: 2022-10-24 16:42:18
tags: [单调栈, 有序集合]
excerpt: 使用双单调栈或者有序集合
categories: Leetcode
index_img: /img/index_img/11.png
banner_img: /img/banner_img/background13.jpg
---

[2454. 下一个更大元素 IV - 力扣（LeetCode）](https://leetcode.cn/problems/next-greater-element-iv/)

## 题目介绍
给你一个下标从 **0** 开始的非负整数数组 `nums` 。对于 `nums` 中每一个整数，你必须找到对应元素的 **第二大** 整数。

如果 `nums[j]` 满足以下条件，那么我们称它为 `nums[i]` 的 **第二大** 整数：

-   `j > i`
-   `nums[j] > nums[i]`
-   恰好存在 **一个** `k` 满足 `i < k < j` 且 `nums[k] > nums[i]` 。

如果不存在 `nums[j]` ，那么第二大整数为 `-1` 。

-   比方说，数组 `[1, 2, 4, 3]` 中，`1` 的第二大整数是 `4` ，`2` 的第二大整数是 `3` ，`3` 和 `4` 的第二大整数是 `-1` 。

请你返回一个整数数组 `answer` ，其中 `answer[i]`是 `nums[i]` 的第二大整数。

  

**示例 1：**

```txt
输入：nums = [2,4,0,9,6]
输出：[9,6,6,-1,-1]
解释：
下标为 0 处：2 的右边，4 是大于 2 的第一个整数，9 是第二个大于 2 的整数。
下标为 1 处：4 的右边，9 是大于 4 的第一个整数，6 是第二个大于 4 的整数。
下标为 2 处：0 的右边，9 是大于 0 的第一个整数，6 是第二个大于 0 的整数。
下标为 3 处：右边不存在大于 9 的整数，所以第二大整数为 -1 。
下标为 4 处：右边不存在大于 6 的整数，所以第二大整数为 -1 。
所以我们返回 [9,6,6,-1,-1] 。
```

**示例 2：**

```txt
输入：nums = [3,3]
输出：[-1,-1]
解释：
由于每个数右边都没有更大的数，所以我们返回 [-1,-1] 。
```
  

**提示：**

-   `1 <= nums.length <= 10^5`
-   `0 <= nums[i] <= 10^9`


## 解题思路

寻找一个数右边第二个比他大的数，可以使用set来维护比它大的数的下标，然后找到第二个下标即可。具体操作是从最大的数开始枚举，得到答案后将下标加入set，
那么后边的数即可使用lower_bound函数求解第一个大于它的下标，地址+1即第二大的数的下标。

## 代码
```c++
class Solution {
public:
    vector<int> secondGreaterElement(vector<int>& w) {
        int n = w.size();
        vector<int> res(n, -1);
        vector<int> p;
        for(int i = 0; i < n; i ++) p.push_back(i);

        //排序
        sort(p.begin(), p.end(), [&](int x, int y) {
            return w[x] > w[y];
        });
        set<int> S;
        S.insert(n + 1);
        S.insert(n + 2);

        for(int i = 0; i < n ; i ++ ) {
            int j = i + 1;
            while(j < n && w[p[j]] == w[p[i]]) j ++;
            //[i, j)
            
            for(int k = i; k < j; k ++ ) {
                auto it = S.upper_bound(p[k]);
                ++ it;
                if(*it < n) res[p[k]] = w[*it];
            }

            for(int k = i; k < j; k ++ ) {
                S.insert(p[k]);
            }

            i = j - 1;
        }
        return res;
    }
};
```
