---
title: 数位dp进阶
date: 2022-10-19 19:42:18
tags: [数位dp]
excerpt: 数位dp进阶理解和练习
categories: 算法模板
index_img: /img/index_img/4.png
banner_img: /img/banner_img/background5.jpg
---

<p class="note note-primary">不统计数量，而是其他数字相关量时，该如何使用数位dp求解</p>


## 简单方法

一个简单的方法是在数字填写完时，统计包含前缀在内的结果。

[数字 1 的个数 - 力扣（LeetCode）](https://leetcode.cn/problems/number-of-digit-one/) 

此时$dfs$和$dp$的意义是在一定前缀状态下，统计的1结果。这个结果是包含前缀中的1的，因此dp的状态需要包含cnt，这样才能确定一个dfs的答案，（比如前缀二个1，则后续返回的要加上2个1）。如果不包含，则$dp[i]$会不断变化。

```c++
class Solution {
public:
    int countDigitOne(int n) {
        string s = to_string(n);

        int dp[s.size()][s.size()];
        memset(dp, -1, sizeof(dp));
        // cnt 为前缀1的个数
        function<int(int, int, bool, bool)>dfs = [&](int i, int cnt, bool is_limit, bool is_num) -> int {
            if(i == s.size()) return cnt * is_num;

            if(!is_limit && is_num && dp[i][cnt] >= 0) return dp[i][cnt]; 

            int res = 0;
            if(!is_num) res += dfs(i + 1, cnt, false, false);
            
            int up = is_limit ? s[i] - '0': 9;
            
            for(int j = 1 - is_num; j <= up; j ++ ) {
                res += dfs(i + 1, cnt + (j == 1), is_limit && up == j, true);
            }
            if(!is_limit && is_num) dp[i][cnt] = res;
            return res;
        };
        return dfs(0, 0, true, false);
    }
};
```

使用这个方法，看似变麻烦了，但是节省空间的，下一道例题不得不用这个方法。此时dfs和dp的意义是在前缀状态下，统计后边的1的结果，这是不包含前缀中的1的，因此dp的状态一维即可。因为在计算所有子串中的结果时，不仅需要统计1的数量，还要统计合法的数字数量，因此使用node作为dp的值。
```c++
class Solution {
public:
    struct node {
        int cnt, cnt1;
    };
    int countDigitOne(int n) {
        string s = to_string(n);
        node dp[s.size()];
        memset(dp, -1, sizeof(dp));
        // cnt 为前缀1的个数
        function<node(int, int, bool, bool)>dfs = [&](int i, int cnt, bool is_limit, bool is_num) -> node {
            if(i == s.size()) return {is_num, 0};

            if(!is_limit && is_num && dp[i].cnt >= 0) return dp[i]; 

            node res = (node){0, 0};
            if(!is_num) res = dfs(i + 1, cnt, false, false);
            
            int up = is_limit ? s[i] - '0': 9;
            
            for(int j = 1 - is_num; j <= up; j ++ ) {
                node son = dfs(i + 1, cnt + (j == 1), is_limit && up == j, true);
                res.cnt += son.cnt;
                res.cnt1 += son.cnt * (j == 1) + son.cnt1;                
            }
            if(!is_limit && is_num) dp[i] = res;
            return res;
        };
        return dfs(0, 0, true, false).cnt1;
    }
};
```

## 进阶方法

[恨7不成妻 - AcWing题库](https://www.acwing.com/problem/content/description/1088/)

>思考如果还使用简单方法，那么需要将前缀信息放到dp状态中，为了确定在前缀状态下的平方和，则需要传递前缀数字，那么状态就太多了，dp爆内存！！！

注意$dp[p][sum][me]$中的$sum$表示前缀状态——各位数字之和，$node$中的$sum$表示数字之和。为了计算平方和，$node$设置了3个属性，数量、和、平方和。

考虑已知子串包含了$n$个合法数，其属性为：

$$son.cnt = \sum_{i=1}^{n}1$$
$$son.sum = \sum_{i=1}^{n}x_i$$
$$son.sum2 = \sum_{i=1}^{n}x_i^2$$

将数字$p$加到所有$son$的左边，所有的数字变成$(p * 10^k + x_i)$，则可以求解$sum$和$sum2$

$$sum = \sum_{i=1}^{up}{son.cnt * B + son.sum} $$
$$sum2 = \sum_{i=1}^{up}{B^2 * son.cnt + son.sum2 + 2 * B * son.sum}$$

```c++
#include <iostream>
#include <cstring>
#include <algorithm>
using namespace std;

#define int long long

struct node {
    int cnt, sum, sum2;
    
}dp[33][10][10];

int P = 1e9 + 7;
int mypow(int x) {
    int res = 1;
    while(x -- ) res = (res * 10) % P;
    return res;
}
int getdp(int x) {
    if(x == 0) return 0;
    string s = to_string(x);
    function<node(int, int, int, bool, bool)> dfs = [&](int p, int sum, int me, bool is_limit, bool is_num) -> node {
        me %= 7;
        sum %= 7;
        if(p == s.size()) return {me && sum, 0, 0};
        
        if(!is_limit && is_num && dp[s.size() - p][sum][me].cnt >= 0) return dp[s.size() - p][sum][me];
        
        node res = (node){0, 0, 0};
        if(!is_num) res = dfs(p + 1, sum, me, false, false);
        
        int up = is_limit ? s[p] - '0': 9;
        for(int i = 1 - is_num; i <= up; i ++ ) {
            if(i == 7) continue;
            node son =  dfs(p + 1, i + sum, i + me * 10, is_limit && i == up, true);
           
            int B = i * mypow(s.size() - p - 1);
            
            (res.cnt += son.cnt) %= P;
            (res.sum += son.sum + B * son.cnt) %= P;
            (res.sum2 += son.cnt * B % P * B % P + son.sum2 + 2 * B % P * son.sum % P) %= P;
        }
        
        if(!is_limit && is_num) dp[s.size() - p][sum][me] = res;
        return res;
    };
    return dfs(0, 0, 0, true, false).sum2;
}
main()
{
    int a, b, T;
    cin >> T;
    memset(dp, -1, sizeof dp);
    while(T --) {
        cin >> a >> b;    
        cout << (getdp(b) - getdp(a - 1) + P) % P << endl;   
    }

}
```