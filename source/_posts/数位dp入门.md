---
title: 数位dp入门
date: 2022-10-19 18:42:18
tags: [数位dp]
excerpt: 数位dp的两个模板
categories: 算法模板
index_img: /img/index_img/3.png
banner_img: /img/banner_img/background2.jpg
---

## 简介
数位DP往往都是这样的题型，给定一个闭区间$[l,r]$，让你求这个区间中满足某种条件的数的总数。而这个区间可能很大，简单的暴力代码如下：
```c++
int ans=0;
for(int i=l;i<=r;i++){
    if(check(i))ans++;
}
```
![](https://raw.githubusercontent.com/univwang/img/main/tmp629F.png)


## 解题思路

一般题目可以通过求解$dp[0, r]$和$dp[0, l - 1]$，两式相减得到答案。
有两种解题模板，一种是提前预处理0-9数字任意取的情况下的答案（也就是左支），然后按照树的顺序遍历数字a。第二种是使用**记忆化搜索**的方法，代码量低

<a class="btn" target="_blank" rel="noopener" style="font-size:20px; color: green" href="https://leetcode.cn/problems/count-special-integers/" title="github">例题：统计特殊整数</a>

## 模板一

```c++
class Solution {
public:
    int jie[12];
    void init() {
        //互不相同的数字，那么就是0-9拿出数进行排列
        //初始化阶乘，然后
        jie[0] = 1;
        for(int i = 1; i < 12; i ++ ) {
            jie[i] = jie[i - 1] * i;
        }
    }
    int A(int m, int n) {
        //从n个数中取m个数
        if(n < m) return 0;
        return jie[n] / jie[n - m];
    }

    int dp(int x) {
        if(x == 0) return 1;
        vector<int> num;
        while(x) num.push_back(x % 10), x /= 10;
        vector<bool> mp(10);//记录10个数是否已经取到

        int res = 0;
        for(int i = num.size() - 1; i >= 0; i -- ) {
            int t = num[i];

            if(t) {
                //可以选择0 -- t-1 的数
                //---i
                for(int j = 1; j < t; j ++) {
                    if(!mp[j]) {
                        res += A(i, 10 - num.size() + i);
                    }

                }
                //第一位不选0
                if(i != num.size() - 1 && !mp[0]) {
                    res += A(i, 10 - num.size() + i);
                }
            }

            //检查能否设置为t
            if(mp[t]) break;
            else {
                mp[t] = 1;
                if(i == 0) res ++;
            }
        }

        //累加前导0的部分，也就是长度为1---size-1
        //第一位是1-9，然后后边的随便从0-9，剔除第一个数选i - 1个
        // cout << res;
        for(int i = 1; i < num.size(); i ++ ) {
            res += 9 * A(i - 1, 9);
        }
        res ++;
        return res;

    }
    int countSpecialNumbers(int n) {
        //我们认为0也是特殊整数
        //1. 初始化，当数字任意取有什么样的结果，这是需要提前计算的？
        init();
        return dp(n) - dp(0);
    }
};
```


## 模板二（推荐）

$dp$数组的维度设置要可以确定后续数组的选择。如该例题，$dp[i][mask]$表示前缀已选$mask$的状态下，剩下的数字随便选的结果。因为只要已选数字的$mask$确定了，后续如何选择是固定的

```c++
class Solution {
public:
    int countSpecialNumbers(int n) {
        string s = to_string(n);//得到n的字符串
        int m = s.size(), dp[m][1 << 11];
        memset(dp, -1, sizeof(dp));

        /*函数参数说明
           @PARAM i:当前的层数
           @PARAM mask: 状态变量，此处为已选择数字
           @PARAM is_limit: 当前位是否有限制
           @PARAM is_num: 是否已填入数字 
        */
        function<int(int, int, bool, bool)> dfs = [&](int i, int mask, bool is_limit, bool is_num) -> int {

            //如果能遍历到最后，则说明结果成立，判断是否填入数字
            if(i == s.size()) return is_num; 

            //记忆化搜索，如果已经有了记录的dp状态，则返回
            if(!is_limit && is_num && dp[i][mask] >= 0) return dp[i][mask];
            int res = 0;
            //如果没有填数字，可以继续跳过
            if(!is_num) res += dfs(i + 1, mask, false, false);
            //计算遍历数字的上限
            int up = is_limit ? s[i] - '0': 9;
            for(int j = 1 - is_num; j <= up; j ++ ) {
                if((mask >> j & 1) == 0)
                    res += dfs(i + 1, mask|(1 << j), is_limit && j == up, true);
            }
            //更新dp数组
            if(!is_limit && is_num) dp[i][mask] = res;
            return res;
        };
        return dfs(0, 0, true, false);//从最高位开始，其中is_limit为true
    }
};
```